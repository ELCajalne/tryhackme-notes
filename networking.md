# Networking 🌐

---

## Table of Contents
- [Careers in Cyber](#careers-in-cyber)
- [What is Networking?](#what-is-networking)
- [Identifying Devices](#identifying-devices-on-a-network)
- [Intro to LAN](#intro-to-lan)
- [Subnetting](#subnetting)
- [ARP](#arp)
- [DHCP](#dhcp)
- [OSI Model](#osi-model)
- [Packets & Frames](#packets--frames)
- [TCP/IP & Three-Way Handshake](#tcpip--the-three-way-handshake)
- [UDP/IP](#udpip)
- [Ports 101](#ports-101)
- [Firewalls 101](#firewalls-101)
- [VPN Basics](#vpn-basics)
- [LAN Networking Devices](#lan-networking-devices)
- [OSI Model](#-osi-model)
- [TCP/IP Model](#-tcpip-model)
- [IP Addresses & Subnets](#-ip-addresses--subnets)
- [UDP & TCP](#-udp--tcp)
- [Encapsulation](#-encapsulation)
- [Telnet](#-telnet)

---

## Careers in Cyber

| Role | Description |
|---|---|
| Security Analyst | Responsible for maintaining the security of an organization's data |
| Security Engineer | Designs, monitors, and maintains security controls, networks, and systems |
| Incident Responder | Identifies and mitigates attacks whilst an attacker's operations are still unfolding |
| Digital Forensics Examiner | Uses digital forensics to investigate incidents and crimes |
| Malware Analyst | Analyses all types of malware to learn how they work and what they do |
| Penetration Tester | Responsible for testing technology products for security loopholes |
| Red Teamer | Plays the role of an adversary, attacking an organisation and providing feedback |

---

## What is Networking?

Networks are simply things connected.

**The Internet** — One giant network that consists of many, many small networks.
- First iteration: ARPANET project in the late 1960s.
- 1989: Tim Berners-Lee invented the World Wide Web.
- Made up of **private networks** (small networks) connected by **public networks** (the internet).

---

## Identifying Devices on a Network

Every device has two means of identification:
- **IP Address** (Logical Identifier)
- **MAC Address** (Physical Identifier)

### IP Address
A set of numbers divided into four octets. Calculated through IP addressing and subnetting.
- Cannot be active simultaneously more than once within the same network.
- **Public Address** — identifies the device on the internet. Given by your ISP.
- **Private Address** — identifies a device within a local network.

**IPv4** — 2^32 IP addresses (4.29 billion).

**IPv6** — 2^128 addresses (340 trillion+). Resolves IPv4 exhaustion. More efficient.

### MAC Address
A sixteen-character hexadecimal number assigned at manufacture. Example: `a4:c3:f0:85:ac:2d`
- First six characters = company that made the network interface.
- Last six characters = unique number.
- Can be faked through **spoofing** — can break poorly implemented security designs.

### PING
Uses **ICMP (Internet Control Message Protocol)** packets to determine performance and reliability of a connection.
```
ping <IP address or website URL>
```

---

## Intro to LAN

**Topology** — The design or look of the network.

### Star Topology
Devices individually connected via a central switch or hub.
- **Pros:** Highly scalable, easy to add devices.
- **Cons:** More expensive. More maintenance as network scales.

### Bus Topology
Relies on a single backbone cable. Devices stem from it like leaves on a branch.
- **Pros:** Easy and cost-efficient to set up.
- **Cons:** Bottlenecks quickly. Single point of failure. Difficult to troubleshoot.

### Ring Topology (Token Topology)
Devices connected directly to each other forming a loop. A device only forwards data if it has nothing to send itself.
- **Pros:** Easy to troubleshoot. Less prone to bottlenecks.
- **Cons:** Not efficient — data may visit many devices first. A cable fault breaks the entire network.

### Switch
Aggregates multiple devices using ports of 4, 8, 16, 32, or 64. Uses **Packet Switching** to break data into packets. Switches and routers connected together increase redundancy.

### Router
Connects networks and passes data between them using the IP protocol. Operates at Layer 3. Determines the most optimal path for data.

---

## Subnetting

Splitting a network into smaller miniature networks using a **subnet mask**.

Subnet mask = four bytes (32 bits), ranging from 0 to 255.

**Subnets use IP addresses in three ways:**

| Type | Description | Example |
|---|---|---|
| Network Address | Identifies the start and existence of a network | `192.168.1.0` |
| Host Address | Identifies a specific device on the subnet | `192.168.1.1` |
| Default Gateway | Device capable of sending info to another network (router) | `.1` or `.254` |

Subnetting provides **efficiency**, **security**, and **full control**.

> Example: A café with two networks — one for employees, one for public hotspot — both still connected to the internet.

---

## ARP

**Address Resolution Protocol (ARP)** — Associates MAC addresses with IP addresses on a network.

Each device keeps an **ARP cache** — a ledger of other devices' identifiers.

### How ARP Works

**ARP Request** — Broadcast to the entire network:
> "Who has IP 192.168.1.20? Tell me your MAC address."
- Every device compares the requested IP with its own.
- Match → replies with MAC address. No match → ignores.

**ARP Reply** — Unicast response directly back to the requester.
- Contains the IP address and MAC address bound to it.
- Requesting device stores the mapping in its ARP cache.

---

## DHCP

**Dynamic Host Configuration Protocol** — Automatically assigns IP addresses.

### DHCP Process

| Step | Message | Description |
|---|---|---|
| 1 | DHCP Discover | Device broadcasts asking if any DHCP servers are present |
| 2 | DHCP Offer | Server replies with an available IP address |
| 3 | DHCP Request | Device confirms it wants the offered IP |
| 4 | DHCP ACK | Server acknowledges — device can now use the IP |

> Real-world example: Connecting to coffee shop Wi-Fi triggers a DHCP Discover automatically.

---

## OSI Model

**Open Systems Interconnection Model** — 7-layer framework dictating how all networked devices send, receive, and interpret data.

> **Encapsulation** — Data moves downward from Layer 7 to Layer 1, with each layer adding headers/trailers.
> **Decapsulation** — Reverse process on the receiving end.

| Layer | Name | Key Responsibility |
|---|---|---|
| 7 | Application | User interaction — browsers, email clients, DNS |
| 6 | Presentation | Translation, encoding, compression, encryption (TLS/SSL) |
| 5 | Session | Creates and maintains connections, checkpoints |
| 4 | Transport | TCP and UDP — reliability vs speed |
| 3 | Network | Routing via IP addresses — routers operate here |
| 2 | Data Link | MAC addressing, framing, error detection (NIC lives here) |
| 1 | Physical | Electrical signals, binary (1s and 0s) |

### Layer 2 — Data Link Detail
1. **Physical Addressing** — Adds source and destination MAC addresses.
2. **Framing** — Header (MAC) + Payload (packet) + Trailer (FCS error-check).
3. **Error Detection** — FCS/CRC detects corrupted bits. Corrupted frames are discarded.
4. **Medium Access Control** — Ethernet CSMA/CD, Wi-Fi CSMA/CA.

### Layer 4 — Transport Detail

**TCP (Transmission Control Protocol)**

| Advantages | Disadvantages |
|---|---|
| Guarantees accuracy of data | If one chunk is missing, entire data cannot be used |
| Synchronizes devices to prevent flooding | Slow connection can bottleneck the other device |
| Performs more processes for reliability | Significantly slower than UDP |

Used for: file sharing, internet browsing, email.

**UDP (User Datagram Protocol)**

| Advantages | Disadvantages |
|---|---|
| Much faster than TCP | Doesn't care if data is received |
| Leaves flow control to the application | Unstable connections = terrible experience |
| Does not reserve a continuous connection | — |

Used for: video streaming, voice chat, ARP, DHCP.

---

## Packets & Frames

| Term | Layer | Description |
|---|---|---|
| Packet | Layer 3 (Network) | Contains IP header and payload |
| Frame | Layer 2 (Data Link) | Encapsulates the packet and adds MAC addresses |

> Think of it like a letter: the envelope is the frame, the letter inside is the packet. This process is called **encapsulation**.

---

## TCP/IP & The Three-Way Handshake

TCP/IP has four layers: Application, Transport, Internet, Network Interface.

TCP is **connection-based** — must establish a connection before sending data.

### TCP Packet Headers

| Header | Description |
|---|---|
| Source Port | Randomly chosen port (0–65535) |
| Destination Port | Port of the service on the remote host (e.g., 80 for web) |
| Source IP | IP of the sending device |
| Destination IP | IP of the destination device |
| Sequence Number | Random number assigned to the first data piece |
| Acknowledgement Number | Sequence number + 1 |
| Checksum | Mathematical value to verify data integrity |
| Data | Actual bytes being transmitted |
| Flag | How the packet should be handled during the handshake |

### Three-Way Handshake

| Step | Message | Description |
|---|---|---|
| 1 | SYN | Client initiates connection |
| 2 | SYN/ACK | Server acknowledges |
| 3 | ACK | Client acknowledges server's sequence number |
| 4 | DATA | Data is sent |
| 5 | FIN | Connection cleanly closed |
| # | RST | Abrupt end due to error or fault |

### Sequence Number Agreement
1. **SYN** — Client: "Here's my ISN (0)."
2. **SYN/ACK** — Server: "Here's my ISN (5,000), I acknowledge yours (0)."
3. **ACK** — Client: "I acknowledge your ISN (5,000). Data at ISN+1 (1)."

### Closing a TCP Connection
Device sends a **FIN** packet → other device acknowledges with FIN/ACK. Best practice: close as soon as possible to free resources.

---

## UDP/IP

**Stateless protocol** — No connection required. No Three-Way Handshake.

### UDP Packet Headers

| Header | Description |
|---|---|
| TTL | Expiry timer so packets don't clog the network |
| Source Address | IP of the sending device |
| Destination Address | IP of the destination device |
| Source Port | Randomly chosen port |
| Destination Port | Port of the service on the remote host |
| Data | Bytes being transmitted |

---

## Ports 101

Numerical values between **0 and 65535** used to enforce communication rules.

- Ports **0–1024** = common ports.
- Non-standard ports must be specified with a colon: `http://example.com:8080`

### Common Ports

| Port | Protocol | Use |
|---|---|---|
| 21 | FTP | File Transfer |
| 22 | SSH | Secure Remote Access |
| 23 | Telnet | Unencrypted Remote Access |
| 25 | SMTP | Email Sending |
| 53 | DNS | Domain Name Resolution |
| 80 | HTTP | Web Traffic |
| 443 | HTTPS | Secure Web Traffic |
| 445 | SMB | Windows File Sharing |
| 3389 | RDP | Remote Desktop |

### Port Forwarding
Allows services on a private network to be accessible from the internet.
- Port forwarding **opens** ports.
- Firewalls **control** traffic through those ports.

---

## Firewalls 101

**Firewall** — Controls what traffic is allowed to enter and exit a network.

Evaluates: source, destination, port, and protocol.

### Types of Firewalls

| Type | Description |
|---|---|
| Stateful | Evaluates entire connection. Blocks entire device if connection is bad. More intelligent, more resource-heavy. |
| Stateless | Evaluates individual packets using static rules. Less intelligent but great for high-volume traffic (DDoS). |

---

## VPN Basics

**Virtual Private Network** — Creates a secure tunnel between devices on separate networks.

### Benefits

| Benefit | Description |
|---|---|
| Geographic connectivity | Connects offices in different locations |
| Privacy | Encrypts data — useful on public Wi-Fi |
| Anonymity | Hides traffic from ISPs. Only as anonymous as the VPN provider's logging policy. |

> TryHackMe uses a VPN to connect you to vulnerable machines safely.

### VPN Technologies

| Technology | Description |
|---|---|
| PPP | Handles authentication and encryption. Non-routable by itself. |
| PPTP | Tunnels PPP over internet. Easy to set up but weakly encrypted. |
| IPSec | Encrypts using existing IP framework. Difficult to set up but strong and widely supported. |

---

## LAN Networking Devices

### Router
- Connects networks and passes data using the IP protocol.
- Operates at **Layer 3**.
- Determines best route based on: shortest path, reliability, fastest medium (copper vs fibre).

### Switch
- Connects multiple devices using ethernet cables (3–63 devices).
- Operates at **Layer 2 or Layer 3** (mutually exclusive).
- **Layer 2 Switch** — Forwards frames using MAC addresses.

# from another module

## 🔗 OSI Model

The OSI (Open Systems Interconnection) model is a conceptual framework that describes how communications occur in a computer network. It has **7 layers**.

| Layer | Name | Main Function | Examples |
|---|---|---|---|
| 7 | Application | Services/interfaces to apps | HTTP, FTP, DNS, SMTP, IMAP |
| 6 | Presentation | Data encoding, encryption, compression | Unicode, MIME, JPEG, PNG |
| 5 | Session | Establishing & maintaining sessions | NFS, RPC |
| 4 | Transport | End-to-end communication & segmentation | TCP, UDP |
| 3 | Network | Logical addressing & routing | IP, ICMP, IPSec |
| 2 | Data Link | Reliable transfer between adjacent nodes | Ethernet (802.3), WiFi (802.11) |
| 1 | Physical | Physical transmission media | Electrical, optical, wireless signals |

### Layer Highlights

**Layer 1 — Physical**
Deals with the physical connection — wires, signals, and the definition of binary 0s and 1s.

**Layer 2 — Data Link**
Handles data transfer between nodes on the **same network segment** (e.g. computers on the same switch).
- Uses **MAC addresses** (Media Access Control) — 6 bytes in hexadecimal
- Format: `a4:c3:f0:85:ac:2d`
  - Left 3 bytes = vendor identifier
  - Right 3 bytes = unique device address

**Layer 3 — Network**
Handles **routing between different networks** — like connecting offices across cities.
- Uses IP addresses, ICMP, VPN protocols (IPSec, SSL/TLS)

**Layer 4 — Transport**
Enables **end-to-end communication** between applications on different hosts.
- Handles flow control, segmentation, and error correction
- Protocols: TCP, UDP

**Layer 5 — Session**
Establishes, maintains, and synchronizes communication sessions between applications.

**Layer 6 — Presentation**
Ensures data is in a format the application layer can understand.
- Handles encoding (ASCII, Unicode), compression, and encryption
- Example: MIME encodes binary files as 7-bit ASCII for email attachments

**Layer 7 — Application**
Provides network services directly to end-user applications.
- Examples: HTTP (web), FTP (files), DNS (domain names), POP3/SMTP/IMAP (email)

---

## 🌐 TCP/IP Model

The TCP/IP model is the practical model used by the modern internet. It groups the OSI layers into 4 (or 5) layers.

| TCP/IP Layer | OSI Equivalent | Protocols |
|---|---|---|
| Application | Layers 5, 6, 7 | HTTP, FTP, DNS, SMTP |
| Transport | Layer 4 | TCP, UDP |
| Internet | Layer 3 | IP, ICMP, IPSec |
| Link | Layer 2 | Ethernet, WiFi |
| *(Physical)* | Layer 1 | Signals, cables |

> The TCP/IP model was designed so the network can **continue functioning even if parts of it go down** — key for military and disaster resilience.

---

## 🌍 IP Addresses & Subnets

### IP Address Basics

An IP address uniquely identifies a device on a network. It consists of **4 octets** (32 bits total).

```
192  .  168  .  1  .  1
8bit    8bit   8bit  8bit  =  32 bits
```

- Each octet ranges from **0 to 255**
- `x.x.x.0` = network address (reserved)
- `x.x.x.255` = broadcast address (sends to all devices on the network)

```bash
ipconfig    # Windows
ifconfig    # Linux/Unix
ip a s      # Linux (modern)
```

### Subnet & Subnet Mask

- **Subnet** — a divided segment of a larger network (like a neighbourhood)
- **Subnet Mask** — defines which part of the IP is the network and which is the host

```
IP Address:   192.168.1.10
Subnet Mask:  255.255.255.0
```
The `255.255.255` = network (fixed for all devices), the `0` = host (unique per device)

With `255.255.255.0`: usable hosts = **1 to 254** (0 = network address, 255 = broadcast)

### CIDR Notation

Shorthand for subnet mask. The `/number` = how many bits are used for the network.

| CIDR | Subnet Mask | Usable Hosts |
|---|---|---|
| `/8` | `255.0.0.0` | ~16 million |
| `/12` | `255.240.0.0` | ~1 million |
| `/16` | `255.255.0.0` | ~65,000 |
| `/24` | `255.255.255.0` | 254 |

> Bigger `/number` = smaller network. Smaller `/number` = bigger network.

### Public vs Private IP

| | Public IP | Private IP |
|---|---|---|
| Visible to internet | Yes | No (by default) |
| Analogy | Home postal address | Apartment number in a compound |

**Private IP ranges (RFC 1918):**
```
10.0.0.0       -  10.255.255.255   (/8)
172.16.0.0     -  172.31.255.255   (/12)
192.168.0.0    -  192.168.255.255  (/16)
```

**A private IP can become exposed through:**
1. **Port Forwarding** — manually opening a door on the router for outside traffic
2. **VPN** — remote user becomes virtually inside the network
3. **Compromised device** — attacker already inside can reach all private IPs
4. **Misconfigured Firewall** — wrong rules accidentally let outside traffic in

### Routing
A **router** forwards data packets between networks at **Layer 3**, inspecting IP addresses and forwarding packets toward their destination — passing through multiple routers until it arrives.

---

## 📡 UDP & TCP

Both are **Layer 4 (Transport)** protocols that allow processes on different hosts to communicate using **port numbers** (range: 1–65535, port 0 is reserved).

| | UDP | TCP |
|---|---|---|
| Full Name | User Datagram Protocol | Transmission Control Protocol |
| Connection | Connectionless | Connection required |
| Delivery Guarantee | ❌ No | ✅ Yes |
| Speed | Faster | Slower |
| Analogy | Letter with no tracking | Parcel with delivery confirmation |
| Best for | Streaming, gaming, VoIP | Web browsing, email, file transfers |

### TCP Three-Way Handshake

Before data flows, TCP establishes a connection in 3 steps:

```
Client            Server
  |---- SYN ------->|    "Hey, I want to connect"
  |<--- SYN-ACK ----|    "Ok, I hear you. I'm ready"
  |---- ACK ------->|    "Great, let's go"
          ↓
     Data flows
```

---

## 📦 Encapsulation

Each network layer **wraps data with its own header** before passing it down — like putting a letter inside multiple envelopes.

| Layer | Data Name | What's Added |
|---|---|---|
| Application | Data | Raw user data |
| Transport | Segment (TCP) / Datagram (UDP) | Port numbers |
| Network | Packet | Source & destination IP addresses |
| Data Link | Frame | MAC addresses + trailer |

```
[ Ethernet Header | IP Header | TCP Header | Application Data | Ethernet Trailer ]
       ↑ Frame                                                         ↑
```

> Each layer only focuses on its own job and doesn't care what's inside — it just wraps and passes it down. This keeps each layer **independent and focused**.

---

## 🖥️ Telnet

TELNET (Teletype Network) is a protocol for remote terminal connection — it lets you connect to and communicate with a remote system and issue text commands over TCP.

```bash
# Connect to a host on a specific port
telnet <ip_address> <port>

# Echo server (port 7) — echoes everything you send
telnet 10.0.0.1 7

# Daytime server (port 13) — returns current date and time then closes
telnet 10.0.0.1 13

# Web server (port 80) — request a webpage
telnet 10.0.0.1 80
GET / HTTP/1.1
Host: example.com
# Press Enter twice
```

**Exiting Telnet:**
```
Ctrl + ]        →  enters telnet prompt
telnet> quit    →  exits completely
```

> ⚠️ Telnet sends data in **plain text** — not secure for production environments. Used mainly for testing and learning purposes.

---

