# 🏦 Competencia 1 — Predicción de Churn Premium

Este sector del repositorio contiene el desarrollo, los scripts y la estrategia utilizada para la **Competencia 1** de la materia. El objetivo es predecir la deserción de los clientes con comportamiento `BAJA+2` para el período objetivo **2021-06**.

## 🛠️ Stack Tecnológico Utilizado
Esta competencia fue desarrollada íntegramente utilizando el ecosistema de **R** ejecutado sobre Google Colab:
* **R v3.x / v4.x** (Runtime alternativo en Colab)
* `data.table` para la manipulación eficiente de grandes volúmenes de datos en memoria.
* `lightgbm` como algoritmo de Gradient Boosting.
* `mlrMBO` + `DiceKriging` para la Optimización Bayesiana de hiperparámetros.

---

## 🚀 Estrategia de Modelado

El flujo de trabajo se dividió en dos líneas de experimentación paralelas (Modelo 1 y Modelo 2), cada una con su propia etapa de feature engineering y su propia etapa de entrenamiento. La diferencia entre ambas líneas es el tratamiento del aguinaldo en el dataset.

### Línea Modelo 1 — Sin ajuste de aguinaldo 

**Paso 1 — Feature Engineering (`Modelo_1`):**
- Genera la variable target `clase_ternaria` (CONTINUA / BAJA+1 / BAJA+2)
- Calcula lags de orden 1 y 2 (`_lag_1`, `_lag_2`) y sus deltas (`_delta_lag_1`, `_delta_lag_2`) para todas las variables numéricas
- No aplica ninguna corrección estacional
- Guarda el dataset resultante como `competencia_01_lags_OK.csv.gz`

**Paso 2 — Entrenamiento (`Modelo_1_2`):**
- Carga `competencia_01_lags_OK.csv.gz`
- Corre Optimización Bayesiana de 50 iteraciones maximizando AUC (cross-validation 5 folds)
- Entrena el modelo final con los mejores hiperparámetros encontrados
- Genera predicciones para los 5 experimentos (una por semilla: 202513, 202519, 202529, 202543, 202549)

### Línea Modelo 2 — Con ajuste de aguinaldo (experimentos 4970–4974)

**Paso 1 — Feature Engineering (`Modelo_2`):**
- Idéntico al Modelo 1, pero **antes** de calcular los lags aplica una corrección al mes futuro (202106): divide por un factor de **1.5** un conjunto seleccionado de ~100 variables monetarias y transaccionales
- Esto evita que el incremento estacional del aguinaldo (junio en Argentina) distorsione los patrones reales de churn al computar las diferencias temporales
- Guarda el dataset resultante como `competencia_01_lags_agui2.csv.gz`

**Paso 2 — Entrenamiento (`Modelo_2_2`):**
- Carga `competencia_01_lags_agui2.csv.gz`
- Mismo proceso de Optimización Bayesiana que el Modelo 1
- Los mejores hiperparámetros difieren por ser un dataset distinto (ej: `num_iterations=2037`, `num_leaves=583`, `min_data_in_leaf=1`)
- Genera predicciones para los 5 experimentos (mismas semillas)

### Optimización Bayesiana — configuración común a ambas líneas

Para ambos modelos se configuró una búsqueda bayesiana de **50 iteraciones** maximizando **AUC** con validación cruzada de 5 folds (`lgb.cv`), con undersampling del 10% sobre la clase CONTINUA. Los rangos de hiperparámetros explorados fueron:

| Hiperparámetro | Mínimo | Máximo |
|---|---|---|
| `num_iterations` | 8 | 2048 |
| `learning_rate` | 0.01 | 0.3 |
| `feature_fraction` | 0.1 | 1.0 |
| `num_leaves` | 8 | 2048 |
| `min_data_in_leaf` | 1 | 8000 |

### Ensamble Final

Para mitigar la varianza por semilla, se promedian las probabilidades de los 10 experimentos (5 del Modelo 1 + 5 del Modelo 2) y se aplica un **corte óptimo de 11.000 envíos**.

---

## 🗂️ Estructura de Archivos

Los notebooks siguen la secuencia lógica del experimento. Cada línea de modelado tiene su propio par de notebooks:

| Archivo | Qué hace | Genera |
|---|---|---|
| `Modelo_1__Agregar_lag_y_delta_lag_1_y_2_y_guardar_CSV` | Feature engineering sin ajuste de aguinaldo | `competencia_01_lags_OK.csv.gz` |
| `Modelo_1_2_-_optimiz_bayesiana_y_probar_con_cada_semilla` | Entrena LightGBM sobre el dataset del Modelo 1, corre BO y genera predicciones (exp 4950–4954) | `prediccion.txt` por experimento |
| `Modelo_2__Agregar_lag_y_delta_lag_1_y_2_y_guardar_CSV` | Feature engineering con corrección de aguinaldo (÷1.5 en mes 202106) | `competencia_01_lags_agui2.csv.gz` |
| `Modelo_2_2_-_optimiz_bayesiana_y_probar_con_cada_semilla` | Entrena LightGBM sobre el dataset del Modelo 2, corre BO y genera predicciones (exp 4970–4974) | `prediccion.txt` por experimento |
| `Ensamble` | Promedia las probabilidades de los 10 experimentos y genera el archivo final de Kaggle | `KA_ENSEMBLE_(...).csv` |

---

## 📈 Parámetros Ganadores Obtenidos

Los mejores hiperparámetros varían por semilla y por dataset. A modo de referencia, dos configuraciones representativas:

**Modelo 1** (sin ajuste de aguinaldo):
- `num_iterations`: 1337 · `learning_rate`: 0.0725 · `feature_fraction`: 0.1015 · `num_leaves`: 12 · `min_data_in_leaf`: 215

**Modelo 2** (con ajuste de aguinaldo):
- `num_iterations`: 2037 · `learning_rate`: 0.0163 · `feature_fraction`: 0.1280 · `num_leaves`: 583 · `min_data_in_leaf`: 1