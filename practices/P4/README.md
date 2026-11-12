# Práctica 4: Ensambles de clasificadores en deserción estudiantil y Bank Marketing

**Introducción a la Ciencia de Datos 2026**  
Posgrado en Ciencias de la Computación — CICESE

Estudio de cuánto mejoran los **ensambles** (bagging, boosting, votación y *stacking*) a cinco clasificadores base que ya están bien ajustados. La pregunta se responde con dos conjuntos de datos reales: la **deserción estudiantil** de la Práctica 3 y la campaña telefónica de **Bank Marketing** de la Tarea 4. Al final se guarda el mejor modelo de Bank Marketing en un archivo `.joblib`.

## Contenido del proyecto

**Necesarios para ejecutar el notebook**

- `Practica4_SSEG.ipynb`: notebook principal. Recupera los cinco clasificadores de la Práctica 3, estudia las tres familias de ensambles, los aplica a Bank Marketing y guarda el mejor modelo.
- `desercion-entrenamiento.csv`: 3,539 estudiantes con la variable objetivo `Target` (el mismo archivo de la Práctica 3).
- `bank-additional-full.csv`: 41,188 clientes de Bank Marketing, con las columnas separadas por punto y coma (`;`).
- `requirements.txt`: versiones de las bibliotecas con las que se obtuvieron los resultados.

**Documentación**

- `bank-additional-names.txt`: descripción oficial de los atributos de Bank Marketing, con el significado de cada columna y de sus valores. El notebook no lo lee; es solo de consulta.

**Archivos de la Práctica 3 que están en la carpeta, pero que este notebook no usa**

- `desercion-prueba.csv`: 885 estudiantes sin `Target`. Este notebook evalúa solo con validación cruzada sobre el entrenamiento.
- `predicciones.csv`: predicciones de la Práctica 3 para esos 885 estudiantes. Este notebook no lo genera ni lo modifica.

**Se genera al ejecutar el notebook**

- `erick-santiago-modelo.joblib`: se crea en la sección 2.7, con el mejor modelo de Bank Marketing.

Los dos archivos CSV que usa el notebook deben estar en la misma carpeta que él.

## Descripción de los conjuntos de datos

| | Deserción estudiantil (Práctica 3) | Bank Marketing (Tarea 4) |
|---|---|---|
| **Fuente** | UCI 697, *Predict Students' Dropout and Academic Success* | UCI 222, *Bank Marketing* (versión con contexto económico) |
| **Origen** | Estudiantes de licenciatura del Instituto Politécnico de Portalegre, Portugal, que ingresaron entre 2008/09 y 2018/19 | Campañas de telemarketing de un banco portugués para ofrecer un depósito a plazo, de mayo de 2008 a noviembre de 2010 |
| **Unidad de observación** | Un estudiante | Un cliente contactado por teléfono |
| **Instancias** | 3,539 | 41,188 |
| **Atributos usados** | 30 (9 nominales, 8 binarios y 13 numéricos) | 19 (10 categóricos y 9 numéricos) |
| **Variable objetivo** | `Target`: `Dropout`, `Enrolled` o `Graduate` | `y`: 1 = `yes` (aceptó el depósito), 0 = `no` |
| **Validación** | `StratifiedKFold(5, shuffle=True, random_state=42)`, los mismos pliegues de la Práctica 3 | `StratifiedKFold(n_splits=5, shuffle=True, random_state=2026)` |
| **Métrica** | F1 macro y sensibilidad de `Enrolled` | F1 macro y sensibilidad de `yes` |
| **Licencia** | CC BY 4.0 | CC BY 4.0 |

### 1. Deserción estudiantil

La variable `Target` describe la situación del estudiante al final de la duración normal de la carrera. La distribución en el entrenamiento es:

| Clase | Estudiantes | Porcentaje |
|---|---:|---:|
| Dropout | 1,137 | 32.1 % |
| Enrolled | 635 | 17.9 % |
| Graduate | 1,767 | 49.9 % |

Se usan los 30 atributos que se conocen al cierre del primer semestre:

- **9 nominales** codificados como enteros: estado civil, modalidad de admisión, carrera, estudios previos, nacionalidad, y estudios y ocupación de la madre y del padre.
- **8 binarios**: turno, desplazado, necesidades especiales, deudor, colegiatura al corriente, género, becario e internacional.
- **13 numéricos**: orden de preferencia, calificaciones previa y de admisión, edad al ingresar, seis variables de unidades curriculares del primer semestre y tres indicadores macroeconómicos.

El archivo de entrenamiento trae además seis atributos del segundo semestre; este notebook no los utiliza. No hay valores nulos ni filas duplicadas. La clase `Enrolled` es la minoritaria y la más difícil de clasificar.

