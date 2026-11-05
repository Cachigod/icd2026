# Práctica 3: Clasificación de la situación académica de estudiantes

**Introducción a la Ciencia de Datos 2026**  
Posgrado en Ciencias de la Computación — CICESE

Análisis y comparación de cinco clasificadores para predecir si un estudiante terminará como **Dropout** (desertor), **Enrolled** (todavía inscrito) o **Graduate** (graduado), utilizando la información disponible al cierre del primer semestre.

## Contenido del proyecto

- `Practica3_SSEG.ipynb`: notebook principal con la preparación de los datos, análisis, búsqueda de hiperparámetros, comparación de modelos y generación de predicciones.
- `desercion-entrenamiento.csv`: conjunto de entrenamiento, con 3,539 estudiantes y la variable objetivo `Target`.
- `desercion-prueba.csv`: conjunto de prueba, con 885 estudiantes y sin la variable `Target`.
- `predicciones.csv`: archivo generado al ejecutar el notebook; contiene las predicciones para los 885 estudiantes de prueba.

Los dos archivos CSV deben estar en la misma carpeta que el notebook.

## Descripción del dataset

Los datos corresponden a estudiantes de licenciatura del Instituto Politécnico de Portalegre, Portugal, que ingresaron entre los ciclos 2008/09 y 2018/19. La fuente es el conjunto *Predict Students' Dropout and Academic Success*, publicado en UCI Machine Learning Repository.

### Unidad de observación y variable objetivo

Cada fila representa un estudiante. La variable `Target` describe su situación al final de la duración normal de la carrera:

- `Dropout`: abandonó sus estudios.
- `Enrolled`: continúa inscrito.
- `Graduate`: terminó sus estudios.

El entrenamiento contiene 3,539 estudiantes y la prueba 885. La distribución de clases en entrenamiento es:

| Clase | Estudiantes | Porcentaje |
|---|---:|---:|
| Dropout | 1,137 | 32.1 % |
| Enrolled | 635 | 17.9 % |
| Graduate | 1,767 | 49.9 % |

El conjunto de prueba no contiene `Target`; por ello, el notebook genera predicciones para esos registros y no calcula una métrica de desempeño real sobre la prueba.

### Variables utilizadas

Para predecir al cierre del primer semestre se utilizan 30 atributos:

- 9 nominales, como carrera, modalidad de admisión, nacionalidad y escolaridad/ocupación de los padres.
- 8 binarios, como género, beca, deudor y colegiatura al corriente.
- 13 numéricos, como edad de ingreso, calificación de admisión y unidades curriculares del primer semestre.

El conjunto de entrenamiento también contiene 6 atributos del segundo semestre. Estos se reservan para un análisis adicional que estima cuánto cambiaría el desempeño si se esperara hasta el cierre de ese semestre. No se utilizan para generar las predicciones finales porque no están disponibles en el archivo de prueba.

## Objetivo del análisis

Comparar cinco clasificadores vistos en clase mediante validación cruzada estratificada y búsqueda de hiperparámetros. La métrica principal es **F1 macro**, que promedia el F1 de las tres clases y evita que la clase mayoritaria domine la evaluación.

Los modelos comparados son:

1. Árbol de decisión.
2. Naive Bayes.
3. k vecinos más cercanos (kNN).
4. Máquinas de soporte vectorial (SVM).
5. Regresión logística.

Cada búsqueda evalúa 100 configuraciones. Los modelos se comparan con los mismos cinco pliegues estratificados y semilla fija. Además, las configuraciones ganadoras se verifican con 15 pliegues nuevos (cinco pliegues repetidos tres veces).

## Procesamiento y metodología

El notebook organiza el trabajo en las siguientes etapas:

1. **Preparación:** importa las bibliotecas, fija la semilla aleatoria y carga los dos CSV.
2. **Verificación:** comprueba dimensiones, columnas esperadas, identificadores, valores nulos y duplicados.
3. **Preprocesamiento:** aplica transformaciones dentro de los `Pipeline` para evitar que la información de validación se filtre al entrenamiento de cada pliegue.
4. **Búsqueda de hiperparámetros:** evalúa 100 configuraciones por clasificador, incluyendo alternativas de codificación de variables nominales, escalamiento, parámetros del modelo y pesos por clase cuando corresponde.
5. **Comparación:** calcula F1 macro, desviación entre pliegues, sensibilidad y especificidad por clase.
6. **Análisis de errores:** estudia la matriz de confusión del mejor modelo y las características de los desertores que se clasifican incorrectamente.
7. **Análisis del segundo semestre:** compara el modelo con 30 atributos del primer semestre contra versiones que incorporan los 6 atributos adicionales.
8. **Modelo final:** entrena el mejor clasificador con todo el conjunto de entrenamiento y genera `predicciones.csv`.

