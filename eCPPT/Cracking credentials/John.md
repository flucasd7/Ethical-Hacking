Para saber que tipo de cifrado usan los hashes, podemos buscar en /etc/login.defs

grep -A 18 ENCRYPT_METHOD /etc/login.defs  *lee las 18 líneas posteriores al término ENCRYP_METHOD donde se encuentra el algoritmo de hash*

## John the Ripper

Es necesario que las credenciales estén a la par con las contraseñas por lo que usarmo unshadow para juntarlo en un solo archivo una vez que lo tengamos en la misma carpeta los archivo /etc/passwd y /etc/shadow

`unshadow password shadow > hackeame`
### Brute Force

`john --incremental --users:lista hackeame`   *--users especifica una lista de usuarios del total en hackeame. Presionando cualquier tecla se podrá ver el progreso del cracking*

`john --show hackeame` *muestra el resultado del crack hecho en hackeame*

### Dictionary attack

`john --wordlist --users=victima1,victima2 hackeame`*victima 1 y 2 son nombres de usuarios, usamos -wordlist con el diccionario por defecto de john*

`john --wordlist=lista_propia --users=victima1,victima2 hackeame` *usamos lista propia. A veces es necesario especificar el formato con “--format=crypto” para que lo reconozca*

`john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt  password.txt`  *--format=NT para formatos de hashdump de meterpreter en Windows*

`john --wordlist=lista_propia --rules --users=victima1,victima2 hackeame` *--rules habilita el dictionary mangling que es una forma de probar un mismo pass con alfanumericos o min.*

`cat /roo/.john/john.pot`  *ruta donde se almacenan los hashes creackeados*

>/usr/share/john/office2john.py /kali/MS_Word_Document.docx > hash  //extrae el hash del password que protege el documento MS_Word usando office2john python script. Este hash puede ser usado con un diccionario

>john --wordlist=/root/Desktop/wordlists/1000000-password-seclists.txt hash

We can find this by using a **rainbowtable**
https://crackstation.net/