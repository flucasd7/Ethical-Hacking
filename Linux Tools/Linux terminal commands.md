## Piping
### xargs
How to pass to a pipeline a command that doesn't accept standard in inputs, we convert from standard input into command line arguments

`$ date | echo`  *echo doesn't accept input arguments, so date will not be shown

`$ data | xargs echo "hello"` *will print hello and after that, the date result*

`$ data | cut --delimiter= " " --fields=1 | xargs echo` *shows only the name of the day*

![[xargs_example.png]]

### Strace
A powerful debugging tool that provides a system call trace, that is, a list of system calls made by a process.
`$ strace ./test.sh `

We can determine for instance that a binary is a server using a port

![[strace.png]]
`$nc localhost 9898` *we test the connection to the server*

### xclip
Copying the result of  cat command to the clipboard:

*cat file.txt | xclip -sel clip*
### nvim

### awk

### Nohup

`nohup` is a command that's used to run another command with hangup signals ignored, so that the command can continue running in the background even after the user has logged out. The syntax for using `nohup` is as follows:

`nohup command [arguments] &`

Here's what happens when you use `nohup` in this manner:

- `nohup` tells the Linux system not to stop the command once you log out.
- `command [arguments]` is the command you want to run in the background.
- The `&` at the end of the command puts it in the background immediately so you can continue using the terminal.

`nohup python script.py &`
