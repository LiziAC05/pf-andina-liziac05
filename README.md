\# Andina Retail — Plataforma de Ingeniería de Datos en Azure



\*\*Proyecto final | P.E. Data Engineer\*\*  

\*\*Tecnologías:\*\* Microsoft Azure, Azure Databricks, Apache Spark y PySpark.



\## 1. Descripción del proyecto



Andina Retail es un caso de estudio de una cadena de tiendas de artículos para el hogar y electrodomésticos que opera mediante establecimientos físicos y canales de comercio electrónico.



El proyecto consiste en diseñar e implementar una plataforma de ingeniería de datos en Microsoft Azure que permita integrar información de distintas fuentes, procesarla mediante una arquitectura Lakehouse y generar información analítica para responder preguntas de negocio.



La solución integra datos transaccionales, archivos operacionales y eventos de comportamiento digital, transformándolos en información organizada y disponible para análisis.



\## 2. Objetivos



\- Integrar información de diferentes fuentes estructuradas y semiestructuradas.

\- Implementar procesos de ingesta y almacenamiento en la nube.

\- Transformar los datos mediante Apache Spark y PySpark.

\- Organizar la información en las capas Bronze, Silver y Gold.

\- Automatizar la ejecución de los procesos mediante Azure Data Factory y Databricks.

\- Implementar controles de calidad e idempotencia.

\- Construir tablas analíticas orientadas a las necesidades del negocio.



\## 3. Arquitectura de la solución



La plataforma utiliza servicios de Microsoft Azure y Azure Databricks.



\### Flujo general



```text

&#x20;      FUENTES DE DATOS

&#x20;             |

&#x20;    +--------+---------+

&#x20;    |                  |

&#x20; Azure SQL       Archivos y eventos

&#x20;    |                  |

&#x20;    +--------+---------+

&#x20;             |

&#x20;             v

&#x20;    AZURE DATA FACTORY

&#x20;       (Orquestación)

&#x20;             |

&#x20;             v

&#x20;   AZURE DATA LAKE STORAGE

&#x20;             |

&#x20;             v

&#x20;      AZURE DATABRICKS

&#x20;       + UNITY CATALOG

&#x20;             |

&#x20;             v

&#x20;         BRONZE

&#x20;             |

&#x20;             v

&#x20;         SILVER

&#x20;             |

&#x20;             v

&#x20;          GOLD

&#x20;             |

&#x20;             v

&#x20;     ANÁLISIS DE NEGOCIO

```



Azure Data Factory coordina la ingesta y la ejecución del procesamiento. Databricks ejecuta los notebooks desarrollados con PySpark y Unity Catalog organiza los datos mediante esquemas.



\### Servicios utilizados



| Servicio | Función |

|---|---|

| Azure Data Factory | Orquestación de procesos |

| Azure Data Lake Storage Gen2 | Almacenamiento de datos |

| Azure Databricks | Procesamiento distribuido |

| Apache Spark / PySpark | Transformación de datos |

| Unity Catalog | Organización y gobierno de datos |

| Azure SQL Database | Fuente transaccional |

| Azure Key Vault | Gestión de secretos |

| Managed Identities | Autenticación entre servicios |



\## 4. Arquitectura Lakehouse



La solución implementa tres capas de procesamiento:



\- \*\*Bronze:\*\* recepción y almacenamiento de los datos procedentes de las fuentes.

\- \*\*Silver:\*\* transformación, normalización y aplicación de controles de calidad.

\- \*\*Gold:\*\* generación de tablas analíticas orientadas a las preguntas de negocio.



\### Tablas analíticas Gold



| Tabla | Descripción |

|---|---|

| `ventas\_diarias` | Ventas por tienda, canal y día |

| `clientes\_valor` | Valor y comportamiento de compra de clientes |

| `productos\_rotacion` | Rotación y comportamiento de productos |

| `embudo\_web` | Conversión del proceso de compra digital |

| `conciliacion\_pagos` | Conciliación de pedidos, pagos y facturas |

| `ventas\_importados\_usd` | Análisis de productos importados y tipo de cambio |

| `segmentos\_campania` | Segmentación para campañas |

| `auditoria\_cargas` | Seguimiento y auditoría de las cargas |



\## 5. Orquestación y procesamiento



\### Azure Data Factory



El pipeline maestro `pl\_pede\_andina\_diario` coordina la ingesta y el procesamiento de datos mediante Databricks.



