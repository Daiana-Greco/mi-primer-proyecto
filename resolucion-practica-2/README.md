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
**Respuesta y conclusion** El canal de pago que presenta la mayor tasa de fraude es transfer. Revisando el resultado noto que el canal de pago con mayor tasa de fraude es transfer pese a ser el canal de pago con menor volumen total de transacciones, esto nos indica que es necesario revisar el canal de pago transfer para detectar y prevenir fraudes.Tambien podemos observar que el canal con mayor numero de transacciones es card pese a tener menor tasa de fraude.

---

### Consigna 3 — Concentración geográfica y de producto
**Pregunta:** ¿Qué combinación de país y categoría genera el mayor monto? ¿Existe una categoría dominante en todos los países o cambia según el mercado?

- **Respuesta:** La combinación con mayor monto vendido es: BR - home ($2,361,959.97)

- **Conclusión:** Observando los resultados notamos que el pais con mayor monto total vendido es Brasil - Home y la combinacion con menor monto de ventas es mexico - electronics. La categoria mas dominante es home seguido de books y de sports.

---

### Consigna 4 — Calidad del pipeline
**Pregunta:** ¿Qué proporción de cada lote fue aceptada y rechazada? ¿El lote nuevo presenta una calidad diferente del lote inicial?

- **Respuesta:**
    lote_id  total_transactions  accepted_transactions  rejected_transactions  prop_accepted  prop_rejected
batch_002                 202                    200                      2          99.01           0.99
batch_003                 203                    201                      2          99.01           0.99
  initial               49999                  49948                     51          99.90           0.10
- **Conclusión:** Observando los resultados notamos que  lotes 2 y 3 tienen la misma cantidad de rejects. Tambien observamos que el lote Inicial es el de mayor cantidad de aceptados y proporcion de aceptadados. 

---

## Preguntas de Análisis y Comprensión

Análisis de las tablas:

¿Cuántas filas físicas recibió cada lote en bronze_transactions_incremental? Escribí una consulta que muestre el resultado por source_batch_id.

source_batch_id	total_filas_fisicas
batch_002	204
batch_003	204

Consulta: SELECT 
    source_batch_id,
    COUNT(*) AS total_filas_fisicas
FROM bronze_transactions_incremental
GROUP BY source_batch_id
ORDER BY source_batch_id;

Para cada lote, ¿cuántas transacciones fueron aceptadas y cuántas quedaron en silver_transactions_quarantine? Reconciliá tus resultados con gold_batch_summary.
SELECT 
    b.source_batch_id,
    COUNT(DISTINCT s.transaction_id) AS aceptadas_silver,
    COUNT(DISTINCT q.transaction_id) AS en_cuarentena,
    g.accepted_transactions AS aceptadas_gold,
    g.rejected_transactions AS rechazadas_gold,
    g.total_amount AS monto_total_gold
FROM bronze_transactions_incremental b
LEFT JOIN silver_transactions s 
    ON b.source_batch_id = s.source_batch_id
LEFT JOIN silver_transactions_quarantine q 
    ON b.source_batch_id = q.source_batch_id
LEFT JOIN gold_batch_summary g 
    ON b.source_batch_id = g.source_batch_id
GROUP BY 
    b.source_batch_id, 
    g.accepted_transactions, 
    g.rejected_transactions,
    g.total_amount
ORDER BY b.source_batch_id;

Respuesta:
source_batch_id	aceptadas_silver	en_cuarentena	aceptadas_gold	rechazadas_gold	monto_total_gold
batch_002	200	2	200	2	144813.00
batch_003	201	2	201	2	147014.99

¿Qué motivos de rechazo aparecen en la cuarentena y cuántos registros tiene cada uno por lote? ¿Los rechazos observados coinciden con los casos introducidos por el generador?

Motivos de rechazo en la cuarentena (quality_reason):

INVALID_AMOUNT: Registros con montos nulos, negativos o fuera del formato esperado.
INVALID_DATE: Marcas de tiempo (event_ts) corruptas o con formatos de fecha inválidos.
SUSPECTED_FRAUD: Registros marcados por anomalías o parámetros sospechosos según las reglas del negocio.

Coincidencia con el generador:
Sí, coinciden plenamente. Los rechazos observados en la tabla de cuarentena responden exactamente a las reglas de calidad de datos (data quality check) aplicadas en la capa Silver sobre los casos de prueba e imperfecciones sintéticas inyectadas intencionalmente por el generador de lotes (00_generate_new_batch).

Seguí la transacción 42 desde bronze_transactions_all hasta silver_transactions. ¿Cuántas versiones existen en Bronze y cuál quedó vigente en Silver? Mostrá las columnas que justifican la elección.

Cantidad de versiones en Bronze:

Existen 2 versiones de la transacción 42 en bronze_transactions_all:
Una versión inicial proveniente del lote initial / batch_001.
Una versión posterior con la corrección histórica inyectada en el lote batch_002.

Versión vigente en Silver:
La versión que quedó vigente en silver_transactions es la proveniente del batch_002 (con el valor actualizado de amount = 1999.99).

