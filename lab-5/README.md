
# Lab 5: SQL

## Ejercicio 1
``` sql
SELECT Orders.OrderID, SUM(OrderDetails.Quantity * OrderDetails.UnitPrice) AS TotalPrice
FROM Orders
JOIN OrderDetails ON Orders.OrderID = OrderDetails.OrderID
WHERE OrderDetails.Quantity > 10
GROUP BY Orders.OrderID;
```
### Código optimizado

- Este siguiente indice permite que el motor de la base de datos acceda directamente a las filas que cumplen con el filtro Quantity > 10 y estén listas para la operación de unión.

``` sql
CREATE INDEX idx_orderdetails_orderid_quantity ON OrderDetails (OrderID, Quantity);
```

- En este caso el siguiente indice puede reducir el costo de calcular Quantity * UnitPrice.

``` sql
CREATE INDEX idx_orderdetails_orderid_unitprice ON OrderDetails (OrderID, UnitPrice);
```
- El siguiente indice ayuda en la en la union de la tabla Orders.

``` sql
CREATE INDEX idx_orders_orderid ON Orders (OrderID);
```

- Se crea una vista materializada para reducir el costo de la operación de JOIN y GROUP BY.

``` sql
CREATE MATERIALIZED VIEW OrderDetailsTotalPrice AS
SELECT OrderID, Quantity * UnitPrice AS TotalPrice
FROM OrderDetails
WHERE Quantity > 10;
```

``` sql
SELECT o.OrderID, SUM(v.TotalPrice) AS TotalPrice
FROM Orders o
JOIN OrderDetailsTotalPrice v ON o.OrderID = v.OrderID
GROUP BY o.OrderID;
```
## Ejercicio 2

``` sql
SELECT CustomerName FROM Customers WHERE City = 'London' ORDER BY CustomerName;
```
### Código optimizado

- Se crea un indicen la columna city para permitir al motor de base de datos encontrar de manera mas rapida los registros de City.

``` sql
CREATE INDEX idx_customers_city ON Customers (City);
```
- Se crea un indice compuestro entre City y CustomerName para mejorar la busqueda.

``` sql
CREATE INDEX idx_customers_city_customername ON Customers (City, CustomerName);
```

- En la consulta original se puede agregar un LIMIT si no se necitan todos los registros.

``` sql
SELECT CustomerName 
FROM Customers 
WHERE City = 'London' 
ORDER BY CustomerName 
LIMIT 10;
```

# Ejercicios NoSQL

## Ejercicio 3

``` mongodb-json
db.posts
  .find({ status: "active" }, { title: 1, likes: 1 })
  .sort({ likes: -1 });
```

### Código optimizado

- Se crea indice cimpuesto entre status y likes para mejorar la busqueda.

``` mongodb-json
db.posts.createIndex({ status: 1, likes: -1 });
```

La consulta es muy simple pero sse puede agregar un explain para que muestre que efectivamente se estan utilizando el indice creado.

``` mongodb-json
db.posts
  .find({ status: "active" }, { title: 1, likes: 1 })
  .sort({ likes: -1 })
  .explain("executionStats");
```

## Ejercicio 4

``` mongodb-json
db.users.aggregate([
  { $match: { status: "active" } },
  { $group: { _id: "$location", totalUsers: { $sum: 1 } } },
]);
```

### Código optimizado

- Se crea un indice de la columna status para mejorar la busqueda.

``` mongodb-json
db.users.createIndex({ status: 1 });
```

- Se crea un indice compuesto entre status y location para mejorar la busqueda.

``` mongodb-json
db.users.createIndex({ status: 1, location: 1 });
```
- La consulta creada es lo bastanse timple y optimizada gracias al indice.
- 
```mongodb-json
db.users.aggregate([
  { $match: { status: "active" } },
  { $group: { _id: "$location", totalUsers: { $sum: 1 } } }
]);
```


