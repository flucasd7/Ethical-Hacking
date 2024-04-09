
## Directory Enumeration

### Dirb

`$dirb [http://192.168.1.1](http://192.168.1.1) /usr/share/metasploit-framework/data/wordlists/directory.txt`        *Ataque de diccionario a la url espceificada*

`$dirb [http://google,com](about:blank) -a “User agent”`        *Configuramos un User Agent String para contactar al servicio web*

>dirb [http://google.com](http://google.com) -p [http://127.0.0.1:8080](http://127.0.0.1:8080) //Configuramos un proxy donde podremos ver la enumeración (el proxy puede ser burpsuite por ejemplo)

>dirb [http://google.com](http://google.com) -p [http://127.0.0.1:808](http://127.0.0.1:808) -c “COOKIE:XYZ”  //Seteamos una cookie predeterminada

>dirb [http://google.com](http://google.com) -p [http://127.0.0.1:808](http://127.0.0.1:808) -u “admin:password” /seteamos un usuario y password como tipo Authorization: Basic AVSVDDS (Codificado)

>dirb [http://google.com](http://google.com) -p [http://127.0.0.1:808](http://127.0.0.1:808) -H “Myhearder: Mycontent  //Creamos un nuevo header.

>dirb [http://google.com](http://google.com) -z 1000        //colocamos un rate para las bquedas de 1000 milisegundos para las búsquedas

>dirb [http://192.168.0.33](http://192.168.0.33) -X “.php,.bak”        //agrega al diccionario las extensiones mencionadas

>dirb [http://192.168.0.33](http://192.168.0.33) -x extensions.txt -z 1000  //Busquedas de 1 segundo usando el diccionario extensions.txt. -o guarda el output en un archivo.

### Gobuter

`$gobuster dir -u [http://www.targetwebsite.com/](http://www.targetwebsite.com/) -w /usr/share/wordlists/big.txt`        *Enumeración de directorios usando diccionarios*

`$gobuster dir -u [http://www.targetwebsite.com/](http://www.targetwebsite.com/) -w /usr/share/wordlists/big.txt -x php,html,htm` *-x especifica las extensiones que queremos*

#### Subdomains Enumeration

`$gobuster vhost -k -u [https://bolt.htb](https://bolt.htb) -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -o gubuster.vhost.443 --append-domain true`

- -k elimina la verificación de TLS -o para guardar el resultado.

`$gobuster vhost -u [http://bolt.htb](http://bolt.htb) -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -o gobuster.vhost.80 --append-domain true`

- Para enumeración de subdominios usamos vhost, también colocamos el --append-domain true para que agregue los valores del diccionario antes, si no, el diccionario tendría que tener el nombre completo con el hostname.

#### Proxy Feature

`$gobuster dir -h http://10.10.2.131 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -t 20 -x html,php,txt --proxy socks5://127.0.0.1:1080`
### WFUZZ

`wfuzz -c --hc 404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt [http://testphp.vulnweb.com/FUZZ](http://testphp.vulnweb.com/FUZZ)`  *reemplaza FUZZ por los valores en el diccionarios. -c es para dar color.--hc no muestra los valores con respuesta especificada (en este caso 404)*

|                                                                                                                                                                    |                                                        |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------ |
| wfuzz -c --hc 404 --hw 38 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt [https://app-test.rimacsev.com/FUZZ](https://app-test.rimacsev.com/FUZZ) | Escondemos todos los resultados que tengan 38 palabras |

### Enumeración de subdominios

> wfuzz -c -t 200 --hc 301 -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.redcross.htb" [https://intra.redcross.htb](https://intra.redcross.htb)

Metasploit

msf>use auxiliary/scanner/http/brute_dirs        //Ataque de diccionario a un host web. Podemos editar el PATH, RHOSTS, etc
## BruteForce

### crunch
`$crunch 8 8 -t kill0r%@ >> passwords` *creates many passwords starting with 'kill0r'. Min 8 and Max 8 characters. % is replaces by numbers and @ for lowcase characters*

We can change the order or % and @ to get new combinations and >> to add new lines to already created file passwords