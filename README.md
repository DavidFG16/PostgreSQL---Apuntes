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
    sexo_camper sexo NOT NULL
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



### Taller de Constraints
> Definir los Constraints 
- Primary Key
- Foreign Key
- NOT NULL
- DEFAULT() 
> Mediante ALTER TABLE 

```sql
CREATE TABLE country (
    id serial,
    name varchar(50)
);

CREATE TABLE region (
    id serial,
    name varchar(50),
    idcountry integer
);

CREATE TABLE city (
    id serial,
    name varchar(50),
    idregion integer
);

ALTER TABLE country ADD CONSTRAINT pk_country PRIMARY KEY(id);
ALTER TABLE city ALTER COLUMN idregion SET NOT NULL;


ALTER TABLE region ADD CONSTRAINT pk_region PRIMARY KEY(id);
ALTER TABLE region ADD CONSTRAINT fk_country FOREIGN KEY(idcountry) REFERENCES country(id);
ALTER TABLE region ALTER COLUMN idcountry SET NOT NULL;



ALTER TABLE city ADD CONSTRAINT pk_city PRIMARY KEY(id);
ALTER TABLE city ADD CONSTRAINT fk_region FOREIGN KEY(idregion) REFERENCES region(id);
ALTER TABLE city ALTER COLUMN idregion SET NOT NULL;









```
