# Mr.Robot (not-completed) ⚠️


## nmap scan

```
nmap -sV -vv --script vuln 10.10.208.76
```

```java
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-18 07:52 EDT
NSE: Loaded 151 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 07:52
Completed NSE at 07:52, 10.12s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 07:52
Completed NSE at 07:52, 0.00s elapsed
Initiating Ping Scan at 07:52
Scanning 10.10.208.76 [4 ports]
Completed Ping Scan at 07:52, 0.42s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 07:52
Completed Parallel DNS resolution of 1 host. at 07:52, 0.16s elapsed
Initiating SYN Stealth Scan at 07:52
Scanning 10.10.208.76 [1000 ports]
Discovered open port 80/tcp on 10.10.208.76
Discovered open port 443/tcp on 10.10.208.76
Stats: 0:00:38 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 55.75% done; ETC: 07:53 (0:00:21 remaining)
Completed SYN Stealth Scan at 07:52, 38.24s elapsed (1000 total ports)
Initiating Service scan at 07:52
Scanning 2 services on 10.10.208.76
Completed Service scan at 07:53, 13.03s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.208.76.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 07:53
NSE Timing: About 93.68% done; ETC: 07:53 (0:00:02 remaining)
NSE Timing: About 98.25% done; ETC: 07:54 (0:00:01 remaining)
NSE Timing: About 98.95% done; ETC: 07:54 (0:00:01 remaining)
NSE Timing: About 99.30% done; ETC: 07:55 (0:00:01 remaining)
NSE Timing: About 99.30% done; ETC: 07:55 (0:00:01 remaining)
Completed NSE at 07:55, 163.03s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 07:55
Completed NSE at 07:56, 16.08s elapsed
Nmap scan report for 10.10.208.76
Host is up, received echo-reply ttl 63 (0.19s latency).
Scanned at 2025-03-18 07:52:19 EDT for 232s
Not shown: 997 filtered tcp ports (no-response)
PORT    STATE  SERVICE  REASON         VERSION
22/tcp  closed ssh      reset ttl 63
80/tcp  open   http     syn-ack ttl 63 Apache httpd
|_http-litespeed-sourcecode-download: Request with null byte did not work. This web server might not be vulnerable
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-server-header: Apache
| http-csrf: 
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=10.10.208.76
|   Found the following possible CSRF vulnerabilities: 
|     
|     Path: http://10.10.208.76:80/js/BASE_URL
|     Form id: 
|     Form action: http://10.10.208.76/
|     
|     Path: http://10.10.208.76:80/js/BASE_URL
|     Form id: 
|     Form action: http://10.10.208.76/
|     
|     Path: http://10.10.208.76:80/js/u;c.appendChild(o);'+(n?'o.c=0;o.i=setTimeout(f2,100)':'')+'}}catch(e){o=0}return
|     Form id: 
|     Form action: http://10.10.208.76/
|     
|     Path: http://10.10.208.76:80/js/u;c.appendChild(o);'+(n?'o.c=0;o.i=setTimeout(f2,100)':'')+'}}catch(e){o=0}return
|     Form id: 
|     Form action: http://10.10.208.76/
|     
|     Path: http://10.10.208.76:80/js/rs;if(s.useForcedLinkTracking||s.bcf){if(!s."
|     Form id: 
|     Form action: http://10.10.208.76/
|     
|     Path: http://10.10.208.76:80/js/rs;if(s.useForcedLinkTracking||s.bcf){if(!s."
|     Form id: 
|     Form action: http://10.10.208.76/
|     
|     Path: http://10.10.208.76:80/js/vendor/null1this.tags.length10%7D1t.get1function11%7Bif1011this.tags.length1return
|     Form id: 
|     Form action: http://10.10.208.76/
|     
|     Path: http://10.10.208.76:80/js/vendor/null1this.tags.length10%7D1t.get1function11%7Bif1011this.tags.length1return
|     Form id: 
|     Form action: http://10.10.208.76/
|     
|     Path: http://10.10.208.76:80/wp-login.php
|     Form id: loginform
|_    Form action: http://10.10.208.76/wp-login.php
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|   /admin/: Possible admin folder
|   /admin/index.html: Possible admin folder
|   /wp-login.php: Possible admin folder
|   /robots.txt: Robots file
|   /feed/: Wordpress version: 4.3.1
|   /wp-includes/images/rss.png: Wordpress version 2.2 found.
|   /wp-includes/js/jquery/suggest.js: Wordpress version 2.5 found.
|   /wp-includes/images/blank.gif: Wordpress version 2.6 found.
|   /wp-includes/js/comment-reply.js: Wordpress version 2.7 found.
|   /wp-login.php: Wordpress login page.
|   /wp-admin/upgrade.php: Wordpress login page.
|   /readme.html: Interesting, a readme.
|   /0/: Potentially interesting folder
|_  /image/: Potentially interesting folder
|_http-jsonp-detection: Couldn't find any JSONP endpoints.
443/tcp open   ssl/http syn-ack ttl 63 Apache httpd
|_http-server-header: Apache
| http-enum: 
|   /admin/: Possible admin folder
|   /admin/index.html: Possible admin folder
|   /wp-login.php: Possible admin folder
|   /robots.txt: Robots file
|   /feed/: Wordpress version: 4.3.1
|   /wp-includes/images/rss.png: Wordpress version 2.2 found.
|   /wp-includes/js/jquery/suggest.js: Wordpress version 2.5 found.
|   /wp-includes/images/blank.gif: Wordpress version 2.6 found.
|   /wp-includes/js/comment-reply.js: Wordpress version 2.7 found.
|   /wp-login.php: Wordpress login page.
|   /wp-admin/upgrade.php: Wordpress login page.
|   /readme.html: Interesting, a readme.
|   /0/: Potentially interesting folder
|_  /image/: Potentially interesting folder
| http-csrf: 
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=10.10.208.76
|   Found the following possible CSRF vulnerabilities: 
|     
|     Path: https://10.10.208.76:443/js/vendor/null1this.tags.length10%7D1t.get1function11%7Bif1011this.tags.length1return
|     Form id: 
|     Form action: https://10.10.208.76:443/
|     
|     Path: https://10.10.208.76:443/js/vendor/null1this.tags.length10%7D1t.get1function11%7Bif1011this.tags.length1return
|     Form id: 
|     Form action: https://10.10.208.76:443/
|     
|     Path: https://10.10.208.76:443/js/rs;if(s.useForcedLinkTracking||s.bcf){if(!s."
|     Form id: 
|     Form action: https://10.10.208.76:443/
|     
|     Path: https://10.10.208.76:443/js/rs;if(s.useForcedLinkTracking||s.bcf){if(!s."
|     Form id: 
|     Form action: https://10.10.208.76:443/
|     
|     Path: https://10.10.208.76:443/js/u;c.appendChild(o);'+(n?'o.c=0;o.i=setTimeout(f2,100)':'')+'}}catch(e){o=0}return
|     Form id: 
|     Form action: https://10.10.208.76:443/
|     
|     Path: https://10.10.208.76:443/js/u;c.appendChild(o);'+(n?'o.c=0;o.i=setTimeout(f2,100)':'')+'}}catch(e){o=0}return
|     Form id: 
|     Form action: https://10.10.208.76:443/
|     
|     Path: https://10.10.208.76:443/js/BASE_URL
|     Form id: 
|     Form action: https://10.10.208.76:443/
|     
|     Path: https://10.10.208.76:443/js/BASE_URL
|     Form id: 
|     Form action: https://10.10.208.76:443/
|     
|     Path: https://10.10.208.76:443/wp-login.php
|     Form id: loginform
|_    Form action: https://10.10.208.76:443/wp-login.php
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-litespeed-sourcecode-download: Request with null byte did not work. This web server might not be vulnerable
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-jsonp-detection: Couldn't find any JSONP endpoints.

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 07:56
Completed NSE at 07:56, 0.00s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 07:56
Completed NSE at 07:56, 0.00s elapsed
Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 244.47 seconds
           Raw packets sent: 2021 (88.900KB) | Rcvd: 24 (956B)
                                                      
```



