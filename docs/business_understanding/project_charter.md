# Project Charter - Entendimiento del Negocio

## **Nombre del Proyecto**

Predicción de la duración de viajes en taxis verdes de la Ciudad de Nueva York.

## **Objetivo del Proyecto**

El objetivo principal del presente proyecto es desarrollar un sistema de aprendizaje automático capaz de predecir con alta precisión la duración en minutos (Tiempo Estimado de Llegada - ETA) de un viaje en taxi verde (*Green Taxi / Boro Taxi*) en la ciudad de Nueva York, utilizando información disponible al momento de solicitar o iniciar el servicio (zona de origen, zona de destino, distancia estimada (en producción, la calculada por el servicio de rutas al solicitar el viaje), franja horaria y día de la semana). 

Esta predicción permite optimizar tres procesos críticos de negocio: ofrecer estimaciones confiables de llegada al pasajero, calcular tarifas dinámicas o anticipadas de manera transparente y mejorar la asignación eficiente de conductores en los distintos distritos (*boroughs*) de la ciudad. El problema se formula como una tarea de **regresión supervisada**, cuya métrica principal de evaluación será el **RMSE (Root Mean Squared Error)** y como métrica secundaria el **MAE (Mean Absolute Error)** expresados en minutos.

## **Alcance del Proyecto**

### Incluye:

- Registros históricos mensuales de viajes de taxis verdes (*Green Taxi Trip Records*) publicados por la *NYC Taxi and Limousine Commission (TLC)* en formato Parquet (aproximadamente entre 50.000 y 70.000 viajes por mes para el año 2024), complementados con el catálogo oficial de zonas geográficas (`taxi_zone_lookup.csv`).

- Filtrado de registros válidos con duraciones de viaje entre 1 y 60 minutos, eliminación de distancias nulas o atípicas y cruce con información espacial de zonas y *boroughs*.

- Entrenamiento y comparación iterativa de modelos de regresión (iniciando con Regresión Lineal como línea base y avanzando hacia modelos basados en árboles como XGBoost o LightGBM optimizados con Optuna), registrando parámetros, métricas y artefactos en **MLflow**.

- Control de versiones de datos mediante **DVC**, registro del modelo ganador en el *Model Registry* de MLflow, exposición mediante una API REST construida con **FastAPI** (endpoint `/predict`) y evaluación de *data drift* mes a mes con **Evidently**.

- Superar el desempeño de la línea base (Regresión Lineal), con una meta de reducción de al menos 15 % en el RMSE sobre datos futuros (entrenamiento en enero, validación en febrero y evaluación mensual de marzo a diciembre de 2024).

### Excluye:

- El análisis y modelamiento de registros correspondientes a taxis amarillos (*Yellow Taxis*), vehículos de alquiler de alto volumen (*FHVHV* como Uber/Lyft) u otros medios de transporte público.

- La integración con fuentes externas en tiempo real no contenidas en los datos históricos de la TLC (como reportes climáticos en vivo o incidentes de tráfico en tiempo real vía GPS).

- El desarrollo de una interfaz gráfica de usuario (frontend móvil o web) para pasajeros; el alcance técnico culmina en la exposición del servicio mediante API REST.

## **Metodología**

El proyecto se desarrollará siguiendo el marco metodológico estándar **CRISP-DM** adaptado a prácticas de **MLOps**:

1. **Entendimiento del Negocio y Carga de Datos:** Definición de objetivos, métricas de éxito (RMSE/MAE), estructuración del repositorio bajo la plantilla TDSP/Cookiecutter, adquisición automatizada de los archivos Parquet y CSV, y versionamiento de datos con DVC.

2. **Preprocesamiento y Análisis Exploratorio (EDA):** Cálculo de la variable objetivo (`duration_min`), filtrado de viajes entre 1 y 60 minutos, depuración de distancias absurdas y análisis de patrones temporales (hora, día de la semana) y espaciales (zonas de mayor demanda).

3. **Ingeniería de Características y Modelamiento:** Creación de variables predictoras (`PU_DO` combinando origen y destino, indicadores de fin de semana, franjas horarias), entrenamiento de línea base y optimización de hiperparámetros con Optuna, trazando cada ejecución en MLflow.

