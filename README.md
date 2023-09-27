# Datapath
## Data Engineering with AWS
## Programa y edición: Nube de especializaciòn AWS 5ta ed.
##Proyecto Final

Docente: Mario Barbosa
Alumno: Allan Mora


Con los archivos csv, se pretende responder 
1- los indices de desarrollo de Costa Rica vs Centro America para saber su posicion en cuanto a turismo regional
2- cuales son los indices de desarrollo que mas afectan de manera positiva a Costa rica Vs la region LatinoAmericada

Se adjuntaron al repositorio los archivos csv corresponientes al ejercicio.

Enumciando: 
diseñar de una arquitecura de solucion para permanecer en el tiempo y consumir para generar las respuestas las preguntas anteriores.


A continuacion el paso a paso de la arquitectura de solución utilizando propuesta AWS Glue y otros servicios de AWS. 

### Paso 1: Preparación de S3
1. Crear los siguientes buckets en Amazon S3:
   - `zone-raw`: Donde se depositarán inicialmente los archivos CSV.
   - `zone-stage`: Zona intermedia para el procesamiento.
   - `zone-cons`: Zona donde se depositarán los datos procesados en formato Parquet.
   - `zone-athena`: Zona de visualización, también en formato Parquet.

### Paso 2: IAM (Identity and Access Management)
1. Crear un rol para AWS Glue. Este rol tendrá permisos para:
   - Leer y escribir en los buckets S3 mencionados.
   - Ejecutar Crawlers.
   - Crear y ejecutar Jobs en AWS Glue.

2. Asignar políticas al rol para dar los permisos necesarios. Estas políticas pueden incluir:
   - `AmazonS3FullAccess` (esto es para simplificar, pero en un entorno de producción, querrías limitar el acceso solo a los buckets necesarios).
   - `AWSGlueServiceRole`.

3. Crear un usuario en IAM con permisos de administrador para ejecutar y monitorear esta solución. Colócalo en un grupo llamado `GlueAdmins` con dichos permisos.

### Paso 3: AWS Glue
1. **Crawlers**:
   - Crear un Crawler para `zone-raw`. Este explorará los archivos CSV y creará una tabla en el catálogo de datos para cada CSV.

2. **Jobs**:
   - Crear un Job que tome la tabla creada por el Crawler de `zone-raw`, transforme los datos (si es necesario), y escriba los datos en `zone-stage` en formato CSV.
   - Crear otro Job que tome los datos de `zone-stage`, los transforme (si es necesario) y los escriba en `zone-cons` en formato Parquet.
   - Finalmente, crea un Job que copie los datos de `zone-cons` a `zone-athena` (también en Parquet).

3. **Triggers**:
   - Crear un Trigger que se active al completar la ejecución del Crawler de `zone-raw`. Este disparador activará el Job que escribe en `zone-stage`.
   - Crear Triggers en cadena para que, al completar un Job, se active el siguiente.

### Paso 4: AWS Lambda
1. Crear una función Lambda que se active cuando se ingrese un nuevo archivo en `zone-raw`. Esta función iniciará el Crawler correspondiente.

### Paso 5: AWS Athena
1. Abre Athena en la consola AWS.
2. Selecciona el catálogo de datos y la base de datos que creó AWS Glue.
3. Ahora, deberías poder ejecutar consultas SQL directamente sobre los datos en `zone-athena`.

### Consideraciones adicionales:
- **Seguridad**: Utiliza cifrado en tus buckets S3 para proteger los datos en reposo. También puedes activar el logging en tus buckets para auditar cualquier acceso.
- **Monitoreo**: Utiliza CloudWatch para monitorear tus Crawlers y Jobs, y recibir alertas si algo falla.
- **Costo**: Desactiva o elimina recursos que no estés utilizando, y considera configurar alertas de costo en AWS Budgets.

Esta es una visión general de cómo configurar esta solución en AWS. 
En un entorno real, podrías necesitar más detalles y ajustes, 
pero esto debería dar una buena idea del proceso. 

