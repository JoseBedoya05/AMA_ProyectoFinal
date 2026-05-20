# Trabajo Final - Aprendizaje de Máquina Aplicado
## Clasificación del desempeño académico en resultados Saber 11

**Autor:** Jose Luis Bedoya Martínez  
**Programa:** Maestría en Ciencias de los Datos y la Analítica  
**Curso:** Aprendizaje de Máquina Aplicado  
**Tema:** Clasificación multiclase del nivel de desempeño académico en Pruebas Saber 11

---

## 1. Descripción general del proyecto

Este repositorio contiene el desarrollo final del proyecto del curso **Aprendizaje de Máquina Aplicado**. El trabajo aborda un problema de clasificación multiclase sobre los resultados de las Pruebas Saber 11 en Colombia. El objetivo es estimar el nivel de desempeño académico de un estudiante a partir de variables institucionales, familiares, geográficas y sociodemográficas, evitando usar como predictores las variables que contienen directamente el resultado de la prueba.

El proyecto parte de una base original de gran tamaño, realiza un proceso de depuración, construye una variable objetivo interpretable y compara diferentes modelos de clasificación. La versión final selecciona un modelo **SVM lineal implementado con `LinearSVC`**, entrenado dentro de un pipeline reproducible de `scikit-learn` y optimizado mediante `GridSearchCV` con validación cruzada estratificada.

El trabajo no busca afirmar que las variables disponibles explican completamente el desempeño académico. Más bien, busca construir un flujo metodológicamente defendible que permita identificar patrones asociados al nivel de desempeño, reconocer las limitaciones de los datos y dejar una base reproducible para futuras mejoras.

---

## 2. Pregunta y objetivo del proyecto

La pregunta central del proyecto puede formularse así:

> ¿Es posible clasificar el nivel de desempeño académico de un estudiante en Saber 11 usando variables institucionales, familiares, geográficas y sociodemográficas, sin utilizar directamente los puntajes de la prueba como predictores?

El objetivo general fue construir un modelo de clasificación multiclase que permitiera estimar cuatro niveles de desempeño:

- **Nivel 1: Insuficiente**
- **Nivel 2: Mínimo**
- **Nivel 3: Satisfactorio**
- **Nivel 4: Avanzado**

Para lograrlo, el proyecto consolidó un flujo completo de aprendizaje supervisado: obtención y depuración del dataset, análisis exploratorio, construcción del target, preprocesamiento, baseline, comparación de modelos, optimización del mejor modelo e interpretación final de resultados.

---

## 3. Fuente de datos

El conjunto de datos usado corresponde a los **Resultados Únicos Saber 11**, disponibles en el portal de Datos Abiertos Colombia.

- **Fuente:** Datos Abiertos Colombia - Resultados Únicos Saber 11
- **Base original:** 7.109.704 registros y 51 variables
- **Periodo modelado:** 2021 y 2022
- **Dataset final depurado:** 566.241 registros
- **Unidad de análisis:** cada fila representa el resultado de un estudiante en una aplicación de la prueba

Por tamaño y trazabilidad, los datos pesados no deberían versionarse directamente en GitHub. Se recomienda mantenerlos en Google Drive o en un sistema externo de almacenamiento y conservar en el repositorio únicamente notebooks, reportes, figuras, archivos livianos de configuración y documentación.

---

## 4. Estructura del repositorio

La estructura del repositorio está organizada para separar código, datos, figuras y entregables finales.

```text
AMA_ProyectoFinal/
│
├── data/
│   ├── input/              # Archivos de entrada o enlaces/zip livianos
│   └── output/             # Salidas generadas por los notebooks
│
├── figures/                # Figuras, datacard y visualizaciones de apoyo
│
├── notebooks/              # Notebooks principales del flujo de trabajo
│   ├── 00_ObtenerDataset.ipynb
│   ├── 01_EDA.ipynb
│   ├── 02_Preprocesamiento.ipynb
│   ├── 03_Version Final 4.ipynb
│   └── 04_SVM_Elegido_final.ipynb
│
├── poster/                 # Póster final del proyecto
│
├── report/                 # Informes de entrega y reporte final
│
├── README.md               # Documentación principal del repositorio
└── requirements.txt        # Dependencias principales del ambiente
```

---

## 5. Descripción de cada etapa realizada

### 5.1 Notebook 00 - Obtención, filtrado y depuración inicial

**Archivo:** `notebooks/00_ObtenerDataset.ipynb`

