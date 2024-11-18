# Práctica 5. Detección y caracterización de caras

En esta práctica hemos realizado una aplicación para un detector de caras en la que recreamos el aspecto del personaje básico principal del videojuego Balatro conocido como Joker (equivalente a la carta del Joker en el póker).

![Joker](./images/joker.png)
![Joker](./images/joker.png)
![Joker](./images/joker.png)
![Joker](./images/joker.png)

El resultado ha sido el siguiente, del que podemos destacar:

- La capacidad de rotación del sombrero acorde a la rotación de la cara.
- La aplicación de un contorno de ojos y de boca simulando el maquillaje del Joker.
- La capacidad de aplicar el filtro a tiempo real a un máximo de 2 personas.

![Filtro con una persona](./images/filter_one_person.gif)

![Filtro con dos personas](./images/filter_two_people.gif)

## Autores

- Óscar Muñoz Hidalgo
- Juan José Quesada Acosta

## Contenidos

[Paquetes a instalar](#paquetes-a-instalar)

[Importes usados](#importes-usados)

[Identificación en landmarks](#identificación-de-landmarks)

[Inicialización de elementos](#inilización-de-elementos)

[Cálculo de posición y rotación de la cara](#cálculo-de-posición-y-rotación-de-la-cara)

[Posicionamiento del sombrero y collar](#posicionamiento-del-sombrero-y-collar)

[Cierre](#cierre)

## Paquetes a instalar

```
conda create --name VC_P5_G13 python=3.9.5
conda activate VC_P5_G13

pip install opencv-python
pip install mediapipe
pip install numpy
```

## Importes usados

```
import cv2
import mediapipe as mp
import numpy as np
import math
```
## Identificación de landmarks

Mediapipe es capaz de detectar numerosos puntos de interes de características de la cara, por lo tanto como primer paso se debe detectar y clasificar aquellos relevantes para su posterior uso.

En este caso los landmarks de interés son los del contorno exterior de la cara, y algunos de los ojos y boca, asi que se extraen los indices relevantes y agrupan por categoría.
```
face_outline_indices = [
    10, 338, 297, 332, 284, 251, 389, 356, 454, 323, 361, 288, 397, 365, 379, 378, 400, 377, 152, 148, 176, 149, 150, 136, 172, 58, 132, 93, 234, 127, 162, 21, 54, 103, 67, 109
]

left_eye_indices = [
    33, 7, 163, 144, 145, 153, 154, 155, 133, 157, 158, 159, 160, 161, 246
]

right_eye_indices = [
    362, 398, 384, 385, 386, 387, 388, 466, 373, 374, 380
]

upper_mouth_indices = [
    61, 185, 40, 39, 37, 0, 267, 270, 409, 291, 375, 321, 405, 314, 17, 84, 181
]

lower_mouth_indices = [
    146, 91, 181, 84, 17, 314, 405, 321, 375, 291, 409, 270, 267, 0, 37, 39, 40, 185
]
```

Para obtener los landmarks deseados se imprimen todos y se anotan aquellos que interesen, pues seran sujeto de cambio dependiendo del objetivo del proyecto.

IMAGEN

## Inilización de elementos

```
# Inicializar MediaPipe Face Mesh (para obtener los landmarks)
mp_face_mesh = mp.solutions.face_mesh
mp_drawing = mp.solutions.drawing_utils

# Cargar la imagen del sombrero y collar (sin cambios)
hat = cv2.imread('./images/hat_crop.png', cv2.IMREAD_UNCHANGED)
hat_h, hat_w, _ = hat.shape

ruff = cv2.imread('./images/ruff_crop.png', cv2.IMREAD_UNCHANGED)
ruff_h, ruff_w, _ = ruff.shape

# Inicializar la cámara
cap = cv2.VideoCapture(0)
```

En este punto se inicializa un FaceMesh() y se genera un bucle para obtener cada frame recogido por la cámara como una imagen.

## Cálculo de posición y rotación de la cara
En cada frame se analizan los puntos nombrados a continuación
### 1. Obtener el tamaño del frame

```
# Dimensiones de la imagen
ih, iw, _ = image_bgr.shape
```
### 2. Discriminar FaceMesh() con los puntos de interés
De todos los landmarks detectados se seleccionan solo aquellos relevantes, osea cuyos indices coincidan con los antes descritos.
```
# Obtener los puntos del contorno de la cara
contour_points = [
    (int(face_landmarks.landmark[i].x * iw), int(face_landmarks.landmark[i].y * ih)) for i in face_outline_indices
]
```
### 3. Obtener la rotación de la cara
Para ello se traza una línea entre dos puntos extremos opuestos en la parte superior de la cabeza y se compara su ángulo con la horizontal.

>[!CAUTION] Se pueden usar otros puntos alineados horizontalmente de la cara, pero se recomienda que sean por encima de la parte superior de la mandíbula pues puede causar fallos

```
# Obtener rotación de la cara a partir de la línea superior de la cara, trazada entre los landmarks 54 y 284

ul_x, ul_y = int(face_landmarks.landmark[54].x * iw), int(face_landmarks.landmark[54].y * ih)    # Esquina superior izquierda (upper-left)

ur_x, ur_y = int(face_landmarks.landmark[284].x * iw), int(face_landmarks.landmark[284].y * ih)  # Esquina superior derecha (upper-right)

# Calcular la diferencia en x e y para obtener el ángulo de rotación de la cara

dx = ur_x - ul_x
dy = ur_y - ul_y

# Calcular el ángulo de rotación, como el ángulo entre la línea y el eje horizontal

angle_deg = -np.degrees(np.arctan2(dy, dx))             # Negativo debido a la inversión de en la imagen de la cámara
```
## Posicionamiento del sombrero y collar

El primer paso es calcular el centro de la línea superior de la cabeza para colocar el resto de accesorios en relación a esta y su longitud, para así saber la escala, para ello no se puede solo calcular su distancia en x, pues al estar girado se debe calcular el módulo del vector o su hipotenusa: 

$$ Hipotenusa=\sqrt{cateto^2+cateto^2} $$
$$ Módulo=\sqrt{Δx^2+Δy^2} $$

Una vez obtenidos estos datos se sigue el siguiente procedimiento para cada objeto superpuesto en el filtro, en este caso serán dos, el gorro y el collar

### 1. Redimensionado y rotación en funcion de la situación de la cara
Para ello se usa la función resize_and_rotate() que se ha creado que funciona mediante multiplicaciones de matrices de rotación 2D.

```
def resize_and_rotate(image, width, height, angle_deg):
    resized = cv2.resize(image, (width, height), interpolation=cv2.INTER_AREA)

    center = (width // 2, height // 2)

    rotation_matrix = cv2.getRotationMatrix2D(center, angle_deg, 1.0)

    rotated = cv2.warpAffine(resized, rotation_matrix, (width, height), flags=cv2.INTER_LINEAR, borderMode=cv2.BORDER_REPLICATE)
    return rotated
```

### 2. Ajustar dimensiones y realización de ajustes cuando salga de los bordes
Se usa la función creada en este proyecto adjust_and_crop().

```
def adjust_and_crop(image, x, y, iw, ih):
    start_x = max(0, x)
    start_y = max(0, y)

    x_offset = 0 if x >= 0 else abs(x)
    y_offset = 0 if y >= 0 else abs(y)

    h_end = min(ih, y + image.shape[0]) - start_y
    w_end = min(iw, x + image.shape[1]) - start_x

    cropped = image[y_offset:y_offset + h_end, x_offset:x_offset + w_end]

    return start_x, start_y, cropped
```
### 3. Superposición de objetos sobre el frame
Usando la función creada en este proyecto overlay_transparent().

```
def overlay_transparent(base_image, overlay, x, y):
    h, w = overlay.shape[:2]
    for i in range(3):  # Para cada canal (BGR)
        base_image[y:y+h, x:x+w, i] = \
            base_image[y:y+h, x:x+w, i] * (1 - overlay[:, :, 3] / 255.0) + \
            overlay[:, :, i] * (overlay[:, :, 3] / 255.0)
```

## Cierre
Finalmente se muestra la imagen usando opencv y se establece como cierre de la aplicación la tecla "q".
