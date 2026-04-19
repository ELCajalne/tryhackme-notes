# Nmap: The Basics

Nmap answers two fundamental questions:
1. Which devices are alive on a network?
2. What services are running on those devices?

---

## Host Discovery: Who Is Online

### Ping Scan (`-sn`)

Nmap's `-sn` option finds which hosts are alive on a network without doing a full port scan. It's more powerful than a regular ping — it uses multiple techniques under the hood.

**Specifying targets:**

```bash
192.168.0.1-10    # range — scans .1 through .10
192.168.0.1/24    # subnet — scans all 256 addresses
example.thm       # hostname
```

### Local Network Scan

When scanning a network you're directly connected to (Ethernet or Wi-Fi), Nmap sends ARP requests to find hosts. If a device responds, it's marked as "Host is up". It can also reveal MAC addresses and network card vendors — useful for guessing what type of device it is.

```bash
sudo nmap -sn 192.168.66.0/24
```

### Remote Network Scan

When scanning a network separated by one or more routers, ARP requests can't reach the target — so Nmap switches to a combination of techniques:

- ICMP Echo requests (ping)
- ICMP Timestamp requests
- TCP SYN packets to port 443
- TCP ACK packets to port 80

If a host doesn't respond to any of these, Nmap marks it as down.

```bash
sudo nmap -sn 192.168.11.0/24
```

### Other Useful Discovery Options

```bash
nmap -sL 192.168.0.1/24    # list scan — shows all targets without scanning them
```

Advanced discovery — more control over how Nmap finds hosts:

```bash
-PS[portlist]    # TCP SYN discovery
-PA[portlist]    # TCP ACK discovery
-PU[portlist]    # UDP discovery
```

> **Note:** Always run Nmap as root or with `sudo` — running as a regular user limits you to basic scan types and restricts what Nmap can do.

- `-sn` is great for quietly mapping who's alive on a network, but it won't tell you what services are running — for that you need a full port scan.
- `-sL` is useful for double-checking your target range is correct before running the real scan.

---

## Port Scanning: Who Is Listening

Every device can have up to 65,535 TCP ports and 65,535 UDP ports. Port scanning figures out which ones have a service actively listening on them.

### Connect Scan (`-sT`)

The most basic TCP scan. Nmap tries to complete a full three-way handshake with every target port.

```bash
nmap -sT 192.168.1.1
```

**What happens on an open port:**

```
Nmap  →  SYN        →  Target
Nmap  ←  SYN-ACK    ←  Target   (port is open)
Nmap  →  ACK        →  Target   (connection established)
Nmap  →  RST-ACK    →  Target   (Nmap tears it down)
```

**What happens on a closed port:**

```
Nmap  →  SYN        →  Target
Nmap  ←  RST-ACK    ←  Target   (port is closed)
```

- `-sT` is the most reliable scan but also the most detectable — it completes full connections which get logged by the target system.
- It's essentially the same as trying to `telnet` to every port one by one, but automated across all ports simultaneously.

```bash
nmap -sT 192.168.1.1              # scan default ports on one host
nmap -sT 192.168.1.1/24           # scan entire subnet
nmap -sT 192.168.1.1 -p 80        # scan specific port
nmap -sT 192.168.1.1 -p 80,443    # scan multiple ports
nmap -sT 192.168.1.1 -p 1-1000    # scan a range of ports
sudo nmap -sT 192.168.1.1         # always use sudo for best results
```

### SYN Scan / Stealth Scan (`-sS`)

Instead of completing the full three-way handshake, the SYN scan only sends the first step and never finishes the connection.

```bash
sudo nmap -sS 192.168.1.1
```

**What happens on an open port:**

```
Nmap  →  SYN        →  Target
Nmap  ←  SYN-ACK    ←  Target   (port is open)
Nmap  →  RST        →  Target   (Nmap kills it before completing)
```

**What happens on a closed port:**

```
Nmap  →  SYN        →  Target
Nmap  ←  RST-ACK    ←  Target   (port is closed)
```

Because the connection is never fully established, the target system is less likely to log it — most logging happens when a full connection completes.

> `-sS` is the default scan when running Nmap as root.

### UDP Scan (`-sU`)

Many important services run on UDP — DNS, DHCP, NTP, SNMP, VoIP. Unlike TCP, UDP has no handshake so the traffic looks different.

```bash
sudo nmap -sU 192.168.1.1
```

