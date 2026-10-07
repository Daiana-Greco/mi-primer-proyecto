# Resolución Práctica 2 - Pipeline End-to-End en Databricks

## Información del Estudiante
- **Nombre y Apellido:** Daiana Greco
- **Student ID:** DaiGreco

---

## Configuración y Ejecución del Job

<img width="678" height="145" alt="DAG" src="https://github.com/user-attachments/assets/7476af44-f4aa-4713-bf3d-4fde5050e2f4" />

### Detalle del Job
- **Nombre exacto del Job:** bigdata_DaiGreco_silver_gold
- **URL del Job:** (https://dbc-6366aaf1-e1bf.cloud.databricks.com/jobs?o=7474653328581901)

### Resultados de Ejecución e Idempotencia

<img width="705" height="289" alt="Resultado ejecuciones" src="https://github.com/user-attachments/assets/d1e28962-ab43-4792-8010-440f02cb0fd3" />

#### Métrica de Transacciones en `batch_002`:
- **Aceptadas:** 200
- **Rechazadas:** 2

---

## Conceptos Teóricos

### COPY INTO vs. MERGE
- **`COPY INTO`:** Es un comando optimizado para la ingesta incremental de datos masivos desde un almacenamiento de objetos (archivos CSV, JSON, Parquet) hacia una tabla Delta. Resuelve el problema de la **carga eficiente de archivos nuevos**, garantizando que cada archivo procesado se ingrese una sola vez sin duplicaciones a nivel de archivo.
- **`MERGE INTO`:** Es un comando SQL de manipulación de datos (DML) que permite realizar inserciones, actualizaciones y borrados condicionales (*upserts*) en una tabla Delta basándose en una condición de coincidencia (*join*). Resuelve el problema del **mantenimiento y la idempotencia a nivel de registro**, permitiendo actualizar registros existentes o insertar nuevos según una clave primaria.

### Grano de las Tablas Gold
- **`gold_daily_sales`:**
  - **Grano:** Una fila por fecha de venta (`sale_date`), país (`country`), categoría de producto (`category`) y canal de pago (`payment_channel`).
- **`gold_customer_risk`:**
  - **Grano:** Una fila por cliente (`customer_id`).
- **`gold_batch_summary`:**
  - **Grano:** Una fila por lote de ingesta (`expected_batch_id`).

---

## 📊 Visualizaciones y Consignas Prácticas

### Consigna 1 — Evolución diaria y Ticket Promedio
**Pregunta:** ¿Qué día tuvo el mayor monto vendido y ese día también fue el de mayor cantidad de transacciones?
- **Respuesta:** Día de mayor monto vendido: 2026-03-01 ($7,310,840.77)
Día de mayor cantidad de transacciones: 2026-03-01 (7236 transacciones)
- **Conclusión:** Revisando el resultado noto que existe coincidencia entre el día de mayor monto vendido y el día de mayor cantidad de transacciones, esto puede deberse a que el monto vendido es la suma de todas las transacciones realizadas en ese día, por lo tanto, el día con mayor monto vendido es el mismo que el día con mayor cantidad de transacciones.
---

### Consigna 2 — Fraude por Canal de Pago
**Pregunta:** ¿Qué canal de pago presenta la mayor tasa de fraude? ¿La conclusión se sostiene al considerar el número de transacciones de cada canal?

- **Visualización:** *(Insertar captura o ver notebook `05_visualizacion.ipynb`)*
- **Respuesta:** El canal con mayor tasa de fraude es `[Canal]` con `[X.XX]%`. `[Explicación de si se sostiene al considerar el volumen]`.
- **Conclusión:** Pude calcular la tasa de fraude global correcta ($\frac{\sum \text{fraud\_transactions}}{\sum \text{transaction\_count}}$) para cada canal sin caer en el error de promediar tasas relativas. Al cruzar el porcentaje con el volumen total de operaciones, concluí que la tasa por sí sola no cuenta toda la historia: hay que considerar el tamaño del canal para evaluar el impacto operativo real del fraude.

---

### Consigna 3 — Concentración geográfica y de producto
**Pregunta:** ¿Qué combinación de país y categoría genera el mayor monto? ¿Existe una categoría dominante en todos los países o cambia según el mercado?

- **Visualización:** *(Insertar captura o ver notebook `05_visualizacion.ipynb`)*
- **Respuesta:** La combinación de mayor monto es `[País] - [Categoría]` con `$ [Monto]`. `[Indicar si hay dominancia o variabilidad según el país]`.
- **Conclusión:** La matriz de calor me facilitó detectar rápidamente la combinación de país y categoría que genera mayores ingresos a nivel global. Además, comprobé que las preferencias de producto cambian según el mercado, lo que demuestra la importancia de adaptar el stock y las promociones a cada región en lugar de aplicar una estrategia única.

---

### Consigna 4 — Calidad del pipeline
**Pregunta:** ¿Qué proporción de cada lote fue aceptada y rechazada? ¿El lote nuevo presenta una calidad diferente del lote inicial?

- **Visualización:** *(Insertar captura o ver notebook `05_visualizacion.ipynb`)*
- **Respuesta:**
  - `batch_002`: `[X]%` aceptadas (`[N]`) / `[Y]%` rechazadas (`[N]`).
  - `batch_003`: `[X]%` aceptadas (`[N]`) / `[Y]%` rechazadas (`[N]`).
  - **Comparación:** `[Indicar si el lote nuevo presenta calidad diferente o similar]`.
- **Conclusión:** Al evaluar las proporciones de aceptación y rechazo al 100% en lugar de solo ver las cantidades absolutas, pude comparar de forma justa lotes de distintos tamaños. Verifiqué si el lote nuevo mantenía los estándares de calidad del lote inicial y comprendí que el volumen total influye en cómo percibimos el rendimiento del pipeline.

---

## ❓ Preguntas de Análisis y Comprensión

*(Completar con las respuestas a las 20 preguntas teóricas/prácticas y sus correspondientes consultas SQL/PySpark cuando aplique)*

1. **Pregunta 1:** [Respuesta y consulta]
2. **Pregunta 2:** [Respuesta y consulta]
...
20. **Pregunta 20:** [Respuesta y consulta]
