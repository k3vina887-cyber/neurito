# Neurito – Mi cerebro artificial

## Experimentos

### Experimento 1 – Dos necesidades fijas y una variable

Se dejaron dos necesidades fijas y la tercera se movió de 0 a 1 en pasos de 0,005, registrando en qué valor cambia la acción predicha.

| Variable | Valores fijos | Cambio de acción | Antes → después |
|----------|---------------|------------------|-----------------|
| Hambre | energía 0,8 (cansancio 0,2) y aburrimiento 0,3 | en 0,30 | JUGAR → COMER |
| Energía | hambre 0,3 y aburrimiento 0,4 | en 0,60 | DORMIR → JUGAR (al subir la energía) |
| Aburrimiento | hambre 0,3 y energía 0,8 (cansancio 0,2) | en 0,30 | COMER → JUGAR |

**Análisis y hallazgos:**

Se puede observar que la acción cambia justo cuando la prioridad que se está moviendo supera a la más alta de las otras dos. En el primer caso la prioridad más alta de las fijas es el aburrimiento (0,3), por eso con hambre entre 0 y 0,29 la acción siempre es JUGAR y, apenas el hambre pasa de 0,30, cambia a COMER. En el caso de la energía el umbral de 0,60 sale de que el cansancio (`1 - energía`) tiene que ser mayor que 0,4: con energía de 0,55 la red da DORMIR con 68,9%, y con 0,65 da JUGAR con 64,6%. Con energía 0 la red da DORMIR casi con el 100%. Mientras la variable no cruza ese umbral la acción no cambia, aunque las probabilidades sí se van moviendo poco a poco: en el primer caso, con hambre de 0,25 JUGAR tiene 63,7% y COMER 26,4%; con 0,30 quedan casi empatadas (COMER 48,4% y JUGAR 44,5%); y con 0,35 COMER sube a 71,1%. En los tres barridos los umbrales de la red coinciden con los de la regla original, así que la red aprendió bien las fronteras.

También se explica algo del estado tranquilo del programa. Con los valores de la tecla `N` (0,2 – 0,9 – 0,2) las prioridades son 0,2 (hambre), 0,1 (cansancio) y 0,2 (aburrimiento), es decir, hay un empate entre hambre y aburrimiento. La red responde COMER 48,2%, JUGAR 44,7% y DORMIR 7,1%. Esto da entender que la mascota se ve tranquila por la regla de visualización y no porque la red haya decidido "no hacer nada".

**Hallazgo interesante:** el punto de cambio no depende de la variable que se mueve sino de las otras dos fijas, y en la zona anterior al umbral mover la variable no cambia la acción, solo cambia qué tan segura está la red.

### Experimento 2 – Prioridades casi iguales y Softmax

Se probaron entradas donde dos o tres prioridades son iguales o casi iguales y se comparó la salida de Softmax con la acción que da la regla.

| Entrada (hambre, energía, aburrimiento) | Prioridades (hambre, cansancio, aburrimiento) | Softmax COMER / DORMIR / JUGAR | Red | Regla |
|---|---|---|---|---|
| 0,5 – 0,5 – 0,5 | 0,50 – 0,50 – 0,50 | 35,2% / 33,4% / 31,3% | COMER | COMER |
| 0,52 – 0,5 – 0,5 | 0,52 – 0,50 – 0,50 | 44,3% / 28,9% / 26,8% | COMER | COMER |
| 0,6 – 0,4 – 0,58 | 0,60 – 0,60 – 0,58 | 39,3% / 37,2% / 23,5% | COMER | COMER |
| 0,2 – 0,9 – 0,2 | 0,20 – 0,10 – 0,20 | 48,2% / 7,1% / 44,7% | COMER | COMER |
| 0,8 – 0,2 – 0,2 | 0,80 – 0,80 – 0,20 | 49,1% / 50,9% / ≈0,0% | DORMIR | COMER |
| 0,9 – 0,1 – 0,1 | 0,90 – 0,90 – 0,10 | 48,8% / 51,2% / ≈0,0% | DORMIR | COMER |

**Análisis y hallazgos:**

