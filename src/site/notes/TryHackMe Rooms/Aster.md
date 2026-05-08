---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/aster/","tags":["ethicalhacking","offensivesecurity","tryhackme","pentesting","writeup"],"created":"2026-05-05T09:33:48.896+02:00","updated":"2026-05-08T17:24:39.835+02:00","dg-note-properties":{"tags":["ethicalhacking","offensivesecurity","tryhackme","pentesting","writeup"]}}
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


![](/img/user/Attachments/manager-show-settings.png)

```bash
Action: Command Command: manager show settings [Enter] [Enter]
```

![](/img/user/Attachments/ami-core-version.png)

```bash
Action: Command Command: core show version [Enter] [Enter]
```




-----




----
## Post-exploitation


------
## Attack Pattern Analysis (APA)