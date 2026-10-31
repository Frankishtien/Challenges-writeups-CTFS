# Takeoff

<img width="1602" height="286" alt="image" src="https://github.com/user-attachments/assets/0061e55f-ab7e-4e2a-8502-17e92b2336c5" />

---

## port scanning 


```ruby
─$ nmap -sCV -Pn 10.129.86.21    
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-31 21:45 UTC
Nmap scan report for 10.129.86.21
Host is up (0.13s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.0p2 Debian 7+deb13u4 (protocol 2.0)
80/tcp open  http    nginx
|_http-title: Book a Seat, See the World - HTB Airways Booking
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.86 seconds
                                                                                                                                                                                              
┌──(kali㉿kali)-[~]

```

---


## enumrate the website


<img width="1919" height="802" alt="image" src="https://github.com/user-attachments/assets/0c4df813-28c2-4f3b-904d-7a5dd2cde477" />

```
feroxbuster -u http://10.129.86.21 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

<img width="1744" height="629" alt="image" src="https://github.com/user-attachments/assets/413e4741-9fbc-4c6a-be16-88c531a2b070" />



## i created account and logged in 

<img width="1919" height="758" alt="image" src="https://github.com/user-attachments/assets/86e5c813-4ca7-4374-8967-179fa5aebcfa" />










