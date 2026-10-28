# GUÍA DE PRÁCTICA DE LABORATORIO
**CURSO:** Infraestructura de Centros de Datos / HCIA-Data Center Facility v2.0  
**TEMA:** Ingeniería Inversa y Auditoría *As-Built* de un Centro de Datos Modular (Huawei FusionModule800)  
**DURACIÓN:** 100 minutos  
**HERRAMIENTA:** Draw.io (*diagrams.net*) 


> ### 📌 Nota para el Docente 
> Antes de entregar el plano a los estudiantes, coloque rectángulos blancos sobre el archivo de imagen para ocultar:
> 1. El ancho del gabinete central (**$1.35\text{ m}$**) y el ancho total de la sala (**$4.35\text{ m}$**).
> 2. Las cotas superior e inferior de holgura (**$1.00\text{ m}$** y **$1.00\text{ m}$**).
> 3. El texto **"FACILITIES"** en el Rack 3.
> 4. Los valores de potencia **"....kW"** de los racks de TI.


## 1. OBJETIVOS DE APRENDIZAJE
Al finalizar la sesión, el estudiante será capaz de:
1. **Deducir cotas y dimensiones arquitectónicas faltantes** aplicando proporciones espaciales y estándares constructivos de Huawei.
2. **Identificar la función técnica de cada gabinete** mediante el análisis del trazado de tuberías frigoríficas y cableado.
3. **Auditar el cumplimiento de distancias mínimas de servicio (*clearances*)** según la normativa de Huawei y TIA-942.
4. **Reconstruir y digitalizar el plano arquitectónico en planta (2D)** a escala en Draw.io con cotas normalizadas y sentido del flujo térmico.


