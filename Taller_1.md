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