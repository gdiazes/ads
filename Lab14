#   DOCUMENTO 1: GUÍA DEL ESTUDIANTE (Para imprimir / entregar)

```text
========================================================================================
TECSUP - ADMINISTRACIÓN DE REDES Y COMUNICACIONES
CURSO: ARQUITECTURA DE SERVIDORES
CASO TÉCNICO: AUDITORÍA Y REDISEÑO DEL SISTEMA DE PUESTA A TIERRA (GROUNDING & BONDING)
========================================================================================
Integrantes del Equipo:
1. __________________________________   2. __________________________________
3. __________________________________   4. __________________________________
Fecha: ____________   Sección: ______
========================================================================================
```

### 1. EL CASO: "Descargas y Corrientes Fantasma en DataCloud Perú"
El Data Center corporativo de la empresa **DataCloud Perú** aloja servicios bancarios y plataformas de facturación electrónica. Tras una reciente tormenta eléctrica y el encendido de un grupo electrógeno en el edificio contiguo, ocurrieron tres fallas críticas simultáneas:
1. Se quemaron las tarjetas de red (NICs) y puertos SFP+ de 8 servidores en distintos racks.
2. Los técnicos sintieron descargas electrostáticas (toques de corriente) al tocar las puertas metálicas de los gabinetes.
3. El analizador de redes reportó un voltaje neutro-tierra de **8.5 V AC** en los racks (el límite normativo es $< 1\text{ V}$).

La empresa auditora detectó que el contratista civil instaló un sistema de aterramiento improvisado sin cumplir las normas internacionales ni las pautas de Huawei/TIA. Su equipo de especialistas debe auditar el informe de la instalación actual, corregir las fallas técnicas y rediseñar el sistema de tierra exterior e interior.

---

### 2. INFORME TÉCNICO DE LA INSTALACIÓN ACTUAL (HALLAZGOS)

> **A. Pozo y Anillo de Tierra Exterior:**
> * Se instaló una sola varilla de cobre de 1.5 m clavada en el jardín frontal a 20 cm de profundidad, unida con cable calibre 10 AWG.
> * El contratista no unió el sistema de tierra a las columnas estructurales de acero del edificio para "no electrificar la construcción".
> * La resistencia de puesta a tierra medida fue de **$18.5\,\Omega$**.
>
> **B. Régimen Eléctrico y Barras Principales:**
> * El sistema de distribución se configuró como **TN-C** (el conductor neutro y la tierra de protección están combinados en un solo cable PEN que entra a la sala de servidores).
> * No existe barra principal de tierra de telecomunicaciones (TMGB) ni barra principal eléctrica (MGB); todo se conectó a una cañería de agua potable de PVC.
>
> **C. Sala de Servidores (Piso Técnico y Racks):**
> * No existe malla de tierra bajo el piso técnico (Mesh-BN). El instalador afirmó que: *"con la tercera espiga de los cables de poder de las fuentes de los servidores es suficiente según código"*.
> * Los pedestales del piso elevado están apoyados sobre el concreto sin ninguna conexión eléctrica entre sí.
> * Los racks metálicos están pintados y las puertas no tienen trenzas de masa flexibles que las unan al bastidor.

---

### 3. RETO DE TRABAJO (COMPLETAR EN EQUIPO)

#### PARTE I: Auditoría de No Conformidades y Corrección Normativa
Con el apoyo de sus diapositivas (Slides 99 a 118 de clase y lineamientos Huawei PPT 03/11), completen la ficha técnica:

| Punto Evaluado | Error Técnico Detectado | Corrección Exigida (Valores Normativos Exactos) | Consecuencia Técnica / Riesgo de no corregir |
| :--- | :--- | :--- | :--- |
| **Anillo de Tierra Exterior** (Slide 102 / Huawei PPT 11) | Cable 10 AWG a 20 cm; 1 varilla de 1.5 m. | • Calibre mín. conductor: **____ mm² (____ AWG)**<br>• Profundidad de enterramiento: **______ m**<br>• Distancia a la pared: **______ m**<br>• Dimensiones varillas: **____ mm x ____ m**<br>• Espaciamiento entre varillas: cada **____ a ____ m** | |
| **Unión al Acero Estructural** (Slide 102) | No se unió a las columnas de acero del edificio. | • Requisito de unión al acero: ______________________ | |
| **Resistencia de Puesta a Tierra** (Huawei PPT 11) | Medición actual: $18.5\,\Omega$. | • Valor máx. de resistencia objetivo: **$\le$ ______ $\Omega$** | |
| **Régimen de Puesta a Tierra** (Huawei PPT 03) | Configuración TN-C (conductor PEN combinado). | • Esquema normativo para Data Center: **Sistema ______**<br>• Regla del Neutro y Tierra (PE): ___________________ | |
| **Malla de Sala (Mesh-BN)** (Slides 108 a 110) | No existe malla bajo el piso; sólo tercera espiga AC. | • Dimensión de pletina de cobre: **____ mm x ____ mm**<br>• Retícula de espaciado de malla: **______ mm**<br>• Si se usa cable redondo desnudo: **____ mm² (____ AWG)**<br>• ¿El cable AC cumple NECA/BICSI 607?: **[ SÍ ]  [ NO ]** | |
| **Unión a Pedestales de Piso** (Slide 110) | Pedestales aislados sin continuidad eléctrica. | • Si es pletina, unir cada: **____° pedestal**<br>• Si es cable redondo, unir cada: **____° pedestal**<br>• Distancia máx. unión pedestal-malla: **______ m** | |
| **Aterramiento de Racks y Equipos** (Slides 113 a 117) | Racks pintados sin unión directa; puertas sueltas. | • Calibre cable rack a malla: **____ mm² (____ AWG)**<br>• Requisito de contacto: ____________________________<br>• Protección de puertas y paneles: ___________________ | |

---

#### PARTE II: Diagrama Esquemático de Conexión Equipotencial
En el siguiente espacio, dibuje el esquema de conexión que una los elementos de la sala de cómputo hacia el exterior (siguiendo el modelo de las Slides 100, 101 y 106):
1. **Electrodo/Anillo exterior** $\to$ **MGB** (Building Main Ground Busbar).
2. **TMGB** (Telecommunications Main Ground Busbar) $\to$ **TGB**.
3. **Malla Mesh-BN** bajo el piso técnico interconectada con los pedestales.
4. **Gabinete de Servidores** (barra interna del rack conectada a la malla Mesh-BN con cable de 16 mm²).

```text
+---------------------------------------------------------------------------------------+
|                       DIAGRAMA ESQUEMÁTICO DEL SISTEMA DE TIERRA                     |
|                                                                                       |
|                                                                                       |
|                                                                                       |
|                                                                                       |
|                                                                                       |
|                                                                                       |
|                                                                                       |
|                                                                                       |
+---------------------------------------------------------------------------------------+
```

---
```text
