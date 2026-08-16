# Linux Process Monitoring — Command Notes

Quick reference notes for viewing, saving, and searching running system processes on Linux.

## 1. Viewing & Saving Running Processes

**Basic process listing:**
```bash
ps aux
```
Shows all running processes — user, PID, %CPU, %MEM, command, etc.

**Save output to a file:**
```bash
ps aux > running_processes.txt
```
Redirects the process list into a text file.

> **Tip:** Avoid spaces in filenames (e.g. `running processes.txt` becomes two separate arguments in bash). Use underscores or hyphens instead: `running_processes.txt`.

## 2. Searching Within the Saved File

**grep — search by text pattern:**
```bash
grep "10" running_processes.txt
```
Finds lines containing "10" anywhere in the line. This is a substring match, so it can catch unintended matches (e.g. PID 100, 210, etc.).

**awk — search by specific column:**
```bash
awk '$2 == "10"' running_processes.txt      # exact match on PID column
awk '$2 == "1"' running_processes.txt       # find PID 1 (init/systemd)
```
`$2` refers to the 2nd column in `ps aux` output (the PID column). Using `==` gives an exact match instead of a loose substring match like `grep`.

## 3. pgrep — Search Live Processes by Name

```bash
pgrep nginx
```
Returns the PID(s) of currently running processes matching a name. This searches live processes directly — it does **not** search text files.

```bash
man pgrep
```
View the manual page for full usage and options.

## 4. Checking a Service's Status

```bash
systemctl status nginx
```
Shows whether a service (e.g. nginx) is active/inactive, recent logs, and its PID.

> Correct syntax: `systemctl status <service_name>` (not `systemctl <service_name> status`).

## 5. Live Monitoring

```bash
top
```
Classic real-time process viewer.

```bash
htop
```
Improved, color-coded, interactive version of `top` (may need to be installed separately).

## 6. Inspecting a Service Unit

```bash
systemctl cat nginx
```
Shows the actual unit file (config) used by systemd for the nginx service — useful for seeing exactly how the service is defined.

```bash
systemctl list-dependencies nginx
```
Shows a tree of services/units that nginx depends on (or that depend on it).

```bash
systemctl is-enabled nginx
```
Checks whether the service is enabled to start automatically at boot (`enabled` / `disabled`).

## 7. Restarting a Service

```bash
systemctl restart nginx
```
Attempts to restart the service. Usually requires elevated privileges.

```bash
sudo systemctl restart nginx
```
Restarts the service with root privileges — the correct way to do it if you're not already root.

```bash
systemctl status nginx
```
Check status immediately after restarting to confirm it's active and running without errors.

## 8. Viewing Service Logs — journalctl

```bash
journalctl -u nginx
```
Shows all logs for the nginx service (`-u` = filter by unit).

```bash
journalctl -u nginx -f
```
Follows the log in real-time (like `tail -f`) — useful for watching logs live as requests/errors come in.

```bash
journalctl -u nginx -n 5
journalctl -u nginx -n 50
```
Shows only the last N lines of logs (`-n`) — e.g. last 5 or last 50 entries. Handy for quickly checking recent activity without scrolling through everything.

```bash
journalctl
```
Shows the full system journal (all services/logs), not just one unit.

```bash
man journalctl
```
View the manual page for full usage and options.

---
 
## Troubleshooting Flow — "Service Won't Respond / Is Down"
 
A repeatable flow for diagnosing a misbehaving service (e.g., `nginx`, `myapp`).
 
```
1. Is it running?
   systemctl status nginx
   ps aux | grep nginx
        │
        ├── Not running / failed
        │       │
        │       ▼
        │   Check recent logs
        │   journalctl -u nginx -n 50
        │       │
        │       ▼
        │   Check the actual config loaded
        │   systemctl cat nginx
        │       │
        │       ▼
        │   Fix config / permissions / missing dependency
        │   Then: systemctl restart nginx
        │       │
        │       ▼
        │   Re-check: systemctl status nginx
        │
        └── Running, but not responding
                │
                ▼
            Is it listening on the expected port?
            lsof -i :8080   OR   ss -tulnp | grep 8080
                │
                ├── Nothing listening → app crashed silently
                │       → check journalctl -u nginx -f (live logs)
                │
                └── Something IS listening
                        │
                        ▼
                    Test locally
                    curl -v http://localhost:8080
                        │
                        ├── Works locally, fails externally
                        │       → check firewall (ufw/iptables), security groups, DNS
                        │
                        └── Fails even locally
                                → check app-level logs
                                → check resource limits (top/htop — CPU/mem maxed?)
                                → check disk space: df -h
```
 
### Command sequence (copy-paste order)
 
```bash
# 1. Status check
systemctl status nginx
 
# 2. Is it enabled to survive reboot?
systemctl is-enabled nginx
 
# 3. Recent logs — look for the actual error
journalctl -u nginx -n 50 --no-pager
 
# 4. Live logs while you retry the request
journalctl -u nginx -f
 
# 5. Port check
ss -tulnp | grep 8080
# or
lsof -i :8080
 
# 6. Local connectivity test
curl -v http://localhost:8080
 
# 7. Resource check
top
df -h
 
# 8. Restart if config was the issue
systemctl restart nginx
systemctl status nginx
```
 
### Golden rule
> **Status → Logs → Port → Connectivity → Resources.**
> Always start narrow (is it running?) before going broad (network/firewall/DNS). Most "it's down" issues resolve at step 1–3.
 
---

## Quick Reference Table

| Goal                        | Command                              |
|------------------------------|---------------------------------------|
| Snapshot all processes       | `ps aux`                              |
| Save snapshot to file        | `ps aux > file.txt`                   |
| Search file by exact column  | `awk '$2 == "PID"' file.txt`          |
| Search file by loose text    | `grep "text" file.txt`                |
| Find live PID by name        | `pgrep processname`                   |
| Check a service's status     | `systemctl status servicename`        |
| Live monitoring (basic)      | `top`                                 |
| Live monitoring (enhanced)   | `htop`                                |
| View service unit config     | `systemctl cat servicename`           |
| Restart a service            | `sudo systemctl restart servicename`  |
| Check if service auto-starts | `systemctl is-enabled servicename`    |
| View service dependencies    | `systemctl list-dependencies servicename` |
| View service logs            | `journalctl -u servicename`           |
| Follow service logs live     | `journalctl -u servicename -f`        |
| View last N log lines        | `journalctl -u servicename -n 50`     |
| View shell command history   | `history`                             |

---

*Notes compiled from personal command history while learning Linux process monitoring.*
