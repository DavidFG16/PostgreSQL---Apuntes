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