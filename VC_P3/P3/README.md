## Práctica 3. Detección y reconocimiento de formas

### Autores:
- Óscar Muñoz Hidalgo
- Juan José Quesada Acosta

### Contenidos

[Detector de monedas](#31-detector-de-monedas)  
[Clasificador de microplásticos](#32-clasificador-de-microplásticos)

### Paquetes a instalar
```
pip install scikit-learn seaborn
```

### 3.1. Detector de monedas

En esta tarea hemos desarrollado un contador de dinero a partir de una imagen con monedas dispersas a partir de la selección de una moneda de 1 euro como medida comparativa, se selecciona la moneda haciendo click izquierdo sobre ella.

A partir de la imagen original, se detectan los contornos principales con Canny introduciendo como input una imagen en escala de grises con un blur Gaussiano.

![Imagen de bordes canny](./Assets/README%20Images/canny_output.png)

Se prosigue con el umbralizado de Otsu al que se le substrae el valor obtenido en Canny para hacer un resalte exagerado de los bordes.

![Imagen de OTSU menos bordes canny](./Assets/README%20Images/monedas_diff.png)

Finalmente esta imagen se procesa para la obtención de los bordes circulares externos de las monedas

![Imagen de discriminación de los circulos de las monedas](./Assets/README%20Images/contours_money.png)

Una vez ya está detectadas todos los posibles contornos se procede con el conteo de dinero.

Al hacer click sobre la moneda de 1 Euro se divide el diámetro real entre el diámetro de la imagen y se obtiene un factor de escala que se usará más adelante.

Previamente se han generado dos diccionarios, uno para los diametros reales de cada moneda y otro para el dinero total contado. 

Se prosigue detectando los contornos activos y preguntando si es una moneda con la función isCoin que comprueba en la imagen anterior si sus pixeles son blancos, en caso afirmativo se multiplica su diámetro en la imagen por el factor de escala para obtener su diámetro real equivalente y se agrupa en el intervalo marcado por el umbral de tolerancia de los diametros de las monedas correspondientes, esto añade una moneda de esta clase al contador del diccionario de totales.

Se prosigue con el conteo hasta que finalmente no queden contornos, sumando la cantidad de cada moneda por su valor y printeando los resultados.

![Resultado de conteo de dinero](./Assets/README%20Images/results_money.png)

### 3.2. Clasificador de microplásticos

En esta tarea hemos desarrollado un clasificador de microplásticos en las 3 clases especificadas (fragmentos, *pellets* y alquitrán), a partir de la aplicación de umbralizado de imágenes y extracción de características a partir del análisis de las características geométricas de los contornos detectados en ellas.

El primer paso consistió en buscar cuál era el mejor método para umbralizar cada una de las imágenes de prueba, tarea que se nos complicó inicialmente debido a las sombras que tienen originalmente. Para solventar ese problema recurrimos a recortarlas las imágenes para obtener una imagen con los microplásticos exclusivamente.

![Imagen de fragmentos umbralizada sin recortar](./Assets/README%20Images/fragmentos-sin-recortar.png)

Tras ello, y jugando con diferentes tipos y valores de umbralizado, alcanzamos el mejor resultado con:

- Desenfoque Gaussiano y umbralizado básico para fragmentos y *pellets*.
- Umbralizado con Otsu para alquitrán.

#### Umbralizado de las imágenes de prueba de cada clase

![Umbralizado de las imágenes de prueba de cada clase](./Assets/README%20Images/umbralizado-por-clases.png)

A continuación, desarrollamos un clasificador de contornos a partir de las imágenes umbralizadas, y cuyos criterios para diferenciar a qué clase pertenece cada uno variaron en cuanto a la dificultad para definirlos. Para los pellets, la relación de aspecto surgió rápido. En cuanto al alquitran, analizar el color debido a que todos los trozos son negros. Para el resto de criterios decidimos analizar en bruto las siguientes características geométricas:

- Compacidad (relación del cuadrado del perímetro con el área C=P^2/A).
- Relación de aspecto del rectángulo mínimo que encierra el contorno.
- Relación del área de la partícula con el área del contenedor que la contiene.
- Relación entre los ejes de la elipse ajustada.

#### Análisis de características geométricas para cada clase de microplásticos

![Conjunto de datos analizados en bruto](./Assets/README%20Images/datos-en-bruto.png)

Finalmente, decidimos elegir lo siguientes criterios.

Para los *pellets*:

- Compacidad entre 13 y 15, y relación de aspecto en torno a 1.

Para el alquitrán:

- Valor promedio del color (luz) de los píxeles menor a 55 (negro), compacidad mayor a 15 y relación del área de la partícula con el área del contenedor que la contiene entre 0.55 y 0.75.

Y para los fragmentos (la clase menos homogénea):

- Relación del área de la partícula con el área del contenedor que la contiene menor a 0.65, número aproximado de lados menor a 9 y relación entre los ejes de la elipse ajustada menor a 0.78.

#### Elementos detectados en cada imagen

Aquí es donde encontramos el principal problema, pues nuestro detector no diferenciaba con exactitud el número de elementos de cada imagen, por lo que las métricas obtenidas tuvieron que ser adaptadas a esta casuística. En comparación:

|            | Elementos reales  | Elementos detectados |
|------------|-------------------|----------------------|
| Fragmentos |   **80**          |   *82*               |
| Pellets    |   **55**          |   *48*               |
| Tar        |   **54**          |   **55**             |

A continuación se muestra la salida del programa con la distribución de elementos encontrados en cada imagen, analizadas en el siguiente orden: imagen de solo fragmentos → pellets → tar

![Elementos detectados por imagen](./Assets/README%20Images/elementos-detectados-por-imagen.png)

Por consiguiente, decidimos analizar las métricas de rendimiento en base a los valores que obtuvimos con nuestro clasificador, excepto el accuracy, que está expresado como **TP / n** :

- Donde *TP* son los verdaderos positivos (diagonal ppal. de la matriz de confusión).
- Y *n* es la suma de, para cada clase de microplásticos, el máximo entre el valor real (anotación o primera columna de la tabla anterior) y el valor detectado por el programa.

En las otras 3 métricas mantenemos la fórmula original, adaptando los valores a los resultados obtenidos por nuestro clasificador. Dichas métricas tienen el siguiente resultado:

#### Resultados del reconocimiento para cada clase

![Resultados del reconocimiento para cada clase](./Assets/README%20Images/resultados-por-clases.png)

Finalmente, mantenemos el valor de accuracy calculado como antes se menciona y calculamos el promedio de las demás métricas.

#### Resultados del reconocimiento en promedio

![Resultados del reconocimiento en promedio](./Assets/README%20Images/resultados-totales.png)

Como conclusión, tenemos un clasificador que tiene un gran margen de mejora en cuanto al reconocimiento de los elementos como tal, pero cuya labor de clasificación tiene unos resultados realmente aceptables. Es capaz de diferenciar y clasificar con bastante precisión los elementos que detecta en base a las características geométricas analizadas.

#### Matriz de confusión resultante

![Matriz de confusión resultante](./Assets/README%20Images/matriz-de-confusion.png)
