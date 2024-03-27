instalamos el binario en una carpeta y con cmd lo ubicamos para correr el ejecutable por consola en windows:

(ruta_hashcat)>hashcat.exe -h   *muestra las opciones del tool*

(ruta_hashcat)>hashcat.exe -b   *realiza un test de speed  de los tipos de hash en la PC*

hashcat.exe -m 0 -a 0 -D2 example.hash dict.txt   *-m indica el modo de hash donde 0 es MD5, -a el tipo de ataque (0 es modo straight, existe fuerza bruta,combinado, etc). -D2 es el* diccionario

>hashcat -a 0 -m 9600 --status hash /root/Desktop/wordlists/1000000-password-seclists.txt --force      //-m 9600 para hash de MS Office 2013, hash es el archivo con el hash del documento Office limpio

rule based attack: Reglas para modificar el password de los diccionarios, como en john

Creamos en un archivo las diferentes reglas que queremos considerar (l for lowercase, r invertir la palabra completa, etc) ver la documentación de hashcat

>hashcat.exe -m 0 -a 0 -D2 example.hash dict.txt -r rules\customrule.txt [\\-r para agregar un archivo con las reglas especificadas](file://-r%20para%20agregar%20un%20archivo%20con%20las%20reglas%20especificadas). El tool tiene su propio archivo de rules, una carpeta dentro del mismo archivo descomprimido (“base64.rule” por ejemplo o “dive”)

mask attack: Ataque de fuerza bruta con reglas para minimizar la cantidad de pruebas

>hashcat.exe -m 0 -a 3 example.hash ?l?l?l?a?a   //-a 3, ataque de fuerza bruta, ?l define un pool de lowercase y ?a todos los caracteres ASCII (solo contraseñas de 5 caracteres)

>type hashcat.potfile     //Muestra todos los hashes encontrados y guardados por el tool