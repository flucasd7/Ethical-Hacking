## Debugging
### gbd
`$gdb ./file_vulnerable -q`
`gdb>r` *to run*
![[gdb.png]] EDP is 'AAAA' so it's a bufferoverflow

gdb> checksec *verify the security mesuare for a binary*

### Mona
https://github.com/corelan/mona/blob/master/mona.py

After downloaded, install in *C:\Program Files\Immunity Inc\Immunity Debugger\PyCommands*

Testing:
![[Pasted image 20240324154807.png]]Creating a working folder in the desktop:
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
![[Pasted image 20240324155517.png]]

Copying the file to the clipboard:
`cat bytearray.txt| xclip -sel clip`

We dd this to out payload

#### Looking for vulnerable services

```
!mona modules
```

We need to look for a service with FALSE in all security measures:
![[Pasted image 20240324161914.png]]

Looking for a specify opcode in the library:
`!mona find -s "\xff\xe4" -m qsqlite.dll`

