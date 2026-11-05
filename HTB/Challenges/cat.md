# cat 




## Part 1: What is an `.ab` file?

`.ab` stands for **Android Backup**. It's the file format produced by Android's built-in backup command:

```bash
adb backup -f mybackup.ab com.some.app
```

or, for a full-device backup:

```bash
adb backup -apk -shared -all -f full.ab
```

**What it contains:**
- App data (databases, shared preferences, internal files)
- Optionally APKs
- Optionally shared storage (`/sdcard/` contents — photos, downloads, etc.)

**Why it matters for pentesting:**
A backup can contain **everything an app stores on the device** — tokens, credentials, cached data, user files. If you can get a `.ab` file, you can read all of that offline. In real assessments, you might obtain one via:
- A physical device you have access to
- A malicious app with backup permissions
- A leaked backup file (as in this challenge)
- ADB access to a rooted/developer-mode device

---

## Part 2: The `.ab` file format (why it's weird)

An `.ab` file is **not** a plain tar. It has a **24-byte header** followed by a **gzipped tar archive**. The header looks like this:

```
ANDROID BACKUP\n
<version>\n
<compressed>\n
<encryption>\n
```

Then the rest of the file is a **gzip stream** containing a tar archive.

**The problem:** standard tools like `tar`, `gunzip`, or `7z` can't open it directly because of those 24 header bytes. They see garbage before the gzip magic bytes (`1f 8b 08 00`).

---

## Part 3: What the `printf | tail` trick actually does

Let's break down the command:

```bash
(printf "\x1f\x8b\x08\x00\x00\x00\x00\x00" ; tail -c +25 cat.ab ) | tar xfvz -
```

**Step by step:**

### `printf "\x1f\x8b\x08\x00\x00\x00\x00\x00"`

This writes **8 bytes** to stdout:
- `1f 8b` — gzip magic number
- `08` — compression method (deflate)
- `00` — flags
- `00 00 00 00` — modification time (zeroed out)

These are the first 8 bytes of a **valid gzip header**.

### `tail -c +25 cat.ab`

This outputs the `.ab` file **starting from byte 25** (i.e., skips the first 24 header bytes). The `+25` means "start at byte 25, 1-indexed" — so bytes 25 onward.

### `( ... ; ... )`

The parentheses group the two commands so their output is **concatenated** and piped together.

### `| tar xfvz -`

- `tar` — the archive tool
- `x` — extract
- `f` — read from a file (next argument)
- `v` — verbose (list files as they extract)
- `z` — gzip decompress
- `-` — the "file" is stdin (the pipe)

**So the whole command says:**
> "Prepend a minimal 8-byte gzip header, then append the `.ab` file starting at byte 25, and pipe the result into tar, which will gunzip and extract it."

**Why this works:**
- The original `.ab` header is 24 bytes.
- The gzip stream inside starts at byte 25.
- The gzip stream's own header (the first 8 bytes of it) is *already valid* — but tar's gzip decompressor needs the gzip magic to appear **at the start of its input**.
- By prepending a fresh 8-byte gzip header and then feeding the rest of the original gzip stream (from byte 25 onward), we hand tar a **valid gzip stream** that it can decompress.

The catch: the 8-byte header we prepend must match what the original gzip header *would have been* (same compression method, etc.). Since the `.ab` format uses standard gzip with deflate, this is always `1f 8b 08 00 00 00 00 00`.

**A more robust alternative** (if the header trick fails for some `.ab` files):

```bash
dd if=cat.ab bs=1 skip=24 of=cat.gz
gunzip -c cat.gz | tar xf -
```

This skips exactly 24 bytes, writes the rest to `cat.gz`, then gunzips and untars. Same result, more explicit.

---

## Part 4: What `abe.jar` does (Option B)

`abe.jar` (Android Backup Extractor) is a Java tool that does the same thing more cleanly:

```bash
java -jar abe.jar unpack cat.ab cat.tar
```

It:
1. Reads the 24-byte header
2. Detects the compression type
3. Strips the header
4. Decompresses the gzip stream
5. Writes a plain `.tar`

Then:

```bash
tar -xf cat.tar
```

extracts the tar normally.

**Why use `abe.jar`?**
- Handles **encrypted** backups (with a password)
- Handles **version differences** in the `.ab` format
- Doesn't rely on a hardcoded 8-byte header
- Cleaner for scripted workflows

For this challenge, both approaches work because the backup is **unencrypted** (that's part of the challenge — the flag is in an unprotected backup).

---

## Part 5: What we actually extracted

After decompression, you got two directories:

```
apps/
shared/
```

- **`apps/`** — per-app data, organized as `apps/<package_name>/...`
- **`shared/`** — shared storage (`/sdcard/`), organized as `shared/0/...` (0 = primary user)

Inside `shared/0/Pictures/`, you found the cat images. One image (`IMAG0004.jpg`) contained the flag printed on paper.

**Why the flag was there:**
The challenge author put a flag in a photo on "shared storage." In a real scenario, this would be a user taking a screenshot of a password, a QR code, a document, etc. — and that image being captured in a backup.

















