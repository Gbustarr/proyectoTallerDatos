# Predicción de Riesgo de Evento Cardíaco con Señales de ECG Usando la Base de Datos PTB Diagnostic ECG y PTB-XL.

El proyecto se centra en la predicción del riesgo de eventos cardíacos utilizando registros de ECG (electrocardiogramas) de la base de datos PTB Diagnostic ECG y PTB-XL de PhysioNet. La finalidad es desarrollar un modelo de aprendizaje profundo que ayude a identificar pacientes en riesgo de sufrir eventos cardíacos, basándose en características extraídas de los datos electrocardiográficos.

## Metodología
En el siguiente gráfico se resume la metodología utilizada en el proyecto:

![Metodologia](/fig/metodologia.png)

1. **Obtención de datos**: Se obtuvieron los registros ECG de las bases de datos PTB Diagnostic ECG y PTB-XL de PhysioNet. De la base de datos PTB Diagnostic ECG se obtuvieron registros de pacientes con infarto al miocardio (STEMI). De la base de datos PTB-XL se obtuvieron registros de pacientes sanos (HC).
2. **Preprocesamiento de registros**: Se realizó un preprocesamiento de los registros ECG, que incluyó la eliminación de canales de posición de electrodos, el downsampling de los registros STEMI de 1000 a 500 Hz y el filtrado de ruido mediante un filtro pasa banda Butterworth de 0.67 a 30 Hz. Los registros HC no fueron preprocesados debido a que ya presentaban un filtrado de ruido y una frecuencia de muestreo de 500 Hz.
3. **Extracción de características**: Se extrajeron características de los registros ECG de ambas bases de datos. El valor maximo, minimo, media y desviación estandar de los 12 canales de cada registro fueron extraidos y proporcionados a los modelos de clasificación.
   1. **Extracción general**: Este proceso se caracteriza por la extracción de características generales de cada canal de cada registro.
   2. **Extracción compleja**: Este proceso se caracteriza por la extracción de características específicas de los segmentos QRST y ST de cada registro.
   3. **Extracción compleja con SMOTE**: Similar al proceso anterior, pero con la inclusión de la técnica SMOTE para balancear la cantidad de caracteristicas extraidas.

## Etapas contenidas en este proyecto

- I. Obtención de datos
- II. Análisis exploratorio de datos.
- III. Extracción de características y selección de modelos.
- IV. Entrenamiento y evaluación de modelos
- V. Comunicación de Resultados

## Etapa I Obtención de datos

Como se mencionó anteriormente, los datos utilizados en este proyecto provienen de las bases de datos PTB Diagnostic ECG y PTB-XL de PhysioNet. Estas bases de datos contienen registros de ECG de pacientes con diferentes condiciones cardíacas, como infarto al miocardio y ritmo cardíaco normal.

La obtención de estos datos se realiza mediante el uso de la libreria boto3 de AWS para la descarga automatica de los registros.

## Etapa II Análisis exploratorio de datos

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

Aqui uno de los atributos relevates es **comments**, el cual contiene valores tales como **'Reason for admission'** el cual registra la razón de admisión del paciente y permite clasificar los registros anómalos y sanos. Por ejemplo, los registros con infarto al miocardio son llamados _'Myocardal infarction'_. También existe la clave **'Acute infarction (localization)'** que indica la localización del infarto.

Debido a que esta base de datos tiene registros tanto de pacientes con Infarto al miocardio y de pacientes sanos, se procedió a realizar una búsqueda de los registros que contienen la clave **'Reason for admission'** con el valor _'Myocardal infarction'_ para identificar los registros de pacientes con infarto al miocardio.

Además se identificó los valores únicos relacionados a la clave **'Acute infarction (localization)'** para identificar las localizaciones del infarto ya que en este apartado existe información faltante y ambigüa como se puede observar en la siguiente imagen:

![Distribución de locaciones](/fig/localizaciones_registros.png)


Para realizar la limpieza de estos elementos, se procedió a realizar un filtro en el cual los registros que contienen los valores **'no'**, **'n/a'** y **'unknown'** fueron eliminados para el análisis de datos dando como resultado la siguiente distribución de locaciones:

![Distribución de locaciones filtradas](/fig/localizaciones_registros_filtrados.png)

