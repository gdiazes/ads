# Guía: Clúster de VMs de Alta Disponibilidad con HAProxy y Keepalived en VMware

Este laboratorio te guiará en la construcción de una infraestructura de servidores web altamente disponible y con balanceo de carga, utilizando únicamente máquinas virtuales. Aprenderás a configurar un par de balanceadores de carga en modo Activo/Pasivo que distribuirán el tráfico a un grupo de servidores web.

**Objetivo:** Implementar un clúster donde la falla de un servidor web o incluso de un balanceador de carga no interrumpa el servicio.

**Tecnologías:**
*   **Hipervisor:** VMware Workstation 17 Pro
*   **Sistema Operativo:** Ubuntu Server 22.04 LTS
*   **Balanceo de Carga:** HAProxy (Software Load Balancer)
*   **Alta Disponibilidad:** Keepalived (para gestionar una IP Virtual flotante)
*   **Carga de Trabajo:** Servidores Web Apache2 (en VMs)

---

## 1. Topología y Plan de Red

Esta arquitectura es fundamental para el éxito del laboratorio. Tendremos dos tipos de máquinas: Balanceadores de Carga y Servidores Web. Todas se comunicarán a través de una red privada.

### Diagrama de la Topología

```
          +--------------------------------------+
          |        Tu PC (Host Físico)           |
          +--------------------------------------+
                         |
           Red NAT (Acceso a Internet para todas las VMs)
                         |
+------------------------+------------------------------------------+
|                        |                                          |
| eth0                   | eth0                                     | eth0
|     +------------------+-----------------+      +-----------------+-----+
+---->|            lb-01 (Activo)         |<---->|           web-01      |
      |                                    |      |                     |
      |      (HAProxy + Keepalived)        |      |      (Apache2)      |
      | eth1                               |      | eth1                |
      +--+---------------------------------+      +----------+----------+
         |                                                    |
         |         LAN Segment: "cluster-net"                 |
         |        (Red Privada: 192.168.100.0/24)             |
         |                                                    |
      +--+---------------------------------+      +----------+----------+
      | eth1                               |      | eth1                |
+---->|            lb-02 (Pasivo)         |<---->|           web-02      |
      |                                    |      |                     |
      |      (HAProxy + Keepalived)        |      |      (Apache2)      |
      +------------------------------------+      +---------------------+
                         |
            IP VIRTUAL (VIP) FLOTANTE
                  192.168.100.50
     (Este es el punto de acceso al servicio)
```

### Tabla de Direccionamiento IP (Red Privada `cluster-net`)

| Hostname | Rol                       | Dirección IP Estática |
| :------- | :------------------------ | :-------------------- |
| `lb-01`  | Balanceador (Master/Activo) | `192.168.100.20`      |
| `lb-02`  | Balanceador (Backup/Pasivo) | `192.168.100.21`      |
| `web-01` | Servidor Web 1            | `192.168.100.30`      |
| `web-02` | Servidor Web 2            | `192.168.100.31`      |
| **N/A**  | **IP Virtual (VIP)**      | **`192.168.100.50`**  |

---

## 2. Creación de la Plantilla y Clonación de VMs

Para agilizar el proceso, crearemos una única plantilla de Ubuntu Server 22.04 y la clonaremos 4 veces.

### Paso 2.1: Crear la VM Plantilla
Sigue los mismos pasos que en la guía anterior para crear una VM `ubuntu-template` con **dos adaptadores de red**:
*   **Adaptador 1:** `NAT` (para `eth0`)
*   **Adaptador 2:** `LAN Segments` > `cluster-net` (para `eth1`)

Realiza la instalación base de Ubuntu Server.

### Paso 2.2: Configurar la Plantilla
Una vez instalada, enciende la VM `ubuntu-template` y realiza la configuración base:
```bash
# Actualizar paquetes e instalar herramientas
sudo apt update && sudo apt install -y curl vim net-tools

# Deshabilitar permanentemente el swap
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Configurar Netplan con una IP temporal para la plantilla
sudo nano /etc/netplan/00-installer-config.yaml```
Usa esta configuración en `netplan`:
```yaml
network:
  ethernets:
    eth0:
      dhcp4: true
    eth1:
      dhcp4: no
      addresses:
        - 192.168.100.1/24 # IP temporal, la cambiaremos en cada clon
  version: 2
```
Aplica la configuración (`sudo netplan apply`) y apaga la plantilla (`sudo shutdown now`).

### Paso 2.3: Clonar y Configurar las 4 VMs
Clona la plantilla `ubuntu-template` 4 veces para crear: `lb-01`, `lb-02`, `web-01` y `web-02`.

Luego, enciende cada una y **configura su hostname y su IP estática** según la tabla del Paso 1. Por ejemplo, para `web-01`:
1.  `sudo hostnamectl set-hostname web-01`
2.  `sudo nano /etc/netplan/00-installer-config.yaml` y cambia la IP a `192.168.100.30/24`.
3.  `sudo netplan apply` y `sudo reboot`.

**Repite este proceso para las 4 VMs hasta que cada una tenga el hostname y la IP correcta.**

---

## 3. Configuración de los Servidores Web (Backend)

En las VMs `web-01` y `web-02`, vamos a instalar un servidor web y crear una página de inicio única para poder identificar qué servidor responde.