Este notebook construye el primer dataset modelable del proyecto. Su propósito es tomar la base original, aplicar reglas de negocio, construir la variable objetivo y generar una salida depurada para el análisis exploratorio.

Principales tareas realizadas:

- Carga del dataset original desde Google Drive.
- Filtrado por periodo, conservando los años **2021 y 2022**.
- Conservación de la población de interés.
- Conversión de puntajes a formato numérico.
- Construcción de una variable de puntaje usada para definir el target.
- Asignación del nivel de desempeño académico.
- Eliminación de registros con puntajes nulos, no convertibles o inválidos.
- Control de duplicados mediante `estu_consecutivo`.
- Exportación del dataset inicial depurado para las siguientes etapas.

Salida principal esperada:

```text
data/output/DataSetInicial.csv
```

---

### 5.2 Notebook 01 - Análisis exploratorio de datos

**Archivo:** `notebooks/01_EDA.ipynb`

Este notebook realiza el análisis exploratorio del dataset depurado. Dado que el conjunto está compuesto principalmente por variables categóricas, el EDA no se centra únicamente en distribuciones, sino también en calidad de datos, cardinalidad, valores faltantes y asociación entre variables.

Principales tareas realizadas:

- Revisión del tamaño y estructura del dataset.
- Identificación de valores nulos.
- Revisión de variables constantes y casi constantes.
- Análisis de cardinalidad de variables categóricas.
- Revisión de la distribución del target.
- Cálculo de asociación entre variables categóricas usando **V de Cramér**.
- Identificación de variables redundantes o de bajo aporte.
- Construcción del listado de columnas candidatas a eliminación.

Salida principal esperada:

```text
data/output/Columas a Eliminar.csv
```

El archivo conserva el nombre usado en los notebooks para mantener compatibilidad con el flujo existente.

---

### 5.3 Notebook 02 - Preprocesamiento y baseline

**Archivo:** `notebooks/02_Preprocesamiento.ipynb`

Este notebook construye el pipeline base de preprocesamiento y entrena el primer modelo de referencia. La decisión metodológica más importante de esta etapa fue encapsular imputación, codificación, escalamiento y modelo dentro de un mismo `Pipeline` de `scikit-learn`.

Principales tareas realizadas:

- Carga del dataset depurado.
- Carga del archivo de columnas a eliminar.
- Eliminación de variables con riesgo de leakage y de variables redundantes.
- Separación entre variables predictoras `X` y variable objetivo `y`.
- División estratificada de datos.
- Agrupación de categorías educativas de padre y madre.
- Agrupación de municipios de ubicación del colegio.
- Creación de la variable `Mayor edad` desde `estu_tipodocumento`.
- Imputación de valores faltantes dentro del pipeline.
- Codificación ordinal de variables con jerarquía.
- One Hot Encoding de variables nominales.
- Entrenamiento de un baseline con Regresión Logística One-vs-One.
- Evaluación del baseline con accuracy, F1 macro, F1 weighted, reporte de clasificación y matriz de confusión.

El baseline permitió construir una línea base metodológica. No se presenta como solución final, sino como punto de comparación para justificar la necesidad de modelos más robustos.

---

### 5.4 Notebook 03 - Comparación de modelos y selección del mejor enfoque

**Archivo:** `notebooks/03_Version Final 4.ipynb`

Este notebook amplía el trabajo del baseline y realiza una comparación entre varias familias de modelos. La evaluación se diseñó con separación entre entrenamiento, validación y test para evitar que el conjunto de prueba influyera en la selección del modelo.

Modelos evaluados:

- KNN Classifier
- SVM lineal con `LinearSVC`
- Decision Tree Classifier
- Random Forest Classifier
- CatBoost Classifier
- XGBoost Classifier

Principales decisiones metodológicas:

- Uso de `GridSearchCV` con **5 folds**.
- Uso de `StratifiedKFold` para conservar la distribución de clases.
- Selección del mejor modelo mediante **F1 macro**, no solo accuracy.
- Codificación numérica controlada del target: Nivel 1 → 1, Nivel 2 → 2, Nivel 3 → 3 y Nivel 4 → 4.
- Mantenimiento del preprocesamiento dentro del pipeline para evitar leakage.
- Reentrenamiento del modelo seleccionado con `train + val` antes de evaluar una única vez en test.

El resultado principal de esta etapa fue que el mejor modelo según F1 macro fue el **SVM lineal**, aunque otros modelos como XGBoost podían alcanzar mayor accuracy. La elección se hizo con F1 macro porque el problema es multiclase y desbalanceado; por tanto, no era suficiente premiar únicamente los aciertos globales.

---

