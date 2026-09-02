# Diseño del Sistema de Control de Péndulo Invertido Rotatorio (RotPen)

**Avance 1 – Corte 1**

**Grupo:** Sistema de control de péndulo invertido rotacional
**Proyecto de Aula – Sistemas Embebidos**
**Microcontrolador base:** Arduino

---

## Tabla de contenido

1. [Definición de requisitos](#1-definición-de-requisitos)
2. [Análisis del problema](#2-análisis-del-problema)
3. [Propuestas de arquitectura](#3-propuestas-de-arquitectura)
4. [Objetivos](#4-objetivos)
5. [Justificación de la decisión de diseño](#51-justificación-de-la-decisión-de-diseño)
6. [Especificaciones mecánicas y de componentes](#6-especificaciones-mecánicas-y-de-componentes)
7. [Presupuesto económico](#7-presupuesto-económico)
8. [Metodología y cronograma](#8-metodología-y-cronograma)

---

## 1. Definición de requisitos

A partir de la referencia funcional del sistema de péndulo invertido rotatorio e investigación del equipo se establecen los siguientes requisitos funcionales y no funcionales del prototipo.

### 1.1 Requisitos funcionales (RF)

- **Lectura de variables:** el Arduino mide el ángulo del péndulo mediante un encoder incremental y la posición del brazo mediante el encoder del motor, en cada ciclo de muestreo.
- **Control PID:** el Arduino calcula el error entre el ángulo del péndulo y el set-point vertical, y genera una señal PWM que corrige la posición del brazo para mantener el péndulo equilibrado.
- **Transmisión de variables:** el sistema envía en tiempo real las variables P, I, D, el set-point y el ángulo de respuesta del péndulo hacia un panel de control mediante comunicación serial o inalámbrica.
- **Supervisión embebida:** el Arduino detecta condiciones anómalas de operación, como un ángulo fuera de rango recuperable, y apaga el motor de forma autónoma sin depender de la comunicación externa.

### 1.2 Requisitos no funcionales (RNF)

- **Frecuencia de actualización:** el lazo de control se ejecuta a un periodo fijo de 5 a 10 milisegundos para garantizar la estabilidad del péndulo.
- **Precisión del control:** el sistema mantiene el péndulo balanceado con un error en estado estacionario menor a 2 grados.
- **Fiabilidad de la comunicación:** el enlace entre el Arduino y el panel de control reanuda la transmisión automáticamente ante una pérdida temporal de conexión.
- **Seguridad eléctrica:** el circuito incluye protección contra sobrecorriente y separación entre la etapa lógica y la etapa de potencia del motor.

---

## 2. Análisis del problema

El péndulo invertido rotatorio es un sistema subactuado y de equilibrio inestable. Un motor gira un brazo horizontal alrededor de un eje vertical, y en el extremo del brazo se articula libremente un péndulo que tiende a caer por gravedad. El sistema cuenta con un solo actuador, el motor del brazo, y debe controlar dos variables acopladas: la posición del brazo y el ángulo del péndulo respecto a la vertical.

El desafío técnico consiste en mover el brazo con la velocidad y dirección precisas para empujar al péndulo hacia arriba y sostenerlo en su punto de equilibrio, reaccionando en milisegundos ante cualquier desviación. El Arduino debe sostener este control en tiempo real con recursos limitados de memoria y procesamiento, sin que la transmisión de datos ni la supervisión de seguridad retrasen el cálculo del PID.

### 2.1 Variables críticas de diseño

- Ángulo del péndulo respecto a la vertical superior — variable controlada principal.
- Posición angular del brazo respecto a su origen — variable manipulada indirectamente.
- Velocidades angulares del péndulo y del brazo, requeridas para la acción derivativa e integral del PID.
- Señal de control (ciclo útil PWM y sentido de giro) aplicada al driver del motor.
- **Par motor y saturación:** límite físico de torque disponible, que acota la capacidad de recuperación ante perturbaciones grandes.
- **Fricción y backlash:** rozamiento en los ejes y holguras mecánicas (poleas, acoples), que introducen no linealidades y ruido en la señal de control.
- **Resolución de los encoders:** pulsos por revolución de los sensores angulares, que determina la resolución mínima detectable del error.
- **Periodo de muestreo T:** tiempo de ciclo del lazo de control, crítico para la estabilidad dado que el péndulo es un sistema de dinámica rápida.

<img width="827" height="709" alt="image" src="https://github.com/user-attachments/assets/9d925fa9-109e-4c7e-807d-07d7e6d2b48e" />


---

## 3. Propuestas de arquitectura

Se plantean tres opciones arquitectónicas, todas basadas en Arduino como microcontrolador principal, que priorizan características de rendimiento distintas: costo, precisión y robustez.

### Opción A — Bajo costo, motor Paso a Paso con encoder incremental

Prioriza: costo accesible y facilidad de réplica (arquitectura empleada en la mayoría de referencias documentadas de RotPen con Arduino).

- Arduino Uno como controlador principal.
- Motor paso a paso STEPPER 17HS19-2004S1 NEMA 17 2A 59Ncm, motor bipolar de controlador gradual.
- Encoder incremental Aideepen (600–1000 PPR) codificador rotativo incremental, 5V-24V en el eje del péndulo; encoder del propio motorreductor para la posición del brazo.
- Controlador de motor paso a paso DVR8825.
- Fuente 12V/3A independiente para la etapa de potencia.

**Compromiso:** el acople por polea/correa introduce holguras que reducen la precisión respecto a un acople directo, pero mantiene el costo y la complejidad de ensamblaje bajos.

### Opción B — Alta precisión, Motor DC

Prioriza: precisión y repetibilidad en el posicionamiento del brazo (control de lazo abierto determinístico).

- Arduino Mega (mayor número de interrupciones y memoria disponibles para el lazo de control y la comunicación simultánea).
- Motor DC con caja reductora (12V) con driver de potencia puente H (L298N).
- Encoder magnético AS5600 (sin contacto, alta resolución) en el eje del péndulo.
- Fuente 12V/2A dedicada al driver del motor paso a paso.

**Compromiso:** el motor paso a paso ofrece posicionamiento del brazo sin deslizamiento ni necesidad de encoder adicional en el brazo, pero su velocidad angular máxima y el rizado de par a bajas velocidades pueden limitar la capacidad de reacción ante perturbaciones rápidas, además de tener mayor costo.

### Opción C — Robustez, motor DC de alto torque con fusión sensorial (IMU)

Prioriza: robustez ante perturbaciones externas y velocidad de respuesta dinámica.

- Arduino Uno o Mega como controlador principal.
- Motor DC de alto torque con caja reductora y encoder integrado en el eje del brazo (acople directo).
- IMU MPU6050 (acelerómetro + giroscopio) en el péndulo como sensor complementario al encoder, combinada mediante filtro complementario/Kalman para reducir ruido y deriva.
- Driver Cytron o BTS7960 de alta corriente para el motor.

**Compromiso:** la fusión sensorial mejora la robustez y la calidad de la medición del ángulo, pero incrementa la carga computacional sobre un microcontrolador de 8 bits y añade complejidad de calibración e integración, además de un consumo de corriente mayor.

---

## 4. Objetivos

### 4.1 Objetivo general

Diseñar e implementar un prototipo embebido, basado en Arduino, capaz de estabilizar un péndulo invertido rotatorio en su posición de equilibrio vertical mediante un controlador PID, integrando acondicionamiento electrónico, transmisión de variables en tiempo real, supervisión embebida de seguridad y una tarjeta de circuito impreso (PCB) propia.

### 4.2 Objetivos específicos

- Definir los requisitos y las variables críticas del sistema de péndulo invertido rotatorio.
- Diseñar y comparar tres arquitecturas de control, seleccionando la más adecuada para el prototipo.
- Implementar un controlador PID en Arduino que estabilice el péndulo en su posición vertical.
- Diseñar el acondicionamiento electrónico, la etapa de potencia y la PCB del sistema.
- Desarrollar la transmisión de variables hacia un panel de control con visualización en tiempo real.

### 4.3 Evaluación comparativa de arquitecturas

Las tres arquitecturas propuestas se evalúan de forma analítica bajo los mismos criterios de diseño, con una calificación cualitativa relativa (Alto / Medio / Bajo) que sustenta la decisión de arquitectura para el prototipo.

| Criterio | Opción A | Opción B (DC + polea) | Opción C (DC + IMU) |
| --- | --- | --- | --- |
| Costo estimado | Bajo | Medio-Alto | Medio |
| Precisión angular | Alta (sin deslizamiento) | Media (holgura de correa) | Alta (fusión sensorial) |
| Complejidad de implementación | Baja | Media | Alta |
| Velocidad de respuesta ante perturbaciones | Alta | Media | Alta |
| Robustez ante perturbaciones externas | Media | Media | Alta |
| Facilidad de escalado / modularidad | Alta | Media | Media |
| Carga computacional sobre Arduino | Baja | Media | Alta |

## 5.1 Justificación de la decisión de diseño

Se selecciona la **Opción A** (motor paso a paso bipolar con encoder incremental) como arquitectura base para el prototipo de este proyecto de aula. Esta decisión se sustenta en que:

1. Coincide con la arquitectura empleada en las referencias funcionales y de apoyo del proyecto, lo que reduce el riesgo técnico en las primeras iteraciones.
2. Mantiene el costo y la complejidad de ensamblaje dentro del alcance de un semestre académico.
3. Su velocidad de respuesta dinámica es adecuada para la naturaleza rápida del péndulo.
4. Su arquitectura modular permite incorporar en fases posteriores elementos de la Opción C (por ejemplo, una IMU de respaldo) sin rediseñar el sistema completo, lo que es coherente con el requisito no funcional de modularidad (RNF05).

La limitación de precisión asociada a la holgura del acople por correa se mitigará mecánicamente mediante tensores y, de ser necesario, se documentará como trabajo futuro.

---

## 6. Especificaciones mecánicas y de componentes

### 6.1 Dimensiones del prototipo (preliminares)

- **Base/soporte:** placa de acrílico o MDF de 20 cm × 20 cm × 6 mm, con perforaciones para anclaje del motor y de los soportes verticales.
- **Columna de soporte:** altura aproximada de 15 cm, en perfil de aluminio, filamento 3D o madera, para elevar el eje de giro del brazo.
- **Brazo rotativo:** barra de aluminio, filamento 3D o madera de 15 cm de longitud desde el eje del motor hasta el punto de articulación del péndulo.
- **Péndulo:** varilla de aluminio de 15–20 cm de longitud, con una masa concentrada de 30–50 g en el extremo libre para definir el momento de inercia.
- **Ejes y rodamientos:** rodamientos de bolas de bajo rozamiento en el eje de articulación del péndulo, para minimizar la fricción no modelada.

### 6.2 Lista de componentes y diagrama de bloques

El sistema se organiza en cuatro bloques funcionales interconectados:

1. Bloque de sensado (encoders)
2. Bloque de control (Arduino + firmware PID)
3. Bloque de potencia (driver + Stepper)
4. Bloque de comunicación/supervisión (transmisión al panel de control y monitoreo de seguridad)

**Flujo de información:**

```
Encoder péndulo/brazo → Acondicionamiento de señal → Arduino (cálculo PID + supervisión)
    → Acondicionamiento paso a paso → Driver → Motor Stepper → Brazo/Péndulo (planta)
```

con una rama paralela desde el Arduino hacia el módulo de comunicación y el panel de control.

| Componente | Cantidad | Función en el sistema |
| --- | --- | --- |
| Arduino Uno | 1 | Controlador principal: adquisición, cálculo del PID y supervisión embebida |
| Motor Paso a paso NEMA17 | 1 | Actuador del brazo rotativo |
| Encoder incremental en cuadratura (Aideepen 600 P/R) | 1 | Medición del ángulo del péndulo (θ) |
| Driver puente H (DRV8825) | 1 | Etapa de potencia y control bidireccional del motor |
| Módulo Bluetooth (HC-05) o Wi-Fi (ESP-01) | 1 | Transmisión inalámbrica de variables al panel de control |
| Fuente de alimentación 12V | 1 | Alimentación de la etapa de potencia |
| Regulador 5V | 1 | Alimentación de la etapa lógica y sensores |
| Rodamientos de bolas | 2 | Reducción de fricción en los ejes de giro |
| Estructura (acrílico/MDF + aluminio) | 1 juego | Soporte mecánico del prototipo |
| PCB de control y potencia | 1 | Integración física de las etapas del sistema |

### 6.3 Caracterización de materiales

- **Acrílico/MDF (base):** material rígido, liviano y de bajo costo, adecuado para mecanizado con corte láser y suficiente estabilidad para absorber vibraciones del motor.
- **Aluminio (brazo y péndulo):** baja densidad y alta rigidez específica, minimiza el momento de inercia no deseado y facilita el mecanizado y la fijación de sensores.
- **Piezas impresas en PLA (soportes y acoples):** fabricación rápida y de bajo costo mediante impresión 3D, útil para adaptadores entre ejes y encoders.
- **Rodamientos metálicos:** reducción del coeficiente de fricción en el eje de articulación del péndulo, variable crítica para no introducir no linealidades adicionales al modelo de control.

---

## 7. Presupuesto económico

Los valores relacionados a continuación corresponden a precios de referencia del mercado colombiano (proveedores de electrónica y plataformas como MercadoLibre), en pesos colombianos (COP), y deben validarse con cotizaciones actualizadas de los proveedores seleccionados antes de la fase de adquisición.

| Componente | Cant. | V. unitario (COP) | Subtotal (COP) | Etapa |
| --- | --- | --- | --- | --- |
| Arduino Uno R3 | 1 | $55.000 | $55.000 | Control |
| Motor paso a paso NEMA17 + encoder | 1 | $70.000 | $70.000 | Actuación |
| Encoder incremental (Aideepen 600 PPR) | 1 | $60.000 | $60.000 | Sensado |
| Driver puente H DRV8825 | 1 | $35.000 | $35.000 | Potencia |
| Módulo Bluetooth HC-05 | 1 | $25.000 | $25.000 | Comunicación |
| Fuente conmutada 12V/3A | 1 | $40.000 | $40.000 | Alimentación |
| Regulador 5V (módulo) | 1 | $8.000 | $8.000 | Alimentación |
| Rodamientos de bolas | 2 | $10.000 | $20.000 | Mecánica |
| Perfil y lámina de aluminio | 1 juego | $45.000 | $45.000 | Mecánica |
| Base acrílico/MDF (corte láser) | 1 | $35.000 | $35.000 | Mecánica |
| Piezas impresas en 3D (PLA) | 1 juego | $30.000 | $30.000 | Mecánica |
| Fabricación de PCB (servicio) | 1 | $60.000 | $60.000 | Electrónica |
| Conectores, cables y tornillería | 1 lote | $30.000 | $30.000 | Varios |

**Total estimado: ≈ $513.000 COP**

Este presupuesto no incluye herramientas de uso compartido en el laboratorio (impresora 3D, cortadora láser, estación de soldadura) ni imprevistos de fabricación, los cuales se estiman en un 10–15% adicional para la fase de implementación.

---

## 8. Metodología y cronograma

### 8.1 Metodología de desarrollo

Se adopta una metodología híbrida: el diseño de la arquitectura electromecánica y del PCB sigue un enfoque secuencial (dado que son entregables con dependencias físicas de fabricación), mientras que el desarrollo e integración del firmware de control (PID, comunicación y supervisión) se aborda de forma iterativa, con ciclos cortos de ajuste y prueba sobre el prototipo físico.

Las fases generales son:

1. **Análisis y diseño:** definición de requisitos, análisis del problema y selección de arquitectura.
2. **Desarrollo electrónico:** diseño del acondicionamiento de señales, la etapa de potencia y el diseño del PCB.
3. **Desarrollo de control:** implementación y sintonización del controlador PID sobre Arduino.
4. **Integración y comunicación:** desarrollo del módulo de transmisión de variables y del panel de control.
5. **Pruebas y ajuste:** validación del desempeño frente a los requisitos no funcionales y ajuste fino del PID.
6. **Documentación y sustentación:** consolidación de entregables y preparación de la sustentación final.

### 8.2 Cronograma (semestre académico – 16 semanas)

| Actividad | Objetivo(s) asociado(s) | Semanas |
| --- | --- | --- |
| Definición de requisitos y análisis del problema | Obj. 1 | 1 – 2 |
| Diseño y evaluación comparativa de arquitecturas | Obj. 2 | 2 – 3 |
| Entrega Avance 1 (este documento) | Obj. 1 y 2 | 3 |
| Diseño de acondicionamiento electrónico y potencia | Obj. 4 | 4 – 6 |
| Diseño y fabricación de la PCB | Obj. 6 | 5 – 8 |
| Ensamble de la estructura mecánica | Obj. 2, 6 | 6 – 8 |
| Implementación y sintonización del PID | Obj. 3 | 8 – 11 |
| Desarrollo del módulo de transmisión y panel de control | Obj. 5 | 9 – 12 |
| Entrega Avance 2 (integración electrónica y de control) | Obj. 3, 4, 5, 6 | 12 |
| Pruebas de desempeño y ajuste fino | Obj. 7 | 12 – 14 |
| Documentación final y preparación de sustentación | Todos | 14 – 15 |
| Sustentación final del proyecto | Todos | 16 |
