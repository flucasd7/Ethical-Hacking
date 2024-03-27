### Stack Canaries

Stack Canaries are very simple - at the beginning of the function, a random value is placed on the stack. Before the program executes `ret`, the current value of that variable is compared to the initial: if they are the same, no buffer overflow has occurred.

If they are not, the attacker attempted to overflow to control the return pointer and the program crashes, often with a `***stack smashing detected***` error message.
#### Bypassing
https://ir0nstone.gitbook.io/notes/types/stack/canaries

## Memory Protection attributes

1. **PAGE_EXECUTE_WRITECOPY**:
    
    - This attribute may be suitable if the exploit involves overwriting executable code in memory and then executing it.
    - If the exploit requires modifying existing code in memory, such as function pointers or return addresses, while still allowing execution of the modified code, `PAGE_EXECUTE_WRITECOPY` can be useful.
    - However, keep in mind that modifications made to pages with this attribute are not immediately propagated back to the file from which they were mapped. This might impact the effectiveness of the exploit, especially if the modified code needs to persist beyond the lifetime of the current process.
2. **PAGE_EXECUTE_READ**:
    
    - This attribute may be suitable if the exploit involves executing existing code in memory without the need for modification.
    - If the exploit relies on executing code already present in the target application's memory space, such as shellcode injected into a buffer, `PAGE_EXECUTE_READ` can be sufficient.
    - However, this attribute prevents modifications to the memory region, so it's not suitable if the exploit requires overwriting existing code or data structures.
-![[Pasted image 20240324162718.png]]