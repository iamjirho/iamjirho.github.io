---
title: TryHackMe Lian_Yu [CHALLENGE]
date: 2024-05-08 00:00:00 +0800
categories: [TryHackMe, Challenge]
tags: [walkthough,write-ups]     # TAG names should always be lowercase
image:
    path: https://tryhackme-images.s3.amazonaws.com/room-icons/c72d580db69a726dfb8da8aa6eaa2f5a.jpeg
---

A beginner level security challenge

## **Skills Needed**
Steganography, Web enumeration, Network Services Exploitation, SSH, FTP, Privilege escalation


## **Reconnaissance**
---
Initially do the Nmap scan.

```bash
nmap -sC -sV -p- $IP --open
```
`--open`: Scans only the open ports and reduces the time it takes to complete the nmap scan.


```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-01 23:00 EDT
Nmap scan report for 10.10.223.26
Host is up (0.31s latency).
Not shown: 65530 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.2
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)
| ssh-hostkey: 
|   1024 56:50:bd:11:ef:d4:ac:56:32:c3:ee:73:3e:de:87:f4 (DSA)
|   2048 39:6f:3a:9c:b6:2d:ad:0c:d8:6d:be:77:13:07:25:d6 (RSA)
|   256 a6:69:96:d7:6d:61:27:96:7e:bb:9f:83:60:1b:52:12 (ECDSA)
|_  256 3f:43:76:75:a8:5a:a6:cd:33:b0:66:42:04:91:fe:a0 (ED25519)
80/tcp    open  http    Apache httpd
|_http-server-header: Apache
|_http-title: Purgatory
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          35052/tcp   status
|   100024  1          36628/udp6  status
|   100024  1          46049/tcp6  status
|_  100024  1          53710/udp   status
35052/tcp open  status  1 (RPC #100024)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 127.84 seconds

```

## **Insights and To-TRY List**
---
OS: Debian

TARGET IP: 10.10.223.26

80 → HTTP

- Arrowverse theme page
- http[:]//10.10.223.26/island/ - Contains a note with a hidden text that is revealed when highlighted.
    - Found a code word “vigilante”
- There is also .jpg file
- /2100 - Has a clue that points out to a file name with .ticket extension
- http[:]//10.10.223.26/island/2100/green_arrow.ticket - Contains base58 encoded password for FTP


21 → FTP

- vigilante:!#th3h00d
- Found another username called slade
- Found some .jpg and .png files which can possibly contained hidden credentials
- These images contains password for SSH

22 → SSH

- slade:M3tahuman

111 → Portmapper

## **Enumeration**
---
### 80 -> HTTP

**Viewing the webpage.**

![Desktop View](/assets/images/lian_yu/homepage.png){: width="972" height="589" }

**Using gobuster**

```go
gobuster dir -u http://10.10.223.26 -w /opt/SecLists/Discovery/Web-Content/raft-large-directories.txt -k -t 30
```

```bash
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.223.26
[+] Method:                  GET
[+] Threads:                 30
[+] Wordlist:                /opt/SecLists/Discovery/Web-Content/raft-large-directories.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/server-status        (Status: 403) [Size: 199]
/island               (Status: 301) [Size: 235] [--> http://10.10.223.26/island/]
Progress: 62284 / 62285 (100.00%)

```
There is a word “vigilante” that can be revealed when highlighted. I wonder how can I use this word to gain an initial foothold.

Visiting the /island directory endpoint

![Desktop View](/assets/images/lian_yu/island.png){: width="972" height="589" }

Looks like there is a hidden text on this webpage. It so happen because the word has been set to color white which blends in to the white background.

![Desktop View](/assets/images/lian_yu/hidetext.png){: width="972" height="589" }

I ran another directory search  for /island endpoint to make sure that I am not missing something hidden before proceeding to brute-forcing either FTP or SSH.

```bash
gobuster dir -u http://10.10.223.26/island/ -w /opt/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt -k -t 30
```
```bash
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.223.26/island/
[+] Method:                  GET
[+] Threads:                 30
[+] Wordlist:                /opt/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/2100                 (Status: 301) [Size: 240] [--> http://10.10.223.26/island/2100/]

```

Welp! It looks like we got another hit! Let us try visiting the /2100 endpoint.

![Desktop View](/assets/images/lian_yu/2100.png){: width="972" height="589" }

Does that mean another directory busting?

```bash
gobuster dir -u http://10.10.223.26/island/2100 -w /opt/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -k -t 30 -x ticket
```

```bash
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.223.26/island/2100
[+] Method:                  GET
[+] Threads:                 30
[+] Wordlist:                /opt/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              ticket
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/green_arrow.ticket   (Status: 200) [Size: 71]

```

![Desktop View](/assets/images/lian_yu/ticket.png){: width="972" height="589" }

