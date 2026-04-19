# Cryptography

## Core Concepts

Cryptography is how we protect data by scrambling it so only the intended recipient can read it.

### The Basic Flow

```
Plaintext + Key → [ Encryption Function ] → Ciphertext
Ciphertext + Key → [ Decryption Function ] → Plaintext
```

### Key Terms

| Term | Definition |
|---|---|
| **Plaintext** | The original readable data before encryption — text, images, credit card numbers, etc. |
| **Ciphertext** | The scrambled, unreadable version after encryption |
| **Cipher** | The algorithm that encrypts and decrypts — usually public knowledge |
| **Key** | The secret string of bits the cipher uses — this is what must stay secret |
| **Encryption** | Converting plaintext into ciphertext using a cipher and key |
| **Decryption** | Converting ciphertext back into plaintext using the same cipher and key |

> The cipher is public — everyone knows what algorithm is being used. The key is the secret. Security depends entirely on keeping the key safe, not hiding which cipher you're using.

---

## Historical Ciphers

Cryptography dates back to ancient Egypt around 1900 BCE. Understanding historical ciphers helps explain why modern encryption is so much more complex.

### Caesar Cipher

One of the simplest ciphers ever — shift every letter by a fixed number.

```
Plaintext:  T R Y H A C K M E
Key:        3 (shift right by 3)
Ciphertext: W U B K D F N P H
```

- Encryption → shift right by the key
- Decryption → shift left by the key
- When you reach Z, wrap back around to A

### Timeline

| Cipher | Era | Notes |
|---|---|---|
| Caesar Cipher | 1st century BCE | Simple letter shifting |
| Vigenère Cipher | 16th century | Uses a keyword instead of a single shift — harder to crack |
| Enigma Machine | World War II | Used by Nazi Germany, broken by Alan Turing |
| One-Time Pad | Cold War | Theoretically unbreakable if used correctly |

---

## Types of Encryption

### Symmetric Encryption

Uses the same key to both encrypt and decrypt. Because the key must stay secret, it's also called **private key cryptography**.

```
Plaintext + Key → [ Encrypt ] → Ciphertext
Ciphertext + Key → [ Decrypt ] → Plaintext
```

**The Key Distribution Problem:**
Both sender and receiver need the same key — but how do you share it securely in the first place? You can't just email it because anyone intercepting it gets the key too.

**Common Symmetric Algorithms:**

| Algorithm | Year | Key Size | Notes |
|---|---|---|---|
| DES | 1977 | 56-bit | Broken in under 24 hours in 1999 — no longer secure |
| 3DES | After DES | 168-bit (112-bit effective) | Just DES applied three times — a temporary fix. Deprecated 2019 |
| AES | 2001 | 128, 192, or 256-bit | Current standard — widely used everywhere today |

> Symmetric encryption is fast — great for encrypting large amounts of data. The weakness is securely sharing the key, not the encryption itself.

---

### Asymmetric Encryption

Two keys instead of one — a public key and a private key. They're mathematically linked but you can't derive one from the other.

```
Plaintext + Public Key  → [ Encrypt ] → Ciphertext
Ciphertext + Private Key → [ Decrypt ] → Plaintext
```

- **Public key** — share with everyone, it's not a secret
- **Private key** — never share this with anyone, ever

Anyone can encrypt a message using your public key. Only you can decrypt it with your private key. This solves the key distribution problem that symmetric encryption has.

**Common Asymmetric Algorithms:**

| Algorithm | Key Size | Notes |
|---|---|---|
| RSA | 2048, 3072, 4096-bit | Most widely used — 2048-bit minimum recommended |
| Diffie-Hellman | 2048, 3072, 4096-bit | Used for securely exchanging keys |
| ECC | 256-bit | Smaller keys, same security as 3072-bit RSA — more efficient |

> In practice, most systems use both — asymmetric encryption to securely exchange a key, then symmetric encryption for the actual data because it's much faster.

---

## Basic Math

### XOR Operation

XOR compares two bits — returns 1 if they're different, 0 if they're the same.

| A | B | Result |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

**Bit-by-bit example:**

```
  1010
⊕ 1100
------
  0110
```

**Key Properties:**

