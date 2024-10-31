# Práctica 4. Detección de personas, coches y matrículas usando YOLO

En esta práctica hemos implementado dos detectores de objetos basados en YOLOv11, la versión más reciente del modelo de Ultralytics, con el objetivo de detectar personas, coches y sus respectivas matrículas en archivos de vídeo. El resultado de la aplicación de estos dos modelos será un vídeo con:

- Seguimiento y rectángulo contenedor de cada uno de los elementos identificados.
- Reconocimiento de matrículas de los coches detectados.
- Lectura (intento) de las matrículas detectadas usando la librería *pytesseract* como implementación de un OCR.
- Volcado del resultado de la detección en un nuevo archivo de vídeo [**detection_result.mp4**](https://alumnosulpgc-my.sharepoint.com/:v:/g/personal/juan_quesada108_alu_ulpgc_es/EWA-sMiBxhJBjdghSf3fcJEBhV6w6ZEZ79TNMb4utaqfyg?nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJPbmVEcml2ZUZvckJ1c2luZXNzIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXciLCJyZWZlcnJhbFZpZXciOiJNeUZpbGVzTGlua0NvcHkifX0&e=0rW4DC) reflejando los elementos anteriores. Acceso al vídeo a través de OneDrive debido al tamaño.
- Generación de un archivo CSV (**results.csv**) con un análisis de las detecciones por fotograma con el siguiente formato.

```
fotograma, tipo_objeto, confianza, identificador_tracking, x1, y1, x2, y2, matrícula_en_su_caso, confianza, mx1,my1,mx2,my2, texto_matricula
```

## Autores

- Óscar Muñoz Hidalgo
- Juan José Quesada Acosta

## Contenidos

[Entrenando nuestro propio detector de matrículas](#entrenando-nuestro-propio-detector-de-matrículas)  

[Hora de combinar modelos](#hora-de-combinar-modelos)

[Introduciendo el detector de matrículas](#introduciendo-el-detector-de-matrículas)

- [Transformaciones de la imagen para una mejor lectura](#transformaciones-de-la-imagen-para-una-mejor-lectura)

- [El dilema de pytesseract](#el-dilema-de-pytesseract)

[Análisis de los resultados](#análisis-de-los-resultados)

- [Pruebas con otros vídeos](#pruebas-con-otros-vídeos)

## Paquetes a instalar

```
pip install toooodoslospaquetes
```

## Entrenando nuestro propio detector de matrículas

Para la detección de matrículas decidimos optar por entrenar nuestro propio modelo de YOLOv11. Tomando [este](https://www.kaggle.com/datasets/fareselmenshawii/large-license-plate-dataset) dataset, construimos un subset del mismo reduciéndolo a un 1/5 de su tamaño y tomando aletoriamente pares imagen-anotación para evitar posibles sesgos debidos al orden de las imágenes en el dataset original.

Con ello, lanzamos un proceso de entrenamiento tomando como base el modelo yolov11n y entrenando con dicho subset durante 40 épocas.

El archivo con los pesos resultantes puede encontrarse en la ruta:

*yolov11-plates/weights/yolov11-plates-best.pt*

El resultado del entrenamiento puede observarse a continuación:

![grafica-entrenamiento](./yolov11-plates/results.png)

Algunos ejemplos de las imágenes del subconjunto de validación durante el entrenamiento del modelo:

![ejemeplo-entrenamiento](./yolov11-plates/val_batch1_labels.jpg)

## Hora de combinar modelos

## Introduciendo el detector de matrículas

### Transformaciones de la imagen para una mejor lectura

### El dilema de pytesseract

## Análisis de los resultados

### Pruebas con otros vídeos
