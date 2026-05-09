---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/aster/","tags":["ethicalhacking","offensivesecurity","tryhackme","pentesting","writeup"],"created":"2026-05-05T09:33:48.896+02:00","updated":"2026-05-09T20:48:50.923+02:00","dg-note-properties":{"tags":["ethicalhacking","offensivesecurity","tryhackme","pentesting","writeup"]}}
---

![](/img/user/Attachments/redteaming2.png)

--------
## Description


---------
## Recon

As always a full nmap scan is required to discover running services and active ports on the system. 



Once the running ports and services are discovered we move forward to enumerating the identified services and ports. 

![](/img/user/Attachments/nmap%204.png)
-------
## Enumeration

![](/img/user/Attachments/output.pyc.png)

```bash
uncompyle6 output.pyc 
```

So first obvious clue is to explore the **output.pyc** file. Seeing it is byte-compiled and unreadable the tool **uncompyle6** is able to decode the bytes. The output shows 2 hex strings. 


![](/img/user/Attachments/decodedascii1.png)

![](/img/user/Attachments/decodedascii2.png)

```bash
python3 -c "print(bytes.fromhex('476f6f64206a6f622c2075736572202261646d696e2220746865206f70656e20736f75726365206672616d65776f726b20666f72206275696c64696e6720636f6d6d756e69636174696f6e732c20696e7374616c6c656420696e20746865207365727665722e').decode('ASCII'))"
```

Running the above command on both hex strings shows that there is a user named **admin** on the system. This opens the door for brute-forcing. Worth nothing that the 2nd hex strings repeeats the same sentence 3 times. Maybe that is also a hint for a type of service running somewhere. 

The 2 VoIP ports are part of the FOSS framework installed on the server. (Asterisk)


![](/img/user/Attachments/hydra-login-found.png)


Both hydra and metasploit confirms these credentials. Let's authenticate with these credentials. 

---

## Exploitation

### Enumerating Asterisk Call Manager 5.0.2 

![](/img/user/Attachments/action-status-error.png)


```bash
Action: Login
Username: admin
Secret: abc123
```

![](/img/user/Attachments/ami-admin-access.png)

And the credentials work. Admin access is achieved. Let's explore the system. 

![](/img/user/Attachments/manager-show-settings.png)

```bash
Action: Command 
Command: manager show settings 
[Enter] 
[Enter]
```

![](/img/user/Attachments/ami-core-version.png)

```bash
Action: Command 
Command: core show version 
[Enter] 
[Enter]
```

![](/img/user/Attachments/pjsip-files-empty.png)

Since `pjsip show auths` returns **"No objects found,"** ==it confirms the server is **not** using the newer PJSIP driver for its users==. Also the empty files for the 2 config files shown in the screenshot further confirms this. 


![](/img/user/Attachments/pw.png)

```bash
Action: Command
Command: sip show users
```

```
# Harry credentials
username: Harry
Secret: p4ss#w0rd!#
```

Testing out these credentials to authenticate with the Asterisk server fails. However it is also revealed via further enumeration that only 1 user is configured - **admin** - Lets try SSH as Harry

![](/img/user/Attachments/harry-ssh.png)

Its a success! Now it's finally looking like something. 

#### 1st flag: thm{bas1c_aster1ck_explotat1on}


----
## Post-exploitation

![](/img/user/Attachments/jarfile.png)

```bash
scp harry@10.112.145.21:Example_Root.jar ./
```

So there is a .jar file. Let's acquire it to my machine. Now time to explore the file. First task is to unzip it and check out the contents. Secondly, is to use strings on the **.class** file. 

```
# extracting 
unzip Example_root.class -d folder/ 

# Reveal info 
strings Example_Root.class
```

![](/img/user/Attachments/strings-java-file.png)

What stands out in the output from **strings** is the **flag.dat & root.txt** file. Time to acquire the root flag.  So upon exploring the system it was easy to determine that **harry** has no permissions to run sudo on the box. Nothing valuable in the crontab and lastly no binaries with SetUID set. 

----

### Moving to Linux Exploit Suggester

Running the script on the server reveals a plethora of kernel exploits which are available. 

### Kernel Exploitation

This was too easy. And upon executing this attack the flag was nowhere to be found. Reading the THM instructions the .jar file must be reversed to get the flag. Les.sh reveals a CVE which can be used to exploit the kernel and gets root access. 

**Kernel Exploitation**
```
# acquire the kernel exploit
https://ssd-disclosure.com/ssd-advisory-overlayfs-pe/

# transfer it to target via python http server or just nano the file

# compile it with gcc
gcc -o exploit.c exploit

# run it
./exploit

```

Root is gained. Not the intended way though. 

![](/img/user/Attachments/root%204.png)

### Reversing the .jar file

So reversing the file is straight forward. 

```
# create the trigger
touch /tmp/flag.dat

# run the file 
java Example_root.jar 
```

Once running the command the root.txt file is generated. The trigger file can be empty. 

------
## Attack Pattern Analysis (APA)