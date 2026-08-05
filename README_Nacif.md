# Predicción de resultados de partidos internacionales de fútbol

Proyecto final del curso de Data Science  
**Alumna:** Abril Nacif

## Descripción del proyecto

Este proyecto busca predecir el resultado de un partido internacional de fútbol a partir de información histórica. El problema se plantea como una **clasificación multiclase**, ya que el modelo puede devolver tres resultados posibles:

- Victoria de la Selección A.
- Empate.
- Victoria de la Selección B.

La idea principal fue comprobar hasta qué punto se puede anticipar un resultado utilizando solamente datos disponibles antes de cada encuentro. Para eso se analizaron partidos históricos, se crearon variables relacionadas con el rendimiento de las selecciones y se compararon seis modelos de Machine Learning.

El modelo final también se aplica de una forma sencilla: el usuario ingresa dos selecciones y la sede del partido, y el programa indica cuál ganaría o si el encuentro terminaría en empate.

## Pregunta y objetivo

La pregunta que guía el trabajo es la siguiente:

> ¿Podemos predecir el resultado de un partido internacional teniendo en cuenta el rendimiento reciente y el desempeño histórico de cada selección?

El objetivo fue construir, entrenar y comparar distintos modelos de clasificación para elegir el que presentara el desempeño más equilibrado al predecir las tres categorías. Para realizar la selección se utilizó principalmente el **F1-score macro**, ya que esta métrica le da la misma importancia a las victorias de A, los empates y las victorias de B.

## Dataset

El notebook carga el archivo `datos_oficial_partidos.csv` directamente desde este repositorio. La base contiene resultados de partidos internacionales y las siguientes columnas originales:

| Variable | Descripción |
|---|---|
| `date` | Fecha del partido. |
| `team_A` | Primera selección del registro. |
| `team_B` | Segunda selección del registro. |
| `score_A` | Goles de la Selección A. |
| `score_B` | Goles de la Selección B. |
| `tournament` | Torneo o tipo de partido. |
| `city` | Ciudad donde se disputó. |
| `country` | País donde se disputó. |
| `neutral` | Indica si se jugó en una sede neutral. |

La base inicial contiene **49.520 registros**. Después de eliminar los partidos que no tenían los datos indispensables, quedaron **47.399 registros válidos**. Finalmente, para el modelado se utilizaron **30.985 partidos disputados entre el 12 de enero de 1990 y el 19 de julio de 2026**.

El historial se limita al **31 de julio de 2026**, porque la aplicación final está pensada para realizar predicciones desde agosto de 2026. De esta manera se evita que los resultados futuros formen parte del entrenamiento.

## Organización del repositorio

```text
Proyecto-Final-Data-Science/
├── Trabajo_Final_Futbol_Nacif.ipynb
├── datos_oficial_partidos.csv
└── README.md
```

- `Trabajo_Final_Futbol_Nacif.ipynb`: contiene el desarrollo completo del proyecto.
- `datos_oficial_partidos.csv`: contiene los resultados históricos utilizados.
- `README.md`: explica el objetivo, el procedimiento y los resultados del trabajo.

## Etapas realizadas

### 1. Comprensión y limpieza de los datos

Primero se revisaron los tipos de datos, los valores faltantes, la cantidad de valores únicos y la existencia de duplicados. Después se realizaron las siguientes tareas:

- Conversión de la fecha al formato `datetime`.
- Conversión de los goles a valores numéricos enteros.
- Conversión de la variable `neutral` a booleana.
- Eliminación de espacios innecesarios en las columnas de texto.
- Eliminación de registros sin fecha, selecciones o goles.
- Control de marcadores negativos y enfrentamientos inválidos.
- Reemplazo de los valores faltantes de `neutral` por su valor más frecuente.
- Ordenamiento cronológico de todos los partidos.

También se conservaron los resultados con una cantidad de goles muy alta. Aunque el método del rango intercuartílico señaló **372 partidos extremos**, estos valores pueden corresponder a encuentros reales y no necesariamente a errores de carga.

### 2. Análisis exploratorio

Durante el análisis exploratorio se estudiaron:

- La distribución de victorias, empates y derrotas.
- La cantidad de partidos internacionales por año.
- Los torneos con mayor cantidad de encuentros.
- La distribución de goles y la presencia de valores extremos.
- La relación entre la sede y el resultado.
- La evolución de los resultados por década.
- La correlación entre las variables numéricas.

La categoría más frecuente fue la victoria de la Selección A, con un **48,59 %** de los partidos. Las victorias de la Selección B representaron un **27,97 %** y los empates un **23,43 %**.

También se observó una ventaja asociada a la localía. Cuando el partido no se disputó en sede neutral, la Selección A ganó el **50,65 %** de los encuentros. En sede neutral, ese porcentaje disminuyó al **43 %**.

### 3. Ingeniería de variables

