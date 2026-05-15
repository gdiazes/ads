# GUÍA DE LABORATORIO: Clúster HA con Ubuntu Server y TrueNAS (Storage Centralizado)

**Objetivo:** Implementar un clúster de alta disponibilidad utilizando dos nodos **Ubuntu Server** y un almacenamiento centralizado **TrueNAS**. Se aplicará el modelo OSI para el diagnóstico y la configuración.

---

## 1. Topología Vertical de Bajo Nivel (Modelo OSI L7 -> L1)

```text
=============================================================================
         TOPOLOGÍA VERTICAL: CLÚSTER UBUNTU + STORAGE CENTRALIZADO
=============================================================================
[ RED VIRTUAL L2 / VMnet8 (NAT) ] Subred: 10.160.10.0/24 | GW: 10.160.10.1
=============================================================================
                                     |
                       [ L7 ] CLIENTE (Navegador Web)
                       [ L3 ] IP: 10.160.10.50
                                     |
                                     v
                  +---------------------------------------+
                  |    [ L3 ] VIP (IP FLOTANTE): .100     |
                  |    [ L2 ] Mecanismo: GARP             |
                  +---------------------------------------+
                                     |
            +------------------------+------------------------+
            |                                                 |
            v                                                 v
+-----------------------+                         +-----------------------+
|        NODO 1         |                         |        NODO 2         |
| [SYS] Ubuntu Server   |  <--- [ L4 ] UDP ---->  | [SYS] Ubuntu Server   |
| [ L7] Host: node1     |      5404 / 5405        | [ L7] Host: node2     |
| [ L4] TCP: 80, 2224   |      (Corosync)         | [ L4] TCP: 80, 2224   |
| [ L3] IP: 10.160.10.11|                         | [ L3] IP: 10.160.10.12|
| [ L2] MAC: ...00:01   |                         | [ L2] MAC: ...00:02   |
| [ L1] IFZ: eth0       |                         | [ L1] IFZ: eth0       |
+-----------------------+                         +-----------------------+
            |                                                 |
            +------------------------+------------------------+
                                     |
                                     | [ L4 ] TCP: 2049 (NFSv4)
                                     v
                  +---------------------------------------+
                  |         NODO 3: TRUENAS (SPOF)        |
                  | [SYS] TrueNAS SCALE / CORE            |
                  | [ L7] NFS: /mnt/pool1/webdata         |
                  | [ L3] IP: 10.160.10.20                |
                  | [ L2] MAC: ...00:03 | MTU: 1500       |
                  +---------------------------------------+
```



## 2. Procedimiento de Configuración

### FASE 1: Preparación del Almacenamiento (TrueNAS)
**Configuración en Nodo 3:**
1. IP Estática: `10.160.10.20/24`.
2. Crear un **Pool** y un **Dataset** en `/mnt/pool1/webdata`.
3. Exportar vía **NFS** con la opción `Mapall User: root` para evitar errores de permisos en el clúster.



### FASE 2: Preparación de Nodos Ubuntu (Nodo 1 y 2)

**Paso 1: Instalación de paquetería base**
```bash
sudo apt update && sudo apt install -y pacemaker pcs corosync fence-agents nfs-common apache2
```
> **Nota:** Se instalan las herramientas de clúster, el cliente NFS para conectar con TrueNAS y el servidor web Apache2.
> **Resultado esperado:** Los servicios se instalan correctamente; `apache2` e `hacluster` (usuario) se crean en el sistema.

**Paso 2: Resolución de nombres y Firewall**
```bash
sudo ufw allow 22/tcp && sudo ufw allow 2224/tcp && sudo ufw allow 5404:5405/udp && sudo ufw allow 80/tcp && sudo ufw allow 2049/tcp && sudo ufw enable
```
> **Nota:** Se abren los puertos para gestión (2224), comunicación del clúster (Corosync), tráfico web (80) y almacenamiento (2049).
> **Resultado esperado:** "Firewall is active and enabled on system startup".

**Paso 3: Identidad y Autenticación**
```bash
sudo passwd hacluster
sudo systemctl enable --now pcsd
```
> **Nota:** Se asigna una contraseña al usuario del clúster y se enciende el demonio de configuración `pcsd` (Puerto 2224).
> **Resultado esperado:** El servicio `pcsd` queda en estado *active (running)*.



