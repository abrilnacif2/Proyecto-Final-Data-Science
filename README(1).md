<div align="center">

# Predicción de resultados de partidos de fútbol internacional

### Trabajo de Machine Learning aplicado a partidos entre selecciones nacionales

</div>

---

## Descripción del proyecto

En este proyecto desarrollamos un modelo de Machine Learning para predecir el resultado de un partido de fútbol internacional entre dos selecciones. A partir de una base histórica de encuentros, buscamos identificar patrones que permitan estimar cuál de los siguientes resultados es el más probable:

- Victoria de la Selección A.
- Empate.
- Victoria de la Selección B.

El trabajo no consiste solamente en entrenar modelos. Antes de llegar a esa etapa realizamos la carga, revisión y limpieza de los datos, analizamos sus principales características y creamos nuevas variables relacionadas con el rendimiento previo de cada selección. Finalmente, entrenamos seis modelos de clasificación, optimizamos sus parámetros y comparamos sus resultados mediante diferentes métricas.

Para que la predicción represente una situación real, todas las variables se calculan únicamente con la información disponible antes de cada partido. De esta manera, el modelo no conoce resultados futuros durante el entrenamiento.

---

## Objetivo

El objetivo principal del proyecto es construir y comparar distintos modelos de clasificación para determinar cuál logra predecir mejor los tres resultados posibles de un partido internacional.

Para alcanzar este objetivo nos propusimos:

- Analizar la calidad de una base histórica de partidos internacionales.
- Detectar y tratar valores faltantes, registros duplicados y tipos de datos incorrectos.
- Estudiar la distribución de las principales variables.
- Crear variables que representen la situación de cada selección antes de jugar.
- Dividir los datos respetando el orden cronológico.
- Entrenar diferentes modelos de clasificación.
- Optimizar sus hiperparámetros mediante validación temporal.
- Comparar los resultados sin favorecer únicamente a la clase mayoritaria.

---

## Fuente de los datos

