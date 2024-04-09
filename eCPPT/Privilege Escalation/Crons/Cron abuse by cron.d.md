As we have write permission to file */etc/cron.d*
![[Pasted image 20240406210055.png]]

If service cron is active, we can inject a  cron task

Payload:`
`echo '* * * * * root sh /tmp/reverse.sh' > reverse`

- `* * * * *`: This is the cron job's schedule, and in this case, it means "run every minute of every hour of every day of every month." The five asterisks represent minute, hour, day of the month, month, and day of the week, respectively.
- `root`: Specifies the user under which the cron job should run. In this case, it's `root`, which has administrative privileges.
- `sh /tmp/reverse.sh`: This is the command that the cron job will execute every minute. It tells the system to use `sh` (a shell) to run a script located at `/tmp/reverse.sh`.