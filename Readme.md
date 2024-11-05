# Predicción de Riesgo de Evento Cardíaco con Señales de ECG Usando la Base de Datos PTB Diagnostic ECG y PTB-XL.

El proyecto se centra en la predicción del riesgo de eventos cardíacos utilizando registros de ECG (electrocardiogramas) de la base de datos PTB Diagnostic ECG y PTB-XL de PhysioNet. La finalidad es desarrollar un modelo de aprendizaje profundo que ayude a identificar pacientes en riesgo de sufrir eventos cardíacos, basándose en características extraídas de los datos electrocardiográficos.

## Etapas contenidas en este proyecto

- I. Análisis exploratorio de datos.
- II. Selección de modelos
- III. Entrenamiento y evaluación de modelos
- IV. Comunicación de Resultados

## Etapa I Análisis exploratorio de datos

Respecto a esta etapa, se realizaron las siguientes actividades:

### Identificación de las propiedades de los registros ECG

Mediante el uso de la librería `wfdb` se identificaron propiedades relevantes de los registros ECG de la base de datos PTB Diagnostic ECG. 

En este caso, tales registros tienen las siguientes propiedades:
- **record_name**: Nombre del registro.
- **n_sig**: Número de señales en el registro.
- **fs**: Frecuencia de muestreo de las señales.
- **sig_name**: Nombre de las señales.
- **sig_len**: Longitud de las señales.
- **comments**: Comentarios sobre el registro.
- **p_signal**: Señales físicas.

Como se puede observar, en el atributo 'comments' existe una lista de información relevante sobre el paciente, en la cual se puede identificar que el valor de la llave **'Reason for admission'** clasifica a al registro como _'Myocardal infarction'_ en este caso. También existe la clave **'Acute infarction (localization)'** que indica la localización del infarto.

Debido a que esta base de datos tiene registros tanto de pacientes con Infarto al miocardio y de pacientes sanos, se procedió a realizar una busqueda de los registros que contienen la clave **'Reason for admission'** con el valor _'Myocardal infarction'_ para identificar los registros de pacientes con infarto al miocardio.

Además se identificó los valores únicos relacionados a la clave **'Acute infarction (localization)'** para identificar las localizaciones del infarto ya que en este apartado existe información faltante y ambigüa como se puede observar en la siguiente imagen:

![Distribución de locaciones](/fig/localizaciones_registros.png)


Para realizar la limpieza de estos elementos, se procedió a realizar un filtro en el cual los registros que contienen los valores **'no'**, **'n/a'** y **'unknown'** fueron eliminados para el análisis de datos dando como resultado la siguiente distribución de locaciones:

![Distribución de locaciones filtradas](/fig/localizaciones_registros_filtrados.png)

### Identificación de la distribución de las clases

Se identifica la distribución de las clases STEMI y HC (Healthy Control) en la base de datos PTB Diagnostic ECG mediante el uso de un atributo "comment" propio de cada registro dando como resultado los las siguientes cantidades:

![Distribución de las clases](/fig/distribucion_preliminar.png)

### Identificación de problematica y solución

Se identifica la problematica de la base de datos PTB Diagnostic ECG, la cual se refiere al desbalance de clases respecto a los registros STEMI y HC. La solución seleccionada respecto a la problematica está relacionada al uso de otra base de datos (PTB-XL) para obtener y balancear los registros HC (Clase minoritaria).

Otras soluciones propuestas fueron:
1. **Uso de técnicas de resampling**: Se puede utilizar técnicas de resampling como oversampling o undersampling para balancear las clases. Oversampling puede generar nuevos ejemplos de la clase minoritaria, mientras que undersampling puede eliminar ejemplos de la clase mayoritaria.
2. **Generación de datos sintéticos**: Se pueden generar datos sintéticos para la clase minoritaria utilizando técnicas como SMOTE (Synthetic Minority Over-sampling Technique) o ADASYN (Adaptive Synthetic Sampling Approach).
3. **Ajuste de pesos de clase**: Algunos algoritmos de machine learning permiten ajustar los pesos de clase para penalizar más los errores en la clase minoritaria. Esto puede ayudar al modelo a prestar más atención a los ejemplos de la clase minoritaria.

Estas soluciones fueron desechadas debido a que se priorizó la obtención de registros reales asi evitando un sesgo creado por la generación de datos sintéticos o el ajuste de pesos de clase.

### Identificación de las propiedades de los registros ECG de la base de datos PTB-XL

Mediante el uso de la librería `wfdb` se identificaron las principales diferencias entre los registros ECG de la base de datos PTB Diagnostic ECG y PTB-XL.

En este caso, tales registros tienen las siguientes diferencias:

- **fs**: 500 Hz
- **comments**: Vacío

Los registros de la base de datos PTB-XL no tienen información en el atributo 'comments' por lo tanto su carga de datos para clasificarlos  se debe de realizar de una manera distinta a la realizada con la base de datos PTB Diagnostic ECG.

