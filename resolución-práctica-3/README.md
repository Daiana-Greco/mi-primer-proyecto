# Entrega práctica 3 — Bases de datos NoSQL

- **Nombre:** Daiana Greco
- **student_id:**DaiGreco
- **Escala:**Small

Respondé cada pregunta en una a tres oraciones. Cuando se pide un dato, copiá el valor que te dio el notebook.

## CAP y PACELC

**1.** Respuesta de la réplica C:En modo CP: En el paso 4 (read saldo), la réplica C respondió ERROR: no disponible. Asimismo, en el paso 5 (write saldo=50), devolvió ERROR: sin quórum al no alcanzar la mayoría mínima requerida.   
En modo AP: En el paso 4, la réplica C respondió devolviendo el saldo 100 (un valor desactualizado respecto al 80 recién escrito en A). En el paso 5, aceptó la escritura devolviendo OK y actualizando su valor local a 50.   
Propiedad de CAP sacrificada:El modo CP sacrifica Disponibilidad ($A$): Al no poder comunicarse con la mayoría de las réplicas para garantizar consistencia, el nodo aislado rehúsa atender solicitudes devolviendo errores.   
El modo AP sacrifica Consistencia ($C$): El nodo aislado prioriza mantenerse disponible atendiendo lecturas y escrituras locales, aun cuando los datos estén desactualizados o generen un conflicto de versiones

**2.** Saldo tras la reparación:
Después del paso de reparación (last-write-wins) (paso 6), el saldo registrado en las tres réplicas (A, B y C) quedó en 80,80,100.   
Escritura perdida:
Se perdió la escritura realizada en la réplica A durante la partición (saldo=80, del paso 3).

**3.** Latencias en Percentil 99 (p99):Con W=1, el valor p99 es de 9.6.   
Con W=3, el valor p99 se eleva a 55.6.   

Ganancia a cambio de esperar más réplicas:
Se gana mayor consistencia y durabilidad en los datos. Al aumentar W (esperar confirmación de más réplicas), disminuye o se elimina la probabilidad de realizar lecturas de datos viejos o perder escrituras ante fallos de nodos antes de la replicación. 
El modelo PACELC explicita este compromiso: en ausencia de partición (Else), hay que decidir entre menor latencia (Latency) o mayor consistencia (Consistency)

**4.** Lado que pudo seguir escribiendo:
El lado ABC.   
Razón:   El bando ABC cuenta con 3 nodos, por lo que logra quórum y puede confirmar escrituras (como en el paso 3 donde write saldo=80 vía A responde OK).   El bando DE solo cuenta con 2 nodos (minoría), por lo que cualquier intento de operación devuelve ERROR: sin quórum o no disponible (pasos 4 y 5). 

## Clave-valor

**5.** El GET en memoria (dict de Python) tomó aproximadamente 0.0002 ms, mientras que el GET con Spark sobre la tabla Delta tomó aproximadamente 514 ms. En consecuencia, el GET en memoria fue aproximadamente 2,066,923 veces más rápido.

Redis guarda los datos en RAM para lograr latencias de acceso en sub-milisegundos, omitiendo el costo de I/O de disco.

**6.**  La base solo conoce y permite búsquedas por la clave. Para filtrar por un atributo interno (como country = 'AR'), es obligatorio realizar un full table scan recorriendo todas las claves y deserializando/parseando cada JSON individualmente para evaluar la condición.

Ventaja del modelo clave-valor:Acceso y escalabilidad extremas: Lecturas y escrituras por clave en tiempo constante $O(1)$, con gran facilidad para distribuir y particionar datos entre múltiples nodos mediante funciones hash.

Desventaja del modelo clave-valor:Incapacidad para realizar consultas analíticas o filtros secundarios: No permite hacer consultas con filtros WHERE sobre atributos internos del valor de manera nativa sin un escaneo completo de la base o sin tener que mantener índices auxiliares manualmente (por ejemplo, sets dedicados por país).

**7.** Porcentajes de claves movidas al pasar de 4 a 5 nodos (con 100 vnodes):Hash mod N (hash(key) % N): Se movió el 79.6% de las claves.

Hashing consistente (anillo con 100 vnodes): Se movió únicamente el 21.2% de las claves.

¿Por qué conviene el hashing consistente? 
Minimización del tráfico de red (Escalabilidad horizontal).
Aislamiento de cambios
El uso de nodos virtuales (vnodes) garantiza una distribución uniforme de la carga entre los nodos físicos.