El pipeline utiliza el parámetro `fecha` para gestionar la fecha de procesamiento.



El desencadenador `tg\_pede\_andina\_0130` está configurado para ejecutarse diariamente a la 01:30, hora de Lima. Se encuentra publicado, pero permanece detenido.



\### Azure Databricks



Se implementaron los siguientes Jobs:



\- `job-pede-andina-e4`: procesamiento diario de las capas Bronze, Silver y Gold.

\- `job-pede-andina-e4-horario`: procesamiento de clics y logs mediante Auto Loader, incluyendo la detección de archivos nuevos y tardíos.



\## 6. Notebooks implementados



| Notebook | Responsabilidad |

|---|---|

| `e4\_00\_clics\_logs\_horario` | Procesamiento de clics y logs |

| `e4\_01\_bronce` | Ingesta y procesamiento de la capa Bronze |

| `e4\_02\_plata` | Transformaciones y controles de la capa Silver |

| `e4\_03\_oro` | Construcción de tablas analíticas Gold |



\## 7. Extensiones propias



\### E2 — Control de calidad de datos



Implementada en `e4\_02\_plata`.



Incorpora controles para validar:



\- Estados HTTP dentro del rango 100–599.

\- Latencias mayores o iguales a cero.



Incluye el parámetro `prueba\_e2`, que permite introducir un registro sintético con estado HTTP 700 para comprobar que la validación detiene el procesamiento mediante una aserción (`assert`).



Se realizaron dos pruebas:



1\. Prueba controlada con `prueba\_e2=si`, que produjo el fallo esperado.

2\. Ejecución normal con `prueba\_e2=no`, que finalizó correctamente y conservó 146.282 registros en `pede\_dev.silver.logs`.



\### E3 — Segunda fecha de carga e idempotencia



Implementada en `e4\_01\_bronce`.



Permite realizar cargas para diferentes fechas utilizando el campo `\_load\_date`.



Al repetir una fecha, se eliminan los registros correspondientes a esa fecha antes de volver a cargar los datos, conservando los registros de otras fechas.



La prueba se realizó sobre las cinco tablas SQL: customers, products, orders, order\_items y payments.



Se verificaron los conteos de registros para distintas fechas, incluyendo la repetición de una fecha previamente cargada.



\## 8. Resultados y controles de calidad



Se verificaron los siguientes controles del proyecto:



| Indicador | Resultado |

|---|---:|

| Pedidos pagados de e-commerce | 1.425 |

| Importe e-commerce pagado | 5.339.767,44 |

| Importe de tiendas físicas | 1.003.274,99 |

| Pedidos sin pago | 75 |

| Facturas legibles | 137 de 140 |

| Sesiones con checkout | 333 |

| Eventos únicos | 34.530 de 35.043 |



Los resultados fueron contrastados con los controles definidos para el caso de estudio.



\## 9. Estructura del repositorio



```text

pf-andina-liziac05/

│

├── adf/

│   ├── linked services/

│   ├── datasets/

│   ├── pipelines/

│   └── triggers/

│

├── notebooks/

│   ├── e4\_00\_clics\_logs\_horario.py

│   ├── e4\_01\_bronce.py

│   ├── e4\_02\_plata.py

│   └── e4\_03\_oro.py

│

├── README.md

├── DECISIONES.md

└── evidencia.md

```



La carpeta `adf/` contiene las definiciones JSON exportadas de Azure Data Factory. La carpeta `notebooks/` contiene los notebooks exportados desde Databricks.



\## 10. Seguridad



La solución utiliza identidades administradas para la comunicación entre servicios de Azure cuando corresponde.



\- Azure Data Factory utiliza System Managed Identity.

\- Databricks accede al Data Lake mediante un Access Connector y Managed Identity.

\- Azure Key Vault permite gestionar los secretos.

\- No se deben incluir contraseñas, tokens ni claves de almacenamiento en el repositorio.



\## 11. Documentación complementaria



\- \*\*`DECISIONES.md`:\*\* justificación de las principales decisiones técnicas y de arquitectura.

\- \*\*`evidencia.md`:\*\* registro de las pruebas, verificaciones y resultados obtenidos.



\---



\*\*Proyecto final — P.E. Data Engineer\*\*  

\*\*Caso de estudio: Andina Retail\*\*  

\*\*Plataforma: Microsoft Azure y Azure Databricks\*\*

