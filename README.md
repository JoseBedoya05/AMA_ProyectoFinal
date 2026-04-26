# Trabajo Final - Aprendizaje de Máquina Aplicado  
## Clasificación del desempeño académico en Pruebas Saber 11

Este repositorio contiene los notebooks desarrollados para construir un flujo reproducible de análisis, preprocesamiento y modelado baseline sobre el dataset **Resultados Únicos Saber 11**. El objetivo del trabajo es formular un problema de clasificación multiclase que permita estimar el nivel de desempeño académico de un estudiante a partir de variables institucionales, geográficas, familiares y socioeconómicas.

El flujo está organizado en tres notebooks principales:

1. `00_ObtenerDataset.ipynb`: carga, filtrado inicial, construcción del target y generación del dataset base.
2. `01_EDA.ipynb`: análisis exploratorio, diagnóstico de calidad, cardinalidad, valores nulos y asociación entre variables categóricas.
3. `02_Preprocesamiento.ipynb`: eliminación de columnas, partición train/test, pipeline de preprocesamiento y entrenamiento del modelo baseline.

---

## 1. Estructura recomendada del repositorio

```text
TrabajoFinal-Saber11/
│
├── notebooks/
│   ├── 00_ObtenerDataset.ipynb
│   ├── 01_EDA.ipynb
│   └── 02_Preprocesamiento.ipynb
│
├── requirements.txt
├── README.md
└── .gitignore
```

La carpeta de datos no se recomienda subir a GitHub porque el dataset original es de gran tamaño. Para ejecutar los notebooks en Google Colab, los archivos deben estar alojados en Google Drive con la estructura descrita en la siguiente sección.

---

## 2. Estructura esperada en Google Drive

Los notebooks fueron preparados para ejecutarse en Google Colab usando la siguiente ruta base:

```text
/content/drive/MyDrive/Aprendizaje de Maquina Aplicado/TrabajoFinal/
```

Dentro de esa ruta se espera la siguiente estructura:

```text
TrabajoFinal/
│
└── Data/
    │
    ├── Input/
    │   ├── Resultados_únicos_Saber_11_20260419.csv
    │   └── DiccionarioDatos.xlsx
    │
    └── Output/
        ├── Resultado_filtrados_periodo_2021_2022.csv
        ├── DataSetInicial.csv
        ├── DataSetInicial_muestra_7000.csv
        ├── Resumen_EDA_Saber11.csv
        ├── Columas a Eliminar.csv
        └── modelo_logistica_ovo_pipeline.joblib
```

### Archivos de entrada requeridos

| Archivo | Ubicación esperada | Descripción |
|---|---|---|
| `Resultados_únicos_Saber_11_20260419.csv` | `Data/Input/` | Dataset original descargado desde Datos Abiertos Colombia. |
| `DiccionarioDatos.xlsx` | `Data/Input/` | Diccionario de datos con metadatos y reglas iniciales de eliminación de columnas. |

### Archivos generados por el flujo

| Archivo | Notebook que lo genera | Descripción |
|---|---|---|
| `Resultado_filtrados_periodo_2021_2022.csv` | `00_ObtenerDataset.ipynb` | Dataset filtrado para los periodos 2021 y 2022. |
| `DataSetInicial.csv` | `00_ObtenerDataset.ipynb` | Dataset base para EDA y modelado, con target construido. |
| `DataSetInicial_muestra_7000.csv` | `00_ObtenerDataset.ipynb` | Muestra liviana para pruebas rápidas. |
| `Resumen_EDA_Saber11.csv` | `01_EDA.ipynb` | Resumen de calidad, cardinalidad y diagnóstico exploratorio. |
| `Columas a Eliminar.csv` | `01_EDA.ipynb` | Listado final de columnas recomendadas para eliminación. |
| `modelo_logistica_ovo_pipeline.joblib` | `02_Preprocesamiento.ipynb` | Artefacto final del pipeline entrenado. |

---

## 3. Instalación del ambiente