**En `web-01` y `web-02`, ejecuta:**
```bash
# Instalar el servidor web Apache
sudo apt update
sudo apt install -y apache2
```

**Ahora, crea la página de prueba:**

**En `web-01`:**
```bash
echo "<h1>Servidor Web 01 - IP: 192.168.100.30</h1>" | sudo tee /var/www/html/index.html
```

**En `web-02`:**```bash
echo "<h1>Servidor Web 02 - IP: 192.168.100.31</h1>" | sudo tee /var/www/html/index.html
```

---

## 4. Configuración de los Balanceadores de Carga

Esta es la parte más importante. Configuraremos HAProxy y Keepalived en `lb-01` y `lb-02`.

### Paso 4.1: Instalar Software
**En `lb-01` y `lb-02`, ejecuta:**
```bash
sudo apt update
sudo apt install -y haproxy keepalived
```

### Paso 4.2: Configurar HAProxy
HAProxy distribuirá el tráfico. La configuración será **idéntica en ambos balanceadores**.

Edita el archivo de configuración:
```bash
sudo nano /etc/haproxy/haproxy.cfg
```
Reemplaza todo el contenido del archivo con lo siguiente:
```cfg
global
    log /dev/log    local0
    log /dev/log    local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000

frontend http_frontend
    # Escucha en el puerto 80 en TODAS las interfaces
    bind *:80
    mode http
    # Usa el backend definido abajo
    default_backend web_servers

backend web_servers
    mode http
    # Estrategia de balanceo: roundrobin (una petición a cada servidor, en ciclo)
    balance roundrobin
    # Define los servidores backend. 'check' habilita el monitoreo de salud.
    server web-01 192.168.100.30:80 check
    server web-02 192.168.100.31:80 check
```
**En ambos balanceadores (`lb-01` y `lb-02`), reinicia HAProxy:**
```bash
sudo systemctl restart haproxy
sudo systemctl enable haproxy
```

### Paso 4.3: Configurar Keepalived
Keepalived gestionará la IP Virtual (`192.168.100.50`). Si `lb-01` cae, `lb-02` tomará control de esa IP automáticamente. La configuración es **ligeramente diferente** para cada nodo.

**En `lb-01` (Master):**
```bash
sudo nano /etc/keepalived/keepalived.conf
```
Pega la siguiente configuración:
```
vrrp_instance VI_1 {
    state MASTER            # Este nodo es el maestro por defecto
    interface eth1          # La interfaz de nuestra red privada
    virtual_router_id 51    # Debe ser el mismo en ambos nodos
    priority 101            # La prioridad más alta gana
    advert_int 1
    
    authentication {
        auth_type PASS
        auth_pass 1111
    }

    virtual_ipaddress {
        192.168.100.50      # La IP Virtual Flotante
    }
}
```

**En `lb-02` (Backup):**
```bash
sudo nano /etc/keepalived/keepalived.conf
```
Pega esta configuración (nota los cambios en `state` y `priority`):
```
vrrp_instance VI_1 {
    state BACKUP            # Este nodo es el de respaldo
    interface eth1
    virtual_router_id 51
    priority 100            # Menor prioridad, solo toma el control si el master falla
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass 1111
    }

    virtual_ipaddress {
        192.168.100.50
    }
}
```

**En ambos balanceadores (`lb-01` y `lb-02`), reinicia Keepalived:**
```bash
sudo systemctl restart keepalived
sudo systemctl enable keepalived
```

---

## 5. Verificación y Pruebas Finales

### Paso 5.1: Probar el Balanceo de Carga
Conéctate por SSH a `lb-01` o `lb-02`. Verifica que la IP virtual está activa en el nodo maestro (`lb-01`):
```bash
# En lb-01 deberías ver la IP 192.168.100.50 en la interfaz eth1
ip a
```
Ahora, usa `curl` en un bucle para hacer peticiones a la IP virtual. Verás cómo la respuesta alterna entre `web-01` y `web-02`.
```bash
watch -n 1 curl http://192.168.100.50```
**Salida esperada (alternando cada segundo):**
```
<h1>Servidor Web 01 - IP: 192.168.100.30</h1>
``````
<h1>Servidor Web 02 - IP: 192.168.100.31</h1>
```
¡El balanceo de carga funciona!

### Paso 5.2: Probar la Alta Disponibilidad (Failover)
Mantén el comando `watch` ejecutándose en una terminal.
Ahora, vamos a simular una falla del balanceador de carga principal.

1.  En la interfaz de VMware Workstation, haz clic derecho en la VM `lb-01` y selecciona `Power > Shut Down Guest`.
2.  **Observa la terminal donde tienes el `watch`**. El servicio podría interrumpirse por 1 o 2 segundos, pero inmediatamente después, `lb-02` tomará el control de la IP virtual y el servicio se reanudará sin que tengas que hacer nada.
3.  Para confirmar, conéctate por SSH a `lb-02` y ejecuta `ip a`. ¡Ahora verás que `lb-02` tiene asignada la IP `192.168.100.50`!

¡La alta disponibilidad funciona!

---

## 6. Limpieza del Entorno
Cuando hayas terminado, simplemente apaga y elimina las cuatro máquinas virtuales desde la interfaz de VMware para liberar los recursos de tu PC.
