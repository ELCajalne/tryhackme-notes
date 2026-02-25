# Windows Command Line ⌨️

---

## Table of Contents
- [Basic System Info](#basic-system-info)
- [Network Troubleshooting](#network-troubleshooting)
- [File & Disk Management](#file--disk-management)
- [Task & Process Management](#task--process-management)
- [Research & Documentation](#research--documentation)

---

## Basic System Info

| Command | Description |
|---|---|
| `set` | Check PATH and environment variables |
| `ver` | Determine OS version |
| `systeminfo` | Detailed system information |
| `systeminfo \| more` | Pipe output for long results |
| `help` | Show available commands |
| `cls` | Clear the CMD screen |

---

## Network Troubleshooting

| Command | Description |
|---|---|
| `ipconfig` | IP address, subnet mask, gateway |
| `ipconfig /all` | Full details — DNS, DHCP, all adapters |
| `ping example.com` | Check connectivity via ICMP packets |
| `tracert example.com` | Show route (hops) packets take to destination |
| `nslookup example.com` | DNS lookup using default DNS server |
| `nslookup example.com 1.1.1.1` | DNS lookup using a specific DNS server |
| `netstat` | Active network connections |
| `netstat -abon` | All connections with executable, PID, numeric format |

### ping output explained
- **Reply** = host is reachable
- **Time** = latency
- **TTL** = hops remaining
- **Packet loss** = connection stability

### tracert output
- Each line = a router your packet travels through
- "Request timed out" = router doesn't respond but packets still pass through

### netstat -abon flags
| Flag | Meaning |
|---|---|
| `a` | All connections |
| `b` | Executable (which program) |
| `o` | PID |
| `n` | Numeric (no hostname resolution) |

**Example insights:**
- Port 22 listening → SSH service running
- `svchost.exe` or `lsass.exe` using a port
- Established connections with remote servers

Useful for finding suspicious connections, checking open ports, identifying malware or unauthorized services.

---

## File & Disk Management

| Command | Description |
|---|---|
| `cd` | Display current directory |
| `cd <directory>` | Change to a target directory |
| `cd ..` | Go up one level |
| `dir` | View current directory contents |
| `dir /a` | Display hidden and system files |
| `dir /s` | Display files in directory and all subdirectories |
| `mkdir <name>` | Create a new directory |
| `rmdir <name>` | Remove a directory |
| `type <file>` | View contents of a text file |
| `copy <src> <dst>` | Copy files |
| `move <src> <dst>` | Move files |
| `del <file>` / `erase <file>` | Delete files |

---

## Task & Process Management

| Command | Description |
|---|---|
| `tasklist` | List all running processes |
| `tasklist /?` | Show tasklist help |
| `taskkill /PID <PID>` | Terminate a process by PID |
| `chkdsk` | Check file system and disk for errors and bad sectors |
| `driverquery` | List all installed drivers |
| `sfc /scannow` | Scan system files for corruption and repair if possible |

---

## Research & Documentation

### GitHub for Security Research
Many researchers upload PoC exploit code, scanners, and tools for vulnerabilities.

Search by:
- CVE ID (e.g., `CVE-2014-0160 exploit`)
- Vulnerability name
- Software version

Useful for:
- Checking multiple PoCs for the same CVE
- Reviewing exploit scripts
- Learning how vulnerabilities are exploited

### Official Documentation Sources

| Source | How to Access |
|---|---|
| Linux Manual Pages | `man <command>` in terminal |
| Windows Documentation | Microsoft's official docs website |
| Snort | Snort official documentation |
| Apache | Apache HTTP Server docs |
| PHP | php.net |
| Node.js | nodejs.org/docs |

> Always check official docs first — most accurate, most updated, most reliable. Avoid outdated blogs.

### Social Media for OSINT
- People often overshare — schools, birthdays, pets can reveal password reset answers.
- Security professionals use social media for exposure assessment, research, and threat intelligence.
- Use temporary emails when exploring platforms to avoid linking to your real identity.
- Follow cybersecurity pages and news sites to stay current on the threat landscape.
