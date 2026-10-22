# Tarea 4: Notebook de métricas de clasificación (Bank Marketing)

Introducción a la Ciencia de Datos 2026 · Posgrado en Ciencias de la Computación, CICESE

En este notebook se estudia un problema de clasificación binaria a partir de una campaña telefónica de un banco portugués que ofrecía depósitos a plazo. El objetivo es estimar qué clientes aceptarían la oferta y evaluar el modelo con métricas que tengan sentido para el banco. Como solo alrededor de 1 de cada 9 clientes aceptó, la exactitud por sí sola puede resultar engañosa. El análisis compara una línea base con una regresión logística, examina las curvas ROC y de precisión contra sensibilidad, y calcula cómo cambia la ganancia al modificar el umbral de decisión. También se muestra por qué incluir la duración de la llamada produce métricas artificialmente mejores: esa información no está disponible antes de llamar.

## Contenido del repositorio

| Archivo | Descripción |
|---|---|
| `SSEG_Tarea4.ipynb` | Notebook con el análisis exploratorio, el modelo y las evaluaciones |
| `bank-additional-full.csv` | Datos de la campaña completa (41,188 clientes, separados por `;`) |
| `README.md` | Este archivo |

## Dataset

- **Nombre:** Bank Marketing, de UCI Machine Learning Repository.
- **Fuente:** Moro, Rita y Cortez, a partir de campañas de telemarketing de un banco portugués realizadas entre mayo de 2008 y noviembre de 2010. El conjunto se describe en el artículo *A data-driven approach to predict the success of bank telemarketing* (2014).
- **Unidad de observación:** un cliente contactado durante una campaña, descrito mediante sus datos personales, información de contacto, campañas previas e indicadores económicos.
- **Archivo utilizado:** `bank-additional-full.csv`, con 41,188 registros y 21 columnas. El repositorio también contiene una muestra del 10 % (`bank-additional.csv`), que no se usa en este notebook.
- **Variables de entrada:** 20 variables: datos del cliente (`age`, `job`, `marital`, `education`, `default`, `housing`, `loan`), del contacto (`contact`, `month`, `day_of_week`, `duration`), de campañas (`campaign`, `pdays`, `previous`, `poutcome`) y cinco indicadores socioeconómicos (`emp.var.rate`, `cons.price.idx`, `cons.conf.idx`, `euribor3m`, `nr.employed`).
- **Variable objetivo:** `y`, que indica si el cliente contrató el depósito a plazo (`yes` o `no`).
- **Desbalance de clases:** 4,640 clientes aceptaron (11.27 %) y 36,548 no aceptaron (88.73 %).
- **Valores desconocidos:** algunas variables categóricas contienen la etiqueta `unknown`; en el notebook se conserva como una categoría más. No hay valores `NaN`.
- **Separador:** punto y coma (`;`).
- **Licencia:** Creative Commons Attribution 4.0 International (CC BY 4.0), según la ficha del conjunto de datos de UCI.

## Estructura del notebook

1. Preparación del entorno y configuración de gráficas.
2. Carga de datos y análisis exploratorio: estructura, duplicados, valores desconocidos y `pdays`.
3. Distribución de la variable objetivo y análisis de variables categóricas.
4. Exclusión de `duration` para evitar fuga de información.
5. Partición estratificada de entrenamiento y prueba, y clasificador de línea base.
6. Regresión logística con umbral de 0.5 y matriz de confusión.
7. Curvas ROC y de precisión contra sensibilidad, con comparación de umbrales.
8. Umbral de decisión según los costos e ingresos del banco.
9. Comparación con un modelo que incluye `duration`.
10. Conclusión, declaración de uso de IA generativa y referencias.

## Metodología

- **Objetivo:** predecir si un cliente aceptará la oferta (`y = yes`).
- **Variable excluida del modelo principal:** `duration`, porque la duración de una llamada solo se conoce después de realizarla. Incluirla al decidir a quién llamar produciría fuga de información.
- **Partición:** 80 % para entrenamiento y 20 % para prueba, con `stratify=y` y `random_state=42`, para conservar la proporción de clases y hacer reproducibles los resultados.
- **Línea base:** `DummyClassifier(strategy="most_frequent")`, que siempre predice la clase más frecuente.
- **Preprocesamiento:** `OneHotEncoder(handle_unknown="ignore")` para las variables categóricas y `StandardScaler` para las numéricas, dentro de un `ColumnTransformer` y un `Pipeline`.
- **Modelo:** `LogisticRegression(max_iter=5000)` de scikit-learn, con regularización L2 por omisión.
- **Métricas:** exactitud, precisión, sensibilidad, especificidad, F1, AUC ROC y precisión promedio (average precision).
- **Umbral de decisión:** se evalúan 0.5, 0.3 y 0.1. Con un costo de llamada de 5 y un ingreso de 50 por aceptación, el umbral de equilibrio es \(5/50 = 0.10\), suponiendo que las probabilidades estimadas estén razonablemente calibradas.
- **Ganancia:** se calcula como `50 × clientes que aceptan entre los llamados − 5 × número de llamadas`.

## Resultados principales

**Desbalance y línea base**

- Aceptaron 4,640 de 41,188 clientes (11.27 %); 36,548 no aceptaron.
- En el conjunto de prueba hay 8,238 clientes, de los cuales 928 aceptan.
- La línea base que siempre responde «no» obtiene una exactitud de 0.887, pero una sensibilidad de 0: no detecta a ningún cliente que aceptaría.

