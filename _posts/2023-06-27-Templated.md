---
title: HackTheBox | Templated
description: Writeup of a easy-rated Web Exploitation Challenge from HackTheBox
date: 2023-06-27 12:00:00 -500
categories: [Challenges, HTB]
tags: [Web Exploitation, SSTI, RCE, Werkzeug, Flask, Jinja2]
pin: true
math: true
mermaid: true
image:
  path: /assets/img/htb-templated/HTB.jpg
---

***

<center><strong><font color="DarkGray"><a href="https://app.hackthebox.com/challenges/templated" target="_blank"><er>Templated</er></a> is a Web Exploitation challenge from HackTheBox that revolves around exploiting a Server-Side Template Injection vulnerability in order to achieve remote code execution on the server and eventually get the flag</font></strong></center>

***

## **<strong><font color="Brown">Challenge Name</font></strong>**
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-templated/2023-06-28 01_12_53-Hack The Box __ Hack The Box — Mozilla Firefox.png" alt="">
<center><i>Templated Challenge</i></center>
<br/>

## **<strong><font color="Brown">Challenge Description</font></strong>**
***
* Can you exploit this simple mistake?

## **<strong><font color="Brown">Write-Up</font></strong>**
***
### **<strong><font color="DarkCyan">Enumeration:</font></strong>** 
#### **<strong><font color="MediumPurple">Front page:</font></strong>**
Navigating to the provided docker instance (``144.126.206.23:32088``), you will be presented with the following web page:
<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-templated/2023-06-27 22_28_04-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>Front page</i></center>

* The website's front page is pretty empty as it primarily consists of text without any interactive elements such as buttons, logos, or clickable features.
* However, the front page reveals the framework and templating engine being used in the backend of this application, which is `Flask/Jinja2`

#### **<strong><font color="MediumPurple">Source code:</font></strong>**
The source code of the page doesn't contain any hidden secrets, such as developer's comments or hidden directories.
<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-templated/2023-06-27 22_32_43-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>Source code</i></center>

#### **<strong><font color="MediumPurple">Server's response:</font></strong>**
Taking a look at the server's response, the `Server` HTTP header discloses the use of `Werkzeug/1.0.1` and  `Python/3.9.0` is used in conjunction with `Flask` as the backend infrastructure.
<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-templated/2023-06-27 22_50_04-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>Server Header</i></center>

#### **<strong><font color="MediumPurple">Werkzeug's console:</font></strong>**
``Werkzeug`` is a comprehensive WSGI (``Web Server Gateway Interface``) utility library for Python. It serves as a foundation for developing web applications, and is often used in conjunction with the web frameworks like `Flask` to handle the level-level aspects of web development. <br/>
``Werkzeug`` provides a development server called `Werkzeug's Debugger` that includes an interactive console known as the `interactive debugger console`. This console allows developers to execute Python code and inspect variables during the debugging process. <br/>
Long story short, if debug mode is enabled, we can access the `/console` endpoint and easily gain `RCE - Remote Code Execution` on the server running the vulnerable web application.

#### **<strong><font color="MediumPurple">Detecting the vulnerability:</font></strong>**
Unfortunately, the debug mode is disabled on the server. However, navigating to `/console` does not return a `404 - NOT FOUND` status-code, rather it returns a `200 FOUND`, which means `404` pages are probably being templated.
<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-templated/2023-06-27 23_12_10-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>Templated 404 pages</i></center>

* As you can see, the web application is reflecting the name of the directory or file we want to access, such as in this example `console`.
* This might lead to an `SSTI - Server-Side Template Injection` vulnerabilities, if user input passed via the `URL` is not properly sanitized.

### **<strong><font color="DarkCyan">Exploitation:</font></strong>** 
#### **<strong><font color="MediumPurple">Remote Code Execution:</font></strong>**
Since we know that the server is using Flask, which is a Python library, and we can leverage the `MRO - Method Resolution Order` to traverse upwards in the `request` library within **Flask**. This allows us to import the `os` library. <br/>
Once we have access to the `os` library, we can execute OS commands on the server hosting the web application.<br/>

```python
{% raw %}{{request.application.__globals__.__builtins__.__import__('os').popen('id').read()}}{% endraw %}
```

<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-templated/2023-06-27 23_31_12-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>id command</i></center>
<br/>
Since we already know the templating engine in use, which is `Jinja2`, we can leverage Jinja2-specific expressions like the following:

```python
{% raw %}{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read()}}{% endraw %}
```
<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-templated/2023-06-28 00_31_10-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>id command</i></center>

* As you can see, this jinja2-specific expression uses the `os` module to execute the `id` command.

#### **<strong><font color="MediumPurple">Flag:</font></strong>**
Now that we have remote code execution on the server, we can execute the `ls` command to list the files and directories located in the current directory:

```python
{% raw %}{{ self.__init__.__globals__.__builtins__.__import__('os').popen('ls').read()}}{% endraw %}
```

<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-templated/2023-06-28 00_38_39-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>ls command</i></center>

* As you can see, there is text file named `flag.txt`

Let's read the content of `flag.txt` file, by injecting the following payload:

```python
{% raw %}{{ self.__init__.__globals__.__builtins__.__import__('os').popen('cat flag.txt').read()}}{% endraw %}
```

<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-templated/2023-06-28 00_40_33-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>Flag</i></center>
