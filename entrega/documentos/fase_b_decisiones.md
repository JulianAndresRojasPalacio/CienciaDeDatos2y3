# Fase B — Pasos de limpieza y justificación de cada decisión

Pipeline: `fase_b_limpieza.ipynb` · Resultado: `datos_limpios.parquet` (15.558 filas × 59 columnas).

| # | Paso | Decisión | Justificación |
|---|---|---|---|
| 1 | Formato numérico | `"56,11%"` → `56.11` | Las tasas vienen como texto con `%` y coma decimal. |
| 2 | **POBLACIÓN 2021** | Parser propio: `"1.148"` → 1148, `"3.89"` → 3890, `"1,174,274"` → 1174274 | En 2021 viene con separador de miles. Una conversión directa dejaba 925 municipios con ~3 habitantes. Se verifica con un `assert`. |
| 3 | Llaves | Códigos DANE rellenados a 5 dígitos (municipio) y 2 (departamento) | 1.776 filas venían sin el cero inicial, lo que partía un municipio en dos (1.272 → 1.124 códigos). |
| 4 | Filas `NACIONAL` | Eliminar | Son el agregado del país, no un municipio. |
| 5 | Nombres | Normalizar por código (Bogotá, San Andrés, Quindío) | Un mismo departamento tenía dos escrituras. |
| 6 | `TAMAÑO_PROMEDIO_DE_GRUPO`, `SEDES_CONECTADAS_A_INTERNET` | **Eliminar columna** | ~51 % nulos. El faltante es estructural (no existen desde 2018); imputar 7 años sería inventar datos. |
| 7 | `DESERCIÓN` nula (142) | **Eliminar fila** | No se imputa el objetivo: se crearían etiquetas falsas. |
| 8 | Cobertura bruta = 0 % (4) | Eliminar fila | Es un registro imposible (error), no un outlier. |
| 9 | `POBLACIÓN_5_16` nula (6) | **Mediana del mismo municipio** | La población cambia poco año a año; la mediana nacional mezclaría Bogotá con veredas. |
| 10 | Coberturas nulas | **KNN (k = 5)**, dentro del `Pipeline` de la Fase C | Los niveles de cobertura están muy correlacionados (r ≈ 0,9), y KNN aprovecha los demás niveles del mismo registro. **Se ajusta solo con train** (sin fuga de información). |
| 11 | Reprobación y repitencia | **Mediana**, dentro del `Pipeline` de la Fase C | Distribuciones sesgadas: la media se desplaza hacia la cola y la mediana no. **Se ajusta solo con train.** |
| 11b | Aprobación y deserción por nivel | **Mediana** en la Fase B | No son features de ningún modelo; solo se usan de forma descriptiva. |
| 12 | Outliers (IQR) | **Conservar** | Son municipios reales; los de deserción extrema son justo los que se quiere detectar. Coberturas > 100 % son válidas según la metodología del MEN. Se controlan con log, árboles y escalado. |
| 13 | Asimetría > 1 (9 variables) | **log1p** | Transformación fija (no aprende de los datos, así que no hay leakage) que maneja ceros. Solo se aplica si reduce la asimetría (se descartó en `COBERTURA_BRUTA_MEDIA`). |
| 14 | `REGION`, `TIPO_ETC` | **One-Hot** (`get_dummies`, `drop_first=True`) | Son nominales y no tienen orden; `drop_first` evita la trampa de la variable dummy. |
| 15 | `TAMAÑO_MUNICIPIO` | **Ordinal** (0 = Muy pequeño … 3 = Grande) | Tiene un orden natural que se conserva en una sola columna. |
| 16 | `DEPARTAMENTO`, `MUNICIPIO` | Solo como identificadores | Alta cardinalidad; la región ya captura la geografía. |
| 17 | `DESERCIÓN_AÑO_ANTERIOR` | Nueva variable, **sin imputar** | Historial conocido al momento de predecir. Inventar historial sería peor que no tenerlo. |
| 18 | `ALTO_RIESGO_DESERCION` | Deserción > 5 % | ~1,6 veces la mediana nacional; deja ~20 % de registros en riesgo, un grupo focalizable. |
| 19 | Escalado | **StandardScaler**, ajustado en la Fase C **después del split** | Tras el log, los datos son aproximadamente simétricos (supuesto del Z-score). MinMax se comprime con los outliers. **El parquet se guarda sin escalar y sin imputar las features** para evitar leakage. |

**Media vs. mediana:** se usó la mediana en todas las tasas porque son asimétricas. La media solo sería apropiada en variables simétricas, y no había ninguna con nulos.
**Sin fuga de información:** KNN y la mediana de las features no se ajustan con todo el dataset. El parquet conserva esos nulos (< 1 % por columna) y el imputador se ajusta **solo con train** dentro del `Pipeline` (la sección 8 de la Fase B lo demuestra; la Fase C lo usa en todos los modelos, en cada pliegue de CV y en la validación temporal). La población se completa con la mediana **del propio municipio**: es un atributo fijo del municipio, no un estadístico aprendido del conjunto.