4. **Despliegue:** Promoción del mejor modelo en el *Model Registry* de MLflow y construcción de un microservicio de inferencia con FastAPI (y empaquetado en Docker).

5. **Evaluación, Drift y Monitoreo:** Validación temporal fuera de muestra (entrenamiento en enero, validación en febrero para el ajuste con Optuna y evaluación mensual de marzo a diciembre de 2024), generando reportes de degradación y *data drift* con Evidently para establecer políticas de reentrenamiento.

## **Cronograma**

| Etapa | Duración Estimada | Fechas |
|------|---------|-------|
| Entendimiento del negocio y carga de datos | 11 días | del 16 de noviembre al 26 de noviembre |
| Preprocesamiento, análisis exploratorio | 11 días | del 23 de noviembre al 03 de diciembre |
| Modelamiento y extracción de características | 11 días | del 30 de noviembre al 10 de diciembre |
| Despliegue | 11 días | del 07 de diciembre al 17 de diciembre |
| Evaluación y entrega final | 6 días | del 14 de diciembre al 19 de diciembre |

Fechas ajustadas al tiempo actual que tenemos para desarrollar cada fase del proyecto y en la última semana propuesta la entrega consolidada del proyecto final de aula.

## **Equipo del Proyecto**

- Dilson Lazaro Ospino [Estadístico e Ing. de Sistemas]
- Daniel Esteban Molina Sánchez [Ingeniero de Ciencias Computacionales]
- Sebastian Paez Rojas [Ingeniero Químico]

## **Presupuesto**

| Categoría de Gasto | Descripción | Costo Estimado (USD) | Notas / Consideraciones |
| :----------------- | :---------- | :------------------- | :---------------------- |
| **I. Personal** | | | |
| Científico de Datos | Formulación del problema, EDA, ingeniería de variables y modelado. | $2,500 | Dedicación parcial durante 5 semanas |
| Ingeniero de Datos / MLOps | Configuración de DVC, MLflow, FastAPI, Docker y monitoreo con Evidently. | $2,500 | Dedicación parcial durante 5 semanas |
| **II. Infraestructura y Software** | | | |
| Cómputo Cloud / Local | Entorno de ejecución (Google Colab / CPU local). | $0 | El volumen mensual (~60k filas) permite entrenar en CPU sin costo de GPU |
| Almacenamiento Remoto | Repositorio GitHub y almacenamiento DVC en Google Drive. | $0 | Uso de capas gratuitas institucionales/personales |
| Licencias de Software | Python, Pandas, Scikit-learn, XGBoost, MLflow, DVC, FastAPI, Evidently. | $0 | Ecosistema 100% Open-Source |
| **III. Misceláneos** | | | |
| Contingencias | Buffer para imprevistos técnicos (10%). | $500 | Reserva operativa |
| **TOTAL ESTIMADO DEL PROYECTO** | | **$5,500** | |

## **Stakeholders**

- Operadores de flota y la TLC: interesados en mejorar la asignación de vehículos según la duración esperada de los trayectos en los *boroughs* exteriores (Brooklyn, Queens, Bronx, Staten Island y norte de Manhattan).

- Pasajeros: beneficiarios finales que requieren estimaciones precisas de tiempo de llegada (ETA) y tarifas predecibles antes de abordar.

- Equipo docente del diplomado: responsables de auditar la reproducibilidad del código, el versionamiento de datos/modelos y la calidad de las métricas de desempeño.

## **Aprobaciones**

- Dilson Lazaro Ospino
- Daniel Molina Sánchez
- Sebastian Paez Rojas

_Nota:_ Al ser un ejercicio académico y colaborativo proponemos un trabajo juicioso donde cada rol debe participar activamente y contribuir en la construcción integral de los productos a entregar en cada fase.

## **Bibliografía**

- New York City Taxi and Limousine Commission. (2026). TLC Trip Record Data [Conjunto de datos]. City of New York. Recuperado el 23 de noviembre de 2026, de https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page
