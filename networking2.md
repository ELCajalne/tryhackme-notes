# Networking Protocols – Study Notes

> These are my personal notes from studying networking fundamentals. I wrote these while going through TryHackMe rooms and Wireshark labs. The goal was to actually *understand* what's happening under the hood — not just memorise definitions.

---

## Table of Contents
1. [DHCP – How devices get their network settings](#dhcp)
2. [ARP – Connecting IP addresses to MAC addresses](#arp)
3. [ICMP – How networks communicate errors and diagnostics](#icmp)
4. [Routing – How data finds its way across the internet](#routing)
5. [NAT – How we stretched IPv4 addresses to last longer](#nat)
6. [DNS – The internet's phonebook](#dns)
7. [HTTP & HTTPS – How the web works](#http)
8. [FTP – Transferring files over a network](#ftp)
9. [SMTP – Sending email](#smtp)
10. [POP3 – Receiving email](#pop3)
11. [IMAP – Syncing email across devices](#imap)
12. [TLS & SSL – Encrypting everything](#tls)
13. [SSH – Secure remote access](#ssh)
14. [SFTP & FTPS – Secure file transfer](#sftp)
15. [VPN – Encrypted tunnels across the internet](#vpn)
16. [Quick Port Reference](#ports)

---

## DHCP – How devices get their network settings

Before your device can communicate on any network, it needs three things: an IP address, a gateway (router), and a DNS server. You *could* set these manually every time — but that's exactly what DHCP exists to avoid.

**DHCP (Dynamic Host Configuration Protocol)** handles all of this automatically the moment you connect to a network.

It uses **UDP** with two ports:
- **Port 67** — the DHCP server listens here
- **Port 68** — your device uses this to communicate

The moment you join a network, your device has no IP yet — so it can't send a direct message to anyone. Instead it shouts a **broadcast** to `255.255.255.255` (everyone on the network), and the DHCP server picks it up and responds with an IP offer.

### The four-step process (DORA)

| Step | Name | What happens |
|---|---|---|
| 1 | **Discover** | Your device broadcasts: *"Is there a DHCP server here?"* |
| 2 | **Offer** | DHCP server replies: *"Yes — here's an IP you can use"* |
| 3 | **Request** | Your device replies: *"Great, I'll take that one"* |
| 4 | **Acknowledge** | Server confirms: *"Done — that IP is yours"* |

---

## ARP – Connecting IP addresses to MAC addresses

Here's something that tripped me up at first: even though we think in terms of IP addresses, data on a local network is actually delivered using **MAC addresses**. IP lives at Layer 3. MAC lives at Layer 2. They don't automatically know about each other — that's ARP's job.

**ARP (Address Resolution Protocol)** answers the question: *"I know the IP I want to reach — but what's their MAC address?"*

The process is simple:
1. Your device broadcasts to the whole network: *"Who has IP `192.168.1.1`?"*
2. The device with that IP responds: *"That's me — my MAC is `7C:DF:A1:D3:8C:5C`"*
3. Your device caches that and can now deliver frames directly

One thing worth noting: ARP packets aren't wrapped in IP or UDP — they're sent **directly inside an Ethernet frame**. That puts them below the IP layer entirely.

### How a packet is actually built

When you send data, it gets wrapped layer by layer from the inside out:

```
your data
└── encrypted by TLS
    └── wrapped by TCP (adds ports)
        └── wrapped by IPv4 (adds IP addresses)
            └── wrapped by Ethernet (adds MAC addresses)
                └── sent as a Frame across the wire
```

When Wireshark captures it, you're peeling those layers back from the outside in.

> **DHCP vs ARP in one line:** DHCP gets you an IP so you can exist on the network. ARP figures out *where* to physically deliver your data once you're there.

---

## ICMP – How networks communicate errors and diagnostics

**ICMP (Internet Control Message Protocol)** isn't used to transfer data — it's used to send messages *about* the network. Two tools you'll use constantly are built on top of it.

### ping

Sends an **ICMP Echo Request (Type 8)** to a target and waits for an **ICMP Echo Reply (Type 0)**. It tells you whether a host is reachable and how long the round trip takes.

```bash
ping 192.168.11.1 -c 4
```

```
64 bytes from 192.168.11.1: icmp_seq=1 ttl=63 time=11.2 ms
64 bytes from 192.168.11.1: icmp_seq=2 ttl=63 time=3.81 ms
64 bytes from 192.168.11.1: icmp_seq=3 ttl=63 time=3.99 ms
64 bytes from 192.168.11.1: icmp_seq=4 ttl=63 time=23.4 ms

4 packets transmitted, 4 received, 0% packet loss
rtt min/avg/max/mdev = 3.805/10.596/23.366/7.956 ms
```

Keep in mind — no reply doesn't always mean the host is down. Firewalls can silently block ICMP.

### traceroute

Maps the path your packets take across the internet, hop by hop. On Linux it's `traceroute`, on Windows it's `tracert`.

The trick it uses is **TTL (Time-to-Live)** — a counter that decrements by 1 at every router. When it hits 0, the router drops the packet and sends back an **ICMP Time Exceeded (Type 11)** message revealing its own IP. By deliberately starting with TTL=1 and incrementing, you can get every router along the path to reveal itself.

```bash
traceroute example.com
```

Reading the output:
- Each line is a **hop** (a router along the path)
- `* * *` means that router didn't respond — it's still there, just silent
- You can watch the packet travel from your local network → ISP → across the internet → destination

---

## Routing – How data finds its way across the internet

Routers don't just randomly forward packets — they follow **routing tables** built by routing protocols that constantly share network information with each other.

| Protocol | What it is |
|---|---|
| **OSPF** (Open Shortest Path First) | Routers share their full network topology and each builds a complete map to calculate the best paths |
| **EIGRP** (Enhanced Interior Gateway Routing Protocol) | Cisco proprietary; factors in bandwidth and delay to pick the most efficient route |
| **BGP** (Border Gateway Protocol) | The backbone of the internet — how ISPs and large networks exchange routing information with each other |
| **RIP** (Routing Information Protocol) | Simple, older protocol for small networks; picks routes based purely on hop count |

---

## NAT – How we stretched IPv4 addresses to last longer

IPv4 gives us about 4.3 billion addresses — which sounds like a lot until you realise almost every device on the planet needs one. NAT buys us time by letting an entire network share a single public IP.

Think of it like an office building:
- The building has **one public street address**
- Inside, each employee has their own **desk extension**
- Incoming calls get forwarded to the right desk
- Outgoing calls all show the **same company number** to the outside world

The router keeps a translation table to track who's who:

| Internal IP | Internal Port | External IP | External Port |
|---|---|---|---|
| 192.168.0.129 | 15401 | 212.3.4.5 | 19273 |

Your laptop thinks it's at `192.168.0.129:15401`. The website thinks you're at `212.3.4.5:19273`. The router quietly swaps the addresses in both directions without either side knowing.

---

## DNS – The internet's phonebook

Nobody memorises IP addresses. Instead we use domain names — and **DNS (Domain Name System)** is what translates those names into IPs behind the scenes. Every time you type a URL, a DNS lookup happens before your browser can even start loading the page.

### Record types

| Record | What it does |
|---|---|
| **A** | Maps a domain to an IPv4 address |
| **AAAA** | Maps a domain to an IPv6 address |
| **CNAME** | Maps a domain to another domain (an alias) |
| **MX** | Points to the mail server for a domain |

DNS uses **UDP port 53** by default (fast, low overhead), and falls back to **TCP port 53** when the response is too large to fit in a single UDP packet.

```bash
# Look up a domain's IP
nslookup www.example.com

# Look up who owns a domain
whois example.com
```

---

## HTTP & HTTPS – How the web works

**HTTP (Hypertext Transfer Protocol)** is the protocol your browser uses to talk to web servers. Every webpage you load is an HTTP conversation happening over TCP.

### HTTP methods

| Method | What it does |
|---|---|
| `GET` | Fetch a resource (a page, image, file) |
| `POST` | Send data to the server (login form, file upload) |
| `PUT` | Create or overwrite a resource on the server |
| `DELETE` | Remove a resource |

**Ports:** HTTP → **TCP 80**, HTTPS → **TCP 443**

### What actually happens when you open a website

```
1. DNS lookup     →  find the IP address for the domain
2. TCP handshake  →  establish a connection with the server
3. HTTP request   →  ask for the page (GET / HTTP/1.1)
4. Response       →  server sends back the HTML
5. TCP close      →  cleanly end the connection
```

For HTTPS, a **TLS handshake** happens between steps 2 and 3. After that, all traffic is encrypted — capture those packets without the private key and you'll see nothing useful.

### You can do this manually with Telnet

```bash
telnet <ip_address> 80
```

Then type (press Enter **twice** after the Host line):
```
GET /index.html HTTP/1.1
Host: anything
```

---

## FTP – Transferring files over a network

**FTP (File Transfer Protocol)** is built specifically for file transfers and can be faster than HTTP under equal conditions. It uses **TCP port 21** for control commands.

### Core commands

| Command | What it does |
|---|---|
| `USER` | Send your username |
| `PASS` | Send your password |
| `RETR` | Download a file from the server |
| `STOR` | Upload a file to the server |

### Transfer modes — this actually matters

Before downloading, tell FTP what kind of file you're getting:

```bash
type ascii    # text files (.txt, .html, .csv)
type binary   # everything else (.jpg, .zip, .exe)
```

If you download a binary file in ASCII mode, FTP will mess with the line endings and **corrupt the file**. Always use binary for anything that isn't plain text.

---

## SMTP – Sending email

**SMTP (Simple Mail Transfer Protocol)** is how email clients send messages to mail servers, and how mail servers relay messages to each other. Default port is **TCP 25**.

### Commands

| Command | What it does |
|---|---|
| `HELO` / `EHLO` | Start an SMTP session |
| `MAIL FROM` | Set the sender address |
| `RCPT TO` | Set the recipient address |
| `DATA` | Start writing the email body |
| `.` | A single dot on its own line ends the message |

You can send an actual email through Telnet to see SMTP raw:

```bash
telnet <mail_server_ip> 25
HELO client.thm
MAIL FROM: <you@domain.com>
RCPT TO: <them@domain.com>
DATA
Subject: Testing SMTP

Hello from Telnet!
.
QUIT
```

---

## POP3 – Receiving email

**POP3 (Post Office Protocol v3)** is how email clients retrieve messages from a mail server. The usual flow is: email arrives via SMTP → you download it via POP3. Default port is **TCP 110**.

### Commands

| Command | What it does |
|---|---|
| `USER <n>` | Identify yourself |
| `PASS <password>` | Authenticate |
| `STAT` | Get message count and total size |
| `LIST` | List all messages and their sizes |
| `RETR <n>` | Download message number n |
| `DELE <n>` | Mark message n for deletion |
| `QUIT` | End the session (deletions apply here) |

> ⚠️ POP3 sends **everything in plain text** — including your password. Anyone on the same network with Wireshark open can read every word. This is a textbook man-in-the-middle scenario. Use POP3S (port 995) if you have to use POP3 at all.

---

## IMAP – Syncing email across devices

**IMAP (Internet Message Access Protocol)** is the modern alternative to POP3. The key difference: POP3 downloads and deletes emails from the server. IMAP keeps everything on the server and syncs your state across all your devices. Default port is **TCP 143**.

### Commands

| Command | What it does |
|---|---|
| `LOGIN <user> <pass>` | Authenticate |
| `SELECT <mailbox>` | Open a mailbox folder |
| `FETCH <n> body[]` | Download message n with headers and body |
| `MOVE <set> <mailbox>` | Move messages to another folder |
| `COPY <set> <mailbox>` | Copy messages to another folder |
| `LOGOUT` | End the session |

---

## TLS & SSL – Encrypting everything

Protocols like HTTP, FTP, POP3, and SMTP send data in **plain text**. Anyone capturing traffic on the same network can read your passwords, emails, and anything else going through the wire. SSL and TLS were created specifically to fix this.

**Quick history:**
- **1995** — Netscape released SSL to solve the plain text problem
- **1999** — SSL was redesigned and renamed to TLS
- **2018** — TLS 1.3 released (current standard)

TLS wraps existing protocols in encryption. HTTP becomes HTTPS. SMTP becomes SMTPS. Nothing in the lower layers (TCP, IP) needs to change — TLS slots in between.

### What TLS actually guarantees

| Property | What it means in practice |
|---|---|
| **Confidentiality** | Nobody intercepting your traffic can read it |
| **Integrity** | Nobody can modify your data in transit without detection |
| **Authenticity** | You're actually talking to the real server, not an impostor |

### How your browser decides to trust a website

Websites prove their identity using **digital certificates** issued by trusted **Certificate Authorities (CAs)**:

1. The website creates a **Certificate Signing Request (CSR)**
2. Sends it to a CA (like DigiCert or Let's Encrypt)
3. The CA verifies it and issues a signed certificate
4. Your browser has a built-in list of trusted CAs — so it can verify the certificate is legitimate

Think of the CA as a government that issues ID cards. Your browser is the bouncer checking whether the ID is real.

### Secure vs insecure ports

| Protocol | Insecure Port | Secure Version | Secure Port |
|---|---|---|---|
| HTTP | 80 | HTTPS | 443 |
| SMTP | 25 | SMTPS | 465 / 587 |
| POP3 | 110 | POP3S | 995 |
| IMAP | 143 | IMAPS | 993 |
| Telnet | 23 | SSH | 22 |

---

## SSH – Secure remote access

**SSH (Secure Shell)** was created in 1995 to replace Telnet, which sent everything — including passwords — in plain text. SSH encrypts the entire session. Default port is **TCP 22**.

Beyond just terminal access, SSH also supports:
- **Public key authentication** — more secure than passwords
- **Port forwarding / tunneling** — routing other protocols through the encrypted SSH connection
- **X11 forwarding** — running graphical apps on a remote machine and displaying them locally

```bash
# Basic connection
ssh username@hostname

# Connect with same username as your local user
ssh hostname

# Enable graphical app forwarding
ssh username@hostname -X
```

---

## SFTP & FTPS – Secure file transfer

Two ways to do FTP securely — they're not the same thing.

**SFTP (SSH File Transfer Protocol)**
- Part of the SSH protocol suite — runs over the same encrypted SSH connection
- Uses **port 22** (same as SSH)
- Commands are Unix-style and slightly different from regular FTP

```bash
sftp username@hostname
get filename    # download a file
put filename    # upload a file
```

**FTPS (FTP Secure)**
- Regular FTP wrapped in **TLS**
- Uses **port 990**
- Requires a certificate set up on the server
- Can cause headaches with strict firewalls because it uses separate connections for commands and data transfer

---

## VPN – Encrypted tunnels across the internet

The internet wasn't designed with privacy in mind — it was designed to deliver packets. Anyone along the path (your ISP, network operators, anyone on the same network) can potentially see your traffic.

A **VPN (Virtual Private Network)** creates an **encrypted tunnel** between your device and a VPN server. From the outside, all anyone sees is encrypted traffic going to one server — not where it's ultimately headed or what's inside.

What this means practically:
- Websites see the **VPN server's IP**, not yours
- Your ISP sees **encrypted traffic**, not your actual browsing
- You can appear to be connecting from a different location

**Worth knowing before using one:**
- The VPN provider can still see your traffic — you're trusting them instead of your ISP
- Some setups can leak your real IP through DNS — run a **DNS leak test** if privacy actually matters
- VPNs are **restricted or illegal in some countries** — check local laws first

---

## Quick Port Reference

| Protocol | Transport | Port |
|---|---|---|
| FTP | TCP | 21 |
| SSH / SFTP | TCP | 22 |
| Telnet | TCP | 23 |
| SMTP | TCP | 25 |
| DNS | UDP / TCP | 53 |
| DHCP server | UDP | 67 |
| DHCP client | UDP | 68 |
| HTTP | TCP | 80 |
| POP3 | TCP | 110 |
| IMAP | TCP | 143 |
| HTTPS | TCP | 443 |
| SMTPS | TCP | 465 / 587 |
| FTPS | TCP | 990 |
| IMAPS | TCP | 993 |
| POP3S | TCP | 995 |

---

*Written while completing TryHackMe networking rooms and hands-on Wireshark labs. Everything here comes from actually doing it — not just reading about it.*