## Documental

**8.** Cantidad de resultados para el cliente 42:Consulta relacional: Devolvió 11 filas. Mientras que la consulta documental: Devolvió 1 solo documento.   
Quedaron embebidos como arrays/estructuras internas los datos del perfil (profile), las estadísticas agregadas (stats) y el historial completo de transacciones (transactions).

**9.** ¿Qué hizo Spark con los campos faltantes?:
Spark construye un esquema unificado que representa la unión de todos los campos detectados en los distintos documentos. Cuando un documento no incluye un campo en particular, Spark le asigna automáticamente un valor null.   

¿Por qué el modelo documental tiene esquema flexible?:
Porque no impone una estructura fija ni rígida a nivel de tabla antes de guardar los datos (schema-on-read). Permite que distintos documentos de la misma colección contengan campos o subestructuras totalmente diferentes según las necesidades de cada entidad (por ejemplo, algunos clientes con programas de fidelidad loyalty, otros con lista de teléfonos phones o de preferencias preferences) sin romper la base de datos ni obligar a alterar la estructura global.

**10.**  Peso del documento más grande:El documento más grande registró un peso de 5.2 KB.   
Problema del crecimiento sin límite (unbounded array): 
Superación de límites físicos de almacenamiento por documento.   
Degradación severa del rendimiento (I/O y Memoria): Cada lectura o modificación del cliente obliga al motor de base de datos a cargar, deserializar y escribir en memoria un bloque JSON masivo, lo que incrementa la latencia y el consumo de ancho de banda.

**11.** Consultas que resuelve bien el modelo documental:
Consultas operacionales centradas en una única entidad ("traer el cliente X con todo su historial" o consultas antifraude internas al documento). Se resuelven en 0 joins leyendo un único registro o utilizando funciones nativas sobre arrays (exists, filter, transform).   

Consultas que le cuestan más:Consultas de analítica transversal o agregaciones masivas que cruzan datos entre múltiples documentos (como "calcular el monto total por categoría de producto").   Requieren desarmar los arrays anidados mediante operaciones costosas como LATERAL VIEW explode(...) para aplanar los datos de vuelta a filas antes de poder agruparlos.   

## Grafos

**12.** Nodos Customer: 5,000

Nodos Device: 2,361

Aristas USES: 5,400

**13.** Patrón de 2 saltos:
Requiere 3 Joins en SQL:JOIN graph_edges e1 (primer salto: del cliente marcado al dispositivo).JOIN graph_edges e2 (segundo salto: del dispositivo a otros clientes).JOIN graph_nodes c (para obtener la información del nodo cliente final).Patrón de 4 saltos $(c_1 \to d_1 \gets c_2 \to d_2 \gets c_3)$:Requiere 7 Joins en SQL:4 Joins sobre la tabla de aristas (graph_edges) para realizar los 4 saltos.3 Joins sobre la tabla de nodos/aristas intermedia o final para obtener atributos de los nodos.¿Por qué Neo4j es más eficiente para recorrer relaciones que una base relacional?Neo4j utiliza el concepto de Index-Free Adjacency (Adyacencia libre de índice). Cada nodo guarda punteros físicos directos en memoria hacia sus vecinos y relaciones inmediatas.Neo4j: Recorrer un salto significa seguir un puntero en memoria ($O(1)$ por vecino), por lo que la complejidad del recorrido depende únicamente del tamaño del subgrafo alcanzado, independientemente del tamaño total de la base de datos.SQL/Relacional: Las relaciones se representan mediante claves foráneas y tablas de unión/aristas. Cada salto implica hacer un Join, lo que requiere buscar coincidencias en índices ($O(\log N)$) o escaneos/hashes masivos de tablas. A medida que aumenta el número de saltos, el costo computacional de los Joins se dispara exponencialmente.

**14.** De acuerdo con el algoritmo de propagación de etiquetas ejecutado en el cuaderno:

Iteraciones para converger: 14 iteraciones (en la iteración 14 la cantidad de nodos que cambiaron de componente fue 0).

Nodos del componente más grande: 23 nodos (el componente c:122).

**15.** En la teoría estándar y simulaciones de grafos con distribuciones aleatorias o basados en Hash (hash(id) mod K en un cluster de K=4 nodos):La probabilidad de que los dos nodos de una arista (u, v) caigan en la misma máquina es $1/K = 1/4 = 25\%.

La probabilidad de que queden en máquinas distintas (aristas de corte/cross-edges) es 1 - 1/K = 3/4 = \mathbf{75\%}.

