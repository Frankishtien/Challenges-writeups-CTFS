# Lian_Yu


## `Nmap scan`

```ruby
nmap -sCV 10.10.4.166
```

```ruby
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-09 09:38 EDT
Nmap scan report for 10.10.4.166
Host is up (0.11s latency).
Not shown: 996 closed tcp ports (reset)
PORT    STATE SERVICE VERSION
21/tcp  open  ftp     vsftpd 3.0.2
22/tcp  open  ssh     OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)
| ssh-hostkey: 
|   1024 56:50:bd:11:ef:d4:ac:56:32:c3:ee:73:3e:de:87:f4 (DSA)
|   2048 39:6f:3a:9c:b6:2d:ad:0c:d8:6d:be:77:13:07:25:d6 (RSA)
|   256 a6:69:96:d7:6d:61:27:96:7e:bb:9f:83:60:1b:52:12 (ECDSA)
|_  256 3f:43:76:75:a8:5a:a6:cd:33:b0:66:42:04:91:fe:a0 (ED25519)
80/tcp  open  http    Apache httpd
|_http-title: Purgatory
|_http-server-header: Apache
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          39824/udp6  status
|   100024  1          47584/tcp   status
|   100024  1          49710/tcp6  status
|_  100024  1          59997/udp   status
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.60 seconds
                                                                                                                  
```


- **`ftp`**
- **`ssh`**
- **`http`**
- **`rpcbind`**


--------

```
http://10.10.4.166/
```

<img width="1919" height="796" alt="image" src="https://github.com/user-attachments/assets/49bf6640-a239-41a8-b725-9d5e3b2cacd9" />


## gobuster

```
gobuster dir -u http://10.10.4.166/ \
-w /home/kali/Downloads/wordlists/directory-list-2.3-medium.txt \            
-x php,txt,html -k -t 50

```

<img width="882" height="453" alt="image" src="https://github.com/user-attachments/assets/8365f9af-4122-4777-bbbb-bc064af4ab14" />

```
http://10.10.4.166/island/
```

<img width="1169" height="374" alt="image" src="https://github.com/user-attachments/assets/cc6ecc9d-e47a-4c65-8a43-be856493720f" />

> view page sourse

<img width="765" height="293" alt="image" src="https://github.com/user-attachments/assets/1b786874-bd49-4d09-bf80-af0da8641ff1" />

> it might be password or something

```
vigilante
```






