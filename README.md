### TC_SQL_Javier

Team Challenge de bases de datos: diseño e implementación desde cero de una base de datos
relacional para un e-commerce de electrónica y accesorios tecnológicos que opera en varios
países de Europa, normalizada hasta 3NF e implementada en Google BigQuery.

# Estructura del repositorio
```
tc-sql-tu_equipo/
├── parte_1_sql_murder_mystery/
│   └── investigacion.ipynb
├── parte_2_modelo_bigquery/
│   ├── data/                          # vacío — los datos viven en BigQuery
│   ├── docs/
│   │   ├── er_diagram.png             # diagrama ER completo 
│   │   └── normalizacion.md           # modelo detallado + justificación 1NF/2NF/3NF
│   └── notebooks/
│       ├── 01_setup_bigquery.ipynb    # crea el dataset y las 7 tablas
│       ├── 02_generate_data.ipynb     # genera datos sintéticos con Faker y los carga
│       └── 03_queries_verification.ipynb  # 7 queries analíticas sobre el modelo
├── .env.example
├── .gitignore
├── README.md                          # este fichero
└── requirements.txt
```

# Flujo de trabajo

Lo primero fue crear el repo, y estructurar los archivos y todo lo necesario para empezar a programar.

Luego empece a crear el ER de la base de datos y normalizandola 3NF siguiendo las instrucciones de la guia y las intrucciones del repositorio.

Despues empece a programar el SQL Murder Mistery.

## Notebooks

# 1. Set-up bigquery

Luego empece con los notebooks siguiendo los pasos marcados. Lo primero fue hacer las credenciales de Google Cloud y crear el Servicies account. De ahi asacar la clave de json y linkar en el .env las credenciales para poder conectarme a mi cuenta.
Desarrolle el primer notebook sin mucho problema, cree las tablas y lo verifique.

# 2.Generate_data

Aqui con la libreria FAker, utilizamos sus funciones para ordenaor de forma correcta las 7 tablas respetando el orden de dependencia FK. Lo separamos todo bien por categorias.

# 3.Verficacion

Aqui hacemos consultas a la bigquery para probar su funcionamiento correcto. Preguntando cosas como ingresos al mes, top 10 productos, etc. basicamente un ejercicio de verificacion de que todo funciona en orden y de foma correcta.