### 2. Bank Marketing

El archivo original trae 20 atributos de entrada, entre ellos cinco indicadores socioeconómicos del Banco de Portugal. El notebook **elimina `duration`** (duración de la última llamada), porque solo se conoce cuando la llamada ya terminó y contiene el resultado. Quedan 19 atributos:

- **10 categóricos**: `job`, `marital`, `education`, `default`, `housing`, `loan`, `contact`, `month`, `day_of_week` y `poutcome`.
- **9 numéricos**: `age`, `campaign`, `pdays`, `previous` y los indicadores `emp.var.rate`, `cons.price.idx`, `cons.conf.idx`, `euribor3m` y `nr.employed`.

La variable objetivo está muy desbalanceada: 4,640 clientes aceptaron (11.3 %) y 36,548 no (88.7 %), es decir, uno de cada nueve. Otros problemas de calidad que se tienen en cuenta:

- Los valores faltantes vienen codificados como la categoría `unknown` en seis columnas.
- En `pdays`, el valor 999 significa «nunca contactado» (96.3 % de los clientes) y no es una cantidad.
- `campaign` y `previous` están muy sesgadas.

## Objetivo del análisis

Responder, con el F1 macro como métrica principal, cuánto ganan los ensambles sobre los cinco clasificadores base de la Práctica 3 (árbol de decisión, Naive Bayes, kNN, SVM y regresión logística) cuando estos ya están bien ajustados, y qué cambia entre los dos problemas. No se usan redes neuronales.

## Procesamiento y metodología

El notebook tiene dos partes.

**Parte 1: deserción estudiantil**

1. **Datos y base:** se cargan los datos igual que en la Práctica 3, se construyen los mismos cinco pliegues y se recuperan los cinco clasificadores base con su preprocesamiento y sus hiperparámetros ganadores, sin repetir su optimización. Se comprueba que reproducen los resultados de la Práctica 3.
2. **Varianza:** un árbol sin límite de profundidad contra bagging de esos árboles, con curva de hasta 400 árboles (`warm_start=True`) y diagnóstico de la varianza.
3. **Sesgo:** un tocón (árbol de profundidad 1) contra AdaBoost de tocones, con curva de hasta 400 rondas (`staged_predict()`).
4. **Combinar los cinco base:** votación dura, votación suave (el SVM se calibra con `CalibratedClassifierCV` dentro de cada partición) y *stacking* con meta-modelo logístico, además de la correlación entre los errores de los cinco.
5. **Bagging optimizado:** Random Forest y Extra-Trees con `RandomizedSearchCV` (50 configuraciones, gráfica de meseta).
6. **Boosting optimizado:** XGBoost y LightGBM con `RandomizedSearchCV` (50 configuraciones, gráfica de meseta).
7. **Comparación:** tabla ordenada por F1 macro con la sensibilidad de `Enrolled`, el tiempo de entrenamiento y diferencias pareadas por pliegue contra el mejor base; matriz de confusión solo del mejor modelo.
8. **Transferencia:** el mejor modelo de la deserción, con su receta, se aplica a Bank Marketing sin reoptimizarlo.

**Parte 2: Bank Marketing**

1. **Protocolo:** archivo sin `duration`, `y` en 1 (`yes`) y 0 (`no`), y el F1 macro de cada modelo sale de `cross_validate` con `StratifiedKFold(n_splits=5, shuffle=True, random_state=2026)`.
2. **Todo dentro del modelo:** preprocesamiento, balanceo y **umbral de decisión** van dentro de un `Pipeline` o de un meta-estimador (`TunedThresholdClassifierCV`), de modo que se ajustan solo con los pliegues de entrenamiento.
3. **Modelos lineales:** regresión logística con el preprocesamiento de la Tarea 4 y con uno nuevo (`pdays` como categórica, transformación logarítmica de `campaign` y `previous`), con pesos por clase y con umbral.
4. **Ensambles de árboles optimizados:** Random Forest, XGBoost y LightGBM, más AdaBoost de tocones, todos con umbral.
5. **Combinaciones:** votación suave y *stacking* de cuatro modelos distintos.
6. **Comparación y modelo final:** tabla ordenada por F1 macro con la sensibilidad de `yes` y el tiempo de entrenamiento, matriz de confusión del mejor modelo, comparación con la deserción y guardado del modelo en `erick-santiago-modelo.joblib`.

## Resultados principales

Los resultados guardados en el notebook muestran lo siguiente.

### Deserción estudiantil (F1 macro, media ± desviación entre 5 pliegues)