### 5.5 Notebook 04 - Iteración final sobre SVM

**Archivo:** `notebooks/04_SVM_Elegido_final.ipynb`

Después de seleccionar SVM como mejor familia de modelo, se realizó una iteración final enfocada únicamente en mejorar su desempeño. Esta fase no cambió el planteamiento del proyecto, sino que profundizó en el modelo ganador mediante una búsqueda más amplia de hiperparámetros.

Principales tareas realizadas:

- Construcción del mismo pipeline de preprocesamiento usado en las etapas anteriores.
- Definición de un `LinearSVC` como clasificador final.
- Ejecución de un `GridSearchCV` ampliado para SVM.
- Exploración de valores de regularización `C`.
- Evaluación de penalización, tolerancia y balanceo de clases.
- Afinamiento local alrededor del mejor valor de `C` encontrado.
- Comparación de candidatos SVM en validación.
- Reentrenamiento final con `train + val`.
- Evaluación final en `test`.
- Guardado del pipeline completo con `joblib`.
- Exportación de métricas, resultados de validación, resultados de CV y mapeo del target.

Artefactos esperados de esta fase:

```text
data/output/modelo_svm_gridsearchcv_extendido_pipeline.joblib
data/output/resultados_svm_gridsearchcv_extendido_validacion.csv
data/output/cv_results_svm_gridsearchcv_amplio.csv
data/output/cv_results_svm_gridsearchcv_fino.csv
data/output/metricas_test_svm_gridsearchcv_extendido.csv
data/output/mapeo_target_svm_gridsearchcv_extendido.csv
```

---

## 6. Modelo final y resultados consolidados

El modelo final seleccionado fue un **SVM lineal con `LinearSVC`**, integrado dentro de un pipeline completo de preprocesamiento. La selección se realizó con F1 macro, dado que el target tiene cuatro clases y presenta desbalance.

Métricas finales en test:

| Métrica | Valor |
|---|---:|
| Accuracy | 0,4928 |
| F1 macro | 0,4366 |
| F1 weighted | 0,4917 |

Estos resultados muestran una mejora frente al baseline de regresión logística. Sin embargo, el desempeño debe interpretarse como moderado. El modelo logra capturar parte de la estructura del problema, pero todavía presenta dificultades para separar con precisión los niveles de desempeño, especialmente en las clases con menor representación.

Un patrón importante observado en la matriz de confusión es que los errores tienden a ocurrir entre clases cercanas. Esto significa que el modelo suele confundir niveles adyacentes, por ejemplo, **Insuficiente** con **Mínimo** o **Mínimo** con **Satisfactorio**. Este comportamiento sugiere que las fronteras entre niveles no son completamente separables con las variables disponibles y que el desempeño académico depende también de factores no observados en el dataset.

---

## 7. Buenas prácticas implementadas

El proyecto incorporó varias prácticas orientadas a garantizar reproducibilidad y control metodológico:

- Construcción del target antes de eliminar variables de puntaje.
- Eliminación posterior de variables que podían generar leakage.
- Control de duplicados mediante `estu_consecutivo`.
- División estratificada de datos.
- Separación de conjuntos de entrenamiento, validación y prueba.
- Uso de `Pipeline` y `ColumnTransformer` para encapsular preprocesamiento y modelo.
- Imputación, codificación y escalamiento ajustados únicamente con los datos permitidos en cada etapa.
- Uso de F1 macro como métrica principal de selección.
- Validación cruzada estratificada de 5 folds.
- Evaluación final una sola vez en test.
- Guardado del pipeline final como artefacto reproducible.

---

## 8. Cómo ejecutar el proyecto

### 8.1 Clonar el repositorio

```bash
git clone https://github.com/JoseBedoya05/AMA_ProyectoFinal.git
cd AMA_ProyectoFinal
```

### 8.2 Crear ambiente virtual local

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

### 8.3 Instalar dependencias

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

### 8.4 Registrar kernel de Jupyter

```bash
python -m ipykernel install --user --name saber11-ml --display-name "Python - Saber11 ML"
```

### 8.5 Abrir Jupyter

```bash
jupyter notebook
```

---

## 9. Ejecución en Google Colab

El proyecto fue desarrollado pensando en ejecución en Google Colab. Para usarlo allí:

```python
from google.colab import drive
drive.mount('/content/drive')
```

Luego se debe clonar el repositorio:

```bash
!git clone https://github.com/JoseBedoya05/AMA_ProyectoFinal.git
%cd AMA_ProyectoFinal
!pip install -r requirements.txt
```

