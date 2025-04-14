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




```
Elliot
```


```
ER28-0652
```





















