# PostgreSQL con Docker

## Creación del Contenedor
```bash
docker run -d --name postgres_container -e  POSTGRES_USER=admin -e POSTGRES_PASSWORD=admin -e POSTGRES_DB=campus -p 5433:5432 -v pgdata:/var/lib/postgresql/data --restart=unless-stopped postgres:15
```

## Conectar al Contenedor de Docker
```bash
docker exec -it postgres_container bash
```

## Conectar con PostgreSQL bajo Consola
```bash
psql --host=localhost --username=admin -d campus -password
```

### Versión Cortilla
```bash
psql -h localhost -U admin -d campus -W
```
- **-W:** Siempre sale la petición de pedir contraseña (W Mayúscula)
- **-w:** Nunca sale la petición de pedir contraseña (w Minúscula)


### Para usuario por defecto
```bash
psql -host=localhost --username=admin
```


# Comandos SQL

- `\l`: Lista las bases de datos
- `CREATE DATABASE <nombre>`: Crear Base de Datos
- `\c {db_name}`: Cambiar a una base de datos existent
- `\d`: Describe las tablas de la base de datos actual
- `\d {table_name}`: Describe la tabla
- `\dt`: Listado de las tablas de la base de datos
- `\ds`: Secuencias, que se crean con el tipo de datos `serial`
- `\di`: Listar los índices
- `\dp \z`: Listado de privilegios de las tablas

## Tipos de datos númericos

| Tipo de dato        | Tamaño de almacenamiento | Rango de valores                                                      | Descripción                                                                 |
|---------------------|--------------------------|------------------------------------------------------------------------|----------------------------------------------------------------------------|
| `smallint`          | 2 bytes                  | -32,768 a 32,767                                                       | Entero pequeño, ahorra espacio cuando no se requieren valores grandes.     |
| `integer`           | 4 bytes                  | -2,147,483,648 a 2,147,483,647                                         | Entero estándar de uso general.                                            |
| `bigint`            | 8 bytes                  | -9,223,372,036,854,775,808 a 9,223,372,036,854,775,807                 | Entero grande para valores muy elevados.                                   |
| `decimal`           | Variable                 | Depende de la precisión especificada                                   | Número exacto con precisión definida por el usuario, ideal para finanzas.  |
| `numeric`           | Variable                 | Depende de la precisión especificada                                   | Número exacto de precisión arbitraria, equivalente a `decimal`.            |
| `real`              | 4 bytes                  | Precisión de 6 dígitos decimales                                       | Número en coma flotante de precisión simple.                               |
| `double precision`  | 8 bytes                  | Precisión de 15 dígitos decimales                                      | Número en coma flotante de alta precisión.                                 |
| `serial`            | 4 bytes                  | 1 a 2,147,483,647                                                      | Entero autoincremental para identificadores únicos.                        |
| `bigserial`         | 8 bytes                  | 1 a 9,223,372,036,854,775,807                                          | Entero autoincremental grande para identificadores únicos muy grandes.     |

### Enumeradores 

```pgsql
CREATE TYPE sexo AS ENUM('Masculino', 'Femenino', 'Otro');

CREATE TABLE camper(
    name varchar(100) NOT NULL,
    sexo_camper sexo NOT NULLV
);
```

## Ejemplo general de Tipos de Datos
```SQL
    CREATE TABLE ejemplo(
    id serial PRIMARY KEY,
    nombre varchar(100) NOT NULL,
    descripcion text NULL,
    precio numeric(10,2) NOT NULL,
    en_stock boolean NOT NULL,
    fecha_creacion date NOT NULL,
    hora_creacion time NOT NULL,
    fecha_hora timestamp NOT NULL,
    fecha_hora_zona timestamp with time zone,
    duracion interval,
    direccion_ip inet,
    direccion_mac macaddr,
    punto_geometrico point,
    datos_json json,
    datos_jsonb jsonb,
    identificador_unico uuid,
    cantidad_monetaria money,
    rangos int4range,
    colores_preferidos varchar(20)[]
    );
```

### Insert para `ejemplo`

