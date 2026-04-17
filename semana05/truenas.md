
#  GUÍA INTEGRAL DE DESPLIEGUE: TRUENAS SCALE 25.10.3 (TECSUP)

## 1. TOPOLOGÍA Y ARQUITECTURA DEL SISTEMA
Esta es la estructura lógica del entorno que hemos construido:

```text
ENTORNO DE ALMACENAMIENTO VIRTUALIZADO (TECSUP)
===========================================================================
[ HOST FÍSICO: Intel i9-12900K | Windows 10/11 ]
      |
      +--- [ HIPERVISOR: VMware Workstation ]
                |
                +--- [ VM: TrueNAS SCALE 25.10.3 (Debian 12) ]
                      |
                      +-- Red (Bridged): IP 10.160.10.127 /24
                      +-- Acceso Web: http://10.160.10.127
                      +-- Credenciales: truenas_admin | Tecsup00
                      +-- Hardware Virtual: 2 vCPUs | 16GB RAM
                      |
                      +-- [ CONTROLADORA SCSI/SAS VIRTUAL ]
                            |-- sda: 20GB (Sistema Operativo - Boot Pool)
                            |-- sdb a sdi: 8 x 20GB (Datos - ZFS Data Pool)
===========================================================================
```

---

## FASE 1: CREACIÓN DE LA MÁQUINA VIRTUAL (VMWARE)
Antes de instalar, preparamos el "hardware" virtual:

1.  **Nuevo Asistente:** "Create a New Virtual Machine" -> **Custom (advanced)**.
2.  **Compatibilidad:** Hardware compatibility: Workstation 17.x (o la más reciente).
3.  **SO:** "I will install the operating system later". Selecciona **Linux -> Debian 12 (64-bit)**.
4.  **Nombre y Ubicación:** `TrueNAS_Tecsup_Lab`.
5.  **Procesadores:** Mínimo 2 Cores.
6.  **Memoria:** Asigna **16384 MB (16GB)**. (*Crítico para el caché ZFS ARC*).
7.  **Red:** Selecciona **Use bridged networking**.
8.  **Controladora I/O:** LSI Logic SAS.
9.  **Discos Virtuales (El Rack):**
    *   Crea el primer disco de **20GB** (Store virtual disk as a single file).
    *   Una vez creada la VM, ve a *Edit Virtual Machine Settings* y usa el botón **Add -> Hard Disk** para añadir **8 discos adicionales de 20GB cada uno**.
10. **ISO:** En la unidad CD/DVD, carga el archivo `TrueNAS-SCALE-25.10.3.iso`.

---

## FASE 2: INSTALACIÓN DEL SISTEMA (CONSOLA AZUL)
Basado en las capturas de pantalla validadas:

1.  **Menú Inicial:** Selecciona **1 Install/Upgrade**.
2.  **Selección de Disco:** Marca con la barra espaciadora **únicamente el disco `sda`** (20 GiB). Los demás (`sdb`-`sdi`) déjalos libres.
3.  **Advertencia:** Selecciona **`< Yes >`** para formatear `sda`.
4.  **Autenticación:** Selecciona **1 Administrative user (truenas_admin)**.
5.  **Contraseña:** Escribe **`Tecsup00`** en ambos campos.
6.  **Modo de Arranque:** Selecciona **UEFI** (o "Allow EFI boot" -> **`< Yes >`**).
7.  **Finalización:** Al terminar, presiona Enter en "OK", selecciona **3 Reboot System** y **desconecta la ISO** en VMware.

---

## FASE 3: CONFIGURACIÓN DE RED (CONSOLA NEGRA)
Al iniciar por primera vez desde el disco duro:

