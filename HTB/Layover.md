# Layover


<img width="1589" height="281" alt="image" src="https://github.com/user-attachments/assets/8b9baa28-bf7b-4170-8a35-83536905d4d8" />

---


## Nmap Scan 

```ruby
└─$ nmap -sCV -Pn 10.129.140.165
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-26 19:59 UTC
Nmap scan report for 10.129.140.165
Host is up (0.40s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH 9.6p1 Ubuntu 3ubuntu13.19 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
3389/tcp open  ms-wbt-server Microsoft Terminal Service
Service Info: OSs: Linux, Windows; CPE: cpe:/o:linux:linux_kernel, cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 44.29 seconds
                                                                          
```


----

> ## note in challange

<img width="1588" height="339" alt="image" src="https://github.com/user-attachments/assets/63f89b27-4878-4ee6-9a60-7c1049e02732" />



## Connect via RDP

```
rdesktop -u contractor -p 'Contractor2026!' 10.129.140.165
```

<img width="1858" height="854" alt="image" src="https://github.com/user-attachments/assets/2bf613c1-9901-46f4-984f-ecc4742cbdc5" />



```
id
sudo -l
sudo su -
```


<img width="837" height="377" alt="image" src="https://github.com/user-attachments/assets/385dd6ab-108d-41a6-9ae9-741db13be41d" />

> ## what ?!!! I get root ???
> it's fucken trap i can't find flags 

