```SQL
    INSERT INTO ejemplo (nombre, descripcion, precio, en_stock, fecha_creacion, 
    hora_creacion, fecha_hora, fecha_hora_zona, duracion, direccion_ip, 
    direccion_mac, punto_geometrico, datos_json, datos_jsonb, identificador_unico, 
    cantidad_monetaria, rangos, colores_preferidos)
    VALUES (
    'Ejemplo A', 'Lorem ipsum......', 9990.99, true, '2025-07-10',
    '20:30:10', '2025-07-10 20:30:10', '2025-07-10 20:30:10-05', '1 day',
    '192.168.0.1','08:00:27:00:00:00','(10, 20)', '{"key":"value"}',
    '{"key":"value"}','b8ac502c-7049-4ae5-aa7e-642ad77ca4f1', '100.00', 
    '[10, 20)', ARRAY ['rojo', 'verde', 'azul', 'otro']
);
```

## Definicion de Constraints (Restricciones)
### Tabla de Ejemplo

```SQL
CREATE TABLE empleados(
    id serial,
    nombre varchar(100) NOT NULL,
    edad integer NOT NULL,
    salario numeric(10,2) NOT NULL,
    fecha_contrato date,
    vigente boolean DEFAULT true
);

CREATE TABLE departamentos(
    id serial,
    nombre varchar(100) NOT NULL,
    vigente boolean DEFAULT true,
    PRIMARY KEY(id)
);

ALTER TABLE empleados ADD COLUMN departamento_id integer NOT NULL;
```

## Constraints a Tablas existentes
### Primary Key
```SQL
    ALTER TABLE empleados ADD PRIMARY KEY(id);
```

### Foreign Key
```SQL
ALTER TABLE empleados ADD CONSTRAINT fk_departamento FOREIGN KEY(departamento_id) REFERENCES departamentos(id);
```

### Unique
> Agregar la restriccion de UNIQUE a `nombre`
```SQL
ALTER TABLE empleados ADD CONSTRAINT nombre_unique UNIQUE(nombre); 
```
### Check
> Agregar restriccion de CHECK para validar que la `edad` del empleado sea >=18
```SQL
ALTER TABLE empleados ADD CONSTRAINT check_edad CHECK(edad>=18);
```
### Default
> Agregar un DEFAULT para `salario` de 400. 
```SQL
ALTER TABLE empleados ALTER COLUMN salario SET DEFAULT 400.0;
``` 

## ESQUEMAS (ESCHEMAS)

Es como tener varias bases de datos dentro de 1 sola, para verlp de un modo más sencillo, es como tener carpetas dentro de una base de datos.
Normalmente se usan los esquemas para proyectos robustos

```sql
CREATE SCHEMA IF NOT EXISTS miscompras;
SET search_path TO miscompras; -- Establece la ruta al esquema creado
```

## Funciones y operadores

1. Top 10 productos más vendidos y su ingreso total 
- `SUM()`
- `USING()`
```sql
SELECT p.id_producto, p.nombre,
SUM(cp.cantidad) AS unidades,
SUM(cp.total) AS ingreso_total
FROM miscompras.compras_productos cp
JOIN miscompras.productos p USING(id_producto)
GROUP BY p.id_producto, p.nombre
ORDER BY unidades DESC
LIMIT 10;
```

2. Venta promedio por compra y mediana aproximada
- `PERCENTILE_CONT(..) WITHIN GROUP (ORDER BY)`
- `ROUND`
- `USING()`

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
3. Compras por cliente y ranking
-  `COUNT`
-  `SUM`
-  `RANK() OVER(ORDER BY... ASC/DESC) ASC/DESC`

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

4. Ticket por compra 
- `COUNT`
- `ROUND`
- `SUM`
- `WITH args AS`

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
5. Búsqueda "tipo-ecommerce": productos activos, disponibles y que empiezan por 'Caf'V
- `ILIKE`
```sql
SELECT  p.id_producto, p.nombre, p.precio_venta, p.cantidad_stock
FROM miscompras.productos p
WHERE p.estado = 1
AND p.cantidad_stock > 0
AND p.nombre ILIKE 'caf%';
```

23. Función. total de una compra (retorna NUMERIC)
-`COALLESCE`
-`SUM`

```sql
CREATE OR REPLACE FUNCTION miscompras.fn_total_compra(p_id_compra INT)
RETURNS NUMERIC LANGUAGE plpgsql AS $$
DECLARE v_total NUMERIC(16,2);
BEGIN
    SELECT COALESCE(SUM(total), 0)
    INTO v_total
    FROM miscompras.compras_productos
    WHERE id_compra = p_id_compra
    
    RETURN v_total;
END;
$$;

SELECT miscompras.fn_total_compra(1) AS total_compra_1 -- llama a la función
```