La ruta base usada en el desarrollo fue:

```text
/content/drive/MyDrive/Aprendizaje de Maquina Aplicado/TrabajoFinal/
```

Dentro de esa ruta se recomienda conservar la estructura:

```text
TrabajoFinal/
└── Data/
    ├── Input/
    └── Output/
```

Si se cambia la ruta de Google Drive, se deben ajustar las variables `BASE_DIR`, `DATASET_PATH` y `COLUMNAS_ELIMINAR_PATH` en los notebooks.

---

## 10. Orden recomendado de ejecución

Para reproducir el flujo completo se recomienda ejecutar los notebooks en este orden:

```text
1. notebooks/00_ObtenerDataset.ipynb
2. notebooks/01_EDA.ipynb
3. notebooks/02_Preprocesamiento.ipynb
4. notebooks/03_Version Final 4.ipynb
5. notebooks/04_SVM_Elegido_final.ipynb
```

Cada notebook genera insumos para la etapa siguiente. Por eso, si se desea ejecutar el proyecto desde cero, es importante respetar el orden.

---

## 11. Entregables del proyecto

El repositorio contiene o está preparado para contener los siguientes entregables:

- **Notebooks reproducibles:** ubicados en `notebooks/`.
- **Reporte final:** ubicado en `report/`.
- **Póster final:** ubicado en `poster/`.
- **Figuras y datacard:** ubicadas en `figures/`.
- **Dependencias:** declaradas en `requirements.txt`.
- **README actualizado:** documento principal de navegación del repositorio.

---

## 12. Interpretación general de resultados

El mejor modelo fue SVM lineal porque presentó el mejor equilibrio entre clases cuando se evaluó con F1 macro. Aunque XGBoost podía alcanzar un valor de accuracy mayor durante la comparación, el criterio central del proyecto no era únicamente acertar en la clase mayoritaria, sino obtener un desempeño más balanceado en un problema multiclase y desbalanceado.

La confiabilidad de los resultados proviene del diseño de evaluación: el conjunto de test se mantuvo separado hasta el final, el preprocesamiento se ajustó dentro del pipeline y la selección de hiperparámetros se realizó mediante validación cruzada. Aun así, las métricas finales muestran que el modelo no debe entenderse como una solución definitiva de alta precisión, sino como un modelo reproducible y metodológicamente sólido con desempeño moderado.

Los patrones que explican el desempeño están asociados a la naturaleza del problema. Las variables disponibles describen condiciones institucionales, familiares y sociodemográficas, pero no capturan completamente elementos como trayectoria escolar, calidad pedagógica, motivación, preparación específica para la prueba o condiciones individuales. Por eso, el modelo encuentra señales útiles, pero no logra separar con alta precisión todos los niveles.

---

## 13. Limitaciones

Las principales limitaciones identificadas son:

- El target presenta desbalance, especialmente en el nivel Avanzado.
- Las variables disponibles no capturan todos los factores que influyen en el desempeño académico.
- La clasificación por niveles puede generar fronteras difusas entre clases cercanas.
- El desempeño final es moderado, por lo que no debe usarse para decisiones individuales de alto impacto.
- La base requiere recursos de procesamiento considerables, especialmente para búsquedas de hiperparámetros.

---

## 14. Trabajos futuros

Para mejorar o desplegar la solución se recomienda:

- Explorar estrategias adicionales de tratamiento del desbalance.
- Probar codificaciones alternativas para variables categóricas.
- Evaluar modelos ordinales, dado que los niveles de desempeño tienen un orden natural.
- Incorporar más variables educativas o contextuales si están disponibles.
- Analizar errores por subgrupos y regiones para detectar sesgos o comportamientos diferenciales.
- Construir un pipeline de inferencia más liviano para despliegue.
- Publicar artefactos pesados en Google Drive, DVC, GitHub Releases o un repositorio externo de modelos.
- Automatizar la ejecución con scripts o notebooks parametrizados.

---

## 15. Nota de reproducibilidad

El repositorio permite seguir la trazabilidad del proyecto desde la construcción del dataset hasta el modelo final. Para reproducir los resultados, se requiere contar con los datos originales en la estructura esperada de Google Drive o adaptar las rutas de los notebooks a la ubicación local de los archivos.

El modelo final debe cargarse junto con el pipeline completo guardado en `joblib`, ya que las transformaciones de preprocesamiento son parte esencial del flujo de inferencia. No se recomienda cargar únicamente el clasificador sin el pipeline, porque esto rompería la consistencia entre entrenamiento, evaluación y predicción.