**What happens:**
- Closed UDP port → target responds with ICMP Destination Unreachable
- Open UDP port → usually no response (silence = possibly open)

UDP scans are slower than TCP because Nmap has to wait for timeouts on ports that don't respond.

### Limiting Target Ports

By default Nmap scans the 1,000 most common ports. You can change this:

```bash
nmap -sS 192.168.1.1 -F             # fast mode — top 100 ports only
nmap -sS 192.168.1.1 -p 10-1024     # scan ports 10 to 1024
nmap -sS 192.168.1.1 -p-25          # scan ports 1 to 25
nmap -sS 192.168.1.1 -p-            # scan ALL 65,535 ports
nmap -sS 192.168.1.1 -p 1-1023      # well-known ports only
```

### Port Scan Reference

| Option | Explanation |
|---|---|
| `-sT` | TCP connect scan — full three-way handshake |
| `-sS` | TCP SYN scan — only first step of the handshake |
| `-sU` | UDP scan |
| `-F` | Fast mode — scans the 100 most common ports |
| `-p [range]` | Specify a range of ports — `-p-` scans all ports |

### Practical Example: Finding a Web Server

**Problem:** Find the listening web server on `10.48.170.185`.

```bash
# Step 1 — scan for open ports
sudo nmap -sS 10.48.170.185 -n
```

**Output:**
```
7/tcp    open  echo
22/tcp   open  ssh
8008/tcp open  http
```

**Step 2** — port `8008` is showing `http` — that's the web server.

```bash
# Step 3 — access it
http://10.48.170.185:8008
```

> **Key lesson:** Web servers don't have to run on ports 80 or 443. Always scan first and look for any port showing `http` or `https` as the service.

---

## Version Detection: Extract More Information

### OS Detection (`-O`)

Nmap analyses various indicators from the target's responses to make an educated guess about what operating system it's running.

```bash
sudo nmap -sS -O 192.168.1.1
```

Example output:

```
Running: Linux 4.X|5.X
OS details: Linux 4.15 - 5.8
```

What Nmap looks at to guess the OS:
- How the target responds to certain packets
- TCP/IP stack behaviour
- MAC address vendor (e.g. QEMU virtual NIC = likely a VM)
- Network distance (how many hops away)

OS detection works better when:
- There's at least one open and one closed port to analyse
- You're running as root/sudo (required for `-O`)
- The target is not behind a firewall filtering responses

> Always treat OS detection as an educated guess — not a fact.

### Service and Version Detection (`-sV`)

Tells Nmap to figure out exactly what software and version is running on each open port.

```bash
sudo nmap -sS -sV 192.168.1.1
```

