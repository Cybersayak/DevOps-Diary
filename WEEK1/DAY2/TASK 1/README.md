# Process Control Mastery: Complete DevOps Guide

## Table of Contents
1. [Foundation Concepts](#foundation-concepts)
2. [Process Lifecycle & States](#process-lifecycle--states)
3. [Background Processes (nohup, &)](#background-processes)
4. [Terminal Multiplexers (screen, tmux)](#terminal-multiplexers)
5. [Job Control (jobs, bg, fg)](#job-control)
6. [Process Termination (kill, killall, pkill)](#process-termination)
7. [Signal Management](#signal-management)
8. [Managing 20+ Processes](#managing-20-processes)
9. [Real-World DevOps Scenarios](#real-world-scenarios)

---

## Foundation Concepts

### What is a Process?

```
┌─────────────────────────────────────────────────────┐
│  PROCESS = Running Instance of a Program            │
├─────────────────────────────────────────────────────┤
│                                                      │
│  Every process has:                                 │
│  ├─ PID (Process ID) - Unique identifier           │
│  ├─ PPID (Parent PID) - Who spawned it             │
│  ├─ User/Owner - Who runs it                       │
│  ├─ Priority - CPU scheduling importance           │
│  ├─ State - Running, Sleeping, Stopped, Zombie     │
│  └─ Resources - Memory, CPU, file descriptors      │
│                                                      │
│  Process Tree Example:                              │
│    systemd (PID 1)                                  │
│    └─ bash (PID 1234)                               │
│       ├─ python script.py (PID 5678)                │
│       └─ background_job (PID 9012)                  │
└─────────────────────────────────────────────────────┘
```

### Process vs Job vs Daemon

```
┌──────────────────────────────────────────────────────────┐
│  TERMINOLOGY CLARIFICATION:                              │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  FOREGROUND PROCESS                                      │
│  ├─ Attached to terminal                                │
│  ├─ Receives keyboard input                             │
│  ├─ Blocks terminal until complete                      │
│  └─ Example: nano file.txt                              │
│                                                           │
│  BACKGROUND PROCESS                                      │
│  ├─ Runs in background                                  │
│  ├─ Terminal remains usable                             │
│  ├─ Dies if terminal closes (usually)                   │
│  └─ Example: python script.py &                         │
│                                                           │
│  JOB                                                     │
│  ├─ Shell's name for process                            │
│  ├─ Can switch between fg/bg                            │
│  ├─ Identified by job number                            │
│  └─ Example: [1]+ Running                               │
│                                                           │
│  DAEMON                                                  │
│  ├─ Background process not tied to terminal             │
│  ├─ Survives terminal closure                           │
│  ├─ Usually started at boot                             │
│  └─ Example: nginx, sshd, systemd                       │
└──────────────────────────────────────────────────────────┘
```

### Why Process Control Matters in DevOps

```
Real-World Scenarios:
  ✓ Running long database migrations without terminal
  ✓ Multiple deployment scripts running simultaneously
  ✓ Log monitoring sessions that survive SSH disconnects
  ✓ Parallel test suites running in background
  ✓ Graceful service restarts with proper signals
  ✓ Emergency process termination during incidents
  ✓ Resource cleanup on crashed processes
  ✓ Session persistence during network issues
```

---

## Process Lifecycle & States

### Process States Explained

```bash
# View process states
ps aux

# Output columns:
# USER  PID  %CPU %MEM   VSZ  RSS  TTY  STAT  START  TIME  COMMAND
# alice 1234  0.5  1.2  1000 5000  pts/0  S+   10:30  0:05  python

# STAT column breakdown:
R   # Running (actively using CPU)
S   # Sleeping (waiting for event)
D   # Uninterruptible sleep (usually I/O)
T   # Stopped (Ctrl+Z)
Z   # Zombie (dead but not reaped by parent)

# Additional modifiers:
+   # Foreground process group
<   # High priority
N   # Low priority
L   # Has pages locked in memory
s   # Session leader
l   # Multi-threaded
```

**Visual Process State Diagram:**

```
┌────────────────────────────────────────────────────┐
│         PROCESS STATE TRANSITIONS                   │
└────────────────────────────────────────────────────┘

    [Created]
       ↓
    [Ready] ←──────────┐
       ↓               │
   [Running] ──────────┤
       ↓               │
    [Waiting]          │
       ↓               │
   [Terminated] ───────┘
       ↓
    [Zombie]
       ↓
    [Reaped]

Triggers:
- fork() → Created
- Scheduler → Running
- I/O wait → Waiting
- Signal/exit → Terminated
- Parent reads status → Reaped
```

### Viewing Process Information

```bash
# Basic process listing
ps

# All processes with full details
ps aux

# Process tree (shows parent-child relationships)
pstree
# OR with PIDs
pstree -p

# Process tree for specific user
pstree alice

# Real-time process monitoring
top
# OR better alternative
htop  # (install with: sudo apt install htop)

# Find processes by name
pgrep nginx
# With full command line
pgrep -a nginx

# Detailed process info
ps -fp 1234  # Replace 1234 with actual PID

# All processes in tree format
ps auxf
```

**Understanding ps Output:**

```bash
ps aux
# Output example:
# USER   PID  %CPU %MEM    VSZ   RSS TTY   STAT START   TIME COMMAND
# alice  1234  0.5  1.2   10000 5000 pts/0 S+   10:30   0:05 python script.py

# Breakdown:
# USER    : Owner of process (alice)
# PID     : Process ID (1234)
# %CPU    : CPU usage percentage (0.5%)
# %MEM    : Memory usage percentage (1.2%)
# VSZ     : Virtual memory size in KB (10000)
# RSS     : Resident Set Size (actual RAM) in KB (5000)
# TTY     : Terminal (pts/0 = pseudo-terminal)
# STAT    : Process state (S+ = Sleeping in foreground)
# START   : When process started (10:30)
# TIME    : CPU time consumed (0:05 = 5 seconds)
# COMMAND : Command that started process
```

---

## Background Processes

### Part 1: Simple Background Execution (&)

**Basic Syntax:**

```bash
# Run command in background
command &

# Example:
sleep 100 &
# Output: [1] 12345
#         └─ job number
#              └─ PID

# Multiple background jobs
sleep 200 &
sleep 300 &
sleep 400 &

# View background jobs
jobs
# Output:
# [1]   Running    sleep 200 &
# [2]-  Running    sleep 300 &
# [3]+  Running    sleep 400 &
```

**Why Use &?**
- Frees up terminal immediately
- Can run multiple tasks simultaneously
- Simple syntax for quick tasks

**Limitations:**
- Process dies when terminal closes
- Output still goes to terminal (messy)
- No persistence across SSH disconnects

**Hands-On Exercise 1:**

```bash
# 1. Start a simple background process
echo "Starting at $(date)" > /tmp/test.log &
jobs

# 2. Redirect output to file (cleaner)
echo "Background test" > /tmp/bg.log &

# 3. Run script in background
cat > /tmp/count.sh << 'EOF'
#!/bin/bash
for i in {1..10}; do
    echo "Count: $i"
    sleep 1
done
EOF

chmod +x /tmp/count.sh
/tmp/count.sh &

# Check it's running
jobs
ps aux | grep count.sh
```

### Part 2: nohup - Persist After Terminal Closes

**Understanding nohup:**

```
nohup = "no hang up"

What it does:
  1. Ignores SIGHUP signal (terminal close signal)
  2. Redirects stdout to nohup.out
  3. Redirects stderr to nohup.out
  4. Process survives terminal closure

Perfect for:
  ✓ Long-running tasks over SSH
  ✓ Background jobs that must complete
  ✓ Tasks that shouldn't die on disconnect
```

**Basic nohup Usage:**

```bash
# Simple nohup
nohup command &

# Example:
nohup sleep 1000 &
# Output: nohup: ignoring input and appending output to 'nohup.out'
# [1] 12345

# Check output file
cat nohup.out

# Redirect to custom file
nohup command > output.log 2>&1 &
# Breakdown:
# nohup          - Make persistent
# command        - What to run
# > output.log   - Redirect stdout
# 2>&1           - Redirect stderr to stdout
# &              - Run in background

# Real example:
nohup python /opt/app/process_data.py > /var/log/app/process.log 2>&1 &
```

**Complete nohup Example:**

```bash
# Create a long-running script
cat > /tmp/long_task.sh << 'EOF'
#!/bin/bash
echo "Starting task at $(date)"
for i in {1..100}; do
    echo "Processing item $i"
    sleep 2
done
echo "Task completed at $(date)"
EOF

chmod +x /tmp/long_task.sh

# Run with nohup
nohup /tmp/long_task.sh > /tmp/task.log 2>&1 &

# Save the PID
echo $! > /tmp/task.pid
# $! is the PID of last background process

# Check it's running
ps -p $(cat /tmp/task.pid)

# Monitor progress
tail -f /tmp/task.log

# Simulate SSH disconnect
exit
# (Re-login to server)

# Verify it's still running
ps -p $(cat /tmp/task.pid)
# Still running! ✓
```

**nohup vs & Comparison:**

```bash
# Test 1: Regular background process (&)
sleep 1000 &
jobs
# Close terminal → Process dies ✗

# Test 2: nohup
nohup sleep 1000 &
# Close terminal → Process survives ✓

# Test 3: Combine for best results
nohup command > log.txt 2>&1 &
```

### Part 3: disown - Detach from Shell

```bash
# Start background process
sleep 1000 &
jobs
# [1]+ Running    sleep 1000 &

# Disown it (remove from job table)
disown %1
jobs
# (empty - no longer tracked by shell)

# Process still runs, check with ps
ps aux | grep sleep

# Alternative: disown all jobs
sleep 2000 &
sleep 3000 &
disown -a  # disown all

# Use case:
# Started job without nohup, need to leave
command &
disown
exit  # Can safely exit now
```

**Complete Workflow Example:**

```bash
# Scenario: Started a job, forgot nohup, need to leave

# 1. Start job (oops, forgot nohup!)
python /opt/scripts/migration.py &
# [1] 12345

# 2. Redirect output (too late for nohup)
# Can't redirect after start, but can use process substitution

# 3. Disown to detach
disown %1

# 4. Safe to close terminal
exit

# Alternative using Ctrl+Z:
# 1. Suspend foreground job
python /opt/scripts/migration.py
# Press Ctrl+Z
# [1]+ Stopped

# 2. Send to background
bg %1

# 3. Disown
disown %1

# 4. Exit safely
exit
```

---

## Terminal Multiplexers

### Part 1: GNU Screen

**Why Screen?**
```
Problems without screen:
  ✗ SSH disconnect = lost session
  ✗ Can't detach/reattach sessions
  ✗ Need multiple terminals
  ✗ No session sharing

Solutions with screen:
  ✓ Sessions survive disconnects
  ✓ Detach and reattach anytime
  ✓ Multiple windows in one SSH session
  ✓ Share sessions with team
```

**Installing Screen:**

```bash
# Install
sudo apt update
sudo apt install screen

# Verify
screen --version
# Output: Screen version 4.xx.xx
```

**Basic Screen Commands:**

```bash
# Start new session
screen

# Start named session (recommended)
screen -S mysession

# List active sessions
screen -ls
# Output:
# There is a screen on:
#     12345.mysession  (Attached)
# 1 Socket in /run/screen/S-alice.

# Detach from session
# Press: Ctrl+A, then D
# Output: [detached from 12345.mysession]

# Reattach to session
screen -r mysession

# Reattach to any session if only one exists
screen -r

# Force reattach (if already attached elsewhere)
screen -d -r mysession

# Kill session
screen -X -S mysession quit
```

**Screen Window Management:**

```bash
# Start screen
screen -S devops

# Create new window: Ctrl+A, then C
# Switch to next window: Ctrl+A, then N
# Switch to previous: Ctrl+A, then P
# List windows: Ctrl+A, then "

# Window with specific command
# Ctrl+A, then :
# Type: screen vim file.txt

# Kill current window: Ctrl+A, then K
# Rename window: Ctrl+A, then A
```

**Essential Screen Keyboard Shortcuts:**

```
┌────────────────────────────────────────────────────┐
│  SCREEN COMMANDS (All start with Ctrl+A)          │
├────────────────────────────────────────────────────┤
│  Ctrl+A D   → Detach session                      │
│  Ctrl+A C   → Create new window                   │
│  Ctrl+A N   → Next window                         │
│  Ctrl+A P   → Previous window                     │
│  Ctrl+A "   → List all windows                    │
│  Ctrl+A 0-9 → Switch to window number             │
│  Ctrl+A K   → Kill current window                 │
│  Ctrl+A [   → Enter copy mode (scroll back)       │
│  Ctrl+A ]   → Paste                               │
│  Ctrl+A ?   → Help                                │
│  Ctrl+A A   → Rename window                       │
│  Ctrl+A S   → Split horizontal                    │
│  Ctrl+A |   → Split vertical                      │
│  Ctrl+A Tab → Switch between splits               │
└────────────────────────────────────────────────────┘
```

**Real-World Screen Workflow:**

```bash
# Scenario: Managing production deployment

# 1. Start named session
screen -S production-deploy

# 2. Create windows for different tasks
# Window 0: Monitoring
top

# Ctrl+A C (new window)
# Window 1: Application logs
tail -f /var/log/app/application.log

# Ctrl+A C (new window)
# Window 2: Deployment script
cd /opt/deployment
./deploy.sh

# Ctrl+A C (new window)
# Window 3: Database monitoring
mysql -u root -p
SHOW PROCESSLIST;

# 3. Detach: Ctrl+A D
# 4. Go home, coffee break, etc.
# 5. Reattach later
screen -r production-deploy

# 6. Check each window
# Ctrl+A " to see list
# Navigate with Ctrl+A N/P
```

**Screen Configuration (~/.screenrc):**

```bash
# Create config file
cat > ~/.screenrc << 'EOF'
# Disable startup message
startup_message off

# Set scrollback buffer
defscrollback 10000

# Status line at bottom
hardstatus on
hardstatus alwayslastline
hardstatus string "%{.kW}%-w%{.bW}%t [%n]%{-}%+w %=%{..G} %H %{..Y} %Y-%m-%d %c"

# Automatically detach on hangup
autodetach on

# Use UTF-8
defutf8 on

# Window numbering starts at 1 (not 0)
bind c screen 1
bind 0 select 10

# Easy window switching
bind j focus down
bind k focus up
bind h focus left
bind l focus right
EOF

# Test configuration
screen -S test
# Status bar should appear at bottom
```

### Part 2: tmux (Modern Alternative)

**Why tmux?**
```
Advantages over screen:
  ✓ More active development
  ✓ Better splitting (panes)
  ✓ Easier configuration
  ✓ Built-in status bar
  ✓ Better scripting support
  ✓ Cleaner keyboard shortcuts
```

**Installing tmux:**

```bash
# Install
sudo apt update
sudo apt install tmux

# Verify
tmux -V
# Output: tmux 3.x
```

**Basic tmux Commands:**

```bash
# Start new session
tmux

# Start named session
tmux new -s mysession

# List sessions
tmux ls
# Output:
# mysession: 1 windows (created Sat Jan 15 10:30:00 2024)

# Attach to session
tmux attach -t mysession
# OR shorthand
tmux a -t mysession

# Detach from session
# Press: Ctrl+B, then D

# Kill session
tmux kill-session -t mysession

# Kill all sessions
tmux kill-server
```

**tmux Window & Pane Management:**

```bash
# Start tmux
tmux new -s devops

# Create new window: Ctrl+B, then C
# Next window: Ctrl+B, then N
# Previous window: Ctrl+B, then P
# List windows: Ctrl+B, then W

# Split horizontally: Ctrl+B, then "
# Split vertically: Ctrl+B, then %
# Navigate panes: Ctrl+B, then arrow keys
# Close pane: Ctrl+B, then X
# Toggle pane zoom: Ctrl+B, then Z

# Resize pane: Ctrl+B, then hold Ctrl, use arrow keys
```

**Essential tmux Keyboard Shortcuts:**

```
┌────────────────────────────────────────────────────┐
│  TMUX COMMANDS (All start with Ctrl+B)            │
├────────────────────────────────────────────────────┤
│  Ctrl+B D     → Detach session                    │
│  Ctrl+B C     → Create new window                 │
│  Ctrl+B N     → Next window                       │
│  Ctrl+B P     → Previous window                   │
│  Ctrl+B 0-9   → Switch to window number           │
│  Ctrl+B W     → List windows                      │
│  Ctrl+B &     → Kill window                       │
│  Ctrl+B "     → Split horizontal                  │
│  Ctrl+B %     → Split vertical                    │
│  Ctrl+B Arrow → Navigate panes                    │
│  Ctrl+B X     → Kill pane                         │
│  Ctrl+B Z     → Zoom pane                         │
│  Ctrl+B [     → Enter copy mode                   │
│  Ctrl+B ]     → Paste                             │
│  Ctrl+B ?     → Help                              │
└────────────────────────────────────────────────────┘
```

**Real-World tmux Workflow:**

```bash
# Scenario: Monitoring production systems

# 1. Create session
tmux new -s monitoring

# 2. Setup layout
# Split horizontally: Ctrl+B "
# Split right pane vertically: Navigate right, Ctrl+B %

# Layout:
# ┌─────────────┬──────────────┐
# │             │   Top Pane   │
# │  Left Pane  ├──────────────┤
# │             │ Bottom Pane  │
# └─────────────┴──────────────┘

# 3. Set up monitoring in each pane
# Left pane: htop
htop

# Navigate to top-right: Ctrl+B Arrow
# Top-right pane: Application logs
tail -f /var/log/app/app.log

# Navigate to bottom-right: Ctrl+B Arrow
# Bottom-right pane: System logs
journalctl -f

# 4. Create new window for database
# Ctrl+B C
mysql -u root -p

# 5. Detach: Ctrl+B D

# 6. Reattach anytime
tmux a -t monitoring
```

**tmux Configuration (~/.tmux.conf):**

```bash
# Create config file
cat > ~/.tmux.conf << 'EOF'
# Change prefix from Ctrl+B to Ctrl+A (like screen)
unbind C-b
set -g prefix C-a
bind C-a send-prefix

# Enable mouse support
set -g mouse on

# Start window numbering at 1
set -g base-index 1
set -g pane-base-index 1

# Renumber windows when one is closed
set -g renumber-windows on

# Increase scrollback buffer
set -g history-limit 10000

# Status bar
set -g status-bg black
set -g status-fg white
set -g status-left '#[fg=green]#H #[fg=cyan]#S'
set -g status-right '#[fg=yellow]%Y-%m-%d %H:%M'

# Highlight active window
setw -g window-status-current-style 'fg=black bg=white bold'

# Split panes with | and -
bind | split-window -h
bind - split-window -v
unbind '"'
unbind %

# Reload config
bind r source-file ~/.tmux.conf \; display "Config reloaded!"

# Vi mode for copy
setw -g mode-keys vi

# Quick pane switching
bind -n M-Left select-pane -L
bind -n M-Right select-pane -R
bind -n M-Up select-pane -U
bind -n M-Down select-pane -D
EOF

# Reload config
tmux source-file ~/.tmux.conf
```

**tmux Session Management Script:**

```bash
# Create reusable tmux session layouts
cat > ~/start-devops-session.sh << 'EOF'
#!/bin/bash
SESSION="devops"

# Check if session exists
tmux has-session -t $SESSION 2>/dev/null

if [ $? != 0 ]; then
    # Create new session
    tmux new-session -d -s $SESSION -n monitoring
    
    # Window 1: System monitoring
    tmux send-keys -t $SESSION:monitoring "htop" C-m
    tmux split-window -h -t $SESSION:monitoring
    tmux send-keys -t $SESSION:monitoring "tail -f /var/log/syslog" C-m
    
    # Window 2: Application
    tmux new-window -t $SESSION -n app
    tmux send-keys -t $SESSION:app "cd /opt/app && python app.py" C-m
    
    # Window 3: Database
    tmux new-window -t $SESSION -n database
    tmux send-keys -t $SESSION:database "mysql -u root -p" C-m
    
    # Window 4: Logs
    tmux new-window -t $SESSION -n logs
    tmux send-keys -t $SESSION:logs "journalctl -f" C-m
fi

# Attach to session
tmux attach-t $SESSION
EOF

chmod +x ~/start-devops-session.sh

# Run
~/start-devops-session.sh
```

---

## Job Control

### Understanding Jobs

```bash
# Jobs are shell-managed processes
# Each job has:
#   - Job number (e.g., [1])
#   - State (Running, Stopped, Done)
#   - Command

# View jobs
jobs
# Output:
# [1]   Running    sleep 100 &
# [2]-  Running    sleep 200 &
# [3]+  Running    sleep 300 &
#
# Legend:
# [1]  → Job number
# +    → Current job (fg/bg affects this)
# -    → Previous job
# No symbol → Other jobs
```

### Part 1: Foreground and Background

**Moving Jobs Between Foreground and Background:**

```bash
# Start background job
sleep 100 &
# [1] 12345

# Bring to foreground
fg %1
# OR just
fg
# (brings most recent job)

# While in foreground, suspend it
# Press Ctrl+Z
# [1]+ Stopped    sleep 100

# Resume in background
bg %1
# [1]+ sleep 100 &

# Resume in foreground
fg %1
```

**Job Specification Shortcuts:**

```bash
# Different ways to refer to jobs
sleep 100 &
sleep 200 &
sleep 300 &

jobs
# [1]   Running    sleep 100 &
# [2]-  Running    sleep 200 &
# [3]+  Running    sleep 300 &

# By job number
fg %1
fg %2

# Current job (marked with +)
fg %+
fg %%  # Same as above

# Previous job (marked with -)
fg %-

# By command name
fg %sleep

# By command prefix
fg %sl

# All of these work with bg, kill, etc.
bg %1
kill %2
```

**Complete Workflow Example:**

```bash
# 1. Start multiple jobs
echo "Job 1" &
sleep 100 &
python -m http.server 8000 &

# 2. View jobs
jobs -l
# Output shows PIDs:
# [1]  12345 Running    echo "Job 1" &
# [2]- 12346 Running    sleep 100 &
# [3]+ 12347 Running    python -m http.server 8000 &

# 3. Bring web server to foreground
fg %3
# Now in foreground, see output

# 4. Suspend it (Ctrl+Z)
# [3]+ Stopped    python -m http.server 8000

# 5. Resume in background
bg %3
# [3]+ python -m http.server 8000 &

# 6. Kill specific job
kill %2
jobs
# [2]- Terminated    sleep 100
```

### Part 2: Job States and Control

```bash
# Job states
jobs -l
# Running    → Currently executing
# Stopped    → Suspended (Ctrl+Z)
# Terminated → Killed/Exited
# Done       → Completed successfully

# Wait for job to complete
sleep 10 &
wait %1
# Shell waits until job finishes

# Wait for all background jobs
sleep 10 &
sleep 20 &
wait
# Waits for both

# Disown job (remove from job table)
sleep 100 &
disown %1
jobs
# Job no longer listed, but process still runs
```

---

## Process Termination

### Part 1: kill Command

**Basic kill Syntax:**

```bash
# Send TERM signal (default, graceful)
kill PID

# Send specific signal
kill -SIGNAL PID

# Examples:
kill 12345           # SIGTERM (15)
kill -9 12345        # SIGKILL (force kill)
kill -KILL 12345     # Same as above
kill -HUP 12345      # SIGHUP (reload config)
kill -TERM 12345     # Explicit SIGTERM

# Kill job instead of process
kill %1
kill -9 %2
```

**Understanding kill:**

```
kill doesn't "kill" - it sends signals!

Default signal: SIGTERM (15)
  - Polite request to terminate
  - Process can catch and cleanup
  - May be ignored by process

When SIGTERM fails: SIGKILL (9)
  - Immediate termination
  - Cannot be caught or ignored
  - No cleanup possible
  - Last resort only!
```

**Proper kill Sequence:**

```bash
# Good practice: Try gentle signals first

# 1. Send TERM (process can cleanup)
kill 12345
sleep 2

# 2. Check if still running
if ps -p 12345 > /dev/null; then
    echo "Process still running"
    
    # 3. Send TERM again, wait longer
    kill 12345
    sleep 5
    
    # 4. If still alive, use KILL
    if ps -p 12345 > /dev/null; then
        echo "Forcing termination"
        kill -9 12345
    fi
fi
```

### Part 2: killall and pkill

**killall - Kill by Process Name:**

```bash
# Kill all processes with name
killall process_name

# Examples:
killall firefox
killall python
killall -9 zombie_process

# Kill all user's processes of specific name
killall -u alice firefox

# Interactive (ask confirmation)
killall -i nginx

# Wait for processes to die
killall -w nginx
# Blocks until all nginx processes terminate

# Send specific signal
killall -HUP nginx  # Reload config
killall -USR1 nginx  # Custom signal
```

**pkill - Kill by Pattern:**

```bash
# Kill by pattern matching
pkill pattern

# Examples:
pkill python         # All python processes
pkill -f "script.py"  # Match full command line
pkill -u alice       # All alice's processes

# Kill by exact name
pkill -x nginx

# Kill oldest/newest
pkill -o python      # Oldest
pkill -n python      # Newest

# Send signal
pkill -HUP nginx
pkill -9 zombie

# Combine filters
pkill -u alice -x python  # alice's python processes
```

**pgrep - Find Processes (Non-Destructive):**

```bash
# Find processes before killing
pgrep nginx
# Output: 1234 5678

# Show full command
pgrep -a nginx
# Output: 
# 1234 nginx: master process
# 5678 nginx: worker process

# Count processes
pgrep -c nginx
# Output: 2

# Full pattern matching
pgrep -f "python.*script"

# Combine with kill
pgrep -f "old_script.py" | xargs kill
```

**Comparison Table:**

```
┌──────────────────────────────────────────────────────┐
│  COMMAND    │  TARGET      │  USE CASE              │
├──────────────────────────────────────────────────────┤
│  kill       │  PID         │  Specific process      │
│  kill %N    │  Job number  │  Shell job             │
│  killall    │  Name        │  All with same name    │
│  pkill      │  Pattern     │  Pattern match         │
│  pgrep      │  Pattern     │  Find before kill      │
└──────────────────────────────────────────────────────┘
```

### Part 3: Advanced Termination

**Killing Process Trees:**

```bash
# Kill process and all children
kill_tree() {
    local pid=$1
    local signal=${2:-TERM}
    
    # Find all child PIDs
    children=$(pgrep -P $pid)
    
    # Kill children first
    for child in $children; do
        kill_tree $child $signal
    done
    
    # Kill parent
    kill -$signal $pid 2>/dev/null
}

# Usage:
kill_tree 12345
kill_tree 12345 KILL  # Force
```

**Killing by Resource Usage:**

```bash
# Kill processes using >90% CPU
ps aux | awk '$3 > 90 {print $2}' | xargs kill

# Kill processes using >1GB memory
ps aux | awk '$6 > 1000000 {print $2}' | xargs kill

# More gentle - list first
ps aux | awk '$3 > 90 {print $2, $11}'
# Review before killing
```

**Killing Zombie Processes:**

```bash
# Identify zombies
ps aux | grep Z
# OR
ps aux | awk '$8 == "Z"'

# Zombies can't be killed directly!
# Must kill parent process

# Find parent of zombie
ps -o ppid= -p ZOMBIE_PID

# Kill parent
kill PARENT_PID

# If parent is init (PID 1), zombie will be reaped automatically
```

---

## Signal Management

### Understanding Signals

```
┌─────────────────────────────────────────────────────┐
│  UNIX SIGNALS - Inter-Process Communication         │
├─────────────────────────────────────────────────────┤
│                                                      │
│  Signals are notifications sent to processes        │
│  Process can:                                       │
│    1. Handle it (run signal handler)               │
│    2. Ignore it (if allowed)                       │
│    3. Default action (usually terminate)           │
└─────────────────────────────────────────────────────┘
```

### Common Signals Reference

```bash
# View all signals
kill -l
# Output: List of all signal numbers and names

# Most common signals:
SIGHUP   (1)   # Hang up (terminal closed)
SIGINT   (2)   # Interrupt (Ctrl+C)
SIGQUIT  (3)   # Quit (Ctrl+\)
SIGKILL  (9)   # Force kill (cannot be caught)
SIGTERM  (15)  # Terminate (polite request)
SIGSTOP  (17)  # Stop process (cannot be caught)
SIGCONT  (18)  # Continue if stopped
SIGUSR1  (10)  # User-defined signal 1
SIGUSR2  (12)  # User-defined signal 2
```

**Signal Usage Table:**

```
┌────────────────────────────────────────────────────────────┐
│  SIGNAL  │ NUM │  PURPOSE           │  CAN CATCH? │ USE   │
├────────────────────────────────────────────────────────────┤
│  SIGHUP  │  1  │ Reload config      │    Yes      │ +++   │
│  SIGINT  │  2  │ Keyboard interrupt │    Yes      │ ++    │
│  SIGQUIT │  3  │ Quit with core dump│    Yes      │ +     │
│  SIGKILL │  9  │ Force terminate    │    NO       │ +++   │
│  SIGTERM │ 15  │ Polite terminate   │    Yes      │ +++++ │
│  SIGSTOP │ 17  │ Pause process      │    NO       │ +     │
│  SIGCONT │ 18  │ Resume process     │    Yes      │ ++    │
│  SIGUSR1 │ 10  │ User-defined       │    Yes      │ ++    │
│  SIGUSR2 │ 12  │ User-defined       │    Yes      │ ++    │
└────────────────────────────────────────────────────────────┘
```

### Part 1: SIGHUP (Reload Configuration)

**Why SIGHUP?**
```
Originally: "Hang Up" - terminal disconnected
Modern use: Reload configuration without restart

Common services that handle SIGHUP:
  ✓ nginx
  ✓ Apache
  ✓ systemd services
  ✓ logging daemons
```

**Using SIGHUP:**

```bash
# Reload nginx configuration
sudo kill -HUP $(cat /var/run/nginx.pid)
# OR
sudo killall -HUP nginx
# OR
sudo nginx -s reload

# Reload rsyslog
sudo killall -HUP rsyslogd

# Test with custom script
cat > /tmp/sighup_test.sh << 'EOF'
#!/bin/bash

CONFIG_FILE="/tmp/app.conf"

reload_config() {
    echo "$(date): Reloading configuration"
    # Reload logic here
    cat $CONFIG_FILE
}

# Trap SIGHUP
trap reload_config SIGHUP

# Initial config load
echo "Initial config: value=10" > $CONFIG_FILE
reload_config

# Main loop
while true; do
    echo "$(date): Running..."
    sleep 5
done
EOF

chmod +x /tmp/sighup_test.sh
/tmp/sighup_test.sh &
PID=$!

# Change config
echo "Updated config: value=20" > /tmp/app.conf

# Send SIGHUP to reload
kill -HUP $PID
# Check output - should show "Reloading configuration"
```

### Part 2: SIGTERM vs SIGKILL

**SIGTERM (15) - Graceful Termination:**

```bash
# Default signal - gives process chance to cleanup
kill PID
# OR explicitly
kill -TERM PID
kill -15 PID

# Process can:
#   - Close file handles
#   - Save state
#   - Release resources
#   - Send notification
#   - Finish current operation
```

**SIGKILL (9) - Immediate Termination:**

```bash
# Nuclear option - immediate death
kill -9 PID
# OR
kill -KILL PID

# Process CANNOT:
#   - Run cleanup code
#   - Save state
#   - Ignore signal
#   - Do anything!

# Use only when:
#   ✓ Process frozen/hung
#   ✓ SIGTERM failed
#   ✓ Emergency situation
#   ✗ Normal shutdown (use SIGTERM)
```

**Proper Termination Sequence:**

```bash
# Script to gracefully terminate process
graceful_kill() {
    local pid=$1
    local timeout=${2:-10}  # Default 10 seconds
    
    echo "Sending SIGTERM to $pid"
    kill -TERM $pid
    
    # Wait for process to exit
    for i in $(seq 1 $timeout); do
        if ! ps -p $pid > /dev/null; then
            echo "Process exited gracefully"
            return 0
        fi
        sleep 1
    done
    
    # Still alive after timeout
    echo "Process didn't exit, sending SIGKILL"
    kill -KILL $pid
    return 1
}

# Usage:
graceful_kill 12345 15  # Try TERM for 15 seconds
```

### Part 3: SIGUSR1 and SIGUSR2 (Custom Signals)

**Purpose:**
```
User-defined signals for custom application behavior
No default action - application must handle them

Common uses:
  ✓ Toggle debug mode
  ✓ Trigger log rotation
  ✓ Dump statistics
  ✓ Change verbosity level
  ✓ Reload specific components
```

**Custom Signal Handler Example:**

```bash
cat > /tmp/custom_signals.sh << 'EOF'
#!/bin/bash

DEBUG=0
VERBOSE=0

# Signal handlers
toggle_debug() {
    if [ $DEBUG -eq 0 ]; then
        DEBUG=1
        echo "$(date): Debug mode ENABLED"
    else
        DEBUG=0
        echo "$(date): Debug mode DISABLED"
    fi
}

toggle_verbose() {
    if [ $VERBOSE -eq 0 ]; then
        VERBOSE=1
        echo "$(date): Verbose mode ENABLED"
    else
        VERBOSE=0
        echo "$(date): Verbose mode DISABLED"
    fi
}

dump_stats() {
    echo "$(date): Statistics Dump"
    echo "  Iterations: $COUNT"
    echo "  Debug: $DEBUG"
    echo "  Verbose: $VERBOSE"
}

# Register signal handlers
trap toggle_debug SIGUSR1
trap toggle_verbose SIGUSR2
trap dump_stats SIGTERM

COUNT=0

# Main loop
while true; do
    COUNT=$((COUNT + 1))
    
    [ $DEBUG -eq 1 ] && echo "DEBUG: Iteration $COUNT"
    [ $VERBOSE -eq 1 ] && echo "VERBOSE: Working on iteration $COUNT"
    
    sleep 2
done
EOF

chmod +x /tmp/custom_signals.sh
/tmp/custom_signals.sh &
PID=$!

# Test signals
echo "Process PID: $PID"

# Toggle debug
kill -USR1 $PID
sleep 3

# Toggle verbose
kill -USR2 $PID
sleep 3

# Dump stats
kill -TERM $PID
```

**Real-World: Nginx Signal Handling:**

```bash
# Nginx uses signals extensively

# Graceful shutdown (finish current requests)
kill -QUIT $(cat /var/run/nginx.pid)

# Fast shutdown (terminate immediately)
kill -TERM $(cat /var/run/nginx.pid)

# Reload configuration
kill -HUP $(cat /var/run/nginx.pid)

# Reopen log files (for rotation)
kill -USR1 $(cat /var/run/nginx.pid)

# Graceful worker restart
kill -USR2 $(cat /var/run/nginx.pid)
```

---

## Managing 20+ Processes

### Part 1: Launch Multiple Processes

**Method 1: Simple Loop:**

```bash
# Launch 20 background processes
for i in {1..20}; do
    sleep $((RANDOM % 100 + 100)) &
    echo "Started process $i: PID $!"
done

# View all jobs
jobs

# View all processes
ps aux | grep sleep
```

**Method 2: Parallel Execution:**

```bash
# Install GNU parallel
sudo apt install parallel

# Launch 20 processes in parallel
seq 1 20 | parallel "sleep {} && echo 'Task {} done'" &

# OR with custom commands
cat > /tmp/tasks.txt << 'EOF'
python script1.py
./process_data.sh
backup_database.sh
compile_reports.py
EOF

parallel < /tmp/tasks.txt
```

**Method 3: Process Array Management:**

```bash
# Track PIDs in array
declare -a PIDS

# Launch processes
for i in {1..20}; do
    (
        echo "Process $i starting"
        sleep $((RANDOM % 60))
        echo "Process $i done"
    ) &
    PIDS[$i]=$!
    echo "Launched process $i with PID ${PIDS[$i]}"
done

# Monitor all processes
monitor_processes() {
    for i in "${!PIDS[@]}"; do
        pid=${PIDS[$i]}
        if ps -p $pid > /dev/null; then
            echo "Process $i (PID $pid): RUNNING"
        else
            echo "Process $i (PID $pid): STOPPED"
        fi
    done
}

# Call monitoring
monitor_processes

# Wait for all to complete
wait

echo "All processes completed"
```

### Part 2: Process Management Dashboard

**Complete Management Script:**

```bash
cat > /tmp/process_manager.sh << 'EOF'
#!/bin/bash

# Process tracking
declare -A PROCESSES
LOG_DIR="/tmp/process_logs"
mkdir -p "$LOG_DIR"

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

# Start a managed process
start_process() {
    local name=$1
    local command=$2
    
    if [ -n "${PROCESSES[$name]}" ]; then
        echo -e "${YELLOW}Process $name already running${NC}"
        return 1
    fi
    
    # Start process in background with log
    eval "$command" > "$LOG_DIR/$name.log" 2>&1 &
    local pid=$!
    
    PROCESSES[$name]=$pid
    echo -e "${GREEN}Started $name (PID: $pid)${NC}"
}

# Stop a process
stop_process() {
    local name=$1
    local pid=${PROCESSES[$name]}
    
    if [ -z "$pid" ]; then
        echo -e "${RED}Process $name not found${NC}"
        return 1
    fi
    
    if ps -p $pid > /dev/null; then
        kill -TERM $pid
        sleep 1
        
        if ps -p $pid > /dev/null; then
            kill -KILL $pid
            echo -e "${YELLOW}Forcefully killed $name${NC}"
        else
            echo -e "${GREEN}Stopped $name${NC}"
        fi
    else
        echo -e "${YELLOW}Process $name already stopped${NC}"
    fi
    
    unset PROCESSES[$name]
}

# List all processes
list_processes() {
    echo "Currently Managed Processes:"
    echo "----------------------------"
    
    for name in "${!PROCESSES[@]}"; do
        pid=${PROCESSES[$name]}
        
        if ps -p $pid > /dev/null; then
            cpu=$(ps -p $pid -o %cpu= | tr -d ' ')
            mem=$(ps -p $pid -o %mem= | tr -d ' ')
            echo -e "${GREEN}✓${NC} $name (PID: $pid) CPU: $cpu% MEM: $mem%"
        else
            echo -e "${RED}✗${NC} $name (PID: $pid) STOPPED"
            unset PROCESSES[$name]
        fi
    done
}

# Interactive menu
show_menu() {
    clear
    echo "===================================="
    echo "  Process Manager"
    echo "===================================="
    echo "1. Start process"
    echo "2. Stop process"
    echo "3. List processes"
    echo "4. View logs"
    echo "5. Restart process"
    echo "6. Stop all"
    echo "7. Exit"
    echo "===================================="
    read -p "Choice: " choice
    
    case $choice in
        1)
            read -p "Process name: " name
            read -p "Command: " cmd
            start_process "$name" "$cmd"
            ;;
        2)
            read -p "Process name: " name
            stop_process "$name"
            ;;
        3)
            list_processes
            ;;
        4)
            read -p "Process name: " name
            tail -20 "$LOG_DIR/$name.log"
            ;;
        5)
            read -p "Process name: " name
            local pid=${PROCESSES[$name]}
            if [ -n "$pid" ]; then
                kill -HUP $pid
                echo "Sent reload signal to $name"
            fi
            ;;
        6)
            for name in "${!PROCESSES[@]}"; do
                stop_process "$name"
            done
            ;;
        7)
            exit 0
            ;;
    esac
    
    read -p "Press Enter to continue..."
    show_menu
}

# Launch 20 test processes
launch_test_suite() {
    for i in {1..20}; do
        start_process "worker_$i" "bash -c 'while true; do echo Working $i; sleep 5; done'"
    done
}

# Main
if [ "$1" == "test" ]; then
    launch_test_suite
    list_processes
else
    show_menu
fi
EOF

chmod +x /tmp/process_manager.sh

# Run with test suite
/tmp/process_manager.sh test
```

### Part 3: Mass Process Control

**Control Multiple Processes by Pattern:**

```bash
# Send signal to all matching processes
signal_by_pattern() {
    local pattern=$1
    local signal=${2:-TERM}
    
    echo "Sending $signal to processes matching '$pattern'"
    
    pids=$(pgrep -f "$pattern")
    
    if [ -z "$pids" ]; then
        echo "No matching processes found"
        return 1
    fi
    
    for pid in $pids; do
        cmd=$(ps -p $pid -o comm=)
        echo "  Signaling $cmd (PID: $pid)"
        kill -$signal $pid
    done
}

# Usage:
signal_by_pattern "python.*worker" HUP
signal_by_pattern "sleep" TERM
```

**Restart Multiple Services:**

```bash
# Restart service pattern
restart_services() {
    local pattern=$1
    
    # Find all matching processes
    pids=$(pgrep -f "$pattern")
    
    if [ -z "$pids" ]; then
        echo "No services found matching $pattern"
        return 1
    fi
    
    # Get commands before killing
    declare -a commands
    for pid in $pids; do
        commands+=("$(ps -p $pid -o args=)")
    done
    
    # Stop all
    echo "Stopping services..."
    pkill -TERM -f "$pattern"
    sleep 2
    
    # Force kill if still running
    if pgrep -f "$pattern" > /dev/null; then
        echo "Force killing remaining processes"
        pkill -KILL -f "$pattern"
    fi
    
    # Restart all
    echo "Restarting services..."
    for cmd in "${commands[@]}"; do
        eval "$cmd" &
        echo "  Restarted: $cmd (PID: $!)"
    done
}

# Usage:
restart_services "worker_script"
```

---

## Real-World DevOps Scenarios

### Scenario 1: Long-Running Database Migration

```bash
# Problem: Database migration takes 6 hours
# Solution: Run with nohup and monitoring

# Migration script
cat > /tmp/db_migration.sh << 'EOF'
#!/bin/bash

LOG="/var/log/migration.log"
LOCK="/tmp/migration.lock"

# Prevent concurrent runs
if [ -f "$LOCK" ]; then
    echo "Migration already running"
    exit 1
fi

touch "$LOCK"
trap "rm -f $LOCK" EXIT

echo "$(date): Starting migration" | tee -a "$LOG"

# Simulate migration steps
for i in {1..100}; do
    echo "$(date): Migrating batch $i/100" | tee -a "$LOG"
    sleep 60  # Each batch takes 1 minute
    
    # Check for stop signal
    if [ -f "/tmp/migration.stop" ]; then
        echo "$(date): Migration stopped by user" | tee -a "$LOG"
        exit 2
    fi
done

echo "$(date): Migration complete" | tee -a "$LOG"
EOF

chmod +x /tmp/db_migration.sh

# Start with nohup
nohup /tmp/db_migration.sh > /var/log/migration_output.log 2>&1 &
echo $! > /tmp/migration.pid

# Monitor progress
tail -f /var/log/migration.log

# Pause if needed (emergency)
kill -STOP $(cat /tmp/migration.pid)

# Resume later
kill -CONT $(cat /tmp/migration.pid)

# Graceful stop
touch /tmp/migration.stop

# Check status
if ps -p $(cat /tmp/migration.pid) > /dev/null; then
    echo "Migration still running"
else
    echo "Migration completed"
fi
```

### Scenario 2: Multi-Stage Deployment Pipeline

```bash
# Deployment with multiple stages running in parallel

cat > /tmp/deploy.sh << 'EOF'
#!/bin/bash

STAGES=(
    "build_frontend"
    "build_backend"
    "run_tests"
    "build_docker_images"
    "push_to_registry"
)

declare -A STAGE_PIDS
LOG_DIR="/var/log/deploy"
mkdir -p "$LOG_DIR"

# Stage functions
build_frontend() {
    echo "Building frontend..."
    sleep 30
    echo "Frontend built"
}

build_backend() {
    echo "Building backend..."
    sleep 30
    echo "Backend built"
}

run_tests() {
    echo "Running tests..."
    sleep 60
    echo "Tests passed"
}

build_docker_images() {
    echo "Building Docker images..."
    sleep 45
    echo "Images built"
}

push_to_registry() {
    echo "Pushing to registry..."
    sleep 20
    echo "Push complete"
}

# Execute stage in background
run_stage() {
    local stage=$1
    
    echo "Starting $stage"
    $stage > "$LOG_DIR/$stage.log" 2>&1 &
    STAGE_PIDS[$stage]=$!
    echo "  $stage PID: ${STAGE_PIDS[$stage]}"
}

# Wait for stage to complete
wait_for_stage() {
    local stage=$1
    local pid=${STAGE_PIDS[$stage]}
    
    echo "Waiting for $stage (PID: $pid)"
    
    while ps -p $pid > /dev/null; do
        echo -n "."
        sleep 5
    done
    
    echo ""
    
    # Check exit status
    wait $pid
    local status=$?
    
    if [ $status -eq 0 ]; then
        echo "✓ $stage completed successfully"
        return 0
    else
        echo "✗ $stage failed with status $status"
        return 1
    fi
}

# Main deployment flow
echo "=========================================="
echo "  Deployment Pipeline"
echo "=========================================="

# Stage 1: Parallel builds
echo "Stage 1: Building..."
run_stage build_frontend
run_stage build_backend

wait_for_stage build_frontend
wait_for_stage build_backend

# Stage 2: Tests (depends on builds)
echo "Stage 2: Testing..."
run_stage run_tests
wait_for_stage run_tests || exit 1

# Stage 3: Docker
echo "Stage 3: Docker images..."
run_stage build_docker_images
wait_for_stage build_docker_images || exit 1

# Stage 4: Push
echo "Stage 4: Registry push..."
run_stage push_to_registry
wait_for_stage push_to_registry || exit 1

echo "=========================================="
echo "  Deployment Complete!"
echo "=========================================="
EOF

chmod +x /tmp/deploy.sh

# Run deployment
/tmp/deploy.sh

# Monitor in real-time with tmux
tmux new-session -d -s deploy "/tmp/deploy.sh"
tmux split-window -h "tail -f /var/log/deploy/build_frontend.log"
tmux split-window -v "tail -f /var/log/deploy/build_backend.log"
tmux attach -t deploy
```

### Scenario 3: Service Health Monitoring & Auto-Restart

```bash
cat > /tmp/service_monitor.sh << 'EOF'
#!/bin/bash

SERVICES=(
    "nginx:nginx"
    "mysql:mysqld"
    "redis:redis-server"
)

CHECK_INTERVAL=30
MAX_RESTARTS=3
declare -A RESTART_COUNT

monitor_service() {
    local name=$1
    local process=$2
    
    if pgrep -x "$process" > /dev/null; then
        echo "$(date): ✓ $name is running"
        RESTART_COUNT[$name]=0
    else
        echo "$(date): ✗ $name is DOWN"
        
        # Increment restart counter
        RESTART_COUNT[$name]=$((${RESTART_COUNT[$name]:-0} + 1))
        
        if [ ${RESTART_COUNT[$name]} -le $MAX_RESTARTS ]; then
            echo "  Attempting restart (${RESTART_COUNT[$name]}/$MAX_RESTARTS)"
            
            case $name in
                nginx)
                    sudo systemctl start nginx
                    ;;
                mysql)
                    sudo systemctl start mysql
                    ;;
                redis)
                    sudo systemctl start redis
                    ;;
            esac
            
            sleep 5
            
            if pgrep -x "$process" > /dev/null; then
                echo "  ✓ $name restarted successfully"
            else
                echo "  ✗ Failed to restart $name"
            fi
        else
            echo "  ! Max restarts reached for $name"
            # Send alert
            echo "ALERT: $name failed to restart" | \
                mail -s "Service Alert" admin@example.com
        fi
    fi
}

# Main monitoring loop
echo "Starting service monitor..."
while true; do
    for service in "${SERVICES[@]}"; do
        IFS=':' read -r name process <<< "$service"
        monitor_service "$name" "$process"
    done
    
    sleep $CHECK_INTERVAL
done
EOF

chmod +x /tmp/service_monitor.sh

# Run in background with nohup
nohup /tmp/service_monitor.sh > /var/log/service_monitor.log 2>&1 &
echo $! > /tmp/service_monitor.pid
```

---

## Cheat Sheet

```bash
# ============================================
# PROCESS CONTROL QUICK REFERENCE
# ============================================

# BACKGROUND PROCESSES
command &                    # Run in background
nohup command &             # Persist after logout
nohup cmd > log 2>&1 &      # With output redirect
disown %1                   # Detach from shell

# JOB CONTROL
jobs                        # List jobs
jobs -l                     # With PIDs
fg %1                       # Bring to foreground
bg %1                       # Resume in background
Ctrl+Z                      # Suspend current job
Ctrl+C                      # Kill current job

# PROCESS INFO
ps aux                      # All processes
ps -ef                      # Full format
pgrep name                  # Find by name
pstree                      # Process tree
top / htop                  # Interactive monitor

# KILL PROCESSES
kill PID                    # Send TERM
kill -9 PID                 # Force kill
kill -HUP PID              # Reload config
killall name               # Kill by name
pkill pattern              # Kill by pattern

# SIGNALS
kill -l                     # List all signals
kill -TERM PID             # Graceful shutdown
kill -KILL PID             # Force kill
kill -HUP PID              # Reload
kill -USR1 PID             # User signal 1
kill -USR2 PID             # User signal 2

# SCREEN
screen                      # Start session
screen -S name             # Named session
screen -ls                 # List sessions
screen -r name             # Reattach
Ctrl+A D                   # Detach
Ctrl+A C                   # New window
Ctrl+A N/P                 # Next/Previous window

# TMUX
tmux                       # Start session
tmux new -s name          # Named session
tmux ls                    # List sessions
tmux attach -t name       # Reattach
Ctrl+B D                   # Detach
Ctrl+B C                   # New window
Ctrl+B "                   # Split horizontal
Ctrl+B %                   # Split vertical

# COMMON PATTERNS
# Graceful kill:
kill -TERM PID; sleep 5; kill -KILL PID

# Find and kill:
pkill -f "pattern"

# Kill all user processes:
pkill -u username

# Monitor process:
watch -n 1 'ps aux | grep process'

# Background with log:
nohup command > output.log 2>&1 &
```

---

## Muscle Memory Drills

### Drill 1: Launch and Control (2 minutes)

```bash
# Practice this sequence 10 times:

# 1. Start background process
sleep 100 &

# 2. Check jobs
jobs

# 3. Bring to foreground
fg

# 4. Suspend (Ctrl+Z)

# 5. Resume in background
bg

# 6. Kill it
kill %1

# Goal: Complete without looking at reference
```

### Drill 2: tmux Quick Setup (3 minutes)

```bash
# Practice creating this layout quickly:

# 1. Start tmux
tmux new -s practice

# 2. Split horizontal
# Ctrl+B "

# 3. Split vertical in top pane
# Navigate to top: Ctrl+B Arrow Up
# Ctrl+B %

# 4. Create new window
# Ctrl+B C

# 5. Rename window
# Ctrl+B ,
# Type: monitoring

# 6. Detach
# Ctrl+B D

# 7. Reattach
tmux attach -t practice

# Goal: < 30 seconds
```

### Drill 3: Mass Process Management (5 minutes)

```bash
# Launch 20 processes quickly:

# Create launcher script:
cat > /tmp/launcher.sh << 'EOF'
for i in {1..20}; do
    sleep $((RANDOM % 100)) &
done
jobs
EOF

bash /tmp/launcher.sh

# Now practice:
# 1. List all jobs (jobs -l)
# 2. Kill odd-numbered jobs (kill %1 %3 %5...)
# 3. Bring one to foreground (fg %2)
# 4. Send to background (Ctrl+Z, bg)
# 5. Kill all remaining (killall sleep)

# Goal: Complete in under 2 minutes
```

---

## Final Test: Manage 20+ Processes

```bash
# Complete this challenge to prove mastery:

# 1. Create test environment
cat > /tmp/worker.sh << 'EOF'
#!/bin/bash
ID=$1
while true; do
    echo "Worker $ID: $(date)"
    sleep 5
done
EOF

chmod +x /tmp/worker.sh

# 2. Launch 20 workers in tmux
tmux new-session -d -s test "for i in {1..20}; do /tmp/worker.sh \$i > /tmp/worker_\$i.log 2>&1 & done; bash"

# 3. Attach and verify
tmux attach -t test
# Expected: 20 background jobs running

# 4. Create monitoring pane
# Ctrl+B "
# In new pane:
watch -n 1 'jobs | wc -l'

# 5. Create log viewer pane
# Ctrl+B %
tail -f /tmp/worker_1.log

# 6. In original pane, control processes:
# - Kill workers 1-5
kill %1 %2 %3 %4 %5

# - Send HUP to workers 6-10
kill -HUP %6 %7 %8 %9 %10

# - Kill all remaining
killall bash

# 7. Verify all stopped
jobs

# Success criteria:
# ✓ All 20 processes launched
# ✓ Monitored in tmux
# ✓ Controlled individually
# ✓ Cleaned up completely
```

---

## Conclusion

You now have mastery over:
- ✅ Background process execution
- ✅ nohup for persistence
- ✅ screen and tmux for session management
- ✅ Job control (jobs, fg, bg)
- ✅ Process termination (kill, killall, pkill)
- ✅ Signal handling (TERM, KILL, HUP, USR1, USR2)
- ✅ Managing 20+ simultaneous processes

**Next Steps:**
1. Practice daily with real workloads
2. Automate deployments using these techniques
3. Build monitoring scripts
4. Create your own process management tools

**Remember:** Process control is a core DevOps skill. Master it, and you'll handle any production scenario with confidence! 🚀