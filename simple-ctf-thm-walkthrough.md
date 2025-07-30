---
title: Simple CTF THM Walkthrough
date: 2025-07-01 17:00 UTC
categories: [Cyber Security , Ethical Hacking,CTF]
author: MUhammad Yaseen Taha
---

## My first CTF
I’ve been learning ethical hacking for almost three months now, and the journey so far has been packed with late nights, terminal, and “aha!” moments. This post isn’t about my very first CTF — I’ve done a few before — but rather a detailed breakdown of a recent one I tackled.


## Simple CTF
![Desktop View](/assets/first_ctf.png)

Today, we're diving into a Linux machine in a Capture The Flag (CTF) style challenge.
This exercise involves not only gaining initial access but also performing privilege escalation to reach root-level access.

As the machine’s name suggests, it’s built for hands-on learning, with a series of questions and hidden flags to discover along the way.

To begin the enumeration process, I ran a basic Nmap scan to identify open ports and services running on the target machine.

```bash
$ nmap -sC -sV TARGET_IP
```
![Desktop View](/assets/nmap_scan.png)

*The Nmap scan revealed two open ports under 1000 — port 21 (FTP) and port 80 (HTTP).*

*Q1. How many services are running under port 1000?*

*A1. 2*

*As you can see, SSH is running on port 2222.*

*Q2 : What is running on the higher port?*

*A2: ssh*

Based on the Nmap scan, port 80 is open, indicating that a web server is running. To further enumerate the web application, I used Gobuster with the directory buster wordlist to identify any hidden directories on the site.

```bash
$ : gobuster dir -u http://MACHINE-IP/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
```
![Desktop View](/assets/gobuster.png)

As you can see, I discovered a page at /simple. Let's check it out. After exploring the website further, I came across a page that revealed the CMS version: 2.2.8. A quick Google search revealed that this version has known vulnerabilities.

![Desktop View](/assets/cms.png)

*Q3: What's the CVE you're using against the application?*

*A3: CVE-2019-9053*

###
An issue was discovered in CMS Made Simple 2.2.8. It is possible with the News module, through a crafted URL, to achieve unauthenticated blind time-based SQL injection via the m1_idlist parameter.

After a bit of searching, I found this.

```bash
$ searchsploit cms made simple
```
![Desktop View](/assets/searchsploit.png)

I came across the file php/webapps/46635.py. Let's take a closer look at what it does.

```bash
$ python 46635.py -u http://TARGET-IP/simple/ --crack -w /usr/share/wordlists/rockyou.txt
```

After resolving a few Python-related errors, I was able to run the script successfully. The result was as follows:

![Desktop View](/assets/webapps.png)

*Q4: What's the password?*
*A4: secret*

Let’s attempt to log in to the server via SSH.
```bash
$ ssh mitch@TARGET-IP -p 2222
```

![Desktop View](/assets/ssh.png)

And here we go — we've successfully retrieved the user.txt flag.

*Q5: What's the user flag?*

*A5: G00d j0b, keep up!*


![Desktop View](/assets/other_user.png)
*Q6: Is there any other user in the home directory? What's its name?*

*A6:sunbath*


After logging in as Mitch, I checked the available sudo permissions.
### Gaining Root Access
If appropriate permissions are available, root access can be obtained using a tool like vim

```bash
$ sudo -l
$ sudo vim -c ":!/bin/sh"
$ id
$ cat /root/root.txt
```
![Desktop View](/assets/root_ganing.png)

*Q7: What can you leverage to spawn a privileged shell?*

*A7:vim*

*Q8: What's the root flag?*

*A8:W3ll d0n3. You made it!*

As this was my first CTF and first blog post, it was both a learning journey and a rewarding accomplishment. I’m excited to continue exploring more machines and sharing what I learn along the way.

Thanks for reading — and happy hacking! 🐚💻
