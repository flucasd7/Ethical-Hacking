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


