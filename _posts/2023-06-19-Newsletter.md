---
title: CyberTalents | Newsletter
description: Writeup of an easy-rated Web Exploitation Challenge from CyberTalents
date: 2023-06-19 12:00:00 -500
categories: [CyberTalents]
tags: [Web Exploitation, Command Injection, Commix, Burpsuite]
pin: true
math: true
mermaid: true
image:
  path: /assets/img/cybertalents-TheRestrictedSessions/cybertalents.png
---

***

<center><strong><font color="DarkGray"><a href="https://cybertalents.com/challenges/web/newsletter" target="_blank"><er>Newsletter</er></a> is a Web Exploitation challenge from CyberTalents that revolves around exploiting a command injection vulnerability with a bypass of a rudimentary input check to obtain the flag.</font></strong></center>

***

## **<strong><font color="Brown">Challenge Name</font></strong>**
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/cybertalents-newsletter/2023-06-21 02_33_46-Newsletter » CyberTalents — Mozilla Firefox.png" alt="">
<center><i>Newsletter Challenge</i></center>
<br/>

## **<strong><font color="Brown">Challenge Description</font></strong>**
***
* The administrator put the backup file in the same root folder as the application, help us download this backup by retrieving the backup file name.

## **<strong><font color="Brown">Write-Up</font></strong>**
***

Navigating to the provided URL, you will be presented with the following web page:
<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/cybertalents-newsletter/2023-06-17 23_53_14-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>Super Newsletter</i></center>
<br/>
There is a field, where we can enter an email address:
<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/cybertalents-newsletter/2023-06-17 23_54_30-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>Valid email address</i></center>
<br/>
But, when an input that does not look like an email address, the web page will return the following response:
<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/cybertalents-newsletter/2023-06-17 23_56_04-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>Invalid email address</i></center>

* From the message `Invalid email, email must contain @ & dot`, we can assume that the user input is checked whether it contains `@` and `dot` or `.` If it's the case, the email address is successfully inserted, otherwise this error message is returned.

### **<strong><font color="DarkCyan">Manual Exploitation</font></strong>**
#### **<strong><font color="DarkKhaki">Bypassing the check:</font></strong>**
We can easily bypass this check by simply providing an input that contains `@` and `.` 
<br/>
It can be as simple as the following input: `@.`
<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/cybertalents-newsletter/2023-06-18 00_00_52-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>Email Check Bypassed</i></center>
<br/>

#### **<strong><font color="DarkKhaki">Command Injection:</font></strong>**
Let's try to inject an OS command like `id` while making sure to bypass the check. 
<br/>
We can do that by injecting the following payload: `;id #@.`
* Here we are executing the `id` command and commenting `@.` in order not to be executed.
* The semi-colon `;` in the beginning is just to separate our command with the command executed in the backend.
<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/cybertalents-newsletter/2023-06-18 00_07_01-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>id</i></center>

* As you can see, the `id` command has been executed in the backend server, which means this entry point is vulnerable to command injection.

#### **<strong><font color="DarkKhaki">Flag:</font></strong>**
In this challenge, the flag is the name of a backup file, which is located in the same root folder as the application. In other words, we can get this filename by simply running `ls`:
<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/cybertalents-newsletter/2023-06-18 00_10_27-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>Flag</i></center>

* The backup filename is `hgdr64.backup.tar.gz`

### **<strong><font color="DarkCyan">Exploitation using Commix:</font></strong>**
[Commix](https://github.com/commixproject/commix) short for [``comm``]and [`i`]njection e[`x`]ploiter, is an open-source penetration testing tool that automates the detection and exploitation of `Command Injection` vulnerabilities. 
<br/>
You can think of it as `SQLmap` but for `Command Injection` rather than `SQL injection` vulnerabilities.

#### **<strong><font color="DarkKhaki">The Wizard Mode:</font></strong>**
You can work with this tool by providing options via arguments, such as ``--url``, or using the ``wizard`` mode, which is an interactive interface that asks you for the required option and all you need to do is provide it.
You can enable this mode using the option `--wizard`:
<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/cybertalents-newsletter/2023-06-18 00_26_24-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>Command Injection via Commix</i></center>

* First of all, it will ask you for the URL, the ``POST`` data you want to test for command injection (`email=hacker@gmail.com`), the injection level (`3`) and finally the target server's operating system.
* After doing its thing, if successful (Command Injection exploited), it will ask you whether you want a pseudo-terminal shell where you can execute commands remotely on the target server.