| Property | Meaning | Example |
|---|---|---|
| `A ⊕ A = 0` | Anything XOR itself = 0 | `1 ⊕ 1 = 0` |
| `A ⊕ 0 = A` | Anything XOR 0 = itself | `1 ⊕ 0 = 1` |
| Commutative | Order doesn't matter | `A ⊕ B = B ⊕ A` |
| Associative | Grouping doesn't matter | `(A ⊕ B) ⊕ C = A ⊕ (B ⊕ C)` |

**XOR as encryption:**

```
Encrypt:  Message ⊕ Key = Locked
Decrypt:  Locked  ⊕ Key = Message
```

Why decryption works:

```
Locked ⊕ Key
= (Message ⊕ Key) ⊕ Key    ← expand Locked
= Message ⊕ (Key ⊕ Key)    ← regroup (associative)
= Message ⊕ 0              ← key cancels itself out
= Message                   ← back to the original
```

**Why XOR alone isn't secure — key reuse:**

```
Message1 ⊕ Key = Locked1
Message2 ⊕ Key = Locked2

Locked1 ⊕ Locked2 = Message1 ⊕ Message2
```

The key completely disappears and the attacker has both messages mixed together — no key needed. With enough pattern analysis they can read both messages.

> XOR is too simple to use alone — but it's a building block inside virtually every modern encryption algorithm. The golden rule: **never use the same key twice.**

---

### Modulo Operation

Modulo (`%` or `mod`) gives you the remainder after division.

```
25 % 5 = 0    because 25 ÷ 5 = 5 remainder 0
23 % 6 = 5    because 23 ÷ 6 = 3 remainder 5
23 % 7 = 2    because 23 ÷ 7 = 3 remainder 2
```

Modulo is not reversible — easy to calculate one way, practically impossible to reverse. This is the foundation of how asymmetric encryption works.

---

## Public Key Cryptography

### Why We Use Both Asymmetric and Symmetric Together

Asymmetric encryption is slow. Symmetric encryption is fast but has the key sharing problem. In practice, systems use both:

- **Asymmetric** to safely share the symmetric key
- **Symmetric** for everything after

**The Locked Box Analogy:**

| Analogy | Real World |
|---|---|
| Secret code | Symmetric encryption key |
| Lock | Server's public key |
| Lock's key | Server's private key |

**How it works:**

1. Server sends you its public key (the open lock)
2. You put the symmetric key in a box and lock it with the server's public key
3. You send the locked box over the internet — anyone can see it but nobody can open it
4. Only the server can open it because only they have the private key
5. Now both of you have the same symmetric key — nobody else does
6. From this point on you communicate using fast symmetric encryption

> This is exactly how HTTPS works every time you visit a website. The padlock in your browser means this whole process happened successfully in the background.

---

## RSA

RSA is a public-key encryption algorithm that enables secure data transmission over insecure channels.

### The Math That Makes RSA Secure

RSA security is based on one simple fact: multiplying two large prime numbers is easy, but factoring the result back is practically impossible.

```
113 × 127 = 14351    ← easy
14351 = ??? × ???    ← hard
```

With numbers that are 300+ digits long, even the most powerful computers can't factor it in a reasonable time.

### How RSA Works — Step by Step

**Step 1 — Pick two prime numbers:**

```
p = 157
q = 199
n = p × q = 31243
```

**Step 2 — Generate the keys:**

```
ϕ(n) = n - p - q + 1 = 30888
Public key  = (n, e) = (31243, 163)
Private key = (n, d) = (31243, 379)
```

- `e` — any number that shares no factors with ϕ(n)
- `d` — calculated so that `e × d mod ϕ(n) = 1` (the mathematical "opposite" of e)
- `n` — p and q multiplied together (the hard part is factoring this back)

**Step 3 — Encrypt:**

```
y = x^e mod n
y = 13^163 mod 31243 = 16341
```

**Step 4 — Decrypt:**

```
x = y^d mod n
x = 16341^379 mod 31243 = 13
```

### Variable Reference

| Variable | What it is |
|---|---|
| `p` and `q` | Two large prime numbers — kept secret |
| `n` | p × q — part of both keys |
| `e` | Part of the public key |
| `d` | Part of the private key |
| `m` | Original plaintext message |
| `c` | Encrypted ciphertext |

- **Public key** = `(n, e)` — share with everyone
- **Private key** = `(n, d)` — never share this

### RSA in CTFs

