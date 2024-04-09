In order to verify the nmap cathegories:

`locate .nse | xargs grep "categories" | grep -oP '".*?"' | sort --u`
![[Pasted image 20240330112156.png]]

nmap --script "vuln and safe" -p80 10.10.1.1 *specifying scripts categories*

