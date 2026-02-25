# Windows Fundamentals 🪟

---

## Table of Contents
- [Part 1 — File System, Accounts, UAC](#part-1--file-system-accounts-and-uac)
- [Part 2 — System Tools, Registry](#part-2--system-tools-and-registry)
- [Part 3 — Updates, Security, BitLocker](#part-3--updates-security-and-bitlocker)

---

## Part 1 — File System, Accounts and UAC

### The Desktop (GUI)
Includes Desktop, Start Menu, Search Box, Taskbar, Task View, Toolbars, and Notification Area. Right-click almost anything to customize.

### Remote Desktop Connection (RDC)
Open `mstsc` → enter IP address, username, and password to connect remotely.

### The File System — NTFS
Modern Windows uses **NTFS (New Technology File System)**. Older systems used FAT16/FAT32 or HPFS.

**NTFS advantages:**
- Supports files larger than 4GB
- File and folder permissions
- File compression
- EFS encryption
- Journaling — automatically repairs files after failures

Check your file system: Right-click `C:\` drive → Properties.

### NTFS Permissions

| Permission | Description |
|---|---|
| Full Control | Complete access — read, write, modify, delete, change permissions |
| Modify | Read, write, and delete |
| Read & Execute | View and run files |
| List Folder Contents | View folder contents |
| Read | View files only |
| Write | Add files and subfolders |

View: Right-click file/folder → Properties → Security tab.

### NTFS Alternate Data Streams (ADS)
Hidden data streams attached to files. Not visible in Windows Explorer.
- Malware hides data or code in ADS.
- Windows uses ADS legitimately for metadata (e.g., "downloaded from internet" flag).
- Revealed using PowerShell or forensic tools.

### Windows & System32

| Path | Description |
|---|---|
| `C:\Windows` | Core OS files. Accessible via `%windir%` |
| `C:\Windows\System32` | Critical files, commands, drivers, DLLs. Deleting here can break Windows permanently. |

Most admin/security tools (`tasklist`, `net`, `sc.exe`) are inside System32.

### Environment Variables

| Variable | Description |
|---|---|
| `%windir%` | Windows installation directory |
| `%TEMP%` | Temporary folder |
| `%PATH%` | Where Windows searches for executables |
| `%USERPROFILE%` | Current user's profile folder |

### User Accounts & Permissions

| Type | Description |
|---|---|
| Administrator | Full system control — install software, change settings, manage users |
| Standard User | Limited to personal files and settings |

User profiles stored at: `C:\Users\<username>`

Manage with: `lusrmgr.msc`

> Users inherit permissions from every group they belong to — group membership is crucial for security and privilege escalation.

### User Account Control (UAC)
Prevents unauthorized system changes. Even admins run with standard privileges by default — UAC prompts when elevated access is needed.

Key points:
- Standard users must enter admin credentials to proceed.
- Built-in Administrator account bypasses UAC by default.
- Helps block malware from making silent system-wide changes.

### Settings vs Control Panel

| Tool | Description |
|---|---|
| Settings | User-friendly. Introduced in Windows 8. Most common changes. |
| Control Panel | Classic menu. Deeper system configuration. |

### Task Manager
Open: `Ctrl + Shift + Esc` or right-click Taskbar.

| Tab | Description |
|---|---|
| Processes | All running apps and services |
| Performance | Live graphs — CPU, memory, disk, network |
| Startup | Programs that run at boot |
| Users | Per-user resource usage |
| Details | Every process with PID — crucial for investigations |
| Services | Manage Windows services |

> Cybersecurity use: spot malicious processes, unusual CPU spikes, hidden background tasks. Essential for malware analysis and incident response.

---

## Part 2 — System Tools and Registry

### System Configuration — MSConfig
Troubleshooting utility for startup and boot issues. Requires admin rights.

Open: `msconfig` from Start Menu or CMD.

| Tab | Purpose |
|---|---|
| General | Normal, Diagnostic, or Selective startup |
| Boot | Safe boot options, logging, timeout |
| Services | Enable/disable services to isolate issues |
| Startup | Redirects to Task Manager |
| Tools | Hub of built-in Windows utilities with exact run commands |

> Cybersecurity use: disable malicious startup behaviors, inspect services malware may hide in.

### Computer Management — `compmgmt.msc`

**System Tools:**

| Tool | Description |
|---|---|
| Task Scheduler | Automates tasks. Important for detecting malware persistence. |
| Event Viewer | All system, security, and application events. Used for forensics and intrusion investigation. |
| Shared Folders | All shares, active sessions, open files. Useful for detecting lateral movement. |
| Local Users and Groups | Same as `lusrmgr.msc` |
| Performance Monitor | Real-time CPU, memory, disk, network monitoring |
| Device Manager | View/manage hardware and drivers |

**Event Types:** Error, Warning, Information, Success Audit, Failure Audit

**Key Log Categories:**
- **Application** — App errors and behavior
- **Security** — Logons, privilege use, file access attempts
- **System** — OS-level events, drivers, failures
- **Custom Logs** — Application-generated logs

**Storage:** Disk Management — create/extend/shrink partitions, assign drive letters. Useful for disk forensics.

**Services and Applications:**
- **Services** — Start, stop, disable background processes. Critical for understanding persistence.
- **WMI Control** — `wmic` is deprecated → use PowerShell + WMI cmdlets instead.

### System Information — `msinfo32`

| Section | Description |
|---|---|
| System Summary | OS version, CPU, BIOS, RAM |
| Hardware Resources | IRQs, DMA, I/O addresses |
| Components | GPU, input devices, network adapters, storage |
| Software Environment | Drivers, running tasks, environment variables, network connections |

> Search bar at the bottom lets you search across all sections. Example: search "IP Address" to find network interface info.

### Resource Monitor — `resmon.exe`
Deeper insight than Task Manager. Four sections: CPU, Memory, Disk, Network — with live graphs.

Useful for finding which process is slowing your PC, using your network, or causing deadlocks.

### Command Prompt — Key Commands

| Command | Description |
|---|---|
| `hostname` | Shows the computer's name |
| `whoami` | Shows the currently logged-on user |
| `ipconfig` | Network info (IP, gateway, subnet mask) |
| `ipconfig /all` | Full details — DNS, DHCP, all adapters |
| `netstat` | Active network connections |
| `netstat -abon` | All connections with executable, PID, numeric format |
| `net` | Parent command for network management |
| `net help` | Help for net commands |
| `cls` | Clears the CMD screen |

### Registry Editor — `regedit`
The Windows Registry is the database of the OS — storing settings for the system, users, apps, and hardware.

**What it stores:**
- User profiles and settings
- Installed applications and file type associations
- Folder and icon settings
- Hardware information and drivers
- Ports being used

> ⚠️ Changing the wrong value can break Windows permanently.

**Cybersecurity relevance:**
- How malware hides persistence
- Where Windows stores system secrets
- How attackers modify system behavior
- Forensic investigation of system changes
- Privilege escalation and persistence in penetration testing

---

## Part 3 — Updates, Security and BitLocker

### Windows Updates
- **Patch Tuesday** — Monthly updates (every 2nd Tuesday). Critical patches can be released anytime.
- Access: Settings → Update & Security → Windows Update
- Windows 10+ forces updates — you can delay but not permanently disable.

### Windows Security
Built-in hub for all protection tools.

**Status Icons:** 🟢 Green (fine) | 🟡 Yellow (review needed) | 🔴 Red (immediate action required)

### Virus & Threat Protection

**Scan Types:**

| Scan | Description |
|---|---|
| Quick Scan | Scans common malware locations. Fast. |
| Full Scan | Scans everything. Slow. |
| Custom Scan | You choose what to scan. |

**Threat History:**
- **Quarantined Threats** — Isolated files, blocked from running.
- **Allowed Threats** — Manually allowed. Dangerous unless you're 100% sure.

**Protection Settings:**

| Setting | Description |
|---|---|
| Real-time Protection | Detects and stops malware instantly |
| Cloud-delivered Protection | Faster detection via Microsoft cloud |
| Automatic Sample Submission | Sends suspicious files to Microsoft |
| Controlled Folder Access | Blocks unauthorized changes to protected folders (anti-ransomware) |
| Exclusions | Files/folders Defender will not scan |

> ⚠️ Exclusions and Allowed Threats are risky unless you know exactly what you're doing.

You can manually scan any file by right-clicking → Scan with Microsoft Defender.

### Windows Firewall Profiles

| Profile | When It Applies |
|---|---|
| Domain | Corporate network with domain controller |
| Private | Home or trusted private network |
| Public | Public Wi-Fi — most restrictive by default |

Advanced settings: `WF.msc`

> ⚠️ Only turn off the firewall if you are 100% sure — otherwise leave it enabled.

### App & Browser Control — SmartScreen
Checks apps, files, and websites to prevent phishing, malware, and malicious downloads. Shows a warning before running suspicious files.

### Exploit Protection
Built into Windows 10 / Server 2019. Protects against attacks targeting software vulnerabilities.

### Device Security

**Core Isolation** — Uses virtualization-based security (VBS) to isolate critical parts of Windows.

**Memory Integrity** — Prevents attackers from injecting malicious code into high-security processes. Protects against kernel-level malware and rootkits.

> ⚠️ Leave Memory Integrity ON unless you have a specific lab reason to disable it.

### Security Processor — TPM
**TPM (Trusted Platform Module)** — Hardware chip that securely stores keys, passwords, and certificates. Used for BitLocker, Windows Hello, secure boot, and device identity verification.

### BitLocker
Full-disk encryption. Drive is unreadable without the correct key.

Unlock methods: TPM, PIN, password, or recovery key.

Protects data even if an attacker removes the hard drive or boots from another OS.

### Volume Shadow Copy Service (VSS)
Creates point-in-time snapshots (shadow copies) for backups and system restore. Stored in the System Volume Information folder.

> ⚠️ Ransomware often deletes shadow copies to prevent recovery — offline and off-site backups are essential.
