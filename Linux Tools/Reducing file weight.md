# Using go

We need to compile at first the go file:
````
git clone https://github.com/jpillora/chisel
````

Then compile:
`go build .`

After compiled, we can execute the command:

```
go build -ldflags "-s -w" .
````

Compiles the Go program in the current directory into an executable binary, applying linker flags to optimize the binary for size by stripping symbols and disabling DWARF generation. This results in a smaller binary size while sacrificing some debugging information.

`upx chisel` *reduce the size of chisel*

`du -hc chisel` *to verify the weight of the file*

![[Pasted image 20240406141155.png]]