# Taller Diseño de base de datos - Postgres

![](https://i.ibb.co/0brN66k/image.png)

Teniendo en cuenta el diagrama entidad relación resuelva los siguientes requerimientos:

1. Genere los comandos DDL que permitan la creación de la base de datos del DER suministrado.
2. Genere los comandos DML que permitan insertar datos en la base de datos creada en el paso 1.

## Resolución del taller

### 1. Comandos DDL

```SQL
CREATE DATABASE miscompras;

DROP SCHEMA IF EXISTS miscompras CASCADE;
CREATE SCHEMA IF NOT EXISTS miscompras;
SET search_path TO miscompras;

CREATE TABLE miscompras.clientes (
    id                 VARCHAR(20)  PRIMARY KEY,
    nombre             VARCHAR(40)  NOT NULL,
    apellidos          VARCHAR(100) NOT NULL,
    celular            NUMERIC(10,0),
    direccion          VARCHAR(80),
    correo_electronico VARCHAR(70)
);

CREATE TABLE miscompras.categorias (
    id_categoria  INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    descripcion   VARCHAR(45) NOT NULL,
    estado        SMALLINT     NOT NULL DEFAULT 1,
    CONSTRAINT categorias_estado_chk CHECK (estado IN (0,1))
);

CREATE TABLE miscompras.productos (
    id_producto    INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nombre         VARCHAR(45)   NOT NULL,
    id_categoria   INT           NOT NULL,
    codigo_barras  VARCHAR(150),
    precio_venta   NUMERIC(16,2) NOT NULL,
    cantidad_stock INT           NOT NULL DEFAULT 0,
    estado         SMALLINT      NOT NULL DEFAULT 1,
    CONSTRAINT productos_precio_chk   CHECK (precio_venta >= 0),
    CONSTRAINT productos_stock_chk    CHECK (cantidad_stock >= 0),
    CONSTRAINT productos_estado_chk   CHECK (estado IN (0,1)),
    CONSTRAINT productos_fk_categoria FOREIGN KEY (id_categoria)
        REFERENCES miscompras.categorias(id_categoria)
        ON UPDATE CASCADE
        ON DELETE RESTRICT
);

-- Unico por cÃ³digo de barras si se usa, permite varios NULL
CREATE UNIQUE INDEX IF NOT EXISTS ux_productos_codigo_barras
    ON miscompras.productos (codigo_barras)
    WHERE codigo_barras IS NOT NULL;

CREATE INDEX IF NOT EXISTS idx_productos_id_categoria
    ON miscompras.productos (id_categoria);


CREATE TABLE miscompras.compras (
    id_compra    INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    id_cliente   VARCHAR(20)  NOT NULL,
    fecha        TIMESTAMP    NOT NULL DEFAULT NOW(),
    medio_pago   CHAR(1)      NOT NULL,
    comentario   VARCHAR(300),
    estado       CHAR(1)      NOT NULL,
    CONSTRAINT compras_fk_cliente FOREIGN KEY (id_cliente)
        REFERENCES miscompras.clientes(id)
        ON UPDATE CASCADE
        ON DELETE RESTRICT
);

-- Indice para busquedas por cliente
CREATE INDEX IF NOT EXISTS idx_compras_id_cliente
    ON miscompras.compras (id_cliente);

CREATE TABLE miscompras.compras_productos (
    id_compra    INT           NOT NULL,
    id_producto  INT           NOT NULL,
    cantidad     INT           NOT NULL,
    total        NUMERIC(16,2) NOT NULL,
    estado       SMALLINT      NOT NULL DEFAULT 1,
    CONSTRAINT compras_productos_pk PRIMARY KEY (id_compra, id_producto),
    CONSTRAINT compras_productos_cantidad_chk CHECK (cantidad > 0),
    CONSTRAINT compras_productos_total_chk    CHECK (total >= 0),
    CONSTRAINT compras_productos_estado_chk   CHECK (estado IN (0,1)),
    CONSTRAINT compras_productos_fk_compra FOREIGN KEY (id_compra)
        REFERENCES miscompras.compras(id_compra)
        ON UPDATE CASCADE
        ON DELETE CASCADE,
    CONSTRAINT compras_productos_fk_producto FOREIGN KEY (id_producto)
        REFERENCES miscompras.productos(id_producto)
        ON UPDATE CASCADE
        ON DELETE RESTRICT
);

-- Indice adicional para acelerar consultas por producto
CREATE INDEX IF NOT EXISTS idx_cp_id_producto
    ON miscompras.compras_productos (id_producto);
```

## 2. Comandos DML

> Inserciones para la tabla `clientes`
```SQL
INSERT INTO miscompras.clientes (id, nombre, apellidos, celular, direccion, correo_electronico) VALUES
('CC1001', 'Camila',   'Ramírez Gómez',     3004567890, 'Cra 12 #34-56, Bogotá',      'camila.ramirez@example.com'),
('CC1002', 'Andrés',   'Pardo Salinas',     3109876543, 'Cl 45 #67-12, Medellín',     'andres.pardo@example.com'),
('CC1003', 'Valeria',  'Gutiérrez Peña',    3012223344, 'Av 7 #120-15, Bogotá',       'valeria.gutierrez@example.com'),
('CC1004', 'Juan',     'Soto Cárdenas',     3155556677, 'Cl 9 #8-20, Cali',           'juan.soto@example.com'),
('CC1005', 'Luisa',    'Fernández Ortiz',   3028889911, 'Cra 50 #10-30, Bucaramanga', 'luisa.fernandez@example.com'),
('CC1006', 'Carlos',   'Muñoz Prieto',      3014567890, 'Cl 80 #20-10, Barranquilla', 'carlos.munoz@example.com'),
('CC1007', 'Diana',    'Rojas Castillo',    3126665544, 'Cra 15 #98-45, Bogotá',      'diana.rojas@example.com'),
('CC1008', 'Miguel',   'Vargas Rincón',     3201234567, 'Cl 33 #44-21, Cartagena',    'miguel.vargas@example.com');
```

> Inserciones para la tabla `categorias`
```SQL
INSERT INTO miscompras.categorias (descripcion, estado) VALUES
('Café',       1),
('Lácteos',    1),
('Panadería',  1),
('Aseo',       1),
('Snacks',     1),
('Bebidas',    1);
```

> Inserciones para la tabla `compras`
```SQL
INSERT INTO miscompras.compras (id_cliente, fecha, medio_pago, comentario, estado) VALUES
('CC1001', '2025-07-02 10:15:23', 'T', 'Compra semanal',            'A'),
('CC1002', '2025-07-03 18:45:10', 'E', 'Para oficina',              'A'),
('CC1003', '2025-07-05 09:05:00', 'C', NULL,                        'A'),
('CC1001', '2025-07-10 14:22:40', 'T', 'Reabastecimiento café',     'A'),
('CC1004', '2025-07-12 11:11:11', 'E', 'Desayuno fin de semana',    'A'),
('CC1005', '2025-07-15 19:35:05', 'T', 'Compras del mes',           'A'),
('CC1006', '2025-07-18 08:55:30', 'C', 'Limpieza y bebidas',        'A'),
('CC1007', '2025-07-20 16:01:00', 'T', 'Merienda en familia',       'A'),
('CC1008', '2025-07-25 12:20:45', 'E', 'Reunión con amigos',        'A'),
('CC1002', '2025-08-01 17:05:12', 'T', 'Compras para semana',       'A'),
('CC1003', '2025-08-02 10:40:33', 'T', 'Bebidas y snacks',          'A'),
('CC1004', '2025-08-05 13:50:00', 'C', 'Dulces y panadería',        'A');
```

> Inserciones para la tabla `productos`
```SQL
INSERT INTO miscompras.productos (nombre, id_categoria, codigo_barras, precio_venta, cantidad_stock, estado) VALUES
('Café de Colombia 500g',
 (SELECT id_categoria FROM miscompras.categorias WHERE descripcion='Café'),
 '7701234567001', 28000.00,  250, 1),
('Café Orgánico Sierra Nevada 250g',
 (SELECT id_categoria FROM miscompras.categorias WHERE descripcion='Café'),
 '7701234567002', 22000.00,  180, 1),
('Leche entera 1L',
 (SELECT id_categoria FROM miscompras.categorias WHERE descripcion='Lácteos'),
 '7701234567003',  4200.00,  600, 1),
('Yogur natural 1L',
 (SELECT id_categoria FROM miscompras.categorias WHERE descripcion='Lácteos'),
 '7701234567004',  6000.00,  400, 1),
('Pan campesino',
 (SELECT id_categoria FROM miscompras.categorias WHERE descripcion='Panadería'),
 '7701234567005',  3500.00,  320, 1),
('Croissant mantequilla',
 (SELECT id_categoria FROM miscompras.categorias WHERE descripcion='Panadería'),
 '7701234567006',  2500.00,  500, 1),
('Detergente líquido 1L',
 (SELECT id_categoria FROM miscompras.categorias WHERE descripcion='Aseo'),
 '7701234567007', 12000.00,  260, 1),
('Jabón en barra 3un',
 (SELECT id_categoria FROM miscompras.categorias WHERE descripcion='Aseo'),
 '7701234567008',  8000.00,  300, 1),
('Papas fritas 150g',
 (SELECT id_categoria FROM miscompras.categorias WHERE descripcion='Snacks'),
 '7701234567009',  5500.00,  700, 1),
('Maní salado 200g',
 (SELECT id_categoria FROM miscompras.categorias WHERE descripcion='Snacks'),
 '7701234567010',  7000.00,  420, 1),
('Gaseosa cola 1.5L',
 (SELECT id_categoria FROM miscompras.categorias WHERE descripcion='Bebidas'),
 '7701234567011',  6500.00,  800, 1),
('Agua sin gas 600ml',
 (SELECT id_categoria FROM miscompras.categorias WHERE descripcion='Bebidas'),
 '7701234567012',  2200.00, 1200, 1),
('Té verde botella 500ml',
 (SELECT id_categoria FROM miscompras.categorias WHERE descripcion='Bebidas'),
 '7701234567013',  3800.00,  650, 1),
('Chocolate de mesa 250g',
 (SELECT id_categoria FROM miscompras.categorias WHERE descripcion='Panadería'),
 '7701234567014',  9000.00,  240, 1),
('Mermelada fresa 300g',
 (SELECT id_categoria FROM miscompras.categorias WHERE descripcion='Panadería'),
 '7701234567015',  7500.00,  260, 1);
```

> Inserciones para la tabla `compras_productos`
```SQL
INSERT INTO miscompras.compras_productos (id_compra, id_producto, cantidad, total, estado) VALUES
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1001' AND fecha='2025-07-02 10:15:23'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Café de Colombia 500g'), 2, 56000.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1001' AND fecha='2025-07-02 10:15:23'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Leche entera 1L'), 3, 12600.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1001' AND fecha='2025-07-02 10:15:23'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Pan campesino'), 2, 7000.00, 1);

-- CC1002
INSERT INTO miscompras.compras_productos (id_compra, id_producto, cantidad, total, estado) VALUES
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1002' AND fecha='2025-07-03 18:45:10'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Gaseosa cola 1.5L'), 4, 26000.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1002' AND fecha='2025-07-03 18:45:10'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Papas fritas 150g'), 5, 27500.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1002' AND fecha='2025-07-03 18:45:10'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Maní salado 200g'), 2, 14000.00, 1);

-- CC1003
INSERT INTO miscompras.compras_productos (id_compra, id_producto, cantidad, total, estado) VALUES
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1003' AND fecha='2025-07-05 09:05:00'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Detergente líquido 1L'), 1, 12000.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1003' AND fecha='2025-07-05 09:05:00'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Jabón en barra 3un'), 1,  8000.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1003' AND fecha='2025-07-05 09:05:00'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Agua sin gas 600ml'), 6, 13200.00, 1);

-- CC1001
INSERT INTO miscompras.compras_productos (id_compra, id_producto, cantidad, total, estado) VALUES
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1001' AND fecha='2025-07-10 14:22:40'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Café Orgánico Sierra Nevada 250g'), 1, 22000.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1001' AND fecha='2025-07-10 14:22:40'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Mermelada fresa 300g'), 1,  7500.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1001' AND fecha='2025-07-10 14:22:40'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Pan campesino'), 1,  3500.00, 1);

-- CC1004
INSERT INTO miscompras.compras_productos (id_compra, id_producto, cantidad, total, estado) VALUES
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1004' AND fecha='2025-07-12 11:11:11'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Yogur natural 1L'), 2, 12000.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1004' AND fecha='2025-07-12 11:11:11'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Té verde botella 500ml'), 3, 11400.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1004' AND fecha='2025-07-12 11:11:11'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Chocolate de mesa 250g'), 1,  9000.00, 1);

-- CC1005
INSERT INTO miscompras.compras_productos (id_compra, id_producto, cantidad, total, estado) VALUES
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1005' AND fecha='2025-07-15 19:35:05'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Café de Colombia 500g'), 1, 28000.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1005' AND fecha='2025-07-15 19:35:05'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Leche entera 1L'), 4, 16800.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1005' AND fecha='2025-07-15 19:35:05'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Gaseosa cola 1.5L'), 2, 13000.00, 1);

-- CC1006
INSERT INTO miscompras.compras_productos (id_compra, id_producto, cantidad, total, estado) VALUES
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1006' AND fecha='2025-07-18 08:55:30'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Detergente líquido 1L'), 2, 24000.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1006' AND fecha='2025-07-18 08:55:30'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Jabón en barra 3un'), 2, 16000.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1006' AND fecha='2025-07-18 08:55:30'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Agua sin gas 600ml'), 12, 26400.00, 1);

-- CC1007
INSERT INTO miscompras.compras_productos (id_compra, id_producto, cantidad, total, estado) VALUES
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1007' AND fecha='2025-07-20 16:01:00'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Croissant mantequilla'), 6, 15000.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1007' AND fecha='2025-07-20 16:01:00'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Café Orgánico Sierra Nevada 250g'), 2, 44000.00, 1);

-- CC1008
INSERT INTO miscompras.compras_productos (id_compra, id_producto, cantidad, total, estado) VALUES
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1008' AND fecha='2025-07-25 12:20:45'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Papas fritas 150g'), 10, 55000.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1008' AND fecha='2025-07-25 12:20:45'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Gaseosa cola 1.5L'), 5, 32500.00, 1);

-- CC1002
INSERT INTO miscompras.compras_productos (id_compra, id_producto, cantidad, total, estado) VALUES
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1002' AND fecha='2025-08-01 17:05:12'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Leche entera 1L'), 8, 33600.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1002' AND fecha='2025-08-01 17:05:12'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Yogur natural 1L'), 4, 24000.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1002' AND fecha='2025-08-01 17:05:12'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Pan campesino'), 4, 14000.00, 1);

-- CC1003
INSERT INTO miscompras.compras_productos (id_compra, id_producto, cantidad, total, estado) VALUES
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1003' AND fecha='2025-08-02 10:40:33'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Té verde botella 500ml'), 5, 19000.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1003' AND fecha='2025-08-02 10:40:33'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Agua sin gas 600ml'), 10, 22000.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1003' AND fecha='2025-08-02 10:40:33'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Maní salado 200g'), 3, 21000.00, 1);

-- CC1004
INSERT INTO miscompras.compras_productos (id_compra, id_producto, cantidad, total, estado) VALUES
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1004' AND fecha='2025-08-05 13:50:00'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Chocolate de mesa 250g'), 2, 18000.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1004' AND fecha='2025-08-05 13:50:00'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Mermelada fresa 300g'), 2, 15000.00, 1),
((SELECT id_compra FROM miscompras.compras WHERE id_cliente='CC1004' AND fecha='2025-08-05 13:50:00'),
 (SELECT id_producto FROM miscompras.productos WHERE nombre='Café de Colombia 500g'), 1, 28000.00, 1);
```


## Consultas

1. Obtén el “Top 10” de productos por unidades vendidas y su ingreso total, usando `JOIN ... USING`, agregación con `SUM()`, agrupación con `GROUP BY`, ordenamiento descendente con `ORDER BY` y paginado con `LIMIT`.
```SQL
SELECT p.id_producto, p.nombre,
SUM(cp.cantidad) AS unidades,
SUM(cp.total) AS ingreso_total
FROM miscompras.compras_productos cp
JOIN miscompras.productos p USING(id_producto)
GROUP BY p.id_producto, p.nombre
ORDER BY unidades DESC
LIMIT 10;
```

2. Calcula el total pagado promedio por compra y la mediana aproximada usando una subconsulta agregada y la función de ventana ordenada `PERCENTILE_CONT(...) WITHIN GROUP`, además de `AVG()` y `ROUND()` para formateo.
```sql
SELECT AVG(t.total_compra) AS promedio_compra,
PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY t.total_compra) AS mediana
FROM(
SELECT c.id_compra, SUM(cp.total) as total_compra
FROM miscompras.compras c
JOIN miscompras.compras_productos cp USING (id_compra)
GROUP BY c.id_compra
) t;
```

3. Lista compras por cliente con gasto total y un ranking global de gasto empleando funciones de ventana (`RANK() OVER (ORDER BY SUM(...) DESC)`), concatenación de texto y `COUNT(DISTINCT ...)`.
```sql
SELECT cl.id, cl.nombre || ' ' || cl.apellidos as cliente,
    COUNT(DISTINCT c.id_compra) AS compras,
    SUM(cp.total) AS gasto_total,
    RANK() OVER(ORDER BY SUM(cp.total)DESC) AS ranking_gasto
FROM miscompras.clientes cl
JOIN miscompras.compras c ON cl.id = c.id_cliente
JOIN miscompras.compras_productos cp USING(id_compra)
GROUP BY cl.id, cliente
ORDER BY ranking_gasto
```

4. Calcula por día el número de compras, ticket promedio y total, usando un `CTE (WITH) o (Common Table Expression)“subconsulta con nombre”`, conversión de `fecha ::date`, y agregaciones (`COUNT, AVG, SUM`) con `ORDER BY`.
```sql
-- CURRENT_DATE, CURRENT_TIME, CURRENT_TIMESTAMP ~ NOW, AGE, EXTRACT
-- COLUMN_NAME::day, COLUMN_NAME::date, COLUMN_NAME::month
WITH t AS(
SELECT c.id_compra, c.fecha::date as dia, SUM(cp.total) as total_compra
FROM miscompras.compras c
JOIN miscompras.compras_productos cp USING(id_compra)
GROUP BY c.id_compra, c.fecha::date
)

SELECT dia,
    COUNT(*) as numero_compra,
    ROUND(AVG(total_compra), 2) as promedio,
    SUM(total_compra) as total_dia
FROM t
GROUP BY dia
ORDER BY dia
```

5. Realiza una búsqueda estilo e-commerce de productos activos y con stock cuyo nombre empiece por “caf”, usando filtros en `WHERE`, comparación numérica y búsqueda case-insensitive con `ILIKE 'caf%'`.
```sql
SELECT  p.id_producto, p.nombre, p.precio_venta, p.cantidad_stock
FROM miscompras.productos p
WHERE p.estado = 1
AND p.cantidad_stock > 0
AND p.nombre ILIKE 'caf%';
```

6. Devuelve los productos con el precio formateado como texto monetario usando concatenación `('||')` y `TO_CHAR(numeric, 'FM999G999G999D00')`, ordenando de mayor a menor precio.

El modelo `'FM999G999G999D00'` se descompone así:

- `FM`: “Fill Mode”. Quita espacios de relleno a la izquierda/derecha.
- `9`: dígito opcional. Se muestran tantos como existan; si faltan, no se rellenan con ceros.
- `0`: dígito obligatorio. Aquí `00` fuerza siempre dos decimales.
- `G`: separador de miles según la configuración regional (p. ej., `,` en `en_US`, `.` en `es_CO`).
- `D`: separador decimal según la configuración regional (p. ej., `.` en `en_US`, `,` en `es_CO`).

Ejemplos

- Con `lc_numeric = 'en_US'`: `to_char(1234567.5, 'FM999G999G999D00')` → `1,234,567.50`
- Con `lc_numeric = 'es_CO'`: `to_char(1234567.5, 'FM999G999G999D00')` → `1.234.567,50`
```sql
SELECT nombre, toMoney(precio_venta)
FROM miscompras.productos;

CREATE OR REPLACE FUNCTION miscompras.toMoney(p_numeric NUMERIC)
RETURNS VARCHAR LANGUAGE plpgsql AS
$$
DECLARE valor VARCHAR(255);
BEGIN
    SELECT CONCAT('$ ', TO_CHAR(p_numeric, 'FM999G999G999D00'))
    INTO valor;
    RETURN valor;
END;
$$
```

7. Arma el “resumen de canasta” por compra: subtotal, `IVA al 19%` y total con IVA, mediante `SUM()` y `ROUND()` sobre el total por ítem, agrupado por compra.
```SQL
SELECT id_compra as id_compra, ROUND(SUM(total)) as subtotal, ROUND(SUM(total)) * 0.19 as iva_producto, ROUND(SUM(total) + SUM(total) * 0.19) as total
    FROM miscompras.compras_productos
    GROUP BY id_compra;

```

8. Calcula la participación (%) de cada categoría en las ventas usando agregaciones por categoría y una ventana sobre el total (`SUM(SUM(total)) OVER ()`), más `ROUND()` para el porcentaje.
```SQL
SELECT 
    c.descripcion AS categoria,
    SUM(cp.total) AS total_ventas_por_categoria,
    ROUND(
        (SUM(cp.total) / SUM(SUM(cp.total)) OVER () * 100),
        2
    ) AS porcentaje_participacion
FROM miscompras.compras_productos cp
JOIN miscompras.productos p ON cp.id_producto = p.id_producto
JOIN miscompras.categorias c ON p.id_categoria = c.id_categoria
GROUP BY c.descripcion
ORDER BY porcentaje_participacion DESC;
```

9. Clasifica el nivel de stock de productos activos (`CRÍTICO/BAJO/OK`) usando `CASE` sobre el campo `cantidad_stock` y ordena por el stock ascendente.
```SQL
SELECT nombre as producto, 
CASE 
    WHEN cantidad_stock < 50 THEN 'Crítico'
    WHEN cantidad_stock < 150 THEN 'BAJO'
ELSE
    'OK'
END AS 'estado_stock'
FROM miscompras.productos;
```

10. Obtén la última compra por cliente utilizando`DISTINCT ON (id_cliente)` combinado con `ORDER BY ... fecha DESC` y una agregación del total de la compra.
```SQL
SELECT DISTINCT ON (com.id_cliente) com.fecha, com.id_cliente, cp.total AS total_compra
FROM miscompras.compras AS com
JOIN miscompras.clientes AS c ON c.id = com.id_cliente
JOIN miscompras.compras_productos AS cp ON cp.id_compra = com.id_compra
ORDER BY com.id_cliente, com.fecha DESC;
```

11. Devuelve los 2 productos más vendidos por categoría usando una subconsulta con `ROW_NUMBER() OVER (PARTITION BY ... ORDER BY SUM(...) DESC)` y luego filtrando `ROW_NUMBER` <= 2.

```SQL
SELECT 
    ranking,
    nombre_producto,
    categoria,
    total_vendido
FROM (
    SELECT 
        p.id_producto,
        p.nombre AS nombre_producto,
        c.descripcion AS categoria,
        SUM(cp.cantidad) AS total_vendido,
        ROW_NUMBER() OVER (PARTITION BY p.id_categoria ORDER BY SUM(cp.cantidad) DESC) AS ranking
    FROM miscompras.compras_productos cp
    JOIN miscompras.productos p ON cp.id_producto = p.id_producto
    JOIN miscompras.categorias c ON p.id_categoria = c.id_categoria
    WHERE cp.estado = 1 AND p.estado = 1
    GROUP BY p.id_producto, p.id_categoria, p.nombre, c.descripcion
) ranked_products
WHERE ranking <= 2
ORDER BY categoria, ranking;
```

12. Calcula ventas mensuales: agrupa por mes truncando la fecha con `DATE_TRUNC('month', fecha)`, cuenta compras distintas (`COUNT(DISTINCT ...)`) y suma ventas, ordenando cronológicamente.

```SQL
SELECT COUNT(DISTINCT c.id_compra) as compras_distintas, DATE_TRUNC('month', c.fecha) as mes, SUM(cp.total) AS total_ventas
    FROM miscompras.compras c
    JOIN miscompras.compras_productos cp USING(id_compra)
    GROUP BY mes
    ORDER BY mes ASC;
```

13. Lista productos que nunca se han vendido mediante un anti-join con `NOT EXISTS`, comparando por id_producto.

`WHERE  NOT EXISTS (
  SELECT *
  FROM   ..
  WHERE  ..
);`
```SQL
SELECT p.nombre
FROM miscompras.productos p
WHERE NOT EXISTS (
    SELECT 1
    FROM miscompras.compras_productos cp
    WHERE cp.id_producto = p.id_producto
);g
```

14. Identifica clientes que, al comprar “café”, también compran “pan” en la misma compra, usando un filtro con `ILIKE` y una subconsulta correlacionada con `EXISTS`.

`WHERE ...  EXISTS (
  SELECT *
  FROM   ..
  WHERE  ..
);`

```SQL
SELECT DISTINCT c.id, c.nombre, c.apellidos
FROM miscompras.clientes c
JOIN miscompras.compras co 
  ON c.id = co.id_cliente
JOIN miscompras.compras_productos cp 
  ON co.id_compra = cp.id_compra
JOIN miscompras.productos p 
  ON cp.id_producto = p.id_producto
WHERE p.nombre ILIKE '%café%'
  AND EXISTS (
      SELECT 1
      FROM miscompras.compras_productos cp2
      JOIN miscompras.productos p2 
        ON cp2.id_producto = p2.id_producto
      WHERE cp2.id_compra = co.id_compra
        AND p2.nombre ILIKE '%pan%'
  );
```

15. Estima el margen porcentual “simulado” de un producto aplicando operadores aritméticos sobre precio_venta y formateo con `ROUND()` a un decimal.

```SQL
SELECT 
    p.nombre,
    p.precio_venta,
    ROUND(((p.precio_venta - (p.precio_venta * 0.7)) / p.precio_venta) * 100, 1) AS margen_porcentual
FROM miscompras.productos p;
```

16. Filtra clientes de un dominio dado usando expresiones regulares con el operador `~*` (case-insensitive) y limpieza con `TRIM()` sobre el correo electrónico.

```SQL
SELECT id, nombre, apellidos, correo_electronico
FROM miscompras.clientes
WHERE TRIM(correo_electronico) ~* '@example\.com$';
```

17. Normaliza nombres y apellidos de clientes con `TRIM()` e `INITCAP()` para capitalizar, retornando columnas formateadas.

```SQL
SELECT 
    id,
    INITCAP(TRIM(nombre))   AS nombre_normalizado,
    INITCAP(TRIM(apellidos)) AS apellidos_normalizados,
    TRIM(correo_electronico) AS correo_limpio
FROM miscompras.clientes;
```

18. Selecciona los productos cuyo `id_producto` es par usando el operador módulo `%` en la cláusula `WHERE`.

```SQL
SELECT id_producto, nombre, precio_venta
FROM miscompras.productos
WHERE id_producto % 2 = 0;
```

19. Crea una vista ventas_por_compra que consolide `id_compra`,` id_cliente`, `fecha` y el `SUM(total)` por compra, usando `CREATE OR REPLACE VIEW` y `JOIN ... USING`.
```SQL
CREATE OR REPLACE VIEW miscompras.ventas_por_compra AS
SELECT 
    com.id_compra,
    com.id_cliente,
    com.fecha,
    SUM(cp.total) AS total_compra
FROM miscompras.compras com
JOIN miscompras.compras_productos cp 
    USING (id_compra)
GROUP BY com.id_compra, com.id_cliente, com.fecha;
```

20. Crea una vista materializada mensual mv_ventas_mensuales que agregue ventas por `DATE_TRUNC('month', fecha);` recuerda refrescarla con `REFRESH MATERIALIZED VIEW` cuando corresponda.
```SQL
CREATE MATERIALIZED VIEW miscompras.mv_ventas_mensuales AS
SELECT 
    DATE_TRUNC('month', com.fecha) AS mes,
    SUM(cp.total) AS total_mensual
FROM miscompras.compras com
JOIN miscompras.compras_productos cp 
    USING (id_compra)
GROUP BY DATE_TRUNC('month', com.fecha)
ORDER BY mes;

REFRESH MATERIALIZED VIEW miscompras.mv_ventas_mensuales;
```
21. Realiza un “UPSERT” de un producto referenciado por codigo_barras usando `INSERT ... ON CONFLICT (...) DO UPDATE`, actualizando nombre y precio_venta cuando exista conflicto.
```SQL
INSERT INTO miscompras.productos (nombre, id_categoria, codigo_barras, precio_venta, cantidad_stock, estado)
VALUES (
    'Café de Colombia Premium 500g',
    (SELECT id_categoria FROM miscompras.categorias WHERE descripcion = 'Café'),
    '7701234567001',
    30000.00,
    250,
    1
)
ON CONFLICT (codigo_barras)
DO UPDATE SET
    nombre = EXCLUDED.nombre,
    precio_venta = EXCLUDED.precio_venta
WHERE miscompras.productos.codigo_barras = EXCLUDED.codigo_barras
RETURNING id_producto, nombre, precio_venta, codigo_barras;
```

22. Recalcula el stock descontando lo vendido a partir de un `UPDATE ... FROM (SELECT ... GROUP BY ...)`, empleando `COALESCE()` y `GREATEST()` para evitar negativos.
```SQL
UPDATE miscompras.productos p
SET cantidad_stock = GREATEST(p.cantidad_stock - COALESCE((
    SELECT SUM(cp.cantidad)
    FROM miscompras.compras_productos cp
    WHERE cp.id_producto = p.id_producto
    GROUP BY cp.id_producto
), 0), 0)
WHERE EXISTS (
    SELECT 1
    FROM miscompras.compras_productos cp
    WHERE cp.id_producto = p.id_producto
);
```

23. Implementa una función PL/pgSQL (`miscompras.fn_total_compra`) que reciba `p_id_compra` y retorne el `total` con `COALESCE(SUM(...), 0);` define el tipo de retorno `NUMERIC(16,2)`.
```SQL
CREATE OR REPLACE FUNCTION miscompras.fn_total_compra(p_id_compra INT)
RETURNS NUMERIC(16,2) LANGUAGE plpgsql AS $$ 
BEGIN
    RETURN (
        SELECT COALESCE(SUM(total), 0)
        FROM miscompras.compras_productos
        WHERE id_compra = p_id_compra
    );
END;
$$;

SELECT miscompras.fn_total_compra(1);
```

24. Define un trigger `AFTER INSERT` sobre `compras_productos` que descuente stock mediante una función `RETURNS TRIGGER` y el uso del registro `NEW`, protegiendo con `GREATEST()` para no quedar bajo cero.
```SQL
UPDATE miscompras.productos p
SET cantidad_stock = GREATEST(p.cantidad_stock - COALESCE((
    SELECT SUM(cp.cantidad)
    FROM miscompras.compras_productos cp
    WHERE cp.id_producto = p.id_producto
    GROUP BY cp.id_producto
), 0), 0)
WHERE EXISTS (
    SELECT 1
    FROM miscompras.compras_productos cp
    WHERE cp.id_producto = p.id_producto
);

CREATE TRIGGER trg_descuento_stock
AFTER INSERT ON miscompras.compras_productos
FOR EACH ROW
EXECUTE FUNCTION miscompras.fn_descuento_stock();
```

25. Asigna la “posición por precio” de cada producto dentro de su categoría con `DENSE_RANK() OVER (PARTITION BY ... ORDER BY precio_venta DESC)` y presenta el ranking.
```SQL
SELECT 
    DENSE_RANK() OVER (PARTITION BY p.id_categoria ORDER BY p.precio_venta DESC) AS posicion,
    p.nombre AS nombre_producto,
    c.descripcion AS categoria,
    p.precio_venta,
    p.codigo_barras
FROM miscompras.productos p
JOIN miscompras.categorias c ON p.id_categoria = c.id_categoria
WHERE p.estado = 1
ORDER BY p.id_categoria, posicion;
```

26. Para cada cliente, muestra su gasto por compra, el gasto anterior y el delta entre compras usando `LAG(...) OVER (PARTITION BY id_cliente ORDER BY dia)` dentro de un `CTE` que agrega por día.
```SQL
WITH GastosPorDia AS (
    SELECT 
        c.id_cliente,
        DATE(c.fecha) AS dia,
        SUM(miscompras.fn_total_compra(c.id_compra)) AS total_diario
    FROM miscompras.compras c
    WHERE c.estado = 'A'
    GROUP BY c.id_cliente, DATE(c.fecha)
)
SELECT 
    gpd.id_cliente,
    cl.nombre || ' ' || cl.apellidos AS nombre_completo,
    gpd.dia,
    gpd.total_diario AS gasto_dia,
    LAG(gpd.total_diario) OVER (PARTITION BY gpd.id_cliente ORDER BY gpd.dia) AS gasto_anterior,
    gpd.total_diario - LAG(gpd.total_diario) OVER (PARTITION BY gpd.id_cliente ORDER BY gpd.dia) AS delta
FROM GastosPorDia gpd
JOIN miscompras.clientes cl ON gpd.id_cliente = cl.id
ORDER BY gpd.id_cliente, gpd.dia;
```