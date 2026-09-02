# Diseño del Sistema de Control de Péndulo Invertido Rotatorio (RotPen)
Diseño del Sistema de Control de Péndulo Invertido Rotatorio (RotPen) 
1. Definición de requisitos 

A partir de la referencia funcional del sistema de péndulo invertido rotatorio e investigación del equipo se establecen los siguientes requisitos funcionales y no funcionales del prototipo. 

1.1 Requisitos funcionales (RF) 

Lectura de variables: el Arduino mide el ángulo del péndulo mediante un encoder incremental y la posición del brazo mediante el encoder del motor, en cada ciclo de muestreo. 

Control PID: el Arduino calcula el error entre el ángulo del péndulo y el set-point vertical, y genera una señal PWM que corrige la posición del brazo para mantener el péndulo equilibrado. 

Transmisión de variables: el sistema envía en tiempo real las variables P, I, D, el set-point y el ángulo de respuesta del péndulo hacia un panel de control mediante comunicación serial o inalámbrica. 

Supervisión embebida: el Arduino detecta condiciones anómalas de operación, como un ángulo fuera de rango recuperable, y apaga el motor de forma autónoma sin depender de la comunicación externa. 

1.2 Requisitos no funcionales (RNF) 

Frecuencia de actualización: el lazo de control se ejecuta a un periodo fijo de 5 a 10 milisegundos para garantizar la estabilidad del péndulo. 

Precisión del control: el sistema mantiene el péndulo balanceado con un error en estado estacionario menor a 2 grados. 

Fiabilidad de la comunicación: el enlace entre el Arduino y el panel de control reanuda la transmisión automáticamente ante una pérdida temporal de conexión. 

Seguridad eléctrica: el circuito incluye protección contra sobrecorriente y separación entre la etapa lógica y la etapa de potencia del motor. 
2. Análisis del problema 

El péndulo invertido rotatorio es un sistema subactuado y de equilibrio inestable. Un motor gira un brazo horizontal alrededor de un eje vertical, y en el extremo del brazo se articula libremente un péndulo que tiende a caer por gravedad. El sistema cuenta con un solo actuador, el motor del brazo, y debe controlar dos variables acopladas: la posición del brazo y el ángulo del péndulo respecto a la vertical. 

El desafío técnico consiste en mover el brazo con la velocidad y dirección precisas para empujar al péndulo hacia arriba y sostenerlo en su punto de equilibrio, reaccionando en milisegundos ante cualquier desviación. El Arduino debe sostener este control en tiempo real con recursos limitados de memoria y procesamiento, sin que la transmisión de datos ni la supervisión de seguridad retrasen el cálculo del PID. 

2.1 Variables críticas de diseño 

ángulo del péndulo respecto a la vertical superior — variable controlada principal. 

posición angular del brazo respecto a su origen — variable manipulada indirectamente. 

velocidades angulares del péndulo y del brazo, requeridas para la acción derivativa e integral del PID. 

señal de control (ciclo útil PWM y sentido de giro) aplicada al driver del motor. 

Par motor y saturación: límite físico de torque disponible, que acota la capacidad de recuperación ante perturbaciones grandes. 

Fricción y backlash: rozamiento en los ejes y holguras mecánicas (poleas, acoples), que introducen no linealidades y ruido en la señal de control. 

Resolución de los encoders: pulsos por revolución de los sensores angulares, que determina la resolución mínima detectable del error. 

Periodo de muestreo T: tiempo de ciclo del lazo de control, crítico para la estabilidad dado que el péndulo es un sistema de dinámica rápida. 

3. Propuestas de arquitectura 

Se plantean tres opciones arquitectónicas, todas basadas en Arduino como microcontrolador principal, que priorizan características de rendimiento distintas: costo, precisión y robustez. 

Opción A — Bajo costo, motor DC con encoder incremental 

Prioriza: costo accesible y facilidad de réplica (arquitectura empleada en la mayoría de referencias documentadas de RotPen con Arduino). 

Arduino como controlador principal. 

Motor DC con caja reductora (12V) STEPPERONLINE 17HS19-2004S1 NEMA 17 2A 59Ncm motor bipolar de controlador gradual acoplado al eje del brazo mediante polea/correa dentada. 

Encoder incremental Aideepen (600–1000 PPR) Codificador rotativo incremental, 5V-24V en el eje del péndulo; encoder del propio motorreductor para la posición del brazo. 

Driver de potencia DVR8825 (puente H) para control bidireccional del motor. 

Fuente 12V/3A independiente para la etapa de potencia. 

Compromiso: el acople por polea/correa introduce holguras que reducen la precisión respecto a un acople directo, pero mantiene el costo y la complejidad de ensamblaje bajos. 

Opción B — Alta precisión, motor paso a paso (stepper) 

Prioriza: precisión y repetibilidad en el posicionamiento del brazo (control de lazo abierto determinístico). 

Arduino Mega (mayor número de interrupciones y memoria disponibles para el lazo de control y la comunicación simultánea). 

Motor paso a paso NEMA17 acoplado directamente al eje del brazo, con driver A4988 o DRV8825. 

Encoder magnético AS5600 (sin contacto, alta resolución) en el eje del péndulo. 

Fuente 12V/2A dedicada al driver del motor paso a paso. 

Compromiso: el motor paso a paso ofrece posicionamiento del brazo sin deslizamiento ni necesidad de encoder adicional en el brazo, pero su velocidad angular máxima y el rizado de par a bajas velocidades pueden limitar la capacidad de reacción ante perturbaciones rápidas, además de tener mayor costo. 

Opción C — Robustez, motor DC de alto torque con fusión sensorial (IMU) 

Prioriza: robustez ante perturbaciones externas y velocidad de respuesta dinámica. 

Arduino Uno o Mega como controlador principal. 

Motor DC de alto torque con caja reductora y encoder integrado en el eje del brazo (acople directo). 

IMU MPU6050 (acelerómetro + giroscopio) en el péndulo como sensor complementario al encoder, combinada mediante filtro complementario/Kalman para reducir ruido y deriva. 
4. Objetivos 

4.1 Objetivo general 

Diseñar e implementar un prototipo embebido, basado en Arduino, capaz de estabilizar un péndulo invertido rotatorio en su posición de equilibrio vertical mediante un controlador PID, integrando acondicionamiento electrónico, transmisión de variables en tiempo real, supervisión embebida de seguridad y una tarjeta de circuito impreso (PCB) propia. 

4.2 Objetivos específicos 

Definir los requisitos y las variables críticas del sistema de péndulo invertido rotatorio. 

Diseñar y comparar tres arquitecturas de control, seleccionando la más adecuada para el prototipo. 

Implementar un controlador PID en Arduino que estabilice el péndulo en su posición vertical. 

Diseñar el acondicionamiento electrónico, la etapa de potencia y la PCB del sistema. 

Desarrollar la transmisión de variables hacia un panel de control con visualización en tiempo real. 

Las tres arquitecturas propuestas se evalúan de forma analítica bajo los mismos criterios de diseño, con una calificación cualitativa relativa (Alto / Medio / Bajo) que sustenta la decisión de arquitectura para el prototipo. 

      
