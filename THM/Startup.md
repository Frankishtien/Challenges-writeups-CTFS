# Startup


## Nmap scan

```ruby
nmap -sCV 10.10.246.53
```

```ruby
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-10 10:21 EDT
Stats: 0:00:27 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.77% done; ETC: 10:21 (0:00:00 remaining)
Nmap scan report for 10.10.246.53
Host is up (0.60s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp [NSE: writeable]
| -rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
|_-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.8.47.102
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b9:a6:0b:84:1d:22:01:a4:01:30:48:43:61:2b:ab:94 (RSA)
|   256 ec:13:25:8c:18:20:36:e6:ce:91:0e:16:26:eb:a2:be (ECDSA)
|_  256 a2:ff:2a:72:81:aa:a2:9f:55:a4:dc:92:23:e6:b4:3f (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Maintenance
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 36.60 seconds

```

- **`Ftp`**
- **`ssh`**
- **`http`**


---

## Ftp

```
ftp 10.10.246.53
```

<img width="1268" height="744" alt="image" src="https://github.com/user-attachments/assets/51dd765c-ab3a-4ffc-bbbe-55fad6e2171c" />


<img width="480" height="148" alt="image" src="https://github.com/user-attachments/assets/2ef55cde-a1a0-4953-bb27-4879fbe47a18" />


<img width="1491" height="675" alt="image" src="https://github.com/user-attachments/assets/87c4f200-bbfd-48de-8288-66734a120094" />