Columnas que justifican la elección:
La selección y vigencia se justifican principalmente por las siguientes columnas aplicadas en la lógica de deduplicación y 

MERGE de Silver:
updated_at: Se selecciona el registro con el timestamp de actualización más reciente (order by updated_at desc), garantizando que la corrección pise a la versión anterior.
source_batch_id: Identifica el origen del lote (batch_002 frente a batch_001 / initial).
amount: Refleja la corrección histórica aplicada (pasando al valor de 1999.99).

Comprobá mediante una consulta que silver_transactions tiene una sola fila por transaction_id. ¿Qué resultado indicaría que la deduplicación falló?

Resultado esperado (Deduplicación exitosa): La consulta debe devolver 0 filas (un conjunto de resultados vacío). Esto confirma que no existe ningún transaction_id duplicado en la tabla.
Resultado que indica fallo: Si la consulta devuelve 1 o más filas, significa que existen transaction_id repetidos con un COUNT(*) >= 2, lo que confirma que el proceso de deduplicación o la clave primaria en el MERGE de Silver no funcionó correctamente.

Calculá la tasa de rechazo de cada lote como rechazadas / (aceptadas + rechazadas) en gold_batch_summary. ¿Es correcto comparar solamente las cantidades absolutas si los lotes tienen tamaños diferentes?

No, no es correcto. Las cantidades absolutas (por ejemplo, "10 transacciones rechazadas") no contemplan la escala ni el volumen total de cada lote.

¿Qué día presenta el mayor monto total y cuál presenta la mayor cantidad de transacciones? Consultá gold_daily_sales y explicá si ambos máximos coinciden.

Día con mayor monto total: Se obtiene identificando la fecha con el valor máximo en la columna total_amount (ordenando por esa columna de forma descendente).

Día con mayor cantidad de transacciones:Se obtiene buscando la fecha con el valor máximo en la columna total_transactions.

¿Coinciden ambos máximos? No necesariamente coinciden. La cantidad de transacciones mide únicamente el volumen de operaciones, mientras que el monto total depende del ticket promedio (monto por transacción). Un día puede registrar pocas transacciones pero de un valor pecuniario muy elevado, superando en ingresos a un día con un alto volumen de transacciones de monto bajo o de poco valor.

¿Qué canal de pago tiene la mayor tasa global de fraude? Calculala como SUM(fraud_transactions) / SUM(transaction_count) y explicá por qué no corresponde promediar directamente fraud_rate.

No corresponde promediar directamente la columna fraud_rate (AVG(fraud_rate)) porque se comete un sesgo por falta de ponderación: un promedio simple le da el mismo peso a un registro con muy pocas transacciones que a uno con miles. Para calcular la tasa global real de forma numéricamente exacta, es obligatorio sumar primero las cantidades absolutas de todos los registros (SUM(fraud_transactions) / SUM(transaction_count)).

¿Qué combinación de país y categoría concentra el mayor monto vendido? Mostrá también la combinación líder dentro de cada país.

BR home
country	category	monto_total_vendido
BR	home	2361959.97
UY	home	2323924.46
AR	books	2267722.26
MX	books	2235941.63
CL	home	2167748.03

Compará las dos primeras filas de pipeline_run_audit correspondientes a la reejecución de batch_002. ¿Qué métricas permanecen iguales y qué columna demuestra que se realizó la comparación de idempotencia?

Métricas que permanecen iguales: Las métricas de volumen y conteos del lote, como accepted_transactions, rejected_transactions, total_amount y fraud_transactions (o los conteos de filas afectadas e insertadas en los procesos de la capa Gold). Al ser un proceso determinista e idempotente, volver a procesar el mismo lote genera exactamente el mismo número de registros procesados, aceptados y rechazados.

Columna que demuestra la comparación de idempotencia: La columna job_run_id (junto con marcas de tiempo como executed_at o refreshed_at).

En 01_ingest_bronze_incremental.ipynb, ¿qué problema resuelve COPY INTO y qué información utiliza para evitar cargar dos veces el mismo archivo físico?

Problema que resuelve:Duplicación de datos e ingesta incremental eficiente: Evita la recarga duplicada de datos al reejecutar el pipeline, garantizando la idempotencia en la capa Bronze. Permite procesar únicamente los archivos nuevos que llegan al directorio incoming/transactions/ sin importar o duplicar archivos previamente ingeridos.

Información que utiliza para evitar reingresar el mismo archivo: Registro interno de metadatos de archivos (file tracking metadata): COPY INTO mantiene un registro (log) interno en la tabla Delta de destino que almacena la ruta/nombre del archivo físico (file path) y su etiqueta de modificación / versión (modification timestamp / file length). De esta manera, antes de procesar la carpeta, compara los archivos presentes contra ese log interno e ignora (skip) de forma automática cualquier CSV que ya haya sido ingresado con anterioridad.

¿Por qué la vista bronze_transactions_all usa UNION ALL en lugar de eliminar duplicados? ¿En qué capa se resuelven los duplicados de negocio y por qué?

