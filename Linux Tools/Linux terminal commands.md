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