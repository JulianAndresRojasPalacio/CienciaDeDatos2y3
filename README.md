# Proyecto 2 y 3 — Anticipar la deserción escolar municipal en Colombia

**Curso:** Introducción a la ciencia de los datos · **Profesor:** Héctor Fabio Ocampo Arbeláez
**Grupo 1:** Juan Felipe Ruiz · Jhorman Ricardo Loaiza · Julian Andres Rojas · Juan Manuel Perez

**Pregunta de negocio:** ¿qué características de un municipio (cobertura, reprobación, repitencia, tamaño, región e historial) permiten anticipar si tendrá una **deserción escolar alta (> 5 %)**, para que el MEN y las Secretarías de Educación focalicen sus programas de permanencia?

**Dataset (el mismo del P1):** `MEN_ESTADISTICAS_EN_EDUCACION_EN_PREESCOLAR,_BÁSICA_Y_MEDIA_POR_MUNICIPIO_20261027.csv` (datos.gov.co): 15.707 registros, 1.122 municipios, 2011–2024.

## Cómo correr el proyecto
```bash
pip install -r requirements.txt
# (opcional) cp .env.example .env  y exportar DATA_PATH si el CSV está en otra ruta
cd entrega
jupyter nbconvert --to notebook --execute --inplace fase_a_eda.ipynb fase_b_limpieza.ipynb fase_c_modelos.ipynb
cd ..
python generar_pdfs.py        # regenera informe_p2p3.pdf y presentacion.pdf
```
O abrir cada notebook en Jupyter y ejecutar **Restart & Run All** **en orden** (A → B → C): la Fase B genera `datos_limpios.parquet`, que usa la Fase C. El CSV crudo debe estar en la raíz del proyecto (o se indica su ruta con `DATA_PATH`).

## Estructura
```
├── entrega/
│   ├── fase_a_eda.ipynb            ← EDA: histogramas, boxplots, Pearson/Spearman, líneas, barras, hexbin, violín + insights
│   ├── fase_b_limpieza.ipynb       ← nulos, duplicados, texto, outliers, log1p, One-Hot/Ordinal, escalado
│   ├── fase_c_modelos.ipynb        ← 4 clasificadores + baseline, 3 regresores, K-Means vs. Ward
│   ├── datos_limpios.parquet
│   ├── informe_p2p3.pdf            ← informe de 8 páginas (Fase D)
│   ├── presentacion.pdf            ← 11 diapositivas para 8–10 min
│   ├── documentos/                 ← documentos breves de las Fases A, B y C
│   ├── figuras/                    ← todas las gráficas (las generan los notebooks)
│   └── resultados/                 ← tablas de métricas y segmentos (CSV)
├── generar_pdfs.py
├── .env.example
├── requirements.txt
└── README.md
```

## Resultados en una línea
Random Forest: **F1 = 0,51, ROC-AUC = 0,80** en el test aleatorio, y **F1 = 0,55, AUC = 0,83** al predecir 2023–2024 sin datos futuros (validación temporal). La imputación y el escalado se ajustan solo con train, dentro del `Pipeline`. K-Means encuentra 4 perfiles de municipios: *Deserción crítica*, *Rurales estables*, *Urbanos de alta cobertura* y *Rezago y baja cobertura*.

## Acuerdo del grupo
> Los integrantes del grupo se comprometen a mantener el dataset elegido durante los tres proyectos del semestre (P1, P2/3, P4). Cambios sólo por autorización del profesor.
>
> Firmas (iniciales): JFR · JRL · JAR · JMP
