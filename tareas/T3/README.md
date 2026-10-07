# Tarea 3: Notebook de regresión (Wine Quality)

Introducción a la Ciencia de Datos 2026 · Posgrado en Ciencias de la Computación, CICESE

En este notebook se usa la **regresión lineal** sobre el mismo conjunto de datos, la calidad de vinos tintos portugueses, para resolver dos problemas distintos. Primero como **regresión**, para predecir la calificación `quality`, y después como **clasificador**, para decidir si un vino es bueno (`quality` ≥ 7) o no. Cada parte se compara contra una línea base que no mira el vino. La pregunta de fondo es qué cambia cuando la variable objetivo deja de ser un número y pasa a ser una clase, y qué le falla a la recta en el segundo caso.

## Contenido del repositorio

| Archivo | Descripción |
|---|---|
| `SSEG_Tarea3.ipynb` | Notebook con todo el análisis |
| `winequality-red.csv` | Datos (1,599 registros de vino tinto, con encabezados) |
| `README.md` | Este archivo |

## Dataset

- **Nombre:** Wine Quality (vino tinto «vinho verde»), UCI Machine Learning Repository.
- **Fuente:** Cortez, Cerdeira, Almeida, Matos y Reis (2009). Donado a UCI el 6 de octubre de 2009. Se usa solo el archivo de vino tinto; los vinos blancos del mismo estudio están en otro archivo y no se usan.
- **Unidad de observación:** una muestra de vino tinto portugués de la región del Vinho Verde.
- **Variables de entrada (11):** pruebas fisicoquímicas de laboratorio: `fixed acidity`, `volatile acidity`, `citric acid`, `residual sugar`, `chlorides`, `free sulfur dioxide`, `total sulfur dioxide`, `density`, `pH`, `sulphates` y `alcohol`.
- **Variable objetivo:** `quality`, la mediana de al menos tres evaluaciones de catadores expertos, en una escala de 0 a 10. En este archivo toma valores de 3 a 8.
- **Faltantes:** ninguno.
- **Separador:** el archivo publicado por UCI separa las columnas con `;`. La celda de carga lo detecta: si al leerlo con `;` queda una sola columna, lo vuelve a leer con comas.
- **Licencia:** Creative Commons Attribution 4.0 International (CC BY 4.0).

## Estructura del notebook

1. Preparación del entorno
2. Los datos: carga, estructura y distribución de `quality`
3. Partición 80/20
4. Regresión lineal simple con `alcohol`: matriz de correlación de Pearson, recta de mínimos cuadrados calculada a mano y con scikit-learn, y gráfica
5. Regresión múltiple con las once variables
6. Las tres predicciones juntas: línea base, regresión simple y regresión múltiple
7. Clasificación: bueno = `quality` ≥ 7
8. Conclusión
9. Declaración de uso de IA generativa y referencias

## Metodología

- **Modelo:** `LinearRegression` de scikit-learn.
- **Partición:** 80 % entrenamiento (1,279 vinos) y 20 % prueba (320 vinos), con `random_state=42`. Es la misma partición para regresión y clasificación.
- **Líneas base:** en regresión, predecir siempre el promedio de calidad del entrenamiento; en clasificación, decir siempre «no bueno».
- **Métricas:** RMSE y R² para la regresión; porcentaje de aciertos para la clasificación.
- **Clasificación con una recta:** se ajusta la regresión a la columna binaria (1 = bueno, 0 = no bueno) y se clasifica como bueno si ŷ ≥ 0.5.

## Resultados principales

**Regresión** (conjunto de prueba):

| Modelo | RMSE | R² |
|---|:---:|:---:|
| Promedio (línea base) | 0.811 | −0.006 |
| Regresión simple (`alcohol`) | 0.707 | 0.236 |
| Regresión múltiple (11 variables) | 0.625 | 0.403 |