Cuando las tres prioridades son iguales Softmax casi no se decide y reparte la probabilidad cerca de 1/3 para cada una (35,2%, 33,4% y 31,3%). Basta una ventaja de 0,02 en el hambre para que COMER suba de 35,2% a 44,3%, lo que muestra que la salida es bastante sensible cerca de los empates. Cuando el empate es entre dos prioridades altas y la tercera es baja, la probabilidad se divide casi 50/50 entre las dos que compiten y la tercera queda en cero. Esto da entender que Softmax reparte la probabilidad entre las opciones que de verdad compiten, y que el porcentaje es una medida de seguridad y no solo un ranking.

Los dos últimos casos son los únicos donde la red y la regla no coinciden, y en ambos hay un empate exacto (0,8 contra 0,8 y 0,9 contra 0,9). La regla desempata con `argmax`, que siempre toma la primera opción (COMER), pero la red nunca vio un criterio de desempate, solo ejemplos casi iguales, y por eso da 50,9% y 51,2% a DORMIR. Un empate exacto no es un error real de la red, es una decisión arbitraria de la regla. Al revisar las 1.000 muestras de prueba esto se confirma: la red falló en 6 casos y en todos la diferencia entre las dos prioridades más altas fue de 0,014 o menos (promedio de 0,005 en los errores contra 0,252 en los aciertos). Con diferencias menores a 0,02 hay 60 muestras y la precisión es de 90,0%; con diferencias de 0,02 o más son 940 muestras y la precisión es de 100%. La confianza promedio en los errores es de 51,1% contra 89,8% en los aciertos.

**Hallazgo interesante:** todos los errores de la red están pegados a las fronteras entre acciones, y la probabilidad de Softmax avisa de eso: cuando la red se equivoca casi nunca está segura.

### Experimento 3 – Tamaño de la capa oculta (3, 6, 10 y 20 neuronas)

Se entrenó la red con cada tamaño. Como un solo entrenamiento depende mucho de la semilla (que cambia tanto los datos como los pesos iniciales), además del resultado con la semilla 42 se calculó el promedio de 5 semillas (42, 0, 1, 2 y 3).

| Neuronas ocultas | Parámetros | Semilla 42: pérdida | Semilla 42: precisión | Promedio 5 semillas: pérdida | Promedio 5 semillas: precisión (mín – máx) |
|---|---|---|---|---|---|
| 3 | 24 | 0,1618 | 96,30% | 0,1750 | 95,98% (95,4 – 96,4) |
| 6 | 45 | 0,1225 | 99,40% | 0,1526 | 97,82% (96,8 – 99,4) |
| 10 | 73 | 0,1262 | 99,30% | 0,1238 | 98,98% (98,5 – 99,3) |
| 20 | 143 | 0,1220 | 98,80% | 0,1215 | 99,10% (98,8 – 99,5) |

**Análisis y hallazgos:**

Con 3 neuronas la red queda claramente por debajo, con cerca de 96% de precisión y la pérdida más alta en todas las semillas. Con tan pocas neuronas la red tiene poca capacidad para armar las tres fronteras (incluyendo la inversión de la energía), así que se queda corta. De 6 a 10 neuronas el promedio mejora de 97,82% a 98,98% y la pérdida baja de 0,1526 a 0,1238, y de 10 a 20 casi no hay diferencia (98,98% a 99,10%, y 0,1238 a 0,1215), es decir, hay rendimientos decrecientes.

Hay que tener cuidado con la lectura de la semilla 42: ahí 6 neuronas (99,40%) parecen mejores que 20 (98,80%), pero esto es suerte de esa semilla, porque en el promedio de 5 semillas 6 neuronas es la configuración menos estable (va de 96,8% a 99,4%) y 20 es la más estable. Además, el conjunto de prueba tiene 1.000 muestras, así que cada error vale 0,1 puntos y diferencias como 99,4% contra 99,3% no significan nada. Lo más probable es que desde 6 neuronas en adelante el límite ya no sea el tamaño de la red sino el tiempo de entrenamiento, ya que con 1.200 épocas la pérdida se queda en 0,12 aproximadamente, pero con 6 neuronas y más épocas baja mucho más (ver Experimento 4).

