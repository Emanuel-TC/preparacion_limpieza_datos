# Preparación y Limpieza de Datos - Riesgo Crediticio

Este repositorio contiene la práctica de la materia de Preparación y Limpieza de Datos del Máster en Big Data Science de la UNAV.

## Objetivo del Proyecto (Negocio)
El objetivo principal de este proyecto es construir un modelo predictivo para **reducir el riesgo crediticio (NPL - Non-Performing Loans)**. Mediante la identificación proactiva de clientes con alta probabilidad de incurrir en impago de su tarjeta de crédito el próximo mes, el banco puede activar protocolos de contención, como la reducción de límites o refinanciación, protegiendo el balance financiero.

## Estructura del Proyecto
- `data/`: Carpeta local que contiene el dataset original `default_of_credit_card_clients.xls` (excluida del control de versiones por seguridad).
- `notebook_principal.ipynb`: Jupyter Notebook principal con el desarrollo paso a paso.
- `.gitignore`: Archivo para excluir datos y entornos virtuales del repositorio.

## Setup del Entorno
Para reproducir este proyecto es necesario crear un entorno virtual (Python 3.x) e instalar las siguientes dependencias:
* `pandas` y `numpy`: Para manipulación de datos y operaciones matriciales.
* `matplotlib` y `seaborn`: Para la visualización de datos (EDA y matrices de confusión).
* `scikit-learn`: Para la partición de datos (`train_test_split`), selección de características (`mutual_info_classif`), preprocesamiento (`StandardScaler`), modelado (`LogisticRegression`) y evaluación (`classification_report`, `confusion_matrix`, `f1_score`, etc.).
* `xlrd` o `openpyxl`: Motor necesario para que Pandas pueda leer archivos `.xls`.
*(Nota: Las librerías `math` y `warnings` utilizadas en el código son nativas de Python y no requieren instalación).*

## Diccionario de Datos (Variables Principales)
* `limit_bal`: Límite de crédito otorgado (en dólares taiwaneses), incluye crédito individual y familiar.
* `sex`: Género (1 = masculino, 2 = femenino).
* `education`: Nivel educativo (1 = posgrado, 2 = universidad, 3 = bachillerato, 4 = otros).
* `marriage`: Estado civil (1 = casado, 2 = soltero, 3 = otros).
* `age`: Edad en años.
* `pay_0` a `pay_6`: Historial de pagos pasados. Estado de pago del mes anterior hasta hace 6 meses. (-1 = pagado a tiempo, 1 = retraso de 1 mes, 2 = retraso de 2 meses, etc.).
* `bill_amt1` a `bill_amt6`: Monto del extracto de la cuenta (facturación) de los últimos 6 meses.
* `pay_amt1` a `pay_amt6`: Monto del pago anterior (cuánto pagó el cliente realmente) en los últimos 6 meses.
* `default_payment`: (Variable Objetivo) 1 = Impago el próximo mes, 0 = Sin impago.

## Resumen de Procesamiento y Limpieza
Durante la fase de preparación, se aplicaron las siguientes reglas de negocio y limpieza:
1. **Eliminación de ruido:** Se eliminó la columna `id` al no tener valor predictivo. No se detectaron filas duplicadas exactas ni columnas de varianza cero.
2. **Inconsistencias:** Se corrigieron valores no documentados en las variables `education` (0, 5, 6) y `marriage` (0), agrupándolos en la categoría "Otros".
3. **Valores Nulos:** Se verificó la ausencia total de valores nulos (NaNs) en el dataset.
4. **Tratamiento de Outliers:** Se utilizó el rango intercuartílico (IQR) para detectar anomalías en variables monetarias. Posteriormente, se aplicó la técnica de *Winsorization* (Topeo) al percentil 1% y 99% para mitigar el impacto matemático de los valores extremos sin perder los registros de los clientes.
5. **EDA:** Se evidenció asimetría positiva en las variables monetarias y un desbalanceo en la variable objetivo (~78% clase 0 vs ~22% clase 1), lo que condicionó la elección de métricas de evaluación en el modelado.

## Modelado y Evaluación
1. **Validación:** Se dividió el dataset en Entrenamiento (80%) y Prueba (20%), manteniendo la proporción de la variable objetivo (`stratify`).
2. **Feature Selection:** Se eliminaron variables demográficas (`sex`, `marriage`) por motivos éticos y regulatorios. Se utilizó Información Mutua (*Mutual Information*) para descartar variables con bajo poder predictivo.
3. **Transformación:** Se aplicó `StandardScaler` a las variables continuas para evitar sesgos de magnitud en el modelo de Regresión Logística.
4. **Baseline Model:** Se entrenó una Regresión Logística inicial con pesos balanceados (`class_weight='balanced'`). Se obtuvo un **Recall del 62%** para la clase minoritaria (impago).
5. **Optimización y Negocio:** Se generó una variable conductual de *Feature Engineering* (`max_delay`) que mejoró la detección base reduciendo los Falsos Negativos a 489. Tras realizar un análisis de *Threshold Tuning* (Optimización de Umbral), se concluyó que la decisión de negocio correcta es mantener el umbral conservador en **0.50**; priorizar el *Recall* (minimizar Falsos Negativos) penaliza matemáticamente el *F1-Score*, pero alinea el modelo con el estricto apetito de riesgo de la entidad bancaria.