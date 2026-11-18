# Sistema de decisión para PagoYa — Grupo 4

**Integrantes:** Padilla Silva Darwin Medardo, Obando Castillo Juan Fernando
**Problema:** Crédito
**Aplicación:** PagoYa Crédito

## 1. Problema y datos

El objetivo es apoyar la decisión de aprobación de solicitudes de microcrédito. La variable objetivo es `default`: 1 indica que la solicitud terminó en incumplimiento y 0 que el cliente fue buen pagador.

El conjunto utilizado contiene **2,500 solicitudes** y una tasa observada de default de **10.88%**. No se encontraron registros duplicados ni valores nulos.

Para enriquecer el análisis se construyó la variable `ratio_monto_ingreso`, que relaciona el monto solicitado con el ingreso mensual del cliente.

También se analizó la variable categórica `tipo_empleo`. Las tasas observadas fueron:

- **Informal:** 20.89% de default.
- **Independiente:** 8.79% de default.
- **Dependiente:** 8.20% de default.

Debido a que las categorías de `tipo_empleo` no poseen un orden numérico natural, esta variable se incorporó al modelado mediante **One-Hot Encoding**.

La decisión de negocio considera que un default genera una pérdida igual al 100 % del monto prestado, mientras que un buen pagador deja una ganancia equivalente al 15 % del monto.

## 2. Modelos comparados

Se compararon tres enfoques: **regresión logística, XGBoost y regresión logística bayesiana**.

Se utilizó una partición fija 70/30 estratificada (`random_state=42`) y el conjunto de validación se obtuvo únicamente del entrenamiento para evitar fuga de información.

La imputación, el escalado y la codificación de `tipo_empleo` se ajustaron exclusivamente utilizando el subconjunto de entrenamiento.

### Métricas en prueba

| Modelo          |    AUC |   Exactitud_balanceada |   Brier |
|:----------------|-------:|-----------------------:|--------:|
| Logística       | 0.7395 |                 0.7065 |  0.1768 |
| Bayes logístico | 0.7371 |                 0.5458 |  0.087  |
| XGBoost         | 0.6809 |                 0.5633 |  0.1168 |

### AUC en validación

| Modelo          |   AUC_validacion |
|:----------------|-----------------:|
| Logística       |           0.7418 |
| Bayes logístico |           0.7415 |
| XGBoost         |           0.6942 |

El modelo elegido fue **Logística**, porque obtuvo el mayor AUC en el conjunto de validación, con un valor de **0.7418**.

En prueba alcanzó:

- **AUC:** 0.7395
- **Exactitud balanceada:** 0.7065
- **Brier:** 0.1768

La selección del modelo se realizó exclusivamente con el conjunto de validación. El conjunto de prueba se utilizó únicamente para medir el desempeño final.

## 3. Qué explica el resultado

La regresión logística bayesiana se utilizó como herramienta de explicabilidad e incertidumbre para analizar el efecto de las variables mediante sus distribuciones posteriores e intervalos creíbles.

La variable con mayor efecto posterior en valor absoluto fue **`tipo_empleo_informal`**, con un coeficiente medio de **1.489**, un intervalo creíble al 90 % entre **1.140** y **1.843**, y un odds ratio de **4.433**.

La categoría **informal**, comparada con la categoría de referencia **dependiente**, **aumenta** las odds de default, manteniendo constantes las demás variables.

La variable categórica `tipo_empleo` fue incorporada mediante **One-Hot Encoding**, utilizando **dependiente** como categoría de referencia. Los coeficientes de las demás categorías se interpretan respecto a esta categoría.

Los intervalos creíbles al 90 % permiten distinguir efectos cuya dirección es relativamente estable de aquellos que presentan mayor incertidumbre.

## 4. Recomendación con cifras

El umbral óptimo de aprobación se seleccionó exclusivamente con el conjunto de validación y fue de **46.00%** sobre el score de riesgo estimado por el modelo.

Aplicando este umbral al conjunto de prueba:

- **Solicitudes aprobadas:** 502
- **Tasa de aprobación:** 66.93%
- **Defaults entre los aprobados:** 28
- **Ganancia estimada:** USD 62,393.00
- **Ganancia aprobando a todos:** USD 12,814.00
- **Mejora frente a aprobar a todos:** USD 49,579.00

La recomendación es aprobar las solicitudes cuyo **score de riesgo estimado por el modelo** sea igual o inferior al umbral y enviar las solicitudes de mayor riesgo a rechazo o revisión manual.

Esta política permite reducir la exposición a solicitudes con mayor riesgo de incumplimiento y mejorar el resultado económico frente a una estrategia de aprobación generalizada.

## 5. Limitaciones y próximos pasos

Los datos del proyecto son didácticos y específicos del Grupo 4, por lo que los resultados no deben interpretarse directamente como una política real de crédito.

La decisión se basa en modelos estadísticos y no debe sustituir controles de riesgo, requisitos legales ni revisión humana.

Antes de una implementación real se recomienda evaluar la calibración del score antes de interpretarlo como una probabilidad real de default, revisar posibles sesgos entre grupos de clientes, verificar la estabilidad del efecto de `tipo_empleo`, monitorear cambios en la distribución de los datos y controlar el deterioro del desempeño del modelo.

También debe revisarse periódicamente el umbral de aprobación cuando cambien la pérdida asociada al default, la rentabilidad del crédito o las condiciones de la cartera.
