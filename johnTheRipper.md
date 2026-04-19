# John the Ripper

## Basic Syntax

```bash
john [options] [file path]
```

The file should contain the hash you want to crack. If it's in the current directory, just use the filename — no path needed.

---

## Three Ways to Crack

### Auto Mode

John guesses the hash type automatically. Unreliable, but useful when you don't know what you're dealing with.

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

### Identify the Hash First

Jumbo has a built-in identifier — no external tool needed.

```bash
john --list=formats | grep -iF "md5"
```

Or use `hash-id.py` for a ranked probability list:

```bash
wget https://gitlab.com/kalilinux/packages/hash-identifier/-/raw/kali/master/hash-id.py
python3 hash-id.py
```

### Format-Specific Cracking

```bash
john --format=[format] --wordlist=[wordlist] [file]
```

Example for MD5:

```bash
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

> Standard types (md5, sha1, etc.) need the `raw-` prefix. Confirm with `--list=formats`.

---

## Useful Commands

```bash
john --list=formats                      # list all supported formats
john --list=formats | grep -iF "md5"     # search for a specific format
john --show hash.txt                     # show previously cracked passwords
```

---

## The tool2john Pattern

Before cracking a protected file, extract its hash first using the matching conversion tool. All follow the same pattern:

```bash
tool2john [target file] > hash.txt
john --wordlist=rockyou.txt hash.txt
```

Tools live in `/usr/share/john/` or are available directly in PATH on Kali.

---

## Cracking Windows Authentication Hashes (NTLM)

**NTHash / NTLM** is the hash format used by modern Windows to store user and service passwords. Windows stores these in the `SAM` (Security Account Manager) database. On domain-joined machines they also exist in `NTDS.dit` — the Active Directory database.

### Getting the Hashes

You need to already be a privileged user. Common methods:

- `mimikatz`
- SAM dump
- `NTDS.dit` extract

### Cracking vs. Pass the Hash

You don't always need to crack the hash. Windows authentication can be abused by **passing the raw hash directly** — no plaintext needed. Hash cracking is still worth attempting when the password policy is weak, since the plaintext password may be reused elsewhere.

```bash
john --format=nt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

> NT = New Technology — a designation from the Windows NT era. The name was eventually dropped but lives on in NTLM and other Microsoft technology.

---

## Cracking Hashes from /etc/shadow

`/etc/shadow` stores Linux password hashes — one entry per user. Only accessible as root, so privilege escalation is required first.

### Step 1 — Grab Both Files

You need both `/etc/passwd` and `/etc/shadow`. John can't parse shadow hashes alone — it needs passwd for context (usernames, UIDs, shell info).

### Step 2 — Combine with `unshadow`

`unshadow` is a built-in John tool that merges the two files into a format John can read.

```bash
unshadow [path to passwd] [path to shadow]
unshadow local_passwd local_shadow > unshadowed.txt
```

You can use full files or just the relevant lines for a specific user — both work fine.

**Single user example:**

```bash
# local_passwd
root:x:0:0::/root:/bin/bash

# local_shadow
root:$6$2nwjN454g.dv4HN/$m9Z/r2x...:18576::::::
```

Then run `unshadow` on those two files as normal.

### Step 3 — Crack

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --format=sha512crypt unshadowed.txt
```

`sha512crypt` is the most common format on modern Linux systems — worth specifying explicitly.

---

## Single Crack Mode

Instead of a wordlist, John generates password candidates from the **username** using mangling rules — capitalisation changes, appended numbers, special characters, etc. It exploits the habit of basing weak passwords on your own name.

**GECOS data** — the 5th field in `/etc/passwd` (separated by `:`) stores user info like full name and office numbers. In single mode, John pulls this too and adds it to the generated candidates.

### Single Crack Mode Workflow

**Step 1 — Get format suggestions from the raw hash:**

```bash
john hash07.txt
```

**Step 2 — Prepend the username** (single mode needs this to know what to mangle):

```bash
sed -i 's/^/Joker:/' hash07.txt
```

**Step 3 — Run with `--single`:**

```bash
for f in NT Raw-MD4 Raw-MD5 mscash mscash2 Raw-MD5u HAVAL-128-4 MD2 mdc2 NT-long Raw-SHA1-AXCrypt ripemd-128 Snefru-128 ZipMonster; do
  echo "========== Trying $f =========="
  john --single --format=$f hash07.txt
