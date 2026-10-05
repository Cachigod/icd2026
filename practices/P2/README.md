# Procesamiento de Datasets: Anuncios de Airbnb en la Ciudad de México (2026)

Análisis exploratorio de datos, limpieza y reducción de dimensionalidad
(PCA) del catálogo de **alojamientos de corta estancia publicados en
Airbnb en la Ciudad de México**, con datos capturados el 15 de junio de
2026.

El proyecto fue desarrollado en Jupyter Notebook y utiliza datos
abiertos publicados por **Inside Airbnb**, un proyecto independiente que
compila la información pública del sitio de Airbnb. Inside Airbnb no
tiene afiliación con Airbnb.

## Contenido del proyecto

-   `Practica2_SSEG.ipynb`: notebook con la carga, exploración,
    visualización, análisis y procesamiento de los datos.
-   `listings.csv`: dataset utilizado por el notebook (versión resumen
    del archivo de listados).

## Descripción del dataset

El archivo `listings.csv` describe cada anuncio con su ubicación (alcaldía
y coordenadas), tipo de alojamiento, precio por noche, estancia mínima,
disponibilidad en el calendario, actividad de reseñas y número de
anuncios del anfitrión.

Los datos son una captura de la información pública del sitio en una
fecha concreta. Por eso, el precio es el **precio publicado**, no el
que se pagó, y un anuncio puede haber cambiado o desaparecido después
de la captura.

### Unidad de observación

Cada fila representa **un anuncio (*listing*) de Airbnb** en la Ciudad
de México. Los registros no son personas ni reservas.

### Dimensiones

El archivo contiene 31,430 registros y 19 columnas. Después de la
limpieza quedan 29,543 registros y 15 columnas.

Entre las variables más importantes se encuentran:

| Variable | Descripción |
|---|---|
| `id` | Identificador único del anuncio. |
| `host_id` | Identificador del anfitrión. |
| `neighbourhood` | Alcaldía del anuncio. |
| `latitude`, `longitude` | Coordenadas del anuncio. |
| `room_type` | Tipo de alojamiento: casa o departamento completo, cuarto privado, cuarto compartido o cuarto de hotel. |
| `price` | Precio por noche en pesos mexicanos. |
| `minimum_nights` | Estancia mínima en noches. |
| `number_of_reviews` | Total de reseñas del anuncio. |
| `number_of_reviews_ltm` | Reseñas de los últimos 12 meses. |
| `reviews_per_month` | Reseñas promedio por mes. |
| `last_review` | Fecha de la última reseña. |
| `calculated_host_listings_count` | Número de anuncios del anfitrión en la ciudad. |
| `availability_365` | Noches disponibles en el calendario durante los próximos 365 días. |

El archivo incluye además `name`, `host_name`, `host_profile_id`,
`neighbourhood_group` y `license`. Las dos últimas vienen
completamente vacías.

## Objetivo del análisis

El notebook realiza un Análisis Exploratorio de Datos para conocer la
estructura y calidad del dataset y explorar patrones en los precios, y
después aplica dos técnicas de procesamiento con comparaciones antes y
después.

Entre los principales aspectos analizados se encuentran:

1.  Dimensiones, tipos de datos y verificación de requisitos.
2.  Valores faltantes, registros duplicados y valores imposibles.
3.  Cardinalidad de las variables categóricas.
4.  Distribución del precio y de las variables numéricas.
5.  Distribución por tipo de alojamiento y alcaldía.
6.  Comparación del precio según tipo de alojamiento, alcaldía y tamaño
    del anfitrión.
7.  Correlación entre variables numéricas.
8.  Distribución geográfica de los precios.
9.  Limpieza de datos.
10. Reducción de dimensionalidad con PCA.

`price` puede utilizarse como variable objetivo para un problema de
regresión. También podría transformarse en una variable categórica
para plantear un problema de clasificación, por ejemplo, separando
anuncios por encima y por debajo de la mediana.

## Procesamiento aplicado

### 1. Limpieza de datos

La limpieza se aplica sobre una copia de los datos. El notebook conserva
el dataset original para poder comparar antes y después.

| Problema | Acción |
|---|---|
| `neighbourhood_group` y `license` 100% vacías | Se eliminan las columnas. |
| `host_profile_id` (pierde exactitud como `float64`) y `host_name` (nombre personal) | Se eliminan las columnas. |
| 1,793 anuncios sin `price` | Se eliminan las filas. |
| Precios extremos (78 anuncios) | Se eliminan los que quedan fuera de 3 veces el IQR en escala logarítmica (de 57 a 53,630 MXN). |
| 16 anuncios con más de 31 reseñas por mes | Se eliminan las filas. |
| `reviews_per_month` vacío en anuncios sin reseñas | Se rellena con 0. |
| 11 faltantes en `minimum_nights` | Se rellenan con la mediana. |
| `last_review` como texto | Se convierte a fecha. |


### 2. Reducción de dimensionalidad (PCA)

Se aplica PCA sobre seis variables numéricas de actividad, reglas del
anuncio y escala del anfitrión, después de una transformación `log1p` y
una estandarización. `price`, las coordenadas y los identificadores
quedan fuera.

El resultado son **3 componentes que conservan 80.6% de la varianza** y
no están correlacionadas entre sí.

## Hallazgos exploratorios principales

El notebook identifica, entre otros, los siguientes patrones:

