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