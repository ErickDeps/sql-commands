# Chuleta de SQL para MySQL

Guía práctica de SQL para **MySQL 8.0 o superior**. Está organizada por acciones e incluye explicaciones breves y ejemplos. Los nombres entre `< >` son marcadores que debes sustituir; no escribas los signos `< >` en las consultas.

> **Precaución:** `UPDATE` o `DELETE` sin `WHERE` afecta a todas las filas de la tabla. Comprueba siempre la consulta antes de ejecutarla, especialmente en producción.

## Esquema de ejemplo

Los ejemplos de esta guía suponen estas tablas:

```sql
CREATE DATABASE tienda;
USE tienda;

CREATE TABLE clientes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE,
    ciudad VARCHAR(80),
    creado_en DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE productos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(120) NOT NULL,
    precio DECIMAL(10, 2) NOT NULL,
    stock INT NOT NULL DEFAULT 0
);

CREATE TABLE productos_archivo (
    id INT PRIMARY KEY,
    nombre VARCHAR(120) NOT NULL
);

CREATE TABLE pedidos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente_id INT NOT NULL,
    fecha DATETIME DEFAULT CURRENT_TIMESTAMP,
    estado VARCHAR(30) DEFAULT 'pendiente',
    total DECIMAL(10, 2) NOT NULL DEFAULT 0,
    FOREIGN KEY (cliente_id) REFERENCES clientes(id)
);

CREATE TABLE detalle_pedido (
    pedido_id INT NOT NULL,
    producto_id INT NOT NULL,
    cantidad INT NOT NULL,
    precio_unitario DECIMAL(10, 2) NOT NULL,
    PRIMARY KEY (pedido_id, producto_id),
    FOREIGN KEY (pedido_id) REFERENCES pedidos(id),
    FOREIGN KEY (producto_id) REFERENCES productos(id)
);
```

## 1. Conectarse y consultar la estructura

| Acción | Explicación y ejemplo |
|---|---|
| Conectar desde la terminal | Abre una sesión interactiva. `mysql -u usuario -p` |
| Conectar a una base de datos | Selecciona una base al conectarte. `mysql -u usuario -p tienda` |
| Mostrar bases de datos | Lista las bases accesibles. `SHOW DATABASES;` |
| Seleccionar una base | Establece la base activa para las siguientes consultas. `USE tienda;` |
| Mostrar tablas | Lista las tablas de la base activa. `SHOW TABLES;` |
| Describir una tabla | Muestra columnas, tipos, claves y valores predeterminados. `DESCRIBE clientes;` |
| Ver la definición de una tabla | Devuelve el SQL con el que se creó. `SHOW CREATE TABLE clientes;` |
| Mostrar columnas | Consulta los nombres y tipos de las columnas. `SHOW COLUMNS FROM productos;` |
| Obtener la versión | Muestra la versión del servidor. `SELECT VERSION();` |

## 2. Crear y administrar bases de datos

| Acción | Explicación y ejemplo |
|---|---|
| Crear una base de datos | `IF NOT EXISTS` evita el error si ya existe. `CREATE DATABASE IF NOT EXISTS tienda;` |
| Crear con codificación recomendada | Usa Unicode completo para almacenar texto internacional. `CREATE DATABASE tienda CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;` |
| Cambiar de base activa | `USE tienda;` |
| Eliminar una base de datos | Borra la base y todo su contenido; la operación es destructiva. `DROP DATABASE tienda;` |
| Cambiar opciones de codificación | Modifica la codificación predeterminada. `ALTER DATABASE tienda CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;` |

## 3. Crear y cambiar tablas

