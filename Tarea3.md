# Data Science Python-R
#### Importar librerias de Python

    import pandas as pd
    import numpy as np
    from datetime import datetime, timedelta
    import random
    import time
    from scipy.stats import zscore
    from PIL import Image
    from IPython.display import display

#### Configurar RPy2
Importar modulos de RPy2

    import os
    import sys # Importar sys para salir si hay problemas con R_HOME

TROUBLESHOOTING / CONFIGURACIÓN

    # --- INICIA EL TROUBLESHOOTING / CONFIGURACIÓN ---
    # 1. Define la ruta RAÍZ de tu instalación de R.
    r_home_path = 'C:/PROGRA~1/R/R-45~1.0' 
    
    # 2. Establece la variable de entorno R_HOME.
    os.environ['R_HOME'] = r_home_path
    os.environ['LANG'] = 'en_US.UTF-8'
    
    # 3. Añade la ruta del ejecutable de R (bin/x64 para 64-bit) a la variable PATH.
    r_bin_path = os.path.join(r_home_path, 'bin', 'x64')
    
    # Añade esta ruta al PATH existente si aún no está.
    if r_bin_path not in os.environ['PATH']:
        os.environ['PATH'] = r_bin_path + os.pathsep + os.environ['PATH']
    
    # --- AÑADIMOS ESTO PARA DIAGNÓSTICO DE RTOOLS ---
    rtools_path = 'C:\\rtools42\\usr\\bin'
    
    # Añade la ruta de RTools al PATH si no está ya.
    if rtools_path not in os.environ['PATH']:
        os.environ['PATH'] = rtools_path + os.pathsep + os.environ['PATH']
    
    print("Verificando PATH después de añadir R y RTools:")
    print(os.environ['PATH'])
    # --- TERMINA EL TROUBLESHOOTING / CONFIGURACIÓN ---

Configurar RPy2

    from rpy2.robjects import r, globalenv
    from rpy2.robjects.packages import importr
    from rpy2.robjects import pandas2ri
    from rpy2.robjects.conversion import localconverter
    import rpy2.robjects as ro
    
    # Importar paquetes R necesarios
    print("\nImportando paquetes R...")
    try:
        with localconverter(ro.default_converter + pandas2ri.converter + numpy2ri.converter) as cv:
            base = importr('base')
            stats = importr('stats')
            utils = importr('utils')
            
            r_packages = ['ggplot2', 'dplyr', 'lubridate']
            for pkg in r_packages:
                try:
                    importr(pkg)
                    print(f"  {pkg} cargado correctamente")
                except Exception as pkg_e:
                    print(f"  Error al cargar {pkg}: {pkg_e}")
                    print(f"  Intentando instalar {pkg}...")
                    utils.install_packages(pkg)
                    print(f"  {pkg} instalado y cargado")
                    
            ggplot2 = importr('ggplot2')
            dplyr = importr('dplyr')
            lubridate = importr('lubridate')
            
    except Exception as e:
        print(f"\n¡ERROR CRÍTICO! RPy2 o R no pudieron inicializarse correctamente: {e}")
        print("Por favor, revisa los pasos de instalación de R y RTools, y la configuración de PATH.")
        sys.exit(1) # Salir del script si hay un error fatal con R.

#### Leer Dataset

    df = pd.read_csv("Dataset_salary_2024.csv")

