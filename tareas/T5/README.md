# Tarea 5: CNN con Global Average Pooling y dígitos desplazados en MNIST

**Introducción a la Ciencia de Datos 2026**  
Posgrado en Ciencias de la Computación — CICESE

Búsqueda de una red neuronal convolucional (CNN) con ***Global Average Pooling*** (GAP) que supere el **95 % de exactitud en validación** con **menos de 54,330 parámetros entrenables**, es decir, con menos parámetros que la CNN con *Flatten* vista en clase. Entre las variantes que lo logran se elige la de menos parámetros, y después se mide qué tan bien reconoce los dígitos de **MNIST** cuando se mueven de su posición, comparada con las dos redes base de la clase.

## Contenido del proyecto

**Necesarios para ejecutar el notebook**

- `SSEG_Tarea5.ipynb`: notebook principal. Recupera los dos modelos de referencia, entrena doce variantes con GAP, selecciona una usando solo validación y compara los tres modelos en prueba con desplazamientos de 0, 2 y 4 píxeles.

**No se necesita ningún archivo de datos.** MNIST lo descarga Keras la primera vez que se ejecuta el notebook (`keras.datasets.mnist.load_data()`) y lo guarda en una carpeta local de caché, así que esa primera ejecución requiere conexión a internet.

**Documentación**

- `README_Tarea5.md`: este archivo.

**No se genera ningún archivo.** El notebook solo muestra tablas y gráficas; no guarda modelos ni resultados en disco.

## Descripción del conjunto de datos

| | MNIST |
|---|---|
| **Fuente** | LeCun, Bottou, Bengio y Haffner (1998), disponible a través de `keras.datasets.mnist` |
| **Unidad de observación** | Una imagen de un dígito manuscrito |
| **Tamaño original** | 60,000 imágenes de entrenamiento y 10,000 de prueba |
| **Imagen** | 28 × 28 píxeles, escala de grises (valores de 0 a 255) |
| **Variable objetivo** | Dígito del 0 al 9 (diez clases) |
| **Métrica** | Exactitud (*accuracy*) |

### Particiones que usa el notebook

Son las mismas particiones estratificadas del notebook de clase (semilla 42):

| Partición | Imágenes | De dónde salen | Para qué se usa |
|---|---:|---|---|
| Entrenamiento | 8,000 | Conjunto oficial de entrenamiento | Ajustar los pesos de todas las redes |
| Validación | 1,500 | Conjunto oficial de entrenamiento, distintas de las anteriores | Todas las decisiones de arquitectura |
| Prueba | 2,000 | Conjunto oficial de prueba | Una sola vez, en la comparación final con desplazamientos |

A cada imagen se le agrega un **margen negro de 4 píxeles** por lado, así que la red recibe imágenes de **36 × 36 × 1**, con valores escalados a [0, 1]. Ese margen permite mover el dígito hasta 4 píxeles en cualquier dirección sin recortar el trazo.

## Objetivo del análisis

Responder tres preguntas:

1. ¿Cuál fue el cambio más importante para que una CNN con GAP superara el 95 % en validación?
2. ¿Cuántos parámetros se ahorran frente a la CNN con *Flatten*?
3. ¿Reconoce mejor los dígitos desplazados que *Flatten*, y a partir de qué distancia se nota la diferencia?

## Procesamiento y metodología

**Los modelos de referencia.** Las dos redes base comparten el extractor: dos convoluciones de 16 filtros de 3 × 3 con ReLU y `padding="same"`, y un *max pooling* de 2 × 2, que deja mapas de 18 × 18 × 16. Solo cambia cómo llegan esos mapas a la capa de diez clases:

- **Flatten** entrega 5,184 valores y la capa densa necesita 51,850 pesos (54,330 parámetros en total).
- **GAP** promedia cada canal sobre todo el mapa y entrega 16 valores, por lo que la capa densa necesita solo 170 pesos (2,650 parámetros en total).

**Condiciones de entrenamiento.** Son las del notebook de clase y no cambian entre modelos: semilla 42, lotes de 128, optimizador Adam con tasa de 0.001 y **20 épocas**. Así, cualquier diferencia entre variantes se debe a la arquitectura y no al entrenamiento.