| Acción | Explicación y ejemplo |
|---|---|
| Crear una tabla | Define columnas y tipos de datos. `CREATE TABLE categorias (id INT PRIMARY KEY AUTO_INCREMENT, nombre VARCHAR(80) NOT NULL);` |
| Crear solo si no existe | No falla si la tabla ya existe. `CREATE TABLE IF NOT EXISTS notas (id INT PRIMARY KEY, texto TEXT);` |
| Crear una tabla desde una consulta | Copia las columnas y los resultados; no copia índices ni restricciones. `CREATE TABLE productos_baratos AS SELECT id, nombre, precio FROM productos WHERE precio < 20;` |
| Añadir una columna | `ALTER TABLE clientes ADD telefono VARCHAR(30);` |
| Cambiar el tipo de una columna | `MODIFY` cambia el tipo y sus atributos. `ALTER TABLE productos MODIFY nombre VARCHAR(160) NOT NULL;` |
| Renombrar una columna | `CHANGE` requiere indicar el nombre nuevo y la definición completa. `ALTER TABLE clientes CHANGE ciudad localidad VARCHAR(80);` |
| Eliminar una columna | Borra también los datos guardados en ella. `ALTER TABLE clientes DROP COLUMN telefono;` |
| Renombrar una tabla | `RENAME TABLE notas TO apuntes;` |
| Vaciar una tabla | Elimina todas las filas y reinicia normalmente el contador `AUTO_INCREMENT`. `TRUNCATE TABLE productos_baratos;` |
| Eliminar una tabla | Borra la tabla y sus datos. `DROP TABLE IF EXISTS productos_baratos;` |

### Tipos de datos frecuentes

| Tipo | Uso típico |
|---|---|
| `INT`, `BIGINT` | Números enteros; `BIGINT` permite valores mayores. |
| `DECIMAL(p, e)` | Importes y otros decimales exactos, por ejemplo `DECIMAL(10, 2)`. |
| `FLOAT`, `DOUBLE` | Aproximaciones numéricas; no son ideales para dinero. |
| `VARCHAR(n)` | Texto de longitud variable, hasta `n` caracteres. |
| `TEXT` | Texto largo. |
| `DATE` | Fecha sin hora. |
| `TIME` | Hora o duración. |
| `DATETIME`, `TIMESTAMP` | Fecha y hora; `TIMESTAMP` tiene conversiones asociadas a la zona horaria de la sesión. |
| `BOOLEAN` | Sinónimo de `TINYINT(1)` en MySQL; se suele usar `TRUE` o `FALSE`. |
| `JSON` | Documento JSON validado por MySQL. |
| `BLOB` | Datos binarios. |

## 4. Insertar datos

| Acción | Explicación y ejemplo |
|---|---|
| Insertar una fila | Indica columnas y valores en el mismo orden. `INSERT INTO clientes (nombre, email, ciudad) VALUES ('Ana Ruiz', 'ana@example.com', 'Lima');` |
| Insertar varias filas | Cada grupo de paréntesis representa una fila. `INSERT INTO productos (nombre, precio, stock) VALUES ('Teclado', 25.90, 10), ('Ratón', 12.50, 20);` |
| Omitir columnas con valor predeterminado | Las columnas omitidas reciben su valor por defecto o `NULL` si se permite. `INSERT INTO productos (nombre, precio) VALUES ('Cable USB', 5.99);` |
| Insertar resultados de una consulta | Copia filas de otra consulta; las columnas deben coincidir. `INSERT INTO productos_archivo (id, nombre) SELECT id, nombre FROM productos WHERE stock = 0;` |
| Obtener el ID autogenerado | Tras un `INSERT` en la misma conexión, `SELECT LAST_INSERT_ID();` |
| Insertar o actualizar al encontrar duplicado | Requiere una clave `PRIMARY KEY` o `UNIQUE` en conflicto. El alias de fila requiere MySQL 8.0.19 o superior. `INSERT INTO productos (id, nombre, precio, stock) VALUES (1, 'Teclado', 27.90, 12) AS nuevo ON DUPLICATE KEY UPDATE precio = nuevo.precio, stock = nuevo.stock;` |
| Ignorar una fila duplicada | `INSERT IGNORE` puede degradar ciertos errores a advertencias; úsalo solo si ese comportamiento es intencional. `INSERT IGNORE INTO clientes (nombre, email) VALUES ('Ana Ruiz', 'ana@example.com');` |

## 5. Consultar datos con `SELECT`

