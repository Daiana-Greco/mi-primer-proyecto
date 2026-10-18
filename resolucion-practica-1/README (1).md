# Clase práctica 1 — Ingesta y capa Bronze

## Propósito

Construir la primera capa de un Lakehouse en Databricks a partir de fuentes CSV, JSON y Parquet. Al terminar, cada alumno tendrá un espacio aislado en Unity Catalog y cuatro tablas Bronze trazables.

## Duración estimada

| Bloque | Minutos |
|---|---:|
| Setup y recorrido del workspace | 25 |
| Generación de fuentes | 25 |
| Lectura, esquemas y calidad inicial | 35 |
| Pausa | 10 |
| Parquet, Delta y tablas Bronze | 40 |
| Desafío de evolución de esquema | 35 |
| Puesta en común y cierre | 10 |

## Antes de la clase

1. Completar la [guía paso a paso de Databricks Free Edition](../GUIA_SETUP_DATABRICKS_FREE.md).
2. Confirmar que se puede abrir `00_setup.ipynb` y ejecutar `SELECT current_catalog()`.
3. No instalar paquetes: la práctica usa únicamente Spark y Delta incluidos en la plataforma.

## Resultados esperados

Al finalizar deben existir:

```text
<catalogo>.bigdata_<alumno>.landing
<catalogo>.bigdata_<alumno>.bronze_customers
<catalogo>.bigdata_<alumno>.bronze_products
<catalogo>.bigdata_<alumno>.bronze_transactions
<catalogo>.bigdata_<alumno>.bronze_events
```

## Formato de entrega

La entrega se hace en el repositorio personal `mi-primer-proyecto` que creaste en la [guía de Git y GitHub](../GUIA_GIT_GITHUB.md).

### Estructura

Creá un directorio `resolucion-practica-1` en la raíz del repositorio con estos archivos:

```text
mi-primer-proyecto/
└── resolucion-practica-1/
    ├── README.md
    ├── 01_ingesta_bronze.ipynb
    └── 02_desafio.ipynb (OPCIONAL)
```

| Archivo | Contenido |
|---|---|
| `README.md` | Nombre, `student_id`, escala y respuestas de la sección **Entrega breve** de `01_ingesta_bronze`: tres observaciones sobre CSV/JSON, Parquet y Delta, y dónde aparece cada una de las cinco V. Si realizaste el desafío opcional, incluí también su reflexión final (máximo 150 palabras). |
| `01_ingesta_bronze.ipynb` | Notebook ejecutado, con las salidas de las cuatro tablas Bronze y del diagnóstico de calidad. |
| `02_desafio.ipynb` (OPCIONAL) | Notebook con las tres consignas resueltas y las aserciones ejecutadas sin errores. |

### Exportar los notebooks desde Databricks

1. Ejecutá cada notebook completo para que las salidas queden visibles.
2. Abrí **File → Export → IPython Notebook** y descargá el archivo `.ipynb`.
3. Copiá los archivos descargados a `resolucion-practica-1/` dentro de tu copia local del repositorio.

### Publicar la entrega

Desde la carpeta de tu repositorio:

```bash
git add resolucion-practica-1
git commit -m "Entrega práctica 1"
git push origin main
```

Verificá en GitHub que el directorio, `README.md` y `01_ingesta_bronze.ipynb` aparecen en `main`. Incluí `02_desafio.ipynb` sólo si realizaste el desafío opcional.

El repositorio tiene que ser **público** para que el docente pueda ver la entrega. Para comprobarlo, abrí su URL en una ventana privada del navegador, sin iniciar sesión: si ves `resolucion-practica-1`, está accesible. Si lo creaste como privado, cambialo desde **Settings → General → Danger Zone → Change repository visibility**.

Enviá la URL de tu repositorio al mail de los profesores dabadie@itba.edu.ar y ghenrion@itba.edu.ar.

### Qué no incluir

