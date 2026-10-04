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


---

<img width="1918" height="720" alt="image" src="https://github.com/user-attachments/assets/c2ed429b-3df7-4a49-8cf8-bab36de17330" />

## i found in source code hint say 

> #### The default password is the device serial number included in your DeviceHub packaging

<img width="1918" height="444" alt="image" src="https://github.com/user-attachments/assets/f1205573-0086-49db-bd67-e7a4a3fe93fa" />

## found endpoint `/api` but i got `403` then i try `/api/status` 

<img width="1222" height="296" alt="image" src="https://github.com/user-attachments/assets/b44a5569-3ade-4d6f-a0cb-b4944048f212" />

````json
{
  "device": "Nexion DeviceHub DH-100",
  "serial": "NX-DH-2024-B7042",
  "firmware": "1.4.2",
  "status": "online",
  "uptime": 14564
}
````

## now let's login with serial number 

<img width="1919" height="841" alt="image" src="https://github.com/user-attachments/assets/04f5e7b6-93e2-4ffc-9e49-c9cf27055098" />



## Credentials Leaked in the Dashboard


|Item|Value|
|---|---|
|Scanner serial|`NX-SR-2024-0042`|
|Printer serial|`NX-TP-2024-0042`|
|**Username**|**`KioskUser`**|
|**Password**|**`K!0sk2026#`**|















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





























































