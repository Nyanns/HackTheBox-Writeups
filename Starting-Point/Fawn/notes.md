# Fawn

## Basic Information

- Machine : Fawn
- Difficulty : Very Easy
- OS : Linux
- IP : 10.129.213.246

---

# Objective

Belajar dasar penggunaan FTP (File Transfer Protocol), melakukan anonymous login, melihat isi direktori server, dan mengunduh file.

---

# Enumeration

## 1. Ping

```bash
ping -c 4 10.129.213.246
```

Target dapat dijangkau.

---

## 2. Nmap

```bash
nmap -Pn -sV 10.129.213.246
```

Hasil:

```
21/tcp open  ftp  vsftpd 3.0.3
```

Informasi yang diperoleh:

- Host aktif
- Port 21 terbuka
- Service FTP
- Software vsftpd 3.0.3
- OS terdeteksi sebagai Unix

---

# FTP Enumeration

Masuk ke FTP.

```bash
ftp 10.129.213.246
```

Login:

```
Username : anonymous
Password : (kosong)
```

Server mengizinkan Anonymous Login.

---

# FTP Commands

Melihat isi folder

```ftp
ls
```

Melihat direktori lokal

```ftp
lpwd
```

Mengunduh file

```ftp
get flag.txt
```

Keluar

```ftp
bye
```

---

# FTP Response Codes

| Code | Arti |
|------|------|
| 220 | FTP Server Ready |
| 331 | Password Required |
| 230 | Login Successful |
| 150 | Starting File Transfer |
| 226 | Transfer Complete |

---

# Flag

```
035db21c881520061c53e0536e44f815
```

---

# Yang Dipelajari

- FTP = File Transfer Protocol.
- FTP menggunakan port 21.
- FTP digunakan untuk upload dan download file.
- Anonymous Login memungkinkan pengguna masuk tanpa akun.
- Nmap dapat mengidentifikasi service dan versinya.
- FTP mempunyai response code seperti HTTP.

---

# Kendala

Command

```ftp
get flag.txt
```

gagal pada FTP client `tnftp` di Ubuntu terbaru dengan pesan:

```
Permission denied
```

Sebagai alternatif berhasil menggunakan:

```bash
curl ftp://anonymous:@10.129.213.246/flag.txt
```

atau

```bash
curl ftp://anonymous:@10.129.213.246/flag.txt -o flag.txt
```

---

# Lessons Learned

- Selalu lakukan enumerasi sebelum mencoba login.
- Jangan terpaku pada satu tool.
- Fokus pada protokol, bukan implementasi client.
- Jika satu tool gagal, verifikasi penyebabnya sebelum berpindah ke tool lain.