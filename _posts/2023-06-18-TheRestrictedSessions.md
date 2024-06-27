---
title: CyberTalents | The Restricted Sessions
description: Writeup of an medium-rated Web Exploitation Challenge from CyberTalents
date: 2023-06-18 12:00:00 -500
categories: [CyberTalents, Medium]
tags: [Web Exploitation, Session Hijacking, Sensitive Data Exposure, cURL, Firefox DevTools]
pin: true
math: true
mermaid: true
image:
  path: /assets/img/cybertalents-TheRestrictedSessions/cybertalents.png
---

***

<center><strong><font color="DarkGray"><a href="https://cybertalents.com/challenges/web/the-restricted-sessions" target="_blank"><er>The Restricted Sessions</er></a> is a Web Exploitation challenge from CyberTalents, where the flag could only be viewed by logged in users. Additionally, there is no apparent login form, which means working with sessions and cookies was necessary to successfully authenticate as one of the already logged in user and get the flag.</font></strong></center>

***

## **<strong><font color="Brown">Challenge Name</font></strong>**
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/cybertalents-therestrictedsessions/2023-06-21 01_42_16-The Restricted Sessions » CyberTalents — Mozilla Firefox.png" alt="">
<center><i>The Restricted Sessions Challenge</i></center>
<br/>

## **<strong><font color="Brown">Challenge Description</font></strong>**
***
* Flag is restricted to logged users only , can you be one of them.

## **<strong><font color="Brown">Write-Up</font></strong>**
***

Navigating to the provided URL, you will be presented with the following web page:
<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/cybertalents-therestrictedsessions/2023-06-17 21_55_21-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>Main page</i></center>
<br/>
It says that the flag can only be seen to logged-in users, and since there is no login form and given the challenge name, it is certainly about ``Cookies`` and ``Sessions``.

#### **<strong><font color="DarkKhaki">Source Code:</font></strong>**
There is a JavaScript code in the code source of the web page which reveals something very interesting regarding the logic behind the check of logged-in users.
<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/cybertalents-therestrictedsessions/2023-06-17 21_59_34-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>JavaScript code</i></center>
<br/>

* The code checks if the `document.cookie` is not an empty string. It then extracts the value of the `PHPSESSID` cookie using regular expression match ``document.cookie.match(/PHPSESSID=([^;]+)/)[1]``
* Next, it sends a ``POST`` request to the server using `$.post()` method. The request is made to the `getcurrentuserinfo.php` endpoint, and the `PHPSESSID` value is included as a parameter in the request body.
* If the server responds successfully, the callback function specified as the 3rd argument of the `$.post()` method will be executed. The `data` parameter of the callback function will contain the response from the server. In this case, it assigns the response data to the variable `cu`.

<br/>

In other words, when a user visit the main page, the JavaScript code will check if the `PHPSESSID` cookie is set and has a value. If it's the case, it will send a POST request to `getcurrentuserinfo.php` endpoint, which will retrieve the current user information based on the provided PHP session ID. The response from the server is then stored in the `cu` variable. 

#### **<strong><font color="DarkKhaki">Stored sessions:</font></strong>**
With that being said, let's send a request, using `curl`, to the main page `/` with a ``PHPSESSID`` cookie set to a random value (Just for testing purposes):
```shell
curl http://shfjhg&.cybertalentslabs.com -H 'Cookie: PHPSESSID=1234567'
```
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/cybertalents-therestrictedsessions/2023-06-17 22_14_31-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>Requesting the main page with PHPSESSID cookie</i></center>

* The response says that the provided `PHPSESSID` does not figure in `data/session_store.txt` file.

Let's ``GET`` the content of this file, by running the following command:
```shell
curl http://shfjhg&.cybertalentslabs.com/data/session_store.txt
```
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/cybertalents-therestrictedsessions/2023-06-17 22_17_42-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>Stored sessions</i></center>

* It looks like this file contains some PHPSESSID values (3 to be exact).

Let's try one of them and send a request to the main page:
```shell
curl http://shfjhg&.cybertalentslabs.com -H 'Cookie: PHPSESSID=iuqwhe23eh23kej2hd2u3h2k23'
```
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/cybertalents-therestrictedsessions/2023-06-17 22_20_12-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>UserInfo Cookie</i></center>

* The response implies that we need another cookie named `UserInfo` that needs to be sent along with `PHPSESSID`

Unfortunately, we don't have a valid username, but let's send a random username (like `admin` for testing purposes):
```shell
curl http://shfjhg&.cybertalentslabs.com -H 'Cookie: PHPSESSID=iuqwhe23eh23kej2hd2u3h2k23; UserInfo=admin'
```
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/cybertalents-therestrictedsessions/2023-06-17 22_22_36-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>Validation failed</i></center>

* As expected, we got a `Validation failed` message as a response from the server.

#### **<strong><font color="DarkKhaki">What's happening in the backend ?</font></strong>**
In the backend, when we sent the latter request (the one with ``PHPSESSID`` and ``UserInfo`` cookies), the JavaScript code (the one we analyzed earlier) is executed and will send a ``POST`` request to `/getcurrentuserinfo.php` endpoint with the value of `PHPSESSID` as a parameter. After that, the server will respond with information regarding the user holding the PHP session ID. 
<br/>
This information may include the ``username``, which will be compared to the `UserInfo` Cookie. If it matches, the validation is valid. Otherwise, a `Validation failed` message is returned from the server.

#### **<strong><font color="DarkKhaki">Interacting with getcurrentuserinfo.php endpoint:</font></strong>**
Let's manually send a POST request, using `curl` as always, to this endpoint while sending one of the stored session IDs as a parameter:
```shell
curl -X POST http://shfjhg&.cybertalentslabs.com -d 'PHPSESSID=iuqwhe23eh23kej2hd2u3h2k23'
```
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/cybertalents-therestrictedsessions/2023-06-17 22_32_34-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>JSON data</i></center>

* The response is ``JSON`` data, which contains information of the user with the PHP session ID we provided.

#### **<strong><font color="DarkKhaki">Flag:</font></strong>**
At this point, we have a valid PHP session ID and a valid username. All that's left to do is send a GET request to the main page `/` with the following cookies: `PHPSESSID=iuqwhe23eh23kej2hd2u3h2k23` and `UserInfo=john`

* Using cURL:
```shell
curl http://shfjhg&.cybertalentslabs.com -H 'Cookie: PHPSESSID=iuqwhe23eh23kej2hd2u3h2k23; UserInfo=john'
```
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/cybertalents-therestrictedsessions/2023-06-17 22_37_17-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>Flag via cURL</i></center>
<br/>

* Using Firefox browser and Developer Tools (Storage):
<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/cybertalents-therestrictedsessions/2023-06-17 22_38_25-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>Flag via Firefox Dev Tools</i></center>
<br/>