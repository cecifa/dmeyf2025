# DMEyF 2025 — Minería de Datos y Aprendizaje Automático (UBA)

Repositorio de trabajo para la materia **Data Mining, Estadística y Aprendizaje Automático** de la **Maestría en Explotación de Datos y Descubrimiento del Conocimiento — UBA, Facultad de Ciencias Exactas y Naturales**.

> ⚠️ Este repositorio es un fork del [repo oficial de la cátedra](https://github.com/dmecoyfin/dmeyf2025). Los scripts y materiales provistos por la cátedra corresponden a sus respectivos autores. Las carpetas `competencia_1`, `competencia_2` y `competencia_3` contienen trabajo propio.

---

## 🏦 El problema

Una empresa cuenta con un segmento de clientes que poseen un producto de alta gama llamado **Paquete Premium**. Actualmente la empresa no hace campañas proactivas de retención de clientes, simplemente, una vez que el cliente manifiesta que se quiere ir, reaccionan intentando retenerlo.

El objetivo es construir un **modelo predictivo de churn** que permita identificar, con dos meses de anticipación, qué clientes tienen alta probabilidad de darse de baja, para poder realizar una **campaña de marketing de retención proactiva** antes de que lo decidan.

---

## 📊 El dataset

- Snapshots mensuales del último día de cada mes (aprox. 163.000 registros por período)
- 154 variables de actividad del cliente + variable objetivo `clase_ternaria`
- Variable target con tres clases:

| Clase | Descripción |
|---|---|
| `BAJA+1` | El cliente se da de baja el mes siguiente |
| `BAJA+2` | El cliente se da de baja en dos meses |
| `CONTINUA` | El cliente sigue activo luego del mes+2 |

---

## 💰 Función de ganancia

La métrica de evaluación es **ganancia real** de la campaña:

```
Ganancia = $780.000 × BAJA+2 - $20.000 × (BAJA+1 + CONTINUA)
```

- Costo del estímulo: $20.000 por envío
- Ganancia por cliente retenido: $1.600.000 (con efectividad del 50% → $800.000 esperados)
- **Umbral óptimo de corte**: prob(BAJA+2) ≥ 0.025

---

## 🗂 Estructura del repositorio

```
dmeyf2025/
├── competencia-01/     ← Entrega 1: trabajo propio
│   └── README.md
├── competencia-02/     ← Entrega 2: trabajo propio
│   └── README.md
├── competencia-03/     ← Entrega 3: trabajo propio
│   └── README.md
└── [resto de carpetas]  ← Material de la cátedra
```

---

## 🛠 Tecnologías utilizadas

- Python · R · Jupyter Notebook
- Pandas · NumPy · Scikit-learn
- Competencia 1 evaluada en plataforma Kaggle (UBA DM EyF)

---

## 📌 Competencias

| Entrega | Período objetivo | Modelo | Ganancia obtenida |
|---|---|---|---|
| Competencia 1 | 202106 | LightGBM con BO — Ensamble de 10 experimentos (5 semillas × 2 datasets) | 340.628 |
| Competencia 2 | 202108 | zlightgbm + LightGBM con BO — Ensamble ponderado de 4 modelos | 345.600 |
| Competencia 3 | 202109 | zlightgbm con 5 canaritos — Semillerio de 40 semillas | — |