### FASE 3: Creación del Clúster (Solo en Nodo 1)

**Paso 1: Autenticación de nodos**
Asignar la ip a cada nodo y modificiar el archivo /etc/hosts con el nombre e ip de cada nodo
```bash
sudo pcs host auth node1 node2 -u hacluster -p password123
```
> **Nota:** El Nodo 1 se comunica con el Nodo 2 vía TCP/2224 para intercambiar llaves de seguridad.
> **Resultado esperado:** `node1: Authorized`, `node2: Authorized`.

**Paso 2: Configuración e inicio**
```bash
sudo pcs cluster setup mi-cluster node1 node2
sudo pcs cluster start --all
sudo pcs cluster enable --all
```
> **Nota:** Se genera el archivo `corosync.conf` y se arrancan los servicios de alta disponibilidad en ambos servidores de forma simultánea.
> **Resultado esperado:** El clúster se sincroniza y ambos nodos aparecen como "Online" al ejecutar `pcs status`.

**Paso 3: Ajuste de propiedades de seguridad**
```bash
sudo pcs property set stonith-enabled=false
sudo pcs property set no-quorum-policy=ignore
```
> **Nota:** Desactivamos el Fencing (STONITH) por ser un entorno virtual sin hardware de energía y permitimos que el clúster funcione incluso con un solo nodo vivo.
> **Resultado esperado:** Pacemaker aceptará configurar recursos sin errores de validación.



### FASE 4: Definición de Recursos Lógicos (Solo en Nodo 1)

**Paso 1: Crear la IP Flotante (VIP - Capa 3)**
```bash
sudo pcs resource create Cluster_VIP ocf:heartbeat:IPaddr2 ip=10.160.10.100 cidr_netmask=24 op monitor interval=20s
```
> **Nota:** Crea la dirección IP que los clientes verán. El monitor revisa cada 20 segundos que la IP esté asignada.
> **Resultado esperado:** La IP `10.160.10.100` aparece como "Started" en el Nodo 1.

**Paso 2: Crear el Montaje NFS (Storage - Capa 7)**
```bash
sudo pcs resource create Web_Storage ocf:heartbeat:Filesystem device="10.160.10.20:/mnt/pool1/webdata" directory="/var/www/html" fstype="nfs" options="rw,sync,hard" op monitor interval=20s
```
> **Nota:** Ordena al clúster montar el disco de TrueNAS. La opción `hard` evita corrupción ante latencia.
> **Resultado esperado:** El comando `df -h` mostrará el montaje NFS solo en el nodo activo.

**Paso 3: Crear el Servidor Web (App - Capa 7)**
```bash
sudo pcs resource create WebServer ocf:heartbeat:apache configfile=/etc/apache2/apache2.conf op monitor interval=30s
```
> **Nota:** Pacemaker toma el control del servicio Apache. Nota que la ruta en Ubuntu es distinta a la de Rocky.
> **Resultado esperado:** El servicio web inicia bajo la supervisión del clúster.

**Paso 4: Aplicar Restricciones de Colocalización y Orden**
```bash
sudo pcs constraint colocation add WebServer with Web_Storage INFINITY
sudo pcs constraint order Web_Storage then WebServer
```
> **Nota:** La colocalización obliga a que Apache y el Disco estén en el mismo nodo. El orden asegura que el disco se monte *antes* de iniciar Apache.
> **Resultado esperado:** Al mover un recurso, todos los demás "viajan" juntos automáticamente.



### FASE 5: Pruebas de Failover (Chaos Engineering)

**Acción:** Apagar bruscamente el Nodo 1 (Power Off en VMware).

**Resultado esperado:**
1.  En el Nodo 2, el comando `pcs status` mostrará al Nodo 1 como "Offline".
2.  El Nodo 2 enviará un **GARP** (Gratuitous ARP) para reclamar la IP `.100`.
3.  El Nodo 2 montará automáticamente el recurso NFS desde el TrueNAS `.20`.
4.  Al refrescar `http://10.160.10.100` en el cliente, la página web seguirá cargando el contenido original almacenado en TrueNAS.


###  Sugerencia:
Si el montaje falla, ejecute `rpcinfo -p 10.160.10.20`. 
*   **Si no responde:** Falla Capa 3/4 (Red o Firewall).
*   **Si responde pero no monta:** Falla Capa 7 (Permisos de exportación en TrueNAS).
