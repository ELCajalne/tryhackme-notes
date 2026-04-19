# Tcpdump

Networking protocols work invisibly behind the scenes — you never actually see an ARP query when accessing local resources, or a three-way handshake when browsing the internet. All the technical complexity is hidden behind smooth user interfaces.

**The solution:** Capturing network traffic and inspecting it directly — this is the best way to truly understand how networks work at a deeper level.

---

## What is Tcpdump?

A command-line packet capture tool that lets you see raw network traffic in your terminal.

- Written in C and C++
- Built on the `libpcap` library — the foundation for many other networking tools today
- Ported to Windows as `WinPcap`
- Extremely stable and fast due to its age and low-level design

> Tcpdump is the terminal version of packet capture — lightweight, fast, and available anywhere a terminal is.

---

## Basic Packet Capture

Running `tcpdump` alone does nothing useful — you need to tell it what to listen to, where to save, and how many packets to grab.

### Pick a Network Interface

```bash
tcpdump -i eth0       # listen on a specific interface
tcpdump -i any        # listen on all interfaces
ip a s                # list all available interfaces on your machine
```

Every interface has a name — `eth0`, `ens5`, `wlan0`, etc. Run `ip a s` to see what yours are called. Without specifying an interface, `tcpdump` doesn't know which door to listen at.

### Save Packets to a File

```bash
tcpdump -i ens5 -w capture.pcap
```

Saves everything to a `.pcap` file you can open later in Wireshark. You won't see packets scrolling on screen — it runs silently until you hit `Ctrl+C`.

### Read Packets from a File

```bash
tcpdump -r capture.pcap
```

`-r` stands for read — instead of capturing live traffic, it opens a saved `.pcap` file. Useful when:

- You captured traffic earlier and want to go back and analyse it
- Someone sent you a capture file containing a network attack to investigate
- You want to study protocol behaviour without capturing live traffic again

The real power is combining `-r` with filters:

```bash
tcpdump -r capture.pcap port 80            # only show port 80 traffic from the file
tcpdump -r capture.pcap host 192.168.1.1   # only show traffic involving this IP
```

Capture everything first, then slice through it later without recapturing.

### Limit How Many Packets to Capture (`-c`)

```bash
tcpdump -i ens5 -c 100
```

Stops automatically after 100 packets. Without `-c` it runs forever until you manually stop it with `Ctrl+C`.

### Filter by Port

```bash
tcpdump -i ens5 -w capture.pcap port 80        # any traffic on port 80
tcpdump -i ens5 -w capture.pcap tcp port 80    # only TCP port 80
tcpdump -i ens5 -w capture.pcap udp port 53    # only UDP port 53
```

Without a filter you're capturing everything — noise included. Adding a port filter narrows it down to only what you care about.

### Don't Resolve IP Addresses and Port Numbers

By default, `tcpdump` tries to be "helpful" by converting IPs to domain names and port numbers to protocol names — so `80` becomes `http`. This requires DNS lookups which slows things down and clutters your output.

```bash
tcpdump -i ens5 -n     # stops DNS lookups (keeps raw IPs)
tcpdump -i ens5 -nn    # stops both DNS lookups AND port name resolution
```

For analysis work, `-nn` is almost always what you want — you see raw IPs and port numbers as they actually are.

### Verbose Output (`-v`)

By default `tcpdump` shows minimal info per packet. Adding `-v` prints more detail like TTL, packet length, and IP options.

```bash
tcpdump -i ens5 -v     # a bit more detail
tcpdump -i ens5 -vv    # even more detail
tcpdump -i ens5 -vvv   # maximum detail
```

---

## Command Reference

| Command | What it does |
|---|---|
| `tcpdump -i INTERFACE` | Capture on a specific interface |
| `tcpdump -w FILE` | Save packets to a file |
| `tcpdump -r FILE` | Read packets from a file |
| `tcpdump -c COUNT` | Stop after a set number of packets |
| `tcpdump -n` | Don't resolve IP addresses |
| `tcpdump -nn` | Don't resolve IPs or port numbers |
| `tcpdump -v / -vv / -vvv` | Increase output verbosity |

### Real Examples

```bash
tcpdump -i eth0 -c 50 -v          # capture 50 packets on eth0, verbose
tcpdump -i wlan0 -w data.pcap     # capture on WiFi, save to file, runs until Ctrl+C
tcpdump -i any -nn                # capture everything, no DNS or port resolution
```

