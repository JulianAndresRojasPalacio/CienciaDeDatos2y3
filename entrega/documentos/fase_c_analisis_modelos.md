# Fase C — Análisis de modelos y justificación del modelo final

Notebook: `fase_c_modelos.ipynb` · Métricas: `resultados/*.csv`

**Protocolo:** un único `train_test_split` 80/20, `stratify` y `random_state=42` para todos los modelos; **imputación (KNN + mediana) y escalado dentro de un `Pipeline`** ajustado solo con train; métricas siempre en **test**. Además, **validación temporal** de todos los modelos. Se excluyen de las features `APROBACIÓN*` (identidad contable con la deserción) y `DESERCIÓN_<nivel>` (componentes del objetivo).

## Clasificación — alto riesgo de deserción (test, n = 2.878, 19 % positivos)

| Modelo | Accuracy | Precision | Recall | F1 | ROC-AUC | Entrenamiento |
|---|---|---|---|---|---|---|
| Baseline (clase mayoritaria) | 0.807 | 0.000 | 0.000 | 0.000 | 0.500 | 0.3 s |
| Regresión logística | 0.720 | 0.375 | **0.674** | 0.482 | 0.785 | 0.3 s |
| KNN (k = 5, distance; CV) | 0.818 | 0.551 | 0.300 | 0.389 | 0.741 | 16 s |
| Random Forest | 0.826 | 0.578 | 0.358 | 0.442 | 0.802 | 2 s |
| Gradient Boosting | **0.829** | **0.610** | 0.324 | 0.423 | **0.804** | 21–30 s |
| **Random Forest, umbral 0.36 (final)** | 0.786 | 0.458 | 0.581 | **0.512** | 0.802 | — |

El umbral 0.36 se eligió maximizando F1 con validación cruzada **dentro de train**; el test no se usó para elegirlo.

## Validación temporal — todos los modelos (entrena 2012–2021, umbral elegido en 2022, prueba 2023–2024)
Evita que el modelo se apoye en patrones compartidos entre años de un mismo municipio: ningún dato futuro toca el entrenamiento ni la elección del umbral.

| Modelo | Umbral (2022) | Precision | Recall | F1 | ROC-AUC |
|---|---|---|---|---|---|
| Regresión logística | 0.52 | 0.376 | 0.873 | 0.525 | **0.846** |
| KNN | 0.14 | 0.260 | **0.906** | 0.404 | 0.750 |
| **Random Forest** | 0.39 | 0.414 | 0.811 | **0.548** | 0.834 |
| Gradient Boosting | 0.42 | **0.476** | 0.636 | 0.544 | 0.825 |

Los modelos **no empeoran** en años que nunca vieron, así que no dependían de "recordar" municipios. La precisión de Random Forest (41 %) equivale a ~1,4 alertas falsas por acierto, 2,1 veces mejor que alertar al azar (19 %). Parte de esas falsas alarmas viene de que 2022, el año en que se eligió el umbral, fue un pico (30 % en riesgo). Si los recursos son limitados, se sube el umbral siguiendo la curva precisión-recall (`figuras/c_pr_temporal.png`).

## Regresión complementaria — tasa de deserción (%)

| Modelo | MSE | RMSE | MAE | R² |
|---|---|---|---|---|
| Regresión lineal | 3.439 | 1.855 | 1.317 | 0.280 |
| **Random Forest Regressor** | **3.202** | **1.789** | **1.266** | **0.329** |
| Gradient Boosting Regressor | 3.302 | 1.817 | 1.265 | 0.309 |

## No supervisado — municipios 2024 (K-Means, k = 4)

| Modelo | Silhouette ↑ | Davies-Bouldin ↓ |
|---|---|---|
| **K-Means** | **0.186** | **1.582** |
| Jerárquico Ward | 0.115 | 1.809 |

Segmentos: **Deserción crítica** (216; 75 % en alto riesgo), **Rurales pequeños y estables** (390), **Urbanos de alta cobertura** (303) y **Rezago y baja cobertura** (212; peor acceso, pero deserción reportada baja: los jóvenes no llegan a matricularse).

## Justificación del modelo final: Random Forest
- **Precisión:** el mejor F1 en **ambos** protocolos: 0.51 en el test aleatorio (umbral 0.36) y 0.55 al predecir 2023–2024 (umbral 0.39 elegido en 2022, AUC 0.83, recall 81 %). Generaliza a años futuros, que es su uso real.
- **Interpretabilidad:** sus importancias se explican en lenguaje de negocio (historial de deserción, cobertura en educación media, reprobación en primaria, tamaño). La regresión logística es más transparente y ordena bien (AUC 0.85 en el esquema temporal), pero con su umbral genera más falsas alarmas (precisión 38 %).
- **Simplicidad y velocidad:** ~2 s de entrenamiento, unas 10 veces más rápido que Gradient Boosting con resultados casi iguales.

**Lectura de negocio:** el modelo detecta entre 6 y 8 de cada 10 municipios que tendrán deserción alta. De cada 10 alertas, unas 4 son casos reales, lo que basta como **alerta temprana** para priorizar visitas y recursos, pero no como diagnóstico definitivo.
