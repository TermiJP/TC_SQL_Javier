# Modelo de datos — TechNova (e-commerce electrónica)

## 1. Resumen del modelo

7 tablas, todas relacionadas mediante claves foráneas simples (INT64 autoincremental como PK):

| Tabla | Rol | PK | FK |
|---|---|---|---|
| `customers` | Entidad | `customer_id` | — |
| `categories` | Entidad | `category_id` | — |
| `products` | Entidad | `product_id` | `category_id` → categories |
| `orders` | Entidad | `order_id` | `customer_id` → customers |
| `order_items` | Asociativa (N:M) | `order_item_id` | `order_id` → orders, `product_id` → products |
| `payments` | Entidad débil | `payment_id` | `order_id` → orders |
| `reviews` | Entidad débil | `review_id` | `order_item_id` → order_items |

Ver `er_diagram.png` para el diagrama completo con tipos de dato.

## 2. Esquema detallado

### customers
| Campo | Tipo | Notas |
|---|---|---|
| customer_id | INT64 | PK |
| first_name | STRING | |
| last_name | STRING | |
| email | STRING | UNIQUE |
| phone | STRING | |
| country | STRING | segmentación geográfica |
| city | STRING | |
| acquisition_channel | STRING | organic / paid_ads / social_media / referral / email_marketing / other |
| registration_date | DATE | |
| created_at | TIMESTAMP | |

### categories
| Campo | Tipo | Notas |
|---|---|---|
| category_id | INT64 | PK |
| category_name | STRING | UNIQUE |
| description | STRING | |

### products
| Campo | Tipo | Notas |
|---|---|---|
| product_id | INT64 | PK |
| category_id | INT64 | FK → categories |
| product_name | STRING | |
| description | STRING | |
| sale_price | NUMERIC | precio de venta **actual** |
| cost | NUMERIC | coste, para calcular margen |
| stock_quantity | INT64 | |
| is_active | BOOL | |
| created_at | TIMESTAMP | |

### orders
| Campo | Tipo | Notas |
|---|---|---|
| order_id | INT64 | PK |
| customer_id | INT64 | FK → customers |
| order_status | STRING | pending / confirmed / shipped / delivered / cancelled / returned |
| shipping_address | STRING | dirección de envío del pedido (no necesariamente la del cliente) |
| shipping_city | STRING | |
| shipping_country | STRING | |
| order_date | TIMESTAMP | |
| shipped_date | TIMESTAMP | NULL hasta que se envía |
| delivered_date | TIMESTAMP | NULL hasta que se entrega |

### order_items
| Campo | Tipo | Notas |
|---|---|---|
| order_item_id | INT64 | PK |
| order_id | INT64 | FK → orders |
| product_id | INT64 | FK → products |
| quantity | INT64 | |
| unit_price | NUMERIC | precio **en el momento de la compra** |
| discount_amount | NUMERIC | descuento aplicado a la línea |

Restricción adicional: `UNIQUE(order_id, product_id)` — un producto no debería repetirse como línea distinta dentro del mismo pedido; si se compran 3 unidades, eso se refleja en `quantity`, no en 3 filas.

> No se almacena `line_total`: es un valor derivable (`quantity * unit_price - discount_amount`) y guardarlo introduciría redundancia. Se calcula en las queries.

### payments
| Campo | Tipo | Notas |
|---|---|---|
| payment_id | INT64 | PK |
| order_id | INT64 | FK → orders |
| payment_method | STRING | credit_card / paypal / bank_transfer / ... |
| payment_status | STRING | completed / refunded / pending / failed |
| amount | NUMERIC | |
| payment_date | TIMESTAMP | |

Diseñada como **1:N** respecto a `orders` (no 1:1) para poder registrar reintentos de pago o reembolsos parciales como filas independientes, en vez de sobrescribir el estado.

### reviews
| Campo | Tipo | Notas |
|---|---|---|
| review_id | INT64 | PK |
| order_item_id | INT64 | FK → order_items |
| rating | INT64 | 1–5 |
| comment | STRING | NULL permitido |
| review_date | TIMESTAMP | |

