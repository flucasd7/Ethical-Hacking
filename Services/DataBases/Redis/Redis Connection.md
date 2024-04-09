Connection to Redis
If service is reached by localhost:
`nc 127.0.0.1 6379`

`INFO` *Information about DBs*

![[Pasted image 20240406163112.png]]

`select 0` *db 0 is chosen*
`keys *` *shows the "tables"*
`get hits` *get the information stored in "table" hits*

![[Pasted image 20240406163329.png]]

