1. The core components of Linux (kernel, user space, init/systemd):
  Kernel: the central core program of a Linux operating system, acting as the direct bridge between computer hardware and software applications. It handles system resources, processes, and memory.
  User space: in Linux is the memory area and execution mode where normal applications, user interfaces, and background daemons run with restricted privileges
INIT/SYSTEMD: An init system is the very first process started by the Linux kernel during boot, assigned Process ID 1 (PID 1). It acts as the parent of all other processes and is responsible for bringing the system to a usable state.
 systemd: is the modern, highly popular initialisation system and service manager that has replaced traditional init frameworks across most major Linux distributions.
 example: systemctl: The definitive administrative CLI to control, start, stop, enable, or disable units

 Q2:
  Process States:
  The Linux kernel schedules CPU time by cycling processes through distinct states:
  Running / Runnable (R): The process is actively using the CPU right now, or is waiting in the queue to get CPU time.
  Interruptible Sleep (S): The process is waiting for an event (like a keystroke, network packet, or timer) to wake it up.
  Uninterruptible Sleep (D): The process is waiting on critical hardware IO (like a slow hard drive read) and cannot be interrupted by signals.
  Stopped (T): The process has been paused manually (e.g., pressing Ctrl + Z).
  Zombie (Z): The process is terminated but waiting for its parent to acknowledge it


  Q3.
  systemd is the central management engine of modern Linux. Once the Linux kernel boots, it hands absolute control over to systemd as Process ID 1 (PID 1). From that moment until power-down, systemd acts as the parent of all other processes, orchestrating everything from background services to system logging