| Modelo | F1 macro | Sensibilidad de `Enrolled` |
|---|---:|---:|
| XGBoost optimizado | **0.6949 ± 0.0126** | 0.553 |
| Stacking de los 5 base con pesos `balanced` | 0.6919 ± 0.0067 | 0.603 |
| Votación dura | 0.6909 ± 0.0070 | 0.512 |
| LightGBM optimizado | 0.6888 ± 0.0176 | 0.532 |
| SVM (mejor clasificador base) | 0.6886 ± 0.0092 | 0.559 |
| Random Forest optimizado | 0.6825 ± 0.0136 | 0.518 |
| Regresión logística | 0.6815 ± 0.0066 | 0.531 |
| Extra-Trees optimizado | 0.6784 ± 0.0061 | 0.557 |
| Bagging de árboles sin límite | 0.6611 ± 0.0258 | 0.326 |
| AdaBoost de tocones | 0.6416 ± 0.0122 | 0.266 |

- **Contra bases bien ajustados, casi nada:** el mejor ensamble supera al SVM por solo +0.006 de F1 macro, con una desviación de la diferencia de 0.014 y ganando en 3 de 5 pliegues, lo que no se distingue del ruido.
- **Contra modelos débiles, mucho:** bagging sube el árbol sin límite de 0.596 a 0.661 (+0.065) y AdaBoost sube el tocón de 0.492 a 0.642 (+0.150).
- **Los errores están correlacionados:** la correlación media entre los cinco clasificadores es 0.54 y los cinco fallan a la vez en 11.8 % de los estudiantes. El SVM y la regresión logística tienen errores casi iguales (0.84).

### Bank Marketing (F1 macro, media ± desviación entre 5 pliegues)

| Modelo | F1 macro | Sensibilidad de `yes` |
|---|---:|---:|
| **LightGBM optimizado + umbral** | **0.7187 ± 0.0060** | 0.543 |
| Random Forest optimizado + umbral | 0.7181 ± 0.0057 | 0.554 |
| *Stacking* de cuatro modelos + umbral | 0.7178 ± 0.0056 | 0.550 |
| Votación suave de cuatro modelos + umbral | 0.7178 ± 0.0051 | 0.547 |
| XGBoost optimizado + umbral | 0.7161 ± 0.0063 | 0.544 |
| Regresión logística + umbral | 0.7070 ± 0.0062 | 0.511 |
| XGBoost con la receta de la deserción | 0.6974 ± 0.0047 | 0.604 |

- **Ganancia pequeña pero consistente:** los cinco mejores ensambles superan a la regresión logística con umbral por +0.009 a +0.012, y ganan en los 5 pliegues.
- **El umbral importa más que el ensamble:** con la misma configuración, ajustar el umbral sube el F1 macro entre 0.06 y 0.07, unas seis veces más que cambiar la regresión logística por el mejor ensamble.
- **Mejor modelo:** LightGBM con 491 árboles poco profundos, sin pesos por clase y con el umbral ajustado dentro del modelo. En la matriz de confusión llamaría al 13.0 % de los clientes (5,335), de los cuales acepta el 47.3 %, y encontraría a 2,521 de los 4,640 que aceptan.
- **La receta no se traslada:** la configuración de XGBoost de la deserción queda por debajo incluso de la regresión logística con umbral, porque su frontera se eligió para otro problema.

### ¿Funcionó lo mismo en los dos problemas?

En parte. Los ensambles ganan poco en ambos y los modelos se equivocan en los mismos casos. Pero en la deserción los pesos por clase eran la herramienta clave, mientras que en Bank Marketing lo fue el umbral. Además, con 12 veces más datos y una señal formada por cortes en atributos con pocos valores, los ensambles de árboles ganan a la regresión logística de forma consistente.

## Archivo de salida

Al ejecutar la sección 2.7 se genera `erick-santiago-modelo.joblib` (alrededor de 1.8 MB), con el mejor modelo de Bank Marketing entrenado con los 41,188 clientes. Es un `TunedThresholdClassifierCV` que contiene el `Pipeline` completo (preprocesamiento y LightGBM) y el umbral elegido con todo el conjunto (0.271). Recibe un `DataFrame` con las 19 columnas del archivo, sin `duration` ni `y`, y devuelve 1 (`yes`) o 0 (`no`). El notebook comprueba que, al volver a cargarlo, da las mismas predicciones, y que el F1 macro con la llamada exacta de `cross_validate` coincide con el de la tabla.

Para usarlo:

```python
import joblib
import pandas as pd

modelo = joblib.load("erick-santiago-modelo.joblib")
clientes = pd.read_csv("bank-additional-full.csv", sep=";").drop(columns=["y", "duration"])

prediccion = modelo.predict(clientes.head(5))   # 1 = aceptaría, 0 = no aceptaría
```

