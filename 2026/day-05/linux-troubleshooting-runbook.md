# Linux Troubleshooting Runbook – Day 05

## Target service / process
**Service chosen:** `sshd` (SSH daemon)
**Why:** Actively in use for remote access to this EC2 instance (Ubuntu 26.04, AWS). Guaranteed to be running and easy to correlate logs → process → network port.

**Environment (host):** `ip-172-31-41-39`, Ubuntu 26.04 LTS "Resolute Raccoon", kernel `7.0.0-1010-aws`, EC2 instance.

---

## Environment basics

```bash
uname -a
```
```
Linux ip-172-31-41-39 7.0.0-1010-aws #10-Ubuntu SMP PREEMPT Thu Jul 23 02:06:04 UTC 2026 x86_64 GNU/Linux
```
**Note:** AWS-tuned kernel (`-aws` suffix), 64-bit architecture. Confirms exact kernel build in case a kernel-level bug/CVE needs checking later.

```bash
cat /etc/os-release
```
```
PRETTY_NAME="Ubuntu 26.04 LTS"
VERSION="26.04 LTS (Resolute Raccoon)"
```
**Note:** Ubuntu 26.04 LTS confirmed — matches the expected EC2 AMI. Good baseline to know when checking package/CVE compatibility later.

---

## Filesystem sanity

```bash
mkdir /tmp/runbook-demo
cp /etc/hosts /tmp/runbook-demo/host-copy && ls -l /tmp/runbook-demo/
```
```
-rw-r--r-- 1 ubuntu ubuntu 221 Aug 18 01:49 host-copy
```
**Note:** `/tmp` is writable and not full — copy succeeded instantly. This is a cheap early sanity check: if `mkdir`/`cp` had hung or failed, that alone would point to disk-full or read-only filesystem issues before touching anything else.

---

## Snapshot: CPU & Memory

```bash
ps -o pid,pcpu,pmem,comm -p $(pgrep -o sshd)
```
```
PID %CPU %MEM COMMAND
659  0.0  0.8 sshd
```
**Note:** `sshd` is essentially idle — 0% CPU, under 1% memory. No sign of resource contention or a runaway process.

```bash
free -h
```
```
              total   used   free   shared  buff/cache  available
Mem:          908Mi   315Mi  245Mi   2.7Mi   457Mi       592Mi
Swap:            0B      0B     0B
```
**Note:** ~908 MiB total RAM (small instance, likely t3.micro/t4g.micro). ~592 MiB available, no swap in use — memory is healthy, no pressure.

---

## Snapshot: Disk & IO

```bash
df -h
df -T
```
```
/dev/root  19G  2.8G  16G  15% /
/dev/nvme0n1p13  989M  163M  760M  18% /boot
```
**Note:** Root volume at only 15% used (16G free). `/boot` at 18% — plenty of headroom on both. No disk-full risk currently.

```bash
sudo du -sh /var/log/
```
```
30M     /var/log/
```
**Note:** Logs total only 30M — not eating disk space. (Note: ran without `sudo` first and got `Permission denied` on `/var/log/private`, `/var/log/chrony`, `/var/log/amazon` — a good reminder that some log dirs are root-only by design.)

---

## Snapshot: Network

```bash
sudo ss -tulpn | grep ssh
```
```
tcp LISTEN 0 4096   0.0.0.0:22  0.0.0.0:*  users:(("sshd",pid=659,fd=3),("systemd",pid=1,fd=100))
tcp LISTEN 0 4096      [::]:22     [::]:*  users:(("sshd",pid=659,fd=4),("systemd",pid=1,fd=101))
```
**Note:** `sshd` (PID 659) confirmed listening on port 22 for both IPv4 and IPv6, matching the PID seen in `ps`. (First attempt without `sudo` returned nothing — `ss` needs elevated privileges to show the process/PID column for other users' sockets.)

```bash
curl -I http://localhost
ping -c 4 8.8.8.8
```
```
HTTP/1.1 200 OK
Server: nginx/1.28.3 (Ubuntu)
---
4 packets transmitted, 4 received, 0% packet loss, rtt avg 1.289 ms
```
**Note:** Nginx is also running and serving on port 80 (separate from the ssh target — noted as a secondary "web tier is healthy" signal). Outbound network to `8.8.8.8` is solid: 0% packet loss, ~1.3ms latency.

---

## Logs reviewed

```bash
journalctl -u ssh -n 5
```
```
Aug 18 01:45:40 sshd-session[1504]: pam_unix(sshd:session): session opened for user ubuntu(uid=1000) by ubuntu(uid=0)
Aug 18 02:09:57 sshd-session[2031]: Connection closed by 190.173.26.85 port 56146
Aug 18 02:10:43 sshd-session[2037]: Connection closed by 190.173.26.85 port 51168
Aug 18 02:15:35 sshd-session[2127]: Connection closed by 45.29.150.182 port 53551
Aug 18 02:16:12 sshd-session[2129]: Connection closed by 45.29.150.182 port 60590
```
**Note:** My own login session opened cleanly at 01:45. The connections closed from `190.173.26.85` and `45.29.150.182` afterward are **not my traffic** — these are unauthenticated IPs hitting port 22 and disconnecting without logging in, a classic pattern of automated internet-wide SSH scanning/bots. Not an active compromise, but worth keeping an eye on.

**Full history check:** `journalctl -u ssh` (unfiltered) also showed the service has been cleanly stopped/restarted several times since Jul 26 (likely scheduled reboots or manual restarts) — each stop/start pair is clean with no crash signatures.

---

## Quick findings

- `sshd` is healthy: idle CPU/memory, listening correctly on port 22 (v4 + v6), no crash or error entries in logs.
- System resources overall are healthy — memory has headroom, disk usage is low (15-18%), no swap pressure.
- Logs show recurring connection attempts from unfamiliar IPs that close without authenticating — normal internet background noise, but a signal worth tracking over time.
- Two services observed running (`sshd` + `nginx`); should isolate to one target service in future drills to keep the runbook clean.

---

## If this worsens (next steps)

1. **Tighten SSH exposure** — if scan traffic from `journalctl` increases or an authenticated session appears from an unrecognized IP, restrict the security group to known IPs and/or enable `fail2ban` to auto-block repeated failed attempts.
2. **Increase log verbosity** — set `LogLevel VERBOSE` in `/etc/ssh/sshd_config`, then `sudo systemctl restart ssh` and re-check `journalctl -u ssh -n 20` for more detail on auth attempts.
3. **Deeper trace if unresponsive** — `sudo strace -p 659` to see live syscalls if sshd hangs, or check `/var/log/auth.log` directly for a longer authentication history than journalctl's default window.

---

*Drill completed on: Aug 18, 2026 — 90DaysOfDevOps Day 05*
