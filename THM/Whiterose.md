# Whiterose

## namp scan

```
nmap -sCV 10.10.210.43
```

```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-18 15:04 EDT
Nmap scan report for 10.10.210.43
Host is up (0.26s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b9:07:96:0d:c4:b6:0c:d6:22:1a:e4:6c:8e:ac:6f:7d (RSA)
|   256 ba:ff:92:3e:0f:03:7e:da:30:ca:e3:52:8d:47:d9:6c (ECDSA)
|_  256 5d:e4:14:39:ca:06:17:47:93:53:86:de:2b:77:09:7d (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.51 seconds

```

![image](https://github.com/user-attachments/assets/9ab68010-8435-40a1-af8e-43bad67a3d1e)


---

```
sudo nano /etc/hosts
```

![image](https://github.com/user-attachments/assets/e45cefd2-30bc-4f32-8a44-eb8cb491545c)


## gobuster

```
ffuf -u http://FUZZ.cyprusbank.thm/ -w /home/kali/Downloads/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```

```
http://admin.cyprusbank.thm
```

![image](https://github.com/user-attachments/assets/97afe100-bade-4262-a6e5-ca157ff13bc9)


login with:

```
Olivia Cortez:olivi8
```


## idor

![image](https://github.com/user-attachments/assets/245feb1e-5493-49c9-89b0-fb655378a106)

on :

```
http://admin.cyprusbank.thm/messages/?c=0
```


![image](https://github.com/user-attachments/assets/411152ee-9f74-4142-8613-ecb0ec2872e4)




---

login with :

```
Gayle Bev:p~]P@5!6;rs558:q
```














