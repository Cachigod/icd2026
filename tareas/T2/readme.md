
# Tarea 2: Preprocesamiento de datos (Breast Cancer Wisconsin)

En este Notebook se explora el dataset **Breast Cancer Wisconsin (Original)**, se aplican dos técnicas de preprocesamiento (**limpieza de datos** y **reducción de dimensionalidad con PCA**) y compara gráficamente los datos antes y después de cada una.

## Contenido del repositorio

| Archivo | Descripción |
|---|---|
| `SSEG_Tarea2-2.ipynb` | Notebook con todo el análisis |
| `breast-cancer-wisconsin.data` | Datos (699 registros, sin encabezados) |
| `breast-cancer-wisconsin.names` | Descripción del dataset |

## Dataset

- **Fuente:** Dr. William H. Wolberg, University of Wisconsin Hospitals, Madison (donado por Olvi Mangasarian, 1992).
- **Unidad de observación:** una muestra de tejido mamario obtenida por biopsia con aguja fina.
- **Variables:** 9 características numéricas con puntajes de 1 a 10, un identificador (`id`) y la clase (`2` = benigno, `4` = maligno).
- **Faltantes:** 16 valores en `bare_nuclei`, marcados con `?`.

## Estructura del notebook

1. Preparación del entorno
2. Carga del dataset
3. Análisis exploratorio (EDA)
4. Limpieza de datos, con gráficas antes/después
5. Reducción de dimensionalidad con PCA, con gráficas antes/después
6. Conclusiones
7. Referencias

## Resultados principales

- La limpieza pasa de 699 × 11 a 691 × 10 (registros × columnas), sin valores faltantes.
- Con 2 componentes principales se conserva el 74.2 % de la varianza (PC1: 65.6 %, PC2: 8.6 %).
- En el plano PC1-PC2 los dos diagnósticos quedan bastante separados, aunque el PCA no usó la etiqueta.

## Requisitos

- Python 3
- `numpy`, `pandas`, `matplotlib`, `seaborn`, `scikit-learn`

```bash
pip install numpy pandas matplotlib seaborn scikit-learn
```

## Referencias

- Wolberg, W. H. y Mangasarian, O. L. (1990). *Multisurface method of pattern separation for medical diagnosis applied to breast cytology.* Proceedings of the National Academy of Sciences, U.S.A., 87, 9193-9196.
- Mangasarian, O. L. y Wolberg, W. H. (1990). *Cancer diagnosis via linear programming.* SIAM News, 23(5), 1 y 18.