En este caso, la información referida a los diagnosticos y etiquetas estan registradas en un .csv llamado ptbxl_database.csv.

Respecto a la frecuencia de muestreo, esta es diferente e inferior a los 1000hz de la base de datos PTB Diagnostic ECG.

### Obtención de registros de pacientes sanos de la base de datos PTB-XL

Para esto, se debió realizar un proceso de filtrado de los registros de la base de datos PTB-XL, donde se seleccionaron los registros que no contenían información relacionada a enfermedades cardíacas.
En este proceso se hizo uso de la información contenida en el archivo ptbxl_database.csv, el cual en su columna 'scp_codes' contiene la codificación referida a las enfermedades cardíacas de los pacientes.
En este caso, se hizo una busqueda de los id los registros que contengan el siguiente string: *'{'NORM': 100.0, 'SR': 0.0}'* lo que representa que el registro contiene una señal de corazon que está latiendo de manera saludable y coordinada.

### Distribución de las clases luego de incorporar PTB-XL

Luego de incorporar los registros de la base de datos PTB-XL, se obtuvo la siguiente distribución de las clases:

![Distribución de las clases luego de incorporar PTB-XL](/fig/distribucion_final_registros.png)

### Preprocesamiento de los registros

Se realizó un preprocesamiento de los registros ECG de ambas bases de datos, el cual consistió en:

- **Guardado de registros STEMI** en sus respectivo directorio para su procesamiento.
- **Eliminación de canales de posicion de electrodos** identificando los nombres Vx, Vy y Vz.
- **Downsampling de canales** de registros STEMI de 1000 a 500 Hz. 
- **Filtrado de ruido** mediante un filtro pasa banda Butterworth de 0.67 a 30 Hz.

La eliminación de la información de los canales de posicion de los electrodos se realizó debido a que no serán utilizados en el desarrollo de este proyecto.

Los registros añadidos de la base de datos PTB-XL no fueron preprocesados debido a que ya presentaban un filtrado de ruido y una frecuencia de muestreo de 500 Hz.

La reducción de la frecuencia de muestreo de los registros STEMI de 1000 a 500 Hz se realizó debido a que los registros de la base de datos PTB-XL tienen una frecuencia de muestreo de 500 Hz, por lo que se decidió homogeneizar la frecuencia de muestreo de todos los registros a 500 Hz.

Un ejemplo de la señal preprocesada de los registros STEMI se puede apreciar en la siguiente imagen:

![Comparativa de señales](/fig/muestr_señales_procesadas.png)

Se puede observar la disminució de las muestras en cada canal debido al downsampling. Tambien se puede percibi la eliminación de ruido en la señal como resultado una señal mas estable y alineada.


## Etapa II Selección de modelos

El modelo seleccionado para la clasificación de los registros ECG de la base de datos PTB Diagnostic ECG y PTB-XL fue el de Random Forest debido a que es menos susceptible a ser influenciado por valores atípicos. Ademas es un modelo rápido de entrenar y evaluar.

Respecto a su relación con la problematica del proyecto, Random Forest se seleccionó debido a que permite la clasificación de registros STEMI y HC con la ayuda de un descriptor que se adquirió a partir de los registros ECG preprocesados.

Cabe destacar que se realizaron 2 tipos de modelado y extracción de caracteristicas:
- Modelado simple
- Modelado complejo

### Uso de un modelado simple
El modelado simple refiere al proceso de extracción de caracteristicas tales como media, desviación estándar, valores minimos y máximos de los 12 canales de cada registro.

Luego de obtener tales caracteristicas se crearon las etiquetas identificadoras de cada una, utilizando el valor 0 para los registros HC y 1 para los registros STEMI. Este proceso se llevó a cabo mediante la identificación de la longitud de caracteristicas STEMI y HC, y la creación de un arreglo de etiquetas con los valores 0 y 1.

La cantidad de caracteristicas se resumen en la siguiente lista:

- Caracteristicas STEMI: 4152
- Caracteristicas HC: 4152

Como se puede observar, la cantidad de caracteristicas es igual para ambas clases, lo que indica que no existe un desbalance en la cantidad de caracteristicas extraidas de los registros STEMI y HC.

### Uso de un modelado complejo
Para el modelado complejo se identificaron los picos R de la señal I  de cada registro ECG, los cuales son los puntos más altos de la onda QRS. Estos picos se identificaron mediante el uso de la librería `biosppy`.

Una vez obtenido estos indices, se obtuvieron los puntos Q,S y T. Los primeros puntos se obtenieron mediante la referencia del valor mínimo antes y después del punto R respectivamente. El punto T se obtuvo mediante la referencia del valor máximo después del punto S.

A continuación se muestran dos imagenes de la obtención de los puntos QRST:

![Puntos QRST](/fig/ejemplo_identificacion_QRST.png)
![Puntos QRST](/fig/ejemplo_identificacion_QRST_Secuencial.png)


Una vez obtenido estos puntos se realizan 2 tipos de selección de segmentos:

