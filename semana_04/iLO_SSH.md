
### **Guía de Administración y Monitoreo de Servidores HPE Gen10 vía SSH a iLO 5**

#### **Introducción**

Esta guía integral proporciona los procedimientos paso a paso para realizar tareas avanzadas de administración y monitoreo en servidores HPE ProLiant Gen10. Todos los comandos se ejecutan en el shell **SMASH CLP** de iLO 5, accesible a través de una conexión SSH. Cubriremos la gestión de usuarios, la revisión de registros de eventos (logs), y el monitoreo de componentes críticos de hardware como temperatura, procesador, almacenamiento y fuentes de alimentación.

#### **Requisitos Previos**

*   **Dirección IP de iLO 5** del servidor.
*   **Credenciales de iLO** con los permisos adecuados (se recomienda un rol de `admin` para todas las tareas).
*   **Cliente SSH** (PuTTY, Terminal, etc.).
*   **Conectividad de Red** al puerto 22 de la IP de iLO.

---

### **Parte 1: Gestión de Usuarios**

La gestión de usuarios es fundamental para la seguridad. Aquí se muestra cómo listar, crear y eliminar usuarios.

**1. Conectarse y Navegar al Contexto de Cuentas**

```bash
ssh usuario_admin@direccion_ip_ilo
</>hpiLO-> cd /map1/accounts1
```
*   **Explicación:** Inicia una sesión SSH y se mueve al "directorio" donde se gestionan las cuentas de usuario.

**2. Listar Todos los Usuarios Creados**

```
</map1/accounts1>hpiLO-> show
```
*   **Explicación:** Muestra una lista de todos los nombres de usuario configurados en iLO. Esencial para auditar quién tiene acceso.

**3. Crear un Nuevo Usuario**

```
</map1/accounts1>hpiLO-> create username=auditor password=SomeSecurePassword! group=login
```
*   **Explicación:** Crea un nuevo usuario llamado `auditor`. El parámetro `group=login` le otorga únicamente permisos para iniciar sesión, ideal para roles de solo lectura.

**4. Eliminar un Usuario**

```
</map1/accounts1>hpiLO-> delete auditor
Are you sure you want to delete /map1/accounts1/auditor? (y/n) y
```
*   **Explicación:** Elimina permanentemente el usuario `auditor` del sistema después de una confirmación.

---

### **Parte 2: Revisión de Logs de Eventos**

Revisar los logs es crucial para diagnosticar problemas de hardware o eventos de seguridad. iLO mantiene dos logs principales: el IML (del sistema) y el log de eventos del propio iLO.

**1. Ver el Log de Gestión Integrado (IML - Integrated Management Log)**

```
</>hpiLO-> show /system1/log1/entries
```
*   **Explicación:** Muestra el contenido del IML, que registra todos los eventos de hardware significativos del servidor (fallos de memoria, errores de disco, problemas de ventilador, etc.). Es el primer lugar donde buscar al diagnosticar un problema físico.

**2. Ver el Log de Eventos de iLO**

```
</>hpiLO-> show /map1/log1/entries
```
*   **Explicación:** Muestra el log de eventos específico de la interfaz iLO. Registra actividades como inicios de sesión (exitosos y fallidos), cambios de configuración, actualizaciones de firmware de iLO, etc. Es vital para auditorías de seguridad.

**3. Limpiar los Logs (¡Usar con precaución!)**

```
</>hpiLO-> set /system1/log1 clear=yes
</>hpiLO-> set /map1/log1 clear=yes
```
*   **Explicación:** Estos comandos borran el IML y el log de iLO, respectivamente. Generalmente, solo se usan después de que un problema ha sido resuelto y documentado.

---

### **Parte 3: Monitoreo de Hardware Crítico**

El shell de iLO ofrece una visibilidad completa del estado del hardware en tiempo real.

**1. Revisar Temperaturas del Sistema**

```
</>hpiLO-> show /system1/sensor*
```
*   **Explicación:** Este comando utiliza un comodín (`*`) para mostrar el estado de **todos los sensores** del sistema. La salida incluirá sensores de temperatura (`SensorType=Temperature`) para componentes como CPU, memoria, chipset y zonas del chasis. Busca las propiedades `CurrentReading` (lectura actual) y `HealthState` (estado de salud).

**2. Revisar el Uso y Estado del Procesador (CPU)**

```
</>hpiLO-> show /system1/cpu*
```
*   **Explicación:** Muestra información detallada sobre cada CPU instalada, incluyendo el `name` (modelo), `speed` (velocidad), `threads` (hilos) y, lo más importante, el `status` (estado de salud). **Nota:** iLO no muestra el porcentaje de uso de la CPU en tiempo real; esa es una métrica del sistema operativo. iLO se centra en el estado físico y operativo del hardware.

**3. Revisar el Estado de los Discos y Almacenamiento**

```
</>hpiLO-> show /system1/storage1
```
*   **Explicación:** Proporciona un resumen completo del subsistema de almacenamiento. La salida está anidada y muestra:
    *   **Controladoras:** Modelo y estado.
    *   **Unidades Lógicas (Arrays):** Estado, tipo de RAID y capacidad.
    *   **Unidades Físicas (Discos):** Estado de cada disco individual. Es la forma más rápida de identificar un disco fallido en un array RAID.

**4. Revisar Valores de las Fuentes de Energía (Power Supplies)**

**A) Estado General de las Fuentes:**
```
</>hpiLO-> show /system1/powersupply*
```
*   **Explicación:** Muestra el estado de cada fuente de alimentación instalada. Busca `HealthState` para ver si está `OK` y `OperationalStatus` para ver si está funcionando.

**B) Lecturas de Consumo de Energía:**
```
</>hpiLO-> show /system1/oemhp_power1
```
*   **Explicación:** Este comando muestra métricas detalladas del consumo energético del servidor. Las propiedades más útiles son:
    *   `oemHPE_PresentPower`: Consumo actual en vatios (lectura casi en tiempo real).
    *   `oemHPE_AvgPower`: Consumo promedio en las últimas 24 horas.
    *   `oemHPE_MaxPower`: Pico máximo de consumo en las últimas 24 horas.

---
### **Finalizar la Sesión**

Cuando hayas completado tus tareas de monitoreo y administración, siempre cierra la sesión.

```
</>hpiLO-> exit