**Hallazgo interesante:** más neuronas no siempre dan mejor resultado en un solo entrenamiento, y para comparar tamaños es mejor mirar varios entrenamientos que uno solo.

### Experimento 4 – Épocas y tasa de aprendizaje

**Épocas** (tasa de aprendizaje 0,1 y 6 neuronas ocultas):

| Épocas | Pérdida | Precisión en prueba |
|---|---|---|
| 50 | 0,9388 | 60,60% |
| 200 | 0,4984 | 91,80% |
| 600 | 0,1944 | 98,60% |
| 1.200 | 0,1225 | 99,40% |
| 3.000 | 0,0711 | 99,50% |
| 6.000 | 0,0484 | 99,60% |

**Tasa de aprendizaje** (1.200 épocas y 6 neuronas ocultas):

| Tasa | Pérdida final | Precisión en prueba | Comportamiento |
|---|---|---|---|
| 0,001 | 1,0333 | 41,80% | Casi no avanza (parte de 1,0742) |
| 0,01 | 0,7376 | 82,90% | Mejora, pero muy lento |
| 0,05 | 0,1943 | 98,60% | Estable |
| 0,1 | 0,1225 | 99,40% | Valor base, estable |
| 0,5 | 0,0484 | 99,60% | Estable y más rápido |
| 1 | 0,0447 | 98,70% | Pérdida más baja, pero precisión menor |
| 2 | 0,0557 | 97,30% | Inestable al inicio (la pérdida sube hasta 1,479) |
| 5 | 1,0979 | 33,80% | Se estanca en ln(3) |
| 10 | 2,8771 | 33,80% | Se estanca en un valor alto |
| 50 | 14,5763 | 34,20% | Inestable, la pérdida salta entre 5,8 y 15,7 |

**Análisis y hallazgos:**

Con las épocas se ve primero una mejora fuerte y después un estancamiento. En las primeras 600 épocas se logra casi todo (la precisión sube de 60,6% a 98,6%), y entre 1.200 y 6.000 épocas, es decir con cinco veces más entrenamiento, la precisión solo sube 0,2 puntos (de 99,4% a 99,6%) aunque la pérdida sí se reduce a menos de la mitad (de 0,1225 a 0,0484). Esto da entender que la red sigue ajustando las probabilidades, pero la precisión ya casi no puede subir porque los errores que quedan son los empates del Experimento 2.

Con la tasa de aprendizaje hay tres zonas. Entre 0,001 y 0,01 el entrenamiento es demasiado lento y con 1.200 épocas no alcanza (41,8% y 82,9%). Entre 0,05 y 1 mejora bien, y se observa que una tasa de 0,5 con 1.200 épocas da exactamente la misma pérdida que una de 0,1 con 6.000 épocas (0,0484), lo que indica que en la zona estable lo que importa es el producto entre la tasa y las épocas. Con 1 y 2 empiezan los problemas: la pérdida baja más (0,0447 con tasa 1), pero la precisión cae a 98,7% y con tasa 2 la pérdida sube a 1,479 en algún momento antes de recuperarse. Con 5 o más el entrenamiento se rompe: con tasa 5 solo queda activa 1 de las 6 neuronas ocultas y apenas el 0,1% de las activaciones es positivo, así que la red solo aprende la proporción de cada clase (34,3%, 34,1% y 31,6%) y la pérdida se queda en 1,0979, casi igual a ln(3). Lo más probable es que los pasos tan grandes empujen las neuronas a valores negativos donde ReLU las apaga y ya no se recuperan. Con tasa 10 la red termina prediciendo casi siempre COMER (95,4% en promedio) y con 50 la pérdida salta sin converger.

**Hallazgo interesante:** una pérdida más baja no siempre significa mejor precisión (tasa 1 contra tasa 0,5), y una precisión de 33,8% es prácticamente lo mismo que adivinar, ya que la clase más frecuente es de 34,2%.

### Experimento 5 – Una cuarta acción

**Análisis y hallazgos:**

Agregar una acción no es solo añadir una salida. Se necesita, en orden:

