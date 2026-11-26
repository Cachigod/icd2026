# Introducción a la Ciencia de Datos 2026

Repositorio personal del curso **Introducción a la Ciencia de Datos 2026** (Posgrado en Ciencias de la Computación, CICESE).
Autor: Erick Gabriel Santiago Suenaga.

Reúne las prácticas, tareas y el proyecto final del curso, junto con los notebooks del libro de referencia. Cada entrega está en su propia carpeta, con su notebook ejecutado, los datos que usa y un README que la explica.

## Contenido

| Carpeta | Contenido |
|---|---|
| [`project/`](project/) | **Proyecto final:** sistema de recomendación de películas con MovieLens (ml-latest-small). Compara modelos para predecir la calificación y para recomendar listas de 10 películas. |
| [`practices/`](practices/) | Prácticas del curso, una carpeta por práctica (P1 a P4). |
| [`tareas/`](tareas/) | Tareas del curso, una carpeta por tarea (T2 a T5). |
| [`book/`](book/) | Notebooks del libro *Python Data Science Handbook* (VanderPlas, 2022), organizados por bloques, con sus datos y figuras. |
| [`classes/`](classes/) | Material de clase. |

## Proyecto final

**Predecir la nota no es recomendar: sistemas de recomendación de películas con MovieLens.**

El proyecto desarrolla y evalúa un sistema que recomienda películas a cada usuario a partir de sus calificaciones y de las de otros usuarios. Se comparan 13 modelos en dos tareas:

- **Predecir la calificación** que un usuario dará a una película, medida con RMSE.
- **Recomendar una lista de 10 películas**, medida con NDCG@10.

Los modelos son:

- baselines simples, como los promedios, los sesgos de usuario y película, y las películas más calificadas;
- filtrado colaborativo con k-NN de usuarios y de películas;
- factorización de matrices (SVD);
- modelos de regresión con atributos (Ridge y gradient boosting).

A cada usuario se le esconden sus 10 calificaciones más recientes, y cada modelo se compara usuario por usuario contra el baseline de su tarea, con intervalos de confianza del 95 %.

El resultado principal es que el modelo que mejor predice la nota (gradient boosting, RMSE 0.919) no es el que mejor recomienda. Solo el k-NN de películas para listas supera a recomendar lo más popular (NDCG@10 de 0.068 contra 0.054).

| Archivo | Contenido |
|---|---|
| [`SSEG_proyecto_recomendador.ipynb`](project/SSEG_proyecto_recomendador.ipynb) | Notebook principal, ejecutado y con salidas |
| [`ml-latest-small/`](project/ml-latest-small/) | Datos de MovieLens (GroupLens), con su licencia original |
| [`figuras/`](project/figuras/) | Figuras generadas por el notebook |
| [`README.md`](project/README.md) | Descripción completa, origen de los datos, requisitos y cómo ejecutarlo |

## Prácticas

| Carpeta | Tema | Notebook | Datos |
|---|---|---|---|
| [`P1`](practices/P1/) | Análisis exploratorio y visualización de la mortalidad por accidente cerebrovascular en adultos de 35 años o más en Estados Unidos (2015–2017) | `Practica1_SSEG.ipynb` | `csv-1.csv` |
| [`P2`](practices/P2/) | Análisis exploratorio, limpieza y reducción de dimensionalidad (PCA) de los anuncios de Airbnb en la Ciudad de México (2026) | `Practica2_SSEG.ipynb` | `listings.csv` |
| [`P3`](practices/P3/) | Comparación de cinco clasificadores para predecir si un estudiante deserta, sigue inscrito o se gradúa | `Practica3_SSEG.ipynb` | `desercion-entrenamiento.csv`, `desercion-prueba.csv` |
| [`P4`](practices/P4/) | Ensambles (bagging, boosting, votación y *stacking*) sobre deserción estudiantil y Bank Marketing; el mejor modelo se guarda en `.joblib` | `Practica4_SSEG.ipynb` | Deserción y `bank-additional-full.csv` |

## Tareas

| Carpeta | Tema | Notebook | Datos |
|---|---|---|---|
| [`T2`](tareas/T2/) | Preprocesamiento: limpieza de datos y PCA, comparando los datos antes y después | `SSEG_Tarea2.ipynb` | Breast Cancer Wisconsin (Original) |
| [`T3`](tareas/T3/) | Regresión lineal usada como regresor (calidad del vino) y como clasificador (vino bueno o no), contra una línea base | `SSEG_Tarea3.ipynb` | Wine Quality (vino tinto) |
| [`T4`](tareas/T4/) | Métricas de clasificación con clases desbalanceadas: curvas ROC y de precisión-sensibilidad, umbral de decisión y fuga por la duración de la llamada | `SSEG_Tarea4.ipynb` | Bank Marketing |
| [`T5`](tareas/T5/) | Red convolucional con Global Average Pooling: más de 95 % de exactitud con menos parámetros, y prueba con dígitos desplazados | `SSEG_Tarea5.ipynb` | MNIST |

## Libro de referencia

La carpeta [`book/`](book/) contiene los notebooks de VanderPlas, J. (2022). *Python Data Science Handbook*, 2a edición, O'Reilly, agrupados en cinco bloques:

| Bloque | Tema |
|---|---|
| `Bloque_01` | IPython y Jupyter |
| `Bloque_02` | NumPy |
| `Bloque_03` | Pandas |
| `Bloque_04` | Visualización con Matplotlib y Seaborn |
| `Bloque_05` | Aprendizaje automático con scikit-learn |

Los datos y las figuras que usan esos notebooks están en `book/data/` y `book/figures/`.

## Estructura

```
.
├── README.md
├── project/          Proyecto final (notebook, datos de MovieLens y figuras)
├── practices/
│   ├── P1/ … P4/     Una carpeta por práctica: notebook, datos y README
├── tareas/
│   ├── T2/ … T5/     Una carpeta por tarea: notebook, datos y README
├── book/             Notebooks del libro, por bloques, con data/ y figures/
└── classes/          Material de clase
```

## Uso

Los notebooks se guardan con sus salidas, así que pueden leerse directamente en GitHub sin ejecutarlos. Para ejecutarlos, se abre cada notebook desde su propia carpeta, de modo que las rutas relativas a los datos funcionen sin cambios. Los requisitos, el origen de los datos y los detalles de cada entrega se describen en su propio README.