CTF challenges usually give you some variables and ask you to decrypt a message. Two useful tools:

- `RSACtfTool` — most popular, works well for most challenges
- `rsatool` — another solid option

> In real life, p and q are 300+ digit numbers each. The example above uses tiny numbers just to show how the math works.

---

## Diffie-Hellman Key Exchange

Diffie-Hellman lets two parties create a shared secret together without ever transmitting the secret itself — even if someone is watching the entire conversation.

**Paint mixing analogy:**
- Alice and Bob both start with the same base colour (public values)
- Each adds their own secret colour privately
- They swap their mixed colours
- Each adds their secret colour to what they received
- Both end up with the same final colour — an observer can't figure out the secret colours from just seeing the swaps

### Step by Step — The Math

**Step 1 — Agree on public values (everyone can see these):**

```
p = 29   (a large prime number)
g = 3    (a generator)
```

**Step 2 — Each picks a private key (never shared):**

```
Alice's private key: a = 13
Bob's private key:   b = 15
```

**Step 3 — Each calculates their public key:**

```
Alice: A = g^a mod p = 3^13 mod 29 = 19
Bob:   B = g^b mod p = 3^15 mod 29 = 26
```

**Step 4 — Exchange public keys:**

```
Alice sends 19 to Bob
Bob sends 26 to Alice
```

An attacker can see both 19 and 26 — but that's useless without the private keys.

**Step 5 — Each calculates the shared secret:**

```
Alice: B^a mod p = 26^13 mod 29 = 10
Bob:   A^b mod p = 19^15 mod 29 = 10
```

Both independently arrive at `10` — the shared secret was never transmitted.

**Why an attacker can't figure out the secret:**

The attacker sees `p`, `g`, `A`, and `B`. To get the shared secret they'd need to solve the **discrete logarithm problem** — which is computationally impossible with real-world numbers hundreds of digits long.

### Diffie-Hellman vs RSA

| | Diffie-Hellman | RSA |
|---|---|---|
| Used for | Agreeing on a shared secret key | Digital signatures, authentication |
| Solves | Key exchange problem | Identity verification |
| Weakness alone | Vulnerable to MITM — can't verify who you're talking to | Slower for key exchange |

> Together: Diffie-Hellman creates the shared key. RSA verifies that the person you're talking to is actually who they say they are — preventing MITM attacks.

---

## SSH

### Authenticating the Server

When you SSH into a server for the first time, your client shows you the server's public key fingerprint and asks you to confirm.

```bash
The authenticity of host '10.10.244.173' can't be established.
ED25519 key fingerprint is SHA256:lLzhZc7YzRBDchm02qTX0qsLqeeiTCJg5ipOT0E/YM8.
Are you sure you want to continue connecting (yes/no)?
```

Your SSH client saves that fingerprint to `~/.ssh/known_hosts`. Every future connection checks against it:

- **Same fingerprint** → connects silently
- **Different fingerprint** → big warning — either the server changed its key or someone is intercepting your connection (MITM)

> Never blindly type `yes` without verifying the fingerprint first — especially on an untrusted network.

### Authenticating the Client

Two ways to authenticate:

- **Username & Password** — works but not best practice
- **Key Authentication** — more secure, uses a public/private key pair

**Generating SSH keys:**

```bash
ssh-keygen -t ed25519
```

This creates two files:

- `id_ed25519` — private key — **never share this**
- `id_ed25519.pub` — public key — goes on the server

The server stores your public key. When you connect, the server uses it to verify you own the matching private key — without you ever sending the private key.

### Supported Key Algorithms

| Algorithm | Notes |
|---|---|
| DSA | Public-key algorithm designed specifically for digital signatures |
| ECDSA | DSA variant using elliptic curve cryptography — smaller key sizes |
| ECDSA-SK | ECDSA with hardware security key support |
| Ed25519 | Uses EdDSA with Curve25519 — modern default, recommended |
| Ed25519-SK | Ed25519 with hardware security key support |

### Adding a Passphrase

When generating keys, you can add a passphrase to encrypt the private key file. Even if someone gets your private key file, they still can't use it without the passphrase.

> Your private key is the most sensitive file on your machine. Anyone who gets it can impersonate you to every server that trusts your public key.

---

## PGP & GPG

**PGP** (Pretty Good Privacy) is software that implements encryption for files, email, and digital signing.