Looks like a password to me! Maybe we can pair this up to the hidden word that we just found earlier and login via FTP or SSH.

`vigilante`:`RTy8yhBQdscX`

### 21 -> FTP

![Desktop View](/assets/images/lian_yu/ftp.png){: width="972" height="589" }

So the hidden word “vigilante” is FTP’s username. However, I was not able to login after trying to enter `RTy8yhBQdscX` as the password. It turns out, that upon trying some trial and error, these random character have been encoded using `base58`

![Desktop View](/assets/images/lian_yu/base58.png){: width="972" height="589" }

```bash
!#th3h00d
```

Now let us try to log in again!

```bash
ftp $IP
vigilante
!#th3h00d
```

![Desktop View](/assets/images/lian_yu/ftp_login.png){: width="972" height="589" }

Let’s enumerate what this FTP contains and download them to our attacking machine. Do not forget to us `ls -la` to list the content of the FTP.

```bash
get <FILENAME>
```

![Desktop View](/assets/images/lian_yu/ftpget.png){: width="972" height="589" }

We have also found out that there is another user called `slade`.

![Desktop View](/assets/images/lian_yu/slade.png){: width="972" height="589" }


## **Gaining an Initial Foothold**

Now that we have these images, I think our next step would be to try extracting hidden information if there’s any.

![Desktop View](/assets/images/lian_yu/examine.png){: width="972" height="589" }

From the looks of it, `Leave_me_alone.png` is corrupted. That’s pretty intriguing.

From what we know magic numbers are typically not visible to the user but can be seen by using a hex editor or by using the `xxd` command. Most importantly, these bytes are essential for a file to be opened. Changing/corrupting these bytes will render the file useless as most tools will not access these files due to potential damaging.

For a PNG file to be fixed, the file signature has to be `89 50 4E 47 0D 0A 1A 0A`

![Desktop View](/assets/images/lian_yu/img_sig.png){: width="972" height="589" }

See more: [https://en.wikipedia.org/wiki/List_of_file_signatures](https://en.wikipedia.org/wiki/List_of_file_signatures)

To view and verify this, we can use `xxd` tool via the terminal.

```bash
xxd <FILENAME> | head
```
![Desktop View](/assets/images/lian_yu/xxd_leave.png){: width="972" height="589" }

![Desktop View](/assets/images/lian_yu/xxd_lian.png){: width="972" height="589" }

Upon inspecting both `Lianyu.png` and `Queen’s_Gambit.png` has the same magic number while `Leave_me_alone.png` seems to be the only one with different header rendering the file to be corrupted.

To edit the header, we can use the following command

```bash
hexedit <FILENAME>
```

Afterwhich, we can now view the password needed for aa.jpg file

![Desktop View](/assets/images/lian_yu/pw.png){: width="972" height="589" }

Reference: [https://www.geeksforgeeks.org/working-with-magic-numbers-in-linux/](https://www.geeksforgeeks.org/working-with-magic-numbers-in-linux/)

Alternatively, you can also use `stegseek` to bruteforce the password for `aa.jpg` file if you find editing the magic number a bit difficult.

```bash
stegseek --crack <FILENAME> /usr/share/wordlists/rockyou.txt
```
![Desktop View](/assets/images/lian_yu/stegseek.png){: width="972" height="589" }

From here, we also found out that the original file is ss.zip in which we can be able to extract.

```bash
steghide extract -sf aa.jpg 
```
![Desktop View](/assets/images/lian_yu/steghide.png){: width="972" height="589" }

Let's unzip it

```bash
unzip ss.zip
```

![Desktop View](/assets/images/lian_yu/unzip.png){: width="972" height="589" }

Viewing the `passwd.txt`

![Desktop View](/assets/images/lian_yu/passwdtxt.png){: width="972" height="589" }

Viewing the `shado` file

![Desktop View](/assets/images/lian_yu/shado.png){: width="972" height="589" }

```bash
M3tahuman
```

Now that looks like a password. Lets try logging in via SSH

```bash
ssh slade@<IP>
```
![Desktop View](/assets/images/lian_yu/sshlogin.png){: width="972" height="589" }

> Do not forget to find the flag
{: .prompt-danger }

## **Privilege Escalation**
---
I initially check `sudo -l` command to list the permissions granted to the current user in the sudoers file. Fortunately, the user we are currently logged-in has a sudo privilege fro `pkexec`.

```bash
sudo -l
```
![Desktop View](/assets/images/lian_yu/pkexec.png){: width="972" height="589" }

`pkexec` allows an authorized user to execute PROGRAM as another user and we can use `gtfobin` to escalate our privilege.

![Desktop View](/assets/images/lian_yu/gtfobin.png){: width="972" height="589" }

And we have a ROOT!

![Desktop View](/assets/images/lian_yu/root.png){: width="972" height="589" }

> Do not forget to find the root flag
{: .prompt-danger }