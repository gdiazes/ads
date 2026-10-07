
# GUÍA DE EVALUACIÓN: CLÚSTER HA DISTRIBUIDO (TrueNAS + Pacemaker)

**Objetivo:** Diseñar, desplegar y auditar una infraestructura de Alta Disponibilidad (HA) distribuida en múltiples equipos físicos, utilizando almacenamiento centralizado y un clúster activo-pasivo, basándose en la metodología **PPDIOO** (Prepare, Plan, Design, Implement, Operate, Optimize).

**Tiempo de Ejecución:** 100 Minutos.
**Modalidad:** Trabajo de 4 estudiantes distribuidos en 4 PCs físicas.



## EL CASO: "Fintech GlobalSys"
La pasarela de pagos "GlobalSys" no puede permitirse un solo minuto de caída en su plataforma. Te han contratado como equipo de Arquitectos e Ingenieros de Infraestructura. El reto es reemplazar su antiguo servidor aislado por un **Clúster Activo-Pasivo de 2 nodos Ubuntu Server con almacenamiento centralizado en TrueNAS**. 

Para garantizar tolerancia a fallos de hardware total, **las máquinas virtuales se desplegarán en computadoras físicas diferentes** conectadas a la red corporativa (la red del laboratorio).



## PARÁMETROS DE RED: IPAM Y SEGREGACIÓN LÓGICA (Fase: Prepare & Plan)
Todos los grupos compartirán el mismo switch físico y la red base del laboratorio: **192.168.17.0/24 (GW: 192.168.17.1)**. 
Para evitar colapsos de red (split-brain entre grupos) y choques de IP Flotante (VIP), el Arquitecto de Red debe respetar estrictamente el bloque asignado a su equipo.

| Equipo | Rango de IPs Libres Asignado | IP Flotante (VIP) | Nombre Único del Clúster |
| :--- | :--- | :--- | :--- |
| **Grupo 1** | 192.168.17.111 al .119 | 192.168.17.110 | cluster-alfa |
| **Grupo 2** | 192.168.17.121 al .129 | 192.168.17.120 | cluster-beta |
| **Grupo 3** | 192.168.17.131 al .139 | 192.168.17.130 | cluster-gamma |
| **Grupo 4** | 192.168.17.141 al .149 | 192.168.17.140 | cluster-delta |

> **TRAMPAS TÉCNICAS DE INFRAESTRUCTURA:**
> 1. **VMware Bridged:** El adaptador de red en VMware no debe estar en NAT. Debe estar en Bridged y apuntar directamente a la tarjeta física de la PC.
> 2. **Sincronización de Tiempo (NTP):** Corosync fallará si las PCs físicas tienen horarios distintos. Obligatorio verificar sincronización NTP.
> 3. **Apache Autostart:** Prohibido que Apache inicie solo. Obligatorio ejecutar `sudo systemctl disable --now apache2` antes de crear el clúster.



## ROLES Y DISTRIBUCIÓN FÍSICA (Fase: Design)
El grupo de 4 estudiantes utilizará 4 PCs físicas en el laboratorio de la siguiente manera:

*   **PC Física 1 - Especialista en Integración HA (L4/L7)**
    *   **Rol:** Aloja y configura la VM del Nodo 1 (Ubuntu). Lidera la ejecución de comandos `pcs cluster setup` (usando el nombre de clúster asignado), autenticación y recursos lógicos (VIP y WebServer).
*   **PC Física 2 - Arquitecto de Red y Seguridad (L2/L3)**
    *   **Rol:** Aloja y configura la VM del Nodo 2 (Ubuntu). Lidera el diseño en pizarra. Asegura que las IPs estáticas y el Firewall (UFW) estén correctos para permitir el tráfico de Corosync entre las PCs físicas.
*   **PC Física 3 - Ingeniero de Almacenamiento (L7)**
    *   **Rol:** Aloja y configura la VM del Nodo 3 (TrueNAS). Configura el Pool ZFS, Dataset y levanta el servicio NFS con permisos estrictos (`Mapall User: root`) para evitar bloqueos del clúster.
