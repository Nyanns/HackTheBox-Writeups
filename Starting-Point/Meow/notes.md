# Meow

## Difficulty
Very Easy

## Target
10.129.209.185

## OS
Linux (Ubuntu 20.04)

## Enumeration

### Ping

Target dapat dijangkau.

### Nmap

```bash
nmap -Pn -sV 10.129.209.185
```

Hasil:

- Port 23 terbuka
- Service: Telnet

## Login

```bash
telnet 10.129.209.185
```

Username:

```
root
```

Password:

```
(kosong)
```

## Flag

Berhasil membaca `/root/flag.txt`.

## Konsep yang Dipelajari

- IP Address
- Port
- TCP
- Nmap
- Service Enumeration
- Telnet
- Root User
- Home Directory
- `ls`
- `cat`

## Analogi

- IP = Gedung
- Port = Pintu
- Service = Ruangan di balik pintu
- Nmap = Orang yang mengetuk semua pintu
- Telnet = Telepon tanpa enkripsi
- SSH = Telepon yang percakapannya dienkripsi

## Yang Harus Diingat

- Jangan langsung eksploitasi.
- Enumerasi selalu langkah pertama.
- Pahami output tool, jangan hanya menjalankannya.