- Complejo QRST: Se selecciona el segmento que va desde el punto Q hasta el punto T.
- Segmento ST: Se selecciona el segmento que va desde el punto S hasta el punto T.

Para los dos tipos de segmentos se obtienen las siguientes caracteristicas:

- Media
- Desviación estándar
- Valor mínimo
- Valor máximo
- Longitud del segmento

Luego de realizar la extracción de características, se obtuvieron los siguientes resultados:

| Tipo de registro | N° Características QRST | N° Características ST |
|------------------|-------------------------|-----------------------|
| STEMI            | 557134                  | 557134                |
| HC               | 44236                  | 44236               |

Se puede observar un gran desvalance entre las cantidades de caracteristicas extraidas de los registros STEMI y HC. Este desbalance fue solucionado mediante la eliminación de caracteristicas de los registros STEMI hasta igualar la cantidad de caracteristicas de los registros HC.

Luego se concatenaron las caracteristicas QRST de los registros STEMI y HC, y se crearon las etiquetas identificadoras de cada una, utilizando el valor 0 para los registros HC y 1 para los registros STEMI. Este proceso se llevó a cabo mediante la identificación de la longitud de caracteristicas STEMI y HC, y la creación de un arreglo de etiquetas con los valores 0 y 1 tal como en el modelo simple.

## Etapa III Entrenamiento y evaluación de modelos

Para el entrenamiento y evaluación de los modelos se utilizó la librería `sklearn` y se dividió el conjunto de datos en un conjunto de entrenamiento y un conjunto de prueba. El conjunto de entrenamiento se utilizó para entrenar el modelo, mientras que el conjunto de prueba se utilizó para evaluar el rendimiento del modelo. 

La distribución en el modelado simple y complejo fue de 80% para entrenamiento y 20% para prueba.

En el caso del modelado complejo, se entrenó un modelo Random Forest para los segmentos QRST y otro para el segmento ST.

### Resultados Modelado Simple

Los resultados obtenidos para el modelado simple fueron los siguientes:

Precisión: 0.8898254063816978
Reporte de clasificación:
|               |precision|    recall|  f1-score   |support|
|---------------|---------|----------|------------|--------|
 |          0    |   0.93  |    0.84  |    0.88    |   835|
 |     1   |    0.86  |    0.94   |   0.89    |   826|
 |   accuracy    |         |         |     0.89  |    1661|
 |  macro avg    |   0.89  |    0.89   |   0.89   |   1661|
|weighted avg    |   0.89   |   0.89   |   0.89  |    1661|

Siendo 0 la clase HC y 1 la clase STEMI.

Y la matriz de confusión:

![Matriz de confusión](/fig//matriz_confusion_RF_Simple.png)



### Resultados Modelado Complejo

**Respecto al complejo QRST:**

Precisión: 0.9593105397004804
Reporte de clasificación:
|            |   precision|    recall | f1-score  | support|
|------------|------------|-----------|-----------|--------|
|           0 |      0.97  |    0.95  |    0.96  |    8864|
|           1  |     0.95  |    0.97  |    0.96  |    8831|
 |   accuracy    |          |       |      0.96   |  17695|
 |  macro avg    |   0.96    |  0.96   |   0.96   |  17695|
|weighted avg    |   0.96    |  0.96     | 0.96   |  17695|

Siendo 0 la clase HC y 1 la clase STEMI.

Su matriz de confusión:
![Matriz de confusión QRST](/fig/matriz_confusion_RF_QRST.png)

Siendo 0 la clase HC y 1 la clase STEMI.

**Respecto al segmento ST:**

Precisión: 0.9074314778185928
Reporte de clasificación:
|             |  precision  |  recall | f1-score |  support|
|-------------|-------------|---------|----------|---------|
|           0   |    0.91   |  0.90   |   0.91    |  8864|
 |          1    |   0.90   |   0.91    |  0.91   |   8831|
|    accuracy     |       |       |        0.91  |   17695|
|   macro avg   |    0.91    |  0.91    |  0.91  |   17695|
|weighted avg    |   0.91    |  0.91   |   0.91   |  17695|

Siendo 0 la clase HC y 1 la clase STEMI.

Su matriz de confusión:

![Matriz de confusión ST](/fig/matriz_confusion_RF_ST.png)

## Conclusiones

El modelo entrenado con las características completas del complejo QRST muestra una mayor capacidad para predecir la etiqueta de registro, ya sea como sano o de tipo STEMI. Es importante señalar que las diferencias entre el modelo simple, que utiliza características generales de cada canal del registro, y el modelo complejo, basado en la extracción del segmento ST, son mínimas, con una variación aproximada del 1%. Esto sugiere que, si es necesario optimizar recursos, optar por una identificación general de los registros sería preferible. Sin embargo, si se requiere una mayor precisión en el modelo, la opción más adecuada sería el modelado complejo que utiliza los puntos QRST.


![Resultados](/fig/metricas_finales.png)
