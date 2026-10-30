# Proyecto Final: Predicción de Suscripción a Depósitos Bancarios (Bank Marketing)

**Curso:** MLOps / Big Data  
**Alumno/a:** Daiana Greco  
**Repositorio:** `BigData2026`  
**Directorio del proyecto:** `tp-final/`  
**Modalidad Seleccionada:** Variante 2 - Opción B (MLflow Experimentation & Comparison)

---

## 📄 Resumen Ejecutivo

Este proyecto aplica una metodología de MLOps de extremo a extremo en **Databricks** utilizando **PySpark ML** y **MLflow**. El objetivo de negocio consiste en predecir si un cliente suscribirá un depósito a plazo fijo (`DEPOSIT = 1`) a partir de campañas de marketing telefónico de una institución bancaria.

El flujo de trabajo incluye:
1. Análisis Exploratorio de Datos (EDA) sobre el dataset *Bank Marketing* (41.188 registros y 21 variables).
2. Preprocesamiento, codificación de variables categóricas (*One-Hot Encoding*) y partición determinística del dataset.
3. Comparación *baseline* de tres algoritmos de clasificación registrados en MLflow: **Logistic Regression**, **Random Forest** y **Gradient-Boosted Trees (GBT)**.
4. Optimización de hiperparámetros, ajuste del umbral de decisión (*threshold*) para abordar el desbalance de clases y evaluación sobre un conjunto reservado de test.
5. Registro de métricas, parámetros y artefactos de evaluación en MLflow.

---

## 🛠️ Estructura del Proyecto

```text
tp-final/
├── 01-EDA.py                  # Carga inicial, limpieza de columnas y análisis exploratorio
├── 02-PREPROCESSING.py        # Pipeline de PySpark ML, One-Hot Encoding y Split 80/20
├── 03-MLFLOW-MODELS.py        # Comparación baseline de 3 modelos en MLflow
├── 04-MODEL-COMPARISON.py     # Tuning de GBT, selección de threshold y evaluación final en Test
└── README.md                  # Documentación del proyecto final
```

---

## 📊 1. Análisis Exploratorio de Datos (EDA)

* **Dataset:** `bank-additional-full.csv`
* **Registros totales:** 41.188 observaciones.
* **Calidad de los datos:** Se normalizaron los nombres de las columnas (eliminando caracteres especiales y puntos que generan conflicto en estructuras de PySpark). No se detectaron valores nulos en el conjunto de datos.
* **Variable Objetivo (`DEPOSIT`):**
  * `0` (No contrata depósito): 88.73%
  * `1` (Contrata depósito): 11.27%
  * *Conclusión:* Existe un desbalance moderado de clases, por lo que la precisión global (Accuracy) resulta engañosa. Se priorizan métricas como **PR-AUC**, **Recall+** y **F1+** para evaluar el rendimiento.

---

## ⚙️ 2. Preprocesamiento y Feature Engineering

* **Variables Categóricas (10):** `job`, `marital`, `education`, `default`, `housing`, `loan`, `contact`, `month`, `day_of_week`, `poutcome`. Se indexaron (`StringIndexer`) y se aplicó *One-Hot Encoding* (`OneHotEncoder`).
* **Variables Numéricas (10):** Variables sociodemográficas y de campañas previas (`age`, `duration`, `campaign`, `pdays`, `previous`, `emp.var.rate`, `cons.price.idx`, `cons.conf.idx`, `euribor3m`, `nr.employed`).
* **Vector de Entrada:** Unificación mediante `VectorAssembler` a una columna vectorial `features`.
* **Estrategia de Split:** Partición determinística (80% Train / 20% Test) basada en el hash del ID de registro (`xxhash64`), garantizando la reproducibilidad del experimento.
* **Persistencia:** Guardado en tablas Delta dentro de Unity Catalog:
  * `workspace.bigdata_daigreco.tp_final_train`
  * `workspace.bigdata_daigreco.tp_final_test`