#### Limpieza y transformacion de DataFrame con Python

    # --- Eliminación de Outliers ---
    # Calcular Z-scores solo en columnas numéricas
    z_scores = np.abs(zscore(df.select_dtypes(include=np.number)))
    
    # Crear máscara para valores no-atípicos
    mask = (z_scores < 3).all(axis=1)
    
    # Aplicar máscara al DataFrame
    df_jobs_salary = df[mask].copy()
    print(f"Original: {df.shape[0]} filas")
    print(f"Sin outliers: {df_jobs_salary.shape[0]} filas")
    
    # --- Transformación de columnas ---
    
    # Eliminar columnas no necesarias
    df_jobs_salary = df_jobs_salary.drop(columns=['salary', 'salary_currency'])
    
    # Crear columna 'remote_type' a partir de 'remote_ratio'
    def remote_category(ratio):
        if ratio == 0:
            return 'IN-PERSON'
        elif ratio == 50:
            return 'HYBRID'
        elif ratio == 100:
            return 'REMOTE'
        else:
            return 'UNKNOWN'
    
    df_jobs_salary['remote_type'] = df_jobs_salary['remote_ratio'].apply(remote_category)
    
    # Normalizar valores a mayúsculas
    cols_to_upper = ['experience_level', 'employment_type', 'employee_residence', 'company_location', 'company_size']
    for col in cols_to_upper:
        df_jobs_salary[col] = df_jobs_salary[col].str.upper()
    
    # Convertir columnas a tipo categoría
    for col in cols_to_upper:
        df_jobs_salary[col] = df_jobs_salary[col].astype('category')
    
    # Ver una muestra del DataFrame limpio
    df_jobs_salary.head()

#### Convertir DataFrame de Pandas a R

    # Usamos un conversor local para pasar el DataFrame de Pandas a R
    with localconverter(ro.default_converter + pandas2ri.converter):
        r_df_jobs_salary = ro.conversion.py2rpy(df_jobs_salary)
    
    # Asignamos el DataFrame a una variable en el entorno R
    ro.globalenv['df_jobs_salary'] = r_df_jobs_salary
    
    # Confirmamos que se asignó correctamente
    print(ro.r('head(df_jobs_salary)'))

#### Analisis de DataFrame con R desde Python

Analisis estadistico y general

    ro.r('''
    library(dplyr)
    
    message("--- Estructura del dataset ---")
    str(df_jobs_salary)
    
    message("--- Resumen detallado de dataset ---")
    print(skim(df_jobs_salary))
    
    message("--- Resumen estadistico ---")
    print(summary(df_jobs_salary))
    
    message("--- Conteo por nivel de experiencia ---")
    print(table(df_jobs_salary$experience_level))
    
    message("--- Salario promedio por nivel de experiencia ---")
    df_jobs_salary %>% 
      group_by(experience_level) %>% 
      summarise(avg_salary = mean(salary_in_usd, na.rm = TRUE)) %>% 
      print()
    
    message("--- Salario promedio por tamanio de empresa ---")
    df_jobs_salary %>% 
      group_by(company_size) %>% 
      summarise(avg_salary = mean(salary_in_usd, na.rm = TRUE)) %>% 
      print()
    
    message("--- Salario promedio por empleo completo (FT) vs parcial (PT) ---")
    df_jobs_salary %>% 
      group_by(employment_type) %>% 
      summarise(avg_salary = mean(salary_in_usd, na.rm = TRUE)) %>% 
      print()
    
    message("--- Correlacion entre variables numericas ---")
    numeric_vars <- df_jobs_salary %>% select_if(is.numeric)
    cor_matrix <- cor(numeric_vars, use = "complete.obs")
    print(round(cor_matrix, 3))
    ''')

