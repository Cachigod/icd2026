# Análisis Exploratorio de Mortalidad por Accidente Cerebrovascular (2015-2017)

Análisis exploratorio de datos y visualización de la **tasa de
mortalidad por accidente cerebrovascular (ACV / stroke)** en adultos de
35 años o más en Estados Unidos durante el periodo 2015-2017.

El proyecto fue desarrollado en Jupyter Notebook y utiliza datos reales
publicados por los **Centers for Disease Control and Prevention (CDC)**
a través del National Vital Statistics System (NVSS).

## Contenido del proyecto

-   `Practica1_SSEG.ipynb`: notebook con la carga,
    exploración, visualización y análisis de los datos.
-   `csv-1.csv`: dataset utilizado por el notebook.

## Descripción del dataset

El conjunto de datos **Stroke Mortality Data Among US Adults (35+) by
State/Territory and County -- 2015-2017** contiene tasas de mortalidad
por accidente cerebrovascular, ajustadas por edad y expresadas como
muertes por cada 100,000 habitantes.

Los datos corresponden a un promedio de tres años (2015-2017) y fueron
recopilados a partir del National Vital Statistics System (NVSS) y
publicados por la División para la Prevención de Enfermedades del
Corazón y el Accidente Cerebrovascular (DHDSP) de los CDC.

### Unidad de observación

Cada fila representa una combinación de:

-   Área geográfica: nación, estado / territorio o condado.
-   Sexo: `Female`, `Male` u `Overall`.
-   Grupo racial/étnico: `Hispanic`, `White`, `Black`,
    `Asian/Pacific Islander`, `American Indian/Alaska Native` u
    `Overall`.

Por lo tanto, los registros no representan personas individuales.
Cada observación es una tasa agregada para un área y subgrupo
poblacional.

### Dimensiones

El archivo contiene 59,094 registros y 24 columnas. El notebook
elimina la columna `index` durante la carga porque es redundante con el
índice de pandas, por lo que el análisis utiliza 23 atributos.

Entre las variables más importantes se encuentran:

  -----------------------------------------------------------------------
  Variable                            Descripción
  ----------------------------------- -----------------------------------
  `Year`                              Año asociado al registro.

  `LocationAbbr`                      Abreviatura del estado, territorio
                                      o código nacional.

  `LocationDesc`                      Nombre del área geográfica.

  `GeographicLevel`                   Nivel geográfico: nacional, estatal
                                      o condado.

  `Data_Value`                        Tasa de mortalidad por ACV por cada
                                      100,000 habitantes.

  `Data_Value_Unit`                   Unidad de medida de la tasa.

  `Data_Value_Type`                   Tipo de tasa, incluyendo ajuste por
                                      edad y, en condados, suavizado
                                      espacial.

  `Stratification1`                   Sexo.

  `Stratification2`                   Grupo racial/étnico.

  `LocationID`                        Identificador del área geográfica.

  `Y_lat`                             Latitud de la ubicación.

  `X_lon`                             Longitud de la ubicación.

  `States`                            Código FIPS estatal.
  
  `Counties`                          Código FIPS de condado.
  
  -----------------------------------------------------------------------

## Objetivo del análisis

El notebook realiza un Análisis Exploratorio de Datos para
conocer la estructura y calidad del dataset y explorar patrones en la
mortalidad por ACV.

Entre los principales aspectos analizados se encuentran:

1.  Dimensiones y tipos de datos.
2.  Valores faltantes y registros duplicados.
3.  Cardinalidad de las variables categóricas.
4.  Valores fuera de rango y posibles valores extremos.
5.  Distribución de `Data_Value`.
6.  Distribución por nivel geográfico, sexo y raza/etnia.
7.  Comparación de tasas según sexo.
8.  Comparación de tasas según raza/etnia.
9.  Correlación entre variables numéricas relevantes.
10. Distribución geográfica de las tasas de mortalidad.

`Data_Value` puede utilizarse como variable objetivo para un problema
de regresión. También podría transformarse en una variable categórica
para plantear un problema de clasificación, por ejemplo, separando tasas
por encima y por debajo de una medida de referencia.

## Hallazgos exploratorios principales

El notebook identifica, entre otros, los siguientes patrones:

-   El dataset contiene 59,094 registros, superando ampliamente el
    requisito mínimo de registros y atributos para el análisis.
