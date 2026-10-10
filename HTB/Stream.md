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

```json
{
  "openapi": "3.1.0",
  "info": {
    "title": "AIDX flight-leg ingest",
    "version": "0.1.0"
  },
  "paths": {
    "/": {
      "get": {
        "summary": "Root",
        "operationId": "root__get",
        "responses": {
          "200": {
            "description": "Successful Response",
            "content": {
              "application/json": {
                "schema": {}
              }
            }
          }
        }
      }
    },
    "/health": {
      "get": {
        "summary": "Health",
        "operationId": "health_health_get",
        "responses": {
          "200": {
            "description": "Successful Response",
            "content": {
              "application/json": {
                "schema": {}
              }
            }
          }
        }
      }
    },
    "/feeds/aidx": {
      "post": {
        "summary": "Ingest",
        "operationId": "ingest_feeds_aidx_post",
        "responses": {
          "200": {
            "description": "Successful Response",
            "content": {
              "application/json": {
                "schema": {}
              }
            }
          }
        }
      }
    }
  }
}
```



## try send simple xml to `/feeds/aidx`

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



The ClickHouse HTTP interface is **notoriously powerful** and often misconfigured. Key points:

- **Port 8123** (default) or routed via nginx as `db.htb-airport.htb:80`
    
- **The Play UI** at `/play` lets you run SQL queries
    
- **The default user is `default`** with **no password** unless configured
    
- ClickHouse has many powerful functions that can lead to RCE:
    
    - `file()` — read arbitrary files
        
    - `url()` — SSRF
        
    - `remote()` — SSRF
        
    - `s3()` — S3 access
        
    - `executable()` — **execute commands** (if enabled)



## Test Basic ClickHouse Access

`Query 1 — Version and current user:`

```
curl -s 'http://db.htb-airport.htb/?query=SELECT%20version(),%20currentUser()'
```


<img width="1178" height="278" alt="image" src="https://github.com/user-attachments/assets/3543ac5e-60d7-4484-8da6-47e97b755a24" />

ClickHouse requires authentication. The error message is actually **very helpful**  it tells us exactly where the password would be on the server:

> The password for default user is typically located at `/etc/clickhouse-server/users.d/default-password.xml`

That confirms a **`default-password.xml`** file exists on the target — and if we can read it (via any file-read primitive), we get the password.