| Acción | Explicación y ejemplo |
|---|---|
| Consultar todas las columnas | Útil para explorar; en aplicaciones es preferible enumerar las columnas necesarias. `SELECT * FROM clientes;` |
| Elegir columnas | Devuelve solo las columnas indicadas. `SELECT nombre, email FROM clientes;` |
| Usar alias | Renombra una columna solo en el resultado. `SELECT nombre AS cliente FROM clientes;` |
| Evitar duplicados | `DISTINCT` elimina filas repetidas en el resultado. `SELECT DISTINCT ciudad FROM clientes;` |
| Limitar resultados | Devuelve hasta `n` filas. `SELECT nombre, precio FROM productos LIMIT 5;` |
| Paginar | `OFFSET` indica cuántas filas omitir; usa un orden estable para páginas consistentes. `SELECT id, nombre FROM productos ORDER BY id LIMIT 10 OFFSET 20;` |
| Consultar una expresión | Calcula un valor en cada fila. `SELECT nombre, precio * 1.18 AS precio_con_impuesto FROM productos;` |
| Consultar la fecha y hora actuales | `SELECT NOW();` |

### Filtrar filas con `WHERE`

| Acción | Explicación y ejemplo |
|---|---|
| Igualdad o desigualdad | `=`, `<>` y `!=` comparan valores. `SELECT * FROM pedidos WHERE estado <> 'cancelado';` |
| Comparar números o fechas | Operadores: `>`, `<`, `>=`, `<=`. `SELECT * FROM productos WHERE precio >= 20;` |
| Combinar condiciones | `AND` exige ambas condiciones; `OR` acepta cualquiera. `SELECT * FROM productos WHERE precio < 50 AND stock > 0;` |
| Agrupar condiciones | Usa paréntesis para dejar clara la precedencia lógica. `SELECT * FROM pedidos WHERE estado = 'pendiente' AND (total > 100 OR fecha >= '2026-01-01');` |
| Buscar entre dos valores | `BETWEEN` incluye ambos extremos. `SELECT * FROM productos WHERE precio BETWEEN 10 AND 30;` |
| Buscar entre varios valores | `IN` equivale a varias alternativas de igualdad. `SELECT * FROM clientes WHERE ciudad IN ('Lima', 'Bogotá');` |
| Buscar texto por patrón | `%` representa cero o más caracteres; `_` representa uno. `SELECT * FROM clientes WHERE nombre LIKE 'A%';` |
| Buscar sin distinguir mayúsculas | Depende de la intercalación (collation) de la columna o expresión. `SELECT * FROM clientes WHERE email LIKE '%@EXAMPLE.COM';` |
| Comprobar valores nulos | Usa `IS NULL` o `IS NOT NULL`, no `= NULL`. `SELECT * FROM clientes WHERE email IS NOT NULL;` |
| Comprobar existencia de filas relacionadas | `EXISTS` es verdadero si la subconsulta devuelve alguna fila. `SELECT c.nombre FROM clientes c WHERE EXISTS (SELECT 1 FROM pedidos p WHERE p.cliente_id = c.id);` |

### Ordenar y agrupar

| Acción | Explicación y ejemplo |
|---|---|
| Orden ascendente | `ASC` ordena de menor a mayor y es el valor predeterminado. `SELECT nombre, precio FROM productos ORDER BY precio ASC;` |
| Orden descendente | `DESC` ordena de mayor a menor. `SELECT nombre, precio FROM productos ORDER BY precio DESC;` |
| Ordenar por varias columnas | Se evalúan en orden, usando la siguiente si hay empate. `SELECT * FROM productos ORDER BY stock ASC, precio DESC;` |
| Contar filas | `COUNT(*)` cuenta filas, incluso si alguna columna contiene `NULL`. `SELECT COUNT(*) AS total_clientes FROM clientes;` |
| Calcular agregados | Funciones comunes: `SUM`, `AVG`, `MIN` y `MAX`. `SELECT AVG(precio) AS precio_promedio FROM productos;` |
| Agrupar resultados | Calcula agregados por cada valor de la columna indicada. `SELECT estado, COUNT(*) AS cantidad FROM pedidos GROUP BY estado;` |
| Filtrar grupos | `HAVING` filtra después de agrupar; `WHERE` filtra filas antes de agrupar. `SELECT estado, COUNT(*) AS cantidad FROM pedidos GROUP BY estado HAVING COUNT(*) > 2;` |

