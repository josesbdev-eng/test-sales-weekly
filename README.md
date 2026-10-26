# Predicción de conversiones con regresión

Prueba técnica de **CSA Havas** — Parte I: análisis exploratorio, estructura temporal, modelado y validación estadística de las conversiones semanales a partir de la inversión publicitaria.

El objetivo es explicar y predecir `Conversions` usando la inversión semanal en dos canales propios (`Spend Channel 1` y `Spend Channel 2`) y la inversión estimada de la competencia (`Spend Competitors`), evaluando además si las variables de calendario aportan información adicional.

---

## Contenido

- [Descripción general](#descripción-general)
- [Dataset](#dataset)
- [Estructura del repositorio](#estructura-del-repositorio)
- [Requisitos e instalación](#requisitos-e-instalación)
- [Orden de ejecución](#orden-de-ejecución)
- [Metodología](#metodología)
- [Resultados principales](#resultados-principales)
- [Conclusiones](#conclusiones)
- [Siguientes pasos](#siguientes-pasos)
- [Notas de reproducibilidad](#notas-de-reproducibilidad)

---

## Descripción general

El proyecto cubre el ciclo completo de un análisis de regresión aplicado a marketing:

1. **EDA general**: calidad de datos, descriptivos, outliers, distribución, correlaciones y codificación de variables.
2. **Estructura temporal**: construcción del calendario semanal, cobertura de la serie y visualización por año y semana ISO.
3. **Modelado**: regresión lineal múltiple con variables de negocio y con variables de tiempo.
4. **Comparación de modelos**: evaluación de múltiples algoritmos de regresión con `LazyPredict`.

El alcance actual corresponde a la **Parte I (Regresión)** del enunciado. La Parte II (curvas de respuesta y optimización de presupuesto) no está desarrollada en este repositorio.

## Dataset

| Atributo | Valor |
|---|---|
| Archivo fuente | `data/raw/raw_data.csv` (separador `;`) |
| Archivo original | `data/raw/Regression-Exercise.xlsx` |
| Frecuencia | Semanal (cierre domingo) |
| Periodo | 2021-03-07 a 2024-12-29 |
| Observaciones | 200 semanas |
| Valores perdidos | 0 |
| Variable objetivo | `Conversions` |

### Variables

| Variable | Tipo | Descripción |
|---|---|---|
| `Week` | Fecha | Semana de cierre (domingo) |
| `Spend Channel 1` | Numérica | Inversión semanal en el canal 1 |
| `Spend Channel 2` | Numérica | Inversión semanal en el canal 2 |
| `Spend Competitors` | Numérica | Inversión semanal estimada de la competencia |
| `Conversions` | Numérica | Conversiones obtenidas en la semana |

### Dataset procesado

`data/processed/data_general_eval.csv` se genera en `11_EDA_general.ipynb` e incorpora variables de calendario codificadas:

| Columna | Descripción |
|---|---|
| `SpendChannel1` | Inversión en canal 1 |
| `SpendChannel2` | Inversión en canal 2 |
| `SpendCompetitors` | Inversión de competidores |
| `YEAR` | Año codificado (1 = 2021, 2 = 2022, 3 = 2023, 4 = 2024) |
| `MONTH` | Mes (1–12) |
| `WN` | Número de semana ISO (1–52) |
| `Conversions` | Variable objetivo |

## Estructura del repositorio

```text
PruebaHavas/
├── data/
│   ├── raw/                          # Datos originales (no versionados)
│   │   ├── raw_data.csv
│   │   └── Regression-Exercise.xlsx
│   └── processed/                    # Datos generados por el EDA
│       └── data_general_eval.csv
├── docs/
│   └── CSA_Regression_Exercise 1.pdf # Enunciado de la prueba
├── notebooks/
│   ├── 11_EDA_general.ipynb          # EDA, descriptivos y codificación
│   ├── 12_EDA_estructura.ipynb       # Calendario y estructura temporal
│   ├── 13_model_regression.ipynb     # Regresión lineal múltiple e inferencia
│   └── 14_model_comparison.ipynb     # Comparación de modelos con LazyPredict
├── .gitignore
├── requirements.txt
└── README.md
```

## Requisitos e instalación

- **Python 3.13** (probado con 3.13.15)
- Entorno virtual recomendado

```bash
# Crear y activar el entorno virtual
python -m venv .venv
.venv\Scripts\activate        # Windows
# source .venv/bin/activate   # macOS / Linux

# Instalar dependencias
pip install -r requirements.txt
```

Dependencias principales: `pandas`, `numpy`, `siuba`, `seaborn`, `matplotlib`, `scipy`, `scikit-learn`, `statsmodels`, `lazypredict` e `ipywidgets`.

## Orden de ejecución

Ejecutar los notebooks desde la raíz del proyecto y en este orden:

1. `notebooks/11_EDA_general.ipynb` — genera `data/processed/data_general_eval.csv`.
2. `notebooks/12_EDA_estructura.ipynb` — construye el calendario y revisa la cobertura temporal.
3. `notebooks/13_model_regression.ipynb` — modelos lineales, métricas e inferencia.
4. `notebooks/14_model_comparison.ipynb` — comparación de algoritmos con `LazyPredict`.

```bash
jupyter lab
```

## Metodología

### 1. Análisis exploratorio

- Revisión de tipos, valores perdidos, ceros y duplicados.
- Estadísticos descriptivos, asimetría, curtosis y outliers (Tukey y z-score).
- Boxplots, pairplot y mapa de valores perdidos.
- Matriz de correlación de Spearman y mapa de calor.
- Codificación de variables de calendario y exportación del dataset procesado.

### 2. Estructura temporal

- Conversión de `Week` a fecha y creación de `Year`, `Month` y `WeekNumber` (semana ISO).
- Construcción de un calendario semanal del 2021-01-01 al 2024-12-31 (209 semanas) y unión con los datos observados.
- Cuantificación de semanas sin datos y mapa de valores perdidos.
- Visualización de cada serie por año, tanto por número de semana como en el eje temporal.

### 3. Modelado

- Partición temporal **80/20** con `shuffle=False`.
- Regresión lineal múltiple:
  - **Modelo principal**: `SpendChannel1 + SpendChannel2 + SpendCompetitors`.
  - **Modelo ampliado**: variables anteriores + `YEAR + MONTH + WN`.
- Métricas: **R²**, **R² ajustado** y **RMSE**.

### 4. Inferencia estadística

- Estimación por MCO con `statsmodels`.
- Prueba **F** global del modelo.
- Pruebas **t** individuales, intervalos de confianza al 95 % y nivel de significancia α = 0.05.
- Interpretación de coeficientes en unidades de conversiones.

### 5. Comparación de modelos

- Evaluación automatizada de más de 40 algoritmos de regresión con `LazyPredict`.
- Comparación por R², R² ajustado, RMSE y tiempo de entrenamiento.

## Resultados principales

### Análisis exploratorio

| Variable | Media | Desv. estándar | Mínimo | Máximo |
|---|---:|---:|---:|---:|
| Spend Channel 1 | 147.04 | 85.85 | 0.70 | 296.40 |
| Spend Channel 2 | 23.26 | 14.85 | 0.00 | 49.60 |
| Spend Competitors | 30.55 | 21.78 | 0.30 | 114.00 |
| Conversions | 14.02 | 5.22 | 1.60 | 27.00 |

- No se detectaron valores perdidos.
- `Spend Competitors` es la serie más asimétrica (asimetría 0.89, curtosis 3.60); con el criterio z > 2.5 se identifican 3 posibles outliers. `Conversions` presenta una asimetría leve (0.41).
- Correlación de Spearman con `Conversions`: canal 1 = **0.801**, canal 2 = **0.554**, competidores = **0.195**.
- `MONTH` y `WN` están fuertemente correlacionadas (ρ ≈ **0.94**), lo que anticipa información redundante si se incluyen ambas.
- Canal 2 y competidores muestran una correlación positiva moderada (ρ ≈ **0.32**).

### Estructura temporal

- El calendario de 209 semanas incluye **9 semanas vacías al inicio** (2021-01-03 a 2021-02-28), equivalentes al **4.31 %** de las filas. Corresponden a semanas anteriores a la primera observación (2021-03-07), no a pérdidas dentro del periodo analizado.
- A partir de la primera observación, la serie es semanal continua y sin duplicados.

### Regresión lineal

| Modelo | R² | R² ajustado | RMSE |
|---|---:|---:|---:|
| Principal (3 variables) | 0.8955 | 0.8868 | 1.6707 |
| Principal + variables de tiempo | 0.8967 | 0.8780 | 1.6606 |

El R² apenas mejora (+0.0013) y el R² ajustado disminuye al añadir año, mes y semana. Las variables de tiempo no aportan poder explicativo suficiente para justificar los parámetros adicionales, por lo que el **modelo principal es preferible por parsimonia**.

**Coeficientes del modelo ampliado:**

| Predictor | Coeficiente | p-valor | Conclusión (α = 0.05) |
|---|---:|---:|---|
| Spend Channel 1 | 0.047208 | 5.12 × 10⁻⁶⁵ | Significativo positivo |
| Spend Channel 2 | 0.179978 | 1.72 × 10⁻³⁹ | Significativo positivo |
| Spend Competitors | −0.001043 | 0.8763 | No significativo |
| Year | −0.048321 | 0.7544 | No significativo |
| Month | −0.006206 | 0.9538 | No significativo |
| WeekNumber | 0.002387 | 0.9219 | No significativo |

- Prueba F global: **F = 220.28**, p ≈ 1.30 × 10⁻⁷² → el modelo es significativo en conjunto.
- Un aumento de una unidad en el canal 1 se asocia con **+0.0472 conversiones**; en el canal 2, con **+0.1800 conversiones**, manteniendo el resto constante.
- Los intervalos de confianza al 95 % de ambos canales excluyen el cero: `[0.0440, 0.0504]` y `[0.1602, 0.1998]`.
- El canal 2 tiene mayor retorno marginal por unidad invertida, aunque el canal 1 concentra la mayor parte del presupuesto.
- La inversión de competidores y las variables de tiempo no muestran evidencia de asociación lineal individual tras controlar por los demás predictores.
- El intercepto es significativo (**3.0417**, p ≈ 3.32 × 10⁻⁶), aunque su interpretación práctica es limitada porque el escenario de predictores en cero está fuera del rango observado.

### Comparación de modelos

Los modelos de ensamble superan a la regresión lineal en el conjunto de prueba:

| Modelo | R² | R² ajustado | RMSE |
|---|---:|---:|---:|
| GradientBoostingRegressor | 0.9881 | 0.9860 | 0.5633 |
| ExtraTreesRegressor | 0.9858 | 0.9832 | 0.6168 |
| HistGradientBoostingRegressor | 0.9768 | 0.9726 | 0.7863 |
| RandomForestRegressor | 0.9762 | 0.9718 | 0.7979 |
| LinearRegression | 0.8967 | 0.8780 | 1.6606 |

## Conclusiones

1. El modelo lineal con los tres regresores de inversión explica aproximadamente el **89 %** de la variabilidad de las conversiones en el conjunto de prueba.
2. Solo la inversión en los dos canales propios es estadísticamente significativa; la inversión de competidores no aporta evidencia de asociación lineal tras controlar por los demás predictores.
3. Las variables de calendario (`YEAR`, `MONTH`, `WN`) no mejoran el R² ajustado y presentan redundancia entre sí, por lo que no se justifica incluirlas en el modelo final.
4. El canal 2 ofrece un mayor efecto marginal por unidad monetaria, mientras que el canal 1 concentra el mayor volumen de inversión.
5. Los modelos basados en árboles mejoran notablemente la capacidad predictiva (R² ≈ 0.99), a costa de menor interpretabilidad y de un mayor riesgo de sobreajuste.
6. La estructura temporal debe interpretarse con cautela: las 9 semanas vacías al inicio son anteriores al estudio y no deben tratarse como datos faltantes que requieran imputación.

## Siguientes pasos

- Desarrollar la **Parte II**: curvas de respuesta, saturación por canal y optimización de presupuesto con ROAS.
- Profundizar el análisis temporal (tendencia, estacionalidad y autocorrelación) antes de considerar modelos con rezagos o adstock.
- Calcular conversiones incrementales, contribución por canal, ingresos y CPA anual, según los apartados restantes del enunciado.
- Evaluar la estabilidad de los modelos de ensamble mediante validación temporal y ajuste de hiperparámetros.

## Notas de reproducibilidad

- Los archivos de `data/raw/` y `data/processed/` están excluidos del control de versiones mediante `.gitignore`. Para reproducir el análisis, coloca `raw_data.csv` en `data/raw/` y ejecuta `11_EDA_general.ipynb` para generar el dataset procesado.
- Los notebooks fijan la raíz del proyecto con `os.chdir(...)` a una ruta local de Windows. Si clonas el repositorio en otra ubicación, ajusta esa ruta o ejecuta los notebooks desde la raíz del proyecto.
- La partición de datos es temporal (`shuffle=False`) para reproducir el uso real del modelo: entrenar con el pasado y evaluar con el futuro.
