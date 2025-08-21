# Taller Diseño de base de datos - Postgres

![](https://i.ibb.co/0brN66k/image.png)

Teniendo en cuenta el diagrama entidad relación resuelva los siguientes requerimientos:

1. Genere los comandos DDL que permitan la creación de la base de datos del DER suministrado.
2. Genere los comandos DML que permitan insertar datos en la base de datos creada en el paso 1.

## Resolución del taller

### 1. Comandos DDL

```SQL
CREATE DATABASE miscompras;

CREATE TABLE compras_productos(
    id_compra serial,
    id_producto serial,
    cantidad integer NOT NULL,
    total numeric(10,2) NOT NULL,
    estado smallint NOT NULL,
    PRIMARY KEY (id_compra, id_producto),
    CONSTRAINT fk_compra FOREIGN KEY (id_compra) REFERENCES compras(id_compra),
    CONSTRAINT fk_producto FOREIGN KEY (id_producto) REFERENCES productos(id_producto)
);

CREATE TABLE compras(
    id_compra serial PRIMARY KEY,
    id_cliente varchar(20) NOT NULL,
    fecha timestamp without time zone,  
    medio_pago char(1) NOT NULL,
    comentario varchar(300),
    estado char(1) NOT NULL,
    CONSTRAINT fk_cliente FOREIGN KEY (id_cliente) REFERENCES clientes(id)
);

CREATE TABLE productos(
    id_producto integer PRIMARY KEY,
    nombre  varchar(45),
    id_categoria integer,
    codigo_barras varchar(150),
    precio_venta numeric(10,2),
    cantidad_stock integer,
    estado smallint,
    CONSTRAINT fk_categoria FOREIGN KEY (id_categoria) REFERENCES categorias(id_categoria)
);

CREATE TABLE clientes(
    id varchar(20) PRIMARY KEY,
    nombre varchar(40),
    apellidos varchar(100),
    celular numeric(10,0),
    direccion varchar(80),
    correo_electronico varchar(70)
);

CREATE TABLE categorias(
    id_categoria serial PRIMARY KEY,
    descripcion varchar(45),
    estado smallint
);
```

## 2. Comandos DML

> Inserciones para la tabla `clientes`
```SQL
INSERT INTO clientes (id, nombre, apellidos, celular, direccion, correo_electronico) VALUES
('CC1001', 'Juan', 'Pérez López', 3001234567, 'Cra 12 #34-56', 'juan.perez@example.com'),
('CC1002', 'María', 'Gómez Torres', 3109876543, 'Calle 45 #67-89', 'maria.gomez@example.com'),
('CC1003', 'Carlos', 'Ramírez Díaz', 3205556677, 'Av. Siempre Viva 742', 'carlos.ramirez@example.com');
```

> Inserciones para la tabla `categorias`
```SQL
INSERT INTO categorias (id_categoria, descripcion, estado) VALUES
(1, 'Bebidas', 1),
(2, 'Snacks', 1),
(3, 'Lácteos', 1),
(4, 'Aseo Personal', 1),
(5, 'mercado', 1);
```

> Inserciones para la tabla `compras`
```SQL
INSERT INTO compras (id_compra, id_cliente, fecha, medio_pago, comentario, estado) VALUES
(1, 'CC1001', '2025-08-15 10:30:00', 'E', 'Compra de productos para la casa', 'A'),
(2, 'CC1002', '2025-08-15 11:15:00', 'T', 'Pago con tarjeta débito', 'A'),
(3, 'CC1003', '2025-08-15 12:00:00', 'E', 'Compra rápida', 'A');
```

> Inserciones para la tabla `productos`
```SQL
INSERT INTO productos (id_producto, nombre, id_categoria, codigo_barras, precio_venta, cantidad_stock, estado) VALUES
(101, 'Coca-Cola 1.5L', 1, '7701234567890', 4500.00, 50, 1),
(102, 'Galletas Oreo 6p', 2, '7702234567891', 2500.00, 100, 1),
(103, 'Leche Entera 1L', 3, '7703234567892', 3200.00, 30, 1),
(104, 'Shampoo Sedal 350ml', 4, '7704234567893', 8500.00, 20, 1),
(105, 'Arroz Diana 1Kg', 5, '7705234567894', 3800.00, 80, 1);
```

> Inserciones para la tabla `compras_productos`
```SQL
INSERT INTO compras_productos (id_compra, id_producto, cantidad, total, estado) VALUES
(1, 101, 2, 9000.00, 1),
(1, 105, 1, 3800.00, 1), 
(2, 102, 3, 7500.00, 1),
(2, 103, 2, 6400.00, 1), 
(3, 104, 1, 8500.00, 1); 
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
);
```

14. Identifica clientes que, al comprar “café”, también compran “pan” en la misma compra, usando un filtro con `ILIKE` y una subconsulta correlacionada con `EXISTS`.

`WHERE ...  EXISTS (
  SELECT *
  FROM   ..
  WHERE  ..
);`

```SQL

```

15. Estima el margen porcentual “simulado” de un producto aplicando operadores aritméticos sobre precio_venta y formateo con `ROUND()` a un decimal.

```SQL
SELECT 
    p.nombre,
    p.precio_venta,
    ROUND(((p.precio_venta - (p.precio_venta * 0.7)) / p.precio_venta) * 100, 1) AS margen_porcentual
FROM miscompras.productos p;
```