**Las doce variantes con GAP.** Todas terminan en `GlobalAveragePooling2D` y una capa `Dense(10)` con softmax. Cambia el extractor, que se escribe como una secuencia de bloques: un número es una convolución con ese número de filtros (3 × 3, salvo que se indique), y `P` es un *max pooling* de 2 × 2. Por ejemplo, `4-P-8-P-16-P-16-P` es conv(4) → pool → conv(8) → pool → conv(16) → pool → conv(16) → pool → GAP → Dense(10). La tabla de resultados incluye el **campo receptivo**: el lado, en píxeles de la entrada, de la región que influye en una neurona del último mapa.

Las variantes forman dos familias. De `V0` a `V3` se conservan 16 filtros por capa y solo se agregan convoluciones o se mueve el pooling. De `V5` a `V11` se parte de `V4` y se cambia una sola cosa a la vez: quitar el último pooling, usar kernels de 5 × 5, quitar la cuarta etapa o cambiar el número de filtros.

**Criterio de selección.** Una variante «cumple» si su exactitud de validación **al final de la época 20** (no la mejor época) es **mayor que 95 %** y tiene **menos de 54,330 parámetros**. Entre las que cumplen se elige la de menos parámetros, y la decisión se toma solo con validación.

**Efecto del desplazamiento.** Con los tres modelos ya entrenados (sin reentrenar) se evalúan las 2,000 imágenes de prueba movidas 0, 2 y 4 píxeles hacia arriba, abajo, izquierda y derecha, y se promedia la exactitud de las cuatro direcciones para 2 y 4 píxeles. Se reporta la caída Acc(0 px) − Acc(4 px) y también qué fracción de su exactitud conserva cada modelo a 4 px, porque un modelo puede caer poco simplemente porque nunca clasificó bien.

## Resultados principales

Los resultados guardados en el notebook muestran lo siguiente.

### Validación de las doce variantes con GAP (exactitud en la época 20)

| Variante | Arquitectura | Campo receptivo (px) | Parámetros | Validación (%) | Cumple |
|---|---|---:|---:|---:|:---:|
| Flatten | 16-16-P + Flatten | 6 | 54,330 | 96.80 | — |
| V0 | 16-16-P (GAP base) | 6 | 2,650 | 43.93 | No |
| V1 | 16-16-16-P | 8 | 4,970 | 53.53 | No |
| V2 | 16-P-16-P-16-P | 22 | 4,970 | 90.20 | No |
| V3 | 16-P-16-P-16-P-16-P | 46 | 7,290 | 95.33 | Sí |
| **V4** | **4-P-8-P-16-P-16-P** | **46** | **3,994** | **95.60** | **Sí** |
| V5 | 4-P-8-P-16-P-16 | 38 | 3,994 | 92.73 | No |
| V6 | 4-P-8-P-16-P-16-P (5 × 5) | 76 | 10,714 | 95.60 | Sí |
| V7 | 4-P-8-P-16-P | 22 | 1,674 | 85.13 | No |
| V8 | 4-P-8-P-16-P-12-P | 46 | 3,374 | 94.47 | No |
| V9 | 3-P-6-P-12-P-12-P | 46 | 2,296 | 94.20 | No |
| V10 | 8-P-8-P-16-P-16-P | 46 | 4,322 | 95.80 | Sí |
| V11 | 6-P-12-P-24-P-32-P | 46 | 10,610 | 97.20 | Sí |

Cinco variantes cumplen los dos requisitos (`V3`, `V4`, `V6`, `V10` y `V11`). La seleccionada es **`V4 · 4-P-8-P-16-P-16-P`**, con **95.60 %** de exactitud en validación y **3,994 parámetros entrenables**. Frente a los 54,330 de *Flatten* ahorra **50,336 parámetros (92.6 % menos)**: *Flatten* es 13.6 veces más grande.

### Qué cambió la exactitud (comparaciones de una sola decisión)

