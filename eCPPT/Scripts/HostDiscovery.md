### Discovery Scan
#### HostDiscovery.sh
````
#!/bin/bash
for i in $(seq 1 254); do
	timeout 1 bash -c "ping -c 1 10.10.0.$i" &>/dev/null && echo "[+] Host 10.10.0. $i - Active" & 
done; wait
````
#### HostDiscoveryPort.sh
 ````
#!/bin/bash

function ctrl_c(){
	echo -e "\n\n[!] Quitting..\n"
	 tput cnorm; exit 1
}

 #Ctrl+C
trap ctrl_c INT
 
tput civis
for port in $(seq 1 65565); do
	timeout 1 bash -c "echo '' > /dev/tcp/172.19.0.2/$port" 2>/dev/null && echo "[+] Port $port --open" &
done; wait
tput cnorm
````
  
The ampersand (`&`) at the end of the command `timeout 1 bash -c "echo '' > /dev/tcp/172.19.0.2/$port" 2>/dev/null && echo "[+] Port $port --open"` is used to run the command in the background. This allows the script to initiate the next command in the loop without waiting for the current command to complete. 

`tput civis`: This command hides the cursor in the terminal.
`tput cnorm`: This command restores the cursor to its normal state (makes it visible again).

#### Host Discovery in multiple networks
````
#!/bin/bash

function ctrl_c(){
	echo -e "\n\n[!] Quitting..\n"
	 tput cnorm; exit 1
}

 #Ctrl+C
trap ctrl_c INT


networks=(172.18.0 172.19.0)
tput civis; for network in ${network(@)};do
	echo -e "\n[+] Enumerating network: ${network}.0/24"
	for i in $(seq 1 254); do
		timeout 1 bash -c "echo -c 1 $network.$i" &>/dev/null && echo -e "[+] Host $network.$i - ACTIVE" &
	done; wait
done; tput cnorm
````
### Scan in a proxychain

`$seq 1 65365 | xargs -P 500 -I {} proxychains nmap -sT -Pn -p{} -open -T5 -v -n IP.168.100.129 2>&1 | grep "tcp open"` *scan using 500 threads



