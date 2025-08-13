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