| Decisión | Comparación | Parámetros | Cambio en validación |
|---|---|---|---:|
| Ubicación del pooling | `V1` → `V2` | 4,970 → 4,970 | **+36.67 pp** |
| Profundidad con pooling al final | `V0` → `V1` | 2,650 → 4,970 | +9.60 pp |
| Profundidad con pooling intercalado | `V2` → `V3` | 4,970 → 7,290 | +5.13 pp |
| Profundidad con pooling intercalado | `V7` → `V4` | 1,674 → 3,994 | +10.47 pp |
| Pooling antes de GAP | `V5` → `V4` | 3,994 → 3,994 | +2.87 pp |
| Tamaño del kernel | `V4` → `V6` | 3,994 → 10,714 | 0.00 pp |
| Filtros en todas las capas | `V3` → `V4` | 7,290 → 3,994 | +0.27 pp |
| Filtros en la última capa | `V8` → `V4` | 3,374 → 3,994 | +1.13 pp |

- **Intercalar el pooling es el cambio más importante.** `V1` y `V2` tienen las mismas tres convoluciones y los mismos parámetros, pero con el pooling intercalado el campo receptivo pasa de 8 a 22 píxeles y la exactitud sube 36.67 puntos porcentuales.
- **Cuatro etapas de convolución y pooling** llevan el campo receptivo a 46 píxeles, más que el ancho de la imagen de 36 × 36, y es lo que lleva a la red por encima del 95 %.
- **Las primeras capas necesitan pocos filtros** (4 y 8 bastan), pero la última no: sus canales son los valores que recibe la capa de salida, y bajarla a 12 filtros (`V8`) o estrechar toda la red (`V9`) deja la exactitud debajo de 95 %.
- **Kernels de 5 × 5 no ayudan:** misma exactitud con 2.7 veces más parámetros.

### Desplazamiento de los dígitos (conjunto de prueba, 2,000 imágenes)

| Modelo | Parámetros | 0 px (%) | 2 px (%) | 4 px (%) | Caída 0 → 4 px (pp) | Conserva a 4 px |
|---|---:|---:|---:|---:|---:|---:|
| CNN + Flatten | 54,330 | 97.10 | 82.80 | 37.30 | 59.80 | 38.4 % |
| CNN + GAP base | 2,650 | 46.25 | 46.25 | 46.25 | 0.00 | 100.0 % |
| GAP seleccionado (`V4`) | 3,994 | 94.90 | 87.85 | 63.25 | 31.65 | 66.7 % |

Las exactitudes a 2 y 4 px son el promedio de las cuatro direcciones.

- **Con los dígitos centrados, *Flatten* es mejor** (97.10 % contra 94.90 %), pero **desde 2 píxeles el orden se invierte**: `V4` obtiene 87.85 % y *Flatten* 82.80 %. A 4 píxeles la diferencia es de 25.95 pp a favor de `V4` (63.25 % contra 37.30 %).
- **El GAP base no cambia nada, pero porque nunca acierta:** su caída es exactamente cero, ya que con un solo pooling de paso 2 los desplazamientos de 2 y 4 píxeles solo trasladan el mapa que llega a GAP, y su promedio no cambia. Se queda en 46.25 % en todas las distancias.
- **`V4` tampoco es invariante** al desplazamiento: sus cuatro poolings acumulan un paso de 16 píxeles, y 2 o 4 no son múltiplos de 16. Por eso su pérdida depende de la dirección; a 4 px conserva 80.75 % hacia la izquierda, pero solo 43.90 % hacia arriba.
- En 570 de las 2,000 imágenes de prueba, *Flatten* acierta el dígito centrado, falla al moverlo 4 píxeles a la derecha y `V4` sigue acertando.

### Respuestas a las tres preguntas

1. **Cambio más importante:** intercalar el *max pooling* entre las convoluciones en lugar de dejarlo solo al final (+36.67 pp con los mismos parámetros), y sobre eso, agregar una cuarta etapa conv + pool con el último pooling justo antes de GAP.
2. **Ahorro de parámetros:** 50,336 menos (92.6 %). Casi todo viene de la capa de salida, que pasa de 51,850 a 170 pesos.
3. **Dígitos desplazados:** sí, `V4` reconoce mejor los dígitos desplazados que *Flatten*, y la diferencia se aprecia desde los **2 píxeles**.

## Requisitos de ejecución

El notebook imprime las versiones con las que se obtuvieron los resultados:

| Biblioteca | Versión |
|---|---|
| NumPy | 2.5.5 |
| pandas | 3.1.1 |
| Matplotlib | 3.11.2 |
| scikit-learn | 1.9.1 |
| TensorFlow (incluye Keras) | 2.21.0 |

