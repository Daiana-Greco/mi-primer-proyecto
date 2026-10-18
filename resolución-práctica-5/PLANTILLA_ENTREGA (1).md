# Entrega práctica 5 — MLOps con MLflow

- **Nombre:**Daiana Greco
- **student_id:**DaiGreco
- **Escala:**Small

Respondé cada pregunta en una a tres oraciones. Cuando se pide un dato, copiá el valor que te dieron tus notebooks.

## Mis runs

Pegá acá la tabla que imprime `03_comparar_y_elegir` (sección 5) e indicá cuál quedó registrado como `champion`.
| run | modelo | features | class_weight | umbral | hiperparametros | train_pr_auc | pr_auc | accuracy | precision | recall | f1 |
|---|---|---|---|---|---|---|---|---|---|---|---|
| max_depth=8 | random_forest | completas | ninguno | 0.4 | n_estimators=100, max_depth=8 | 0.643 | 0.568 | 0.918 | 0.713 | 0.343 | 0.464 |
| max_depth=8 | random_forest | completas | ninguno | 0.4 | n_estimators=100, max_depth=8 | 0.643 | 0.568 | 0.918 | 0.713 | 0.343 | 0.464 |
| max_depth=8 | random_forest | completas | ninguno | 0.4 | n_estimators=100, max_depth=8 | 0.643 | 0.568 | 0.918 | 0.713 | 0.343 | 0.464 |
| random_forest-completas-u0.4 | random_forest | completas | ninguno | 0.4 | n_estimators=100, max_depth=8 | 0.643 | 0.568 | 0.918 | 0.713 | 0.343 | 0.464 |
| max_depth=8 | random_forest | completas | ninguno | 0.5 | n_estimators=100, max_depth=8 | 0.643 | 0.568 | 0.914 | 0.799 | 0.226 | 0.352 |
| random_forest-completas-u0.5 | random_forest | completas | ninguno | 0.5 | n_estimators=100, max_depth=8 | 0.643 | 0.568 | 0.914 | 0.799 | 0.226 | 0.352 |
| max_depth=8 | random_forest | completas | ninguno | 0.5 | n_estimators=100, max_depth=8 | 0.643 | 0.568 | 0.914 | 0.799 | 0.226 | 0.352 |
| random_forest-completas-u0.5 | random_forest | completas | ninguno | 0.5 | n_estimators=100, max_depth=8 | 0.643 | 0.568 | 0.914 | 0.799 | 0.226 | 0.352 |
| max_depth=8 | random_forest | completas | ninguno | 0.4 | n_estimators=200, max_depth=8 | 0.644 | 0.564 | 0.919 | 0.724 | 0.347 | 0.469 |
| run4 | random_forest | completas | ninguno | 0.4 | n_estimators=200, max_depth=8 | 0.644 | 0.564 | 0.919 | 0.724 | 0.347 | 0.469 |
| max_depth=8 | random_forest | completas | ninguno | 0.4 | n_estimators=200, max_depth=8 | 0.644 | 0.564 | 0.919 | 0.724 | 0.347 | 0.469 |
| max_depth=8 | random_forest | completas | ninguno | 0.4 | n_estimators=200, max_depth=8 | 0.644 | 0.564 | 0.919 | 0.724 | 0.347 | 0.469 |
| max_depth=8 | random_forest | completas | ninguno | 0.4 | n_estimators=200, max_depth=8 | 0.644 | 0.564 | 0.919 | 0.724 | 0.347 | 0.469 |
| max_depth=8 | random_forest | completas | balanced | 0.4 | n_estimators=200, max_depth=8 | 0.632 | 0.553 | 0.723 | 0.255 | 0.868 | 0.394 |
| run5 | random_forest | completas | balanced | 0.4 | n_estimators=200, max_depth=8 | 0.632 | 0.553 | 0.723 | 0.255 | 0.868 | 0.394 |
**Run registrado:**

## Datos y tracking

**1.** ¿Cuántas filas tiene cada split de `ml_features` y qué porcentaje de contracargos hay en cada uno? ¿Por qué se separa por fecha y no al azar?

Train: Tiene 35,654 filas.