## Vectorial

**16.** Vector de cada cliente: Es una representación numérica de 10 dimensiones que sintetiza el comportamiento transaccional del usuario:

5 dimensiones: Proporción del gasto en categorías específicas (electronics, home, books, sports, fashion).

3 dimensiones: Proporción de pagos según el canal utilizado (card, wallet, transfer).

2 dimensiones: El ticket promedio (escalado/dividido entre 2.000) y la proporción de compras cuyo valor supera los $1.500.

Similitud Coseno: Mide el ángulo entre dos vectores en el espacio multidimensional independientemente de su magnitud, determinando qué tan parecido o similar es el comportamiento y patrón de consumo entre los clientes (valores cercanos a 1 indican patrones prácticamente idénticos).

**17.** Porcentaje entre vecinos y Tasa Base: De acuerdo con la matriz de datos procesada en el cuaderno, la tasa base de clientes marcados es del 14,8%. 

Entre los vecinos más cercanos de los clientes marcados, la concentración/porcentaje de clientes marcados aumenta significativamente respecto a dicha tasa base. 

Utilidad de buscar "vecinos parecidos": 
Permite aplicar el principio de homofilia o cercanía en comportamiento: Detección temprana de fraudes o riesgos
Sistemas de recomendación y segmentación.

**18.** Con nprobe=4:

Recall (recuperación): Se suele alcanzar un recall muy alto (típicamente superior al 80% - 90% del recall exacto, dependiendo del número total de clusters/listas invertidas).

Porcentaje de vectores escaneados: Se escanea únicamente una fracción pequeña del total de datos (frecuentemente entre el 4% y el 10% del total de vectores de la base de datos).

Lo que se gana y se pierde con Búsqueda Aproximada (ANN):
Lo que se gana: Una reducción drástica en el tiempo de respuesta (latencia) y en el cómputo requerido, ya que no es necesario comparar el vector de consulta contra el 100% de la base de datos (evita la búsqueda exhaustiva $O(N)$).
Lo que se pierde: Un pequeño margen de precisión/exactitud (Recall < 100%), puesto que existe la posibilidad de omitir algunos de los vecinos más cercanos reales si estos quedaron asignados a clusters no inspeccionados por el valor de nprobe.

6## Columnar

**19.** Tamaño de los datos en disco:
CSV: Ocupa 19.8 MB.   
JSON: Ocupa 48.0 MB.   
Parquet: Ocupa 2.1 MB.

Parquet resulta ser aproximadamente 9.4 veces más liviano que CSV y 22.8 veces más liviano que JSON gracias a la compresión columnar.   

Columnas leídas por Spark (ReadSchema):Para la consulta de agregación sobre amount, Spark aplicó column pruning y leyó únicamente la columna requerida.   

Archivos salteados con los datos ordenados (Data Skipping):Al distribuir los datos en 8 archivos ordenados por la columna amount, Spark/Parquet puede evaluar el rango mínimo y máximo guardado en los metadatos de cada archivo. 
Para la condición amount > 1900, se pueden saltear 7 de los 8 archivos, procesando únicamente el último archivo cuyo rango de valores contiene montos superiores a 1900.

**20.** Por qué es bueno para analítica:

Lectura selectiva de columnas (Column Pruning): Las consultas analíticas suelen calcular agregaciones sobre unas pocas columnas específicas de tablas con muchas dimensiones (por ejemplo, SUM(amount)). 
El almacenamiento columnar permite leer de disco únicamente los bloques de memoria asignados a las columnas necesarias, evitando procesar el resto.   

Alta compresión: Al guardar datos del mismo tipo de forma contigua en disco (enteros con enteros, cadenas con cadenas, etc.), se aplican algoritmos de compresión como Snappy o Dictionary Encoding de forma mucho más eficiente que en formatos orientados a filas.   

Salteo de datos (Data Skipping): Los metadatos integrados por bloque/archivo registran estadísticos, lo que permite omitir bloques de datos enteros durante las lecturas analíticas mediante filtros (PushedFilters).   

Por qué es poco conveniente para modificar una fila por vez (OLTP / Row-level updates):Dado que los valores de una misma fila se encuentran dispersos y fragmentados a lo largo de distintos archivos/bloques de columnas en disco, insertar, actualizar o eliminar un solo registro exige reescribir bloques enteros de múltiples columnas o reestructurar completamente el archivo Parquet, lo cual genera un alto overhead de computación y de E/S (Input/Output).

