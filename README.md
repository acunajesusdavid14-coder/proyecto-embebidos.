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
