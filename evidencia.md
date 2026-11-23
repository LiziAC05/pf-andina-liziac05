# Evidencias del proyecto Andina Retail



Este documento organiza las evidencias de implementación, ejecución, validación y costos del proyecto final de Data Engineering.



## 1. Arquitectura



| Archivo | Descripción |
|---|---|
| `evidencias/arquitectura/andina_arqui.jpg` | Diagrama general de la arquitectura implementada en Azure. |



## 2. Azure Data Factory



| Archivo | Descripción |
|---|---|
| `evidencias/adf/Captura de pantalla 2026-11-21 231510.png` | Ejecución de las actividades del pipeline de ingesta. |
| `evidencias/adf/Captura de pantalla 2026-11-22 140830.png` | Ejecución exitosa del pipeline maestro `pl_pede_andina_diario`. |
| `evidencias/adf/detalle_copia_sql_adls.png` | Detalle de transferencia de datos desde Azure SQL Database hacia ADLS Gen2. |



La ejecución del pipeline maestro muestra la finalización satisfactoria de las actividades `sv_fecha`, `ep_ingesta`, `job_andina` y `sv_resultado`.



## 3. Azure Databricks



### Ejecución y procesamiento

| Archivo | Descripción |
|---|---|
| `evidencias/databricks/Captura de pantalla 2026-11-21 225310.png` | Evidencia de ejecución de Databricks. |
| `evidencias/databricks/Captura de pantalla 2026-11-22 152827.png` | Evidencia de ejecución y procesamiento. |
| `evidencias/databricks/Captura de pantalla 2026-11-22 153158.png` | Evidencia de procesamiento y resultados. |
| `evidencias/databricks/Captura de pantalla 2026-11-22 153545.png` | Evidencia de resultados del entorno Databricks. |
| `evidencias/databricks/ejecucion_job_andina.png` | Ejecución satisfactoria del Job diario, con las tareas Bronze, Silver y Gold completadas. |

### Tablas analíticas Gold

| Archivo | Descripción |
|---|---|
| `evidencias/databricks/tablas_gold.png` | Consulta del esquema `pede_dev.gold`, donde se identifican las ocho tablas analíticas. |
| `evidencias/databricks/ventas_diarias.png` | Resultados de la tabla de ventas diarias. |
| `evidencias/databricks/clientes_valor.png` | Resultados de la tabla de valor de clientes. |
| `evidencias/databricks/productos_rotacion.png` | Resultados de la tabla de rotación de productos. |



## 4. Extensión E2: control de calidad



La extensión E2 incorpora validaciones de calidad de datos que detienen el procesamiento cuando se detectan registros inválidos.



| Archivo | Descripción |
|---|---|
| `evidencias/e2/Captura de pantalla 2026-11-22 191124.png` | Configuración de la prueba de calidad. |
| `evidencias/e2/prueba_fallida.png` | Prueba negativa: detección de un registro inválido y generación de `AssertionError`. |
| `evidencias/e2/prueba_exitosa.png` | Prueba positiva: ejecución satisfactoria con el parámetro `prueba_e2=no`. |



La prueba negativa utilizó un registro sintético con estado HTTP 700, fuera del rango permitido y provocó la interrupción esperada mediante una aserción. La prueba positiva permitió completar la validación y conservar 146.282 registros en `pede_dev.silver.logs`.



## 5. Extensión E3: idempotencia



La extensión E3 permite repetir la carga de una fecha sin duplicar los registros correspondientes a esa misma fecha y conservando los datos de fechas anteriores.



| Archivo | Descripción |
|---|---|
| `evidencias/e3/cambio_parametro.png` | Configuración del parámetro de fecha `2026-11-22`. |
| `evidencias/e3/ejecucion_bronce.png` | Ejecución satisfactoria de la tarea Bronze para la fecha seleccionada. |



La validación de conteos por fecha se realizó mediante consultas en Databricks. Los resultados de dichas consultas se documentan como parte de la comprobación de idempotencia.



## 6. Costos



| Archivo | Descripción |
|---|---|
| `evidencias/costos/costos_general.png` | Vista general del análisis de costos de Azure. |
| `evidencias/costos/costos_recursos.png` | Desglose de costos por recursos. |



## 7. Consideraciones



- Las capturas corresponden a las ejecuciones realizadas durante la implementación y validación del proyecto.

- Las pruebas negativas de E2 fueron intencionales y forman parte de la demostración del control de calidad.

- La ejecución de E3 documentada en este repositorio corresponde a la tarea Bronze; la validación de idempotencia se complementa con los conteos consultados por fecha.

- Los estados de los triggers y Jobs se mantienen según la configuración de seguridad y control operativo del proyecto.
  
- El desencadenador diario tg_pede_andina_0130 se encuentra publicado, pero detenido. El Job horario de Databricks permanece en pausa.



