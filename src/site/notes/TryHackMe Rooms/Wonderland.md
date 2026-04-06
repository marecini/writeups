---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/wonderland/","tags":["tryhackme","offensivesecurity","ethicalhacking","wonderland","steganography"],"created":"2026-04-06T13:26:22.348+02:00","updated":"2026-04-06T14:35:40.990+02:00","dg-note-properties":{"tags":["tryhackme","offensivesecurity","ethicalhacking","wonderland","steganography"]}}
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

------

## Post-exploitation

## Pwnage

First flag: 
Second flag: 

## Attack Pattern Analysis (APA)