-   El dataset contiene 31,430 registros, superando ampliamente el
    requisito mínimo de registros y atributos para el análisis.
-   El precio por noche está muy sesgado a la derecha: la mediana es
    1,756 MXN, la media 2,914 MXN y el máximo 1,141,520 MXN.
-   La oferta está muy concentrada: 66.8% de los anuncios son casas o
    departamentos completos, y tres alcaldías (Cuauhtémoc, Miguel
    Hidalgo y Benito Juárez) reúnen 73.0% de la oferta.
-   El precio cambia con el tipo de alojamiento y la alcaldía. Las
    medianas van de 503 MXN (cuarto compartido) a 2,163 MXN (alojamiento
    completo), y de 628 MXN (Tláhuac) a 2,212 MXN (Miguel Hidalgo).
-   El precio casi no se relaciona con las variables numéricas
    (correlaciones de Spearman entre −0.04 y 0.06), salvo con la
    longitud (−0.22). El mapa muestra precios más altos en el centro y
    el poniente.
-   Las tres variables de reseñas (`number_of_reviews`,
    `number_of_reviews_ltm` y `reviews_per_month`) son redundantes entre
    sí.
-   El 43.9% de los anuncios pertenece a anfitriones con cinco o más
    anuncios, que mantienen su calendario más abierto.
-   Los identificadores (`id`, `host_id`) son números y no deben
    interpretarse como cantidades continuas.

## Calidad y limitaciones de los datos

El dataset es un conjunto de datos reales y presenta características que
deben considerarse antes de realizar análisis estadísticos o modelos
predictivos.

### Valores faltantes

-   `neighbourhood_group` y `license` están 100% vacías.
-   `last_review` y `reviews_per_month` faltan en 5,499 anuncios (17.5%).
    Coinciden uno a uno con los anuncios sin reseñas, por lo que no son
    datos perdidos sino información que todavía no existe.
-   `price` falta en 1,793 anuncios (5.7%). No es un faltante aleatorio:
    el 71.3% de esos anuncios tiene cero noches disponibles, frente a
    0.05% de los que sí tienen precio. Por eso se eliminan y no se
    imputan.

### Valores extremos

Hay 79 anuncios con precios mayores a 50,000 MXN por noche, incluido un
cuarto privado de 1,141,520 MXN que probablemente es un error de
captura. También hay 16 anuncios con más de 31 reseñas por mes (máximo
121.89), un valor poco creíble para un solo alojamiento.

### Registros casi duplicados

No hay duplicados exactos ni `id` repetidos. Al ignorar el `id`
aparecen 10 registros idénticos. Todos tienen cero reseñas y pertenecen a
anfitriones con varios anuncios, por lo que probablemente son unidades
iguales de un mismo edificio u hotel. No se eliminan.

### Alcance de los datos

-   Es una captura de un solo momento (15 de junio de 2026).
-   Este archivo resumen no incluye características de la propiedad
    como habitaciones, huéspedes o servicios, que sí están en el archivo
    detallado `listings.csv.gz`.
-   Los cuartos de hotel (86), los cuartos compartidos (301) y alcaldías
    como Milpa Alta (26) y Tláhuac (49) tienen pocos casos, y sus
    estadísticas son poco estables.

## Requisitos de ejecución

Se necesita un entorno con:

-   Python 3.x
-   Jupyter Notebook o JupyterLab
-   `numpy`
-   `pandas`
-   `matplotlib`
-   `seaborn`
-   `scikit-learn` (para el PCA)

### Instalación

Desde una terminal, dentro de la carpeta del proyecto:

``` bash
pip install numpy pandas matplotlib seaborn scikit-learn jupyter
```

## Instrucciones de uso

### 1. Descargar o clonar el repositorio

Clona el repositorio y entra a la carpeta de esta práctica:

``` bash
git clone https://github.com/Cachigod/icd2026/
cd icd2026
```

### 2. Verificar los archivos

La estructura esperada es:

``` text
.
├── Practica2_SSEG.ipynb
├── README.md
└── listings.csv
```

Es importante que `listings.csv` se encuentre en la misma carpeta que
el notebook.

Si no está incluido, puede descargarse directamente de:

https://data.insideairbnb.com/mexico/df/mexico-city/2026-06-15/visualisations/listings.csv

Debe ser el archivo `listings.csv` de la carpeta `visualisations`
(resumen), no el archivo detallado `listings.csv.gz`.

### 3. Iniciar Jupyter

Ejecuta:

``` bash
jupyter notebook
```

o:

``` bash
jupyter lab
```

### 4. Abrir el notebook

Abre:

``` text
Practica2_SSEG.ipynb
```

## Fuentes de los datos

**Fuente original:**

Inside Airbnb. (2026, 15 de junio). *Mexico City, listings.csv (resumen)*
[Conjunto de datos].

https://data.insideairbnb.com/mexico/df/mexico-city/2026-06-15/visualisations/listings.csv

**Página de descarga:**

https://insideairbnb.com/get-the-data/

**Diccionario de datos:**

https://docs.google.com/spreadsheets/d/1iWCNJcSutYqpULSQHlNyGInUvHg2BoUGoNRIGa6Szc4/edit?usp=sharing

**Licencia:**

Creative Commons Attribution 4.0 International (CC BY 4.0), según la
página de descarga consultada en octubre de 2026.

https://creativecommons.org/licenses/by/4.0/

**Fecha de recolección:** 15 de junio de 2026.