- `alcohol` es la variable más correlacionada con `quality` (r = 0.48). La recta con todos los vinos es ŷ = 1.87 + 0.361 · `alcohol`: cada punto porcentual más de alcohol se asocia con 0.36 puntos más de calidad en promedio.
- Las otras diez variables aportan información adicional: el R² pasa de 0.24 (solo alcohol) a 0.40 (las once).

**Clasificación** (conjunto de prueba, 320 vinos):

| Método | Aciertos |
|---|:---:|
| Siempre «no bueno» (línea base) | 85.31 % (273 de 320) |
| Recta con umbral ŷ ≥ 0.5 | 86.25 % (276 de 320) |

- La ventaja sobre la línea base es de tres vinos. La recta clasifica como buenos solo 7 vinos y detecta apenas 5 de los 47 buenos de prueba.
- 82 de las 320 predicciones (25.6 %) son negativas, y el máximo de ŷ es 0.54: la salida no se puede leer como probabilidad y casi nunca alcanza el umbral.
- Una recta sirve para estimar una magnitud, pero no para decidir una clase; un porcentaje de aciertos solo significa algo si se compara contra la línea base.

## Calidad y limitaciones de los datos

- **Filas duplicadas:** hay 240 filas duplicadas (15 % del archivo). No se eliminan, porque la tarea parte el archivo tal como viene. Como consecuencia, 64 de los 320 vinos de prueba tienen un gemelo exacto en entrenamiento, y las métricas de prueba son algo más favorables que las de un conjunto sin repetidos.
- **Desbalance:** el 82.5 % de los vinos (1,319 de 1,599) tiene calidad 5 o 6, y solo el 13.6 % (217) tiene calidad 7 o más. Esto limita lo que la regresión puede explicar y hace que la clase «bueno» sea minoritaria.
- **Corte de «bueno»:** `quality` ≥ 7 es el criterio de la tarea, no uno enológico.
- **Escalas distintas:** las columnas están en escalas muy diferentes, lo cual no afecta el ajuste de `LinearRegression`, pero sí haría falta tenerlo en cuenta para comparar coeficientes entre variables.

## Requisitos

- Python 3
- `numpy`, `pandas`, `matplotlib`, `scikit-learn`
- Jupyter Notebook o JupyterLab

```bash
pip install numpy pandas matplotlib scikit-learn jupyter
```

## Instrucciones de uso

### 1. Descargar o clonar el repositorio

Clona el repositorio y entra a la carpeta de la tarea:

```bash
git clone https://github.com/Cachigod/icd2026.git
cd icd2026/tareas/T3
```

### 2. Verificar los archivos

La estructura esperada es:

```text
.
├── README.md
├── SSEG_Tarea3.ipynb
└── winequality-red.csv
```

Es importante que `winequality-red.csv` se encuentre en la misma carpeta que el notebook.

### 3. Iniciar Jupyter

```bash
jupyter notebook
```

o:

```bash
jupyter lab
```

### 4. Abrir y ejecutar el notebook

Abre `SSEG_Tarea3.ipynb` y ejecuta las celdas en orden, desde el inicio hasta el final.

## Declaración de uso de IA generativa

El notebook utilizó Claude (Anthropic) como asistente para estructurar el notebook (Markdown), revisar la redacción de los textos y depurar el código.

## Referencias

- Cortez, P., Cerdeira, A., Almeida, F., Matos, T. y Reis, J. (2009). *Wine Quality* [Conjunto de datos]. UCI Machine Learning Repository. https://doi.org/10.24432/C56S3T. Licencia CC BY 4.0.
- Cortez, P., Cerdeira, A., Almeida, F., Matos, T. y Reis, J. (2009). Modeling wine preferences by data mining from physicochemical properties. *Decision Support Systems*, 47(4), 547-553.
- James, G., Witten, D., Hastie, T. y Tibshirani, R. (2021). *An Introduction to Statistical Learning*, 2a ed., capítulo 3. Springer. https://www.statlearning.com/
- scikit-learn: [LinearRegression](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LinearRegression.html) · [train_test_split](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.train_test_split.html) · [Métricas](https://scikit-learn.org/stable/modules/model_evaluation.html)