## 6. Actualizar y eliminar datos

| Acción | Explicación y ejemplo |
|---|---|
| Actualizar filas | Usa `WHERE` para limitar las filas afectadas. `UPDATE productos SET precio = 29.90 WHERE id = 1;` |
| Actualizar varias columnas | `UPDATE productos SET precio = 29.90, stock = 15 WHERE id = 1;` |
| Actualizar usando una expresión | El valor se calcula por fila. `UPDATE productos SET stock = stock - 1 WHERE id = 1 AND stock > 0;` |
| Actualizar con otra tabla | `JOIN` permite usar datos relacionados en la actualización. `UPDATE pedidos p JOIN clientes c ON c.id = p.cliente_id SET p.estado = 'revisar' WHERE c.ciudad = 'Lima';` |
| Eliminar filas seleccionadas | Verifica el filtro antes de ejecutar. `DELETE FROM pedidos WHERE id = 10;` |
| Eliminar mediante condición de relación | `JOIN` limita las filas con información de otra tabla. `DELETE p FROM pedidos p JOIN clientes c ON c.id = p.cliente_id WHERE c.ciudad = 'Lima';` |
| Eliminar todas las filas | `DELETE` sin filtro borra todas las filas y puede activar disparadores. `DELETE FROM pedidos;` |
| Vaciar rápidamente una tabla | `TRUNCATE TABLE pedidos;` suele ser más rápido que `DELETE`, pero no permite `WHERE` y tiene comportamiento de DDL. |

## 7. Combinar tablas con `JOIN`

| Acción | Explicación y ejemplo |
|---|---|
| Coincidencias en ambas tablas (`INNER JOIN`) | Devuelve solo clientes que tengan pedidos. `SELECT c.nombre, p.id AS pedido FROM clientes c INNER JOIN pedidos p ON p.cliente_id = c.id;` |
| Todas las filas de la izquierda (`LEFT JOIN`) | Incluye clientes sin pedidos; las columnas del pedido quedan en `NULL`. `SELECT c.nombre, p.id AS pedido FROM clientes c LEFT JOIN pedidos p ON p.cliente_id = c.id;` |
| Todas las filas de la derecha (`RIGHT JOIN`) | Conserva todas las filas de la tabla derecha. `SELECT c.nombre, p.id AS pedido FROM clientes c RIGHT JOIN pedidos p ON p.cliente_id = c.id;` |
| Producto cartesiano (`CROSS JOIN`) | Combina cada fila de una tabla con cada fila de la otra; puede generar muchas filas. `SELECT c.nombre, pr.nombre FROM clientes c CROSS JOIN productos pr;` |
| Unir una tabla consigo misma | Usa alias distintos para comparar filas de la misma tabla. Este ejemplo encuentra clientes de la misma ciudad. `SELECT a.nombre, b.nombre FROM clientes a JOIN clientes b ON a.ciudad = b.ciudad AND a.id < b.id;` |
| Unir más de dos tablas | Encadena las relaciones necesarias. `SELECT p.id, c.nombre, pr.nombre, d.cantidad FROM pedidos p JOIN clientes c ON c.id = p.cliente_id JOIN detalle_pedido d ON d.pedido_id = p.id JOIN productos pr ON pr.id = d.producto_id;` |

## 8. Subconsultas y consultas comunes de MySQL 8