**GPG** (GnuPG) is the open-source implementation of the OpenPGP standard — the real-world tool that puts asymmetric encryption, digital signatures, and key pairs all together.

### Generating a GPG Key

```bash
gpg --full-gen-key
```

It asks for:
1. Key type — RSA, ECC, etc. (ECC with Curve 25519 is the modern default)
2. Expiry date — how long the key is valid (`0` = never expires)
3. Your info — name, email, comment

### Common GPG Commands

```bash
gpg --full-gen-key                          # generate a new key pair
gpg --import backup.key                     # import a key from a file
gpg --decrypt file.gpg                      # decrypt an encrypted file
```

### GPG in CTFs

```bash
gpg --import backup.key                              # import the private key first
gpg --decrypt confidential_message.gpg               # then decrypt the file
```

If the private key is protected with a passphrase, try cracking it:

```bash
gpg2john backup.key > hash.txt    # convert to John format
john hash.txt                     # crack the passphrase
```

> Always keep a backup of your GPG private key in a secure location. Lose it and any messages encrypted to that key are gone forever.

---

## Attack Types

| Attack | How it works |
|---|---|
| **Brute Force** | Try every possible key or password combination until one works. Slow but guaranteed to eventually succeed |
| **Dictionary Attack** | Only try real words and common passwords from a wordlist. Much faster than brute force because most people use predictable passwords |
| **Cryptanalysis** | The science of breaking or bypassing cryptographic systems without knowing the key |

---

## Hashing

### What Is Hashing?

A hash function takes any input of any size and produces a fixed-size output called a **hash value**. Unlike encryption, there is no key and it's designed to be impossible to reverse.

**Example use case:**

```bash
Hash the original   →  a3f5c7d8e9b2...
Hash your download  →  a3f5c7d8e9b2...

Match    = identical ✅
No match = corrupted or tampered ❌
```

**Key properties:**

- Fast to compute
- Impossible to reverse — can't get the input back from the output
- One bit change = completely different hash

**Example — T vs U differ by one bit, but produce completely different hashes:**

```bash
md5sum file1.txt   →  b9ece18c950afbfa6b0fdbfa4ff731d3
md5sum file2.txt   →  4c614360da93c0a041b22e537de151eb

sha256sum file1.txt →  e632b7095b0bf32c260fa4c539e9fd7b852d0de454e9be26f24d0d6f91d069d3
sha256sum file2.txt →  a25513c7e0f6eaa80a3337ee18081b9e2ed09e00af8531c8f7bb2542764027e7
```

### Why Hashing Matters for Passwords

Servers never store your actual password — they store the **hash** of your password. When you log in, the server hashes what you typed and compares it to the stored hash. The real password is never saved anywhere.

### Hash Collisions

A collision is when two different inputs produce the same hash output. This is unavoidable — inputs are unlimited but outputs are fixed-size (the **pigeonhole effect**).

**MD5 and SHA1 are broken** — both have had collisions deliberately engineered:

- MD5 — demonstrated and proven broken
- SHA1 — the SHAttered attack proved it broken in 2017

> Don't use MD5 or SHA1 for anything security-related. Use SHA256 or SHA512 instead.

---

## Insecure Password Storage

### Common Insecure Practices

**Storing passwords in plaintext:**
- If database is leaked → instant compromise
- RockYou breach: 14+ million plaintext passwords leaked → led to the creation of `rockyou.txt`

**Using deprecated encryption instead of hashing:**
- Encryption is reversible — wrong approach for authentication
- Adobe breach: used outdated encryption + stored password hints in plaintext → passwords quickly recovered

**Using weak hashing algorithms without salting:**
- LinkedIn breach: used SHA-1 with no salting → hashes cracked quickly

### Why Salting Matters

A **salt** is a random value added to a password before hashing.

**Without salting:**
- Same password → same hash → one crack exposes all matching accounts
- Vulnerable to rainbow table attacks (precomputed hash → plaintext lookup tables)

**With salting:**
- Same password → different hash every time
- Rainbow tables become useless
- Cracking becomes significantly harder

### Secure Storage Process

1. Choose a strong hashing algorithm
2. Generate a unique salt per user
3. Combine password + salt
4. Hash the result
5. Store the hash and salt