Usar UNION ALL garantiza que todos los eventos o archivos ingresados en distintos momentos y lotes (initial, batch_002, batch_003, etc.) queden registrados tal como llegaron, sin alteración ni pérdida de trazabilidad.
Los duplicados de negocio se resuelven en la capa Silver.

Por qué en Silver: Aplicación de reglas de calidad y limpieza: Silver es la capa diseñada para limpiar, tipar, validar y conformar un modelo de datos confiable.
Estrategia de deduplicación y MERGE: Es en esta capa donde se aplica la lógica de de-duplicación (por ejemplo, utilizando funciones de ventana como ROW_NUMBER() OVER (PARTITION BY transaction_id ORDER BY updated_at DESC)) y sentencias MERGE (upsert) para garantizar que la tabla Silver conserve una única versión vigente por transaction_id.
Aislamiento de responsabilidades: Separa la responsabilidad de guardar todo sin perder datos (Bronze) de la responsabilidad de garantizar la calidad, unicidad e integridad para el negocio (Silver y Gold).

¿Por qué las transacciones iniciales reciben source_batch_id='initial' y usan event_ts como updated_at? ¿Cómo afecta eso a la corrección de la transacción 42?

En quality_rules.py, ¿qué ventaja ofrece try_cast frente a un cast convencional cuando llega un importe como N/A?

Evita que el pipeline aborte por error (Fault Tolerance): CAST('N/A' AS DECIMAL) convencional: En versiones modernas de Spark (modo ANSI activado por defecto), intentar parsear una cadena no numérica como 'N/A' lanza una excepción de análisis en tiempo de ejecución (AnalysisException / NumberFormatException), haciendo que el Job o la tarea del DAG falle por completo. try_cast('N/A' AS DECIMAL): En lugar de lanzar una excepción y romper la ejecución de la tarea, evalúa de manera segura el valor no válido y retorna NULL.

Permite el ruteo seguro a la tabla de Cuarentena: Al convertir los valores inválidos como 'N/A' en NULL sin romper el proceso, la regla de calidad en Silver puede detectar fácilmente la invalidez (por ejemplo, con WHERE amount IS NULL OR amount <= 0). Esto permite desviar de forma limpia esos registros defectuosos hacia la tabla silver_transactions_quarantine con el motivo de rechazo INVALID_AMOUNT, mientras el resto del lote continúa procesándose normalmente hacia silver_transactions.

Las reglas de calidad asignan una única quality_reason. ¿Qué sucede si un registro viola más de una regla y por qué importa el orden de las condiciones?

Se asigna únicamente la primera regla que se evalúe como verdadera: El motor de ejecución evalúa las condiciones secuencialmente de arriba abajo. En el momento en que un registro cumple la primera condición de fallo, se le asigna ese quality_reason específico y el flujo condicional finaliza para esa fila, ignorando las violaciones posteriores.

El segundo error queda oculto en la cuarentena: Aunque el registro viole múltiples controles (por ejemplo, tener una fecha corrupta y un importe negativo), la tabla silver_transactions_quarantine solo guardará el motivo correspondiente al primer filtro que saltó.

El orden de las condiciones es crítico por tres motivos principales:

Jerarquía y prioridad de negocio:Determina qué tipo de error se considera más severo o relevante para la auditoría. Si el equipo de Data Engineering considera que los errores estructurales de formato (como INVALID_DATE o INVALID_AMOUNT) son más graves que las anomalías de negocio (como SUSPECTED_FRAUD), los controles sintácticos deben colocarse en los primeros puestos del CASE WHEN.

Métricas de diagnóstico y categorización en cuarentena: El conteo total de registros agrupado por quality_reason en la capa Silver/Gold se verá sesgado hacia las reglas que se evalúen primero. Si una regla muy general se ubica al principio, absorberá la mayoría de los rechazos, "ocultando" la frecuencia real de otros errores subyacentes.

Prevención de errores de ejecución (Short-circuit Evaluation): Colocar validaciones previas de valores nulos o tipos de datos antes de aplicar transformaciones complejas o funciones matemáticas evita que el código falle o intente procesar expresiones no válidas en evaluaciones subsecuentes.

Explicá cómo se construye _record_key y cómo se usa junto con row_number. ¿Qué caso cubre el hash cuando transaction_id no puede convertirse a un número?

Interpretá las dos cláusulas principales del MERGE de silver_transactions. ¿Cuándo se actualiza una fila existente y cuándo se inserta una nueva?

¿Por qué las tablas Gold se reconstruyen completamente en esta práctica mientras Silver se actualiza con MERGE? Mencioná una ventaja y una limitación de cada estrategia.

¿Por qué expected_batch_id no participa en la detección del archivo nuevo? Indicá qué parte del pipeline descubre batch_003 y qué parte utiliza el parámetro.

Si la tarea build_silver falla, ¿qué ocurre con build_gold y validate en el Job? Explicá cómo las dependencias del DAG evitan publicar o validar resultados incompletos.
