## Decisión 01 — Generación de datos simulados
Fecha: 30/03/2026

### Problema
No contamos aún con datos reales del sensor flex + Arduino.

### Decisión
Generar señales sintéticas basadas en parámetros clínicos reales:
- Respiración normal: 12–20 ciclos/minuto
- Apnea leve: pausas de 10–20 segundos
- Apnea severa: pausas de 20–28 segundos

### Por qué funciona
Las señales reales del sensor flex son ondas cuasisinusoidales
con ruido. Modelarlas como senos con ruido gaussiano es una
aproximación válida para entrenar la arquitectura inicial.

### Próximo paso
Cuando llegue el Arduino, sustituir el generador por lectura
del puerto serial — el resto del pipeline no cambia.