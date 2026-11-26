# Predecir la nota no es recomendar: sistemas de recomendación de películas con MovieLens

Proyecto final del curso **Introducción a la Ciencia de Datos** (CICESE).

## Descripción

El proyecto desarrolla y evalúa un sistema que recomienda películas a cada usuario a partir de sus calificaciones y de las de otros usuarios. Se comparan 13 modelos en dos tareas con una misma partición de los datos:

| Tarea | Pregunta | Métrica principal |
|---|---|---|
| 1. Predecir la nota | ¿Qué calificación dará el usuario a una película? | RMSE (en estrellas) |
| 2. Recomendar 10 películas | ¿La lista incluye películas que el usuario después vio y le gustaron? | NDCG@10 |

**Modelos:**
- **Siete baselines:** aleatorio, promedio global, promedio del usuario, promedio de la película, sesgos usuario + película, más calificadas y mejor promedio ajustado.
- **Filtrado colaborativo:** k-NN de usuarios y k-NN de películas sobre residuos, y k-NN de películas para listas.
- **Factorización de matrices:** SVD con sesgos.
- **Modelos con atributos:** regresión Ridge y gradient boosting.
- **Análisis complementario:** K-means sobre los factores del SVD.

**Evaluación:**
- **Partición:** a cada usuario se le esconden sus 10 calificaciones más recientes (test). Las 5 anteriores sirven para elegir hiperparámetros (validación).
- **Comparación:** cada modelo se compara usuario por usuario contra el baseline de su tarea, con intervalos de confianza del 95 %.

**Resultados principales:**
- **Tarea 1:** solo el gradient boosting predice la nota mejor que el modelo de sesgos (RMSE 0.919 contra 0.944).
- **Tarea 2:** solo el k-NN de películas para listas recomienda mejor que lo más popular (NDCG@10 de 0.068 contra 0.054).
- **Conclusión general:** el modelo que mejor predice la nota no es el que mejor recomienda.

## Origen de los datos

Se usa **MovieLens ml-latest-small** de GroupLens Research (Universidad de Minnesota), en su versión de septiembre de 2018: 100,836 calificaciones (de 0.5 a 5 estrellas) de 610 usuarios sobre 9,742 películas, registradas entre 1996 y 2018.

- **Descarga:** https://grouplens.org/datasets/movielens/ (archivo directo: https://files.grouplens.org/datasets/movielens/ml-latest-small.zip).
- **Archivos usados:** `ratings.csv` (usuario, película, calificación y fecha) y `movies.csv` (título con año y géneros). `tags.csv` y `links.csv` no se usan.
- **Licencia:** permite redistribuir el conjunto bajo las mismas condiciones, con fines no comerciales y citando la fuente. Por eso la carpeta `ml-latest-small/` se incluye en este repositorio sin modificaciones, con su `README.txt` original, que contiene la licencia completa. Pesa menos de 4 MB.
- **Cita:** F. Maxwell Harper y Joseph A. Konstan. 2015. *The MovieLens Datasets: History and Context*. ACM Transactions on Interactive Intelligent Systems (TiiS) 5, 4: 19:1–19:19. https://doi.org/10.1145/2827872

Si la carpeta no estuviera, se descarga `ml-latest-small.zip` del enlace anterior y se descomprime junto al notebook, de modo que quede `ml-latest-small/ratings.csv`.

## Requisitos

Python 3.12 con estas librerías. Son las versiones con las que se obtuvieron los resultados; el notebook las imprime en la sección 0.

| Librería | Versión |
|---|---|
| numpy | 2.5.5 |
| pandas | 3.1.1 |
| scipy | 1.18.1 |
| scikit-learn | 1.9.1 |
| matplotlib | 3.11.2 |
| numba (opcional) | 0.68.1 |

```bash
pip install numpy==2.5.5 pandas==3.1.1 scipy==1.18.1 scikit-learn==1.9.1 matplotlib==3.11.2 numba==0.68.1 jupyter
```

`numba` solo acelera el entrenamiento del SVD; sin él los resultados son los mismos, pero esa parte tarda más.

## Cómo ejecutarlo

1. Clonar el repositorio. La carpeta `ml-latest-small/` debe quedar al mismo nivel que el notebook.
2. Abrir `SSEG_proyecto_recomendador.ipynb` en Jupyter o Visual Studio Code.
3. Ejecutar todas las celdas en orden. La ejecución completa tarda entre 3 y 10 minutos en una computadora personal; no requiere GPU.

**Reproducibilidad:**
- **Rutas:** todas son relativas.
- **Aleatoriedad:** todo lo aleatorio usa la semilla `SEMILLA = 42`, así que cada ejecución produce las mismas cifras.
- **Figuras:** se guardan en `figuras/`; el notebook crea la carpeta si no existe.

## Estructura de archivos

```
.
├── README.md                          Este archivo
├── SSEG_proyecto_recomendador.ipynb   Notebook principal, ejecutado y con salidas
├── figuras/                           Figuras generadas por el notebook (se vuelven a crear al ejecutarlo)
│   ├── fig_exploracion.png            Distribución de notas y calificaciones por usuario y por película
│   ├── fig_importancia.png            Importancia de atributos del gradient boosting
│   ├── fig_rmse_ndcg.png              RMSE contra NDCG@10 y NDCG@10 contra popularidad
│   ├── fig_ic.png                     Diferencias pareadas con intervalos de confianza del 95 %
│   ├── fig_popularidad.png            NDCG@10 contra popularidad de lo recomendado
│   └── fig_grupos.png                 NDCG@10 por tercio de usuarios según su historial
└── ml-latest-small/                   Datos de MovieLens (GroupLens), sin modificar
    ├── ratings.csv
    ├── movies.csv
    ├── tags.csv
    ├── links.csv
    └── README.txt                     Licencia y descripción original de GroupLens
```

## Organización del notebook

| Etapa | Secciones | Contenido |
|---|---|---|
| Datos | 0 | Carga, entorno y versiones |
| Análisis exploratorio | 1 | Cifras generales y distribuciones |
| Preprocesamiento | 2–3 | Limpieza, variables derivadas y partición sin fuga de información |
| Evaluación (definición) | 4 | Métricas de las dos tareas, probadas con casos conocidos |
| Modelado | 5–8 | Baselines, k-NN, SVD y modelos con atributos |
| Evaluación (resultados) | 9 | Tablas finales, comparación pareada, grupos de usuarios y películas nuevas |
| Análisis complementario | 10–11 | K-means y análisis de errores con un usuario |
| Conclusiones | 12 | Respuestas a las preguntas de investigación y limitaciones |

La última celda contiene la declaración de uso de IA generativa.
