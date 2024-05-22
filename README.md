# Implementación y optimización de consultas a base de datos

## 1. Optimización de consultas SQL

1. Consulta de pedidos: Recuperar los pedidos con muchos articulos y calcular el precio total

```
SELECT Orders.OrderID, SUM(OrderDetails.Quantity * OrderDetails.UnitPrice) AS TotalPrice
FROM Orders
JOIN OrderDetails ON Orders.OrderID = OrderDetails.OrderID
WHERE OrderDetails.Quantity > 10
GROUP BY Orders.OrderID;
```

Análisis:
- Debido a que en la consulta se recupera información de la tabla OrderDetails, se puede prescindir del join  con Order para mejorar el performance 
- Podriamos mejorar el tiempo de respuesta en las consultas agregando indices sobre los campos Quantity y OrderID que son los que se utilizan como criterios de filtrado y agrupamiento en la nueva consulta
- Cabe mencionar que al aplicar indices sobre los campos mencionados en el punto anterior,  puede ocurrir que los tiempos de insercion y actualizacion aumenten
- En caso de que el número de registros sea muy grande podemos aplicar paginado a las consulta Ejemplo: regresar 1000 registros empezando por la pagina u offset: 0

Solución:

```
CREATE INDEX idx_order ON OrderDetails(OrderID);
CREATE INDEX idx_quantity ON OrderDetails(Quantity);

SELECT OrderID, SUM(Quantity * UnitPrice) AS TotalPrice
FROM OrderDetails 
WHERE Quantity > 10
GROUP BY OrderID LIMIT 1000 OFFSET 0;
```

2.Consulta de clientes : Buscar todos los clientes de londres y ordenar por nombre de cliente

```
SELECT CustomerName FROM Customers WHERE City = 'London' ORDER BY CustomerName;
```

Análisis:

- En caso de que se desee obtener constantemente informacion tomando en cuenta los criterios de City y CustomerName podemos aplicar indexado sobre esos campos    
- En caso de que el número de registros sea muy grande podemos aplicar paginado a las consulta Ejemplo: regresar 1000 registros empezando por la pagina u offset: 0

Solución:
```
-- se crea el indice para el campo City dentro de la Customers 
CREATE INDEX idx_city ON Customers(City);

-- se crea el indice para el campo nombre de CustomerName  dentro de la Customers
CREATE INDEX idx_name ON Customers(CustomerName);

SELECT CustomerName FROM Customers WHERE City = 'London' ORDER BY CustomerName LIMIT 1000  OFFSET 0;
```

Solución alterna:
Como solucion alterna podriamos normalizar para separar el campo city en su propia tabla y asi evitar repetir informacion de ciudades, la solucion seria la siguiente:

```
-- se crea la nueva tabla de ciudades
CREATE TABLE Cities (
    CityID int NOT NULL,
    CityName varchar(255),
    PRIMARY KEY (CityID)
);

-- se modifica la tabla de clientes pra contar con la referencia al id de ciudad

ALTER TABLE Customers
ADD CityID int,
ADD FOREIGN KEY (CityID) REFERENCES Cities(CityID);

-- La nueva consulta contará con la referencia al campo de nombre de ciudad

SELECT CustomerName 
FROM Customers 
JOIN Cities ON Customers.CityID = Cities.CityID
WHERE Cities.CityName = 'London' 
ORDER BY CustomerName 
LIMIT 1000 OFFSET 0;

-- Se ajustaran los indices de los campos actualizados para que concuerden con las tablas/campos

-- Se crea un indice para el nombre del cliente
CREATE INDEX idx_customers_customer_name ON Customers(CustomerName);

-- Se crea el indice para el id de ciudad
CREATE INDEX idx_customers_city_id ON Customers(CityID);

-- Se crea el indice para el nombre de la ciudad
CREATE INDEX idx_cities_city_name ON Cities(CityName);

```

2. Implementacion de consultas NoSQL
    Los participantes optimizaran las consultas NoSQL provistas enfocandoce en recuperacion eficiente de datos y latencia minima


## 2. Consultas NoSQL para revisión:

1. Consulta de los post de usuarios: Recuperar los post activos mas populares y desplegar su titulo y conteo de me gusta

```
db.posts
  .find({ status: "active" }, { title: 1, likes: 1 })
  .sort({ likes: -1 });
```
Análisis:
- con el fin de mejorar la consulta se sugiere agregar indices a la tabla de post
- se puede mejorar el performance  de la consulta validando campos nulos con el operador exists de este modo reducimos la cantidad de elementos recuperados y mejoramos el tiempo

Solución:
```
db.posts.createIndex({ status: 1, likes: -1 });
db.posts.find({ status: { $exists: true, $eq: "active" } }, { title: 1, likes: 1 }).sort({ likes: -1 });
```

2. Consulta de agregado de datos de usuario: resuma la cantidad de usuarios activos por ubicación.

```
  db.users.aggregate([
  { $match: { status: "active" } },
  { $group: { _id: "$location", totalUsers: { $sum: 1 } } },
]);
```

Análisis:
- Con el fin de mejorar la consulta se sugiere agregar indices a la tabla de user sobre los campos de status y location

Solución:
```
db.users.createIndex({ status: 1, location: 1 });
db.users.aggregate([
  { $match: { status: "active" } },
  { $group: { _id: "$location", totalUsers: { $sum: 1 } } },
]);
```
