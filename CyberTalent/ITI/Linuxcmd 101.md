## Linuxcmd 101



```
tar -xvzf linux_chal.tar.gz
```

```
strings ./- | egrep -i "flag|FLAG|key|secret|CTF|you|flag{" | head
strings ./- | head -n 80
```

<img width="989" height="541" alt="image" src="https://github.com/user-attachments/assets/7f97a982-d812-41fb-b3cb-981a3185e75a" />

```
passforasciiii
```


---


<img width="822" height="589" alt="image" src="https://github.com/user-attachments/assets/7a5bf869-b042-4b79-9547-2fe547ab2b75" />


---

### شوف أول بايتات كل ملف (تدور لو فيه تواقيع زي PK أو 7z أو ELF):

```
for f in f*; do echo "== $f =="; xxd -l 16 "$f"; done
```

<img width="891" height="526" alt="image" src="https://github.com/user-attachments/assets/7cbf9d61-ebde-47f8-acef-97c29791e07e" />


```
thisisasciiiiiprintapleeeee
```

<img width="885" height="753" alt="image" src="https://github.com/user-attachments/assets/4b3663ce-f0d3-43bb-bc9e-ed7595ebfc72" />

---

```
# تجربة كل ملف كـ password (كل ملف فيه كلمة سر واحدة)
for f in test*; do pw=$(cat "$f"); echo "trying: $pw"; unzip -P "$pw" next.zip && break; done

```

<img width="1148" height="497" alt="image" src="https://github.com/user-attachments/assets/328a5754-2d2e-4662-acc4-4d8e070aa522" />

```
thissssisssthepasswordfornexxtfileeee
```


----

<img width="898" height="361" alt="image" src="https://github.com/user-attachments/assets/fce9c165-176b-4bea-8f68-03fb093bd343" />

```
strings -a -n 4 decodeme1.zip | head -n 60
# أو افحص لو فيه base64 ظاهر
xxd -p decodeme1.zip | tr -d '\n' | fold -w 64 | grep -E '^[A-Za-z0-9+/=]{40,}$' -m1 || echo "no obvious base64"

```


<img width="1124" height="296" alt="image" src="https://github.com/user-attachments/assets/ea88f163-1c34-457a-bddb-46fc873082c5" />



---------------


```
7z x decodeme1.zip -p$(zip2john decodeme1.zip > hash; john -w=One hash | tail -n 4 | cut -d " " -f 1 | tail -n 1)

```

<img width="1248" height="577" alt="image" src="https://github.com/user-attachments/assets/5a55f741-6416-4e58-ade8-8f5291bd5cc5" />


---




















```
synt{f1zcyr_yvahk_101}
```


```
flag{s1mple_linux_101}
```