Para cargarlo se necesitan las mismas bibliotecas con que se generó (ver los requisitos).

## Requisitos de ejecución

El notebook utiliza las siguientes versiones, con las que se obtuvieron los resultados:

| Biblioteca | Versión |
|---|---|
| Python | 3.12 |
| NumPy | 2.5.4 |
| pandas | 2.2.3 |
| SciPy | 1.16.3 |
| scikit-learn | 1.7.0 |
| XGBoost | 3.0.2 |
| LightGBM | 4.6.0 |
| Matplotlib | 3.11.2 |
| Seaborn | 0.13.2 |
| joblib | 1.6.0 |
| cycler | cualquier versión reciente |

Todas están listadas en `requirements.txt`. Para instalarlas desde una terminal, en la carpeta del proyecto:

```bash
pip install -r requirements.txt
```

Además, una de las primeras celdas del notebook instala por sí misma las versiones que más cambian los resultados (`scikit-learn`, `xgboost`, `lightgbm`, `scipy` y `pandas`), así que también se puede ejecutar sin este paso.

## Instrucciones de uso

1. Coloca en una misma carpeta `Practica4_SSEG.ipynb`, `desercion-entrenamiento.csv`, `bank-additional-full.csv` y `requirements.txt`, e instala las dependencias con `pip install -r requirements.txt`.
2. Inicia Jupyter desde esa carpeta:

   ```bash
   jupyter notebook
   ```

   o:

   ```bash
   jupyter lab
   ```

3. Abre `Practica4_SSEG.ipynb` y ejecuta todas las celdas en orden, desde el inicio (*Restart & Run All*). Si es la primera vez que se instalan las versiones fijas, puede ser necesario reiniciar el kernel y volver a ejecutar desde el principio.
4. Revisa las tablas y las gráficas de cada sección. La salida final es el archivo `erick-santiago-modelo.joblib`, en la misma carpeta.

El notebook completo tarda unos 9 minutos en una computadora con varios núcleos; en Google Colab (2 núcleos) puede tardar bastante más. Lo más costoso son las búsquedas aleatorias y los ensambles sobre los 41,188 clientes. Todas las rutas son relativas y las semillas están fijas, por lo que los resultados se repiten.

## Limitaciones

- Los datos de la deserción provienen de una sola institución y de ciclos de 2008/09 a 2018/19; los de Bank Marketing, de una sola campaña de un banco portugués entre 2008 y 2010. Los resultados no necesariamente se generalizan a otras instituciones o periodos.
- La búsqueda de hiperparámetros y la comparación usan los mismos pliegues, así que la mejor configuración de cada modelo es algo optimista. Con 41 mil clientes ese sesgo es pequeño; con 3,539 estudiantes es mayor.
- En la deserción, las diferencias entre los mejores modelos son menores que la desviación entre pliegues, por lo que no permiten declarar un ganador.
- Los dos problemas tienen un techo de información: los modelos fuertes fallan en los mismos casos (estudiantes con desempeño intermedio, clientes cuya decisión depende de cosas que no están en los datos).
- Los resultados corresponden a validación cruzada; no hay un conjunto de prueba con etiquetas.
- El archivo `.joblib` solo se puede cargar de forma confiable con las versiones de las bibliotecas indicadas.

## Fuentes

- Realinho, V., Martins, M. V., Machado, J., & Baptista, L. (2021). *Predict Students' Dropout and Academic Success* [Conjunto de datos]. UCI Machine Learning Repository. https://doi.org/10.24432/C5MC89
- Realinho, V., Machado, J., Baptista, L., & Martins, M. V. (2022). Predicting student dropout and academic success. *Data, 7*(11), 146. https://doi.org/10.3390/data7110146
- Moro, S., Rita, P., & Cortez, P. (2012). *Bank Marketing* [Conjunto de datos]. UCI Machine Learning Repository. https://doi.org/10.24432/C5K306
- Moro, S., Cortez, P., & Rita, P. (2014). A data-driven approach to predict the success of bank telemarketing. *Decision Support Systems, 62*, 22-31. https://doi.org/10.1016/j.dss.2014.03.001
- scikit-learn: [Pipeline](https://scikit-learn.org/stable/modules/generated/sklearn.pipeline.Pipeline.html), [StackingClassifier](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.StackingClassifier.html), [TunedThresholdClassifierCV](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.TunedThresholdClassifierCV.html) y [métricas de evaluación](https://scikit-learn.org/stable/modules/model_evaluation.html).
- [XGBoost](https://xgboost.readthedocs.io/) y [LightGBM](https://lightgbm.readthedocs.io/).

**Licencia de los datos:** Creative Commons Attribution 4.0 International (CC BY 4.0), según las fichas de los dos conjuntos en UCI.
