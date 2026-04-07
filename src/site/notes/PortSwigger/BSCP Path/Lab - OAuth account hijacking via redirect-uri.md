---
{"dg-publish":true,"permalink":"/port-swigger/bscp-path/lab-o-auth-account-hijacking-via-redirect-uri/","created":"2026-03-25T18:48:15.386+01:00","updated":"2026-04-07T10:22:14.908+02:00","dg-note-properties":{}}
---


## Description of the Lab
This lab uses an OAuth service to allow users to log in with their social media account. A misconfiguration by the OAuth provider makes it possible for an attacker to steal authorization codes associated with other users' accounts.

To solve the lab, steal an authorization code associated with the admin user, then use it to access their account and delete the user `carlos`.

The admin user will open anything you send from the exploit server and they always have an active session with the OAuth service.

You can log in with your own social media account using the following credentials: `wiener:peter`.


## Enumeration
Visiting the site, logging in with the creds as per the instructions and using Burp as the proxy capturing all of the requests. 

The admin code is seen in the URL callback.

**Admin code:** `"oXoc6s-O_BgHobOpiRlndIoxeH2Rq5Chg7Xxl6XrYEA"`

![Admin_callback_code.png](/img/user/Pentesting/BSCP_Path/lab2/Admin_callback_code.png)



## Payload Crafting