-   Existe una cantidad importante de valores sin tasa reportada. En
    particular, `Data_Value` presenta registros marcados como
    `Insufficient Data`.
-   La tasa de mortalidad presenta una distribución con sesgo
    positivo y valores extremos asociados principalmente con subgrupos
    pequeños a nivel de condado.
-   A nivel estatal, las diferencias entre las tasas medianas por sexo
    son relativamente pequeñas dentro de este dataset.
-   Se observa una diferencia considerable entre las tasas medianas de
    algunos grupos raciales/étnicos.
-   Las variables de latitud y longitud permiten visualizar un patrón
    geográfico de las tasas de mortalidad.
-   Los identificadores `States` y `Counties` aparecen originalmente
    como variables numéricas, aunque representan códigos y deben
    interpretarse como identificadores categóricos, no como cantidades
    continuas.
-   `LocationDesc` no debe utilizarse como identificador geográfico
    único, ya que nombres de condados pueden repetirse en diferentes
    estados. Para identificar una ubicación se recomienda utilizar
    `LocationID` o combinar `LocationAbbr` con `LocationDesc`.

## Calidad y limitaciones de los datos

El dataset es un conjunto de datos real de salud pública y presenta
características que deben considerarse antes de realizar análisis
estadísticos o modelos predictivos.

### Valores faltantes

Una parte importante de los registros no tiene una tasa disponible. La
ausencia está relacionada especialmente con áreas pequeñas y subgrupos
poblacionales con pocos eventos.

Por este motivo, no se recomienda eliminar o imputar automáticamente
todos los valores faltantes sin estudiar primero el motivo de su
ausencia.

### Valores extremos

El análisis identifica registros con tasas superiores a 300 muertes por
cada 100,000 habitantes. Estos registros corresponden a combinaciones de
condados pequeños y subgrupos raciales/étnicos con poblaciones
reducidas.

Estos valores no necesariamente representan errores de captura: pueden
reflejar la inestabilidad estadística producida por denominadores
poblacionales pequeños.

### Diseño de la tabla

El dataset tiene una estructura factorial que cruza áreas geográficas
con categorías de sexo y raza/etnia. Por ello, las frecuencias de estas
categorías reflejan en gran medida la forma en que fue construida la
tabla y no deben interpretarse directamente como la composición
demográfica de Estados Unidos.

## Requisitos de ejecución

Se necesita un entorno con:

-   Python 3.x
-   Jupyter Notebook** o JupyterLab
-   `numpy`
-   `pandas`
-   `matplotlib`
-   `seaborn`

### Instalación

Desde una terminal, dentro de la carpeta del proyecto:

``` bash
pip install numpy pandas matplotlib seaborn jupyter
```

Después:

``` bash
pip install numpy pandas matplotlib seaborn jupyter
```

## Instrucciones de uso

### 1. Descargar o clonar el repositorio

Clona el repositorio y entra a la carpeta:

``` bash
git clone https://github.com/Cachigod/icd2026/
cd /practices/P1
```

### 2. Verificar los archivos

La estructura esperada es:

``` text
.
├── Practica1_SSEG.ipynb
├── README.md
└── csv-1.csv
```

Es importante que `csv-1.csv` se encuentre en la misma carpeta que el
notebook

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
Practica1_SSEG.ipynb
```

### 5. Ejecutar el análisis

Ejecuta las celdas en orden, desde el inicio hasta el final del
notebook.

El notebook:

1.  Importa las bibliotecas necesarias.
2.  Configura las opciones de visualización.
3.  Carga `csv-1.csv`.
4.  Elimina la columna redundante `index`.
5.  Verifica los requisitos y la estructura del dataset.
6.  Analiza la calidad de los datos.
7.  Realiza análisis univariado y bivariado.
8.  Explora relaciones multivariadas.
9.  Genera visualizaciones.
10. Presenta conclusiones.

## Fuentes de los datos

**Fuente original:**

Centers for Disease Control and Prevention, National Vital Statistics
System. *Stroke Mortality Data Among US Adults (35+) by State/Territory
and County -- 2015-2017*.

https://data.cdc.gov/api/views/v246-z5tb

**Copia utilizada en el proyecto:**

Kaggle --- *US Stroke Mortality in Adults Over 35 (2015-2017)*,
publicada por `thedevastator`.

https://www.kaggle.com/datasets/thedevastator/us-stroke-mortality-in-adults-over-35-2015-2017
