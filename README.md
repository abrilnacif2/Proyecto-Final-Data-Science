<div align="center">

# ⚽ Predicción de resultados de partidos de fútbol internacional

### Proyecto de Machine Learning para clasificar el resultado de un partido entre selecciones

![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Google Colab](https://img.shields.io/badge/Google%20Colab-Notebook-F9AB00?style=for-the-badge&logo=googlecolab&logoColor=white)
![Scikit-learn](https://img.shields.io/badge/Scikit--learn-ML-F7931E?style=for-the-badge&logo=scikitlearn&logoColor=white)
![XGBoost](https://img.shields.io/badge/XGBoost-Ensemble-189AB4?style=for-the-badge)

</div>

---

## 📌 Descripción del proyecto

Este proyecto aplica técnicas de **Machine Learning** para predecir el resultado de un partido de fútbol internacional entre dos selecciones.

El problema se plantea como una clasificación multiclase con tres resultados posibles:

- **Victoria de la Selección A**.
- **Empate**.
- **Victoria de la Selección B**.

Para realizar la predicción se utiliza información disponible **antes de cada partido**, como la fortaleza histórica de las selecciones, su rendimiento reciente, los goles marcados y recibidos, los días de descanso y sus enfrentamientos directos.

---

## 🎯 Objetivo

El objetivo principal es construir, entrenar y comparar distintos modelos de clasificación para identificar cuál logra predecir con mayor equilibrio los tres resultados posibles de un partido.

Además, el proyecto busca:

- Limpiar y preparar una base histórica de partidos internacionales.
- Analizar los datos mediante técnicas univariadas y bivariadas.
- Crear variables que representen el estado de cada selección antes del partido.
- Evitar el uso de información futura durante el entrenamiento.
- Comparar modelos con métricas adecuadas para clasificación multiclase.
- Optimizar sus hiperparámetros mediante validación temporal.

---

## 📂 Fuente de datos

Se utiliza una base histórica de resultados de partidos internacionales de selecciones nacionales, publicada en el repositorio [International football results](https://github.com/martj42/international_results).

La base original contiene información como:

- Fecha del partido.
- Selección local y visitante.
- Goles de cada selección.
- Torneo disputado.
- Ciudad y país del encuentro.
- Condición de sede neutral.

Para trabajar el tratamiento de datos incompletos, la versión utilizada en el proyecto contiene **valores faltantes distribuidos de forma aleatoria**. Los campos críticos se validan antes de crear las variables y los faltantes de las variables predictoras se tratan dentro del pipeline de preprocesamiento.

---

## 🔄 Etapas del proyecto

1. **Carga de datos:** importación del archivo CSV y revisión inicial de su estructura.
2. **Calidad de datos:** análisis de valores faltantes, duplicados, tipos de datos y consistencia.
3. **Limpieza:** conversión de fechas y variables numéricas, tratamiento de faltantes y eliminación de registros que no pueden utilizarse.
4. **Análisis exploratorio:** estudio de la distribución de resultados, goles, torneos y otras variables relevantes.
5. **Análisis bivariado y ANOVA:** evaluación de la relación entre las variables numéricas y el resultado del partido.
6. **Ingeniería de variables:** creación de indicadores históricos y recientes para cada selección.
7. **Preprocesamiento:** imputación, escalado de variables numéricas y codificación de variables categóricas.
8. **Entrenamiento:** comparación de seis modelos de clasificación.
9. **Optimización:** búsqueda de mejores hiperparámetros con `GridSearchCV`.
10. **Evaluación final:** comparación de los modelos sobre partidos que no fueron utilizados durante el entrenamiento.

---

## 🛠️ Ingeniería de variables

Las variables se calculan respetando el orden cronológico de los partidos. Primero se genera la información disponible antes del encuentro y recién después se incorpora su resultado al historial.

| Variable | Descripción |
|---|---|
| **Puntaje Elo** | Representa la fortaleza de cada selección. Todas comienzan con 1500 puntos y el valor se actualiza después de cada partido. |
| **Forma reciente** | Promedio de puntos obtenidos en los últimos cinco partidos: 3 por victoria, 1 por empate y 0 por derrota. |
| **Goles recientes a favor** | Promedio de goles marcados por cada selección en sus últimos cinco encuentros. |
| **Goles recientes en contra** | Promedio de goles recibidos por cada selección en sus últimos cinco encuentros. |
| **Partidos previos** | Cantidad de encuentros disputados anteriormente por cada selección. |
| **Días de descanso** | Días transcurridos desde el último partido de cada selección. |
| **Enfrentamientos directos** | Rendimiento de la Selección A en los últimos partidos disputados contra la Selección B. |
| **Diferencias A-B** | Comparación entre las selecciones mediante la resta de sus valores de Elo, forma, goles, experiencia y descanso. |
| **Información del partido** | Torneo, año, mes, condición neutral y ventaja de sede de la Selección A. |

Este procedimiento evita la **fuga de información**, ya que ninguna variable utiliza datos posteriores al partido que se intenta predecir.

---

## 📊 Análisis bivariado y ANOVA

Se utiliza **ANOVA** para analizar si el promedio de cada variable numérica cambia según el resultado del partido.

Por ejemplo, se compara el promedio de la diferencia de Elo entre:

- Los partidos ganados por la Selección A.
- Los partidos empatados.
- Los partidos ganados por la Selección B.

Un **p-valor menor a 0,05** indica que existen diferencias significativas entre al menos dos de los grupos. Esto sugiere que la variable podría aportar información útil para distinguir los resultados.

ANOVA se utiliza como análisis exploratorio: la capacidad real de predicción se comprueba posteriormente mediante el entrenamiento y la evaluación de los modelos.

---

## 🤖 Modelos evaluados

| Modelo | Descripción |
|---|---|
| **Regresión Logística** | Modelo lineal utilizado como referencia inicial. |
| **K-Nearest Neighbors (KNN)** | Clasifica cada partido según los casos históricos más parecidos. |
| **Árbol de Decisión** | Crea reglas de decisión a partir de las variables disponibles. |
| **Random Forest** | Combina múltiples árboles para obtener una predicción más estable. |
| **AdaBoost** | Combina clasificadores débiles y aumenta la importancia de los casos difíciles. |
| **XGBoost** | Construye árboles secuencialmente para corregir los errores de los anteriores. |

---

## ⚙️ Preprocesamiento

El preprocesamiento se integra con cada modelo mediante un `Pipeline` de Scikit-learn. Esto permite aplicar exactamente las mismas transformaciones durante el entrenamiento y la evaluación.

- Las variables numéricas faltantes se imputan y luego se escalan.
- Las variables categóricas faltantes se completan y se transforman mediante `OneHotEncoder`.
- La variable objetivo se codifica numéricamente cuando el modelo lo requiere.
- Se utiliza una semilla fija para facilitar la reproducción de los resultados.

---

## ⏳ División y validación temporal

Los datos no se dividen aleatoriamente. Los partidos más antiguos se utilizan para entrenar y los más recientes para evaluar, simulando una situación real en la que se intenta predecir el futuro a partir del pasado.

Para optimizar los hiperparámetros se utiliza:

- `TimeSeriesSplit` con cuatro divisiones cronológicas.
- `GridSearchCV` para probar diferentes combinaciones de parámetros.
- **F1-macro** como criterio para seleccionar la mejor combinación.

---

## 📏 Métricas de evaluación

| Métrica | ¿Qué mide? |
|---|---|
| **Accuracy** | Proporción total de predicciones correctas. |
| **Precision macro** | Exactitud promedio de las predicciones para las tres categorías. |
| **Recall macro** | Capacidad promedio para detectar cada resultado real. |
| **F1-macro** | Equilibrio entre precision y recall, dando la misma importancia a las tres categorías. |
| **ROC-AUC macro** | Capacidad del modelo para diferenciar cada resultado frente a los demás. |

La métrica principal es **F1-macro**, porque permite comparar los modelos sin favorecer a la categoría que aparece con mayor frecuencia.

---

## 🧰 Tecnologías utilizadas

- Python
- Google Colab / Jupyter Notebook
- Pandas
- NumPy
- Matplotlib
- Seaborn
- SciPy
- Scikit-learn
- XGBoost

---

## ▶️ Cómo ejecutar el proyecto

1. Clonar o descargar este repositorio.
2. Abrir el archivo `.ipynb` en **Google Colab** o **Jupyter Notebook**.
3. Verificar que el archivo CSV se encuentre disponible o que su enlace esté correctamente configurado.
4. Instalar las librerías necesarias si el entorno todavía no las incluye:

```bash
pip install pandas numpy matplotlib seaborn scipy scikit-learn xgboost
```

5. Ejecutar todas las celdas en orden desde el comienzo.

En Google Colab se puede utilizar la opción:

```text
Entorno de ejecución → Ejecutar todas
```

---

## 📁 Contenido del repositorio

| Archivo | Contenido |
|---|---|
| **Notebook `.ipynb`** | Desarrollo completo del análisis, procesamiento, entrenamiento y evaluación. |
| **Dataset `.csv`** | Base de partidos internacionales utilizada en el proyecto. |
| **README.md** | Descripción, metodología e instrucciones para reproducir el trabajo. |

---

## ✅ Reproducibilidad

Para facilitar la reproducción del proyecto:

- El procesamiento respeta el orden cronológico.
- Las transformaciones se realizan mediante pipelines.
- Los modelos utilizan una semilla fija cuando lo permiten.
- La optimización se realiza con divisiones temporales.
- El notebook contiene el código completo y los resultados de cada etapa.

---

## 🏁 Conclusión

El proyecto desarrolla un proceso completo de Machine Learning aplicado a resultados de fútbol internacional: desde la limpieza de los datos y la creación de variables hasta la comparación y optimización de distintos modelos.

La metodología busca que la evaluación sea realista, ya que cada predicción se realiza únicamente con información disponible antes del partido. Finalmente, los modelos se comparan mediante métricas multiclase, con especial atención al **F1-macro**, para determinar cuál logra el mejor equilibrio entre la predicción de victorias, empates y derrotas.

---

<div align="center">

### ⚽ Datos históricos, análisis y Machine Learning aplicados al fútbol

</div>