1. **Un dato nuevo de entrada.** La red solo puede decidir con lo que recibe, así que la cuarta acción debe tener su propia necesidad. Por ejemplo, BAÑARSE con una necesidad de **suciedad** (0 limpio, 1 muy sucio). Como la suciedad sube cuando la necesidad sube, igual que el hambre y el aburrimiento, no necesita inversión. Si la necesidad nueva se midiera al revés, como la energía (por ejemplo "salud"), habría que invertirla igual que el cansancio. Si no se agrega una entrada nueva, la etiqueta de la cuarta acción tendría que ser una función de las mismas tres necesidades, y entonces le quitaría casos a otra acción o sería redundante, y la red no tendría forma de distinguirla.
2. **Una etiqueta nueva.** La regla de generación pasa a ser `argmax([hambre, 1 - energía, aburrimiento, suciedad])` y la clase 3 corresponde a la cuarta acción. También hay que definir cómo se desempata y agregar la acción a la tupla `actions` y a los textos de `TEXT`.
3. **Datos suficientes y balanceados.** Debe haber ejemplos de la clase nueva en los datos de entrenamiento y de prueba. Con valores uniformes en 4 entradas el reparto quedó en 25,0%, 25,8%, 25,0% y 24,3%, así que ninguna clase domina.
4. **Cambios en la forma de la red.** `X` pasa de 3 a 4 columnas, `W1` de 3×6 a 4×6, `W2` de 6×3 a 6×4 y `b2` de 1×3 a 1×4, para un total de 58 parámetros en lugar de 45.
5. **Cambios en la interfaz.** Hace falta una tecla para subir y bajar la suciedad, una cuarta barra, nuevos sprites (por ejemplo `banarse_1.png` y `banarse_2.png`), una entrada nueva en `frames` y en `needs`, y actualizar la regla del estado neutral para que también tenga en cuenta la suciedad.

Se probó el entrenamiento con esta cuarta acción (misma semilla, 1.200 épocas, tasa 0,1): con 6 neuronas la pérdida fue de 0,2073 y la precisión de 97,80%, y con 10 neuronas fue de 0,1790 y 98,10%. Es peor que la red de tres acciones (99,40%), y lo más probable es que sea porque con cuatro clases hay más pares de prioridades que se pueden empatar y la red tiene más que aprender con el mismo número de épocas. Es un solo entrenamiento por configuración, así que sirve como referencia y no como conclusión.

**Hallazgo interesante:** la parte más difícil de agregar una acción no es el código de la red sino definir bien la regla que genera las etiquetas, porque la red solo aprende lo que esa regla le enseña.




### 1. ¿En qué se diferencian pesos y sesgos?

Los **pesos** (`W1` y `W2`) multiplican a cada entrada y definen cuánto influye cada valor sobre cada neurona. Por ejemplo, un peso grande y negativo sobre la energía hace que una neurona se active cuando la energía es baja. Los **sesgos** (`b1` y `b2`) se suman después de esa multiplicación y no dependen de la entrada, es decir, mueven el punto en el que una neurona se activa o el punto de partida de cada salida. Los pesos fijan la dirección en la que reacciona la red y los sesgos fijan desde dónde parte.

Un ejemplo claro sale de la red entrenada: si las tres necesidades valen 0, la multiplicación por `W1` da cero y la salida depende solo de los sesgos y de `W2`. En ese caso la red predice DORMIR con 99,996%, que es correcto porque energía 0 significa cansancio 1. El sesgo de salida aprendido fue `b2 = [-0,982, 1,030, -0,048]`, que le da a DORMIR una ventaja inicial. Otro detalle es que en el código los sesgos empiezan en cero (`np.zeros`) pero los pesos empiezan aleatorios (`np.random.randn * 0.5`); esto es necesario porque si todos los pesos empezaran iguales todas las neuronas aprenderían lo mismo, mientras que los sesgos sí pueden empezar iguales sin ese problema.

### 2. ¿Por qué el cansancio es `1 - energía`?

