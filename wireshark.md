# Wireshark – Study Notes

> Personal notes from TryHackMe Wireshark rooms and hands-on packet analysis labs. Written to actually understand what's happening — not just memorise menus.

---

## Contents
[Tool Overview](#tool-overview) · [Packet Dissection](#packet-dissection) · [Packet Navigation](#packet-navigation) · [Packet Filtering](#packet-filtering)

---

## Tool Overview

Wireshark is an open-source, cross-platform network packet analyzer. It lets you capture live traffic and inspect saved packet captures (PCAPs).

**What it's used for:**
- Detecting and troubleshooting network problems (load failures, congestion)
- Detecting security anomalies (rogue hosts, suspicious traffic, abnormal port usage)
- Investigating and learning protocol behaviour in detail

**What it's NOT:** Wireshark is not an IDS. It doesn't alert you to threats automatically — it just shows you the packets. Whether something is suspicious depends entirely on the analyst reading it. It also never modifies packets, only reads them.

### The GUI

When Wireshark opens, five sections stand out:

| Section | What it does |
|---|---|
| **Toolbar** | Menus and shortcuts for sniffing, filtering, sorting, exporting and merging |
| **Display Filter Bar** | Where you type queries to filter traffic |
| **Recent Files** | Recently opened capture files — double-click to reopen |
| **Capture Filter & Interfaces** | Choose your network interface and set capture filters before sniffing |
| **Status Bar** | Shows current profile, sniffing status, and packet counts |

Once a PCAP is loaded, three panes appear:

| Pane | What it shows |
|---|---|
| **Packet List** | A row for every packet — source/destination, protocol, basic info |
| **Packet Details** | Full protocol breakdown of the selected packet, layer by layer |
| **Packet Bytes** | Raw hex and ASCII. Clicking a field in Packet Details highlights it here |

### Packet Colouring

Wireshark colour-codes packets so you can spot protocols and anomalies at a glance. Most traffic is green because most traffic is standard TCP/HTTP.

**Temporary rules** — last only the current session. Created via right-click → Conversation Filter.

**Permanent rules** — saved to your profile and persist across sessions. Created via View → Coloring Rules. Toggle with View → Colorize Packet List.

### Traffic Sniffing & File Management

- **Blue shark button** — start capturing · **Red** — stop · **Green** — restart
- **Merge PCAPs:** File → Merge. Save the result before working on it.
- **View file details:** Statistics → Capture File Properties (hash, capture time, interface, stats)

---

## Packet Dissection

Packet dissection is how Wireshark breaks down each packet into its individual layers and fields. Clicking any field in the details pane highlights the corresponding bytes in the bytes pane.

A fully dissected packet can have up to seven layers:

| Layer | What it shows |
|---|---|
| **Frame (Layer 1)** | Physical layer info — frame number, size, capture time |
| **Source MAC (Layer 2)** | Source and destination MAC addresses (Data Link layer) |
| **Source IP (Layer 3)** | Source and destination IP addresses (Network layer) |
| **Protocol (Layer 4)** | TCP or UDP details and source/destination ports (Transport layer) |
| **Protocol Errors** | TCP segments that needed reassembly |
| **Application Protocol (Layer 5)** | Protocol-specific details — HTTP, FTP, SMB etc. |
| **Application Data** | The actual application-level data being transferred |

---

## Packet Navigation

### Finding & Marking Packets

**Go to Packet** — Ctrl + G → type a packet number → jump straight there.

**Find Packets** — Edit → Find Packet. Two things to get right:

*Input type (what you're searching with):* Display Filter, Hex, String, or Regex. String and Regex are the most commonly used.

*Search field (where you're searching):* Packet List (high-level summary), Packet Details (protocol breakdown), or Packet Bytes (raw hex/ASCII).

> ⚠️ Searching in the wrong pane gives no results even if the data exists. Always match your search to where the data actually lives.

**Mark Packets** — Right-click → Mark/Unmark Packet. Marked packets turn **black**. Useful for flagging events of interest. Marks are lost when the file is closed.

**Packet Comments** — Edit → Packet Comment. Unlike marks, comments stay in the capture file until manually removed. Good for leaving notes for other analysts.

### Exporting

**Export Packets** — File → Export Packets. Pull out only the relevant packets into a new smaller PCAP. Options: all, selected, marked, a range, or currently displayed packets.

**Export Objects (Files)** — File → Export Objects → choose protocol. Wireshark can reconstruct actual files that were transferred over the network.

| Protocol | What you can extract |
|---|---|
| HTTP | Images, scripts, downloads |
| SMB | Windows file shares |
| TFTP | Simple transfers |
| IMF | Emails |
| DICOM | Medical images |

### Other Useful Features

**Time Display Format** — Default is "Seconds Since Beginning of Capture" which is useless for correlating with real-world logs. Change it: View → Time Display Format → UTC Date and Time of Day.

**Expert Info** — Analyze → Expert Information. Wireshark flags protocol anomalies automatically by severity: Chat (blue), Note (cyan), Warn (yellow), Error (red). These are suggestions — false positives happen.

---

## Packet Filtering

**Capture Filter** — set before sniffing. Only records matching packets. Everything else is never captured.

**Display Filter** — applied after capture. All packets are still there, non-matching ones are just hidden. Remove the filter to see everything again.

> **The golden rule:** *"If you can click on it, you can filter and copy it."* Right-click any value anywhere in Wireshark and apply it as a filter instantly — no need to memorise syntax for basic tasks.

### Filter Options

**Apply as Filter** — right-click any value → Apply as Filter. Filters for that single value immediately.

**Conversation Filter** — right-click → Conversation Filter. Filters the entire conversation between two endpoints (both IPs and ports). Everything unrelated disappears. Use this when you want to see the full back-and-forth between two machines.

**Colorize Conversation** — right-click → Colorize Conversation. Same as Conversation Filter but highlights instead of hiding. Undo with View → Colorize Conversation → Reset Colorization.

| | Conversation Filter | Colorize Conversation |
|---|---|---|
| Hides other packets | ✅ Yes | ❌ No |
| Highlights conversation | ❌ No | ✅ Yes |

**Prepare as Filter** — right-click → Prepare as Filter. Writes the query to the filter bar but doesn't run it yet. Chain conditions with "…and" / "…or" before hitting Enter.

**Apply as Column** — right-click any field in Packet Details → Apply as Column. That value becomes a visible column across all packets. Remove via right-click on the column header → Remove Column.

**Follow Stream** — right-click → Follow → TCP/UDP/HTTP Stream. Reconstructs the full conversation in one readable view. Client traffic is red, server is blue. If the traffic is unencrypted, you'll see everything — usernames, passwords, commands. Hit **X** on the filter bar to return to all packets after.

### Common Display Filter Queries

**By protocol name:**
```
http · arp · dhcp · ftp · smtp · pop · imap
```

**By port:**
```bash
tcp.port == 80     # HTTP
tcp.port == 443    # HTTPS
tcp.port == 21     # FTP
tcp.port == 25     # SMTP
udp.port == 53     # DNS
```

**By IP:**
```bash
ip.addr == 192.168.1.2      # either side
ip.src == 192.168.1.2       # traffic FROM this IP
ip.dst == 192.168.1.2       # traffic TO this IP
```

---

*Notes from hands-on TryHackMe Wireshark labs. Written while actually doing the rooms — not just reading about them.*