## 2. MATERIALES Y RECURSOS
* Plano parcial/incompleto de Climatización del Data Center Estudiantil (entregado por el docente).
* Computadora con navegador web y acceso a [Draw.io](https://app.diagrams.net/).
* Calculadora científica o plantilla Excel.
* Resumen de especificaciones del sistema *Huawei FusionModule800*.


## 3. EL RETO TÉCNICO (MISIÓN DE AUDITORÍA FORENSE)

La institución requiere actualizar la documentación oficial de su **Data Center Estudiantil**, pero el archivo CAD original se encuentra parcialmente dañado: se han perdido las cotas de profundidad del módulo, las holguras en los extremos de la hilera, la etiqueta del gabinete central y el valor de potencia nominal por rack.

Como **Ingeniero Especialista en Infraestructura**, tu labor es **rescatar y validar la información técnica** completando tres fases de trabajo en un tiempo límite de **100 minutos**.
```
[00:00 - 00:25] FASE 1: Deducción Matemática y Análisis de Ingeniería (25 min)
[00:25 - 00:75] FASE 2: Digitalización del Plano As-Built en Draw.io (50 min)
[00:75 - 00:95] FASE 3: Auditoría Normativa y Preguntas de Control (20 min)
[00:95 - 01:40] FASE 4: Cierre y Carga de Entregables (5 min)
```


## 4. PROCEDIMIENTO PASO A PASO

### FASE 1: Deducción Matemática y Análisis de Ingeniería (25 min)
Antes de ingresar a Draw.io, resuelve los siguientes cálculos y justificaciones en tu memoria técnica:

#### 1.1 Deducción de las Dimensiones de la Sala
Observa las cotas visibles en el plano:
* Pasillo posterior (distancia de mantenimiento): $....\text{ m}$.
* Pasillo frontal (área de acceso / pasillo frío): $....\text{ m}$.
* La profundidad del gabinete estándar Huawei con cerramiento es de $....\text{ m}$.
  $$\text{Ancho Total de la Sala} = .....\text{ m} + ....\text{ m} + .....\text{ m} = \mathbf{[\text{ ? }]}\text{ m}$$

#### 1.2 Deducción de Holguras en los Extremos de la Hilera
* Largo total de la sala: $....\text{ m}$.
* La hilera está conformada por ... gabinetes estándar de $....\text{ mm}$ ($....\text{ m}$) de ancho nominal cada uno.
  $$\text{Longitud de la Hilera} \approx ... \times .....\text{ m} = ....\text{ m a } ....\text{ m}$$
  $$\text{Espacio remanente total} = .....\text{ m} - ......\text{ m} = ....\text{ m}$$
* Al estar la hilera centrada geométricamente:
  $$\text{Distancia a la pared norte} = \mathbf{[\text{ ? }]}\text{ m} \quad\text{y}\quad \text{Distancia al muro de vidrio sur} = \mathbf{[\text{ ? }]}\text{ m}$$

#### 1.3 Identificación del Gabinete N.° .... y Carga de TI
* Observa el recorrido de las tuberías de cobre (roja y azul) y de drenaje (celeste). ¿A qué gabinete ingresan directamente?
* Si la memoria descriptiva general indica que la sala soporta un total de **$......\text{ kW}$ de TI distribuida de forma uniforme**, determina:
  $$\text{Carga por cada rack de TI} = \frac{.....\text{ kW}}{3\text{ racks de TI}} = \mathbf{[\text{ ? }]}\text{ kW/rack}$$


### FASE 2: Digitalización del Plano As-Built en Draw.io (50 min)

1. **Configuración del Espacio de Trabajo:**
   * Ingresa a [Draw.io](https://app.diagrams.net/).
   * Ajusta el formato de página en **Horizontal (Landscape)**.
   * Activa **Grid (Cuadrícula)** y define el paso en `10 pt` (cada .... celdas = $.....\text{ m}$).

2. **Trazado de Paredes Perimetrales y Accesos:**
   * Dibuja el perímetro interior de la habitación con las cotas calculadas ($.....\text{ m} \times ....\text{ m}$).
   * Dibuja el muro frontal de **vidrio templado pavonado** con la puerta batiente de $......\text{ m}$ con apertura hacia el exterior.
   * Representa el muro de fondo con espesor de $........\text{ m}$.

3. **Modelado de la Hilera FusionModule800:**
   * Dibuja los 5 gabinetes alineados verticalmente según su orientación real (**FRONT hacia la derecha / Este**):
     * **Rack 1 (Norte):** `BATERÍAS` ($......\text{ m} \times ......\text{ m}$)
     * **Rack 2:** `TI (...... kW)` ($......\text{ m} \times ......\text{ m}$)
     * **Rack 3:** `FACILITIES (NetCol + UPS)` ($......\text{ m} \times ......\text{ m}$)
     * **Rack 4:** `TI (...... kW)` ($......\text{ m} \times ......\text{ m}$)
     * **Rack 5 (Sur):** `TI (...... kW)` ($......\text{ m} \times ......\text{ m}$)

4. **Trazado Mecánico (Climatización y Drenaje):**
   * Dibuja las **2 Unidades Condensadoras Exteriores** ubicadas fuera del muro norte (separadas a $......\text{ m}$ del muro).
   * Dibuja las rutas de tubería hacia el Rack 3 (*FACILITIES*):
     * **Línea Roja gruesa:** Circuito de refrigerante principal.
     * **Línea Azul discontinua:** Circuito de refrigerante de backup.
     * **Línea Celeste delgada:** Tubería de drenaje de condensados con salida exterior.

5. **Acotado Técnico Completo:**
   * Inserta cotas numéricas legibles: ancho total ($......\text{ m}$), largo ($......\text{ m}$), pasillo frontal ($......\text{ m}$), pasillo posterior ($......\text{ m}$) y holguras superior/inferior ($......\text{ m}$).
   * Inserta **flechas azules** que indiquen la inyección de aire frío por el frente (derecha) y **flechas rojas** que indiquen el escape de aire caliente por la parte posterior (izquierda).


### FASE 3: Auditoría Normativa y Preguntas de Control (20 min)

Responde de forma fundamentada en tu memoria técnica:

1. **Auditoría de Clearances (Huawei Standard):**
   * Completa la siguiente tabla comparativa de cumplimiento:

   | Parámetro Evaluado | Valor Mínimo Huawei | Valor en Plano Real | ¿Cumple? (SÍ / NO) | Justificación Técnica |
   | :--- | :--- | :--- | :--- | :--- |
   | **Pasillo Frontal (Operación)** | $\ge ......\text{ m}$ | | | |
   | **Pasillo Trasero (Escape/Mantenimiento)** | $\ge ......\text{ m}$ | | | |
   | **Holgura a Extremos de Fila** | $\ge ......\text{ m}$ | | | |

2. **Estrategia de Ubicación del Rack de FACILITIES:**
   * ¿Por qué el gabinete de climatización (*FACILITIES*) se ubicó en la posición central (Rack 3) y no en uno de los extremos? Explica el impacto en la distribución del flujo de aire y en el balance de presión.

3. **Análisis de Accesibilidad y Maniobra de Obra:**
   * El vano libre de la puerta de vidrio es de $......\text{ m}$, mientras que el gabinete de TI tiene una profundidad de $......\text{ m}$ (y $......\text{ m}$ con el marco de cerramiento). ¿Cómo se introduce físicamente este rack dentro de la sala sin desmontar la pared de vidrio?

4. **Esquema de Redundancia Frigorífica:**
   * Al observar que existen dos condensadores exteriores y dos pares de tuberías (roja y azul), ¿qué tipo de esquema de confiabilidad se está aplicando según la clasificación de centros de datos?


## 5. ENTREGABLES

Subir a la plataforma académica en una carpeta comprimida (`.zip` con formato `Apellido_Nombre_Lab_FM800`):
1. **Archivo editable:** Plano digitalizado en formato `.drawio`.
2. **Plano final:** Exportado en formato `.pdf` o `.png` de alta resolución.
3. **Memoria de Auditoría:** Documento en PDF con los cálculos de la Fase 1 y las respuestas del cuestionario de la Fase 3.


## 6. RÚBRICA DE EVALUACIÓN (Escala 0 a 20 puntos)

| Criterio Evaluado | Nivel Excelente (Puntaje Máx.) | Nivel Regular | Nivel Deficiente |
| :--- | :--- | :--- | :--- |
| **1. Deducción de Cotas y Cálculos Faltantes** *(Fase 1)* | **(5 pts)** Deduce con exactitud geométrica las medidas de la sala ($4.35\text{ m}$), las holguras ($1.00\text{ m}$) y la potencia unitaria ($1.2\text{ kW}$) mostrando el procedimiento matemático. | **(3 pts)** Deduce las medidas con errores menores de redondeo o sin mostrar el sustento de cálculo. | **(1 pt)** No logra deducir las dimensiones o los valores son inconsistentes con el plano. |
| **2. Calidad de Modelado y Acotado en Draw.io** *(Fase 2)* | **(6 pts)** Reconstrucción exacta a escala métrica, gabinetes correctamente rotulados, cotas visibles completas y simbología de puertas y muros fiel al plano As-Built. | **(4 pts)** El plano presenta distorsión en la escala, faltan cotas críticas o los textos no están alineados. | **(1 pt)** Diagrama incompleto, sin escala métrica o elementos desproporcionados. |
| **3. Representación de Tuberías y Flujo Térmico** *(Fase 2)* | **(4 pts)** Traza con precisión los circuitos de refrigerante (rojo/azul), drenaje (celeste) y representa correctamente la inyección/retorno de aire. | **(2 pts)** Traza las tuberías pero confunde los circuitos o no representa la dirección del aire frío/caliente. | **(0 pts)** Omite las instalaciones termomecánicas o los condensadores. |
| **4. Auditoría Normativa y Preguntas de Control** *(Fase 3)* | **(5 pts)** Tabla de clearances completada correctamente y respuestas técnicas sólidamente sustentadas bajo estándares Huawei / TIA-942. | **(3 pts)** Respuestas breves con justificación superficial o errores en la interpretación de redundancia. | **(1 pt)** No responde a las preguntas críticas o sus argumentos violan normas de seguridad. |