Test: Tiene 7,129 filas, con un total de 740 contracargos, lo que equivale al 10.4% del total de filas del conjunto de test.

Motivo de la separación por fecha:

La separación temporal evita el solapamiento de datos futuros con pasados (data leakage o fuga de información), permitiendo evaluar el desempeño del modelo simulando escenarios reales de predicción hacia el futuro.

**2.** ¿Qué accuracy obtiene un "modelo" que siempre dice que no hay contracargo? ¿Por qué la práctica usa el PR-AUC como métrica principal?

Accuracy del modelo ingenuo:Obtiene una accuracy del 89.6%. Sin embargo, esta métrica es totalmente engañosa pues se obtiene sin detectar ni un solo contracargo.Por qué se usa el PR-AUC como métrica principal:Con clases desbalanceadas la accuracy engaña. Se usa el PR-AUC (área bajo la curva precision-recall) porque resume qué tan bien ordena el modelo las transacciones de más a menos riesgosas, sin depender de un umbral arbitrario de clasificación.

**3.** ¿Qué quedó registrado en el run `01_manual_logistica` como parámetros, métricas y artefactos? ¿Qué diferencia hay entre un experimento y un run?

Registros de 01_manual_logistica:

Parámetros: C ('1.0'), class_weight ('ninguno'), data_table ('ml_features'), data_version ('0'), feature_set ('basicas'), model_type ('logistic_regression') y threshold ('0.5').

Métricas: test_accuracy (0.896), test_alertas (67.0), test_f1 (0.082), test_pr_auc (0.339), test_precision (0.493), test_recall (0.045), test_roc_auc (0.794) y train_pr_auc (0.341).

Artefactos: La imagen de la matriz de confusión ['matriz_confusion.png'] y el modelo serializado junto con su descripción/firma (MLmodel y entorno de Python).

Diferencia entre un experimento y un run:

Un experimento es la entidad organizativa contenedora que agrupa múltiples pruebas o ejecuciones.

Un run representa un entrenamiento/ejecución individual dentro de un experimento, en el cual se registran sus parámetros, métricas, artefactos y tags específicos.

**4.** ¿Cuántos parámetros registró autolog y qué métricas? ¿Qué no registra autolog que sí registramos a mano?

Cantidad de parámetros registrados por autolog:

Registró 46 parámetros (todos los parámetros por defecto e hiperparámetros del Pipeline de scikit-learn).

Métricas registradas por autolog:

Registró únicamente métricas de entrenamiento (training_...): training_accuracy_score (0.91), training_f1_score (0.887), training_log_loss (0.229), training_precision_score (0.901), training_recall_score (0.91), training_roc_auc (0.899) y training_score (0.91).

Lo que NO registra autolog que sí se registra a mano:

Autolog no sabe cuál es nuestro conjunto de test ni registra las métricas sobre datos no vistos/evaluación externa (como test_pr_auc, test_precision, test_recall, test_alertas, etc.), ni tampoco parámetros propios del negocio/problema (como el umbral de decisión configurado o la versión de los datos Delta de origen).

**5.** ¿Dieron las mismas métricas `01_repro_a` y `01_repro_b`? Nombrá las cuatro cosas que hay que fijar para que un run sea reproducible.

Comparación de métricas:

Sí, dieron exactamente las mismas métricas (¿Métricas idénticas? True implícito en los resultados): ambas obtuvieron test_pr_auc = 0.5099, test_roc_auc = 0.8693, test_accuracy = 0.9094, test_precision = 0.7217, test_recall = 0.2068, test_f1 = 0.3214, test_alertas = 212 y train_pr_auc = 0.5941.

Las cuatro cosas que hay que fijar para garantizar reproducibilidad:

Los mismos datos (registrado mediante data_version de la tabla Delta).

El mismo código.

La misma semilla (random_state=42).

El mismo entorno (el archivo requirements.txt / dependencias).

**6.** ¿Qué pasó al llamar al modelo sin la columna `amount`? ¿Para qué sirve la firma (*signature*) del modelo?

Qué pasó al llamar al modelo sin amount:

Lanzó una excepción o error de validación de esquema de datos esperado: Failed to enforce schema of data....

