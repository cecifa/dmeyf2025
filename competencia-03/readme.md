# 🏦 Competencia 3 — Predicción de Churn Premium

Este sector del repositorio contiene el desarrollo, los scripts y la estrategia utilizada para la **Competencia 3** de la materia. El objetivo es predecir la deserción de los clientes con comportamiento `BAJA+2` para el período objetivo **2021-09**.

## 🛠️ Stack Tecnológico Utilizado
Esta competencia fue desarrollada en **R** sobre Google Colab:
* `data.table` para la manipulación eficiente de grandes volúmenes de datos en memoria.
* `zlightgbm` (versión extendida de LightGBM) como algoritmo principal.
* `Rcpp` para el cálculo de estadísticas históricas en C (máxima performance).

---

## 🚀 Estrategia de Modelado

A diferencia de las competencias anteriores, esta entrega se resolvió con un **único notebook** que integra feature engineering, entrenamiento y scoring. No se realizó Optimización Bayesiana: los hiperparámetros fueron fijados manualmente a partir de la experiencia acumulada en competencias previas.

---

### Modelo · `apo-504-c3-202109`

#### Dataset

- Usa el dataset **unido de Competencias 2 y 3** (`competencia_02_03_unido.csv.gz`), lo que amplía significativamente el histórico disponible para entrenamiento respecto a competencias anteriores
- Genera `clase_ternaria` (CONTINUA / BAJA+1 / BAJA+2) desde el dataset crudo
- Elimina `mprestamos_personales` y `cprestamos_personales`

#### Feature Engineering

- **Intra-mes:** variable de estacionalidad (`kmes`), normalización de transacciones por antigüedad del cliente (`ctrx_quarter_normalizado`), ratio payroll/edad (`mpayroll_sobre_edad`)
- **Lags históricos:** lags de orden 1 y 2 y sus deltas para todas las variables numéricas
- **Promedios históricos** de los últimos 6 meses por variable (calculados en C vía Rcpp) — a diferencia del Modelo 3 de la Competencia 2, aquí se usa **promedio** en lugar de tendencia por pendiente

#### Entrenamiento

- **Sin Optimización Bayesiana** — hiperparámetros fijados directamente:
  - `canaritos = 5` (control de overfitting via zlightgbm)
  - `min_data_in_leaf = 200`
  - `learning_rate = 1.0`
  - `gradient_bound = 0.01`
  - `num_iterations = 9999`, `num_leaves = 9999` (zlightgbm se detiene solo)
  - `feature_fraction = 0.5`
- Undersampling del **5%** sobre la clase CONTINUA
- Entrena sobre **31 meses**: 201901–202107
- Semillerio de **40 semillas** con APO = 1 (un solo grupo, sin anti-overfitting multi-grupo)
- Predice el período **202109**

#### Clasificación

- Se genera el archivo final de Kaggle con un **corte de 11.000 envíos**, determinado en forma artesanal analizando meses anteriores
- Se exploran además cortes alternativos: 8000, 8500, 9000, 9500, 10000, 10500, 11500, 12000

---

## 🗂️ Estructura de Archivos

| Archivo | Qué hace |
|---|---|
| `apo-504-c3-202109` | Pipeline completo: FE + entrenamiento con zlightgbm (40 semillas, 5 canaritos) + predicción 202109 |

---

## 📌 Diferencias clave respecto a las competencias anteriores

| | Competencia 1 | Competencia 2 | Competencia 3 |
|---|---|---|---|
| **Dataset** | `competencia_01` | `competencia_02` | `competencia_02_03` unido |
| **Período predicho** | 202106 | 202108 | 202109 |
| **Algoritmo** | LightGBM estándar / zlightgbm | zlightgbm / LightGBM + BO | zlightgbm |
| **Optimización Bayesiana** | Sí (en modelos _2) | Sí (Workflow) | No |
| **Canaritos** | No | 10 / 50 (Línea de Muerte) | 5 |
| **Semillerio** | 5 semillas | 30–40 semillas | 40 semillas |
| **Feature histórico** | Lags + deltas | Lags + tendencias + RF hojas | Lags + promedios 6 meses |
| **Notebooks** | 4 + ensamble | 4 + ensamble | 1 notebook único |