# Data-Science-Python-R-IAML3

## Integracion de Python y R

En esta tarea se presenta un análisis de datos que integra las fortalezas de **Python** y **R**, dos lenguajes ampliamente utilizados en ciencia de datos. Utilizamos **Python** para la **importación**, **limpieza** y **transformación** del dataset, aprovechando sus potentes librerías como `pandas` y `numpy`. 

Por otro lado, aprovechamos la capacidad gráfica y estadística avanzada de **R**, mediante la librería `ggplot2`, para realizar un análisis visual profundo y estadístico del conjunto de datos.

La integración se realiza mediante la librería **`rpy2`**, que permite ejecutar código R directamente desde Python y compartir objetos entre ambos entornos de manera eficiente y fluida. Esto nos brinda un flujo de trabajo que combina lo mejor de ambos mundos: la flexibilidad de Python para la manipulación de datos y la sofisticación gráfica y analítica de R.

El objetivo final es extraer información relevante sobre el mercado laboral analizando variables clave como el salario en USD, el nivel de experiencia, el tamaño de la empresa y el tipo de empleo. De esta forma, buscamos identificar patrones y tendencias que faciliten la comprensión del comportamiento salarial y las características de los puestos de trabajo en el dataset.

-   **Dataset:**

[Dataset_salary_2024](https://www.kaggle.com/datasets/zeesolver/data-eng-salary-2024)

### Exploración de salarios en roles tecnológicos a nivel global.

#### ¿Quién gana más en tecnología? Un análisis exploratorio de salarios por experiencia, tipo de empresa y rol.

El presente análisis se ha realizado con un conjunto de datos registrados desde el año 2020 hasta la fecha, teniendo una significativamente mayor cantidad de registros en los años 2023 y 2024.

#### ¿Qué estaremos analizando?

-   Se abordara principalmente un análisis de salarios en promedio y distribución dependiendo de variables cualitativas o categóricas, como lo son el nivel de experiencia de los empleados y el tamaño de la empresa para la cual trabajan.
    
-   Se ha determinado que variables como la modalidad de trabajo no es significativa para los rangos salariales.
    
-   De la misma forma, variables como la residencia del empleado, la ubicación de la empresa no serán tomadas en cuenta de forma profunda debido a su muy grande concentración en Estados Unidos y Canadá (en menor medida).
    

#### ¿Cuál es el trabajo mejor pagado?

-   Analizaremos la naturaleza de los datos y tomaremos en cuenta la población para determinar los trabajos mejor pagados, si bien el pico puede estar en un puesto especifico hay que tomar en cuenta tambien la cantidad de registros por cada puesto dentro del estudio.

#### Acompáñanos en este análisis de salarios en roles tecnológicos a nivel global
