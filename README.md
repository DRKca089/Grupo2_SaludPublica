
# ANÁLISIS DE LA VARIACIÓN EN LA CALIDAD DEL AIRE EN LAS REGIONES DEFINIDAS POR EL CENSO EN ESTADOS UNIDOS

## Grupo2_Salud

### Integrantes:

- JUSTAILS: Justin Guevara  
- DRKca089: Derek Cahuate  

### Descripción del proyecto

El proyecto consiste en el análisis de la variación histórica del Índice de Calidad del Aire (AQI) en Estados Unidos durante el periodo 1980–2021, utilizando datos recopilados por la Agencia de Protección Ambiental de los Estados Unidos (EPA). El análisis se realiza agrupando los registros según las cuatro regiones definidas por el Censo de Estados Unidos: Northeast, Midwest, South y West.

Los datos utilizados se limpiaron, normalizarón y agruparon en regiones. Luego se aplicaron los métodos númericos para modelar el comportamiento del AQI. Se utiliza la interpolación del spline cúbico para representar la tendencia histórica del Median AQI, y se usa mínimos cuadrados para realizar extrapolaciones.

Finalmente, los resultados obtenidos son representados graficamente el objetivo de analizar las tendencias del AQI, comparar diferencias entre regiones e identificar seguira su tendencia hacia el futuro.

### Variables utilizadas

Para el análisis se utilizaron las siguientes variables, las cuales se encuentran en el dataset normalizado generado durante la etapa de limpieza:

- **Year:** Año en el que fueron registrados los datos.
- **Region:** Región geográfica asignada según la clasificación del Censo de Estados Unidos.
- **Median AQI:** Valor mediano anual del Índice de Calidad del Aire utilizado como variable principal para analizar la tendencia histórica.
- **AQI level days:** Proporción o cantidad normalizada de días clasificados según las categorías del AQI (Good, Moderate, Unhealthy for Sensitive Groups, Unhealthy, Very Unhealthy y Hazardous).
- **Pollutant days:** Proporción o cantidad normalizada de días donde cada contaminante fue el principal responsable del AQI registrado (CO, NO2, Ozone, SO2, PM2.5 y PM10).

### Descripción del Github

El repositorio contiene las siguientes carpetas y archivos necesarios para la ejecución y documentación del proyecto. Su estructura se divide en:

- **Avances:** Contiene el informe del proyecto que se actualiza conforme se va avanzando el proyecto.
- **Codigo:** Incluye los códigos desarrollados en Python para la limpieza del dataset, aplicación de spline cúbico y aplicación de mínimos cuadrados.
- **Dataset:** Contiene el dataset original obtenido desde Kaggle y el dataset limpiado utilizado para el análisis final.
- **Graficas:** Almacena las graficas generadas durante la aplicación de splines cúbicos y mínimos cuadrados.
- **Readme:** Contiene la información general del proyecto. 

### Fases del desarrollo del proyecto

**Fase 1: Preparación y análisis de datos**

Se realizó la limpieza y transformación del dataset original, seleccionando los registros del periodo 1980–2020 debido a la menor disponibilidad de datos en 2021. Se asignó cada registro a una de las cuatro regiones definidas por el Censo de Estados Unidos y se eliminaron valores nulos y registros no válidos.

Además, se normalizaron las variables relacionadas con categorías del AQI y contaminantes mediante proporciones, y los datos fueron agrupados por región y año.

**Fase 2: Aplicación de Splines Cúbicos**

Se implementó el método de interpolación mediante splines cúbicos naturales. Los coeficientes obtenidos permitieron construir funciones polinómicas por tramos para representar la evolución histórica del AQI en cada región, utilizando condiciones de frontera natural.

**Fase 3: Aplicación de Mínimos Cuadrados**

Se aplicó el método de mínimos cuadrados polinómicos para modelar la tendencia del AQI en cada región, utilizando Year como variable independiente y Median AQI como variable dependiente.

Mediante la matriz de Vandermonde se obtuvieron los coeficientes del polinomio de grado 2, utilizado posteriormente para realizar extrapolaciones basadas en la tendencia histórica del AQI.

**Fase 4: Validación del modelo**

En esta fase se compararon los resultados obtenidos mediante los métodos de splines cúbicos y mínimos cuadrados, analizando la tendencia histórica dada por el spline cúbico y su relación con las variables de la distribución de días según las categorías del AQI y los contaminantes principales.

Para el método de mínimos cuadrados se utilizó un ajuste polinómico de grado 2, seleccionado debido a que permite representar la tendencia general del AQI manteniendo una adecuada estabilidad del modelo y evitando oscilaciones excesivas durante la extrapolación.

Para evaluar el comportamiento del modelo, se utilizaron los datos comprendidos entre 1980 y 2015 para realizar el ajuste, mientras que los registros correspondientes al periodo de 2016 a 2020 fueron utilizados como datos de validación. De esta manera, se verificó si la tendencia estimada por los modelos mantiene un comportamiento similar al observado históricamente.

**Fase 5: Interpretación de resultados**

Finalmente, se realizará un análisis comparativo entre las cuatro regiones utilizando las gráficas generadas durante las etapas anteriores.

A partir de estos resultados se buscará identificar:

- Tendencias crecientes o decrecientes del AQI.
- Contaminantes responsables del nivel del AQI.
- Diferencias entre regiones.
- Tendencia futura del comportamiento del AQI.

### Ejecución del proyecto

Para ejecutar el proyecto se deben seguir los siguientes pasos:

1. **Clonar el repositorio**

   Descargar el repositorio de GitHub y acceder a la carpeta principal del proyecto.

2. **Ejecutar el notebook principal**

   Abrir y ejecutar las celdas del archivo `Proyecto.ipynb`, donde se encuentran los procesos de:

   - Limpieza y normalización de datos.
   - Aplicación de splines cúbicos y mínimos cuadrados.
   - Generación de gráficas.

3. **Verificación de resultados**

   Al finalizar la ejecución se debe comprobar la generación de los archivos resultantes:

   - El dataset procesado se almacena en la carpeta `dataset`.
   - Las gráficas generadas por los métodos numéricos se almacenan en la carpeta `graficas`.