Analisis de graficos

    def visualizacion_r(df):
        """
        Crea y guarda gráficos avanzados usando ggplot2 en R, 
        basado en el dataframe df_jobs_salary con datos de salarios.
        """
        print("\nVISUALIZACIÓN AVANZADA CON GGPLOT2")
        print("=" * 50)
        
        try:
            # Convierte pandas DataFrame a DataFrame R y asigna a variable 'data'
            with localconverter(pandas2ri.converter):
                df_r = pandas2ri.py2rpy(df)
                ro.globalenv['data'] = df_r
            
            ro.r('''
                library(ggplot2)
                library(dplyr)
                library(scales)
                
                # Convertir variables a factores con niveles definidos
                data$experience_level <- factor(data$experience_level, levels = c("EN", "MI", "SE", "EX"))
                data$company_size <- factor(data$company_size, levels = c("S", "M", "L"))
                
                # 1. Histograma distribución salario
                p1 <- ggplot(data, aes(x = salary_in_usd)) +
                  geom_histogram(bins = 30, fill = "skyblue", color = "black") +
                  geom_density(aes(y = ..count.. * 10), color = "darkblue", size = 1) +
                  labs(title = "Distribución de salario en USD",
                       x = "Salario (USD)", y = "Frecuencia") +
                  scale_x_continuous(labels = comma) +
                  theme_minimal()
                ggsave("histograma_salario.png", p1, width = 10, height = 6, dpi = 300)
                
                # 2. Cantidad de empleados por nivel de experiencia
                p2 <- ggplot(data, aes(x = experience_level)) +
                  geom_bar(fill = "skyblue") +
                  labs(title = "Cantidad de empleados por nivel de experiencia",
                       x = "Nivel de experiencia", y = "Cantidad") +
                  theme_minimal()
                ggsave("empleados_por_experiencia.png", p2, width = 8, height = 6, dpi = 300)
                
                # 3. Boxplot salario por nivel de experiencia
                p3 <- ggplot(data, aes(x = experience_level, y = salary_in_usd)) +
                  geom_boxplot(fill = "lightgreen") +
                  labs(title = "Salario según nivel de experiencia",
                       x = "Nivel de experiencia", y = "Salario (USD)") +
                  scale_y_continuous(labels = comma) +
                  theme_minimal()
                ggsave("boxplot_salario_experiencia.png", p3, width = 8, height = 6, dpi = 300)
                
                # 4. Cantidad de empleados por tamaño de empresa
                p4 <- ggplot(data, aes(x = company_size)) +
                  geom_bar(fill = "orange") +
                  labs(title = "Cantidad de empleados por tamaño de empresa",
                       x = "Tamaño de la empresa", y = "Cantidad") +
                  theme_minimal()
                ggsave("empleados_por_empresa.png", p4, width = 8, height = 6, dpi = 300)
                
                # 5. Boxplot salario por tamaño de empresa
                p5 <- ggplot(data, aes(x = company_size, y = salary_in_usd)) +
                  geom_boxplot(fill = "lightcoral") +
                  labs(title = "Salario según tamaño de la empresa",
                       x = "Tamaño de la empresa", y = "Salario (USD)") +
                  scale_y_continuous(labels = comma) +
                  theme_minimal()
                ggsave("boxplot_salario_empresa.png", p5, width = 8, height = 6, dpi = 300)
                
                # 6. Boxplot salario por experiencia y tamaño empresa (facet)
                p6 <- ggplot(data, aes(x = experience_level, y = salary_in_usd, fill = experience_level)) +
                  geom_boxplot() +
                  facet_wrap(~ company_size) +
                  labs(title = "Salario por nivel de experiencia y tamaño de empresa",
                       x = "Nivel de experiencia", y = "Salario (USD)") +
                  scale_y_continuous(labels = comma) +
                  scale_fill_brewer(palette = "Pastel1") +
                  theme_minimal()
                ggsave("boxplot_salario_experiencia_empresa.png", p6, width = 12, height = 6, dpi = 300)
                
                # 7. Salario promedio por experiencia y tamaño empresa (barras)
                resumen <- data %>%
                  group_by(experience_level, company_size) %>%
                  summarise(mean_salary = mean(salary_in_usd), .groups = "drop") %>%
                  mutate(experience_level = factor(experience_level, levels = c("EN", "MI", "SE", "EX")),
                         company_size = factor(company_size, levels = c("S", "M", "L")))
                
                p7 <- ggplot(resumen, aes(x = company_size, y = mean_salary, fill = experience_level)) +
                  geom_col(position = "dodge") +
                  labs(title = "Salario promedio por experiencia y tamaño de empresa",
                       x = "Tamaño de empresa", y = "Salario promedio (USD)", fill = "Nivel de experiencia") +
                  scale_y_continuous(labels = comma) +
                  scale_fill_brewer(palette = "Set2") +
                  theme_minimal()
                ggsave("promedio_salario_experiencia_empresa.png", p7, width = 10, height = 6, dpi = 300)
                
                # 8. Top 10 trabajos mejor pagados (barra horizontal)
                top_jobs <- data %>%
                  group_by(job_title) %>%
                  summarise(avg_salary = mean(salary_in_usd, na.rm = TRUE)) %>%
                  arrange(desc(avg_salary)) %>%
                  slice(1:10)
                
                p8 <- ggplot(top_jobs, aes(x = reorder(job_title, avg_salary), y = avg_salary)) +
                  geom_bar(stat = "identity", fill = "purple") +
                  labs(title = "Top 10 trabajos mejor pagados",
                       x = "Puesto", y = "Salario promedio (USD)") +
                  scale_y_continuous(labels = comma) +
                  coord_flip() +
                  theme_minimal()
                ggsave("top10_trabajos_mejor_pagados.png", p8, width = 10, height = 6, dpi = 300)
                
                # 9. Distribución nivel experiencia en top 10 trabajos mejor pagados
                data_top_jobs <- data %>% filter(job_title %in% top_jobs$job_title)
                
                p9 <- ggplot(data_top_jobs, aes(x = job_title, fill = experience_level)) +
                  geom_bar(position = "dodge") +
                  labs(title = "Distribución del nivel de experiencia en los 10 trabajos mejor pagados",
                       x = "Título del trabajo", y = "Cantidad", fill = "Nivel de experiencia") +
                  theme_minimal() +
                  theme(axis.text.x = element_text(angle = 45, hjust = 1))
                ggsave("distribucion_experiencia_top10.png", p9, width = 12, height = 6, dpi = 300)
                
                # 10. Distribución tamaño empresa en top 10 trabajos mejor pagados
                data_top_jobs$job_title <- factor(data_top_jobs$job_title, levels = top_jobs$job_title)
                data_top_jobs$company_size <- factor(data_top_jobs$company_size, levels = c("S", "M", "L"))
                
                p10 <- ggplot(data_top_jobs, aes(x = job_title, fill = company_size)) +
                  geom_bar(position = "dodge") +
                  labs(title = "Distribución de tamaños de empresa en los 10 trabajos mejor pagados",
                       x = "Título del trabajo", y = "Cantidad de empleados", fill = "Tamaño de empresa") +
                  theme_minimal() +
                  theme(axis.text.x = element_text(angle = 45, hjust = 1))
                ggsave("distribucion_empresa_top10.png", p10, width = 12, height = 6, dpi = 300)
            ''')
            
            print("\nTodos los gráficos han sido generados y guardados como archivos PNG.")
            
        except Exception as e:
            print(f"Error en la visualización: {e}")
    
    visualizacion_r(df_jobs_salary)

