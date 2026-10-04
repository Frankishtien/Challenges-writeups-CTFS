# Touch

<img width="1585" height="280" alt="image" src="https://github.com/user-attachments/assets/85077655-bce9-476f-ac22-e43010f7c2d3" />

---


## nmap scan

```ruby
Nmap scan report for 10.129.209.73
Host is up (0.40s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
3389/tcp open  ms-wbt-server Microsoft Terminal Service
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8443/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-trane-info: Problem with XML parsing of /evox/about
| http-title: Nexion DeviceHub - Login
|_Requested resource was /login
|_http-cors: GET POST PUT OPTIONS
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```


|Info|Value|Source|
|---|---|---|
|Hostname|`KIOSK-042`|RPC dump|
|Device model|`Nexion DeviceHub DH-100`|Login page|
|Gate|`B7`|Login page|
|Booking code|`KS7X2M`|Layover description|
|Name|`Jenny Crawford`|Layover description|
|Login hint|password = device serial number|Tooltip|
|API|`/api` → 403 auth required|gobuster|


```
impacket-rpcdump 10.129.209.73 | grep -iE "hostname|name|kiosk|serial"
```





























