| Acción | Explicación y ejemplo |
|---|---|
| Subconsulta escalar | Usa una consulta que devuelve un solo valor. `SELECT nombre, precio FROM productos WHERE precio > (SELECT AVG(precio) FROM productos);` |
| Subconsulta con `IN` | Compara con una lista producida por otra consulta. `SELECT nombre FROM clientes WHERE id IN (SELECT cliente_id FROM pedidos);` |
| Expresión condicional `CASE` | Devuelve un resultado según condiciones. `SELECT nombre, CASE WHEN stock = 0 THEN 'Agotado' WHEN stock < 5 THEN 'Pocas unidades' ELSE 'Disponible' END AS disponibilidad FROM productos;` |
| CTE (`WITH`) | Da un nombre temporal a una consulta para reutilizarla en la instrucción. `WITH ventas AS (SELECT cliente_id, SUM(total) AS importe FROM pedidos GROUP BY cliente_id) SELECT cliente_id, importe FROM ventas WHERE importe > 100;` |
| CTE recursiva | Repite una consulta para recorrer jerarquías o generar secuencias. `WITH RECURSIVE numeros AS (SELECT 1 AS n UNION ALL SELECT n + 1 FROM numeros WHERE n < 5) SELECT n FROM numeros;` |
| Función de ventana | Calcula sobre un conjunto relacionado sin agrupar ni colapsar las filas. `SELECT nombre, precio, AVG(precio) OVER () AS promedio FROM productos;` |
| Numerar filas | `ROW_NUMBER()` numera filas dentro de cada grupo. `SELECT cliente_id, id, fecha, ROW_NUMBER() OVER (PARTITION BY cliente_id ORDER BY fecha DESC) AS numero FROM pedidos;` |
| Unir resultados (`UNION`) | Combina consultas con el mismo número y tipos compatibles de columnas; elimina duplicados. `SELECT ciudad FROM clientes WHERE ciudad LIKE 'L%' UNION SELECT ciudad FROM clientes WHERE ciudad LIKE 'B%';` |
| Unir conservando duplicados (`UNION ALL`) | Evita el paso de eliminar duplicados. `SELECT ciudad FROM clientes WHERE ciudad LIKE 'L%' UNION ALL SELECT ciudad FROM clientes WHERE ciudad LIKE 'B%';` |

## 9. Restricciones e índices

| Acción | Explicación y ejemplo |
|---|---|
| Clave primaria | Identifica de forma única cada fila; solo puede haber una por tabla. `ALTER TABLE clientes ADD PRIMARY KEY (id);` |
| Clave foránea | Mantiene la relación y la integridad entre tablas. `ALTER TABLE pedidos ADD CONSTRAINT fk_pedidos_clientes FOREIGN KEY (cliente_id) REFERENCES clientes(id);` |
| Restricción única | Impide valores duplicados. `ALTER TABLE clientes ADD CONSTRAINT uq_clientes_email UNIQUE (email);` |
| Restricción de no nulo | Exige que la columna tenga un valor. `ALTER TABLE productos MODIFY nombre VARCHAR(120) NOT NULL;` |
| Restricción `CHECK` | Valida una condición al insertar o actualizar. `ALTER TABLE productos ADD CONSTRAINT chk_precio_positivo CHECK (precio >= 0);` |
| Valor predeterminado | Se usa cuando no se proporciona la columna. `ALTER TABLE pedidos ALTER COLUMN estado SET DEFAULT 'pendiente';` |
| Índice sencillo | Puede acelerar búsquedas y ordenamientos, con un coste de espacio y escritura. `CREATE INDEX idx_productos_nombre ON productos(nombre);` |
| Índice compuesto | El orden de columnas importa; diseña el índice según las consultas. `CREATE INDEX idx_pedidos_cliente_fecha ON pedidos(cliente_id, fecha);` |
| Índice único | Impide duplicados y puede acelerar búsquedas. `CREATE UNIQUE INDEX uq_clientes_email ON clientes(email);` |
| Eliminar un índice | `DROP INDEX idx_productos_nombre ON productos;` |

## 10. Vistas

| Acción | Explicación y ejemplo |
|---|---|
| Crear una vista | Guarda una consulta reutilizable; normalmente no almacena por sí misma las filas. `CREATE VIEW resumen_pedidos AS SELECT cliente_id, COUNT(*) AS cantidad, SUM(total) AS importe FROM pedidos GROUP BY cliente_id;` |
| Consultar una vista | Se usa como una tabla en una consulta. `SELECT * FROM resumen_pedidos;` |
| Reemplazar una vista | Crea la vista o actualiza su definición. `CREATE OR REPLACE VIEW resumen_pedidos AS SELECT cliente_id, SUM(total) AS importe FROM pedidos GROUP BY cliente_id;` |
| Eliminar una vista | `DROP VIEW IF EXISTS resumen_pedidos;` |

## 11. Transacciones

Las tablas deben usar un motor transaccional, como **InnoDB**, para que las transacciones y las claves foráneas tengan el comportamiento esperado.