## Conclusiones

-   Se logró una **integración efectiva entre Python y R** mediante el uso de la librería `rpy2`, lo que permitió aprovechar las fortalezas de ambos lenguajes en un mismo entorno de análisis.
    
-   Python fue utilizado principalmente para la **lectura, limpieza y transformación del dataset**, aplicando técnicas como la eliminación de outliers con Z-score y la estandarización de variables categóricas. Esto facilitó un preprocesamiento estructurado y reproducible.
    
-   R, a través del paquete `ggplot2`, permitió generar **visualizaciones estadísticas de alta calidad**, representando gráficamente la distribución salarial, la influencia del nivel de experiencia, el tamaño de la empresa y otros factores clave.
    
-   Se identificaron patrones importantes en los datos, como:
    
    -   A mayor nivel de experiencia, mayor es el salario promedio.
        
    -   Las empresas grandes tienden a pagar salarios más altos, aunque la que en promedio presenta salarios mas altos son las empresas medianas.
        
    -   Los trabajos mejor pagados están mayormente asociados a niveles de experiencia altos y empresas de mayor tamaño.
        
-   La integración permitió también la **exportación automatizada de los gráficos generados**, lo cual es útil para reportes, presentaciones o análisis posteriores.
    
-   En conjunto, esta práctica demuestra cómo un flujo de trabajo mixto Python-R puede mejorar la eficiencia y profundidad del análisis de datos, siendo altamente recomendable en proyectos donde se requiere tanto manipulación avanzada de datos como visualizaciones profesionales.