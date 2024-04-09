### TTY
`$script /dev/null -c bash`
`$tty`
>/dev/pts/2 *we verify that we have now  tty*

Control+*z*
`AttakerMachine$stty raw -echo;fg`
`reset`
`xterm` *terminal will ask the type*

We will come back to the shell:

`$export TERM=xterm` *control+L will work with this*

`$export SHELL=bash` *we change the default sh shell to bash*

`$stty rows 51 columns 189` *changing the size of windows*

#### No script in shell
In case there is not *script* command in the machine:
`which python3`
`which bash` *verifying if there is bash in the machine*
`which sh` *if, not, verify if there is sh and use it*
`python3 -c 'import pty;pty.spawn("/bin/sh")'` *if there is bash, use /bin/bash*

  
The command you provided, `python3 -c 'import pty;pty.spawn("/bin/sh")'`, is a Python one-liner that invokes a Python 3 interpreter (`python3`) with the `-c` flag, which allows you to execute a single Python command passed as a string.

This particular command imports the `pty` module, which stands for "pseudo-terminal utilities," and then calls the `spawn` function from that module, passing `"/bin/sh"` as an argument.

The `spawn` function is used to spawn a new process and connect it to the controlling terminal. In this case, it's spawning a shell (`/bin/sh`). By doing this, it effectively creates an interactive shell within the current terminal session, which can be useful for various purposes, such as privilege escalation or interactive exploration of the system.