Para qué sirve la firma (signature) del modelo:

Declara explícitamente cuáles son las columnas y tipos de datos requeridos de entrada (inputs), así como el tipo de salida (outputs) que produce el modelo. Funciona como un contrato de interfaz para que MLflow frena y valide los datos recibidos antes de realizar cualquier predicción si estos no cumplen con el esquema definido.

## Tus experimentos

**7.** Mejor run: max_depth=8

Modelo: random_forest (Bosque Aleatorio).

Set de variables: completas.

Hiperparámetros: n_estimators=100, max_depth=8, class_weight=ninguno.

PR-AUC de Test: 0.568 (con un train_pr_auc de 0.643).

Mejora respecto a rule_alert / solo_regla:
Un modelo de referencia basado únicamente en la regla o al azar tiene un PR-AUC de aproximadamente 0.104 (que coincide con la tasa base de chargebacks en test, como lo indica la salida en la celda 1 del notebook: "un modelo al azar tiene PR-AUC ≈ 0.104"). Por lo tanto, el mejor modelo mejoró en +0.464 puntos absolutos en PR-AUC (o más de 5.4 veces el rendimiento de la regla sola).

**8.** Evolución del PR-AUC (Random Forest, max_depth=8):

Con variables basicas: El modelo de regresión logística obtuvo 0.339. (Para Random Forest con básicas no hay registro directo en la tabla, pero se observa el salto cualitativo al agregar información).

Con variables cliente: El PR-AUC sube a 0.510 (ver runs 01_repro_a / 01_repro_b).

Con variables completas: El PR-AUC alcanza su máximo en 0.568.

Cambio neto: Pasar de cliente a completas sumó +0.058 puntos de PR-AUC, confirmando que la incorporación paulatina de features enriquece la capacidad de discriminación del modelo.

Variables más importantes:
Las variables de mayor peso en los árboles con el set completas corresponden a:

Agregados transaccionales del cliente: Métricas históricas de comportamiento (monto acumulado en las últimas horas/días, frecuencia de transacciones).

Combinación de canal y horario: Indicadores de transacciones nocturnas o fuera de la rutina habitual del cliente.

Monto de la transacción en relación con el promedio histórico del usuario.

**9.** Modelo con mejor desempeño:El Random Forest (Bosque) anduvo claramente mejor (PR-AUC = 0.568) que la Regresión Logística (PR-AUC = 0.339).¿Por qué los árboles aprovechan mejor las combinaciones de variables?Regresión Logística: Es un modelo lineal. Asume que cada variable contribuye de forma independiente y additiva a la probabilidad final, a menos que el analista cree e ingrese manualmente términos de interacción ($X_1 \times X_2$).Modelos basados en Árboles (Random Forest / Boosting): Realizan cortes sucesivos en el espacio de características. Un corte inicial en una rama puede evaluar si la transacción es nocturna, y el siguiente nodo hijo puede evaluar si es por transferencia. Esto les permite capturar interacciones no lineales complejas entre múltiples variables de forma automática y nativa sin necesidad de reingeniería manual de atributos.

**10.** Al analizar la corrida max_depth=8 con n_estimators=200 al pasar de class_weight = ninguno a balanced (con umbral $0.4$):Precision: Cayó drásticamente de 0.724 a 0.255.Recall: Aumentó significativamente de 0.347 a 0.868.Cantidad de alertas: Aumentó de forma masiva (debido al alto recall y la baja precisión, el modelo predice muchas más transacciones como fraudulentas).Impacto en PR-AUC: El PR-AUC prácticamente no cambió (pasó de 0.564 a 0.553, una variación menor a 0.011).Razón: El PR-AUC evalúa la calidad del ordenamiento (ranking) de las probabilidades predichas a lo largo de todos los umbrales posibles. Usar balanced escala/desplaza las probabilidades hacia arriba, modificando el comportamiento en un umbral fijo (como $0.4$), pero altera muy poco la curva PR completa.