### Identificación de la distribución de las clases

Se identifica la distribución de las clases STEMI y HC (Healthy Control) en la base de datos PTB Diagnostic ECG mediante el uso de un atributo "comment" propio de cada registro dando como resultado los las siguientes cantidades:

![Distribución de las clases](/fig/distribucion_preliminar.png)

### Desbalance de clases y solución propuesta

Se identifica que la base de datos PTB Diagnostic ECG contiene un desbalance de clases entre los registros STEMI y HC, lo cual puede afectar el rendimiento de los modelos de clasificación.

Respecto a esto, se propuso la **incorporación de registros de pacientes sanos de la base de datos PTB-XL** para balancear las clases.

Otras soluciones que se tuvieron en cuenta fueron la de uso de técnicas de resampling, generación de datos sintéticos y ajuste de pesos de clase. Sin embargo, estas soluciones fueron desechadas debido a que se priorizó la obtención de registros reales asi evitando un sesgo creado por la generación de datos sintéticos o el ajuste de pesos de clase.


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

Cabe destacar que la base de datos PTB-XL posee mucho mas registros que la base de datos PTB Diagnostic ECG pero solo se limitó a obtener la cantidades necesarias para balancear las clases.

### Preprocesamiento de los registros

Se realizó un preprocesamiento de los registros ECG de ambas bases de datos, el cual consistió en:

- **Guardado de registros STEMI** en sus respectivo directorio para su procesamiento.
- **Eliminación de canales de posicion de electrodos** identificando los nombres Vx, Vy y Vz.
- **Downsampling de canales** de registros STEMI de 1000 a 500 Hz. 
- **Filtrado de ruido** mediante un filtro pasa banda Butterworth de 0.67 a 30 Hz [(Basado en este artículo)](https://www.cinc.org/archives/2020/pdf/CinC2020-403.pdf).

La eliminación de la información de los canales de posicion de los electrodos se realizó debido a que no serán utilizados en el desarrollo de este proyecto.

Los registros añadidos de la base de datos PTB-XL no fueron preprocesados debido a que ya presentaban un filtrado de ruido y una frecuencia de muestreo de 500 Hz.

La reducción de la frecuencia de muestreo de los registros STEMI de 1000 a 500 Hz se realizó debido a que los registros de la base de datos PTB-XL tienen una frecuencia de muestreo de 500 Hz, por lo que se decidió homogeneizar la frecuencia de muestreo de todos los registros a 500 Hz.

Un ejemplo de la señal preprocesada de los registros STEMI se puede apreciar en la siguiente imagen:

![Comparativa de señales](/fig/muestr_señales_procesadas.png)

Se puede observar la disminució de las muestras en cada canal debido al downsampling. Tambien se puede percibir que la eliminación de ruido en la señal como resultado una señal mas estable y alineada.


## Etapa III Extracción de características y selección de modelos

Como se mostró en la metodología, se realizaron tres tipos de extracción de características:

- **Extracción general**: Este proceso se caracteriza por la extracción de características generales de cada canal de cada registro.
- **Extracción específica**: Este proceso se caracteriza por la extracción de características específicas de los segmentos QRST y ST de cada registro.
- **Extracción específica con SMOTE**: Similar al proceso anterior, pero con la inclusión de la técnica SMOTE para balancear la cantidad de caracteristicas extraidas.

### Extracción general de caracteristicas por canal

 Proceso de extracción de caracteristicas tales como media, desviación estándar, valores minimos y máximos de los 12 canales de cada registro.

Luego de obtener tales caracteristicas se crearon las etiquetas identificadoras de cada una, utilizando el valor 0 para los registros HC y 1 para los registros STEMI. Este proceso se llevó a cabo mediante la identificación de la longitud de caracteristicas STEMI y HC, y la creación de un arreglo de etiquetas con los valores 0 y 1.

La cantidad de caracteristicas se resumen en la siguiente lista:

- Caracteristicas STEMI: 4152
- Caracteristicas HC: 4152

Como se puede observar, la cantidad de caracteristicas es igual para ambas clases, lo que indica que no existe un desbalance en la cantidad de caracteristicas extraidas de los registros STEMI y HC.

### Extracción específica de características por segmento QRST y ST

Proceso de extracción de caracteristicas en el cual se identificaron los picos R de la señal I  de cada registro ECG, los cuales son los puntos más altos de la onda QRS. Estos picos se identificaron mediante el uso de la librería `biosppy`.

Obteniendo estos indices, se obtuvieron los puntos Q,S y T. Los primeros puntos se obtuvieron mediante la referencia del valor mínimo antes y después del punto R respectivamente. El punto T se obtuvo mediante la referencia del valor máximo después del punto S.

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
| STEMI            | 595172                  | 595172               |
| HC               | 45632                 | 45632                |
|TOTAL             | 640804                 | 640804               |

Se puede observar un gran desvalance entre las cantidades de caracteristicas extraidas de los registros STEMI y HC. Este desbalance se mantuvo en esta etapa.

Luego se concatenaron las caracteristicas QRST de los registros STEMI y HC, y se crearon las etiquetas identificadoras de cada una, utilizando el valor 0 para los registros HC y 1 para los registros STEMI. Este proceso se llevó a cabo mediante la identificación de la longitud de caracteristicas STEMI y HC, y la creación de un arreglo de etiquetas con los valores 0 y 1 tal como en el modelo simple.


### Extracción específica de características por segmento QRST y ST con SMOTE

El desbalanceo de caracteristicas se puede observar en el siguiente gráfico:

![Desbalance de caracteristicas](/fig/desbalance_caracteristicas.png)

Siendo la barra roja para las caracteristicas extraidas de los registros STEMI y la barra verde para las caracteristicas extraidas de los registros HC.

Lo que se implementó en esta etapa fuela la misma extracción de caracteristicas que en el proceso anterior, pero se balanceó las caracteristicas extraidas de los registros HC de ambos segmentos con la técnica SMOTE al 80% perteneciente al segmento de entrenamiento.


| Tipo de registro | 80% Características QRST | 80% Características ST |
|------------------|-------------------------|-----------------------|
| STEMI            | 471812                | 471812             |
| HC               | 471812               | 471812              |
|TOTAL             | 943624                 | 943624                |



## Etapa IV Entrenamiento y evaluación de modelos

Para el entrenamiento y evaluación de los modelos se utilizó la librería `sklearn` y se dividió el conjunto de datos en un conjunto de entrenamiento y un conjunto de prueba. El conjunto de entrenamiento se utilizó para entrenar el modelo, mientras que el conjunto de prueba se utilizó para evaluar el rendimiento del modelo. 

La distribución en todos las etapas fue de 80% para entrenamiento y 20% para prueba.

### Resultados Extracción general de caracteristicas por canal

**RANDOM FOREST**

Precisión: 0.8908872901678657
Reporte de clasificación:
|               |precision|    recall|  f1-score   |support|
|---------------|---------|----------|------------|--------|
 |          0    |   0.96  |    0.85  |    0.90    |   984|
 |     1   |    0.81  |    0.95   |   0.88    |   684|
 |   accuracy    |         |         |     0.89  |    1668|
 |  macro avg    |   0.89  |    0.90   |   0.89   |   1668|
|weighted avg    |   0.90   |   0.89   |   0.89  |    1668|

Siendo 0 la clase HC y 1 la clase STEMI.

Y la matriz de confusión:

![Matriz de confusión](/fig//matriz_confusion_RF_Simple.png)

**SVM**

Precisión: 0.6504796163069544

Reporte de clasificación:
|              | precision |   recall | f1-score |  support|
|--------------|-----------|----------|----------|---------|
|           0  |     0.74  |    0.62  |    0.68  |     984|
|           1  |     0.56  |    0.69  |    0.62  |     684|
|    accuracy  |           |          |    0.65  |    1668|
|   macro avg  |     0.65  |    0.66  |    0.65  |    1668|
|weighted avg  |     0.67  |    0.65  |    0.65  |    1668|

![Matriz de confusión](/fig/matriz_confusion_SVM_Simple.png)

**Redes neuronales**

Precisión: 0.9075841307640076

**ADABOOST**

Precisión: 0.86

**XGBOOST**

Precisión: 0.90


### Resultados Extracción específica de características por segmento

#### Respecto al complejo QRST

**RANDOM FOREST**

Precisión: 0.8858204023970652
Reporte de clasificación:
|             |  precision |  recall | f1-score |  support|
|-------------|------------|---------|----------|---------|
|0            |     0.29   |   0.58  |   0.39   |    8301|
|1            |     0.97   |   0.91  |   0.94   |  123360|
|accuracy     |            |         |   0.89   |  131661|
|macro avg    |     0.63   |   0.74  |   0.66   |  131661|
|weighted avg |     0.93   |   0.89  |   0.90   |  131661|


Siendo 0 la clase HC y 1 la clase STEMI.

Su matriz de confusión:

![Matriz de confusión QRST](/fig/matriz_confusion_RF_QRST.png)

Siendo 0 la clase HC y 1 la clase STEMI.

**SVM**

Sin resultados despues de 5 horas de entrenamiento.

**Redes neuronales**

Precisión: 0.9557033777236938

![Matriz de confusión QRST](/fig/matriz_confusion_RN_QRST.png)

**ADABOOST**

Precisión: 0.94

**XGBOOST**

Precisión: 0.95

![Matriz de confusión QRST](/fig/matriz_confusion_XG_QRST.png)

#### Respecto al complejo ST

**RANDOM FOREST**

Precisión: 0.9028630252556135
Reporte de clasificación:
|             |  precision |  recall | f1-score |  support|
|-------------|------------|---------|----------|---------|
|0            |     0.23   |   0.44  |   0.31   |    6262|
|1            |     0.97   |   0.93  |   0.95   |  122937|
|accuracy     |            |         |   0.90   |  129199|
|macro avg    |     0.60   |   0.68  |   0.63   |  129199|
|weighted avg |     0.93   |   0.90  |   0.92   |  129199|

**SVM**

Omitido por tiempo de entrenamiento.

**Redes neuronales**

Precisión: 0.9368395209312439

![Matriz de confusión ST](/fig/matriz_confusion_RN_ST.png)

**ADABOOST**

Precisión: 0.94

**XGBOOST**

Precisión: 0.95

![Matriz de confusión ST](/fig/matriz_confusion_XG_ST.png)


### Resultados Extracción específica de características por segmento con SMOTE

#### Respecto al complejo QRST

**RANDOM FOREST**

Precisión: 0.6886093831886435
Reporte de clasificación:
|             |  precision |  recall | f1-score |  support|
|-------------|------------|---------|----------|---------|
|0            |     0.15   |   0.80  |   0.25   |    8301|
|1            |     0.98   |   0.68  |   0.80   |  123360|
|accuracy     |            |         |   0.69   |  131661|
|macro avg    |     0.56   |   0.74  |   0.52   |  131661|
|weighted avg |     0.93   |   0.69  |   0.77   |  131661|


![Matriz de confusión QRST](/fig/matriz_confusion_RF_QRST_SMOTE.png)

**SVM**

Omitido por tiempo de entrenamiento.

**Redes neuronales**

Precision : 0.8295830488204956

![Matriz de confusión QRST](/fig/matriz_confusion_RN_QRST_SMOTE.png)

**ADABOOST**

Precision: 0.65

**XGBOOST**

Precision: 0.67

#### Respecto al complejo ST

**RANDOM FOREST**

Precisión: 0.7317161897537907
Reporte de clasificación:
|             |  precision |  recall | f1-score |  support|
|-------------|------------|---------|----------|---------|
|0            |     0.12   |   0.74  |   0.21   |    6262|
|1            |     0.98   |   0.73  |   0.84   |  122937|
|accuracy     |            |         |   0.73   |  129199|
|macro avg    |     0.55   |   0.73  |   0.52   |  129199|
|weighted avg |     0.94   |   0.73  |   0.81   |  129199|

![Matriz de confusión ST](/fig/matriz_confusion_RF_ST_SMOTE.png)

**SVM**

Omitido por tiempo de entrenamiento.

**Redes neuronales**

Precision: 0.784372866153717

![Matriz de confusión ST](/fig/matriz_confusion_RN_ST_SMOTE.png)

**ADABOOST**

Precision: 0.66

**XGBOOST**

Precision: 0.69


## Conclusiones

