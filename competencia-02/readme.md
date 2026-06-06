# 🏦 Competencia 2 — Predicción de Churn Premium

Este sector del repositorio contiene el desarrollo, los scripts y la estrategia utilizada para la **Competencia 2** de la materia. El objetivo es predecir la deserción de los clientes con comportamiento `BAJA+2` para el período objetivo **2021-08**.

## 🛠️ Stack Tecnológico Utilizado
Esta competencia fue desarrollada íntegramente utilizando el ecosistema de **R** ejecutado sobre Google Colab:
* `data.table` para la manipulación eficiente de grandes volúmenes de datos en memoria.
* `zlightgbm` (versión extendida de LightGBM) como algoritmo principal en los modelos Línea de Muerte.
* `lightgbm` estándar para los modelos Workflow.
* `mlrMBO` + `DiceKriging` para la Optimización Bayesiana de hiperparámetros.
* `Rcpp` para el cálculo de tendencias históricas en C (máxima performance).

---

## 🚀 Estrategia de Modelado

Se corrieron cuatro modelos independientes con enfoques distintos, luego ensamblados en un paso final. El orden de ejecución es el siguiente:

---

### Modelo 1 — Línea de Muerte (10 canaritos) · `1_-_Mod_LinMue_10`
**ID de experimento:** `zmuerte-10-2`

Feature engineering y entrenamiento en un solo notebook, sin Optimización Bayesiana.

- Genera `clase_ternaria` (CONTINUA / BAJA+1 / BAJA+2) desde el dataset crudo
- Elimina las variables `mprestamos_personales` y `cprestamos_personales`
- Calcula lags de orden 1 y 2 y sus deltas para todas las variables numéricas
- Usa **`zlightgbm`** con **10 canaritos** como mecanismo de control de overfitting (columnas aleatorias que actúan como "piso": si una variable real no supera a un canarito, no se usa)
- Parámetros fijos: `num_iterations=9999`, `num_leaves=9999`, `learning_rate=1.0`, `feature_fraction=0.5`, `gradient_bound=0.1` — zLightGBM se detiene solo cuando corresponde
- Undersampling del 10% sobre la clase CONTINUA
- Entrena sobre meses 202101–202106, predice 202108

---

### Modelo 2 — Línea de Muerte (50 canaritos) · `2_-_Mod_LinMue_50`
**ID de experimento:** `zmuerte-50-2`

Idéntico al Modelo 1 en todo, con una diferencia:

- Usa **50 canaritos** en lugar de 10, lo que impone una selección de variables más estricta y reduce el riesgo de overfitting
- `num_leaves` acotado a 999 (en lugar de 9999)
- Todo lo demás (feature engineering, datos de entrenamiento, hiperparámetros) es igual al Modelo 1

---

### Modelo 3 — Workflow con Bayesiana · `3_-_Mod_WF`
**ID de experimento:** `seg-001`

Notebook de mayor complejidad, con feature engineering extendido y Optimización Bayesiana. Usa `lightgbm` estándar (no zlightgbm).

- Genera `clase_ternaria` desde el dataset crudo
- Feature engineering intra-mes: variable de estacionalidad (`kmes`), normalización de transacciones por antigüedad (`ctrx_quarter_normalizado`), ratio payroll/edad (`mpayroll_sobre_edad`)
- Lags de orden 1 y 2 y sus deltas para todas las variables numéricas
- **Tendencias históricas** de los últimos 6 meses calculadas con regresión lineal por mínimos cuadrados (implementada en C vía Rcpp): pendiente de tendencia por variable
- **Feature engineering con hojas de Random Forest**: entrena un RF de 20 árboles con 16 hojas cada uno y agrega como variables binarias la pertenencia de cada cliente a cada hoja
- Optimización Bayesiana de 30 iteraciones maximizando **ganancia en meseta** (no AUC) sobre datos de testing (202104)
- Entrenamiento final con semillerio de 30 semillas, promediando sus probabilidades
- Entrena sobre meses 201901–202104, predice 202106 *(nota: el futuro en este notebook quedó en 202106, no en 202108)*

---

### Modelo 4 — Workflow APO con Bayesiana · `4_-_Mod_WF_A`
**ID de experimento:** `apo-006`

Prácticamente idéntico al Modelo 3 en feature engineering, con diferencias en la estrategia de producción.

- Mismo pipeline de preprocesamiento que el Modelo 3 (tendencias, Random Forest hojas, intra-mes)
- También elimina `mprestamos_personales` y `cprestamos_personales`
- Optimización Bayesiana de 30 iteraciones sobre testing en 202106
- Entrenamiento final con estrategia **APO** (Anti-Pseudo-Overfitting): genera 5 grupos de 10 semillas cada uno (50 modelos en total), promedia las probabilidades dentro de cada grupo, y elige el corte óptimo mirando la ganancia promedio entre grupos — esto evita seleccionar un corte que solo funcione por azar
- Entrena sobre meses 201901–202106, predice **202108**

---

### Ensamble Final · `5_-_ENSEMBLE`
**ID de experimento:** `zensemble-202108`

Combina las predicciones de los 4 modelos anteriores con un **promedio ponderado por score de Kaggle** obtenido al subir el período 202106.

| Modelo | ID | Score Kaggle (202106) | Peso |
|---|---|---|---|
| Workflow APO | `apo-006` | 399.759 | ~26% |
| Línea de Muerte 10 | `zmuerte-10-2` | 385.599 | ~25% |
| Línea de Muerte 50 | `zmuerte-50-2` | 388.799 | ~25% |
| Workflow | `seg-001` | 399.999 | ~26% |

- Filtra las predicciones al período 202108
- Aplica el promedio ponderado de probabilidades
- Genera el archivo final de Kaggle con un **corte de 11.000 envíos**

---

## 🗂️ Estructura de Archivos

| Archivo | Qué hace |
|---|---|
| `1_-_Mod_LinMue_10` | FE + LightGBM con 10 canaritos, sin BO (exp `zmuerte-10-2`) |
| `2_-_Mod_LinMue_50` | FE + LightGBM con 50 canaritos, sin BO (exp `zmuerte-50-2`) |
| `3_-_Mod_WF` | FE extendido + tendencias + RF hojas + BO + semillerio 30 (exp `seg-001`) |
| `4_-_Mod_WF_A` | Igual al 3 + estrategia APO anti-overfitting (exp `apo-006`) |
| `5_-_ENSEMBLE` | Promedio ponderado de los 4 modelos → archivo final Kaggle |