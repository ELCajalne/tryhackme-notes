# Linux Fundamentals 🐧

---

## Table of Contents
- [Part 1 — Basic Navigation](#part-1--basic-navigation)
- [Part 2 — SSH, Files, Directories](#part-2--ssh-files-and-directories)
- [Part 3 — Editors, Processes, Automation](#part-3--editors-processes-and-automation)

---

## Part 1 — Basic Navigation

### Core Commands

| Command | Description |
|---|---|
| `ls` | List contents of current directory |
| `ls <directory>` | List contents of a specific directory |
| `cd <directory>` | Change to a specific directory |
| `pwd` | Print Working Directory — shows full current path |
| `cat <filename>` | Display contents of a file |
| `cat /path/to/file` | Display file using full path |

### Searching for Files

`find -name <filename>` — Search recursively from current directory.
```
find -name passwords.txt
find -name *.txt
```
`*` = wildcard (matches anything)

`grep` — Search contents of files for specific values.
```
grep "81.143.211.90" access.log
```

### Shell Operators

| Operator | Description |
|---|---|
| `&` | Run a command in the background |
| `&&` | Run commands in sequence — second only runs if first succeeds |
| `>` | Redirect output to a file (overwrites) |
| `>>` | Redirect output to a file (appends) |

Examples:
```
echo hey > welcome        # Creates file with "hey". Overwrites if exists.
echo hello >> welcome     # Appends "hello" to the file.
```

---

## Part 2 — SSH, Files and Directories

### SSH (Secure Shell)
Connects to and interacts with the command line of a remote Linux machine using encrypted communication.
```
ssh <username>@<IP address>
```
> Password input shows no visible feedback — this is normal. Type and press Enter.

### Flags and Switches
Most commands accept flags that modify default behavior.
```
ls -a          # Show hidden files (starting with .)
ls --help      # Show available options
man ls         # Full manual page
```

### Filesystem Commands

| Command | Description |
|---|---|
| `touch <filename>` | Create a new blank file |
| `mkdir <foldername>` | Create a new folder |
| `cp <source> <destination>` | Copy a file or folder |
| `mv <source> <destination>` | Move or rename a file or folder |
| `rm <filename>` | Delete a file permanently |
| `rm -R <foldername>` | Delete a folder and all its contents |
| `file <filename>` | Determine the type of a file |

> ⚠️ `rm` is permanent — no recycle bin.

> 💡 `cp` duplicates. `mv` transfers or renames without leaving a copy.

> 💡 In Linux, file extensions are optional. `file` checks actual content, not just the name.

### Switching Between Users
```
su <username>       # Switch to another user
su -l <username>    # Switch and load that user's environment fully
```

### Common Directories

| Directory | Description |
|---|---|
| `/etc` | System config files. Contains `sudoers`, `passwd`, `shadow` (encrypted passwords) |
| `/var` | Variable data — logs at `/var/log` |
| `/root` | Home directory for the root user (not `/home/root`) |
| `/tmp` | Temporary storage — cleared on reboot. Writable by all users. Useful for storing pentest data. |

---

## Part 3 — Editors, Processes and Automation

### Connecting to Remote Machines
```
ssh <username>@<IP address>
```

### Terminal Text Editors

**Nano** — Simple and beginner-friendly.
```
nano <filename>
```
- Arrow keys to navigate.
- `Ctrl + X` to exit.
- Features: search, copy/paste, jump to line number.

**VIM** — Advanced editor.
- Customizable keyboard shortcuts.
- Syntax highlighting for code.
- Works on all terminals (nano may not always be installed).

### Downloading Files — wget
Downloads files from the internet using HTTP, HTTPS, or FTP.
```
wget <URL>
wget https://assets.tryhackme.com/additional/linux-fundamentals/part3/myfile.txt
```

### Serving Files with Python
Turn any folder into a temporary web server.
```
python3 -m http.server
```
- Default port: **8000**
- Stop with `Ctrl + C`

Download from another terminal:
```
wget http://<YOUR_IP>:8000/<filename>
```

> ⚠️ Requires two terminals — one for the server, one for downloading. No file index — you must know the exact filename.

### Processes

| Command | Description |
|---|---|
| `ps` | Processes in current terminal session |
| `ps aux` | All processes on the system |
| `top` | Real-time process monitoring — updates every 10 seconds |
| `kill <PID>` | Stop a process |

**Kill Signals:**

| Signal | Description |
|---|---|
| SIGTERM | Polite stop — lets process clean up |
| SIGKILL | Force stop — no cleanup |
| SIGSTOP | Pause/suspend a process |

**Process Hierarchy:**
- PID 0 — Kernel process (starts at boot)
- PID 1 — `systemd` (main init process)
- All other processes are children of systemd

### Managing Services — systemctl
```
systemctl start apache2      # Start service now
systemctl stop apache2       # Stop service
systemctl enable apache2     # Auto-start on boot
systemctl disable apache2    # Prevent auto-start on boot
```

### Background Processes

| Action | Command |
|---|---|
| Run in background | `command &` |
| Suspend running process | `Ctrl + Z` |
| Bring to foreground | `fg` |

### Cron Jobs — Automation
**Cron** — Runs tasks automatically at scheduled times.
**Crontab** — The file where tasks are listed.

```
MIN HOUR DOM MON DOW CMD
```

| Field | Meaning |
|---|---|
| MIN | Minute (0–59) |
| HOUR | Hour (0–23) |
| DOM | Day of month (1–31) |
| MON | Month (1–12) |
| DOW | Day of week (0–6, Sun=0) |
| CMD | Command to run |

`*` = any value (don't care about this field)

Example — backup every 12 hours:
```
0 */12 * * * cp -R /home/cmnatic/Documents /var/backups/
```

Edit crontab: `crontab -e`

### Package Management — APT
```
sudo apt install <package-name>    # Install a package
sudo apt remove <package-name>     # Remove a package
sudo apt update                    # Update package list
```

### Logs — `/var/log`

| Log | Purpose |
|---|---|
| `/var/log/apache2/access.log` | Every web request |
| `/var/log/apache2/error.log` | Web server errors |
| `/var/log/auth.log` | Authentication attempts |
| fail2ban logs | Brute-force monitoring |
| UFW logs | Firewall connections |

Logs are automatically **rotated** (archived or deleted) to save disk space.

Useful for: monitoring system health, detecting security issues, debugging, investigating intrusions.