*   **PC Física 4 - SRE / Troubleshooter (Operate & Optimize)**
    *   **Rol:** No aloja servidores. Su PC actúa como Cliente y Auditoría. Ejecuta los pings a la VIP, visualiza el portal web y monitorea remotamente los logs (`pcs status`). Lidera la aplicación de restricciones lógicas (Colocalización y Orden).

---

## LÍNEA DE TIEMPO (100 Minutos)
*   **00 - 15 min:** Diseño en Pizarra. El Arquitecto sustenta la topología, IPs y justificación del modelo OSI al docente.
*   **15 - 65 min:** Implementación en Paralelo. Instalación de VMs en PCs físicas, configuración de SO, IPs, NTP y NFS.
*   **65 - 85 min:** Ensamblaje del Clúster. Unión de los nodos físicos, configuración de Pacemaker y restricciones.
*   **85 - 100 min:** Auditoría Docente y Pruebas de Estrés.



## RÚBRICA DE EVALUACIÓN DUAL (ESCALA 0 - 20 PUNTOS)
La nota final del alumno es la suma de la Parte A (hasta 8 pts) + Parte B (hasta 12 pts).

### PARTE A: Evaluación GRUPAL (Hasta 8 Puntos) - Producto Final
Aplica a todo el equipo si la infraestructura sobrevive a las pruebas conjuntas.

| Criterio | Excelente (2 ptos) | Regular (1 pto) | Deficiente (0 ptos) |
| :--- | :--- | :--- | :--- |
| **Diseño y Normativa IPAM** | Pizarra OSI impecable. Respetaron el bloque IPAM y el Bridged. | Dudas en diseño o IP fuera de rango corregida a tiempo. | Colisión de IPs/Clúster con otro grupo. (Fallo grave). |
| **Integración de Capas L7** | El Storage NFS monta automáticamente en el clúster (Capa 7 sana). | Requiere comandos manuales para montar el NFS temporalmente. | NFS con acceso denegado o no monta en los nodos. |
| **Failover Lógico (Chaos Lvl 1)** | Al ejecutar `killall apache2` en el Nodo Activo, Pacemaker lo revive automáticamente. | Demora más de 1 minuto en revivir o se requiere reinicio. | El servicio queda caído sin respuesta del clúster. |
| **Failover Físico (Chaos Lvl 2)** | Al apagar bruscamente la PC Física del Nodo Activo, la web carga en la otra PC en < 30 seg. | La VIP migra, pero hay errores al cargar el NFS o Apache. | El sistema colapsa (Split-Brain) o no hay recuperación. |

### PARTE B: Evaluación INDIVIDUAL (Hasta 12 Puntos) - Defensa Técnica
El docente realizará una prueba y una pregunta de sustentación de alto nivel a cada especialista.

| Rol | Excelente (12 - 10 ptos) | Suficiente (9 - 6 ptos) | Insuficiente (5 - 0 ptos) |
| :--- | :--- | :--- | :--- |
| **Arquitecto de Red (PC 2)** | Dominio total. Explica cómo la VIP y el protocolo GARP actualizan las tablas MAC de los switches físicos del laboratorio. | Logra conectividad base pero duda al explicar cómo fluye el tráfico de la IP Flotante a nivel L2. | Error crítico en Firewalls o IP duplicada. Desconoce funcionamiento del modo Bridged. |
| **Ingeniero Storage (PC 3)** | TrueNAS operando perfecto. Explica sólidamente la relación entre `Mapall root`, ZFS y la prevención de corrupción. | El disco funciona, pero no sabe qué hace el parámetro `Mapall` internamente en Linux. | Falla de NFS L7, permisos denegados, no exportó la ruta correctamente. |
| **Integrador HA (PC 1)** | Dominio de comandos `pcs`. Deshabilitó `systemd` y explica con claridad el riesgo de no tener sincronización NTP. | Clúster levantado, pero no puede diagnosticar si el estado muestra "Offline". | Corosync falla, nodos no se ven, olvidó apagar systemd en Apache. |
| **SRE (PC 4)** | Lee logs con fluidez. Explica el concepto de INFINITY en colocalización y por qué el Storage debe ir *before* Apache. | Entiende el concepto pero depende de ayuda para leer el `pcs status` durante una crisis. | Las restricciones no fueron aplicadas, Apache arranca sin disco montado. |


