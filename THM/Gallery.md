# Gallery

## **`Nmap`** Scan

```
nmap -sCV 10.10.128.82
```

```ruby
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-19 13:52 EST
Nmap scan report for 10.10.128.82
Host is up (0.10s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 2b:5e:fb:12:37:8a:e4:b6:f6:82:da:c9:14:7a:c3:e1 (RSA)
|   256 64:1f:c3:12:42:11:ca:f4:1e:22:41:ac:29:21:68:ef (ECDSA)
|_  256 5f:42:0d:ab:e7:30:6c:8d:bd:d4:05:4d:0c:87:00:a9 (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.41 (Ubuntu)
8080/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-title: Simple Image Gallery System
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.25 seconds

```

## Gobuster

```ruby
gobuster dir -u http://10.10.128.82/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

<img width="1004" height="357" alt="image" src="https://github.com/user-attachments/assets/de9db1b1-081d-439e-9c00-4a783f573cab" />


```
http://10.10.128.82/gallery/login.php
```



























