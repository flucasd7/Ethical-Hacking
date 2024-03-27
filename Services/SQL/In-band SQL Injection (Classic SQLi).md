
In-band SQL Injection is the most common and easy-to-exploit of SQL Injection attacks. In-band SQL Injection occurs when an attacker is able to use the same communication channel to both launch the attack and gather results.

The two most common types of in-band SQL Injection are _Error-based SQLi_ and _Union-based SQLi_.
### Error-based SQLi
[[Error-based SQL-Injection]]

Error-based SQLi is an in-band SQL Injection technique that relies on error messages thrown by the database server to obtain information about the structure of the database. In some cases, error-based SQL injection alone is enough for an attacker to enumerate an entire database. While errors are very useful during the development phase of a web application, they should be disabled on a live site, or logged to a file with restricted access instead.

### Union-based SQLi

Union-based SQLi is an in-band SQL injection technique that leverages the UNION SQL operator to combine the results of two or more SELECT statements into a single result which is then returned as part of the HTTP response.

#### Concepts
Retreival de data de la DB usndo UNION SQL command. Por eso es conocida como UNION-bases SQL injection. Este tipo de ataqueite extraer la DB en forma de DB names, tablas y data.

Al inyectar el siguiente códugo, dado que no existe el id=9999, la aplicación ejecuta la segunda consulta. Ver que usamos el UNION ALL para evitar el efecto de DISTINCT

![SELECT name FROM Users WHERE ALL SELECT 
cc num FROM creditcards WHERE user id=l;](file:///C:/Users/fredd/AppData/Local/Temp/msohtmlclip1/02/clip_image001.png)

Un buen truco es agregar ;-- al final de la consulta para poner como comentario la sintaxis interna que seguiría.

Tener en cuenta que:

- El field type del segundo SELECT debe coincidir con el del primero
- El numero de fields en el segundo SELECT debe coincidir con el número del primero
- Necesitamos saber la estructura de la DB en terminos de tablas y columnas.

1. Enumeración de campos de DB

Primero debemos considerar el tipo de cada variable (int, varchar, etc) y la cantidad de columnas, podemos ver los siguientes mensajes de error para descartar:

![[Pasted image 20240316065429.png]]

![[Pasted image 20240316065531.png]]

Podemos inyectar valores NULL en los UNION hasta dejar de presentar el error

![[Pasted image 20240316065544.png]]

1. 9999 UNION SELECT NULL; -- -
2. 9999 UNION SELECT NULL, NULL; -- -
3. 9999 UNION SELECT NULL, NULL, NULL; -- -
4. 9999 UNION SELECT NULL, NULL, NULL, NULL; -- -

##### Enumeración BLIND

En caso de que la aplicación no muestre errores inyectamos campos NULL de la misma forma pero hasta obtener un query valido. Cuando vemos un comportamiento diferente con uno de las sentencias, tenemos el numero de fields.

#### Identificando los tipos de los campos de la DB

Progamos reemplazando los NULLs que usamos para el numero de campos con un intenger o varchar y observar el comportamiento de la aplicación

|                             |                                                                                                                                                |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| 'UNION SELECT 1, NULL; -- - | Si todo funciona bien, vamos por el segundo campo dependiente del comportamiento del site si con True o False tiene un comportamiento definido |

Como paso 3 ya vendría el Dumpeo del contenido de la DB. Podemos buscar sheet en [https://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet](https://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet)

|   |   |
|---|---|
|' union select database()-- -|Extraemos el nombre de la base de datos|
|=9999'UNION SELECT @@version, 222;'els3'; -- -|Como ya determinamos los valores de cada campo, podemos aprovechar en llamar variables de la DB para extraer información. @@version muestra la version de la DB|
|=9999'UNION SELECT user_name(), 222;'els3'; -- -|Extraemos el usuario actual de la DB|
|=9999'UNION SELECT name, 222;'els3' FROM master..syslogins; -- -|Extraemos el primer usuario del DB server|
|=9999'UNION SELECT name, 222;'els3' FROM master..syslogins WHERE name NOT IN ('DOMAIN\Usuario1'); -- -|Extraemos el segundo usuario, podemos iterar este proceso para los demás.|
|=9999'UNION SELECT name, 222;'els3' FROM master..syslogins WHERE name NOT IN ('DOMAIN\Usuario1','DOMAIN\Usuario2'); -- -|Segunda interación para enumerar usuarios.|

|   |   |
|---|---|
|=9999'UNION SELECT db_name(), 222;'els3'; -- -|Extraemos el DB name|

#### Casos ejemplo

|                            |                                                                                          |
| -------------------------- | ---------------------------------------------------------------------------------------- |
| ->1' or 1=1 limit 3,2;-- - | Limita mostrar la cuarta fila tomándolo de 2 en dos las filas (empieza el 0 el contador) |
## Inferential SQLi (Blind SQLi)

Inferential SQL Injection, unlike in-band SQLi, may take longer for an attacker to exploit, however, it is just as dangerous as any other form of SQL Injection. In an inferential SQLi attack, no data is actually transferred via the web application and the attacker would not be able to see the result of an attack in-band (which is why such attacks are commonly referred to as “[blind SQL Injection attacks](https://www.acunetix.com/websitesecurity/blind-sql-injection/)”). Instead, an attacker is able to reconstruct the database structure by sending payloads, observing the web application’s response and the resulting behavior of the database server.

The two types of inferential SQL Injection are _Blind-boolean-based SQLi_ and _Blind-time-based SQLi_.

### Boolean-based (content-based) Blind SQLi

Boolean-based SQL Injection is an inferential SQL Injection technique that relies on sending an SQL query to the database which forces the application to return a different result depending on whether the query returns a TRUE or FALSE result.

Depending on the result, the content within the HTTP response will change, or remain the same. This allows an attacker to infer if the payload used returned true or false, even though no data from the database is returned. This attack is typically slow (especially on large databases) since an attacker would need to enumerate a database, character by character.

### Time-based Blind SQLi

Time-based SQL Injection is an inferential SQL Injection technique that relies on sending an SQL query to the database which forces the database to wait for a specified amount of time (in seconds) before responding. The response time will indicate to the attacker whether the result of the query is TRUE or FALSE.

Depending on the result, an HTTP response will be returned with a delay, or returned immediately. This allows an attacker to infer if the payload used returned true or false, even though no data from the database is returned. This attack is typically slow (especially on large databases) since an attacker would need to enumerate a database character by character.

## Out-of-band SQLi

[Out-of-band SQL Injection](https://www.acunetix.com/blog/articles/blind-out-of-band-sql-injection-vulnerability-testing-added-acumonitor/) is not very common, mostly because it depends on features being enabled on the database server being used by the web application. Out-of-band SQL Injection occurs when an attacker is unable to use the same channel to launch the attack and gather results.

Out-of-band techniques, offer an attacker an alternative to inferential time-based techniques, especially if the server responses are not very stable (making an inferential time-based attack unreliable).

Out-of-band SQLi techniques would rely on the database server’s ability to make DNS or HTTP requests to deliver data to an attacker. Such is the case with Microsoft SQL Server’s `xp_dirtree` command, which can be used to make DNS requests to a server an attacker controls; as well as Oracle Database’s UTL_HTTP package, which can be used to send HTTP requests from SQL and PL/SQL to a server an attacker controls.