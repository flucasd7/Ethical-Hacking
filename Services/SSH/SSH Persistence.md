
In our attacker machine, we crete a public and private key:

`AttackerMachine$ ssh-keygen`

Then move to: /home/*user*/.ssh/id_rsa

`$cat ~/.ssh/id_rsa.pub` *ssh key pair verification*

`$cat ~/.ssh/id_rsa.pub | tr -d '\n' | xclip -sel clip` *tr -d removes the line break \n of the file and xclip copy the result*

We copy this key in the victime machine by creating the *authorized_keys* file in *~/.ssh* directory

