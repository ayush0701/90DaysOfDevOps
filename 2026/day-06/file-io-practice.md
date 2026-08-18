# Day 06 – Linux Fundamentals: Read and Write Text Files

## What I Did
Practiced basic file read/write in Linux using `touch`, `echo`, redirection (`>`, `>>`),
`tee`, `cat`, `head`, and `tail` on my AWS EC2 Ubuntu instance.

## Terminal Session

```bash
ubuntu@ip-172-31-41-39:~/devops$ touch notes.txt
ubuntu@ip-172-31-41-39:~/devops$ ls
notes.txt  running_processes.txt

ubuntu@ip-172-31-41-39:~/devops$ echo "Line 1 Devops Important tools" > notes.txt
ubuntu@ip-172-31-41-39:~/devops$ echo "Line 2 docker, kubernetes, linux, shell scripting" >> notes.txt

# Mistake: piped the WRONG way. `tee -a` should sit BEFORE the file, connected
# with a pipe ( | ), not written after >>. This line did not add "Line 3" to notes.txt.
ubuntu@ip-172-31-41-39:~/devops$ echo "Line 3 thanks for reading" >> tee -a notes.txt

ubuntu@ip-172-31-41-39:~/devops$ cat notes.txt
Line 1 Devops Important tools
Line 2 docker, kubernetes, linux, shell scripting
# <- Line 3 missing here, proving the command above failed silently

# Corrected version: pipe the echo output INTO tee -a
ubuntu@ip-172-31-41-39:~/devops$ echo "Line 3 thanks for reading" | tee -a notes.txt
Line 3 thanks for reading

ubuntu@ip-172-31-41-39:~/devops$ cat notes.txt
Line 1 Devops Important tools
Line 2 docker, kubernetes, linux, shell scripting
Line 3 thanks for reading

ubuntu@ip-172-31-41-39:~/devops$ head -n 2 notes.txt
Line 1 Devops Important tools
Line 2 docker, kubernetes, linux, shell scripting

ubuntu@ip-172-31-41-39:~/devops$ tail -n 1 notes.txt
Line 3 thanks for reading
```

## Final Content of notes.txt

```
Line 1 Devops Important tools
Line 2 docker, kubernetes, linux, shell scripting
Line 3 thanks for reading
```

## Commands Used & Why Each One Matters

### `touch notes.txt`
Creates an empty file instantly (or just updates its timestamp if it already exists).
**Why it matters:** In DevOps you constantly need placeholder files — for scripts,
config stubs, or log files — before a process starts writing to them.

### `echo "text" > file`
Writes text into a file, **overwriting** anything already there.
**Why it matters:** Used to reset or initialize a file — e.g. clearing an old log
before a fresh deployment run, or writing a fresh config value.

### `echo "text" >> file`
**Appends** text to the end of a file without touching existing content.
**Why it matters:** This is how logs grow — every new event gets appended, never
overwritten. Critical for audit trails, cron job outputs, and monitoring scripts.

### `echo "text" | tee -a file`
Sends output to the file **and** prints it to the terminal at the same time,
in append mode.
**Why it matters:** Extremely common in DevOps scripting — you want a command's
output saved to a log file while also watching it happen live, without running
two separate commands.
> ⚠️ **Gotcha I hit:** `tee -a` must come **after a pipe (`|`)**, not after `>>`.
> Writing `echo "text" >> tee -a notes.txt` doesn't work — it tries to redirect
> into a file literally named `tee`. The correct form is:
> `echo "text" | tee -a notes.txt`

### `cat file`
Prints the entire file content to the screen in one shot.
**Why it matters:** The fastest way to sanity-check a config or log file's full
contents during debugging.

### `head -n 2 file`
Shows only the first N lines (top of the file).
**Why it matters:** Useful for peeking at file headers, CSV column names, or the
start of a large log without scrolling through everything.

### `tail -n 1 file`
Shows only the last N lines (bottom of the file).
**Why it matters:** The go-to command for checking the most recent log entries —
often combined with `-f` (`tail -f`) to watch a log file live as new lines arrive.

## Key Takeaway
`>` overwrites, `>>` appends, and `tee` writes + displays at once — but `tee`
only works when piped **into** it (`command | tee -a file`), not redirected
**into** it with `>>`. Small syntax details like this matter a lot once you start
writing real deployment scripts.

#90DaysOfDevOps #DevOpsKaJosh #TrainWithShubham