Se relaciona con `order_items` (no con `orders`) porque la valoración es sobre **un producto concreto recibido**, y un pedido puede tener varios productos con opiniones distintas.

## 3. Pregunta clave: relación pedidos–productos

Un pedido puede contener varios productos, y un producto puede aparecer en muchos pedidos → es una relación **N:M**, que no puede modelarse con una FK directa en ninguna de las dos tablas.

Además, la relación en sí tiene **atributos propios** que no pertenecen ni a `orders` ni a `products`: cantidad, precio pagado y descuento son hechos sobre *esa combinación pedido-producto*, no sobre el pedido en general ni sobre el producto en general.

Por eso se necesita la tabla asociativa `order_items`, que convierte la relación N:M en dos relaciones 1:N (`orders`→`order_items` y `products`→`order_items`) y sirve además como contenedor natural de esos atributos.

## 4. Justificación de las formas normales

### 1NF — atomicidad
- Ningún campo contiene listas ni valores compuestos (p. ej. no hay una columna `products_bought` con IDs separados por comas en `orders`).
- No hay grupos repetidos: en vez de columnas `product_1, product_2, product_3` en `orders`, cada producto de un pedido es una fila distinta en `order_items`.
- Todas las tablas tienen PK única.

### 2NF — sin dependencias parciales
Solo es relevante en tablas con clave compuesta. Aquí todas las PK son de una sola columna (surrogate keys), así que 2NF se cumple trivialmente. Si en su lugar `order_items` usara clave compuesta `(order_id, product_id)`, seguiría cumpliéndose: `quantity`, `unit_price` y `discount_amount` dependen de **ambas** columnas a la vez (esa cantidad/precio/descuento son propios de la combinación pedido+producto), no de una sola parte de la clave.

### 3NF — sin dependencias transitivas
Ningún atributo no-clave depende de otro atributo no-clave:
- `products` guarda `category_id`, no `category_name`: si guardáramos el nombre de la categoría directamente, `category_name` dependería de `category_id`, que a su vez depende de `product_id` → dependencia transitiva. Se evita con la FK.
- Igual razonamiento aplica a `orders.customer_id` vs. duplicar datos del cliente en `orders`.

## 5. Preguntas del enunciado, respondidas

**¿Por qué `unit_price` está en `order_items` y no se lee directamente de `products.price`?**
Porque `products.sale_price` cambia con el tiempo (subidas, rebajas), y un pedido histórico debe reflejar el precio que el cliente pagó *en ese momento*, no el precio actual del catálogo. No guardarlo causaría pérdida de información (los ingresos históricos cambiarían cada vez que se actualiza un precio), no redundancia — por eso es correcto guardarlo, no una violación de 3NF.

**¿Por qué `country` está directamente en `customers` y no en una tabla `countries` separada?**
`country` es un atributo funcionalmente dependiente únicamente de `customer_id`, y no necesitamos ningún dato adicional *sobre el país* (moneda, región, código telefónico...) para las necesidades de negocio descritas. Crear una tabla `countries` sin atributos adicionales que justifiquen su existencia sería sobre-normalizar sin beneficio real. Si en el futuro se necesitaran esos metadatos, sí se extraería a una tabla propia.

**Si `orders` guardase `customer_name` además de `customer_id`, ¿qué forma normal se violaría?**
3NF. `customer_name` depende de `customer_id` (un atributo no-clave de `orders`), no de `order_id` (la PK). Es una dependencia transitiva: `order_id → customer_id → customer_name`. Además introduce redundancia (el nombre se repite en cada pedido del cliente) y riesgo de inconsistencia si el cliente cambia su nombre.

## 6. Cardinalidades (resumen)

- `categories` 1 — N `products`
- `customers` 1 — N `orders`
- `orders` 1 — N `order_items`
- `products` 1 — N `order_items`
- `orders` 1 — N `payments`
- `order_items` 1 — N `reviews` (en la práctica, 0 o 1 review por línea)