![image](https://github.com/user-attachments/assets/ed501f05-1c56-41fe-b5db-f6b132f01ea3)


## ``gobuster``

```
gobuster dir -u http://10.10.195.194/ -w /home/kali/Downloads/wordlists/directory-list-2.3-medium.txt
```

```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.195.194/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /home/kali/Downloads/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 236] [--> http://10.10.195.194/images/]
/blog                 (Status: 301) [Size: 234] [--> http://10.10.195.194/blog/]
/rss                  (Status: 301) [Size: 0] [--> http://10.10.195.194/feed/]
/sitemap              (Status: 200) [Size: 0]
/login                (Status: 302) [Size: 0] [--> http://10.10.195.194/wp-login.php]
/0                    (Status: 301) [Size: 0] [--> http://10.10.195.194/0/]
/feed                 (Status: 301) [Size: 0] [--> http://10.10.195.194/feed/]
/video                (Status: 301) [Size: 235] [--> http://10.10.195.194/video/]
/image                (Status: 301) [Size: 0] [--> http://10.10.195.194/image/]
/atom                 (Status: 301) [Size: 0] [--> http://10.10.195.194/feed/atom/]
/wp-content           (Status: 301) [Size: 240] [--> http://10.10.195.194/wp-content/]
/admin                (Status: 301) [Size: 235] [--> http://10.10.195.194/admin/]
/audio                (Status: 301) [Size: 235] [--> http://10.10.195.194/audio/]
/intro                (Status: 200) [Size: 516314]
/wp-login             (Status: 200) [Size: 2613]
/css                  (Status: 301) [Size: 233] [--> http://10.10.195.194/css/]
/rss2                 (Status: 301) [Size: 0] [--> http://10.10.195.194/feed/]
/license              (Status: 200) [Size: 309]
/wp-includes          (Status: 301) [Size: 241] [--> http://10.10.195.194/wp-includes/]
/js                   (Status: 301) [Size: 232] [--> http://10.10.195.194/js/]
/Image                (Status: 301) [Size: 0] [--> http://10.10.195.194/Image/]
/rdf                  (Status: 301) [Size: 0] [--> http://10.10.195.194/feed/rdf/]
/page1                (Status: 301) [Size: 0] [--> http://10.10.195.194/]
/readme               (Status: 200) [Size: 64]
/robots               (Status: 200) [Size: 41]
/dashboard            (Status: 302) [Size: 0] [--> http://10.10.195.194/wp-admin/]
/%20                  (Status: 301) [Size: 0] [--> http://10.10.195.194/]
Progress: 5014 / 220561 (2.27%)[ERROR] Get "http://10.10.195.194/kudos": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
Progress: 5629 / 220561 (2.55%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 5634 / 220561 (2.55%)
===============================================================
Finished
===============================================================
```

```
https://10.10.195.194/robots
```

```
User-agent: *
fsocity.dic
key-1-of-3.txt
```


---

```
https://10.10.195.194/key-1-of-3.txt
```

```
073403c8a58a1f80d943455fb30724b9
```

---

```
https://10.10.195.194/fsocity.dic
```

![image](https://github.com/user-attachments/assets/b3ab514a-49a8-4a0d-afa5-ab57c635d085)



i notice that if i enter wrong usernanme page say ``invalid suername`` but when doing brute force if username valid it say ``The password you entered for the username``


![image](https://github.com/user-attachments/assets/262b08ef-57fb-427e-a15a-4cdc7285a6c4)

![image](https://github.com/user-attachments/assets/55773854-ef9e-4c9a-a118-35a0c595964e)


so username is: 

```
Elliot
```

now do the same brute force but on passowrd


![image](https://github.com/user-attachments/assets/ebb70d54-f68f-4867-8b56-2712a643e06b)

so the password is:

```
ER28-0652
```





















