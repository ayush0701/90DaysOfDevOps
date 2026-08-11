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

## File Management

| Command | Purpose | Common Example |
| :--- | :--- | :--- |
| `df` | Displays available and used disk space on mounted filesystems. | • `df -h`: Human-readable format (shows sizes in GB/MB instead of blocks).<br>• `df -T`: Shows the filesystem type (e.g., ext4, xfs, btrfs). |
| `du` | Estimates file and directory space usage. | • `du -sh *`: Displays a summary (`-s`) in human-readable format (`-h`) of all items in current directory.<br>• `du -h --max-depth=1 /var`: Shows disk space consumed by top-level directories inside `/var`. |
| `chmod` | Manages file and directory permissions. | • `chmod 755 script.sh`: Sets read/write/execute permissions (User: rwx, Group: r-x, Others: r-x).<br>• `chmod -R 644 /var/www/html`: Recursively applies read/write permissions to files inside a directory. |
| `chown` | Changes file or directory ownership. | `chown -R www-data:www-data /var/www/html`: Changes owner and group recursively. |
| `pwd` | Displays your current working directory. | `pwd` |
| `ls -la` | Shows a detailed list view including hidden files. | `ls -la` |
| `rm` | Removes files or directories. | `rm -r` (Deletes files or directories recursively) |

## Network Troubleshooting

| Command / Request | Purpose / Description |
| :--- | :--- |
| `netstat -tulpn` | Lists all the ports your server is currently listening on. |
| `telnet localhost 22` | Checks if the SSH port is open locally. |
| `nslookup google.com` | Queries DNS to find the IP address. |
| `ufw status` | Checks firewall status (`sudo ufw status`). |
| `ping google.com` | Built-in network diagnostic tool used to **test host reachability** and **measure round-trip data latency**. |
| `traceroute google.com` | Identifies each hop (router) along the path to a destination to pinpoint where packet loss occurs. |
| `curl https://api.github.com/users/octocat` | GET Request (Fetches page / JSON content). |
| `curl -X POST -d "username=john&password=secret" https://example.com/login` | POST Request with Form Data. |
| `curl -X POST -H "Content-Type: application/json" -d '{"name": "Alice", "role": "admin"}' https://api.example.com/users` | POST Request with JSON Data. |
| `curl -O https://example.com/archive.tar.gz` | Downloads file. |