El repositorio es público: cualquier persona puede ver los archivos y su historial. Revisá [qué implica que sea público](../GUIA_GIT_GITHUB.md#qué-implica-que-el-repositorio-sea-público) antes de hacer el push.

- Datos generados, archivos del volumen ni exportaciones de tablas: se reconstruyen ejecutando los notebooks.
- Tokens, contraseñas u otras credenciales.
### Respuestas de 01_ingesta_bronze
## 1. Observaciones sobre CSV/JSON, Parquet y Delta

* **CSV / JSON:** Son formatos de texto no estructurados o semi-estructurados. Tienen un rendimiento bajo en volúmenes masivos de datos porque no almacenan metadatos ni estadísticas, lo que obliga a Spark a escanear todo el archivo (Full Table Scan) y a inferir esquemas, introduciendo inestabilidad y errores de tipo de dato (como el caso de `amount` leído como `string` por valores como `"N/A"`).
* **Parquet:** Es un formato columnar comprimido y binario. Es significativamente más eficiente para consultas analíticas ya que permite el *column pruning* (leer solo las columnas necesarias) y el *predicate pushdown*, reduciendo drásticamente los tiempos de lectura e I/O en comparación con CSV/JSON.
* **Delta Lake:** Agrega una capa de almacenamiento ACID sobre Parquet mediante un log de transacciones (`_delta_log`). Permite *time travel* (historial de cambios), garantizando integridad de datos, esquemas fuertemente tipados y optimización del plan de ejecución mediante data skipping y recolección de estadísticas.

---

## 2. Las 5 V del Big Data en este caso

* **Volumen:** Representado por la escala de transacciones procesadas (decenas de miles a millones de registros) que hacen inviable el procesamiento relacional tradicional o en memoria local sin un motor distribuido como Spark.
* **Velocidad:** La necesidad de procesar e ingestar eventos de transacciones en tiempo o casi-tiempo real a medida que llegan a las capas *Bronze/Silver/Gold*.
* **Variedad:** La convivencia de datos en diversos formatos estructurados y semi-estructurados (archivos planos CSV/JSON de entrada y almacenamiento optimizado columnar Parquet/Delta).
* **Veracidad:** La calidad de los datos analizados mediante el diagnóstico (identificación de 11 IDs duplicados y 52 importes inválidos o nulos) y la necesidad de aplicar reglas de limpieza para garantizar datos confiables.
* **Valor:** La transformación de datos crudos con inconsistencias en tablas consolidadas y limpias para detección de fraudes, reportes de negocio y toma de decisiones estratégicas sobre los canales de pago.
 
### Observaciones y Diagnóstico

1. **Reconocimiento de `amount` como String:**
   Spark lee `amount` como `string` debido a que el archivo CSV contiene valores no numéricos (como el texto `"N/A"`). Al no tener un esquema definido explícitamente, asigna `string` para evitar la pérdida de datos.

2. **Inferencia de Esquema (`inferSchema=True`):**
   * **Tiempo de ejecución:** Tarda 5 segundos (proceso más lento por el pase extra sobre el archivo)[cite: 11].
   * **Resultado:** No lo soluciona, `amount` sigue siendo `string`.
   * **Motivo:** `inferSchema=True` requiere un escaneo completo de los datos (duplicando costos de I/O) y genera inestabilidad: basta un solo registro con formato no válido para forzar a Spark a interpretar toda la columna como texto.

3. **Diagnóstico de Calidad:**
   * Se identificaron **11 registros duplicados** (50.011 filas totales vs. 50.000 `transaction_id` únicos)[cite: 10].
   * Se encontraron **52 importes inválidos** que no pueden convertirse a formato numérico (`DECIMAL`)[cite: 10].

4. **Delta Lake vs. Parquet:**
   Delta Lake construye sobre Parquet agregando un log de transacciones (`_delta_log`) que proporciona:
   * Historial de cambios y auditoría (*Time Travel*).
   * Garantía de transacciones ACID y control de versiones.
   * Métricas y estadísticas avanzadas para optimizar el plan de ejecución (*Data Skipping* y *Predicate Pushdown*).