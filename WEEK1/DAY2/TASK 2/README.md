# System Monitoring Warfare: Complete DevOps Mastery Guide

## Table of Contents
1. [Foundation Concepts](#foundation-concepts)
2. [Essential Monitoring Tools Installation](#essential-monitoring-tools-installation)
3. [Real-Time Interactive Monitoring](#real-time-interactive-monitoring)
4. [Performance Analysis Tools](#performance-analysis-tools)
5. [Process-Level Monitoring](#process-level-monitoring)
6. [Bottleneck Identification Framework](#bottleneck-identification-framework)
7. [Real-World Scenarios](#real-world-scenarios)
8. [Keyboard Shortcuts & Muscle Memory](#keyboard-shortcuts-muscle-memory)
9. [Quick Reference Cheat Sheets](#quick-reference-cheat-sheets)

---

## Foundation Concepts

### Why System Monitoring Matters in DevOps

```
┌─────────────────────────────────────────────────────────┐
│  CRITICAL DEVOPS MONITORING SCENARIOS:                  │
├─────────────────────────────────────────────────────────┤
│  ✓ Production incident response (site down!)           │
│  ✓ Performance degradation investigation                │
│  ✓ Capacity planning and resource optimization         │
│  ✓ Cost reduction (right-sizing instances)             │
│  ✓ Security breach detection (unusual activity)        │
│  ✓ Application deployment validation                   │
│  ✓ Database query optimization                         │
│  ✓ Network troubleshooting                             │
└─────────────────────────────────────────────────────────┘
```

### The Four Pillars of System Performance

```
┌──────────────────────────────────────────────────────┐
│         SYSTEM RESOURCE HIERARCHY                     │
├──────────────────────────────────────────────────────┤
│                                                       │
│  1. CPU (Compute)                                    │
│     ├─ User time (application code)                 │
│     ├─ System time (kernel operations)              │
│     ├─ I/O wait (waiting for disk/network)          │
│     └─ Idle (available capacity)                    │
│                                                       │
│  2. MEMORY (RAM)                                     │
│     ├─ Used (active applications)                   │
│     ├─ Cached (filesystem cache)                    │
│     ├─ Buffers (I/O buffers)                        │
│     └─ Swap (disk-based virtual memory)             │
│                                                       │
│  3. DISK I/O (Storage)                               │
│     ├─ Read throughput (MB/s)                       │
│     ├─ Write throughput (MB/s)                      │
│     ├─ IOPS (operations per second)                 │
│     └─ Latency (response time)                      │
│                                                       │
│  4. NETWORK (Bandwidth)                              │
│     ├─ RX (receive/download)                        │
│     ├─ TX (transmit/upload)                         │
│     ├─ Packet loss                                  │
│     └─ Latency (ping time)                          │
└──────────────────────────────────────────────────────┘
```

### Understanding System Load

```bash
# Load Average Explained
uptime
# Output: 10:30:01 up 5 days, 2:15, 3 users, load average: 1.50, 2.00, 1.80
#                                                          └─1min 5min 15min

# What does load average mean?
# Load = Number of processes waiting for CPU or in uninterruptible state
# 
# For a 4-core CPU:
#   Load 4.00 = 100% utilized (healthy)
#   Load 8.00 = 200% utilized (overloaded, queuing)
#   Load 2.00 = 50% utilized (underutilized)
#
# Formula: Load per core = Load Average / Number of Cores
```

**Visual Load Representation:**

```
Single-Core System:
Load 1.0:  [████████████████████] 100% - Fully utilized
Load 2.0:  [████████████████████]+[████████████████████] - Double queued!

Quad-Core System:
Load 4.0:  [████][████][████][████] 100% - Perfectly balanced
Load 8.0:  [████][████][████][████]+[████][████][████][████] - Overloaded!
```

---

## Essential Monitoring Tools Installation

### Installing All Required Tools

```bash
# Update package list first
sudo apt update

# Install all monitoring tools in one command
sudo apt install -y \
    htop \
    iotop \
    nethogs \
    iftop \
    sysstat \
    strace \
    dstat \
    nmon \
    glances

# Verify installations
which htop iotop nethogs iftop vmstat iostat sar pidstat strace
```

**Why Each Tool?**

```
htop      → Interactive process viewer (better than top)
iotop     → Disk I/O by process (who's writing/reading?)
nethogs   → Network bandwidth by process
iftop     → Network bandwidth by connection
sysstat   → Contains vmstat, iostat, sar, pidstat
strace    → System call tracing (debugging)
dstat     → Versatile resource statistics
nmon      → IBM's performance monitor
glances   → All-in-one monitoring dashboard
```

### Enabling SAR Data Collection

```bash
# SAR (System Activity Reporter) needs to be enabled
# It collects historical data every 10 minutes

# Enable data collection
sudo systemctl enable sysstat
sudo systemctl start sysstat

# Edit collection interval (optional)
sudo nano /etc/cron.d/sysstat
# Change: */10 * * * * (every 10 minutes)
# To:     */5 * * * *  (every 5 minutes)

# Verify SAR is collecting data
sar -u 1 3  # CPU usage, 1-second intervals, 3 samples

# Historical data location
ls -lh /var/log/sysstat/
```

---

## Real-Time Interactive Monitoring

### Part 1: htop - The Ultimate Process Manager

**Starting htop:**

```bash
# Launch htop
htop

# Launch with specific options
htop -u username    # Show only user's processes
htop -p 1234,5678  # Monitor specific PIDs
htop --tree        # Show process tree
```

**Understanding the htop Interface:**

```
┌────────────────────────────────────────────────────────┐
│  HTOP INTERFACE BREAKDOWN                              │
└────────────────────────────────────────────────────────┘

Header Section (Top):
  CPU usage bars: [|||  25%] per core
  Memory bar:     [████ 60%] RAM usage
  Swap bar:       [     0%]  Swap usage
  
  Colors meaning:
    Green  = User processes (normal priority)
    Blue   = Low priority processes
    Red    = Kernel processes
    Yellow = IRQ time

Process List (Middle):
  PID    USER      PRI  NI  VIRT   RES  SHR S  CPU% MEM%   TIME+ Command
  1234   alice      20   0  500M  100M  50M R  25.0  2.5  0:15.00 python

  PID  = Process ID
  USER = Owner
  PRI  = Priority (lower = higher priority)
  NI   = Nice value (-20 to 19, lower = higher priority)
  VIRT = Virtual memory
  RES  = Resident memory (actual RAM used)
  SHR  = Shared memory
  S    = State (R=Running, S=Sleeping, D=Disk wait, Z=Zombie)
  CPU% = CPU usage percentage
  MEM% = Memory usage percentage
  TIME+= Total CPU time consumed
  
Footer (Bottom):
  F1-F10 function keys for actions
```

**Essential htop Keyboard Shortcuts:**

```
┌────────────────────────────────────────────────────┐
│  HTOP KEYBOARD SHORTCUTS (MUSCLE MEMORY!)          │
├────────────────────────────────────────────────────┤
│  Navigation:                                       │
│    ↑/↓         → Move selection up/down           │
│    PgUp/PgDn   → Page up/down                     │
│    Home/End    → Jump to first/last process       │
│                                                     │
│  Sorting:                                          │
│    F6 or >     → Sort by column menu              │
│    P           → Sort by CPU%                     │
│    M           → Sort by MEM%                     │
│    T           → Sort by TIME+                    │
│                                                     │
│  Filtering:                                        │
│    F4 or \     → Filter processes                 │
│    F5 or t     → Tree view toggle                 │
│    u           → Filter by user                   │
│    /           → Search                           │
│                                                     │
│  Actions:                                          │
│    F9 or k     → Kill process menu                │
│    F7/F8       → Nice - (increase)/+ (decrease)   │
│    Space       → Tag process                      │
│    U           → Untag all                        │
│    I           → Invert sort order                │
│    H           → Hide/show user threads           │
│    K           → Hide/show kernel threads         │
│                                                     │
│  Display:                                          │
│    F2 or S     → Setup menu                       │
│    F10 or q    → Quit                             │
└────────────────────────────────────────────────────┘
```

**Hands-On Exercise 1: htop Mastery**

```bash
# 1. Launch htop
htop

# 2. Sort by CPU usage
# Press: P

# 3. Find top memory consumer
# Press: M

# 4. Filter to show only your processes
# Press: u
# Select your username

# 5. Search for specific process
# Press: /
# Type: nginx

# 6. Switch to tree view
# Press: F5 or t

# 7. Kill a process (BE CAREFUL!)
# Select process with arrow keys
# Press: F9
# Choose signal (15 = TERM for graceful, 9 = KILL for force)

# 8. Customize display
# Press: F2
# Navigate to "Columns" and add/remove columns

# 9. Exit
# Press: F10 or q
```

**htop Color-Coded CPU Bars Explained:**

```
CPU Usage Colors:
  Green (Low Priority):   Nice > 0 processes
  Blue  (Normal):         User processes
  Red   (Kernel):         System/kernel processes
  Cyan  (Virtualization): Virtual CPU
  Magenta (IRQ):          Interrupt handling
  Yellow (Soft-IRQ):      Software interrupts
```

### Part 2: iotop - Disk I/O Monitor

**Starting iotop:**

```bash
# Requires root privileges
sudo iotop

# Show only processes doing I/O
sudo iotop -o

# Show accumulated I/O instead of bandwidth
sudo iotop -a

# Non-interactive mode (useful for logging)
sudo iotop -b -n 5  # 5 iterations
```

**Understanding iotop Output:**

```
┌────────────────────────────────────────────────────────┐
│  IOTOP INTERFACE                                       │
└────────────────────────────────────────────────────────┘

Header:
  Total DISK READ:  100.00 M/s | Total DISK WRITE: 50.00 M/s
  
Process List:
  TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN   IO    COMMAND
  1234  be/4 alice    10.00 M/s  5.00 M/s    0.00 %  50.00% python script.py

  TID   = Thread ID
  PRIO  = I/O priority (be = best effort, rt = real-time)
  USER  = Process owner
  DISK READ  = Current read speed
  DISK WRITE = Current write speed
  SWAPIN     = Percentage of time swapping in
  IO         = I/O percentage
  COMMAND    = Process command
```

**iotop Keyboard Shortcuts:**

```
┌────────────────────────────────────────────────────┐
│  IOTOP SHORTCUTS                                   │
├────────────────────────────────────────────────────┤
│  o    → Toggle showing only active I/O            │
│  p    → Toggle processes/threads                  │
│  a    → Toggle accumulated/current I/O            │
│  r    → Reverse sort order                        │
│  →/←  → Change sort column                        │
│  q    → Quit                                      │
└────────────────────────────────────────────────────┘
```

**Real-World iotop Scenario:**

```bash
# Scenario: Database server running slow
# Question: Is it disk-bound?

# 1. Launch iotop
sudo iotop -o

# 2. Observe output
# If you see high DISK READ/WRITE values (>50 MB/s sustained):
#   → Disk I/O bottleneck confirmed!
#   → Check which process is causing it

# 3. Identify culprit
# Look for processes with high IO% column

# Example problematic output:
# TID   PRIO  USER     DISK READ  DISK WRITE  IO     COMMAND
# 5678  be/4  mysql    150.00 M/s 200.00 M/s  99.99% mysqld
#                      └─────────────────────────── Problem!

# 4. Actions:
#   - Check MySQL query logs for slow queries
#   - Consider adding indexes
#   - Investigate if RAID array is degraded
#   - Check if disk is failing (smartctl)
```

### Part 3: nethogs - Network Monitor by Process

**Starting nethogs:**

```bash
# Monitor default interface
sudo nethogs

# Monitor specific interface
sudo nethogs eth0

# Monitor multiple interfaces
sudo nethogs eth0 wlan0

# Show KB/s instead of default
sudo nethogs -a
```

**Understanding nethogs Output:**

```
┌────────────────────────────────────────────────────────┐
│  NETHOGS INTERFACE                                     │
└────────────────────────────────────────────────────────┘

NetHogs version 0.8.6

  PID USER     PROGRAM              DEV   SENT  RECEIVED
  1234 alice   /usr/bin/firefox     eth0  50.0  200.0 KB/s
  5678 bob     sshd: bob@pts/0      eth0   2.0    1.0 KB/s
  
  TOTAL                                   52.0  201.0 KB/s

  PID      = Process ID
  USER     = Process owner
  PROGRAM  = Full path to executable
  DEV      = Network interface
  SENT     = Upload speed
  RECEIVED = Download speed
```

**nethogs Keyboard Shortcuts:**

```
┌────────────────────────────────────────────────────┐
│  NETHOGS SHORTCUTS                                 │
├────────────────────────────────────────────────────┤
│  m    → Change display units (KB/s, MB/s, etc.)   │
│  r    → Sort by received                          │
│  s    → Sort by sent                              │
│  q    → Quit                                      │
└────────────────────────────────────────────────────┘
```

**Real-World nethogs Use Case:**

```bash
# Scenario: Network bandwidth suddenly maxed out
# Question: Which process is consuming bandwidth?

# 1. Launch nethogs
sudo nethogs

# 2. Look for high SENT/RECEIVED values

# Example problematic output:
# PID  USER    PROGRAM           DEV   SENT    RECEIVED
# 9012 www-data /usr/bin/nginx   eth0  0.5     950.0 MB/s
#                                       └─────────────── Wow!

# 3. Identify issue
#   - Nginx serving large files? (Check access logs)
#   - DDoS attack? (Check IP addresses in logs)
#   - Legitimate traffic spike? (Check analytics)

# 4. Quick mitigation
#   - Rate limit in nginx config
#   - Block suspicious IPs with iptables
#   - Scale horizontally (add more servers)
```

### Part 4: iftop - Network Traffic by Connection

**Starting iftop:**

```bash
# Monitor default interface
sudo iftop

# Monitor specific interface
sudo iftop -i eth0

# Show ports
sudo iftop -P

# Show source/destination
sudo iftop -n  # Numeric IPs (no DNS lookup)
```

**Understanding iftop Output:**

```
┌────────────────────────────────────────────────────────┐
│  IFTOP INTERFACE                                       │
└────────────────────────────────────────────────────────┘

Top section (bandwidth graph over time):
  192.168.1.10 => 8.8.8.8        5.00Mb  10.0Mb  15.0Mb
                                 ████████████████

Connection list:
  192.168.1.10    => 8.8.8.8:443        10.0Mb  5.00Mb  2.50Mb
                  <=                     1.00Mb  0.50Mb  0.25Mb
  
  Source IP       Direction   Dest IP:Port  2s avg  10s avg  40s avg

Bottom section (totals):
  TX: cum:  100MB   peak: 50.0Mb   rates:  10.0Mb  5.00Mb  2.50Mb
  RX: cum:   50MB   peak: 25.0Mb   rates:   5.00Mb  2.50Mb  1.25Mb
  TOTAL:     150MB            75.0Mb          15.0Mb  7.50Mb  3.75Mb
  
  TX    = Transmitted (sent)
  RX    = Received
  cum   = Cumulative total since iftop started
  peak  = Highest bandwidth seen
  rates = Current averages
```

**iftop Keyboard Shortcuts:**

```
┌────────────────────────────────────────────────────┐
│  IFTOP SHORTCUTS                                   │
├────────────────────────────────────────────────────┤
│  n    → Toggle DNS resolution                     │
│  s    → Toggle source host display                │
│  d    → Toggle destination host display           │
│  p    → Toggle port display                       │
│  P    → Pause display                             │
│  t    → Toggle text/bar graph interface           │
│  1/2/3 → Sort by 2s/10s/40s average               │
│  </>  → Sort by source/destination                │
│  q    → Quit                                      │
└────────────────────────────────────────────────────┘
```

**Real-World iftop Scenario:**

```bash
# Scenario: Investigating unusual network activity
# Question: Who is the server talking to?

# 1. Launch iftop with port numbers
sudo iftop -P -n

# 2. Look for suspicious connections

# Example concerning output:
# 192.168.1.50:45678 => 203.0.113.123:6667   50.0Mb
#                                             └────── IRC port (unusual!)

# 3. Investigate further
#   - Port 6667 = IRC (Internet Relay Chat)
#   - Could be botnet command & control
#   - Check process: lsof -i :45678
#   - Check listening: netstat -tulpn | grep 6667

# 4. Actions if compromised
#   - Block IP: iptables -A OUTPUT -d 203.0.113.123 -j DROP
#   - Kill process
#   - Investigate how system was compromised
#   - Scan for rootkits: rkhunter --check
```

---

## Performance Analysis Tools

### Part 1: vmstat - Virtual Memory Statistics

**Basic vmstat Usage:**

```bash
# Display statistics every 1 second, 5 times
vmstat 1 5

# Output columns explained:
# procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
#  r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
#  2  0      0 500000  50000 200000    0    0   100   200  500 1000 10  5 85  0  0
```

**Understanding vmstat Output:**

```
┌─────────────────────────────────────────────────────────┐
│  VMSTAT COLUMNS EXPLAINED                               │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  PROCS (Process States):                                │
│    r  = Running + Runnable (waiting for CPU)            │
│    b  = Uninterruptible sleep (blocked, usually I/O)    │
│                                                          │
│  MEMORY (Kilobytes):                                    │
│    swpd = Virtual memory used                           │
│    free = Idle memory                                   │
│    buff = Buffer cache                                  │
│    cache = Page cache                                   │
│                                                          │
│  SWAP (KB/s):                                           │
│    si = Swapped in from disk (bad!)                     │
│    so = Swapped out to disk (bad!)                      │
│                                                          │
│  IO (Blocks/s):                                         │
│    bi = Blocks received from block device               │
│    bo = Blocks sent to block device                     │
│                                                          │
│  SYSTEM (Count/s):                                      │
│    in = Interrupts                                      │
│    cs = Context switches                                │
│                                                          │
│  CPU (Percentage):                                      │
│    us = User time                                       │
│    sy = System time (kernel)                            │
│    id = Idle                                            │
│    wa = I/O wait (CPU waiting for I/O)                  │
│    st = Stolen (virtualization overhead)                │
└─────────────────────────────────────────────────────────┘
```

**Interpreting vmstat - Bottleneck Detection:**

```bash
# Example 1: CPU-bound system
vmstat 1 5
# procs memory   cpu
#  r  b us sy wa id
# 10  0 95  3  0  2  ← High 'r' (runnable), high 'us', low 'id'
#                      BOTTLENECK: CPU (need more CPU cores)

# Example 2: I/O-bound system
vmstat 1 5
# procs memory   cpu
#  r  b us sy wa id
#  2  5  5  2 80 13  ← High 'b' (blocked), high 'wa', low 'us'
#                      BOTTLENECK: Disk I/O (slow disk or heavy I/O)

# Example 3: Memory pressure
vmstat 1 5
# swap    memory
# si   so swpd
# 1000 2000 500000 ← Non-zero si/so (swapping!)
#                    BOTTLENECK: Memory (add more RAM)

# Example 4: Healthy system
vmstat 1 5
# procs memory   cpu
#  r  b us sy wa id
#  1  0 20  5  0 75  ← Low 'r', no blocking, high idle
#                      HEALTHY: No bottleneck
```

**Advanced vmstat Options:**

```bash
# Display in megabytes
vmstat -S M 1 5

# Show disk statistics
vmstat -d

# Show detailed memory statistics
vmstat -s

# Show slab cache info (kernel memory)
vmstat -m
```

### Part 2: iostat - I/O Statistics

**Basic iostat Usage:**

```bash
# CPU and disk stats
iostat

# Update every 2 seconds
iostat 2

# Extended disk statistics
iostat -x 2 5

# Show stats for specific device
iostat -x sda 2
```

**Understanding iostat Output:**

```bash
iostat -x 2
# Output:
Device   r/s   w/s   rkB/s   wkB/s  avgqu-sz  await  %util
sda     10.0  20.0  1000.0  2000.0      5.0   100.0   95.0
```

```
┌─────────────────────────────────────────────────────────┐
│  IOSTAT EXTENDED COLUMNS EXPLAINED                      │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  r/s      = Reads per second                            │
│  w/s      = Writes per second                           │
│  rkB/s    = Kilobytes read per second                   │
│  wkB/s    = Kilobytes written per second                │
│  avgrq-sz = Average request size (sectors)              │
│  avgqu-sz = Average queue length                        │
│  await    = Average wait time (ms)                      │
│  svctm    = Average service time (ms) [DEPRECATED]      │
│  %util    = Utilization percentage                      │
│                                                          │
│  KEY METRICS FOR BOTTLENECK DETECTION:                  │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│  %util > 80%     → Disk is busy (potential bottleneck) │
│  await > 20ms    → High latency (slow disk)            │
│  avgqu-sz > 1    → Requests queuing (overloaded)       │
└─────────────────────────────────────────────────────────┘
```

**Interpreting iostat:**

```bash
# Example 1: Healthy disk
iostat -x sda 1 1
# Device  r/s  w/s  await  %util
# sda     5.0  3.0   2.0    25.0  ← Low utilization, low latency
#                                  HEALTHY

# Example 2: Saturated disk
iostat -x sda 1 1
# Device  r/s  w/s  await  %util
# sda    100 200    150     99.9  ← High utilization, high latency!
#                                  BOTTLENECK: Disk saturated

# Example 3: Sequential I/O (good)
iostat -x sda 1 1
# Device  avgrq-sz
# sda        512    ← Large request size = sequential I/O
#                     GOOD: Efficient disk access

# Example 4: Random I/O (bad)
iostat -x sda 1 1
# Device  avgrq-sz
# sda          8    ← Small request size = random I/O
#                     BAD: Inefficient (many seeks)
```

### Part 3: sar - System Activity Reporter

**Basic sar Usage:**

```bash
# CPU usage (all CPUs)
sar -u 1 5

# CPU usage per core
sar -P ALL 1 5

# Memory usage
sar -r 1 5

# Swap usage
sar -S 1 5

# I/O statistics
sar -b 1 5

# Network statistics
sar -n DEV 1 5

# View historical data (from yesterday)
sar -u -f /var/log/sysstat/sa$(date -d yesterday +%d)
```

**Understanding sar CPU Output:**

```bash
sar -u 1 3
# Output:
# 10:30:01 AM  %user  %nice  %system  %iowait  %steal  %idle
# 10:30:02 AM   25.0    0.0      5.0      2.0     0.0   68.0
```

```
┌─────────────────────────────────────────────────────────┐
│  SAR CPU COLUMNS EXPLAINED                              │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  %user    = User-space processes (your applications)    │
│  %nice    = Low-priority processes (nice > 0)           │
│  %system  = Kernel operations                           │
│  %iowait  = Waiting for I/O (disk/network)              │
│  %steal   = Stolen by hypervisor (VM overhead)          │
│  %idle    = CPU doing nothing                           │
│                                                          │
│  BOTTLENECK DETECTION:                                  │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│  %user > 80%      → Application CPU-bound              │
│  %system > 30%    → Kernel bottleneck (syscalls, etc.) │
│  %iowait > 20%    → I/O bottleneck                     │
│  %steal > 10%     → VM resource contention             │
└─────────────────────────────────────────────────────────┘
```

**sar Memory Analysis:**

```bash
sar -r 1 3
# Output:
# 10:30:01  kbmemfree  kbmemused  %memused  kbcached
# 10:30:02     500000    3500000      87.5    200000
```

**Interpreting sar Memory:**

```bash
# Memory pressure indicators:
# 1. %memused > 90% AND kbcached < 10% of total
#    → Real memory shortage

# 2. kbswpused increasing over time
#    → System swapping (memory pressure)

# 3. pgpgin/s and pgpgout/s high (sar -B)
#    → Paging activity (bad for performance)
```

**sar Network Analysis:**

```bash
sar -n DEV 1 3
# Output:
# 10:30:01  IFACE   rxpck/s   txpck/s   rxkB/s   txkB/s
# 10:30:02  eth0     1000.0    800.0    500.0    400.0
```

**Historical Analysis with sar:**

```bash
# View CPU usage at specific time yesterday
sar -u -s 10:00:00 -e 11:00:00 -f /var/log/sysstat/sa$(date -d yesterday +%d)

# Generate daily report
sar -A > /tmp/system_report_$(date +%Y%m%d).txt
```

---

## Process-Level Monitoring

### Part 1: pidstat - Per-Process Statistics

**Basic pidstat Usage:**

```bash
# CPU usage by process
pidstat 1 5

# Memory usage by process
pidstat -r 1 5

# Disk I/O by process
pidstat -d 1 5

# Context switches by process
pidstat -w 1 5

# Threads statistics
pidstat -t 1 5

# Monitor specific process
pidstat -p 1234 1 5
```

**Understanding pidstat Output:**

```bash
pidstat 1 3
# Output:
# 10:30:01  UID   PID  %usr  %system  %CPU  CPU  Command
# 10:30:02  1000  5678  25.0     5.0  30.0    2  python
```

```
┌─────────────────────────────────────────────────────────┐
│  PIDSTAT COLUMNS EXPLAINED                              │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  CPU USAGE (pidstat):                                   │
│    %usr     = User-space CPU                            │
│    %system  = Kernel-space CPU                          │
│    %CPU     = Total CPU (usr + system)                  │
│    CPU      = CPU core number                           │
│                                                          │
│  MEMORY USAGE (pidstat -r):                             │
│    minflt/s = Minor faults (page in RAM)                │
│    majflt/s = Major faults (page from disk) - BAD!      │
│    VSZ      = Virtual memory size                       │
│    RSS      = Resident Set Size (actual RAM)            │
│    %MEM     = Memory percentage                         │
│                                                          │
│  DISK I/O (pidstat -d):                                 │
│    kB_rd/s  = Kilobytes read per second                 │
│    kB_wr/s  = Kilobytes written per second              │
│    kB_ccwr/s = Cancelled writes                         │
│                                                          │
│  CONTEXT SWITCHES (pidstat -w):                         │
│    cswch/s  = Voluntary context switches               │
│    nvcswch/s = Involuntary switches (preemption)       │
└─────────────────────────────────────────────────────────┘
```

**Real-World pidstat Use Case:**

```bash
# Scenario: Identify which process is causing high load

# 1. Find top CPU consumers
pidstat 1 5 | sort -k 7 -rn | head -10

# 2. Check if process is I/O bound
pidstat -d -p 5678 1 5
# If kB_rd/s or kB_wr/s is high → I/O bound

# 3. Check memory behavior
pidstat -r -p 5678 1 5
# If majflt/s > 0 → Memory pressure (swapping)

# 4. Check context switching
pidstat -w -p 5678 1 5
# If nvcswch/s is very high → CPU contention
```

### Part 2: strace - System Call Tracer

**Basic strace Usage:**

```bash
# Trace a running process
sudo strace -p 1234

# Trace a new process
strace ls -la

# Count system calls
strace -c ls -la

# Show timestamps
strace -t ls -la

# Show time spent in each syscall
strace -T ls -la

# Follow forks
strace -f ./script.sh

# Filter specific syscalls
strace -e trace=open,read,write ls -la
```

**Understanding strace Output:**

```bash
strace ls
# Output:
# execve("/bin/ls", ["ls"], [/* 20 vars */]) = 0
# open("/etc/ld.so.cache", O_RDONLY) = 3
# read(3, "\177ELF...", 832) = 832
# close(3) = 0
```

```
┌─────────────────────────────────────────────────────────┐
│  STRACE OUTPUT EXPLAINED                                │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  syscall(arguments) = return_value                      │
│                                                          │
│  Example:                                               │
│    open("/etc/passwd", O_RDONLY) = 3                   │
│    └──┬─┘ └─────┬─────┘ └───┬───┘   └┬┘               │
│    Syscall   File path   Flags    File descriptor      │
│                                                          │
│  Common syscalls:                                       │
│    open/openat  = Open file                            │
│    read/write   = Read/write data                      │
│    stat/lstat   = Get file info                        │
│    mmap         = Map memory                           │
│    socket/connect = Network operations                 │
│    futex        = Locking (multithreading)             │
│                                                          │
│  Return values:                                         │
│    >= 0  = Success (often file descriptor)             │
│    -1    = Error (check errno)                         │
└─────────────────────────────────────────────────────────┘
```

**Real-World strace Scenarios:**

```bash
# Scenario 1: Find what files a program opens
strace -e trace=open,openat ./myapp 2>&1 | grep -v ENOENT

# Scenario 2: Why is this process slow?
strace -T -p 5678
# Look for syscalls taking > 1 second

# Scenario 3: Network debugging
strace -e trace=network -p 5678
# Shows all socket operations

# Scenario 4: Count syscalls to find bottleneck
strace -c ./myapp
# Output shows syscall frequency and time

# Example output:
# % time     seconds  usecs/call     calls    errors syscall
# ------ ----------- ----------- --------- --------- --------
#  99.50    0.500000       50000        10           read
#   0.50    0.002500         250        10           write
# 
# Interpretation: 99.5% of time spent in read() → I/O bound!
```

**strace Performance Considerations:**

```bash
# WARNING: strace has SIGNIFICANT overhead!
# Can slow process by 10-100x

# Use -c for minimal overhead (just counts)
strace -c -p 1234

# For production, consider alternatives:
#   - perf (lower overhead)
#   - BPF tools (minimal overhead)
#   - Application-level tracing
```

---

## Bottleneck Identification Framework

### The 30-Second Methodology

```
┌──────────────────────────────────────────────────────┐
│  30-SECOND BOTTLENECK DETECTION WORKFLOW             │
├──────────────────────────────────────────────────────┤
│                                                       │
│  STEP 1 (5 seconds): Quick Overview                  │
│    $ uptime                                          │
│    → Check load average                              │
│    → Load > cores*2 = likely bottleneck              │
│                                                       │
│  STEP 2 (10 seconds): Resource Check                 │
│    $ htop (or top)                                   │
│    → CPU bars maxed out? → CPU bottleneck            │
│    → Memory/Swap full? → Memory bottleneck           │
│    → Processes in 'D' state? → I/O bottleneck        │
│                                                       │
│  STEP 3 (10 seconds): Quick Stats                    │
│    $ vmstat 1 3                                      │
│    → 'r' > cores*2? → CPU bottleneck                 │
│    → 'b' > 0 consistently? → I/O bottleneck          │
│    → 'si/so' > 0? → Memory bottleneck (swapping)     │
│    → 'wa' > 20%? → I/O bottleneck                    │
│                                                       │
│  STEP 4 (5 seconds): Identify Process                │
│    → If CPU: Note top CPU process from htop          │
│    → If I/O: $ sudo iotop -o                         │
│    → If Network: $ sudo nethogs                      │
│    → If Memory: Note top MEM process from htop       │
└──────────────────────────────────────────────────────┘
```

### Bottleneck Decision Tree

```
START: System slow?
│
├─ Check: uptime (load average)
│  │
│  ├─ Load > cores*4 → Severe CPU or I/O bottleneck
│  ├─ Load = cores*1-2 → Healthy
│  └─ Load < cores → Underutilized or I/O bound
│
├─ Check: htop
│  │
│  ├─ CPU bars all red/maxed
│  │  └─→ CPU BOTTLENECK
│  │      Actions: Scale vertically (more cores)
│  │               Optimize code
│  │               Add caching
│  │
│  ├─ Memory bar maxed + swap used
│  │  └─→ MEMORY BOTTLENECK
│  │      Actions: Scale vertically (more RAM)
│  │               Fix memory leaks
│  │               Optimize queries/caching
│  │
│  └─ Many processes in 'D' state
│     └─→ I/O BOTTLENECK
│         Check: sudo iotop
│         Actions: Faster disks (SSD)
│                  RAID optimization
│                  Reduce I/O operations
│
└─ Check: vmstat 1 5
   │
   ├─ 'wa' (I/O wait) > 20%
   │  └─→ I/O BOTTLENECK
   │      Deep dive: iostat -x 1
   │
   ├─ 'si/so' (swap in/out) > 0
   │  └─→ MEMORY BOTTLENECK (swapping)
   │      Deep dive: sar -r 1
   │
   └─ 'r' (runnable) > cores*2
      └─→ CPU BOTTLENECK
          Deep dive: pidstat 1
```

### Hands-On Exercise: Bottleneck Detection

**Create Test Scenarios:**

```bash
# Scenario 1: CPU bottleneck simulation
# Install stress tool
sudo apt install stress

# Generate CPU load
stress --cpu 8 --timeout 60s &

# Now detect:
# 1. Run: uptime (load will be high)
# 2. Run: htop (CPU bars maxed)
# 3. Run: vmstat 1 5 (high 'us', low 'id')
# Conclusion: CPU bottleneck ✓

# Scenario 2: Memory pressure simulation
stress --vm 2 --vm-bytes 1G --timeout 60s &

# Now detect:
# 1. Run: htop (memory bar full)
# 2. Run: vmstat 1 5 (non-zero 'si/so')
# 3. Run: sar -r 1 5 (high %memused)
# Conclusion: Memory bottleneck ✓

# Scenario 3: I/O bottleneck simulation
stress --io 4 --timeout 60s &

# Now detect:
# 1. Run: htop (processes in 'D' state)
# 2. Run: sudo iotop (high I/O activity)
# 3. Run: vmstat 1 5 (high 'wa')
# Conclusion: I/O bottleneck ✓

# Clean up
killall stress
```

**Practice Detection (Timed Exercise):**

```bash
# Timer: 30 seconds to identify bottleneck

# Run mystery workload:
./mystery_load.sh &  # (You create various loads)

# Your turn:
# 1. Check uptime
# 2. Quick htop scan
# 3. vmstat 1 3
# 4. Identify bottleneck type
# 5. Name the process causing it

# Expected: <30 seconds from start to answer
```

---

## Real-World Scenarios

### Scenario 1: Web Server Under Load

```bash
# Symptom: Website slow, users complaining
# Time: Production incident, need fast diagnosis

# Step 1: Initial check (5 sec)
uptime
# load average: 25.0, 20.0, 15.0
# Server has 8 cores → Load per core = 3.1
# Indication: Overloaded (should be < 2.0)

# Step 2: Quick resource check (10 sec)
htop
# Observation:
#   - CPU: 25% user, 5% system, 70% iowait
#   - Memory: 60% used
#   - Many processes in 'D' state
# Indication: I/O bottleneck!

# Step 3: Confirm with vmstat (10 sec)
vmstat 1 3
# procs  cpu
#  r  b  wa  id
#  5 15  70   5
#     └──────── Many blocked processes
#        └───── High I/O wait
# Confirmed: I/O bottleneck

# Step 4: Find culprit (10 sec)
sudo iotop -o
# TID  USER  DISK READ  DISK WRITE  COMMAND
# 5678 mysql  500 MB/s  200 MB/s   mysqld
# Found: MySQL doing heavy I/O

# Step 5: Investigate (20 sec)
# Check MySQL slow query log
sudo tail -50 /var/log/mysql/slow.log

# Step 6: Quick mitigation (30 sec)
# Option 1: Kill long-running query
mysql -e "SHOW PROCESSLIST;"
# Find problematic query
mysql -e "KILL 12345;"

# Option 2: Add index if missing
# (Requires application knowledge)

# Option 3: Restart MySQL (if safe)
sudo systemctl restart mysql

# Total time: ~90 seconds
# Result: Identified root cause, applied quick fix
```

### Scenario 2: Application Memory Leak

```bash
# Symptom: Application getting slower over days
# Time: Non-urgent, systematic investigation

# Step 1: Check historical memory usage
sar -r | tail -50
# Observation: %memused gradually increasing
# Day 1: 40%
# Day 2: 55%
# Day 3: 70%
# Day 4: 85%
# Pattern: Memory leak suspected

# Step 2: Find the leaking process
htop
# Sort by MEM% (press M)
# Top process: python app.py (PID 5678) - 45% memory

# Step 3: Monitor process over time
pidstat -r -p 5678 1 60
# Observation: RSS (resident memory) growing continuously
# Time    RSS
# 10:00  500 MB
# 10:01  520 MB
# 10:02  540 MB
# Pattern: 20 MB/minute growth
# Confirmed: Memory leak in PID 5678

# Step 4: Detailed investigation
# Check process maps
cat /proc/5678/smaps | grep -A 1 heap
# Heap size growing

# Step 5: Application-level debugging
# (Requires app knowledge)
# - Check for unclosed file handles: lsof -p 5678
# - Profile with memory profiler
# - Review recent code changes

# Step 6: Immediate mitigation
# Restart application with cron (temporary fix)
echo "0 2 * * * systemctl restart myapp" | sudo crontab -

# Long-term fix: Code review and fix the leak
```

### Scenario 3: Network Bandwidth Exhaustion

```bash
# Symptom: Network slow, API timeouts
# Time: Production issue, medium urgency

# Step 1: Check network utilization
sudo iftop -n
# Observation:
# 192.168.1.10 => 8.8.8.8:443    950 Mb/s
# Interface: 1 Gbps link
# Utilization: 95%
# Pattern: Single connection using all bandwidth

# Step 2: Identify process
sudo nethogs
# PID  USER    PROGRAM        SENT    RECEIVED
# 9012 alice   /usr/bin/wget  950 Mb  0 Mb
# Found: wget downloading large file

# Step 3: Check what's being downloaded
ps aux | grep 9012
# wget https://example.com/bigfile.iso

# Step 4: Investigate legitimacy
# Check user activity
last alice
# Check scheduled jobs
crontab -u alice -l
# Check if part of backup script
grep -r "bigfile.iso" /etc/cron.*

# Step 5: Action based on finding
# If legitimate: Rate limit
sudo tc qdisc add dev eth0 root tbf rate 100mbit burst 32kbit latency 400ms

# If malicious: Block and investigate
kill 9012
sudo iptables -A OUTPUT -d example.com -j DROP
# Investigate how unauthorized download started

# Step 6: Monitor resolution
sudo iftop -n
# Confirm bandwidth back to normal
```

---

## Keyboard Shortcuts & Muscle Memory

### Terminal Navigation Shortcuts

```bash
┌────────────────────────────────────────────────────┐
│  TERMINAL SHORTCUTS (UNIVERSAL)                    │
├────────────────────────────────────────────────────┤
│  Ctrl + C       → Kill current command             │
│  Ctrl + Z       → Suspend current command          │
│  Ctrl + L       → Clear screen                     │
│  Ctrl + A       → Jump to start of line            │
│  Ctrl + E       → Jump to end of line              │
│  Ctrl + U       → Delete to start of line          │
│  Ctrl + K       → Delete to end of line            │
│  Ctrl + R       → Reverse search history           │
│  Ctrl + D       → Exit terminal                    │
│  !!             → Repeat last command              │
│  !$             → Last argument of previous cmd    │
│  Alt + .        → Last argument (repeatable)       │
└────────────────────────────────────────────────────┘
```

### Monitoring Commands Muscle Memory Drill

**Practice this sequence until it becomes automatic:**

```bash
# 1-Minute System Check Drill
# Practice until you can do it in <60 seconds

# Start timer
time (
  # Quick overview
  uptime
  
  # CPU check
  htop &  # Open htop
  sleep 2
  killall htop
  
  # Memory check
  free -h
  
  # Disk I/O
  iostat -x 1 2
  
  # Network
  ss -s
  
  # Process list
  ps aux | head -20
)

# Goal: Familiarize with flow
# After 10 practice runs, you'll do this instinctively
```

### Tool Selection Decision Flow

```
Problem → Tool Selection (muscle memory)

Slow response?
  → uptime (load)
  → htop (resources)
  → vmstat (CPU/memory/I/O)

Disk issues?
  → iostat -x (disk stats)
  → iotop (process I/O)

Network issues?
  → iftop (connections)
  → nethogs (process bandwidth)

Specific process?
  → pidstat (process stats)
  → strace (syscall trace)

Historical analysis?
  → sar (system activity)
```

---

## Quick Reference Cheat Sheets

### One-Line Bottleneck Detectors

```bash
# CPU bottleneck quick check
uptime | awk '{print "Load per core:", $(NF-2)/`nproc`}'

# Memory pressure check
free | awk 'NR==2{printf "Memory: %.2f%% used\n", $3*100/$2}'

# Disk I/O check
iostat -x 1 2 | awk '/^[sv]d/{if($NF>80)print $1 " is " $NF "% busy - BOTTLENECK!"}'

# Network saturation check
ifstat 1 1 | awk 'NR==3{if($1>900000)print "RX saturated!"; if($2>900000)print "TX saturated!"}'

# Swap usage check
vmstat 1 3 | awk 'NR>2{if($7>0||$8>0)print "SWAPPING DETECTED - memory pressure!"}'
```

### Essential Command Combinations

```bash
# Top 10 CPU processes
ps aux --sort=-%cpu | head -11

# Top 10 memory processes
ps aux --sort=-%mem | head -11

# Monitor specific process
watch -n 1 'ps aux | grep -E "PID|5678"'

# Network connections by state
ss -s

# Open files by process
lsof -p 5678

# Process tree
pstree -p 5678

# CPU usage by all cores
mpstat -P ALL 1 5

# Disk space usage
df -h

# Directory sizes
du -sh /*

# Active network connections
netstat -tunapl | grep ESTABLISHED
```

### Monitoring Automation Scripts

```bash
# Create comprehensive system report
cat > /usr/local/bin/sysreport << 'EOF'
#!/bin/bash
echo "=== System Report $(date) ==="
echo ""
echo "--- Load Average ---"
uptime
echo ""
echo "--- CPU Usage ---"
mpstat 1 1
echo ""
echo "--- Memory Usage ---"
free -h
echo ""
echo "--- Disk Usage ---"
df -h
echo ""
echo "--- Top Processes by CPU ---"
ps aux --sort=-%cpu | head -6
echo ""
echo "--- Top Processes by Memory ---"
ps aux --sort=-%mem | head -6
echo ""
echo "--- Disk I/O ---"
iostat -x 1 2
echo ""
echo "--- Network Connections ---"
ss -s
EOF

chmod +x /usr/local/bin/sysreport

# Run it
sysreport > /tmp/system_report_$(date +%Y%m%d_%H%M%S).txt
```

### Monitoring Dashboard Script

```bash
cat > /usr/local/bin/dashboard << 'EOF'
#!/bin/bash
while true; do
  clear
  echo "=== System Dashboard $(date) ==="
  echo ""
  echo "Load: $(uptime | awk -F'load average:' '{print $2}')"
  echo ""
  echo "CPU:"
  mpstat 1 1 | tail -1
  echo ""
  echo "Memory:"
  free -h | grep Mem
  echo ""
  echo "Disk I/O:"
  iostat -x 1 1 | grep -E '^Device|sda'
  echo ""
  echo "Network:"
  ifstat 1 1 | tail -1
  echo ""
  echo "Top 5 CPU:"
  ps aux --sort=-%cpu | head -6 | tail -5
  echo ""
  echo "Press Ctrl+C to exit"
  sleep 5
done
EOF

chmod +x /usr/local/bin/dashboard
```

---

## Final Mastery Test

### 30-Second Bottleneck Challenge

```bash
# Instructor runs this (or you can create scenarios):

# Scenario 1: CPU-bound
stress --cpu $(nproc) --timeout 300s &

# Scenario 2: Memory-bound
stress --vm 4 --vm-bytes 1G --timeout 300s &

# Scenario 3: I/O-bound
dd if=/dev/zero of=/tmp/test bs=1M count=10000 &

# Your task: Identify which bottleneck in <30 seconds
# Tools allowed: uptime, htop, vmstat
# Answer format: "CPU", "Memory", or "I/O"
```

### Complete System Analysis Exercise

```bash
# Perform a complete analysis and document:

# 1. Current system state
uptime
free -h
df -h

# 2. Resource utilization
htop (screenshot)
vmstat 1 10 > vmstat.log
iostat -x 1 10 > iostat.log

# 3. Top resource consumers
ps aux --sort=-%cpu | head -20 > top_cpu.txt
ps aux --sort=-%mem | head -20 > top_mem.txt

# 4. Network analysis
sudo nethogs -t > nethogs.log &
sleep 30
killall nethogs

# 5. Bottleneck identification
# Document:
#   - Bottleneck type (CPU/Memory/I/O/Network)
#   - Evidence (metric values)
#   - Process responsible (PID + name)
#   - Recommended action

# Expected completion time: 5 minutes
```

---

## Conclusion

You now have mastery of:
- ✅ Real-time monitoring tools (htop, iotop, nethogs, iftop)
- ✅ Performance analysis (vmstat, iostat, sar)
- ✅ Process-level monitoring (pidstat, strace)
- ✅ 30-second bottleneck identification
- ✅ Real-world DevOps scenarios
- ✅ Keyboard shortcuts for speed
- ✅ Muscle-memory workflows

**Daily Practice Routine:**
1. Morning: Run `sysreport` to baseline your system
2. Practice: 5-minute drill of all tools
3. Simulate: Create bottlenecks and detect them
4. Review: Analyze historical data with `sar`

**Remember:** In production incidents, speed and accuracy save money. Master these tools, and you'll diagnose issues before others finish logging in! 🚀