Para cada partido se construyeron variables utilizando únicamente información de encuentros anteriores. Esto es importante para evitar que el modelo utilice información que todavía no existía al momento del partido.

Las principales variables creadas fueron:

- **Puntaje Elo:** representa la fortaleza histórica de cada selección.
- **Forma reciente:** promedio de puntos obtenidos en los últimos cinco partidos.
- **Goles a favor y en contra:** promedios de los últimos cinco encuentros.
- **Partidos previos:** cantidad de partidos disputados antes del encuentro.
- **Días de descanso:** diferencia entre los días transcurridos desde el último partido de cada selección.
- **Enfrentamientos directos:** puntos obtenidos en los últimos cruces entre ambas selecciones.
- **Diferencias entre A y B:** comparación de Elo, forma y experiencia.
- **Contexto del partido:** torneo, año, mes, sede neutral y ventaja de localía.

La construcción se realizó en orden cronológico. Primero se calcularon las variables previas y recién después se actualizó el historial con el resultado real. De esta manera se evitó la filtración de información futura o *data leakage*.

### 4. Separación de entrenamiento y prueba

Los datos no se dividieron de manera aleatoria, porque eso podría mezclar partidos antiguos y recientes. Se realizó una separación temporal:

- **Entrenamiento:** 24.788 partidos, desde el 12/01/1990 hasta el 10/10/2019.
- **Prueba:** 6.197 partidos, desde el 10/10/2019 hasta el 19/07/2026.

El 80 % inicial se utilizó para entrenar y optimizar los modelos. El 20 % más reciente se reservó para la evaluación final.

### 5. Preprocesamiento

El preprocesamiento se incorporó dentro de un `Pipeline` de scikit-learn:

- Los valores numéricos faltantes se completaron con la mediana.
- Las variables numéricas se estandarizaron con `StandardScaler`.
- Los faltantes categóricos se completaron con la categoría más frecuente.
- Las variables categóricas se transformaron mediante `OneHotEncoder`.
- Las categorías nuevas se configuraron para que no generaran errores durante una predicción.

### 6. Modelos evaluados

Se entrenaron y compararon seis modelos:

1. Regresión Logística.
2. K-Nearest Neighbors (KNN).
3. Árbol de Decisión.
4. Random Forest.
5. AdaBoost.
6. XGBoost.

Cada modelo se evaluó primero con una configuración inicial y después con una versión optimizada. Para buscar los mejores hiperparámetros se utilizó `GridSearchCV` junto con `TimeSeriesSplit` de cuatro divisiones. Así, cada validación respetó el orden temporal: el modelo se entrenó con el pasado y se validó con períodos posteriores.

## Métricas utilizadas

- **Accuracy:** proporción total de predicciones correctas.
- **Precision macro:** indica qué proporción de las predicciones de cada clase fue correcta y luego calcula el promedio entre las tres clases.
- **Recall macro:** indica qué proporción de cada resultado real pudo detectar el modelo.
- **F1-score macro:** combina precision y recall y les da la misma importancia a las tres clases.
- **ROC-AUC macro:** mide la capacidad del modelo para diferenciar cada resultado frente a los demás.

La métrica principal fue el **F1-macro**. Esto se debe a que los resultados no están completamente equilibrados y elegir solamente por accuracy podría favorecer a un modelo que predijera bien la clase más frecuente, pero tuviera un desempeño bajo con los empates.

## Resultados

Resultados de los modelos optimizados:

| Modelo | F1-macro en validación temporal | Accuracy en prueba | F1-macro en prueba | ROC-AUC macro |
|---|---:|---:|---:|---:|
| Regresión Logística | **0,4954** | 0,5683 | **0,5246** | **0,7356** |
| Árbol de Decisión | 0,4869 | 0,5664 | 0,5150 | 0,7191 |
| Random Forest | 0,4873 | 0,5779 | 0,5075 | 0,7277 |
| KNN | 0,4545 | 0,5709 | 0,4889 | 0,7009 |
| XGBoost | 0,4525 | 0,5925 | 0,4770 | 0,7278 |
| AdaBoost | 0,4098 | 0,6009 | 0,4367 | 0,7295 |

El modelo seleccionado fue la **Regresión Logística optimizada**, con los parámetros `C = 0.1` y `solver = "lbfgs"`. Fue elegida porque obtuvo el mayor F1-macro promedio en la validación temporal.

Su rendimiento final en el conjunto de prueba fue:

- Accuracy: **0,5683**.
- Precision macro: **0,5227**.
- Recall macro: **0,5308**.
- F1-score macro: **0,5246**.
- ROC-AUC macro: **0,7356**.

Aunque otros modelos obtuvieron una accuracy mayor, su F1-macro fue inferior. Esto significa que acertaban una mayor cantidad total de partidos, pero su rendimiento era menos equilibrado entre las tres clases. La Regresión Logística fue la opción que mejor respetó el criterio definido antes de evaluar el conjunto de prueba.

El reporte de clasificación también mostró que el empate fue el resultado más difícil de reconocer:

