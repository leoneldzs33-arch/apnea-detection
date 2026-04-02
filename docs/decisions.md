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

## Decisión 03 — Arquitectura LSTM y resultados
Fecha: 31/03/2025

### Arquitectura elegida
LSTM(64) → Dropout(0.2) → LSTM(32) → Dropout(0.2) → Dense(16) → Dense(3)
Total: 29,891 parámetros — modelo compacto para uso en móvil.

### Resultados
- Accuracy general: 93.3%
- Recall apnea severa: 100% (ningún caso severo no detectado)
- Recall apnea leve: 74% (margen de mejora con datos reales)

### Por qué LSTM y no otra arquitectura
Las señales respiratorias son secuencias temporales donde el
contexto importa. Una pausa solo es apnea si dura más de 10
segundos — LSTM recuerda ese contexto, una red densa no.

### EarlyStopping en época 28
El modelo encontró su mejor punto en la época 28 de 50.
EarlyStopping evitó sobreajuste y ahorró tiempo de cómputo.

### Exportación TFLite
Se requirió SELECT_TF_OPS para las capas LSTM.
Tamaño final: 136 KB — viable para app móvil sin conexión.

## Decisión 05 — Análisis nocturno e IAH
Fecha: 31/03/2025

### Implementación
Se simuló una noche completa de 8 horas (960 ventanas de 30s).
El modelo analizó cada ventana y acumuló los resultados para
calcular el IAH — métrica clínica estándar para diagnóstico
de apnea del sueño.

### Resultado de la demo
- 960 ventanas analizadas
- 95.6% de precisión nocturna
- IAH calculado: 26.5 eventos/hora
- Diagnóstico correcto: Apnea moderada

### Patrón clínico reproducido
La simulación reproduce el patrón real: más apneas en la
segunda mitad de la noche durante fases REM. Esto valida
que el sistema puede detectar no solo si hay apnea, sino
cuándo ocurre durante la noche.

### Visualizaciones generadas
1. Hipnograma — mapa de color de la noche completa
2. Eventos por hora — gráfica de barras con umbrales clínicos
3. Probabilidad de apnea — curva continua suavizada

## Decisión 06 — Datos reales y Data Augmentation
Fecha: 01/04/2025

### Problema
30 grabaciones reales insuficientes para entrenar (66.7%).
Mezclar simulados + reales colapsó el modelo (48.4%) por
diferencia de distribución entre ambos datasets.

### Solución — Data Augmentation
Cada grabación real generó 9 variaciones mediante:
- Ruido gaussiano (variación eléctrica del sensor)
- Escala de amplitud (diferente presión del tope)
- Desplazamiento temporal (inicio diferente de ventana)

30 ejemplos → 300 ejemplos balanceados (100 por clase)

### Resultado
86.7% accuracy con datos 100% reales aumentados.
Recall apnea leve: 100% — ningún caso leve no detectado.
Recall apnea severa: 60% — mejorará con más grabaciones.

### Próximo paso
Con más sesiones de grabación real el modelo superará
el 93.3% del modelo simulado. Meta: 200 grabaciones reales.