---

## Filtering Expressions

Although you can run `tcpdump` without filtering expressions, it won't be useful — it's impossible to see everything at once.

### Filtering by Host

```bash
tcpdump host example.com -w http.pcap      # all traffic to AND from example.com
tcpdump host 192.168.1.50 -w printer.pcap  # all traffic to AND from this IP
```

Filter by direction:

```bash
tcpdump src host 192.168.1.50   # only traffic FROM this host
tcpdump dst host 192.168.1.50   # only traffic TO this host
```

- You can use either an IP address or a hostname — both work.
- Capturing packets requires root privileges. Always use `sudo` or be logged in as root.

```bash
sudo tcpdump host example.com -w http.pcap
```

### Filtering by Port

```bash
sudo tcpdump -i ens5 port 53    # all DNS traffic (UDP & TCP)
sudo tcpdump -i ens5 port 80    # all HTTP traffic
sudo tcpdump -i ens5 port 443   # all HTTPS traffic
```

Filter by direction:

```bash
tcpdump src port 53   # only traffic FROM port 53
tcpdump dst port 53   # only traffic TO port 53
```

Combine host and port filters:

```bash
sudo tcpdump host 192.168.1.50 and port 80   # traffic to/from this IP on port 80 only
```

### Filtering by Protocol

```bash
sudo tcpdump -i ens5 icmp    # only ICMP traffic
sudo tcpdump -i ens5 tcp     # only TCP traffic
sudo tcpdump -i ens5 udp     # only UDP traffic
sudo tcpdump -i ens5 ip      # only IPv4 traffic
sudo tcpdump -i ens5 ip6     # only IPv6 traffic
```

> **Reading ICMP traffic:** Seeing `ICMP echo request` + `ICMP echo reply` back to back = someone ran `ping`. Seeing `ICMP time exceeded` messages = someone ran `traceroute` (it deliberately manipulates TTL to make routers reveal themselves). The packet types tell the whole story even without seeing the commands.

### Logical Operators

| Operator | Behaviour | Example |
|---|---|---|
| `and` | Both conditions must be true | `tcpdump host 1.1.1.1 and tcp` |
| `or` | Either condition can be true | `tcpdump udp or icmp` |
| `not` | Condition must be false | `tcpdump not tcp` |

### Basic Filter Reference

| Command | What it does |
|---|---|
| `tcpdump host 1.1.1.1` | Traffic to or from this IP |
| `tcpdump src host 1.1.1.1` | Traffic FROM this IP only |
| `tcpdump dst host 1.1.1.1` | Traffic TO this IP only |
| `tcpdump port 80` | Traffic on this port (either direction) |
| `tcpdump src port 80` | Traffic FROM this port |
| `tcpdump dst port 80` | Traffic TO this port |
| `tcpdump tcp` | Filter by protocol (`tcp`, `udp`, `icmp`, `ip`, `ip6`) |

### Real Examples

```bash
tcpdump -i any tcp port 22                              # SSH traffic on all interfaces
tcpdump -i wlo1 udp port 123                            # NTP traffic on WiFi
tcpdump -i eth0 host example.com and tcp port 443 -w https.pcap   # HTTPS traffic to example.com
```

### Counting Packets with `wc`

```bash
tcpdump -r traffic.pcap icmp -n | wc
```

- `-r traffic.pcap` — read from file instead of live capture
- `icmp` — filter to only ICMP packets
- `-n` — don't resolve IPs (keeps it fast and clean)
- `|` — pipe output into the next command
- `wc` — word count; each line = one packet, so the first number returned = number of ICMP packets

```bash
tcpdump -r traffic.pcap udp port 53 -n -c 1   # first DNS query in a capture file
```

> Use `-n` whenever you don't want `tcpdump` wasting time doing DNS lookups. In practice that's almost always — especially when analysing files, counting packets with `wc`, or when you already know the IPs you're looking for.

---

## Advanced Filtering

### Filter by Packet Size

```bash
tcpdump -r traffic.pcap greater 500   # packets 500 bytes or larger
tcpdump -r traffic.pcap less 100      # packets 100 bytes or smaller
```

Useful when looking for large file transfers or tiny control packets. For every possible filter expression, check `man pcap-filter`.

---