| Resultado | Precision | Recall | F1-score |
|---|---:|---:|---:|
| Victoria de la Selección A | 0,72 | 0,65 | 0,68 |
| Empate | 0,29 | 0,28 | 0,29 |
| Victoria de la Selección B | 0,55 | 0,66 | 0,60 |

## Cómo ejecutar el proyecto

### Opción 1: Google Colab

1. Abrir el notebook desde el botón **Abrir en Google Colab** ubicado al comienzo de este README.
2. En Colab, seleccionar `Entorno de ejecución > Ejecutar todas`.
3. Esperar a que se carguen los datos, se entrenen los modelos y finalice la optimización.
4. En la última celda, ingresar las dos selecciones y la sede cuando el programa lo solicite.

El dataset se carga desde una URL pública, por lo que no es necesario subirlo manualmente a Colab.

### Opción 2: ejecución local

Se necesita Python 3 y las siguientes librerías:

```bash
pip install numpy pandas matplotlib seaborn scikit-learn xgboost jupyter
```

Luego se debe clonar el repositorio, ingresar a la carpeta y abrir el notebook:

```bash
git clone https://github.com/abrilnacif2/Proyecto-Final-Data-Science.git
cd Proyecto-Final-Data-Science
jupyter notebook Trabajo_Final_Futbol_Nacif.ipynb
```

Una vez abierto, se deben ejecutar las celdas en orden desde el principio. La optimización de los seis modelos puede demorar algunos minutos, dependiendo del equipo utilizado.

## Cómo realizar una predicción

Al ejecutar la última celda aparecen tres preguntas:

```text
Primera selección: Argentina
Segunda selección: Marruecos
Sede (país o Neutral): Argentina
```

En este ejemplo, Argentina es local. Si Marruecos fuera local, se debería escribir `Marruecos` en la sede. Si ninguna selección juega como local, se debe escribir `Neutral`.

La salida tiene el siguiente formato:

```text
Partido: Argentina vs. Marruecos
Sede: Argentina
Predicción: Gana Argentina y pierde Marruecos.
```

La función acepta nombres de selecciones en español o en inglés, siempre que la selección tenga antecedentes dentro del dataset. También ignora diferencias entre mayúsculas, minúsculas y tildes en los nombres contemplados por el programa.

## Conclusiones

Este trabajo permitió aplicar las principales etapas de un proyecto de Data Science a un caso real: limpieza de datos, análisis exploratorio, creación de variables, preprocesamiento, entrenamiento, optimización y evaluación de modelos.

Los resultados muestran que la información histórica ayuda a reconocer ciertos patrones, especialmente la fortaleza representada por el Elo, la forma reciente, los enfrentamientos directos y la ventaja de localía. Sin embargo, predecir un partido de fútbol continúa siendo un problema difícil debido a la gran cantidad de situaciones imprevistas que pueden modificar el resultado.

También se comprobó que un modelo más complejo no siempre obtiene un mejor resultado. En este caso, la Regresión Logística optimizada presentó un desempeño más equilibrado que Random Forest, AdaBoost y XGBoost. Por lo tanto, el modelo final debe interpretarse como una estimación basada en antecedentes y no como una certeza.

## Limitaciones

- El historial utilizado finaliza el 31 de julio de 2026 y debe actualizarse para conservar su utilidad con el paso del tiempo.
- No se incluyen convocatorias, lesiones, formaciones, suspensiones, cambios de entrenador ni rendimiento individual de los jugadores.
- Tampoco se incorporan variables como el clima, la importancia del partido o el ranking oficial vigente en la fecha del encuentro.
- El empate es la clase menos frecuente y la más difícil de predecir.
- La aplicación final utiliza `Friendly` como torneo para simplificar el ingreso de datos.
- El modelo predice victoria, empate o derrota, pero no estima el marcador exacto.
- Solamente se pueden utilizar selecciones que tengan antecedentes en la base histórica.

## Posibles mejoras

Como continuación del proyecto se podrían incorporar datos de jugadores, rankings internacionales, lesiones, convocatorias y tipo de competencia. También sería posible actualizar el historial automáticamente, probar nuevas técnicas para mejorar la detección de empates, calibrar las probabilidades y desarrollar una interfaz gráfica para realizar las predicciones.

## Bibliografía y fuentes

- [Dataset del proyecto](https://github.com/abrilnacif2/Proyecto-Final-Data-Science/blob/main/datos_oficial_partidos.csv).
- [International football results, dataset original](https://github.com/martj42/international_results).
- [Documentación oficial de pandas](https://pandas.pydata.org/docs/).
- [Documentación oficial de scikit-learn](https://scikit-learn.org/stable/).
- [Documentación oficial de seaborn](https://seaborn.pydata.org/).
- Chen, T. y Guestrin, C. (2016). [*XGBoost: A Scalable Tree Boosting System*](https://doi.org/10.1145/2939672.2939785).