**Regresión logística sin `duration`, con umbral 0.5**

| Métrica | Resultado |
|---|---:|
| Exactitud | 0.901 |
| Precisión | 0.690 |
| Sensibilidad | 0.219 |
| Especificidad | 0.987 |
| F1 | 0.332 |
| AUC ROC | 0.801 |

La exactitud es apenas superior a la de la línea base, y el modelo detecta 203 de los 928 clientes que aceptan. La sensibilidad es importante porque los falsos negativos representan clientes que podrían generar ingresos y que no reciben una llamada.

**Efecto del umbral y ganancia**

- Con umbral 0.5, el modelo llama a 294 clientes; 203 aceptan y la ganancia total en prueba es 8,680.
- Con umbral 0.10, llama a 1,900 clientes; 632 aceptan y la ganancia total es 22,100.
- Llamar a todos produce una ganancia de 5,210; no llamar a nadie produce 0.
- El umbral 0.10 aumenta la ganancia frente al umbral 0.5, bajo los costos e ingresos definidos en el notebook.

**Fuga de información al incluir `duration`**

Al incluir la duración de la llamada, las métricas mejoran: la sensibilidad pasa de 0.219 a 0.436, el F1 de 0.332 a 0.541, la exactitud de 0.901 a 0.917 y el AUC de 0.801 a 0.942. Sin embargo, `duration` no está disponible antes de llamar, por lo que estas métricas no representan el desempeño que se podría conseguir en una campaña real.

## Calidad y limitaciones de los datos

- **Desbalance:** la clase positiva (`yes`) representa solo el 11.27 % de los registros. Por eso, la exactitud debe interpretarse junto con la sensibilidad, la precisión, F1 y las curvas de evaluación.
- **Valores `unknown`:** se conservan como una categoría para las variables categóricas, de acuerdo con el enfoque descrito en el notebook.
- **`pdays = 999`:** representa a clientes para los que no se registró un contacto previo en la campaña anterior; debe interpretarse con cuidado y no como un número ordinario de días.
- **Fuga de información:** `duration` está relacionada con el resultado, pero se conoce solo después de la llamada. Su inclusión sirve como comparación ilustrativa, no como modelo válido para decidir a quién contactar.
- **Umbral económico:** el umbral de 0.10 depende de los costos e ingresos estipulados y supone probabilidades razonablemente calibradas. En una aplicación real, estos valores y la calibración deben validarse.
- **Partición de prueba:** los resultados corresponden a una única partición aleatoria estratificada con `random_state=42`; no garantizan el mismo desempeño en otras campañas.

## Requisitos

- Python 3
- `numpy`, `pandas`, `matplotlib`, `scikit-learn`
- Jupyter Notebook o JupyterLab

Instala las dependencias con:

```bash
pip install numpy pandas matplotlib scikit-learn jupyter
```

## Instrucciones de uso

### 1. Descargar o clonar el repositorio

Si el repositorio conserva la estructura de las tareas anteriores, entra a la carpeta de la Tarea 4:

```bash
git clone https://github.com/Cachigod/icd2026.git
cd icd2026/tareas/T4
```

### 2. Verificar los archivos

La estructura esperada es:

```text
.
├── README.md
├── SSEG_Tarea4.ipynb
├── bank-additional-full.csv
└── bank-additional-names.txt
```

Es importante que `bank-additional-full.csv` se encuentre en la misma carpeta que el notebook, porque se carga con una ruta relativa.

### 3. Iniciar Jupyter

```bash
jupyter notebook
```

También puedes usar:

```bash
jupyter lab
```

### 4. Abrir y ejecutar el notebook

Abre `SSEG_Tarea4.ipynb` y ejecuta las celdas en orden, desde el inicio hasta el final.

## Declaración de uso de IA generativa

Este notebook utilizó Claude (Anthropic) como asistente para estructurar el notebook (Markdown), redactar y revisar los textos, así como depurar el código.

## Referencias

- Moro, S., Cortez, P., & Rita, P. (2014). A data-driven approach to predict the success of bank telemarketing. *Decision Support Systems*, 62, 22–31. https://doi.org/10.1016/j.dss.2014.03.001
- Moro, S., Rita, P., & Cortez, P. *Bank Marketing* [Conjunto de datos]. UCI Machine Learning Repository. https://doi.org/10.24432/C5K306. Licencia CC BY 4.0.
- Kaufman, S., Rosset, S., Perlich, C., & Stitelman, O. (2012). Leakage in data mining: Formulation, detection, and avoidance. *ACM Transactions on Knowledge Discovery from Data*, 6(4), 1–21.
- Fawcett, T. (2006). An introduction to ROC analysis. *Pattern Recognition Letters*, 27(8), 861–874.
- Saito, T., & Rehmsmeier, M. (2015). The precision-recall plot is more informative than the ROC plot when evaluating binary classifiers on imbalanced datasets. *PLOS ONE*, 10(3), e0118432.
- Elkan, C. (2001). The foundations of cost-sensitive learning. *Proceedings of the 17th International Joint Conference on Artificial Intelligence (IJCAI)*, 973–978.
- scikit-learn: [LogisticRegression](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html) · [train_test_split](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.train_test_split.html) · [Métricas de clasificación](https://scikit-learn.org/stable/modules/model_evaluation.html#classification-metrics) · [RocCurveDisplay](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.RocCurveDisplay.html) · [PrecisionRecallDisplay](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.PrecisionRecallDisplay.html)

