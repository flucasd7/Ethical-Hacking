
As port 3128 SquidProxy (generally HTTP) is open, we can perform an internal enumeration:

`wfuzz -c --hc=404,503 -t 200 -z range,1-65535 -p 10.10.10.67:3128:HTTP http://127.0.0.1:FUZZ` 
- -z is to set a payload,
- 127.0.0.1 because it's the internal host of the proxy that will be scan
- -p for proxy
-
We found port 22 open
![[Pasted image 20240401052413.png]]

As we don't have access to port 22 from our machine, we can use **proxychains** so that through the proxy, we can reach that port.

### Condiguring Proxychains file
File /etc/proxychains.conf

![[Pasted image 20240401053152.png]]`proxychains ssh -q root@localhost` *localhost because is squiproxy which will be ask for port 22 in its own server*

![[Pasted image 20240401053420.png]]

### Discovering containers

`/proc/net/fib_trie` *identifies if we are ina  container*
Vulnerable LFI machine has as IP 192.168.0.10 which is not the IP of main machine so we can infer that web service is given by a container.
![[Pasted image 20240401053706.png]]

`/proc/sched_debug` *To get the services running on the server*

We found *lxc* service which stands for Linux Container
![[Pasted image 20240401055626.png]]

### Lookig for SquidProxy configuration files

By looking in internet, we found default configuration files we can examinate:
`/etc/squid/squid.conf`

We can tranfer the output a a file *output* to better filter after

`./lfi.sh | tee output` *saves the result of this script in output file*

`cat outout | grep -v "^#" | sed '/^\s*$/d'`
- *-v ^#* to remove all lines that start with # 
- in **sed**: *^\s *$* refers to lines with starts with an space and finished with nothing (all lines with no characters). It won consider i.e.: "       test"
![[Pasted image 20240401061132.png]]