Utilizamos como punto de partida la base pública [International football results](https://github.com/martj42/international_results), que reúne resultados históricos de partidos disputados entre selecciones nacionales.

Entre sus principales columnas se encuentran:

| Columna | Descripción |
|---|---|
| `date` | Fecha en la que se disputó el partido. |
| `home_team` | Selección que figura como local. |
| `away_team` | Selección que figura como visitante. |
| `home_score` | Goles marcados por la selección local. |
| `away_score` | Goles marcados por la selección visitante. |
| `tournament` | Torneo o competencia correspondiente. |
| `city` | Ciudad en la que se jugó el partido. |
| `country` | País donde se disputó el encuentro. |
| `neutral` | Indica si el partido se jugó en una sede neutral. |

Para trabajar el tratamiento de datos incompletos, usamos una versión modificada del archivo que contiene valores faltantes distribuidos de manera aleatoria y en distintas cantidades según la columna. Estos valores fueron analizados y tratados de acuerdo con la importancia y el tipo de cada variable.

---

## Etapas del trabajo

El proyecto se desarrolló siguiendo las siguientes etapas:

1. Carga de la base de datos.
2. Revisión general de las columnas y los tipos de datos.
3. Análisis de calidad de los datos.
4. Limpieza y preparación de la información.
5. Análisis exploratorio univariado y bivariado.
6. Creación de la variable que se desea predecir.
7. Ingeniería de nuevas variables.
8. División cronológica en entrenamiento y prueba.
9. Preprocesamiento de variables numéricas y categóricas.
10. Entrenamiento de seis modelos de clasificación.
11. Optimización de hiperparámetros.
12. Evaluación y comparación final de los modelos.

---

## Calidad y limpieza de los datos

Antes de entrenar los modelos revisamos si la base presentaba problemas que pudieran afectar los resultados. Para eso analizamos:

- La cantidad de filas y columnas.
- Los tipos de datos de cada variable.
- La presencia de valores faltantes.
- La existencia de registros duplicados.
- La consistencia de las fechas y los resultados.
- La presencia de goles negativos o valores imposibles.

Las columnas necesarias para conocer qué selecciones jugaron, cuándo se disputó el partido y cuál fue el resultado se consideraron críticas. Cuando faltaba alguno de esos datos, el registro no podía utilizarse para construir el historial ni para entrenar los modelos.

En cambio, los valores faltantes de las variables predictoras se trataron dentro del preprocesamiento. Los faltantes numéricos se imputaron con un valor representativo y los categóricos con una categoría adecuada. Esto permitió conservar la mayor cantidad posible de observaciones sin introducir información del conjunto de prueba en el entrenamiento.

También convertimos la fecha al formato correspondiente, ordenamos los partidos cronológicamente y creamos nombres de variables más claros para trabajar con la Selección A y la Selección B.

---

## Variable objetivo

La variable objetivo, llamada `target`, representa el resultado que el modelo debe aprender a predecir. Se construyó comparando los goles marcados por las dos selecciones:

| Resultado del partido | Categoría asignada |
|---|---|
| La Selección A marcó más goles | Victoria de la Selección A |
| Ambas selecciones marcaron la misma cantidad | Empate |
| La Selección B marcó más goles | Victoria de la Selección B |

Se trata de un problema de clasificación multiclase porque existen tres categorías posibles y el modelo debe elegir una de ellas para cada partido.

---

## Ingeniería de variables

La base original muestra lo que ocurrió en cada encuentro, pero no describe de manera directa cómo llegaba cada selección antes de jugar. Por ese motivo creamos nuevas variables utilizando los partidos anteriores de cada equipo.

| Variable | Explicación |
|---|---|
| Puntaje Elo | Representa la fortaleza de cada selección. Todas comienzan con 1500 puntos y el valor se actualiza después de cada encuentro según el resultado y la dificultad del rival. |
| Forma reciente | Promedio de puntos obtenidos en los últimos cinco partidos. Se asignan 3 puntos por victoria, 1 por empate y 0 por derrota. |
| Goles recientes a favor | Promedio de goles marcados por cada selección en sus últimos cinco encuentros. |
| Goles recientes en contra | Promedio de goles recibidos por cada selección en sus últimos cinco encuentros. |
| Partidos previos | Cantidad de partidos que había disputado cada selección hasta ese momento. |
| Días de descanso | Cantidad de días transcurridos desde el último partido de cada selección. |
| Enfrentamientos directos | Rendimiento previo de la Selección A frente a la Selección B. |
| Diferencias entre A y B | Resta entre los valores de ambas selecciones, por ejemplo la diferencia de Elo, forma, experiencia o descanso. |
| Información del encuentro | Torneo, año, mes, condición neutral y ventaja de sede de la Selección A. |

Un valor positivo en una diferencia indica que la Selección A presenta un valor mayor que la Selección B. Un valor negativo indica lo contrario.

### Prevención de la fuga de información

Las variables se construyeron respetando el orden cronológico. Antes de cada partido calculamos el Elo, la forma, los goles recientes, el descanso y los enfrentamientos directos utilizando solamente los encuentros anteriores. Recién después incorporamos el resultado actual al historial.

Este orden es importante porque evita la fuga de información. Si utilizáramos el resultado del propio partido o información posterior para calcular una variable, el modelo recibiría datos que no estarían disponibles al momento de realizar una predicción real.

---

## Análisis exploratorio

Realizamos un análisis exploratorio para comprender cómo se distribuyen los datos antes de entrenar los modelos.

En el análisis univariado estudiamos cada variable por separado. Observamos, por ejemplo, la frecuencia de los tres resultados, la distribución de los goles, los torneos con mayor cantidad de partidos y el comportamiento de las variables numéricas creadas.

En el análisis bivariado comparamos dos variables para analizar si existía alguna relación entre ellas. Esto permitió estudiar cómo cambiaban el Elo, la forma reciente, los goles o el descanso según el resultado del encuentro.

---

## Análisis ANOVA

Utilizamos ANOVA para comprobar si una variable numérica presenta valores promedio diferentes según el resultado del partido.

Por ejemplo, para analizar la diferencia de Elo se separan los encuentros en tres grupos:

- Partidos ganados por la Selección A.
- Partidos empatados.
- Partidos ganados por la Selección B.

Luego, ANOVA compara el promedio de la diferencia de Elo entre esos tres grupos. El p-valor se interpreta de la siguiente manera:

| Resultado | Interpretación |
|---|---|
| p-valor menor a 0,05 | Existen diferencias significativas entre el promedio de al menos dos grupos. La variable podría aportar información para distinguir los resultados. |
| p-valor mayor o igual a 0,05 | No encontramos evidencia suficiente para afirmar que los promedios sean diferentes. |

ANOVA se utilizó como una herramienta exploratoria. Que una variable tenga un p-valor menor a 0,05 no garantiza que un modelo vaya a predecir correctamente, ya que la prueba analiza cada variable por separado. El desempeño real se determina posteriormente al entrenar y evaluar los modelos con datos que no fueron utilizados durante el entrenamiento.

---

## División de los datos

La división entre entrenamiento y prueba se realizó respetando el tiempo. Los partidos más antiguos se utilizaron para entrenar los modelos y los más recientes se reservaron para la evaluación final.

No utilizamos una división aleatoria porque podría provocar que el modelo aprenda con partidos futuros y luego sea evaluado con encuentros anteriores. La división cronológica representa mejor el uso real del modelo: aprender a partir del pasado para intentar predecir partidos posteriores.

---

## Preprocesamiento

El preprocesamiento se integró con cada modelo mediante un `Pipeline` de Scikit-learn. Esto permite que las mismas transformaciones se apliquen de forma ordenada durante el entrenamiento, la validación y la prueba.

El procedimiento incluye:

- Imputación de valores faltantes numéricos.
- Tratamiento de valores faltantes categóricos.
- Escalado de las variables numéricas.
- Codificación de las variables categóricas con `OneHotEncoder`.
- Conversión de la variable objetivo a valores numéricos cuando el modelo lo requiere.

El uso de pipelines también reduce el riesgo de fuga de información, ya que las transformaciones se ajustan solamente con los datos correspondientes al entrenamiento.

---

## Modelos utilizados

Entrenamos seis modelos de clasificación para comparar enfoques diferentes:

| Modelo | Funcionamiento general |
|---|---|
| Regresión Logística | Estima la probabilidad de cada resultado a partir de la relación entre las variables. Se utiliza como modelo inicial de referencia. |
| K-Nearest Neighbors | Busca los partidos históricos más parecidos al encuentro que se desea clasificar. |
| Árbol de Decisión | Construye reglas de decisión mediante divisiones sucesivas de los datos. |
| Random Forest | Combina una gran cantidad de árboles de decisión para obtener una clasificación más estable. |
| AdaBoost | Entrena varios clasificadores simples de manera secuencial y presta mayor atención a los casos que fueron clasificados incorrectamente. |
| XGBoost | Construye árboles de forma secuencial, donde cada árbol nuevo intenta corregir los errores de los anteriores. |

La comparación incluye modelos simples y modelos de ensamble. Esto nos permite observar si los métodos más complejos realmente mejoran el desempeño sobre los datos del proyecto.

---

## Optimización de hiperparámetros

Después del entrenamiento inicial utilizamos `GridSearchCV` para probar distintas combinaciones de hiperparámetros. Algunos ejemplos son la cantidad de vecinos de KNN, la profundidad de los árboles, la cantidad de estimadores y la tasa de aprendizaje.

La validación se realizó con `TimeSeriesSplit`, utilizando cuatro divisiones cronológicas. En cada división, el modelo se entrena con un período anterior y se valida con un período posterior. De esta manera, la búsqueda de parámetros también respeta el orden temporal de los partidos.

La mejor combinación se selecciona según el F1-macro obtenido en la validación.

---

## Métricas de evaluación

Para comparar los modelos utilizamos varias métricas, ya que una sola medida no alcanza para describir todo su comportamiento.

| Métrica | Qué representa |
|---|---|
| Accuracy | Proporción total de partidos clasificados correctamente. |
| Precision macro | Indica qué tan correctas fueron las predicciones realizadas para cada categoría y luego calcula un promedio entre las tres. |
| Recall macro | Mide qué proporción de cada resultado real pudo detectar el modelo. |
| F1-macro | Combina precision y recall, dando la misma importancia a Victoria de A, Empate y Victoria de B. |
| ROC-AUC macro | Evalúa la capacidad del modelo para diferenciar cada categoría frente a las demás a partir de sus probabilidades. |

La métrica principal es F1-macro. La elegimos porque calcula el rendimiento de cada categoría por separado y luego realiza un promedio, sin permitir que la clase más frecuente tenga mayor importancia que las demás.

---

## Evaluación de los resultados

Primero evaluamos los seis modelos con sus parámetros iniciales. Luego realizamos la optimización y volvimos a medir su desempeño sobre el conjunto de prueba.

La comparación final se realiza principalmente mediante F1-macro, pero también observamos accuracy, precision, recall y ROC-AUC. Además, analizamos la matriz de confusión para identificar qué resultados reconoce mejor cada modelo y cuáles tiende a confundir.

Los valores obtenidos y la selección del modelo final se muestran en las últimas secciones del notebook. No se incluyen números fijos en este README porque pueden cambiar si se modifica la base, el tratamiento de los datos o los parámetros evaluados.

---

## Herramientas utilizadas

- Python.
- Google Colab.
- Pandas.
- NumPy.
- Matplotlib.
- Seaborn.
- SciPy.
- Scikit-learn.
- XGBoost.

---

## Contenido del repositorio

| Archivo | Contenido |
|---|---|
| Notebook `.ipynb` | Contiene el desarrollo completo del proyecto, desde la carga de datos hasta la evaluación de los modelos. |
| Dataset `.csv` | Contiene la base de partidos internacionales utilizada en el análisis. |
| `README.md` | Explica el objetivo, la metodología y los pasos necesarios para reproducir el trabajo. |

---

## Cómo ejecutar el proyecto

### Opción 1: Google Colab

1. Descargar el notebook desde este repositorio.
2. Ingresar a [Google Colab](https://colab.research.google.com/).
3. Seleccionar la opción para subir un notebook.
4. Cargar el archivo `.ipynb`.
5. Verificar que el enlace al dataset sea correcto o subir el archivo CSV.
6. Ejecutar todas las celdas en orden desde el comienzo.

### Opción 2: Jupyter Notebook

1. Clonar o descargar el repositorio.
2. Instalar las librerías necesarias:

```bash
pip install pandas numpy matplotlib seaborn scipy scikit-learn xgboost
```

3. Abrir el archivo `.ipynb` en Jupyter Notebook.
4. Verificar la ruta del dataset.
5. Ejecutar las celdas en el orden en el que aparecen.

---

## Reproducibilidad

Para que otra persona pueda repetir el análisis y obtener resultados comparables:

- El repositorio incluye el notebook completo.
- La fuente de los datos se encuentra identificada.
- Los partidos se procesan en orden cronológico.
- Las transformaciones se realizan mediante pipelines.
- Se utiliza una semilla fija en los modelos que lo permiten.
- La optimización respeta el orden temporal mediante `TimeSeriesSplit`.
- El notebook explica los pasos realizados y muestra los resultados de cada etapa.

---

## Conclusión

En este trabajo aplicamos las principales etapas de un proyecto de Machine Learning a un caso relacionado con el fútbol internacional. Partimos de una base histórica con problemas de calidad, realizamos su limpieza y análisis, y construimos variables que representan el rendimiento previo de cada selección.

Luego entrenamos modelos con funcionamientos diferentes y los comparamos utilizando métricas adecuadas para un problema con tres categorías. También optimizamos sus hiperparámetros mediante una validación que respeta el orden cronológico.

El aspecto más importante de la metodología es que cada partido se analiza solamente con información disponible hasta ese momento. Por lo tanto, la evaluación se acerca más a una situación real de predicción y evita que los modelos obtengan resultados artificialmente altos por utilizar información futura.

Este proyecto nos permitió aplicar de forma práctica conceptos de limpieza de datos, análisis exploratorio, ingeniería de variables, clasificación, optimización y evaluación de modelos.