Porque para comparar cuál necesidad es la más urgente las tres tienen que ir en el mismo sentido, donde un valor más alto significa más urgencia. El hambre y el aburrimiento ya funcionan así: mientras más altos, más se necesita comer o jugar. La energía va al revés, ya que una energía alta significa que la mascota está descansada y no necesita dormir. Si se comparara la energía directamente, `argmax` elegiría DORMIR justo cuando la mascota tiene más energía. Al usar `1 - energía` se obtiene el cansancio, que sí sube cuando la energía baja (energía 0,9 es cansancio 0,1 y energía 0,1 es cansancio 0,9). Se resta de 1 porque todos los valores van de 0 a 1, y así el cansancio queda en la misma escala que las otras dos prioridades.

Hay un detalle: esa inversión solo está en la regla que genera las etiquetas. A la red le entra la energía tal cual, y es ella la que tiene que aprender sola la inversión, lo cual se ve en el Experimento 1, donde por debajo de 0,60 de energía la red elige DORMIR.

### 3. ¿Qué hace ReLU y qué hace Softmax?

**ReLU** es `max(0, x)`: deja pasar los valores positivos tal cual y convierte los negativos en cero. Se aplica en la capa oculta y sirve para introducir una no linealidad. Sin ella, las dos capas se podrían reducir a una sola operación lineal y la capa oculta no aportaría nada. En este proyecto se comprobó que un modelo lineal sin capa oculta, con el mismo entrenamiento, llega a 95,70% de precisión y pérdida de 0,3893, contra 99,40% y 0,1225 de la red con capa oculta y ReLU. También se vio su lado negativo en el Experimento 4: con una tasa de aprendizaje muy alta las neuronas ReLU quedan apagadas (siempre en cero) y dejan de aprender.

**Softmax** se aplica en la salida y convierte los tres puntajes de la red en probabilidades: todas quedan entre 0 y 1 y suman 100%. Para esto calcula `e^puntaje / suma de e^puntajes`, y en el código se resta el máximo antes de calcular para evitar números demasiado grandes. Gracias a esto la red no entrega solo una acción sino qué tan segura está de cada una, que es lo que se ve en el Experimento 2 y en los porcentajes de la ventana. Además, es la salida que se combina con la entropía cruzada: la pérdida es `-log(probabilidad de la acción correcta)`, así que castiga más cuando la red le asigna poca probabilidad a la acción correcta.

### 4. ¿Por qué una precisión alta puede ser engañosa?

La precisión de 99,40% de Neurito es un buen resultado, pero hay varias razones por las que no se puede leer sin cuidado:

- **Mide contra una regla inventada.** El programa lo dice en su propia salida: "precisión frente a la política sintética". Los datos de prueba se generan con la misma regla que los de entrenamiento, así que la precisión solo indica qué tan bien la red copia esa regla, no si esa regla es una buena forma de cuidar una mascota.
- **Solo cuenta la acción ganadora y no la seguridad.** De los aciertos, 10 se dieron con menos de 50% de confianza, y la confianza promedio en los errores (51,1%) es muy distinta de la de los aciertos (89,8%). Dos redes con la misma precisión pueden estar mucho más o menos seguras de sus respuestas.
- **Esconde dónde falla.** El 0,60% de error está concentrado en los empates: en muestras con diferencia menor a 0,02 entre prioridades la precisión es de 90%, y en el resto es de 100%. El número global no dice que el problema está en las fronteras.
- **Depende del reparto de clases.** Aquí las clases están balanceadas (34,3%, 34,1% y 31,6% en entrenamiento), por eso adivinar siempre la misma acción daría cerca de 34%. Si una acción fuera el 95% de los casos, una red que siempre responda esa acción tendría 95% de precisión sin haber aprendido nada.
- **No siempre va de la mano con la pérdida.** En el Experimento 4 la tasa de 1 tuvo la pérdida más baja (0,0447) pero una precisión menor (98,70%) que la de 0,5 (0,0484 y 99,60%).
- **El conjunto de prueba es pequeño.** Con 1.000 muestras un solo error cambia la precisión en 0,1 puntos, así que diferencias de 99,4% contra 99,3% no son significativas, y con otra semilla la misma red de 6 neuronas llegó a 96,8%.
- **No evalúa todo el programa.** El estado neutral es una regla aparte y la precisión no lo incluye.
