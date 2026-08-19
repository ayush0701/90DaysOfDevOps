# Part 1:
# Linux File System Hierarchy

Linux organizes everything into a single tree rooted at `/`, unlike Windows' separate drive letters (`C:`, `D:`, etc.). This structure is standardized by the **Filesystem Hierarchy Standard (FHS)**, so most distributions look similar at the top level.

## Simple Analogy

Think of `/` as the **main entrance of a building**. Inside, there are different **departments (folders)**, each with a clear job.

| Folder | Simple Meaning | Real-life Analogy |
|---|---|---|
| `/` | The main entrance — root of everything | Building's front door |
| `/home` | Where each user's personal files live | Employee's private office/desk |
| `/root` | The admin's (root user's) personal folder | CEO's office |
| `/bin` | Everyday basic commands everyone needs | Toolbox with hammer, screwdriver |
| `/sbin` | Tools only admins use | Manager's special toolkit |
| `/etc` | Configuration files for the whole system | Company policy folder |
| `/var` | Data that keeps changing (logs, mail, cache) | Logbook, visitor register |
| `/tmp` | Temporary scratch files, cleared on reboot | Sticky notes, thrown away later |
| `/usr` | Installed user programs and data | Extra equipment rooms |
| `/opt` | Optional/third-party software | Rented machines from another company |
| `/dev` | Device files (disks, USB, terminals) | Doors to server room, parking lot |
| `/proc` | Live info about running processes | CCTV screen showing current activity |
| `/sys` | Structured kernel/device info | Building's control panel |
| `/boot` | Files needed to start the system (kernel, bootloader) | Main power switch + generator |
| `/mnt`, `/media` | Mount points for external/removable drives | Guest parking for visitors' cars |
| `/srv` | Data for services hosted on the system | Storage room for client deliveries |

## Directory Tree Overview

```
/
├── bin       → essential commands
├── boot      → kernel & bootloader
├── dev       → device files
├── etc       → configuration files
├── home      → user directories
├── lib       → shared libraries
├── media     → removable media mounts
├── mnt       → temporary mounts
├── opt       → optional software
├── proc      → virtual process info
├── root      → root user's home
├── run       → runtime data (PIDs, sockets)
├── sbin      → system admin binaries
├── srv       → service data
├── sys       → virtual kernel/device info
├── tmp       → temporary files
├── usr       → user programs & data
└── var       → variable/log data
```

## Three Buckets to Remember

1. **Needed to start/run the system** → `/bin`, `/sbin`, `/etc`, `/boot`, `/lib`
2. **Where people/programs keep their stuff** → `/home`, `/root`, `/usr`, `/opt`
3. **Things that change constantly or are temporary** → `/var`, `/tmp`, `/proc`, `/dev`

## Key Ideas

- **Static vs. variable**: `/usr` holds mostly static, installed software; `/var` holds data that changes constantly.
- **Essential vs. optional**: `/bin`, `/sbin`, `/etc` are needed for basic boot/recovery; `/usr`, `/opt` hold the "extra" stuff.
- **Real vs. virtual**: `/proc` and `/sys` aren't real files on disk — they're windows into live kernel/process state.

---

*Summary: Everything in Linux lives under one root folder `/`, and each subfolder has a specific job — some hold programs, some hold settings, some hold user files, and some just show live system info.*
# Part 2:
# Linux Server Troubleshooting — Scenarios & Commands

A practical cheat sheet of common Linux server troubleshooting scenarios, the commands used to diagnose them, and why each command matters.

---

## Scenario 1: Service Not Starting

**Situation:** A web application service called `myapp` failed to start after a server reboot.

**Diagnosis steps (in order):**

```bash
# 1. Check current status of the service
systemctl status myapp

# 2. Review recent logs for error messages
journalctl -u myapp -n 50 --no-pager

# 3. Verify the unit file / config is correct
systemctl cat myapp

# 4. Check system-wide logs around boot time for related errors
journalctl -xe

# 5. Attempt to restart after identifying/fixing the issue
systemctl restart myapp
```

**Why in this order:** Always check *status* and *logs* first to understand the failure before blindly restarting — restarting without knowing the cause just repeats the failure.

---

## Scenario 2: High CPU Usage

**Situation:** Manager reports the application server is slow. Need to identify which process is consuming high CPU.

**Diagnosis steps (in order):**

```bash
# 1. Check overall system load
uptime

# 2. View live CPU usage per process
htop
# or if htop isn't installed:
top

# 3. Get a snapshot of top CPU-consuming processes
ps aux --sort=-%cpu | head -10

# 4. Drill into a specific process over time (replace <PID>)
pidstat -p <PID> 1
```

**Note:** `du` is for disk usage, not CPU — don't confuse the two when diagnosing performance issues.

---

## Scenario 3: Finding Service Logs

**Situation:** A developer asks where the logs for the `docker` service (managed by systemd) are.

**Diagnosis steps (in order):**

```bash
# 1. View full log history for the service
journalctl -u docker

# 2. View the most recent entries
journalctl -u docker -n 50

# 3. Tail logs live (follow mode)
journalctl -u docker -f

# 4. Optional: filter logs by time range
journalctl -u docker --since "1 hour ago"
```

---

## Scenario 4: File Permissions Issue

**Situation:** A script at `/home/user/backup.sh` fails to execute with `Permission denied`.

**Diagnosis steps (in order):**

```bash
# 1. Check current file permissions
ls -l /home/user/backup.sh

# 2. Add execute permission (safer than chmod 770, which also changes read/write)
chmod +x /home/user/backup.sh

# 3. Verify the execute bit is now set
ls -l /home/user/backup.sh

# 4. Run the script again to confirm it works
./backup.sh
```

**Note:** Prefer `chmod +x` over `chmod 770` unless you specifically need to change group/owner read-write permissions too — the goal here is just to make the file executable.

---

## Quick Reference Table

| Scenario | Key Commands |
|---|---|
| Service not starting | `systemctl status`, `journalctl -u`, `systemctl cat`, `systemctl restart` |
| High CPU usage | `uptime`, `htop`/`top`, `ps aux --sort=-%cpu`, `pidstat` |
| Finding service logs | `journalctl -u <service>`, `-n`, `-f`, `--since` |
| File permission issues | `ls -l`, `chmod +x`, verify, re-run |
