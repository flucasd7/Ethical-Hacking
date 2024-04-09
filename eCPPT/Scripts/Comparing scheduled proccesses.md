````
#!/bin/bash
old_process = $(ps -eo command)

while true; do
	new_process=$(ps -eo command)
	diff <(echo "$old_process") <(echo "$new_process") | grep "[\>\<]" | grep -vE "command|procmon|kworker"
	old_process=$new_process
done
````
### The `while true; do ... done` Loop

This is an infinite loop that keeps the script running until it's manually terminated. Inside this loop, the script performs its checks repeatedly.

### The `ps -eo command` Command

`ps -eo command` lists all currently running processes with their command lines. The `-e` option selects all processes, and the `-o command` option formats the output to show only the command name.

### The Use of `diff`

`diff <(command1) <(command2)` is a powerful construct that compares the output of two commands. Here, it's being used to compare the list of processes at two different times. `diff` outputs lines that are different between the two sets of data:

- Lines beginning with `<` show items present in the first set but not in the second.
- Lines beginning with `>` show items present in the second set but not in the first.

This is how changes in the running processes are detected.

### The `grep "[\>\<]" | grep -vE "command|procmon|kworker"` Part

- `grep "[\>\<]"` filters the output of `diff` to only show lines indicating differences (those starting with `<` or `>`).
- `grep -vE "command|procmon|kworker"` excludes lines containing `command`, `procmon`, or `kworker` from the results. The `-v` option inverts the match, meaning it only shows lines that do **not** match the given patterns. `E` enables extended regular expressions, allowing for the use of the `|` operator for logical OR.

### Variable Assignment and Usage

- `new_process=$(ps -eo command)` captures the current list of running processes.
- The `diff` command then compares this list (`new_process`) to a previously saved list (`old_process`).
- After `diff` has been executed, `old_process=$new_process` updates `old_process` to reflect the current state of running processes, readying it for the next iteration of the loop.