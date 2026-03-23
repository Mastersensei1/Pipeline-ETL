# Pipeline-ETL
Este proyecto implementa un pipeline ETL completo orquestado con Apache Airflow y contenerizado con Docker Compose. La arquitectura se divide en tres capas: el host local, el entorno Docker, y las herramientas de visualización externas.


🖥️ Host (Tu máquina local)
El archivo fuente books_amazon.csv reside en tu máquina local, dentro del directorio ./data/. Este directorio se monta como un volumen de Docker, lo que permite que los contenedores accedan al CSV sin necesidad de copiarlo dentro de la imagen. Lo mismo aplica para los directorios ./dags/ y ./scripts/, que se montan en tiempo de ejecución para facilitar el desarrollo sin reconstruir la imagen.

🐳 Entorno Docker (docker-compose.yml)
Todos los servicios corren sobre una red interna compartida llamada etl_network. El entorno está compuesto por los siguientes contenedores:
postgres_airflow
Base de datos PostgreSQL dedicada exclusivamente al metadata interno de Airflow (estado de DAGs, logs de ejecución, variables, conexiones, etc.). No almacena datos del ETL. Sus datos persisten en el volumen postgres_airflow_data.
airflow-init Contenedor de inicialización que se ejecuta una sola vez al levantar el stack. Corre airflow db migrate para preparar el esquema de la base de datos y crea el usuario administrador. Depende de postgres_airflow.
airflow-webserver Expone la UI de Airflow en localhost:8080. Permite monitorear y disparar manualmente los DAGs, revisar logs de cada tarea y gestionar conexiones y variables

Airflow-scheduler
El corazón del orquestador. Detecta automáticamente los DAGs en la carpeta montada, evalúa los schedules y despacha las tareas al executor (LocalExecutor). Comparte la misma imagen que el webserver, construida desde el Dockerfile local.
Dockerfile (imagen base personalizada)

La imagen base es apache/airflow:2.7.0, extendida con: Dependencias de sistema: gcc, python3-dev, libpq-dev (necesarias para compilar el driver psycopg2 que conecta Python con PostgreSQL). Dependencias Python instaladas desde requirements.txt. postgres_data

Base de datos PostgreSQL destinada a almacenar los datos procesados por el ETL. Es el destino final del pipeline. Expone el puerto 5433 en el host (mapeado internamente al 5432 estándar), lo que permite conectarse desde herramientas externas. Sus datos persisten en el volumen postgres_data.

pgAdmin
Herramienta web de administración de PostgreSQL, accesible en localhost:5050. Permite explorar las tablas, ejecutar queries y verificar el resultado del ETL sin instalar nada adicional. Su configuración (conexiones guardadas, preferencias) persiste en el volumen pgadmin_data.