Output adds a VERSION column:

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10
```

Instead of just knowing port 22 is open, you now know it's running OpenSSH 8.9p1 on Ubuntu — useful for finding known vulnerabilities in specific versions.

### Aggressive Scan (`-A`)

Runs everything at once: OS detection, version detection, traceroute, and extra scripts.

```bash
sudo nmap -A 192.168.1.1
```

### Skip Host Discovery (`-Pn`)

By default, if a host doesn't respond during host discovery (e.g. it blocks ICMP/ping), Nmap marks it as down and skips it entirely. `-Pn` tells Nmap to skip that check and treat every target as alive.

```bash
sudo nmap -sS -Pn 192.168.1.1
```

Useful when:
- The target is blocking ping requests
- A firewall is filtering ICMP
- You know the host is up but Nmap thinks it's down

### Full Options Reference

| Option | What it does |
|---|---|
| `-sT` | TCP connect scan — full handshake |
| `-sS` | TCP SYN scan — stealth |
| `-sU` | UDP scan |
| `-sV` | Detect service versions |
| `-O` | Detect operating system |
| `-A` | OS + version + traceroute + extras |
| `-F` | Fast mode — top 100 ports |
| `-p [range]` | Specify ports — `-p-` scans all 65,535 |
| `-Pn` | Skip host discovery — treat all hosts as online |
| `-n` | Don't resolve hostnames |

---

## Timing: How Fast Is Fast?

Running a scan too fast can trigger an IDS or other security tools. Nmap gives you full control over speed.

### Timing Templates (`-T`)

Six levels from slowest to fastest:

| Template | Name | Speed | Use case |
|---|---|---|---|
| `-T0` | Paranoid | ~9.8 hours | Avoid IDS at all costs |
| `-T1` | Sneaky | ~27 mins | Stay under the radar |
| `-T2` | Polite | ~40 secs | Low network impact |
| `-T3` | Normal | ~0.15 secs | Default |
| `-T4` | Aggressive | ~0.13 secs | Fast, reliable networks |
| `-T5` | Insane | Fastest possible | Speed over accuracy |

```bash
sudo nmap -sS 192.168.1.1 -T0          # slowest — least detectable
sudo nmap -sS 192.168.1.1 -T4          # fast — good for lab environments
sudo nmap -sS 192.168.1.1 -T paranoid  # same as -T0, by name
```

### Parallel Probes

Controls how many ports Nmap probes at the same time — like sending multiple people to knock on doors simultaneously instead of one by one.

```bash
--min-parallelism 10    # always have at least 10 ports being probed at once
--max-parallelism 50    # never probe more than 50 ports at once
```

If the network is struggling and dropping packets, Nmap automatically slows down to 1 probe at a time. On a fast, clean network it can run hundreds simultaneously.

### Packet Rate

Controls how many packets Nmap fires per second across the entire scan.

```bash
--max-rate 50      # send max 50 packets/sec — slow and quiet
--max-rate 1000    # send max 1000 packets/sec — fast and noisy
--min-rate 100     # guarantee at least 100 packets/sec — prevents scan from dragging
```

- **Too fast** → IDS/firewall notices the flood and blocks you or raises an alarm
- **Too slow** → scan takes hours unnecessarily
- **Just right** → stay under the radar while finishing in a reasonable time

> The rate applies to the whole scan — if you're scanning 10 hosts at once, those 50 packets/sec are shared across all 10 hosts.

### Host Timeout

Sets a deadline per host — if a host hasn't responded within your time limit, Nmap moves on.

```bash
--host-timeout 30s    # give up after 30 seconds
--host-timeout 5m     # give up after 5 minutes
```

Good for large scans where you don't want one slow host holding everything up.

### Putting It All Together

```bash
sudo nmap -sS 192.168.1.0/24 -T3 --min-rate 100 --max-rate 300 --host-timeout 30s
```

Scans the whole subnet at normal timing, sends between 100–300 packets per second, and gives up on any host that takes longer than 30 seconds.

---

## Output: Controlling What You See

### Verbosity (`-v`)

Without `-v` you stare at a blank screen until the scan finishes. With `-v` you see real-time updates as the scan runs.

```bash
sudo nmap -sS 192.168.1.0/24 -v
```

Shows in real time:
- ARP ping scan starting
- DNS resolution happening
- Which hosts are up or down
- Which ports are being scanned
- Open ports as they're discovered

Stack more `v`s for more detail:

```bash
-v      # a bit more info
-vv     # more detail
-vvvv   # maximum verbosity
-v2     # same as -vv, written differently
```

> You can press `v` on your keyboard mid-scan to increase verbosity without restarting.

### Debugging (`-d`)

Goes even deeper than verbosity — shows raw internal workings of Nmap.

```bash
-d      # basic debugging
-dd     # more debugging
-d9     # maximum — thousands of lines
```

| Situation | Use |
|---|---|
| Scan is taking long and you want updates | `-v` |
| Learning how Nmap works step by step | `-v` or `-vv` |
| Something is broken and you need to diagnose why | `-d` |
| Deep troubleshooting | `-d9` (be prepared for a wall of text) |

### Saving Scan Results

Running a scan and not saving it means you lose everything when the terminal closes.

```bash
-oN results.txt       # normal — human readable, like what you see on screen
-oX results.xml       # XML — good for importing into other tools
-oG results.gnmap     # grepable — easy to search with grep/awk
-oA gateway           # all three formats at once
```

Save all three at once:

```bash
sudo nmap -sS 192.168.1.1 -oA gateway
```

This creates three files automatically:
- `gateway.nmap` — normal output
- `gateway.xml` — XML output
- `gateway.gnmap` — grepable output

| Format | Use for |
|---|---|
| `-oN` | Reading results yourself later |
| `-oX` | Importing into tools like Metasploit or reporting software |
| `-oG` | Searching results with `grep` — e.g. find all hosts with port 80 open |
| `-oA` | When unsure — saves everything so you have all options |

> Always save your scan results with `-oA`. You never know when you'll need to go back and reference them, and re-scanning isn't always possible.