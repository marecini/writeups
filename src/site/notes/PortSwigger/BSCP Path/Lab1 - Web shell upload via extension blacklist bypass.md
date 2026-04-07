---
{"dg-publish":true,"permalink":"/port-swigger/bscp-path/lab1-web-shell-upload-via-extension-blacklist-bypass/","created":"2026-03-24T12:28:11.031+01:00","updated":"2026-04-07T10:22:21.935+02:00","dg-note-properties":{}}
---


# Description 

This lab contains a vulnerable image upload function. Certain file extensions are blacklisted, but this defense can be bypassed due to a fundamental flaw in the configuration of this blacklist.

To solve the lab, upload a basic PHP web shell, then use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: `wiener:peter`

####  Endpoints
Homepage: https://0a48000b04f5708c80bd306b00f6005b.web-security-academy.net/
Login : https://0a48000b04f5708c80bd306b00f6005b.web-security-academy.net/login
My Account: https://0a48000b04f5708c80bd306b00f6005b.web-security-academy.net/my-account?id=wiener
File Upload Location: https://0a48000b04f5708c80bd306b00f6005b.web-security-academy.net/files/avatars/shell.php5

![account_overview.png](/img/user/Pentesting/BSCP_Path/Lab1/account_overview.png)



## Payload Crafting

**Example Payloads from PortSwigger (PS)**
`"<?php echo file_get_contents('/path/to/target/file'); ?>"`

**Payload in Question**
`"<?php echo file_get_contents('/home/carlos/secret'); ?>"`

Uploading this payload in a **.php** file results in a server error 
`"Server Error: Gateway Timeout (0) connecting to 0ae000f3035e5448820038ef00df007f.web-security-academy.net"`

## Payload Process

**Crafting the payload saving it to a file and via burp the /my-account endpoint is visited and the file is uploaded**
![obscure_filename.png](/img/user/Pentesting/BSCP_Path/Lab1/obscure_filename.png)
**Successfully Uploaded shell.php5**
![shell_uploaded.png](/img/user/Pentesting/BSCP_Path/Lab1/shell_uploaded.png)

**Now locating which endpoint the .php file is uploaded to is achieved by checking the source code**
![locate_file_location.png](/img/user/Pentesting/BSCP_Path/Lab1/locate_file_location.png)


**The webserver serves the .php file as a static file instead of executing it.**
![php5_response.png](/img/user/Pentesting/BSCP_Path/Lab1/php5_response.png)

**Trying a different obscure file extension name**

After updating the extension name to **.pHp** uploading the payload is not possible 
![err_msg.png](/img/user/Pentesting/BSCP_Path/Lab1/err_msg.png)

Uploading **.html** works 
![htmlworsk.png](/img/user/Pentesting/BSCP_Path/Lab1/htmlworsk.png)

**Response**
The shell is served statically

## Hint

You need to upload two different files to solve this lab.


**Uploading .htaccess**
This approach is very aggressive and will affect all other user's images on the server. 
`"SetHandler application/x-httpd-php "`

**Changing Upload Request**

```
"Content-Disposition: form-data; name="avatar"; filename=".htaccess"
Content-Type: text/plain

SetHandler application/x-httpd-php"
```

**Response**
Server responds with 200 OK and afterwards calling the image previously uploaded with a GET request will execute the .php script and the secret will be sent


![secret_retrieved.png](/img/user/Pentesting/BSCP_Path/Lab1/secret_retrieved.png)


