# Decisiones técnicas — Proyecto Andina Retail



## 1. Introducción



Este documento describe las principales decisiones técnicas adoptadas durante el diseño e implementación de la plataforma de ingeniería de datos de Andina Retail.



Las decisiones consideran los requerimientos del caso de estudio, la arquitectura implementada en Microsoft Azure y las necesidades de procesamiento, seguridad, escalabilidad y control de calidad de los datos.



## 2. Decisión 1: Separación de cargas diarias y horarias



**Decisión:** implementar dos mecanismos de procesamiento independientes para las cargas diarias y los eventos de comportamiento digital.



**Implementación:**

- `job-pede-andina-e4`: procesa las fuentes principales mediante las capas Bronze, Silver y Gold.

- `job-pede-andina-e4-horario`: procesa clics y logs mediante Auto Loader.



**Justificación:**



Los datos transaccionales y los eventos digitales tienen características diferentes. Mientras que las fuentes principales se procesan mediante una ejecución diaria, los clics y logs pueden recibir archivos nuevos durante el día.



Separar ambos procesos permite gestionar las distintas frecuencias de llegada sin obligar a ejecutar nuevamente toda la cadena de procesamiento diario.



Además, Auto Loader permite detectar y procesar archivos nuevos, incluyendo archivos que llegan de manera tardía.

**Nota**: El Job horario se encuentra configurado, pero permanece en pausa. Su implementación permite procesar los eventos digitales de manera independiente de la cadena diaria.

## 3. Decisión 2: Uso de Managed Identities para la autenticación



**Decisión:** El uso de identidades administradas permite establecer mecanismos de autenticación entre servicios de Azure sin incorporar credenciales directamente en los notebooks o archivos de configuración. Complementariamente, Azure Key Vault permite centralizar la gestión de los secretos requeridos por las conexiones que los utilizan.



**Implementación:**

- Azure Data Factory utiliza System Managed Identity.

- Azure Databricks accede al Data Lake mediante un Access Connector con Managed Identity.

- Azure Key Vault centraliza la gestión de secretos necesarios para las conexiones.



**Justificación:**



El uso de identidades administradas permite establecer mecanismos de autenticación entre servicios sin tener que almacenar credenciales directamente en los notebooks o archivos de configuración.



Este enfoque facilita la administración de permisos y reduce la exposición de información sensible.



Los permisos se asignaron de acuerdo con las responsabilidades de cada servicio y los recursos a los que necesita acceder.



## 4. Decisión 3: Uso de Serverless en Databricks



**Decisión:** utilizar cómputo Serverless para ejecutar los Jobs de procesamiento.



**Implementación:**



Los Jobs diarios y horarios utilizan Serverless para ejecutar las tareas de los notebooks.



**Justificación:**


Se eligió esta modalidad para simplificar la administración de la infraestructura de cómputo y evitar la gestión manual de clústeres tradicionales.

El aprovisionamiento del cómputo se gestiona mediante la modalidad Serverless, lo que facilita ejecutar los procesos bajo demanda. Esta decisión también permite concentrar los esfuerzos en el desarrollo y mantenimiento de los procesos de ingeniería de datos, sin administrar directamente la infraestructura subyacente.

En un entorno de desarrollo y aprendizaje, resulta importante complementar esta decisión con el seguimiento del consumo y los costos asociados a las ejecuciones.



## 5. Decisión 4: Arquitectura de datos Bronze, Silver y Gold



**Decisión:** organizar el procesamiento mediante una arquitectura Lakehouse de tres capas.



**Implementación:**



**Bronze**

- Recibe los datos provenientes de las fuentes.

- Conserva la información de las cargas.

- Permite mantener trazabilidad mediante campos como `_load_date`.



**Silver**

- Aplica transformaciones y normalizaciones.

- Integra información procedente de diferentes fuentes.

- Incorpora controles de calidad sobre los datos.



**Gold**

- Genera tablas analíticas orientadas a las necesidades del negocio.

- Organiza los resultados para facilitar consultas y análisis.

