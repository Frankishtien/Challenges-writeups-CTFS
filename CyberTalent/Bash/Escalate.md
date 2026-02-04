# Escalate


```
sudo -l
```


<img width="1601" height="319" alt="image" src="https://github.com/user-attachments/assets/9f80f6d4-5a05-4c94-86f8-cf4c540fd4be" />


### we will try to do Python Module Hijacking


## 1. first create new file call **`pyfiglet.py`** in **`/tmp`**

```
cd tmp/
nano pyfiglet.py
```
> ## put in it melicous code that will give us root 

```python
import os

os.system("/bin/bash")

```


## 2. Run the script as root with PYTHONPATH


```
sudo PYTHONPATH=/tmp /usr/bin/python3 /opt/cool.py
```


1️⃣ `sudo`

----------

`sudo`

✔️ Runs the command **with root privileges**

✔️ From `sudo -l` we know that you are allowed to run this command **without a password**

* * * * *


2️⃣ `PYTHONPATH=/tmp`

---------------------

`PYTHONPATH=/tmp`

This is the most important thing 🔥

### What does PYTHONPATH mean?

-   This is **Environment Variable**

- He says to Python:

    > “Before you go to any library, look first in this folder.”

By default, Python searches through the libraries in this order:

1\.  Current directory

2\.  PYTHONPATH

3\.  System libraries (`/usr/lib/python3/...`)

Here we say:

👉 **Role in the first `/tmp`**



* * * * *

3️⃣ Why `/tmp` in particular?

----------------------

Because you created this file in it:

`/tmp/pyfiglet.py`

The original script contains:

`import pyfiglet`

👈 Python says:

> Ah! I found `pyfiglet.py` in `/tmp`

> I will import it instead of the original library 😈

* * * * *


4️⃣ `/usr/bin/python3`

----------------------

`/usr/bin/python3`

This:

✔️ Python path

✔️ It is required to write like this because sudo specifies the command **with the full name**

* * * * *

5️⃣ `/opt/cool.py`

------------------

`/opt/cool.py`

This is the script that:

-   Owned by root

-   It runs as root

-   And it uses `import pyfiglet`

* * * * *


🔥 What actually happens (simply)

--------------------------

1\.  `sudo` runs the script as root

2\.  `PYTHONPATH=/tmp` makes Python look at the first `/tmp`

3\.  Finds **pyfiglet.py malware**

4\.  It executes the code inside it

5\.  Code:

`os.system("/bin/bash")`

6\.  💥Unlocks **root shell**





<img width="1912" height="422" alt="image" src="https://github.com/user-attachments/assets/d1621e31-fff8-4ace-9b94-4685debd6922" />








```
flag{D1d_y0u_kn0w_ab0ut_pyth0n_l1brary_h1j4ck1ng_??}
```