## Binary Operations

Before filtering by TCP flags, you need to understand binary operations — these work directly on bits (0s and 1s).

### `&` (AND) — both inputs must be 1

| Input 1 | Input 2 | Result |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | 1 |

### `|` (OR) — returns 1 unless both are 0

| Input 1 | Input 2 | Result |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 1 |

### `!` (NOT) — flips the bit

| Input | Result |
|---|---|
| 0 | 1 |
| 1 | 0 |

These operations are the foundation for filtering TCP flags.

---

## Header Bytes

```bash
proto[expr:size]
```

- `proto` — the protocol (`tcp`, `udp`, `ip`, `icmp`, `arp`, `ether`, `ip6`)
- `expr` — which byte to look at (0 = first byte)
- `size` — how many bytes to read (optional, defaults to 1)

This lets you filter packets based on the actual contents of a protocol's header bytes.

---

## TCP Flags

Use `tcp[tcpflags]` to filter by TCP flags specifically.

| Flag | Meaning |
|---|---|
| `tcp-syn` | SYN — initiating a connection |
| `tcp-ack` | ACK — acknowledging data |
| `tcp-fin` | FIN — closing a connection cleanly |
| `tcp-rst` | RST — forcefully resetting a connection |
| `tcp-push` | Push — send data immediately |

### Filtering by TCP Flags

```bash
# only SYN flag set, everything else unset
tcpdump "tcp[tcpflags] == tcp-syn"

# at least SYN flag is set (other flags may also be set)
tcpdump "tcp[tcpflags] & tcp-syn != 0"

# at least SYN or ACK is set
tcpdump "tcp[tcpflags] & (tcp-syn|tcp-ack) != 0"
```

**The difference between `==` and `& != 0`:**
- `==` — only that flag is set, nothing else
- `& != 0` — at least that flag is set, others may be too

### Why This Matters for Security

Filtering by TCP flags lets you find specific events instantly:

- Looking for new connections being initiated? Filter for `SYN`
- Looking for connections forcefully dropped? Filter for `RST`
- Looking for connections cleanly closed? Filter for `FIN`

This is the foundation for spotting port scans — an attacker sends a flood of SYN packets to find open ports.

---

## Displaying Packets

By default, `tcpdump` shows a basic one-line summary per packet. These flags change how much detail you see and in what format.

| Flag | What it does |
|---|---|
| `-q` | Brief output — just the essentials |
| `-e` | Show MAC addresses (link-level header) |
| `-A` | Show packet data as ASCII |
| `-xx` | Show packet data as hex |
| `-X` | Show packet data as hex + ASCII side by side |

### `-q` Quick Output

Strips out extra details — just timestamp, IPs, ports, and size. Good for a quick overview without noise.

```bash
tcpdump -r file.pcap -q
```

### `-e` Show MAC Addresses

Adds source and destination MAC addresses to output. Useful when investigating Layer 2 protocols like ARP and DHCP, or tracking unusual packets on a local network.

```bash
tcpdump -r file.pcap -e
tcpdump -r traffic.pcap arp -e -n
```

### `-A` Show Packet Data as ASCII

Displays raw packet contents as readable text. If the traffic is unencrypted, you might be able to read actual content — usernames, URLs, commands, etc.

```bash
tcpdump -r file.pcap -A
```

### `-xx` Hex Only

Shows every byte in the packet as hexadecimal. Lets you inspect raw IP and TCP headers byte by byte.

```bash
tcpdump -r file.pcap -xx
# 0x0000: 0283 1e40 5d17 44df 65d8 fe6c 0800 4500
# 0x0010: 004d fbd8 4000 3506 d229 6812 0c95 c0a8
```

> **Why hex?** ASCII only works for plain text. If traffic is encrypted, compressed, or binary — ASCII shows garbage. Hex solves this because any byte can always be represented as two hex digits regardless of content.

### `-X` Hex + ASCII Side by Side

The best of both worlds — hex on the left, ASCII on the right. If any part of the packet is readable text it'll show up on the right; everything else appears as dots.

```bash
tcpdump -r file.pcap -X
# 0x0000: 4500 004d fbd8 4000 3506 d229 6812 0c95  E..M..@.5..)h...
# 0x0010: c0a8 4259 01bb b0c0 a0b1 037c aa3b 357d  ..BY.......|.;5}
```