- Incluye tablas como `ventas_diarias`, `clientes_valor`, `embudo_web` y `conciliacion_pagos`.



**Justificación:**



La separación por capas permite diferenciar los datos recibidos originalmente de los datos transformados y de los resultados analíticos.



También facilita identificar en qué etapa se encuentra un dato y organizar las responsabilidades de cada proceso.



Esta arquitectura permite incorporar nuevas transformaciones y necesidades analíticas sin mezclar las responsabilidades de ingesta, preparación y consumo.



## 6. Decisión 5: Implementación de E2 — Control de calidad de datos



**Decisión:** incorporar una validación explícita que detenga el procesamiento cuando se detecten estados HTTP o valores de latencia fuera de los rangos permitidos.



**Implementación:**



Se modificó el notebook `e4_02_plata` mediante la incorporación del parámetro `prueba_e2`.



Las reglas implementadas son:



- Estado HTTP entre 100 y 599.

- Latencia mayor o igual a cero.



Se incorporó una prueba controlada que permite generar un registro sintético con estado HTTP 700.



La validación utiliza una aserción (`assert`) para detener la ejecución cuando se encuentran registros inválidos.



**Justificación:**



La existencia de datos incorrectos no debería pasar inadvertida durante el procesamiento. Por ello, se decidió incorporar un control que no solo identifique anomalías, sino que también impida continuar cuando se incumplan las reglas definidas.



**Resultados de las pruebas:**



1. Con `prueba_e2=si`, se introdujo un registro sintético inválido y la tarea Silver terminó con el error esperado.

2. Con `prueba_e2=no`, la tarea terminó correctamente y la tabla `pede_dev.silver.logs` conservó 146.282 registros.



La prueba se realizó sobre la tarea Silver del Job, no sobre una ejecución completa de toda la cadena Bronze → Silver → Gold.



## 7. Decisión 6: Implementación de E3 — Segunda fecha de carga e idempotencia



**Decisión:** incorporar un mecanismo de carga por fecha que permita repetir una carga sin duplicar los registros correspondientes a esa misma fecha.



**Implementación:**



Se modificó la lógica existente del notebook `e4_01_bronce`.



El mecanismo utiliza el campo `_load_date` y contempla los siguientes pasos:



1. Identificar la fecha de carga.

2. Eliminar los registros asociados a esa misma fecha.

3. Realizar nuevamente la ingesta.

4. Conservar los registros correspondientes a otras fechas.



La implementación se aplicó a las cinco tablas provenientes de Azure SQL Database:



- customers

- products

- orders

- order_items

- payments


La repetición de la carga se verificó para la fecha 2026-11-22. Se comprobaron los conteos de registros de las cinco tablas para dicha fecha y la conservación de los registros correspondientes a fechas anteriores. La validación se basó en la comparación de conteos de registros, no en una comparación de hashes o identidad fila por fila.


**Justificación:**



Durante los procesos de ingeniería de datos es posible necesitar repetir una carga por errores, verificaciones o reprocesamientos.



Sin un mecanismo de control, repetir una fecha podría generar registros duplicados.



La estrategia implementada permite volver a procesar una fecha conservando la información de las demás fechas de carga.



**Validación:**



Se realizaron cargas para distintas fechas, incluyendo la repetición de una fecha previamente procesada.



Se verificaron los conteos por `_load_date` para las cinco tablas y se comprobó que los registros de otras fechas se conservaran.



La validación se basó en la comparación de conteos de registros, no en una comparación de hashes o identidad fila por fila.



## 8. Consideraciones finales



Las decisiones adoptadas responden a las necesidades de integración, procesamiento y análisis de información de Andina Retail.



La separación de frecuencias de procesamiento, el uso de identidades administradas, el cómputo Serverless y la arquitectura Lakehouse permiten organizar la solución y facilitar su operación.



Por otro lado, las extensiones E2 y E3 incorporan controles adicionales para mejorar la confiabilidad de los procesos de transformación y carga.



Estas decisiones constituyen la base técnica de la plataforma implementada en Azure y Databricks.



---
