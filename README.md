# Prueba Tecnica Atlas.

Instalacion de dependencias del proyecto, configurar conexiones a BD y probar ETL + API.

## Instaladores

- **Python 3.14**  
  Descarga: https://www.python.org/downloads/ 

- **MySQL WorkBench**  
  Descarga: https://dev.mysql.com/downloads/workbench/
  Opcional: MySQL Workbench para administrar la BD.

- **Git**   
  Descarga: https://git-scm.com/install/windows

- **Power BI Desktop**  
  Descarga: https://www.microsoft.com/es-es/power-platform/products/power-bi/desktop

  **"Opcional" PostMan**        
  Descarga: https://www.postman.com/downloads/

## Librerias y dependencias

Las librerias instaladas en el proyecto fueron pandas, sqlalchemy, pymysql. Estas fueron almacenadas en el archivo "requirements.txt". para luego ser ejecutadas con el siguiente comando.

```markdown
pip install -r requirements.txt
```

Para poder realizar la API se instalo la dependencia "FastAPI uvicorn" ejecutando el siguiente comando
```markdown
pip install fastapi uvicorn sqlalchemy pymysql
```

## Construccion de Base de Datos
### Diseño, creación y consultas SQL del proyecto
Este parte del documento describe todo el proceso realizado en la Sesión A, incluyendo la construcción de la base de datos origen `tienda`, la definición de sus tablas, cargas iniciales, vistas y consultas analíticas requeridas para el dashboard y la API.
#### 1. Objetivo de la Sesión A
El propósito de esta sesión es:

- Crear una base de datos origen llamada **`tienda`** con sus respectivas tablas  .  
- Insertar datos de ejemplo (clientes, productos, ventas).  
- Construir vistas útiles para análisis.  
- Generar queries que luego utilizarán Power BI y la API.

Esta base de datos será la fuente de datos para la **ETL (Sesión B)** y la posterior **API (Sesión C).**

#### 2. Creación de la base de datos "Sección A"
- Archivo *database.sql*

    Este archivo es el encargado de crear la base de datos inicial **`tienda`**

- Archivo *tables.sql*

    Este es el archivo encargado de crear las 3 tablas que utilizaremos (clientes,productos y ventas)
- Archivo *inserts.sql*

    Este archivo es el encargado de ingresar los datos en cada una de las tablas

    **Tabla Clientes:**
    nombre, ciudad, fecha_registro
    
    **Tabla Productos:**
    categoria, precio

    **Tabla Ventas:**
    id_cliente, id_producto, fecha_venta, cantidad

    Para validar los registros insertados se puede ejecutar
    ```markdown
    USE tienda;
    SELECT * FROM clientes;
    SELECT * FROM productos;
    SELECT * FROM ventas;
    ```
    
- Archivo *queries.sql*

    Este sera el encargado de ejecutar las consultas para obtener Las ventas totales por mes y categoría de producto y el TOP 5 de clientes con mayores compras en el último año. 

- Archivo *views.sql*

    Este ultimo archivo crea una vista que muestra: cliente, ciudad, total de compras y última fecha de compra. 

## Construccion de ETL


Esta sección describe el proceso de conexion, creación, configuración y ejecución de la ETL desarrollada en Python para extraer los datos de la base de datos **`tienda`**, tranformarlos y cargarlos en una nueva base de datos que se llamara **`nueva_tienda`**

#### 3. Construccion ETL "Sección B"
- Archivo de configuracion *config.py*
    
    En este archivo se define dos diccionarios en Python que contienen las credenciales y parámetros de conexión para las dos bases de datos. la de origen y la que se va a crear en la ETL.

- Archivo de extraccion *extract.py*

    En este archivo se conecta a la base de datos con las credenciales del archivo anterior de **`config.py`** y extrae los datos de las tablas (`clientes`, `productos`, `ventas`) de la primera base de datos y los carga en DataFrames para su posterior transformación.

- Archivo de transformacion *transform.py*

    Recibe los 3 DataFrames de **`extract.py`**, convierte todos los nombres de columnas a minúsculas usando str.lower() y los devuelve transformados

- Archivo de carga y creacion de nueva base de datos  *load.py*

    Este archivo es el encargado de crear la nueva base de datos destino **`nueva_tienda`** y replica toda la data almacenada.