1.  Selecciona la **Opción 1: Configure network interfaces**.
2.  Elige la interfaz detectada (ej. `enp0s1`).
3.  ¿Borrar configuración? **No**. ¿DHCP? **No**. ¿IPv4? **Yes**.
4.  **IPv4 Address:** Escribe **`10.160.10.127/24`**.
5.  **Default Gateway:** La IP de tu router (ej. `10.160.10.1`).
6.  Al finalizar, la consola mostrará: `The web user interface is at: http://10.160.10.127`.

---

## FASE 4: LABORATORIO EXPERIMENTAL DE RAID (8 DISCOS)
Una vez dentro de la Web GUI (`http://10.160.10.127`), ve a **Storage -> Create Pool**. Aquí es donde practicaremos los niveles de redundancia profesional usando tus 8 discos:

### Escenario A: RAID 10 Pro (Stripe of Mirrors) - *Máximo Rendimiento*
*   **Cómo hacerlo:** Crea un VDEV tipo **Mirror** con 2 discos. Añade otro VDEV tipo Mirror con otros 2. Repite hasta usar los 8.
*   **Resultado:** ZFS hará un "Stripe" entre los 4 espejos.
*   **Comentario de Sysadmin:** Es la configuración más rápida en IOPS. Ideal para bases de datos pesadas e iSCSI. Pierdes el 50% de capacidad pero ganas velocidad extrema.

### Escenario B: RAIDZ1 (Equivalente a RAID 5) - *Máxima Capacidad*
*   **Cómo hacerlo:** Selecciona los 8 discos y elige Layout **RAIDZ1**.
*   **Resultado:** Tienes 140GB útiles (7 discos para datos, 1 para paridad).
*   **Comentario:** Solo tolera el fallo de **1 disco**. En un entorno de 8 discos, es arriesgado porque si falla uno, el estrés de reconstruir los otros 7 puede romper un segundo disco y pierdes todo.

### Escenario C: RAIDZ2 (Equivalente a RAID 6) - *Estándar de Oro*
*   **Cómo hacerlo:** Selecciona los 8 discos y elige Layout **RAIDZ2**.
*   **Resultado:** Tienes 120GB útiles (6 para datos, 2 para paridad).
*   **Comentario:** Es la configuración más equilibrada. Puedes perder **2 discos cualesquiera** simultáneamente y tu servidor seguirá online sin perder datos. Es lo que yo instalaría en una empresa real.

### Escenario D: RAIDZ3 (Triple Paridad) - *Máxima Seguridad*
*   **Cómo hacerlo:** Selecciona los 8 discos y elige Layout **RAIDZ3**.
*   **Resultado:** Tienes 100GB útiles (5 para datos, 3 para paridad).
*   **Comentario:** Tolera el fallo de **3 discos**. Se usa en sistemas masivos (JBODs de 60 discos) donde la probabilidad de fallos múltiples es alta.
  
---
## FASE 5: PUBLICACIÓN Y ACCESO FINAL
1.  **Crear Dataset:** Después de elegir tu RAID favorito, crea un Dataset llamado `Proyectos_Tecsup`.
2.  **Publicar SMB:** Ve a **Shares -> Windows Shares**, elige la ruta del dataset y dale a *Save*.
3.  **Acceso desde Host:** Desde Windows presiona `Win+R`, escribe `\\10.160.10.127`.
4.  **Login Web/SMB:** 
    *   **User:** `truenas_admin`
    *   **Pass:** `Tecsup00`



## NOTAS FINALES DEL ESPECIALISTA
*   **Gestión de Memoria:** Verás en el Dashboard que TrueNAS usa gran parte de tus 16GB de RAM. No te asustes, es el **ZFS ARC** trabajando para que el acceso a tus datos sea instantáneo.
*   **Protección:** Configura **Snapshots** automáticos. Si borras algo en tu Windows que está conectado al NAS, podrás recuperarlo en segundos desde la pestaña *Data Protection*.
*   **Apagado:** Para evitar errores en el Pool, siempre usa la opción **10) Shutdown** en la consola o el botón de Power en la Web GUI antes de cerrar VMware.