### Opción A: ejecución en Google Colab

Google Colab ya incluye muchas de las librerías necesarias. Aun así, para asegurar consistencia, se recomienda ejecutar la siguiente celda al inicio de cada notebook:

```python
!pip install -r requirements.txt
```

Si el archivo `requirements.txt` está en el repositorio clonado dentro de Colab:

```python
!git clone <URL_DEL_REPOSITORIO>
%cd TrabajoFinal-Saber11
!pip install -r requirements.txt
```

Luego se debe montar Google Drive:

```python
from google.colab import drive
drive.mount("/content/drive")
```

### Opción B: ejecución local

Crear un ambiente virtual:

```bash
python -m venv .venv
```

Activar el ambiente:

```bash
# Windows
.venv\Scripts\activate

# Linux / macOS
source .venv/bin/activate
```

Instalar dependencias:

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

Registrar el kernel para Jupyter:

```bash
python -m ipykernel install --user --name saber11-ml --display-name "Python - Saber11 ML"
```

Ejecutar Jupyter:

```bash
jupyter notebook
```

---

## 4. Orden de ejecución de los notebooks

Los notebooks deben ejecutarse en este orden:

```text
1. 00_ObtenerDataset.ipynb
2. 01_EDA.ipynb
3. 02_Preprocesamiento.ipynb
```

Este orden es importante porque cada notebook genera insumos que son usados por el siguiente.

---

## 5. Notebook 00 - Obtener dataset

### Objetivo

Preparar el dataset inicial para el análisis exploratorio y el modelado.

### Principales tareas realizadas

- Montaje de Google Drive.
- Lectura del dataset original desde `Data/Input/`.
- Filtrado de registros por periodo, conservando los años 2021 y 2022.
- Filtrado de la población de interés, manteniendo estudiantes activos.
- Conversión de variables de puntaje a formato numérico.
- Construcción de la variable objetivo `target`.
- Eliminación inicial de columnas con riesgo de leakage o baja utilidad.
- Eliminación de registros duplicados usando `estu_consecutivo`.
- Exportación de `DataSetInicial.csv` y de una muestra de 7.000 registros.

### Salidas esperadas

Al finalizar este notebook deben existir los siguientes archivos:

```text
Data/Output/DataSetInicial.csv
Data/Output/DataSetInicial_muestra_7000.csv
```

---

## 6. Notebook 01 - EDA

### Objetivo

Realizar el análisis exploratorio del dataset depurado para definir decisiones de preprocesamiento, imputación y eliminación de variables.

### Principales tareas realizadas

- Carga de `DataSetInicial.csv`.
- Revisión de tamaño, estructura y tipos de datos.
- Diagnóstico de valores nulos.
- Análisis de cardinalidad absoluta y relativa.
- Identificación de columnas constantes, casi constantes o de alta cardinalidad.
- Análisis de distribución del target.
- Evaluación de asociación entre variables categóricas mediante **V de Cramér**.
- Identificación de pares de variables altamente asociadas.
- Eliminación iterativa de variables redundantes, conservando aquellas con mayor asociación con el target.
- Generación de un archivo con las columnas recomendadas para eliminar.

### Salidas esperadas

Al finalizar este notebook debe existir:

```text
Data/Output/Columas a Eliminar.csv
```

El nombre del archivo conserva la escritura usada en los notebooks: `Columas a Eliminar.csv`.

---

## 7. Notebook 02 - Preprocesamiento y baseline

### Objetivo

Construir el pipeline final de preprocesamiento y entrenar un modelo baseline de clasificación multiclase.

### Principales tareas realizadas

