# Overpass

## Nmap scan 

```bash
nmap -sCV 10.10.197.109
```

```sql
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-21 19:56 EDT
Nmap scan report for 10.10.197.109
Host is up (0.22s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 37:96:85:98:d1:00:9c:14:63:d9:b0:34:75:b1:f9:57 (RSA)
|   256 53:75:fa:c0:65:da:dd:b1:e8:dd:40:b8:f6:82:39:24 (ECDSA)
|_  256 1c:4a:da:1f:36:54:6d:a6:c6:17:00:27:2e:67:75:9c (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Overpass
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.78 seconds

```


---

```
http://10.10.197.109/
```

![image](https://github.com/user-attachments/assets/b95c3937-51a7-4f15-9c6d-840e60751811)


## gobuster

```
gobuster dir -u http://10.10.197.109/ -w /home/kali/Downloads/wordlists/directory-list-2.3-medium.txt
```


![image](https://github.com/user-attachments/assets/4c4a4d71-b7d0-4a0d-aff3-801fc2334eb1)





















