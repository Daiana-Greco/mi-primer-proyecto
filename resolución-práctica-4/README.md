# Entrega práctica 4 — Streaming

- **Nombre:** Daiana Greco
- **student_id:** DaiGreco
- **Escala:** Small

Respondé cada pregunta en una a tres oraciones. Cuando se pide un dato, copiá el valor que te dio el notebook.

## Log y offsets

**1.** En la cola quedaron 2 mensajes y luego en el log 5 mensajes

Cola total de 2 mensajes:
2 en la cola: ['pago_4', 'pago_5']

Log total de 5 mensajes.

**2.** Después de la llegada 2 antifraude habia leido 180 mensajes mientras que reportes habia leido 0. Esto no afecta, por que despues de la llegada 3 de reportes logra leer los 220 mensajes acumulados.

**3.** OFFSET son las posiciones del log hasta que llegó un consumidor o sea, desde dónde debe continuar leyendo.

**4.** El replay leyo 280 mensajes. Kappa propone reprocesar los datos leyendo nuevamente el log desde el principio con la lógica de streaming. Por otro lado, Lambda tiene dos capas, una lote y otra de streaming.

## Ingesta y enriquecimiento

**5.** Bronze: 60, 120, 40, 40 y 20. Al reejecutar procesó 0 filas. El motivo por que no se leyo es por Auto Loader que reconoce o recuerda los archivos ya fueron leídos para no procesarlos otra vez.

**6.** Se detectaronmINVALID_AMOUNT y UNKNOWN_CUSTOMER. Bronze conserva el original para no perder información.

**7.** El join sumo país (silver_customers) y categoría (silver_products). Es un join stream-tabla, porque los eventos llegan como stream y se enriquecen utilizando tablas estáticas de referencia.

## Tiempo y ventanas

**8.** El event time indica cuándo ocurrió un evento, mientras que el process time nos muestra cuando es procesado por el sistema. late_ok ocurrió a las 12:03, pero llegó mas tarde.

**9.** Watermark: 11:54, 11:58, 12:16, 12:17 y 12:35. Son 10 minutos menos del tiempo máximo del evento observado.

**10.** Aparecen en la llegada 3. Esto es por el watermark, en la llegada 3 alcanza las 12:16 permitiendo cerrar las ventanas anteriores.

**11.** El watermark estaba en 12:16 cuando llega late_bad, (12:02), como era menor fue descartado. late_ok llegó antes de que la marca de agua cerrara, por tanto no se descarto.

**12.** Tumbling: Tiene una duración fija, sus ventanas no se superponen.

Hopping: Tiene una duración fija, pero se repite con un intervalo menor (avance), lo que hace que las ventanas se superpongan. 

Session Window: No tiene duración fija. 

**13.** Update se emitió dos veces. en la llegada 1 con 40 compras y monto 6000, y en la llegada 2 con 60 compras y monto 8400. 

Append se emitió sola en la llegada 3, con 60 compras y monto 8400. 

**14.** Al reducir el watermark las ventanas se cerrarian antes, lo que reduce la latencia.

## Garantías y fallas

**15.** Sin deduplicar: 220 filas  

Deduplicando: 200 filas

Se puede enviar el mismo evento dos o mas veces ante una no confirmación, con el fin de no perderlo.

**16.** Al reiniciar con el mismo checkpoint mostro 200 filas. Al perder el checkpoint sed volvio a ejecutar y mostro 400 filas.

**17.** Aplicando MERGE se mostraron 200 filas con y sin el checkpoint. Esto sucede por que MERGE identifica el event id y no lo duplica.


## CDC

**18.** Genero 13 eventos de cambio. Update genera el estado anterior y posterior al cambio.

**19.** Si, la tabla es igual a la original.

## Cierre

**20.** 