Modern algorithms like `bcrypt`, `scrypt`, `Argon2`, and `PBKDF2` handle salting automatically.

**Why not use encryption?**
Encryption is reversible. If the key is compromised, all passwords can be decrypted. Hashing is one-way — the proper choice for authentication.

---

## Recognising Password Hashes

When you find a hash, identify the type before trying to crack it. Tools like `hashid` help but aren't always accurate — always combine tools with context.

- From a web app database → likely MD5
- From a Windows system → likely NTLM

### Linux Password Storage

Modern Linux stores hashes in `/etc/shadow` (root only). Older systems used `/etc/passwd` (world-readable — insecure).

**Hash format:**

```
$prefix$options$salt$hash
```

**Common prefixes (strongest → weakest):**

```
$y$         → yescrypt (modern default)
$gy$        → gost-yescrypt
$7$         → scrypt
$2b$, $2y$  → bcrypt
$6$         → SHA-512
$1$, $md5   → MD5 variants
```

**Example:**

```
strategos:$y$j9T$76UzfgEM5PnymhQ7TlJey1$/OOSg64dhfF.TigVPdzqiFang6uZA4QA1pzzegKdVm4
```

- `$y$` → yescrypt
- `j9T` → parameters
- `76Uzfg...` → salt
- `/OOSg...` → hash

### Windows Password Storage

Windows stores hashes in the **SAM** (Security Accounts Manager).

- Uses **NTLM** (based on MD4) — looks very similar to MD4/MD5, easy to misidentify
- Separates NT hashes (modern) and LM hashes (older, weaker)
- Tools like `mimikatz` can extract these hashes

---

## Password Cracking

### How Cracking Works

Password hashes can't be decrypted — you crack them by:

1. Taking candidate passwords (e.g. from `rockyou.txt`)
2. Adding the salt (if present)
3. Hashing each guess
4. Comparing to the target hash

Once a match is found, you've recovered the password.

**Common tools:** Hashcat, John the Ripper

### GPU vs CPU

GPUs are much faster for cracking — thousands of cores that process many hashes simultaneously. However, algorithms like `bcrypt` are specifically designed to limit GPU advantage, making them slow even on high-end hardware.

**Cracking on VMs:**
- No direct GPU access in most setups → much slower
- Use Hashcat on your **host machine** to maximise GPU usage
- John the Ripper works in VMs but is still faster on the host

### Hashcat Basics

```bash
hashcat -m <hash_type> -a <attack_mode> hashfile wordlist
```

| Flag | Meaning |
|---|---|
| `-m` | Hash type (e.g. `1000` = NTLM, `3200` = bcrypt) |
| `-a` | Attack mode (`0` = wordlist attack) |
| `hashfile` | File containing the hash |
| `wordlist` | List of password guesses |

**Example:**

```bash
hashcat -m 3200 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

---

## Hashing for Integrity Checking

Hash functions aren't just for passwords — they verify that data hasn't been changed.

**Use cases:**

- **Verifying downloads** — compare your file's hash with the official hash. Match = identical and untampered.
- **Detecting duplicates** — two files with the same hash are considered identical.

---

## HMAC (Keyed Hashing)

**HMAC** (Hash-based Message Authentication Code) adds a secret key to hashing to provide both:

- **Integrity** — data hasn't changed
- **Authenticity** — confirms who created the message

Unlike normal hashing, HMAC proves the sender is trusted because only they know the secret key.

**Formula:**

```
HMAC(K, M) = H((K ⊕ opad) || H((K ⊕ ipad) || M))
```

| Variable | Meaning |
|---|---|
| `K` | Secret key |
| `M` | Message |
| `H` | Hash function |

> Normal hashing ensures data integrity. HMAC adds a secret key to also guarantee authenticity.

---

## Quick Summary

| Topic | Key Point |
|---|---|
| RSA | Asymmetric encryption based on difficulty of factoring large numbers |
| Diffie-Hellman | Securely agree on a shared key over an insecure channel |
| SSH Key Pairs | Authenticate to servers using public/private keys instead of passwords |
| GPG | Real-world tool combining asymmetric encryption and digital signatures |
| Hashing | One-way — used for passwords and integrity checking, not encryption |
| Salting | Random value added to passwords before hashing — defeats rainbow tables |
| HMAC | Keyed hashing — provides both integrity and authenticity |