- Orquestador de la ETL *main.py*

    Este archivo utiliza las funciones creadas en los archivos anteriores **`extract_data`**, **`transform_data`**, **`create_database`** y **`load_data`** para ejecutar el proceso de ETL, coordinando la lectura de datos desde la BD origen aplicando las transformaciones y cargando el resultado en la BD destino, mostrando el progreso.

##### Ejecucion:
Desde la carpeta del proyecto:
```
cd etl
```
luego ejecuta 
```
python main.py
```
aparecera la siguiente informacion en la terminal
```
Extrayendo datos...
Transformando datos...
Creando base destino...
Cargando datos...
ETL EJECUTADO CORRECTAMENTE
```
Una vez ejecutada, se podra visualizar la nueva base de datos creada. **`nueva_tienda`** con las 3 tablas cargadas de forma correcta. Esta se podra visualizar ejecutando desde MYSQL 
```
SHOW DATABASES;
```
o 
```
USE nueva_tienda;
SHOW TABLES;
```
## Creaccion de API


En esta seccion describe la creaccion de la API con un endpoint que devuelve un JSON del total de las ventas por categoria
#### 4. Creaccion API "Sección C"

- Archivo *database.py*

    Este fragmento prepara la conexión a la base de datos destino y ajusta el `PYTHONPATH`  para importar módulos del proyecto. Crea el engine con SQLAlchemy (`create_engine`), que se usará para leer/escribir datos en MySQL.

- Archivo *routes.py*

    Este módulo define un router de FastAPI con el endpoint `GET /ventas-por-categoria` que calcula el total de ventas por categoría de producto consultando la base de datos vía SQLAlchemy.

- Archivo *main.py*

    Este archivo realiza lo siguiente:

    - Crea la instancia de la aplicación FastAPI (`app`).
    - Configura el título de la API: API Ventas.
    - Incluye el router definido en `routes.py`, que contiene los endpoints ventas por categoria



##### Ejecucion:

Desde la carpeta del proyecto ejecuta:
```
uvicorn api.main:app --reload
```

En la consola de la terminal aparecera lo siguiente:
```
INFO:     Will watch for changes in these directories: ['Directorio_donde_esta_el_proyecto']
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [4016] using StatReload
INFO:     Started server process [6460]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```
una vez lanzada la API, podremos visualizar la informacion en el navegador, o en Postman con el metodo GET y pegando la siguiente URL.
```
http://127.0.0.1:8000/ventas-por-categoria
```

Visualizando desde Postman o el navegador tiene que lanzar el resultado JSON de esta forma.
```
{
    "status": "ok",
    "total_por_categoria": [
        {
            "categoria": "Electrónica",
            "total_ventas": 6500.0
        },
        {
            "categoria": "Hogar",
            "total_ventas": 2450.0
        },
        {
            "categoria": "Ropa",
            "total_ventas": 400.0
        }
    ]
}
```
Confirmando asi la creaccion correcta de la API.

## Creaccion y visualizacion de Dashboards
Esta sección documenta el proceso realizado para diseñar, construir y publicar los dashboards de análisis de ventas utilizando Power BI Desktop. El objetivo principal es visualizar la información cargada en la base de datos
#### 5. Creaccion Dashboards en Power Bi "Sección D"

En Power BI Desktop, se configuró la conexión mediante:
```
Inicio → Obtener datos → SQL Server
```
Parámetros usados:

- Servidor: ``` localhost```

- Base de datos: ```nueva_tienda```

- Método de autenticación: ```SQL Login (usuario y contraseña configurados en la ETL)```

- Modo: ```Import ```

Una vez conectados se seleccionaron las tablas:
- ```Clientes```

- ```Productos```

- ```Ventas```

##### Relacciones del modelo:
- Clientes (1) → Ventas (N)
- Productos (1) → Ventas (N)

##### Creación de columnas calculadas y medidas DAX:

- ```Total Ventas = SUM(Ventas[Monto])```

- ```Cantidad Productos = COUNT(Ventas[IdProducto])```

- ```Ticket Promedio = [Total Ventas] / DISTINCTCOUNT(Ventas[IdVenta])```

Una vez con estas configuracion, se procedio a crear los 3 dashboards 
- Grafico de ventas por categoria 
- Gráfico de categorías por mes 
- Clientes que más compras

Cabe aclarar que el archivo .pbix se subio en el repositorio en la carpeta de Power BI, donde se podra descargar y visualizar los dashboards, de igual manera, el archivo se subio en la misma carpeta en formato .zip para su correcta visualizacion.
