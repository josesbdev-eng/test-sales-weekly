# Predicción de conversiones semanales

### Resultados de las notebooks 11–14

Prueba técnica de **CSA Havas** — análisis exploratorio, estructura temporal, regresión lineal múltiple y comparación de algoritmos para explicar las conversiones semanales a partir de la inversión publicitaria.

El objetivo es estimar `Conversions` usando la inversión semanal en dos canales propios (`Spend Channel 1` y `Spend Channel 2`) y la inversión estimada de la competencia (`Spend Competitors`), evaluando además si las variables de calendario aportan información adicional.

---

## Contenido

- [Resumen ejecutivo](#resumen-ejecutivo)
- [Datos](#datos)
- [Resultados por notebook](#resultados-por-notebook)
  - [11 — EDA general](#11--eda-general)
  - [12 — Estructura temporal](#12--estructura-temporal)
  - [13 — Regresión lineal](#13--regresión-lineal)
  - [14 — Comparación de modelos](#14--comparación-de-modelos)
- [Conclusiones](#conclusiones)
- [Limitaciones](#limitaciones)
- [Reproducibilidad](#reproducibilidad)
- [Estructura del repositorio](#estructura-del-repositorio)

---

## Resumen ejecutivo

- Se analizan **200 semanas** de datos, desde el **7 de marzo de 2021** hasta el **29 de diciembre de 2024**, sin valores perdidos en el archivo original.
- La inversión en los dos canales propios explica aproximadamente el **89 %** de la variabilidad de las conversiones en el conjunto de prueba.
- Solo **Spend Channel 1** y **Spend Channel 2** presentan una asociación lineal estadísticamente significativa. **Spend Competitors** y las variables de tiempo no son significativas tras controlar por los demás predictores.
- Añadir `YEAR`, `MONTH` y `WN` apenas mejora el R² y **reduce el R² ajustado**, por lo que el modelo de tres variables es preferible por parsimonia.
- Los modelos de ensamble evaluados con `LazyPredict` alcanzan un R² de hasta **0.988** en el conjunto de prueba, pero requieren validación temporal y ajuste de hiperparámetros antes de considerarse modelos finales.

---

## Datos

| Atributo | Valor |
|---|---|
| Archivo fuente | `data/raw/raw_data.csv` (separador `;`) |
| Archivo original | `data/raw/Regression-Exercise.xlsx` |
| Frecuencia | Semanal (cierre domingo) |
| Periodo | 2021-03-07 a 2024-12-29 |
| Observaciones | 200 semanas |
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

`data/processed/data_general_eval.csv` se genera en `11_EDA_general.ipynb`:

| Columna | Descripción |
|---|---|
| `SpendChannel1` | Inversión en canal 1 |
| `SpendChannel2` | Inversión en canal 2 |
| `SpendCompetitors` | Inversión de competidores |
| `YEAR` | Año codificado (1 = 2021, 2 = 2022, 3 = 2023, 4 = 2024) |
| `MONTH` | Mes (1–12) |
| `WN` | Número de semana ISO (1–53) |
| `Conversions` | Variable objetivo |

---

## Resultados por notebook

### 11 — EDA general

**Notebook:** `notebooks/11_EDA_general.ipynb`

#### Calidad y estadísticos descriptivos

No se encontraron valores perdidos en las variables analizadas. El dataset final contiene **200 filas × 7 columnas** (4 numéricas y 3 de calendario codificadas).

| Variable | Media | Desv. estándar | Mínimo | Mediana | Máximo |
|---|---:|---:|---:|---:|---:|
| Spend Channel 1 | 147.04 | 85.85 | 0.70 | 149.75 | 296.40 |
| Spend Channel 2 | 23.26 | 14.85 | 0.00 | 22.90 | 49.60 |
| Spend Competitors | 30.55 | 21.78 | 0.30 | 25.75 | 114.00 |
| Conversions | 14.02 | 5.22 | 1.60 | 12.90 | 27.00 |

#### Distribución, asimetría y curtosis

La curtosis se calculó con el criterio de Pearson (`fisher=False`), donde una distribución normal tiene curtosis igual a 3.

| Variable | Asimetría | Curtosis | Interpretación |
|---|---:|---:|---|
| Spend Channel 1 | −0.07 | 1.77 | Aproximadamente simétrica y platicúrtica |
| Spend Channel 2 | 0.09 | 1.74 | Aproximadamente simétrica y platicúrtica |
| Spend Competitors | 0.89 | 3.60 | Asimétrica positiva y leptocúrtica |
| Conversions | 0.41 | 2.57 | Ligeramente asimétrica positiva |

`Spend Competitors` es la variable con mayor asimetría y curtosis, consistente con semanas de inversión inusualmente alta.

#### Valores atípicos

Con el criterio de **z-score > 2.5**:

| Variable | Observaciones señaladas |
|---|---:|
| Spend Channel 1 | 0 |
| Spend Channel 2 | 0 |
| Spend Competitors | 3 |
| Conversions | 0 |

El boxplot había señalado dos posibles outliers en `Spend Competitors`. La diferencia se debe al criterio utilizado: el boxplot aplica la regla de 1.5 × IQR, mientras que el z-score usa 2.5 desviaciones estándar. En ambos casos se trata de observaciones **potenciales**, no de errores confirmados.

#### Correlaciones de Spearman

| Variable | ρ con `Conversions` |
|---|---:|
| Spend Channel 1 | **0.801** |
| Spend Channel 2 | **0.554** |
| Spend Competitors | 0.195 |
| YEAR | −0.051 |
| MONTH | −0.010 |
| WN | 0.012 |

Relaciones entre predictores relevantes:

- `MONTH` y `WN`: **ρ = 0.941**, información prácticamente redundante.
- `Spend Channel 2` y `Spend Competitors`: **ρ = 0.317**, asociación positiva moderada.
- `Spend Channel 1` con `Spend Channel 2` y con competidores: **ρ ≈ 0.05**, asociación muy débil.

El canal 1 es el predictor con mayor asociación monotónica con las conversiones, seguido del canal 2. Estas correlaciones son descriptivas y no implican causalidad ni miden la importancia de cada variable tras controlar por las demás.

**Salida:** `data/processed/data_general_eval.csv`.

---

### 12 — Estructura temporal

**Notebook:** `notebooks/12_EDA_estructura.ipynb`

Se construyó un calendario semanal completo del **2021-01-03 al 2024-12-29** con `freq="W-SUN"`, obteniendo **209 semanas**. Al unirlo con las observaciones, quedan **9 semanas sin datos** al inicio de la serie.

| Columna | Valores perdidos | Porcentaje |
|---|---:|---:|
| Spend Channel 1 | 9 | 4.31 % |
| Spend Channel 2 | 9 | 4.31 % |
| Spend Competitors | 9 | 4.31 % |
| Conversions | 9 | 4.31 % |
| Week / Year / Month / WeekNumber | 0 | 0.00 % |

- Las 9 semanas vacías corresponden al periodo **2021-01-03 a 2021-02-28**, anterior a la primera observación (2021-03-07). No son pérdidas dentro del periodo observado, por lo que no se imputaron.
- La primera fila del calendario (2021-01-03) corresponde a la **semana ISO 53 de 2020**, aunque se etiquete como 2021 según el año calendario.
- A partir de la primera observación, la serie es semanal continua.
- El notebook no guarda un archivo procesado; su función es validar la cobertura y visualizar las series por año y número de semana.

---

### 13 — Regresión lineal

**Notebook:** `notebooks/13_model_regression.ipynb`

#### Partición de datos

- Partición temporal **80/20** con `shuffle=False`: **160 semanas para entrenamiento** y **40 para prueba**.
- Se ajustaron dos modelos:
  1. **Principal:** `SpendChannel1 + SpendChannel2 + SpendCompetitors`.
  2. **Ampliado:** variables anteriores + `YEAR + MONTH + WN`.

#### Métricas en el conjunto de prueba

| Modelo | R² | R² ajustado | RMSE |
|---|---:|---:|---:|
| Principal (3 variables) | 0.8955 | **0.8868** | 1.6707 |
| Principal + variables de tiempo | 0.8967 | 0.8780 | 1.6606 |

El R² apenas aumenta (+0.0013), mientras que el **R² ajustado disminuye**. Las variables de calendario no aportan poder explicativo suficiente para justificar los tres parámetros adicionales. El modelo principal es preferible por parsimonia.

> **Nota:** el texto de cierre de la notebook indica que el R² ajustado aumenta al incluir las variables de tiempo. La tabla de resultados muestra lo contrario; la interpretación correcta es la descrita aquí.

#### Coeficientes e inferencia

La inferencia se realizó con MCO (`statsmodels`) sobre las **160 observaciones de entrenamiento**.

| Predictor | Coeficiente | Error estándar | t | p-valor | IC 95 % | Decisión (α = 0.05) |
|---|---:|---:|---:|---:|---|---|
| Intercepto | 3.041665 | 0.630098 | 4.827 | 3.32 × 10⁻⁶ | [1.7969, 4.2865] | Significativo |
| Spend Channel 1 | 0.047208 | 0.001600 | 29.500 | 5.12 × 10⁻⁶⁵ | [0.0440, 0.0504] | Significativo positivo |
| Spend Channel 2 | 0.179978 | 0.010026 | 17.951 | 1.72 × 10⁻³⁹ | [0.1602, 0.1998] | Significativo positivo |
| Spend Competitors | −0.001043 | 0.006690 | −0.156 | 0.8763 | [−0.0143, 0.0122] | No significativo |
| YEAR | −0.048321 | 0.154190 | −0.313 | 0.7544 | [−0.3529, 0.2563] | No significativo |
| MONTH | −0.006206 | 0.106838 | −0.058 | 0.9538 | [−0.2173, 0.2049] | No significativo |
| WN | 0.002387 | 0.024310 | 0.098 | 0.9219 | [−0.0456, 0.0504] | No significativo |

**Prueba F global:** `F = 220.278`, `p = 1.297 × 10⁻⁷²`. Se rechaza la hipótesis nula de que todos los coeficientes sean cero; el modelo es significativo en conjunto.

#### Interpretación

- Un aumento de una unidad en **Spend Channel 1** se asocia con **+0.0472 conversiones**, manteniendo constante el resto de variables.
- Un aumento de una unidad en **Spend Channel 2** se asocia con **+0.1800 conversiones**, también con el resto constante.
- El canal 2 tiene un mayor efecto marginal por unidad invertida; el canal 1 concentra un volumen de inversión mucho mayor.
- Los intervalos de confianza de ambos canales excluyen el cero.
- **Spend Competitors**, **YEAR**, **MONTH** y **WN** no muestran evidencia de asociación lineal individual tras controlar por los demás predictores. Esto no prueba que su efecto sea exactamente cero.
- El intercepto es significativo, pero su interpretación práctica es limitada porque el escenario de todos los predictores en cero está fuera del rango observado.

**Diagnóstico del modelo:** Durbin-Watson = 2.119; Jarque-Bera = 129.010 con `p = 9.68 × 10⁻²⁹`; asimetría de residuos = −1.391; curtosis = 6.408; condición = 811.

---

### 14 — Comparación de modelos

**Notebook:** `notebooks/14_model_comparison.ipynb`

Se evaluaron **40 algoritmos de regresión** con `LazyPredict`, usando las seis variables disponibles (`SpendChannel1`, `SpendChannel2`, `SpendCompetitors`, `YEAR`, `MONTH`, `WN`) y la misma partición temporal 80/20.

#### Mejores resultados en el conjunto de prueba

| # | Modelo | R² | R² ajustado | RMSE |
|---:|---|---:|---:|---:|
| 1 | GradientBoostingRegressor | **0.9881** | **0.9860** | **0.5633** |
| 2 | ExtraTreesRegressor | 0.9858 | 0.9832 | 0.6168 |
| 3 | HistGradientBoostingRegressor | 0.9768 | 0.9726 | 0.7863 |
| 4 | RandomForestRegressor | 0.9762 | 0.9718 | 0.7979 |
| 5 | ExtraTreeRegressor | 0.9740 | 0.9693 | 0.8331 |
| 6 | BaggingRegressor | 0.9689 | 0.9632 | 0.9114 |
| 7 | DecisionTreeRegressor | 0.9560 | 0.9481 | 1.0834 |
| 8 | AdaBoostRegressor | 0.9458 | 0.9360 | 1.2029 |
| 9 | PoissonRegressor | 0.9370 | 0.9255 | 1.2974 |
| 10 | LinearSVR | 0.9017 | 0.8839 | 1.6198 |

Referencias:

| Modelo | R² | R² ajustado | RMSE |
|---|---:|---:|---:|
| LinearRegression | 0.8967 | 0.8780 | 1.6606 |
| DummyRegressor | −0.0016 | −0.1838 | 5.1718 |

Los modelos de ensamble basados en árboles superan claramente a la regresión lineal. Sin embargo:

- Se usaron **hiperparámetros por defecto**, sin búsqueda ni validación cruzada temporal.
- El conjunto de prueba tiene solo **40 observaciones**, por lo que las diferencias entre los primeros puestos pueden ser inestables.
- Los modelos de árboles no extrapolan bien fuera del rango de inversión observado, una limitación relevante si se usan para planificar presupuestos.
- Un R² tan alto debe validarse con ventanas temporales adicionales antes de adoptar el modelo.

---

## Conclusiones

1. **Los datos son consistentes:** 200 semanas consecutivas, sin valores perdidos ni semanas duplicadas dentro del periodo observado.
2. **Las dos variables de inversión propias son los predictores relevantes.** `Spend Channel 1` (ρ = 0.801) y `Spend Channel 2` (ρ = 0.554) son las que mejor se asocian con las conversiones; ambas resultan significativas en la regresión.
3. **La inversión de competidores no muestra un efecto lineal significativo** tras controlar por los dos canales. Su correlación descriptiva con las conversiones (ρ = 0.195) es débil y puede estar influida por su relación con el canal 2.
4. **Las variables de calendario no mejoran el modelo.** `MONTH` y `WN` están fuertemente correlacionadas (ρ = 0.941) y, junto con `YEAR`, no son significativas. El R² ajustado baja de 0.8868 a 0.8780 al incluirlas.
5. **El modelo lineal principal es la mejor opción para interpretación:** explica cerca del **89 %** de la variabilidad en prueba y permite cuantificar el efecto marginal de cada canal.
6. **Los modelos de ensamble son la mejor opción predictiva**, con R² de hasta **0.988**, pero su ventaja debe confirmarse con validación temporal, ajuste de hiperparámetros y análisis de estabilidad.
7. **El periodo inicial vacío no debe imputarse.** Las 9 semanas sin datos son anteriores al inicio del estudio y no forman parte del periodo observado.

---

## Limitaciones

- **Tamaño muestral:** 200 observaciones y solo 40 en prueba. Las métricas pueden variar con otra partición.
- **Una sola partición temporal:** no se aplicó validación cruzada de series temporales ni backtesting con múltiples ventanas.
- **Supuestos de la inferencia:** los residuales del modelo no son normales (Jarque-Bera muy significativo), lo que afecta la validez de los p-values e intervalos en muestras pequeñas. No se reportaron pruebas formales de heterocedasticidad ni autocorrelación.
- **Errores estándar no robustos:** la inferencia usa la covarianza no robusta de `statsmodels`.
- **Multicolinealidad:** `MONTH` y `WN` tienen una correlación de 0.941. Aunque el número de condición (811) no es extremo, los coeficientes de tiempo son difíciles de interpretar por separado.
- **Codificación ordinal del año:** `YEAR` se codifica como 1–4, lo que impone una tendencia lineal. Una codificación categórica o una tendencia temporal explícita podría ser más adecuada.
- **Comparación de modelos:** `LazyPredict` usa hiperparámetros por defecto y no optimiza para series temporales. Los R² de los ensambles pueden ser optimistas.
- **Causalidad:** las asociaciones estimadas son correlacionales. La inversión no se asignó experimentalmente, por lo que no debe interpretarse como efecto causal sin supuestos adicionales.
- **Medición de competidores:** `Spend Competitors` es una estimación, por lo que puede contener error de medición.
- **Rutas locales:** las notebooks fijan la raíz con `os.chdir(...)` a una ruta de Windows; deben ajustarse si se ejecutan en otro equipo.

---

## Reproducibilidad

- **Python 3.13** (probado con 3.13.15).
- Dependencias principales: `pandas`, `numpy`, `siuba`, `seaborn`, `matplotlib`, `scipy`, `scikit-learn`, `statsmodels`, `lazypredict` e `ipywidgets`.

```bash
# Crear y activar el entorno virtual
python -m venv .venv
.venv\Scripts\activate        # Windows
# source .venv/bin/activate   # macOS / Linux

# Instalar dependencias
pip install -r requirements.txt
```

### Orden de ejecución

1. `notebooks/11_EDA_general.ipynb` — genera `data/processed/data_general_eval.csv`.
2. `notebooks/12_EDA_estructura.ipynb` — construye el calendario y revisa la cobertura temporal.
3. `notebooks/13_model_regression.ipynb` — ajusta los modelos lineales y realiza la inferencia.
4. `notebooks/14_model_comparison.ipynb` — compara 40 algoritmos con `LazyPredict`.

```bash
jupyter lab
```

> Los archivos de `data/raw/` y `data/processed/` están excluidos del control de versiones mediante `.gitignore`. Para reproducir el análisis, coloca `raw_data.csv` en `data/raw/` y ejecuta la notebook 11 para generar el dataset procesado.

---

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
│   ├── 13_model_regression.ipynb     # Regresión lineal e inferencia
│   └── 14_model_comparison.ipynb     # Comparación de 40 modelos
├── .gitignore
├── requirements.txt
└── README.md
```
