## Core Process Management Commands

| Command | Purpose | Common Example |
| :--- | :--- | :--- |
| `ps` | Displays a static snapshot of current active processes. | `ps aux` (Lists all system processes with details) |
| `top` | Displays a dynamic, real-time list of running processes. | `top` (Updates live by CPU/memory usage)<br>• `M` -> sort by memory<br>• `P` -> sort by CPU<br>• `K` -> kill a process by typing its PID |
| `htop` | Interactive, colorful, and user-friendly version of `top`. | `htop` (Allows scrolling and mouse clicks) |
| `pgrep` | Searches for a process based on its name and returns the ID. | `pgrep nginx` (Finds the process ID of nginx) |
| `pidof` | Finds the exact Process ID (PID) of a running program. | `pidof bash` (Returns the PID for bash) |
| `kill` | Sends a specific termination signal to a process using its PID. | `kill -9 1234` (Force-stops process ID 1234) |
| `pkill` | Terminates processes based on their name instead of PID. | `pkill chrome` (Kills all Chrome processes) |
| `killall` | Terminates all processes matching the exact name. | `killall sleep` (Kills all processes named sleep) |
| `systemctl` | Manages system background services and processes. | `systemctl status sshd` (Checks SSH service status) |
| `nice` | Adjusts process CPU scheduling priority (Nice values range from **-20** [highest priority] to **19** [lowest priority]). | `nice -n -10 command` (Starts a command with high priority) |
| `renice` | Increases or changes the CPU priority of an already running process. | `renice -n -5 -p 1234` (Changes priority for PID 1234) |
