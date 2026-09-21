# NahamStore Tryhackme Writeup


<img width="1907" height="355" alt="image" src="https://github.com/user-attachments/assets/cd2de51e-c29a-4144-bbee-f04b26c64748" />




<details>
  <summary>Setup</summary>



```
sudo nano /etc/hosts

## put 

10.114.163.104  nahamstore.thm

```

## now try to visit 

```
nahamstore.thm
```

<img width="1918" height="798" alt="image" src="https://github.com/user-attachments/assets/efa25a19-000a-4571-a6d5-2c69204ca52e" />


  
</details>


<details>
  <summary>Recon</summary>

## find open ports 

```
nmap -sCV -Pn 10.114.163.104
```

<img width="1196" height="512" alt="image" src="https://github.com/user-attachments/assets/2de9c6b9-0161-4ccf-8e90-300ed181bcc3" />



## find subdomanins 

```
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
-u http://nahamstore.thm \
-H "Host: FUZZ.nahamstore.thm" \
-ac

```

<img width="1135" height="510" alt="image" src="https://github.com/user-attachments/assets/3d159c7b-46c9-4a3e-842b-6241df908cc9" />

> lets add them to /etc/hosts


## found endpoints 

```
gobuster dir -u http://nahamstore.thm/ -w /usr/share/wordlists/dirb/common.txt
```

<img width="1268" height="581" alt="image" src="https://github.com/user-attachments/assets/00761bd2-263c-4fd4-a7ca-ce6f64dc9677" />



## in subdomains found in 

```
http://marketing.nahamstore.thm/
```

<img width="1897" height="336" alt="image" src="https://github.com/user-attachments/assets/5773a1f8-8f4a-475a-b12f-3c84e0c6d85a" />

### and in 

```
http://stock.nahamstore.thm/
```


<img width="1856" height="235" alt="image" src="https://github.com/user-attachments/assets/fc3506d5-4275-4b1e-ab18-a0d125a0ac5b" />

## on port 8000 found login page at `/admin` i try username and password `admin and it work`


<img width="1919" height="487" alt="image" src="https://github.com/user-attachments/assets/4e00575f-1a3b-4f34-bc00-90583441af82" />



  
</details>

























































