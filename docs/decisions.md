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


## Decisión 02 — Preprocesamiento y división del dataset
Fecha: 31/03/2025

### Normalización
Se usó MinMaxScaler (rango 0-1) porque las señales del sensor
flex pueden variar entre pacientes según la colocación física.
Normalizar elimina esa variabilidad de escala sin perder la
forma de la onda, que es lo que realmente importa.

### División 70/15/15
Se usó stratify=y para garantizar que los tres grupos tengan
la misma proporción de clases. Sin esto, el modelo podría
entrenarse con pocos ejemplos de apnea severa y nunca aprender
a detectarla correctamente.

### Reshape (muestras, 300, 1)
La LSTM necesita ver los datos como secuencia temporal.
El 1 al final representa una sola característica por paso
de tiempo. Cuando integremos oximetría (SpO2) ese valor
cambiará a 2, sin modificar la arquitectura del modelo.

### scaler.pkl
Se guarda el scaler entrenado para usarlo en la app móvil.
Es crítico aplicar exactamente la misma normalización a los
datos reales del Arduino, de lo contrario el modelo falla.