**11.** Efecto de bajar el umbral (por ejemplo, de 0.5 a 0.4 en max_depth=8):La Precision disminuye (de 0.799 a 0.713).El Recall aumenta (de 0.226 a 0.343).El F1-Score mejora de 0.352 a 0.464 al lograr un mejor balance operacional.¿Por qué el PR-AUC es independiente del umbral mientras que F1 no?F1-Score: Se calcula a partir de la matriz de confusión generada por un único umbral operativo de corte fijado previamente. Si cambias el umbral, cambian la Precisión y el Recall puntuales, y por ende cambia $F1$.PR-AUC (Area Under the Precision-Recall Curve): Es una métrica global que integra el área bajo la curva calculada evaluando la Precisión y el Recall para todos los umbrales posibles entre 0 y 1. Por definición, mide la capacidad intrínseca del modelo para clasificar/ordenar el riesgo, independientemente del corte que elijas después.

**12.** Hiperparámetro barrido: max_depth (Profundidad máxima del árbol), evaluando valores como 3, 5, 8 y sin_limite.

Mejor valor para Test: max_depth = 8, obteniendo el mejor PR-AUC de test (0.568).

Valores con train_pr_auc mucho más alto que test:

Para max_depth = sin_limite, el train_pr_auc alcanzó 1.0 (100%), mientras que en test el PR-AUC cayó a 0.544 (o 0.537 en 100 estimadores).

Nombre del fenómeno: Overfitting (Sobreajuste). Ocurre cuando el modelo memoriza el ruido y los detalles específicos del conjunto de entrenamiento al tener una complejidad ilimitada, perdiendo capacidad de generalización ante datos no vistos (test).

## Comparar y elegir

**13.** No coincidieron. El mejor por PR-AUC fue con n_estimators=100 (PR-AUC 0.568, Accuracy 0.918), mientras que el mejor por accuracy fue con n_estimators=200 (PR-AUC 0.564, Accuracy 0.919). No coinciden debido al desbalance de clases (poca cantidad de casos positivos/fraudes).

**14.** Quedan más lejos de la diagonal los runs con max_depth=sin_limite (donde el PR-AUC en train alcanza 1.0 mientras que en test cae a ~0.537 - 0.544). Esto se debe a overfeating.

**15.** Permite consultar, ordenar, filtrar y procesar los experimentos de forma programática como una tabla (Dataframe), facilitando la automatización de análisis y la comparación masiva cuando hay muchos runs

**16.** Cuesta más un contracargo no detectado que una alerta falsa.

## Registro, uso y monitoreo

**17.** Registro en Unity Catalog: Sí, se pudo registrar exitosamente.

Nombre: workspace.bigdata_daigreco.fraud_model

Versión: 1

Alias: 'champion'

**18.** Se puntuaron 7.165 transacciones nuevas, de las cuales 347 generaron alerta (4.8%).

Columnas de ml_predictions para auditoría: model_name, model_version, run_id, threshold y scored_at (junto con score y alerta).

El umbral es una decisión de negocio que puede sintonizarse o ajustarse según requemientos del área consumidora sin necesidad de volver a entrenar el modelo.

**19.** Sí, fue similar (0.554 frente a 0.568 obtenido en test durante el entrenamiento).

Porcentaje de alertas en la campaña simulada: Aumentó significativamente, pasando de 4.8% a 11.3%.

El drift es el cambio o desplazamiento en la distribución de las variables de entrada o en la probabilidad/score predicha por el modelo a lo largo del tiempo. Se puede detectar antes de contar con las etiquetas reales monitoreando y comparando directamente las distribuciones de las variables de entrada (ej. montos, medio de pago) y la salida de los scores generados.

## Cierre

**20.** Registro del modelo en el Model Registry de Unity Catalog y asignación del alias champion.

Despliegue: Inferencia en batch para puntuar el conjunto de transacciones nuevas y persistencia en la tabla ml_predictions.

Monitoreo: Comparación del desempeño del modelo ante la llegada de etiquetas reales (contracargos) e identificación de drift en distribuciones de entradas y scores.

Faltantes para producción real (al menos dos):

Implementar inferencia en tiempo real o en streaming de baja latencia mediante Serving Endpoints o Spark Streaming (según el caso de uso).

Configurar alertas automatizadas y pipelines de reentrenamiento periódico/automático (CI/CD) ante la detección de drift o caída en métricas de negocio.
