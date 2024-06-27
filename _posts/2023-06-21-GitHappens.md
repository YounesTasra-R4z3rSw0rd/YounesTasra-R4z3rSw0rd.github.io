---
title: TryHackMe | Git Happens
description: Writeup of a easy-rated Linux Machine from TryHackMe
date: 2023-06-20 12:00:00 -500
categories: [TryHackMe, Easy]
tags: [Web Exploitation, Git Repository, Sensitive Data Exposure, Information Disclosure, git]
pin: true
math: true
mermaid: true
image:
  path: /assets/img/thm-githappens/gitHappens.png
---

***

<center><strong><font color="DarkGray"><a href="https://tryhackme.com/room/githappens" target="_blank"><er>Git Happens</er></a> is a vulnerable machine from TryHackMe where there is a web server running on port 80, which has an exposed .git folder, allowing the access to the source code of the application and ultimately gain access to the super secret password of the admin user.</font></strong></center>

***

## **<strong><font color="Brown">Reconnaissance</font></strong>**
***
### **<strong><font color="DarkCyan">Scanning</font></strong>**

As always, let's start our recon process by scanning the machine with ```Nmap```:
```bash
nmap -sC -sV -A -T4 -p- 10.10.10.10
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-20 14:38 EDT
Nmap scan report for 10.10.129.187
Host is up (0.11s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.0 (Ubuntu)
| http-git: 
|   10.10.129.187:80/.git/
|     Git repository found!
|_    Repository description: Unnamed repository; edit this file 'description' to name the...
|_http-title: Super Awesome Site!
|_http-server-header: nginx/1.14.0 (Ubuntu)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 2.6.32 (92%), Linux 3.1 - 3.2 (92%), Linux 3.11 (92%), Linux 3.2 - 4.9 (92%), Linux 3.5 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   114.79 ms 10.8.0.1
2   108.29 ms 10.10.129.187

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.98 seconds
```
* As you can see, ``Nmap`` found only one open open port `80` running an HTTP server `nginx 1.14.0`
* The scan also indicates the presence of a Git repository in `http://10.10.10.10/.git`, which means, we may have access to the source code of the application.

## **<strong><font color="Brown">Service Enumeration</font></strong>**
***
### **<strong><font color="DarkCyan">HTTP - TCP 80</font></strong>**
#### **<strong><font color="MediumPurple">Website</font></strong>**

Navigating to `http://MACHINE_IP`, you will be presented with a login form, where you can enter a username and a password in order to login:
<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/thm-githappens/2023-06-20 20_14_07-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png">
<center><i>login page</i></center>

#### **<strong><font color="MediumPurple">Downloading /.git</font></strong>**
Since we already know the presence of a `Git Repository`, which might contain the source code of the application, in `http://MACHINE_IP/.git`,
<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/thm-githappens/2023-06-20 20_18_24-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png">
<center><i>/.git</i></center>
<br/>

let's download the `.git` folder using the following command:
```bash
wget -r http://MACHINE_IP/.git
```
* This command will download the contents of the Git repository located at `http://MACHINE_IP/.git`, including the source code, commit history and other associated files, by traversing the directory structure within the Git repo and retrieve all accessible files.

<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/thm-githappens/2023-06-20 20_21_34-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png">
<center><i>Downloading .git folder</i></center>

#### **<strong><font color="MediumPurple">Enumerating .git:</font></strong>**
When it comes to `.git` enumeration, the first command that I like to run is:
```bash
git log 
```
* This command will display the commit history in a Git repo, by presenting detailed information about each commit, including the commit hash, author, date and commit message (Comment)

<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/thm-githappens/2023-06-20 20_30_28-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png">
<center><i>git log</i></center>
<br/>

On the other hand, the below command provides a more concise output, by displaying, on a single line, each commit.
```bash
git log --oneline
```
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/thm-githappens/2023-06-20 20_32_11-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png">
<center><i>git log --oneline</i></center>
<br/>

## **<strong><font color="Brown">Super Secret Password:</font></strong>**
Since the goal of this challenge is to find the password to the application, the commit that holds particular interest to us is the one with the message `Made the login page, boss!` and the corresponding hash `395e087`<br/>
To display the differences between the current state of the repository and this particular commit (`395e087`), we can run the following command:
```bash
git diff 395e087
```
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/thm-githappens/2023-06-20 20_43_35-HACKING_MACHINE - VMware Workstation 17 Player (Non-commercial use only).png">
<center><i>git diff</i></center>

* As you can see from this output, some changes has been made to the file `index.html`, including the removal of the JavaScript function `login()` which performs a check on the values of `username` and `password`, stored in plaintext.

#### **<strong><font color="SteelBlue">Flag:</font></strong>**
With that said, we have found the super secret password, and all we need to do is submit it in order to complete the challenge:
<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/thm-githappens/2023-06-21 00_06_48-TryHackMe _ Git Happens — Mozilla Firefox.png">
<center><i>super secret password</i></center>
<br/>