| Acción | Explicación y ejemplo |
|---|---|
| Iniciar una transacción | `START TRANSACTION;` |
| Confirmar cambios | Hace permanentes los cambios de la transacción. `COMMIT;` |
| Deshacer cambios | Revierte los cambios no confirmados de la transacción. `ROLLBACK;` |
| Crear un punto de retorno | Marca un punto al que se puede volver. `SAVEPOINT antes_de_actualizar;` |
| Volver a un punto de retorno | Revierte solo lo ocurrido después del punto. `ROLLBACK TO SAVEPOINT antes_de_actualizar;` |
| Ejemplo completo | La aplicación debe comprobar que el `UPDATE` afectó una fila; si no, debe ejecutar `ROLLBACK` en lugar de `COMMIT`. `START TRANSACTION; UPDATE productos SET stock = stock - 1 WHERE id = 1 AND stock > 0; INSERT INTO pedidos (cliente_id, total) VALUES (1, 29.90); COMMIT;` |

> Algunas instrucciones de definición de datos (DDL), por ejemplo `CREATE TABLE` o `ALTER TABLE`, pueden provocar un `COMMIT` implícito. No las mezcles con una transacción suponiendo que se podrán deshacer.

## 12. Usuarios y permisos

Ejecuta estas instrucciones con una cuenta autorizada. Prefiere cuentas con privilegios mínimos y evita conceder `ALL` sin necesidad.

| Acción | Explicación y ejemplo |
|---|---|
| Crear un usuario | `CREATE USER 'lector'@'localhost' IDENTIFIED BY 'Cambia_Esta_Clave_123!';` |
| Conceder permisos de lectura | `GRANT SELECT ON tienda.* TO 'lector'@'localhost';` |
| Conceder permisos concretos | `GRANT SELECT, INSERT, UPDATE ON tienda.* TO 'app'@'localhost';` |
| Ver permisos | `SHOW GRANTS FOR 'lector'@'localhost';` |
| Revocar permisos | `REVOKE INSERT ON tienda.* FROM 'app'@'localhost';` |
| Eliminar un usuario | `DROP USER 'lector'@'localhost';` |

## 13. Diagnóstico y optimización

| Acción | Explicación y ejemplo |
|---|---|
| Ver el plan de ejecución | `EXPLAIN` muestra cómo MySQL planea ejecutar una consulta. `EXPLAIN SELECT * FROM pedidos WHERE cliente_id = 1;` |
| Analizar una consulta ejecutada | `EXPLAIN ANALYZE` ejecuta la consulta y presenta tiempos y filas observados. Úsalo con consultas cuyo coste conoces. `EXPLAIN ANALYZE SELECT * FROM pedidos WHERE cliente_id = 1;` |
| Ver índices de una tabla | `SHOW INDEX FROM pedidos;` |
| Contar y resumir por columna | `SELECT ciudad, COUNT(*) FROM clientes GROUP BY ciudad;` |
| Consultar estado de la sesión | `SHOW STATUS;` |
| Consultar variables del servidor | `SHOW VARIABLES LIKE 'max_connections';` |

## 14. Importar y exportar

| Acción | Explicación y ejemplo |
|---|---|
| Ejecutar un archivo SQL desde la terminal | Importa el archivo en la base indicada. `mysql -u usuario -p tienda < respaldo.sql` |
| Exportar una base con `mysqldump` | Genera un archivo SQL para respaldo. `mysqldump -u usuario -p --routines --triggers tienda > respaldo.sql` |
| Importar CSV desde SQL | Requiere que el servidor pueda leer la ruta y que la configuración permita `LOAD DATA`. `LOAD DATA INFILE '/var/lib/mysql-files/clientes.csv' INTO TABLE clientes FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' LINES TERMINATED BY '\n' IGNORE 1 ROWS (nombre, email, ciudad);` |
| Exportar resultados a CSV con `SELECT` | `INTO OUTFILE` escribe en el servidor, no necesariamente en tu equipo; el destino debe ser permitido y no existir. `SELECT id, nombre, email FROM clientes INTO OUTFILE '/var/lib/mysql-files/clientes.csv' FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' LINES TERMINATED BY '\n';` |