- Carga de `DataSetInicial.csv`.
- Carga de `Columas a Eliminar.csv`.
- Eliminación de columnas indicadas por el EDA y eliminación adicional de `periodo`.
- Separación de variables predictoras `X` y variable objetivo `y`.
- División estratificada en `X_train`, `X_test`, `y_train`, `y_test`.
- Construcción del pipeline de preprocesamiento.
- Imputación de valores faltantes usando la moda.
- Agrupación de categorías en variables familiares e institucionales.
- Creación de la variable derivada `Mayor edad` desde `estu_tipodocumento`.
- Eliminación de `estu_tipodocumento` después del feature engineering.
- Codificación ordinal de variables con jerarquía.
- One Hot Encoding para variables categóricas nominales.
- Escalamiento posterior para estabilizar el entrenamiento de la regresión logística.
- Entrenamiento de un baseline con **Regresión Logística One-vs-One**.
- Evaluación sobre test usando:
  - Accuracy.
  - F1 macro.
  - F1 weighted.
  - Reporte de clasificación.
  - Matriz de confusión.
- Guardado del pipeline final entrenado con `joblib`.

### Salida esperada

```text
Data/Output/modelo_logistica_ovo_pipeline.joblib
```

---

## 8. Consideraciones metodológicas

### Evitar data leakage

Las variables de puntajes por prueba y el puntaje global se eliminan después de construir el target porque contienen información directamente relacionada con la variable objetivo. Mantenerlas dentro del entrenamiento generaría fuga de información y métricas artificialmente altas.

### Imputación dentro del pipeline

La imputación de valores faltantes se realiza dentro del pipeline de `scikit-learn`. Esto garantiza que los parámetros de imputación se aprendan únicamente con `X_train` y luego se apliquen sobre `X_test`, evitando contaminación del conjunto de prueba.

### Preprocesamiento aplicado a test

El pipeline se ajusta con:

```python
modelo.fit(X_train, y_train)
```

Y se evalúa con:

```python
y_pred = modelo.predict(X_test)
```

Esto asegura que `X_test` recibe exactamente las mismas transformaciones que `X_train`, pero sin recalcular imputadores, codificadores ni escaladores sobre los datos de prueba.

---

## 9. Modelo baseline

El modelo baseline implementado es una **Regresión Logística multiclase bajo estrategia One-vs-One**. Esta elección permite tener un primer punto de comparación interpretable y reproducible frente a modelos posteriores más flexibles.

El baseline no representa necesariamente el mejor modelo posible, sino una referencia inicial para evaluar mejoras futuras.

En fases posteriores se recomienda comparar este resultado con modelos como:

- Random Forest.
- XGBoost.
- CatBoost.
- ExtraTrees.
- Modelos con ajuste de hiperparámetros más amplio.
- Estrategias de balanceo o ponderación de clases.

---

## 10. Buenas prácticas para GitHub

Se recomienda no subir archivos pesados de datos ni artefactos entrenados si exceden los límites del repositorio. Para ello, incluir un archivo `.gitignore` con reglas como:

```text
Data/
*.csv
*.xlsx
*.joblib
*.pkl
.ipynb_checkpoints/
__pycache__/
.venv/
```

Si se desea compartir el modelo entrenado, puede usarse una alternativa como:

- Google Drive.
- GitHub Releases.
- DVC.
- Hugging Face Hub.
- Zenodo.

---

## 11. Comandos rápidos de ejecución

### Clonar repositorio

```bash
git clone <URL_DEL_REPOSITORIO>
cd TrabajoFinal-Saber11
```

### Instalar dependencias

```bash
pip install -r requirements.txt
```

### Abrir notebooks

```bash
jupyter notebook
```

### Ejecutar en orden

```text
notebooks/00_ObtenerDataset.ipynb
notebooks/01_EDA.ipynb
notebooks/02_Preprocesamiento.ipynb
```

---

## 12. Notas finales

Este proyecto construye una primera versión reproducible del flujo de Machine Learning para clasificación del desempeño académico en Saber 11. El principal valor de esta etapa es dejar consolidado el dataset, documentar las decisiones de limpieza y construir un baseline metodológicamente sólido. Las siguientes fases deben enfocarse en mejorar el desempeño predictivo mediante modelos no lineales, validación cruzada más robusta y análisis detallado de errores por clase.
