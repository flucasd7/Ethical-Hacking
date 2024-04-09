**SUID Bit (Set User ID)**: When the SUID bit is set on an executable file, it allows the file to be executed with the permissions of the file's owner rather than the user who is executing it. This can be useful for allowing regular users to execute certain privileged actions that require elevated permissions.
`find / \-perm -4000 2>/dev/null` *looking for files with SUID permissions*
