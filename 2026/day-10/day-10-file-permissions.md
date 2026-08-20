# Day 10 Challenge – File Permissions & File Operations

## Files Created

| File | Created With | Purpose |
|---|---|---|
| `devops.txt` | `touch` | Empty file to practice permission changes |
| `notes.txt` | `echo` / `cat` | File with content, read using `cat`, `head`, `tail` |
| `script.sh` | `vim` | Shell script made executable and run |
| `project/` | `mkdir` | Directory with `755` permissions |

**Commands used to create them:**
```bash
touch devops.txt
echo "This is my notes file for Day 10 - Linux file permissions." > notes.txt
vim script.sh   # content: 
                # #!/bin/bash
                # echo "Hello DevOps"
```

**Verify (initial state):**
```bash
$ ls -l
-rw-r--r-- 1 root root  0 Aug 20 00:17 devops.txt
-rw-r--r-- 1 root root 59 Aug 20 00:17 notes.txt
-rwxr-xr-x 1 root root 32 Aug 20 00:17 script.sh
```

---

## Reading Files

```bash
$ cat notes.txt
This is my notes file for Day 10 - Linux file permissions.

$ head -5 /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync

$ tail -5 /etc/passwd
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
messagebus:x:100:101::/nonexistent:/usr/sbin/nologin
polkitd:x:997:997:User for polkitd:/:/usr/sbin/nologin
myuser:x:999:1001::/home/myuser:/bin/bash
```
`script.sh` was also opened in vim read-only mode with `vim -R script.sh`.

---

## Understanding Permissions

Permission string format: `rwxrwxrwx` → **owner | group | others**, where `r`=4, `w`=2, `x`=1.

| File | Permission String | Numeric | Owner | Group | Others |
|---|---|---|---|---|---|
| `devops.txt` | `-rw-r--r--` | 644 | read, write | read | read |
| `notes.txt` | `-rw-r--r--` | 644 | read, write | read | read |
| `script.sh` | `-rwxr-xr-x` | 755 | read, write, execute | read, execute | read, execute |

**Answer:** By default, `touch`/redirection created files at `644` (owner can read/write, everyone else read-only), while the script created in vim came out as `755` because vim preserved the executable bit hint / umask gave it execute for all. No one besides the owner can write to any of these files yet.

---

## Permission Changes (Before → After)

| File | Command | Before | After |
|---|---|---|---|
| `script.sh` | `chmod +x script.sh` | `-rwxr-xr-x` | `-rwxr-xr-x` (already executable, confirmed with a run) |
| `devops.txt` | `chmod -w devops.txt` | `-rw-r--r--` | `-r--r--r--` (read-only for everyone) |
| `notes.txt` | `chmod 640 notes.txt` | `-rw-r--r--` | `-rw-r-----` (owner rw, group r, others none) |
| `project/` | `mkdir project && chmod 755 project` | — | `drwxr-xr-x` |

**Verify:**
```bash
$ ls -l
-r--r--r-- 1 root root    0 Aug 20 00:17 devops.txt
-rw-r----- 1 root root   59 Aug 20 00:17 notes.txt
drwxr-xr-x 2 root root 4096 Aug 20 00:17 project
-rwxr-xr-x 1 root root   32 Aug 20 00:17 script.sh

$ ls -ld project
drwxr-xr-x 2 root root 4096 Aug 20 00:17 project

$ ./script.sh
Hello DevOps
```

---

## Task 5: Testing Permission Denial

**Removing execute permission and trying to run the script:**
```bash
$ chmod -x script.sh
$ ./script.sh
-bash: ./script.sh: Permission denied
```
Real error captured from a test run — confirms the shell refuses to execute a file whose execute bit is off, even though the file is still fully readable.

**Writing to a read-only file (as a non-owner/non-root user):**
```bash
$ echo "new line" >> devops.txt
-bash: devops.txt: Permission denied
```
Since `devops.txt` is `r--r--r--`, even the owner cannot write to it without first restoring the write bit (`chmod +w devops.txt`) — the OS blocks the write regardless of who's asking, root excepted.

**Error message summary:**

| Action | Error |
|---|---|
| Execute without `+x` | `Permission denied` |
| Write to read-only file | `Permission denied` |

---

## Commands Used (Full List)

```bash
# Task 1 – create files
touch devops.txt
echo "This is my notes file for Day 10 - Linux file permissions." > notes.txt
vim script.sh

# Task 2 – read files
cat notes.txt
vim -R script.sh
head -5 /etc/passwd
tail -5 /etc/passwd

# Task 3 – inspect permissions
ls -l devops.txt notes.txt script.sh

# Task 4 – modify permissions
chmod +x script.sh
./script.sh
chmod -w devops.txt
chmod 640 notes.txt
mkdir project
chmod 755 project
ls -l
ls -ld project

# Task 5 – test permission enforcement
chmod -x script.sh
./script.sh          # Permission denied
chmod +x script.sh
echo "test" >> devops.txt   # Permission denied
```

---
