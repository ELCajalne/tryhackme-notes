# Windows Command Line ⌨️

---

## Table of Contents
- [Basic System Info](#basic-system-info)
- [Network Troubleshooting](#network-troubleshooting)
- [File & Disk Management](#file--disk-management)
- [Task & Process Management](#task--process-management)
- [Research & Documentation](#research--documentation)
- [PowerShell](#-powershell)
- [Linux Shells & Bash](#-linux-shells--bash)
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

## 💻 PowerShell

PowerShell is designed for task automation and configuration management.
- Combines CLI and scripting language built on **.NET**
- **Object-oriented** — handles complex data types and interacts with system components more effectively
- Originally Windows-only but now supports macOS and Linux

### Objects in PowerShell
In PowerShell, everything is an **object** — an item with:
- **Properties** (characteristics) — e.g. file name, size, extension
- **Methods** (actions) — e.g. copy a file, stop a process

> Example: A `car` object has properties like `Color` and `Model`, and methods like `Drive()` and `Refuel()`.

---

### Basic Commands

```powershell
# List all available cmdlets, functions, aliases, and scripts
Get-Command

# Filter by command type
Get-Command -CommandType Cmdlet

# Filter by verb (proper way)
Get-Command -Verb "Remove"

# Filter by name using wildcard
Get-Command -Name "Remove-*"
```

**`-Verb` vs `-Name` wildcard:**
| Parameter | Usage | Wildcard needed? |
|---|---|---|
| `-Verb "Remove"` | Filters by verb portion specifically | No |
| `-Name "Remove-*"` | Plain text pattern matching on full name | Yes (`*`) |

```powershell
# Get help for a cmdlet
Get-Help Get-Process

# Show usage examples
Get-Help Get-Process -Examples

# List all aliases
Get-Alias

# View all approved verbs
Get-Verb
```

**CommandType values:**
| Type | Description |
|---|---|
| `Cmdlet` | Compiled .NET commands (e.g. `Get-Item`) |
| `Function` | PowerShell functions |
| `Alias` | Shortcuts like `ls`, `dir` |
| `Script` | .ps1 script files |
| `Application` | External executables |

---

### Navigating the File System

```powershell
# List files and directories
dir
Get-ChildItem

# List specific path
dir -Path "C:\Users"

# Show hidden files
dir -Force

# Create a directory
New-Item -Path ".\folder\subfolder" -ItemType "Directory"

# Create a file
New-Item -Path ".\folder\file.txt" -ItemType "File"

# Remove a file or directory
Remove-Item -Path ".\folder\file.txt"

# Copy a file
Copy-Item -Path ".\file.txt" -Destination ".\file_copy.txt"

# Move a file
Move-Item -Path ".\file.txt" -Destination ".\newfolder\file.txt"

# Read file contents
Get-Content filename.txt
cat filename.txt
```

---

### Piping, Filtering, and Sorting

**Piping `|`** — passes the output of one command as the input to another. In PowerShell, it passes full **objects** (not just text), making it more powerful than traditional CLI piping.

```powershell
# Sort files by size
Get-ChildItem | Sort-Object Length

# Filter by file extension
Get-ChildItem | Where-Object -Property "Extension" -eq ".txt"

# Filter by name pattern
Get-ChildItem | Where-Object -Property "Name" -like "ship*"

# Show only specific properties
Get-ChildItem | Select-Object Name, Length

# Search for text in a file (like grep)
Select-String -Path ".\file.txt" -Pattern "keyword"

# Case sensitive search
Select-String -Path ".\file.txt" -Pattern "keyword" -CaseSensitive
```

**Comparison Operators:**
| Operator | Meaning |
|---|---|
| `-eq` | Equal to |
| `-ne` | Not equal |
| `-gt` | Greater than |
| `-ge` | Greater than or equal to |
| `-lt` | Less than |
| `-le` | Less than or equal to |
| `-like` | Match with wildcard pattern |

---

### System & Network Information

```powershell
# Comprehensive system info
Get-ComputerInfo

# List local user accounts
Get-LocalUser

# Network configuration
ipconfig
Get-NetIPConfiguration
Get-NetIPAddress

# View running processes
Get-Process

# View services
Get-Service

# View active TCP connections
Get-NetTCPConnection
Get-NetTCPConnection | Select-Object LocalAddress, LocalPort, RemoteAddress, State, OwningProcess

# Cross-reference PID with process name
Get-NetTCPConnection | Select-Object LocalPort, RemoteAddress, State, OwningProcess,
  @{Name="ProcessName"; Expression={(Get-Process -Id $_.OwningProcess).Name}}

# Generate file hash
Get-FileHash -Path .\file.txt
```

> **`OwningProcess`** — shows the Process ID (PID) of the process that owns the network connection.

---

### Scripting

Scripting automates tasks you would normally perform manually. PowerShell scripts use the `.ps1` extension.

**Why scripting matters in cybersecurity:**
- 🔵 **Blue Team** (incident response, threat hunting) — automate log analysis, detect anomalies, extract IOCs
- 🔴 **Red Team** (pen testing) — automate enumeration, execute remote commands, craft obfuscated scripts
- 🛠️ **Sysadmins** — automate integrity checks, enforce security policies, manage configurations

```powershell
# Run a script on a local computer
Invoke-Command -FilePath c:\scripts\test.ps1 -ComputerName Server01

# Run a script on a remote computer
Invoke-Command -ComputerName Server01 -Credential Domain01\User01 -ScriptBlock { Get-Culture }
```

---

## 🐧 Linux Shells & Bash

### Types of Shells

```bash
# Check current shell
echo $SHELL

# List all available shells
cat /etc/shells

# Permanently change default shell
chsh -s /usr/bin/zsh
```

| Shell | Full Name | Notable Features |
|---|---|---|
| `bash` | Bourne Again Shell | Tab completion, command history, widely used default |
| `fish` | Friendly Interactive Shell | Auto spell correction, colored syntax, beginner friendly |
| `zsh` | Z Shell | Advanced tab completion, spell correction, highly customizable |

```bash
# View command history
history
```

---

### Shell Scripting

A shell script is a set of commands saved in a file to automate tasks instead of running them one by one.

```bash
# Create a script
nano first_script.sh

# Make it executable
chmod +x first_script.sh

# Run the script
./first_script.sh
```

**Every script should start with a Shebang:**
```bash
#!/bin/bash
```
The `#!` followed by the interpreter path tells the system how to execute the script.

---

### Script Building Blocks

**Variables:**
```bash
#!/bin/bash
echo "Hey, what's your name?"
read name
echo "Welcome, $name"
```

**Loops:**
```bash
for i in {1..10};
do
  echo $i
done
```

**Conditional Statements:**
```bash
#!/bin/bash
echo "Who goes beyond the gates of NightCity?"
read name
if [ "$name" = "David Martinez" ]; then
    echo "Welcome to NightCity, Choom!"
else
    echo "Error! Corpo Alert! Error! Corpo Alert!"
fi
```
> ⚠️ Always put spaces inside `[ ]` brackets or bash will throw an error.

**Comments:**
```bash
# This is a comment — written for your own understanding
```

---

### Nano Editor Quick Reference

```
Open/create file:    nano filename.sh
Save (keep editing): Ctrl + O  →  Enter
Save and exit:       Ctrl + X  →  Y  →  Enter
Navigate:            Arrow keys
Delete:              Backspace
Cancel command:      Ctrl + C
```

---

### chmod (Change Mode)

Controls who can access a file and what they can do with it.

```bash
# Make a file executable
chmod +x script.sh

# Set specific permissions
chmod 755 script.sh
```

**Permission types:** `r` (read), `w` (write), `x` (execute)

**Number reference:**
| Number | Permission |
|---|---|
| `7` | read + write + execute |
| `5` | read + execute |
| `4` | read only |
| `0` | no permission |