## Resultados principales

Los resultados guardados en el notebook muestran el siguiente desempeño:

| Clasificador | F1 macro (media) | Desv. entre pliegues |
|---|---:|---:|
| SVM | **0.689** | 0.009 |
| Regresión logística | 0.682 | 0.007 |
| Árbol de decisión | 0.645 | 0.010 |
| Naive Bayes | 0.625 | 0.007 |
| kNN | 0.582 | 0.023 |

El mejor modelo es un **SVM con kernel RBF**, con `C` ≈ 172.76, `gamma` ≈ 0.000778, peso de la clase `Enrolled` multiplicado por 2 y agrupación de categorías con menos de 10 estudiantes.

En la validación cruzada, el SVM identifica correctamente 744 de los 1,137 desertores (sensibilidad de 65.4 %). De los 877 estudiantes que predice como `Dropout`, 744 realmente pertenecen a esa clase (precisión de 84.8 %).

La regresión logística queda cerca del SVM (F1 macro de 0.682), con menor variabilidad entre pliegues y mayor facilidad de interpretación. Naive Bayes obtiene la mayor sensibilidad para `Dropout` (70.4 %), mientras que el árbol recupera mejor la clase `Enrolled` (63.3 %). kNN presenta la mayor dificultad con esa clase (sensibilidad de 16.4 %).

### ¿Conviene esperar al segundo semestre?

Al incorporar los seis atributos del segundo semestre, el F1 macro del SVM pasa de 0.689 a 0.717 con la misma configuración (+0.028), y a 0.713 al reoptimizar el modelo (+0.025). La mejora se concentra principalmente en la detección de desertores. El notebook concluye que no conviene posponer la intervención hasta el segundo semestre: es preferible usar el modelo del primer semestre y, si es posible, reevaluar posteriormente.

## Archivo de salida

Al ejecutar la última sección del notebook se genera `predicciones.csv`, con 885 filas y dos columnas:

| Columna | Descripción |
|---|---|
| `id` | Identificador del estudiante del conjunto de prueba. |
| `Target` | Clase predicha: `dropout`, `enrolled` o `graduate`. |

El notebook verifica que el archivo tenga las columnas requeridas, que conserve los identificadores del conjunto de prueba y que las predicciones pertenezcan a las tres etiquetas permitidas.

## Requisitos de ejecución

El notebook utiliza:

- Python 3.x
- Jupyter Notebook o JupyterLab
- NumPy
- pandas
- SciPy
- scikit-learn
- Matplotlib
- Seaborn
- cycler

El notebook incluye una celda para instalar `scikit-learn==1.7.0`. Para instalar las dependencias desde una terminal:

```bash
pip install numpy pandas scipy scikit-learn==1.7.0 matplotlib seaborn cycler jupyter
```

Después, inicia Jupyter:

```bash
jupyter notebook
```

o:

```bash
jupyter lab
```

Abre `Practica3_SSEG.ipynb` y ejecuta las celdas en orden, desde el inicio. Es importante que ambos archivos CSV estén en el directorio de trabajo del notebook.

## Limitaciones

- Los datos proceden de una sola institución y de estudiantes que ingresaron entre 2008/09 y 2018/19; los resultados no necesariamente se generalizan a otras instituciones o periodos.
- La clase `Enrolled` es la minoritaria y la más difícil de clasificar.
- El SVM omite aproximadamente un tercio de los desertores reales, por lo que sus predicciones no deben utilizarse como decisiones individuales definitivas.
- Los atributos del segundo semestre mejoran el desempeño, pero se conocen demasiado tarde para una intervención temprana.
- El F1 macro reportado corresponde a validación cruzada. Como el archivo de prueba no incluye las etiquetas reales, no se puede calcular aquí su desempeño real.

## Fuentes

- Realinho, V., Martins, M. V., Machado, J., & Baptista, L. (2021). *Predict Students' Dropout and Academic Success* [Conjunto de datos]. UCI Machine Learning Repository. https://doi.org/10.24432/C5MC89
- Realinho, V., Machado, J., Baptista, L., & Martins, M. V. (2022). Predicting student dropout and academic success. *Data, 7*(11), 146. https://doi.org/10.3390/data7110146
- scikit-learn: [Pipeline](https://scikit-learn.org/stable/modules/generated/sklearn.pipeline.Pipeline.html), [OneHotEncoder](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html), [ParameterSampler](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.ParameterSampler.html) y [métricas de evaluación](https://scikit-learn.org/stable/modules/model_evaluation.html).

**Licencia de los datos:** Creative Commons Attribution 4.0 International (CC BY 4.0), según la ficha del conjunto en UCI.