## 15. Procedimientos, funciones y disparadores

Estas instrucciones son útiles para lógica almacenada en el servidor. En clientes de terminal, `DELIMITER` cambia el terminador que reconoce el cliente; no es una instrucción SQL enviada al servidor.

| Acción | Explicación y ejemplo |
|---|---|
| Crear un procedimiento | Agrupa instrucciones que se ejecutan al invocarlo. Ejecuta cada línea en el cliente `mysql`. |
| Ejecutar un procedimiento | `CALL listar_productos();` |
| Eliminar un procedimiento | `DROP PROCEDURE IF EXISTS listar_productos;` |
| Crear una función | Devuelve un valor y se puede usar en expresiones. Ejecuta cada línea en el cliente `mysql`. |
| Llamar a una función | `SELECT aplicar_descuento(100.00);` |
| Crear un disparador | Ejecuta lógica al insertar, actualizar o borrar filas. `CREATE TRIGGER antes_insertar_producto BEFORE INSERT ON productos FOR EACH ROW SET NEW.nombre = TRIM(NEW.nombre);` |
| Eliminar un disparador | `DROP TRIGGER IF EXISTS antes_insertar_producto;` |

Procedimiento:

```sql
DELIMITER //
CREATE PROCEDURE listar_productos()
BEGIN
    SELECT id, nombre, precio FROM productos;
END //
DELIMITER ;
```

Función:

```sql
DELIMITER //
CREATE FUNCTION aplicar_descuento(precio DECIMAL(10,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
RETURN precio * 0.90 //
DELIMITER ;
```

## 16. Funciones útiles

| Función | Uso y ejemplo |
|---|---|
| `CONCAT(a, b)` | Une texto. `SELECT CONCAT(nombre, ' <', email, '>') FROM clientes;` |
| `UPPER(texto)`, `LOWER(texto)` | Cambia el texto a mayúsculas o minúsculas. `SELECT LOWER(email) FROM clientes;` |
| `CHAR_LENGTH(texto)` | Cuenta caracteres. `SELECT CHAR_LENGTH(nombre) FROM clientes;` |
| `TRIM(texto)` | Quita espacios al principio y al final. `SELECT TRIM(nombre) FROM clientes;` |
| `ROUND(numero, decimales)` | Redondea un número. `SELECT ROUND(AVG(precio), 2) FROM productos;` |
| `COALESCE(a, b, ...)` | Devuelve el primer argumento no nulo. `SELECT COALESCE(email, 'Sin correo') FROM clientes;` |
| `IF(condición, a, b)` | Devuelve `a` si la condición es verdadera y `b` en caso contrario. `SELECT nombre, IF(stock > 0, 'Disponible', 'Agotado') FROM productos;` |
| `DATE()` | Extrae la fecha de una expresión de fecha y hora. `SELECT DATE(fecha) FROM pedidos;` |
| `DATE_ADD(fecha, INTERVAL n unidad)` | Suma un intervalo. `SELECT DATE_ADD(NOW(), INTERVAL 7 DAY);` |
| `DATEDIFF(fecha1, fecha2)` | Devuelve la diferencia en días entre fechas. `SELECT DATEDIFF(CURDATE(), DATE(fecha)) FROM pedidos;` |
| `JSON_EXTRACT(doc, ruta)` | Lee un valor dentro de un documento JSON. `SELECT JSON_EXTRACT('{"tema":"azul"}', '$.tema');` |

## Buenas prácticas rápidas

- Selecciona columnas explícitamente en consultas de aplicación en vez de depender de `SELECT *`.
- Usa parámetros preparados desde tu lenguaje de programación; no construyas SQL concatenando entradas del usuario.
- Define claves primarias, claves foráneas y restricciones que representen las reglas de tus datos.
- Usa `DECIMAL` para importes y agrega índices según consultas reales.
- Antes de un `UPDATE` o `DELETE` importante, ejecuta primero un `SELECT` con el mismo filtro.
- Guarda respaldos y prueba la restauración; un archivo de respaldo no sirve si no puede recuperarse.
- Consulta la documentación oficial de MySQL para detalles dependientes de la versión: <https://dev.mysql.com/doc/>.