Además se usa `IPython` (para `display`), que viene con Jupyter. Se necesita una versión de Python compatible con TensorFlow 2.21. Para instalar las bibliotecas desde una terminal:

```bash
pip install numpy pandas matplotlib scikit-learn tensorflow jupyter
```

Si quieres reproducir los resultados con exactitud, instala las versiones indicadas en la tabla (por ejemplo, `pip install tensorflow==2.21.0`). El entrenamiento es en CPU y no necesita GPU.

## Instrucciones de uso

1. Coloca `SSEG_Tarea5.ipynb` en una carpeta e instala las dependencias.
2. Inicia Jupyter desde esa carpeta:

   ```bash
   jupyter notebook
   ```

   o:

   ```bash
   jupyter lab
   ```

3. Abre `SSEG_Tarea5.ipynb` y ejecuta todas las celdas en orden, desde el inicio (*Restart & Run All*). En la primera ejecución Keras descarga MNIST, por lo que hace falta conexión a internet.
4. Revisa las tablas y las gráficas de cada sección: modelos de referencia, variantes con GAP, selección y efecto del desplazamiento.

El notebook entrena 13 redes pequeñas (las dos de referencia y las variantes `V1` a `V11`; `V0` reutiliza el GAP base) durante 20 épocas cada una. Todas las rutas son relativas y las semillas están fijas, pero las exactitudes pueden variar unas décimas entre ejecuciones y versiones de TensorFlow. Por ejemplo, en el notebook *Flatten* obtuvo 96.80 % en validación frente a 96.87 % en la ejecución de clase, y el GAP base 43.93 % frente a 44.07 %. Los parámetros coinciden exactamente (54,330 y 2,650).

## Limitaciones

- **Una sola ejecución por variante.** Cada arquitectura se entrenó una vez con una semilla, así que diferencias de unas décimas entre variantes no son concluyentes.
- **La selección se queda con poco margen.** `V4` supera el umbral con 95.60 % contra 95 %, unas 9 imágenes de validación sobre 1,500. En prueba obtuvo 94.90 %, debajo del 95.60 % de validación; con 1,500 y 2,000 imágenes una diferencia así es esperable. `V10` (95.80 %) y `V11` (97.20 %) tienen más margen, pero la regla de la tarea es quedarse con la de menos parámetros.
- **Entrenamiento con un subconjunto.** Se usan 8,000 imágenes del conjunto oficial de entrenamiento, no las 60,000.
- **Alcance de los desplazamientos.** Solo se miden 0, 2 y 4 píxeles en cuatro direcciones, sin rotaciones ni cambios de escala. Los dígitos de MNIST están centrados y limpios, y las conclusiones sobre la sensibilidad a la posición no necesariamente se generalizan a imágenes más complejas.
- **Exactitud en valores absolutos.** La exactitud de 94.90 % de `V4` con dígitos centrados queda por debajo de *Flatten* (97.10 %); la ventaja de GAP está en el menor número de parámetros y en la robustez al desplazamiento, no en la exactitud con dígitos centrados.
- **Dependencia de versiones.** Los resultados dependen de la versión de TensorFlow y del orden de las operaciones en CPU.

## Fuentes

- LeCun, Y., Bottou, L., Bengio, Y., & Haffner, P. (1998). Gradient-based learning applied to document recognition. *Proceedings of the IEEE, 86*(11), 2278-2324.
- Lin, M., Chen, Q., & Yan, S. (2014). Network in network. *International Conference on Learning Representations (ICLR)*. https://arxiv.org/abs/1312.4400
- Zhang, R. (2019). Making convolutional networks shift-invariant again. *Proceedings of the 36th International Conference on Machine Learning (ICML)*, 7324-7334.
- Azulay, A., & Weiss, Y. (2019). Why do deep convolutional networks generalize so poorly to small image transformations? *Journal of Machine Learning Research, 20*(184), 1-25.
- Keras: [Conv2D](https://keras.io/api/layers/convolution_layers/convolution2d/), [MaxPooling2D](https://keras.io/api/layers/pooling_layers/max_pooling2d/) y [GlobalAveragePooling2D](https://keras.io/api/layers/pooling_layers/global_average_pooling2d/).