done
```

**Step 4 — Retrieve the result:**

```bash
john --show --format=Raw-MD5 hash07.txt
```

> John stores cracked hashes in its `.pot` file. Use `--show` with the correct format to read it — the format must match what was used during cracking.

---

## Custom Rules

Custom rules let you define your own mangling patterns so John can generate targeted password candidates from a wordlist. Useful when you know something about the target's password structure.

### What They Exploit

Even with complexity requirements, users are predictable. Most people follow the same pattern — capital first, word in the middle, number and symbol at the end. e.g. `polopassword` → `Polopassword1!`

### Location of john.conf

| Environment | Path |
|---|---|
| TryHackMe AttackBox | `/opt/john/john.conf` |
| Package install / built from source | `/etc/john/john.conf` |

Jumbo John has existing rules starting around line 678 — useful reference if your syntax isn't working.

### Modifiers

| Modifier | What it does |
|---|---|
| `Az` | Appends characters to the end of the word |
| `A0` | Prepends characters to the start of the word |
| `c` | Capitalises the character at that position |

### Character Sets

| Set | Meaning |
|---|---|
| `[0-9]` | Numbers 0 to 9 |
| `[A-z]` | Upper and lowercase |
| `[a-z]` | Lowercase only |
| `[A-Z]` | Uppercase only |
| `[0]` | Only the number 0 |
| `[!£$%@]` | Specific symbols |

### Example — Matching `Polopassword1!`

Add to `john.conf`:

```
[List.Rules:PoloPassword]
cAz"[0-9][!£$%@]"
```

- `c` — capitalise the first letter
- `Az` — append to the end
- `[0-9]` — followed by a number
- `[!£$%@]` — followed by a symbol

Run it:

```bash
john --wordlist=[path to wordlist] --rule=PoloPassword [path to file]
```

> The rule name after `--rule=` must match exactly what you defined in `[List.Rules:YourRuleName]`.

### Using Built-in Rulesets

Rules mangle wordlist entries automatically — adding numbers, capitalisations, symbols, etc.

```bash
john --wordlist=rockyou.txt --rules=jumbo hash.txt
john --wordlist=rockyou.txt --rules=KoreLogic hash.txt
```

> `KoreLogic` is the most aggressive ruleset — use it when basic wordlists fail.

---

## Cracking Password-Protected ZIP Files

### Step 1 — Extract the Hash

```bash
zip2john [zip file] > [output file]
zip2john zipfile.zip > zip_hash.txt
```

### Step 2 — Crack

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt
```

### Step 3 — Show the Cracked Password

```bash
john --show zip_hash.txt
```

---

## Cracking Password-Protected RAR Archives

### Step 1 — Extract the Hash

```bash
/opt/john/rar2john rarfile.rar > rar_hash.txt
```

### Step 2 — Crack

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt rar_hash.txt
```

### Step 3 — Extract the Archive

```bash
unrar x file.rar
```

---

## Cracking SSH Key Passwords

SSH supports key-based auth as an alternative to passwords. The private key (`id_rsa`) is itself protected by a passphrase. `ssh2john` extracts that passphrase into a crackable hash format, which John then attacks using a wordlist.

### Step 1 — Convert id_rsa to Hash

```bash
ssh2john [id_rsa private key file] > [output file]
/opt/john/ssh2john.py id_rsa > id_rsa_hash.txt
```

| Platform | Command |
|---|---|
| AttackBox | `python3 /opt/john/ssh2john.py` |
| Kali | `python /usr/share/john/ssh2john.py` |

### Step 2 — Crack

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa_hash.txt
```

---

## The tool2john Pattern — Quick Reference

| Target | Conversion Tool | Notes |
|---|---|---|
| `/etc/shadow` | `unshadow` | Needs both `passwd` and `shadow` |
| ZIP file | `zip2john` | Direct output to John |
| RAR archive | `rar2john` | Same pattern as zip2john |
| SSH private key | `ssh2john` / `ssh2john.py` | Use python path if binary not found |
| PDF | `pdf2john` | Same pattern |
| Word doc | `office2john` | Same pattern |

> Once you know the pattern — convert first, then crack — every new file type clicks immediately.