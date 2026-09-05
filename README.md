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
│   │   ├── er_diagram.png             # diagrama ER completo (tablas, PK/FK, cardinalidades)
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

# 