---

## 🧪 3. Experimentos Baseline (MLflow)

Se entrenaron y compararon tres modelos baseline registrados en el experimento de MLflow `/Shared/tp_final_bank_marketing_mlflow`:

| Modelo | ROC-AUC | PR-AUC | Accuracy | Precision (+) | Recall (+) | F1 (+) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Logistic Regression** | 0.7850 | 0.4320 | 0.8980 | 0.6720 | 0.2450 | 0.3590 |
| **Random Forest** | 0.7920 | 0.4480 | 0.8995 | 0.7100 | 0.2180 | 0.3330 |
| **Gradient-Boosted Trees (GBT)** | **0.8010** | **0.4650** | **0.9020** | **0.6550** | **0.3820** | **0.4820** |

* **Conclusión del Baseline:** El modelo **Gradient-Boosted Trees (GBT)** demostró el mejor desempeño en `PR-AUC` y una superior capacidad para identificar clientes que efectivamente suscriben el depósito (`Recall` del 38.2% frente al ~22% de RF). Se seleccionó GBT como el algoritmo candidato para la etapa de optimización.

---

## 📈 4. Tuning de Hiperparámetros, Threshold y Resultados Finles

### A. Tuning de GBT
Se realizó una partición de validación interna sobre Train (80/20) para evaluar 4 configuraciones de hiperparámetros. La combinación con `maxDepth=3`, `maxIter=50` y `stepSize=0.10` ofreció la mejor generalización y el valor más alto de `PR-AUC` sin sobreajuste (*overfitting*).

### B. Optimización del Umbral de Decisión (Threshold)
Analizando el rendimiento sobre el conjunto de validación se evaluaron umbrales entre 0.15 y 0.50. Se seleccionó un **threshold de 0.25**, el cual maximiza la métrica F1 para la clase positiva (`DEPOSIT = 1`).

### C. Evaluación Final sobre el Conjunto de Test Reservado
Reentrenando el modelo final con todo el conjunto de entrenamiento y aplicando el umbral optimizado (`0.25`) sobre el conjunto de prueba nunca antes visto:

* **ROC-AUC:** `0.8025`
* **PR-AUC:** `0.4682`
* **Accuracy:** `0.8910`
* **Precision (DEPOSIT=1):** `0.5240`
* **Recall (DEPOSIT=1):** `0.5890`
* **F1-Score (DEPOSIT=1):** `0.5546`

#### Matriz de Confusión en Test:
* **True Positives (TP):** Verdaderos clientes que suscribieron identificados correctamente.
* **False Positives (FP):** Clientes contactados que no suscribieron (costo de llamada).
* **False Negatives (FN):** Clientes potenciales no detectados.
* **True Negatives (TN):** Clientes que no suscribieron correctamente descartados.

---

## 📦 5. Registro de Artefactos en MLflow

En la última corrida del experimento `GBT_FINAL_BANK_MARKETING`, se registraron automáticamente como artefactos en MLflow:
1. `evaluation/threshold_comparison.csv`: Tabla con la variación de métricas según el umbral.
2. `evaluation/confusion_matrix_test.csv`: Matriz de confusión final en test.

---

## 📌 Conclusiones y Próximos Pasos

1. **Impacto de Negocio:** El ajuste del umbral de clasificación de 0.50 a 0.25 permitió elevar sustancialmente el *Recall* de la clase positiva (capturando casi un 60% de las conversiones potenciales), aceptando un incremento controlado de falsos positivos. Esto maximiza la eficiencia operacional de los equipos de venta bancarios.
2. **Limitaciones:** El estudio se acota a algoritmos tabulares nativos de PySpark.
3. **Líneas Futuras:** Incorporar validación cruzada (*Cross-Validation* k-fold), probar técnicas de remuestreo (SMOTE) para mitigar el desbalance, e integrar un *Model Registry* en Databricks para automatizar la etapa de despliegue.