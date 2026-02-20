#  Log Analysis Mastery   🔍


---

# 📚 Table of Contents

1. [Understanding Linux Logging Architecture](#part-1-understanding-linux-logging-architecture)
2. [Journalctl Mastery](#part-2-journalctl-mastery)
3. [Real-Time Log Monitoring](#part-3-real-time-log-monitoring)
4. [Automated Log Parsing](#part-4-automated-log-parsing)
5. [5-Minute Root Cause Analysis](#part-5-5-minute-root-cause-analysis)
6. [Muscle Memory Drills](#part-6-muscle-memory-drills)
7. [Cheat Sheets](#part-7-cheat-sheets)

---

# Part 1: Understanding Linux Logging Architecture

## 🏗️ The Big Picture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    LINUX LOGGING ARCHITECTURE                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐             │
│   │   Kernel     │    │   Services   │    │ Applications │             │
│   │   Messages   │    │   (systemd)  │    │   (nginx,etc)│             │
│   └──────┬───────┘    └──────┬───────┘    └──────┬───────┘             │
│          │                   │                   │                      │
│          ▼                   ▼                   ▼                      │
│   ┌─────────────────────────────────────────────────────────┐          │
│   │                    systemd-journald                      │          │
│   │              (Binary Journal Storage)                    │          │
│   │                /var/log/journal/                         │          │
│   └─────────────────────────┬───────────────────────────────┘          │
│                             │                                           │
│                             ▼                                           │
│   ┌─────────────────────────────────────────────────────────┐          │
│   │                      rsyslog                             │          │
│   │              (Traditional Text Logs)                     │          │
│   │                   /var/log/                              │          │
│   └─────────────────────────────────────────────────────────┘          │
│                                                                         │
│   KEY LOG FILES:                                                        │
│   ├── /var/log/syslog      → General system messages                   │
│   ├── /var/log/auth.log    → Authentication events                     │
│   ├── /var/log/kern.log    → Kernel messages                           │
│   ├── /var/log/dpkg.log    → Package management                        │
│   ├── /var/log/nginx/      → Web server logs                           │
│   └── /var/log/mysql/      → Database logs                             │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## 🎯 Why Two Logging Systems?

### journald (Modern)
```
┌────────────────────────────────────────┐
│           journald BENEFITS            │
├────────────────────────────────────────┤
│ ✓ Structured data (JSON-like)          │
│ ✓ Indexed for fast searching           │
│ ✓ Automatic rotation                   │
│ ✓ Cryptographic sealing                │
│ ✓ Boot session tracking                │
│ ✓ Rich metadata (PID, UID, etc.)       │
└────────────────────────────────────────┘
```

### rsyslog (Traditional)
```
┌────────────────────────────────────────┐
│           rsyslog BENEFITS             │
├────────────────────────────────────────┤
│ ✓ Plain text (easy to grep)            │
│ ✓ Remote logging support               │
│ ✓ Widely compatible                    │
│ ✓ Simple configuration                 │
│ ✓ Third-party tool support             │
└────────────────────────────────────────┘
```

## 📊 Log Priority Levels

```
┌────────┬────────────────┬─────────────────────────────────────────────┐
│ Level  │ Name           │ Description & When to Use                   │
├────────┼────────────────┼─────────────────────────────────────────────┤
│   0    │ emerg          │ 🔴 System unusable - panic situation        │
│   1    │ alert          │ 🔴 Immediate action required                │
│   2    │ crit           │ 🔴 Critical conditions - hardware errors    │
│   3    │ err            │ 🟠 Error conditions - service failures      │
│   4    │ warning        │ 🟡 Warning conditions - disk space low      │
│   5    │ notice         │ 🟢 Normal but significant - config changes  │
│   6    │ info           │ 🔵 Informational - service started          │
│   7    │ debug          │ ⚪ Debug messages - verbose output          │
└────────┴────────────────┴─────────────────────────────────────────────┘

MEMORY TIP: "Every Angry Camel Eats Waffles, Not Ice-cream, Dude"
              E     A     C    E     W        N     I          D
```

---

# Part 2: Journalctl Mastery

## 🔧 Basic Command Structure

```bash
journalctl [OPTIONS] [MATCHES]
```

### Anatomy of a journalctl Command

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   journalctl -u nginx --since "1 hour ago" --priority=err -o json      │
│   ─────────── ─────── ────────────────────── ────────────── ────────   │
│       │          │              │                  │           │       │
│       │          │              │                  │           │       │
│    Command    Unit          Time Filter       Priority      Output     │
│               Filter                          Filter        Format     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## 📋 Essential Options Reference

### Category 1: Unit and Service Filtering

```bash
# View logs for a specific service
journalctl -u nginx.service

# WHY: -u stands for "unit" (systemd unit)
# The .service suffix is optional

# Multiple services
journalctl -u nginx -u php-fpm -u mysql

# View user services
journalctl --user -u myapp.service
```

**Expected Output:**
```
-- Logs begin at Mon 2024-01-15 08:00:00 UTC, end at Mon 2024-01-15 14:30:00 UTC. --
Jan 15 08:00:01 webserver nginx[1234]: Starting A high performance web server...
Jan 15 08:00:01 webserver nginx[1234]: Started A high performance web server.
Jan 15 08:00:05 webserver nginx[1234]: 192.168.1.100 - - [15/Jan/2024:08:00:05] "GET / HTTP/1.1" 200
```

### Category 2: Time-Based Filtering

```bash
# ═══════════════════════════════════════════════════════════════
# RELATIVE TIME EXPRESSIONS
# ═══════════════════════════════════════════════════════════════

# Last hour
journalctl --since "1 hour ago"

# Last 30 minutes
journalctl --since "30 minutes ago"

# Since yesterday
journalctl --since yesterday

# Today only
journalctl --since today

# ═══════════════════════════════════════════════════════════════
# ABSOLUTE TIME EXPRESSIONS
# ═══════════════════════════════════════════════════════════════

# Specific date
journalctl --since "2024-01-15"

# Specific date and time
journalctl --since "2024-01-15 08:00:00"

# Time range
journalctl --since "2024-01-15 08:00:00" --until "2024-01-15 12:00:00"

# ═══════════════════════════════════════════════════════════════
# COMBINED EXAMPLES
# ═══════════════════════════════════════════════════════════════

# Last hour's nginx errors
journalctl -u nginx --since "1 hour ago" -p err

# Yesterday's SSH attempts
journalctl -u ssh --since yesterday --until today
```

### Time Format Quick Reference

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    TIME FORMAT EXAMPLES                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  RELATIVE:                                                              │
│  ├── "1 hour ago"      │  "30 minutes ago"    │  "2 days ago"          │
│  ├── "yesterday"       │  "today"             │  "now"                 │
│  └── "-1h"             │  "-30m"              │  "-2d"                 │
│                                                                         │
│  ABSOLUTE:                                                              │
│  ├── "2024-01-15"                    (midnight of that day)            │
│  ├── "2024-01-15 14:30"              (specific time)                   │
│  ├── "2024-01-15 14:30:00"           (with seconds)                    │
│  └── "14:30"                         (today at that time)              │
│                                                                         │
│  SPECIAL:                                                               │
│  ├── @1705320000                     (Unix timestamp)                  │
│  └── "2024-01-15 14:30:00 UTC"       (with timezone)                   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Category 3: Priority Filtering

```bash
# ═══════════════════════════════════════════════════════════════
# SINGLE PRIORITY
# ═══════════════════════════════════════════════════════════════

# Only errors
journalctl -p err

# Only warnings
journalctl -p warning

# ═══════════════════════════════════════════════════════════════
# PRIORITY RANGE
# ═══════════════════════════════════════════════════════════════

# Emergency through Error (0-3)
journalctl -p emerg..err

# Warning through Notice (4-5)
journalctl -p warning..notice

# ═══════════════════════════════════════════════════════════════
# PRACTICAL EXAMPLES
# ═══════════════════════════════════════════════════════════════

# Critical and above (system breaking)
journalctl -p crit

# All errors in the last hour for nginx
journalctl -u nginx -p err --since "1 hour ago"
```

**Understanding Priority Filtering:**
```
When you use -p err, you get err AND everything more severe:
┌─────────────────────────────────────────────────────────────┐
│  -p emerg    → Only emerg (0)                               │
│  -p alert    → emerg + alert (0-1)                          │
│  -p crit     → emerg + alert + crit (0-2)                   │
│  -p err      → emerg + alert + crit + err (0-3)  ← COMMON   │
│  -p warning  → Levels 0-4                                   │
│  -p notice   → Levels 0-5                                   │
│  -p info     → Levels 0-6                                   │
│  -p debug    → All levels (0-7)                             │
└─────────────────────────────────────────────────────────────┘
```

### Category 4: Output Formats

```bash
# ═══════════════════════════════════════════════════════════════
# OUTPUT FORMAT OPTIONS (-o or --output)
# ═══════════════════════════════════════════════════════════════

# Short (default) - one line per entry
journalctl -o short

# Short with precise timestamps
journalctl -o short-precise

# Short with ISO timestamps
journalctl -o short-iso

# Verbose - all fields
journalctl -o verbose

# JSON format (single line)
journalctl -o json

# JSON format (pretty printed)
journalctl -o json-pretty

# Export format (for piping)
journalctl -o export

# Cat format (message only)
journalctl -o cat
```

**Output Format Comparison:**

```bash
# Example log entry in different formats:

# short (default):
Jan 15 14:30:00 webserver nginx[1234]: Connection from 192.168.1.100

# short-precise:
Jan 15 14:30:00.123456 webserver nginx[1234]: Connection from 192.168.1.100

# short-iso:
2024-01-15T14:30:00+0000 webserver nginx[1234]: Connection from 192.168.1.100

# verbose:
Mon 2024-01-15 14:30:00.123456 UTC [s=abc123;i=1234;b=boot-id...]
    _TRANSPORT=syslog
    PRIORITY=6
    SYSLOG_FACILITY=3
    _UID=33
    _GID=33
    _PID=1234
    _COMM=nginx
    MESSAGE=Connection from 192.168.1.100
    
# json-pretty:
{
    "__REALTIME_TIMESTAMP" : "1705329000123456",
    "_HOSTNAME" : "webserver",
    "_COMM" : "nginx",
    "MESSAGE" : "Connection from 192.168.1.100",
    "PRIORITY" : "6"
}

# cat:
Connection from 192.168.1.100
```

### Category 5: Boot and Session Management

```bash
# ═══════════════════════════════════════════════════════════════
# BOOT SESSION COMMANDS
# ═══════════════════════════════════════════════════════════════

# List all boots
journalctl --list-boots

# Expected output:
# -3 abc123def456 Mon 2024-01-12 08:00:00 UTC—Mon 2024-01-12 23:59:00 UTC
# -2 def456abc789 Tue 2024-01-13 08:00:00 UTC—Tue 2024-01-13 23:59:00 UTC
# -1 789abc123def Wed 2024-01-14 08:00:00 UTC—Wed 2024-01-14 23:59:00 UTC
#  0 current12345 Thu 2024-01-15 08:00:00 UTC—Thu 2024-01-15 14:30:00 UTC

# Current boot only
journalctl -b

# Previous boot
journalctl -b -1

# Specific boot (by number)
journalctl -b -2

# Specific boot (by ID)
journalctl -b abc123def456

# ═══════════════════════════════════════════════════════════════
# WHY BOOT FILTERING MATTERS
# ═══════════════════════════════════════════════════════════════
# After a crash, you need to see what happened BEFORE the reboot
# journalctl -b -1 shows the previous session where the crash occurred
```

### Category 6: Field Matching (Advanced)

```bash
# ═══════════════════════════════════════════════════════════════
# SPECIFIC FIELD MATCHING
# ═══════════════════════════════════════════════════════════════

# By PID
journalctl _PID=1234

# By UID (user ID)
journalctl _UID=1000

# By executable path
journalctl _EXE=/usr/sbin/nginx

# By command name
journalctl _COMM=nginx

# By hostname
journalctl _HOSTNAME=webserver

# By systemd unit
journalctl _SYSTEMD_UNIT=nginx.service

# ═══════════════════════════════════════════════════════════════
# COMBINING FIELD MATCHES
# ═══════════════════════════════════════════════════════════════

# AND logic (both conditions)
journalctl _COMM=nginx _UID=33

# OR logic (use + between)
journalctl _COMM=nginx + _COMM=php-fpm

# ═══════════════════════════════════════════════════════════════
# VIEWING AVAILABLE FIELDS
# ═══════════════════════════════════════════════════════════════

# List all field names
journalctl -N

# List values for a specific field
journalctl -F _COMM

# Expected output for -F _COMM:
# nginx
# php-fpm
# mysql
# sshd
# systemd
```

### Category 7: Practical Combination Examples

```bash
# ═══════════════════════════════════════════════════════════════
# SCENARIO 1: Web Server Troubleshooting
# ═══════════════════════════════════════════════════════════════

# All nginx errors in the last hour, JSON format
journalctl -u nginx \
    --since "1 hour ago" \
    -p err \
    -o json-pretty

# ═══════════════════════════════════════════════════════════════
# SCENARIO 2: Authentication Audit
# ═══════════════════════════════════════════════════════════════

# Failed SSH attempts today
journalctl -u ssh \
    --since today \
    -g "Failed|Invalid|refused" \
    --no-pager

# ═══════════════════════════════════════════════════════════════
# SCENARIO 3: System Stability Analysis
# ═══════════════════════════════════════════════════════════════

# Kernel messages from previous boot with errors
journalctl -k \
    -b -1 \
    -p err \
    -o short-iso

# ═══════════════════════════════════════════════════════════════
# SCENARIO 4: Service Restart Investigation
# ═══════════════════════════════════════════════════════════════

# When did MySQL restart and why?
journalctl -u mysql \
    --since "24 hours ago" \
    -g "start|stop|restart|killed|fail" \
    -o short-precise

# ═══════════════════════════════════════════════════════════════
# SCENARIO 5: Disk and Hardware Issues
# ═══════════════════════════════════════════════════════════════

# Look for disk errors
journalctl -k \
    --since "7 days ago" \
    -g "error|fail|I/O|sector|disk" \
    -p warning
```

## ⌨️ Keyboard Navigation in Journalctl

```
┌─────────────────────────────────────────────────────────────────────────┐
│              JOURNALCTL PAGER NAVIGATION (less)                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  MOVEMENT:                                                              │
│  ┌────────────┬───────────────────────────────────────────────────┐    │
│  │ Key        │ Action                                            │    │
│  ├────────────┼───────────────────────────────────────────────────┤    │
│  │ j / ↓      │ Move down one line                                │    │
│  │ k / ↑      │ Move up one line                                  │    │
│  │ Space / f  │ Move down one page                                │    │
│  │ b          │ Move up one page                                  │    │
│  │ g          │ Go to beginning                                   │    │
│  │ G          │ Go to end (most recent)                           │    │
│  │ 100g       │ Go to line 100                                    │    │
│  │ 50%        │ Go to 50% of file                                 │    │
│  └────────────┴───────────────────────────────────────────────────┘    │
│                                                                         │
│  SEARCHING:                                                             │
│  ┌────────────┬───────────────────────────────────────────────────┐    │
│  │ /pattern   │ Search forward for pattern                        │    │
│  │ ?pattern   │ Search backward for pattern                       │    │
│  │ n          │ Next search result                                │    │
│  │ N          │ Previous search result                            │    │
│  │ &pattern   │ Show only lines matching pattern                  │    │
│  └────────────┴───────────────────────────────────────────────────┘    │
│                                                                         │
│  OTHER:                                                                 │
│  ┌────────────┬───────────────────────────────────────────────────┐    │
│  │ q          │ Quit                                              │    │
│  │ h          │ Help                                              │    │
│  │ -S         │ Toggle line wrapping                              │    │
│  │ -N         │ Toggle line numbers                               │    │
│  │ F          │ Follow mode (like tail -f)                        │    │
│  │ Ctrl+C     │ Exit follow mode                                  │    │
│  └────────────┴───────────────────────────────────────────────────┘    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## 🔄 Following Logs in Real-Time

```bash
# Follow current logs (like tail -f)
journalctl -f

# Follow specific service
journalctl -f -u nginx

# Follow with filtering
journalctl -f -u nginx -p err

# Follow multiple services
journalctl -f -u nginx -u php-fpm -u mysql
```

---

# Part 3: Real-Time Log Monitoring

## 🔍 The Classic: tail -f

### Basic Usage

```bash
# ═══════════════════════════════════════════════════════════════
# BASIC TAIL COMMANDS
# ═══════════════════════════════════════════════════════════════

# Follow a single log file
tail -f /var/log/syslog

# WHY: -f means "follow" - shows new lines as they're added

# Show last 50 lines then follow
tail -n 50 -f /var/log/syslog

# Follow multiple files
tail -f /var/log/nginx/access.log /var/log/nginx/error.log

# ═══════════════════════════════════════════════════════════════
# ADVANCED TAIL OPTIONS
# ═══════════════════════════════════════════════════════════════

# Retry if file doesn't exist (useful for rotated logs)
tail -F /var/log/nginx/access.log

# WHY: -F (capital) will retry and reopen if the file is recreated

# Follow with line count
tail -f -n 100 /var/log/syslog

# Follow from beginning
tail -f -n +1 /var/log/syslog

# Quiet mode (no headers with multiple files)
tail -f -q /var/log/nginx/*.log
```

### Combining tail with grep

```bash
# ═══════════════════════════════════════════════════════════════
# REAL-TIME FILTERING
# ═══════════════════════════════════════════════════════════════

# Follow and filter for errors
tail -f /var/log/syslog | grep -i error

# Filter for multiple patterns
tail -f /var/log/syslog | grep -iE "error|warning|fail"

# Exclude patterns
tail -f /var/log/nginx/access.log | grep -v "healthcheck"

# Case-insensitive with color
tail -f /var/log/syslog | grep -i --color=always error

# ═══════════════════════════════════════════════════════════════
# IMPORTANT: Use --line-buffered for real-time filtering
# ═══════════════════════════════════════════════════════════════

tail -f /var/log/syslog | grep --line-buffered error

# WHY: Without --line-buffered, grep may buffer output and delay display

# Chain multiple greps
tail -f /var/log/auth.log | \
    grep --line-buffered "sshd" | \
    grep --line-buffered -iE "fail|invalid"
```

## 🎨 Multitail: Advanced Multi-Log Monitoring

### Installation

```bash
sudo apt update
sudo apt install multitail -y
```

### Basic Multitail Usage

```bash
# ═══════════════════════════════════════════════════════════════
# SIMPLE MULTI-FILE VIEWING
# ═══════════════════════════════════════════════════════════════

# Two files, split horizontally
multitail /var/log/syslog /var/log/auth.log

# Two files, split vertically
multitail -s 2 /var/log/syslog /var/log/auth.log

# Three files with custom splits
multitail -s 3 /var/log/syslog /var/log/auth.log /var/log/kern.log
```

### Visual Layout

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         MULTITAIL LAYOUTS                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  DEFAULT (horizontal split):         VERTICAL (-s 2):                   │
│  ┌─────────────────────────┐         ┌────────────┬────────────┐       │
│  │ /var/log/syslog         │         │ syslog     │ auth.log   │       │
│  │ ...log entries...       │         │ ...        │ ...        │       │
│  │                         │         │            │            │       │
│  ├─────────────────────────┤         │            │            │       │
│  │ /var/log/auth.log       │         │            │            │       │
│  │ ...log entries...       │         │            │            │       │
│  └─────────────────────────┘         └────────────┴────────────┘       │
│                                                                         │
│  CUSTOM LAYOUT (-s 3):                                                  │
│  ┌────────────┬────────────┬────────────┐                              │
│  │ syslog     │ auth.log   │ kern.log   │                              │
│  │            │            │            │                              │
│  └────────────┴────────────┴────────────┘                              │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Advanced Multitail Features

```bash
# ═══════════════════════════════════════════════════════════════
# MERGING LOGS (combine into single view with colors)
# ═══════════════════════════════════════════════════════════════

multitail -I /var/log/syslog -I /var/log/auth.log

# WHY: -I (capital i) merges files into one window with different colors

# ═══════════════════════════════════════════════════════════════
# FILTERING WITH MULTITAIL
# ═══════════════════════════════════════════════════════════════

# Filter for specific pattern
multitail -e "error" /var/log/syslog

# Multiple filters
multitail -e "error" -e "warning" /var/log/syslog

# Exclude patterns
multitail -ev "DEBUG" /var/log/myapp.log

# Regular expression filtering
multitail -E "error|fail|critical" /var/log/syslog

# ═══════════════════════════════════════════════════════════════
# EXECUTING COMMANDS
# ═══════════════════════════════════════════════════════════════

# Monitor command output
multitail -l "journalctl -f -u nginx"

# Multiple commands
multitail -l "journalctl -f -u nginx" -l "journalctl -f -u mysql"

# Mix files and commands
multitail /var/log/syslog -l "dmesg -w"
```

### Multitail Color Schemes

```bash
# ═══════════════════════════════════════════════════════════════
# CREATING COLOR SCHEMES
# ═══════════════════════════════════════════════════════════════

# Use predefined color scheme
multitail -cS apache /var/log/nginx/access.log

# Available schemes: apache, syslog, acctail, etc.

# ═══════════════════════════════════════════════════════════════
# CUSTOM COLOR CONFIGURATION
# ═══════════════════════════════════════════════════════════════

# Create custom config: ~/.multitailrc
cat > ~/.multitailrc << 'EOF'
# Custom color for errors
colorscheme:myapp
cs_re:red:.*ERROR.*
cs_re:yellow:.*WARNING.*
cs_re:green:.*INFO.*
cs_re:blue:.*DEBUG.*
EOF

# Use custom scheme
multitail -cS myapp /var/log/myapp.log
```

### Multitail Keyboard Shortcuts

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    MULTITAIL KEYBOARD SHORTCUTS                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  NAVIGATION:                                                            │
│  ┌─────────────┬──────────────────────────────────────────────────┐    │
│  │ b           │ Scroll back in current window                    │    │
│  │ B           │ Scroll back in all windows                       │    │
│  │ ↑/↓         │ Scroll line by line                              │    │
│  │ a           │ Add another file                                 │    │
│  │ d           │ Delete current window                            │    │
│  │ g           │ Add search filter                                │    │
│  │ e           │ Edit filter                                      │    │
│  │ /           │ Search within scrollback                         │    │
│  └─────────────┴──────────────────────────────────────────────────┘    │
│                                                                         │
│  WINDOW MANAGEMENT:                                                     │
│  ┌─────────────┬──────────────────────────────────────────────────┐    │
│  │ Tab         │ Switch between windows                           │    │
│  │ o           │ Clear scrollback buffer                          │    │
│  │ O           │ Clear all scrollback buffers                     │    │
│  │ p           │ Pause/unpause                                    │    │
│  │ m           │ Mark current position                            │    │
│  │ w           │ Write buffer to file                             │    │
│  └─────────────┴──────────────────────────────────────────────────┘    │
│                                                                         │
│  OTHER:                                                                 │
│  ┌─────────────┬──────────────────────────────────────────────────┐    │
│  │ h           │ Help                                             │    │
│  │ q / Q       │ Quit                                             │    │
│  │ i           │ Info about current log                           │    │
│  │ t           │ Toggle timestamp                                 │    │
│  │ c           │ Toggle colors                                    │    │
│  └─────────────┴──────────────────────────────────────────────────┘    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## 🛠️ Creating Custom Monitoring Scripts

### Real-Time Dashboard Script

```bash
#!/bin/bash
# File: ~/scripts/log-dashboard.sh
# Purpose: Create a comprehensive log monitoring dashboard

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Clear screen and show header
clear
echo -e "${BLUE}═══════════════════════════════════════════════════════════${NC}"
echo -e "${BLUE}           REAL-TIME LOG MONITORING DASHBOARD               ${NC}"
echo -e "${BLUE}═══════════════════════════════════════════════════════════${NC}"
echo ""

# Function to monitor with colors
monitor_logs() {
    tail -f /var/log/syslog | while read line; do
        if echo "$line" | grep -qi "error\|fail\|critical"; then
            echo -e "${RED}[ERROR] $line${NC}"
        elif echo "$line" | grep -qi "warning\|warn"; then
            echo -e "${YELLOW}[WARN]  $line${NC}"
        elif echo "$line" | grep -qi "success\|started\|running"; then
            echo -e "${GREEN}[OK]    $line${NC}"
        else
            echo "[INFO]  $line"
        fi
    done
}

monitor_logs
```

### Multi-Service Monitor

```bash
#!/bin/bash
# File: ~/scripts/service-monitor.sh
# Purpose: Monitor multiple services simultaneously

SERVICES=("nginx" "mysql" "php-fpm" "redis")

echo "Starting service log monitor..."
echo "Press Ctrl+C to exit"
echo ""

# Build multitail command
CMD="multitail"

for service in "${SERVICES[@]}"; do
    CMD="$CMD -l 'journalctl -f -u $service --no-pager'"
done

eval "$CMD"
```

---

# Part 4: Automated Log Parsing

## 🤖 Creating Error Detection Scripts

### Script 1: Basic Error Detector

```bash
#!/bin/bash
# File: ~/scripts/error-detector.sh
# Purpose: Detect and alert on log errors

# ═══════════════════════════════════════════════════════════════
# CONFIGURATION
# ═══════════════════════════════════════════════════════════════

LOG_FILE="/var/log/syslog"
ERROR_LOG="/var/log/error-alerts.log"
ALERT_EMAIL="admin@example.com"
CHECK_INTERVAL=60  # seconds

# Error patterns to watch for
ERROR_PATTERNS=(
    "error"
    "fail"
    "critical"
    "fatal"
    "exception"
    "out of memory"
    "disk full"
    "connection refused"
)

# ═══════════════════════════════════════════════════════════════
# FUNCTIONS
# ═══════════════════════════════════════════════════════════════

log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$ERROR_LOG"
}

check_for_errors() {
    local since_time=$(date -d "-${CHECK_INTERVAL} seconds" '+%Y-%m-%d %H:%M:%S')
    
    # Build grep pattern
    local pattern=$(IFS="|"; echo "${ERROR_PATTERNS[*]}")
    
    # Find errors
    local errors=$(journalctl --since "$since_time" -p err -o short-iso 2>/dev/null)
    
    if [ -n "$errors" ]; then
        local error_count=$(echo "$errors" | wc -l)
        log_message "ALERT: Found $error_count error(s) in the last ${CHECK_INTERVAL}s"
        echo "$errors" >> "$ERROR_LOG"
        
        # Send alert if email is configured
        if command -v mail &> /dev/null && [ -n "$ALERT_EMAIL" ]; then
            echo "$errors" | mail -s "Server Alert: $error_count Errors Detected" "$ALERT_EMAIL"
        fi
        
        return 1
    fi
    
    return 0
}

# ═══════════════════════════════════════════════════════════════
# MAIN LOOP
# ═══════════════════════════════════════════════════════════════

log_message "Error detector started"
log_message "Monitoring: $LOG_FILE"
log_message "Check interval: ${CHECK_INTERVAL}s"

while true; do
    check_for_errors
    sleep "$CHECK_INTERVAL"
done
```

### Script 2: Comprehensive Log Analyzer

```bash
#!/bin/bash
# File: ~/scripts/log-analyzer.sh
# Purpose: Analyze logs and generate reports

# ═══════════════════════════════════════════════════════════════
# CONFIGURATION
# ═══════════════════════════════════════════════════════════════

REPORT_DIR="/var/log/reports"
REPORT_FILE="$REPORT_DIR/log-analysis-$(date +%Y%m%d-%H%M%S).txt"

mkdir -p "$REPORT_DIR"

# ═══════════════════════════════════════════════════════════════
# ANALYSIS FUNCTIONS
# ═══════════════════════════════════════════════════════════════

generate_header() {
    cat << EOF
╔═══════════════════════════════════════════════════════════════════════════╗
║                         LOG ANALYSIS REPORT                               ║
║                   Generated: $(date '+%Y-%m-%d %H:%M:%S')                       ║
╚═══════════════════════════════════════════════════════════════════════════╝

EOF
}

analyze_error_frequency() {
    echo "╔═══════════════════════════════════════════════════════════════════════════╗"
    echo "║                    ERROR FREQUENCY ANALYSIS (Last 24h)                   ║"
    echo "╚═══════════════════════════════════════════════════════════════════════════╝"
    echo ""
    
    echo "Errors by Hour:"
    echo "───────────────"
    for hour in {0..23}; do
        local count=$(journalctl --since "24 hours ago" -p err \
            --until "23 hours ago" 2>/dev/null | wc -l)
        printf "  %02d:00 - %02d:59  │ " $hour $hour
        local bar=$(printf '█%.0s' $(seq 1 $((count / 10 + 1))) 2>/dev/null)
        echo "$bar ($count)"
    done
    echo ""
}

analyze_top_error_sources() {
    echo "╔═══════════════════════════════════════════════════════════════════════════╗"
    echo "║                      TOP ERROR SOURCES (Last 24h)                        ║"
    echo "╚═══════════════════════════════════════════════════════════════════════════╝"
    echo ""
    
    echo "By Service/Unit:"
    echo "────────────────"
    journalctl --since "24 hours ago" -p err -o json 2>/dev/null | \
        jq -r '._SYSTEMD_UNIT // ._COMM // "unknown"' 2>/dev/null | \
        sort | uniq -c | sort -rn | head -10 | \
        while read count source; do
            printf "  %-30s │ %5d errors\n" "$source" "$count"
        done
    echo ""
}

analyze_failed_logins() {
    echo "╔═══════════════════════════════════════════════════════════════════════════╗"
    echo "║                    FAILED LOGIN ATTEMPTS (Last 24h)                      ║"
    echo "╚═══════════════════════════════════════════════════════════════════════════╝"
    echo ""
    
    echo "By IP Address:"
    echo "──────────────"
    journalctl --since "24 hours ago" -u ssh 2>/dev/null | \
        grep -i "failed\|invalid" | \
        grep -oE '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' | \
        sort | uniq -c | sort -rn | head -10 | \
        while read count ip; do
            printf "  %-20s │ %5d attempts\n" "$ip" "$count"
        done
    echo ""
}

analyze_disk_warnings() {
    echo "╔═══════════════════════════════════════════════════════════════════════════╗"
    echo "║                     DISK & STORAGE WARNINGS                              ║"
    echo "╚═══════════════════════════════════════════════════════════════════════════╝"
    echo ""
    
    journalctl --since "24 hours ago" -k 2>/dev/null | \
        grep -iE "disk|storage|I/O|sector|error" | \
        tail -20
    echo ""
}

generate_recommendations() {
    echo "╔═══════════════════════════════════════════════════════════════════════════╗"
    echo "║                        RECOMMENDATIONS                                    ║"
    echo "╚═══════════════════════════════════════════════════════════════════════════╝"
    echo ""
    
    local error_count=$(journalctl --since "24 hours ago" -p err 2>/dev/null | wc -l)
    local critical_count=$(journalctl --since "24 hours ago" -p crit 2>/dev/null | wc -l)
    
    if [ "$critical_count" -gt 0 ]; then
        echo "⚠️  CRITICAL: $critical_count critical errors in the last 24 hours"
        echo "   Action: Immediate investigation required"
        echo ""
    fi
    
    if [ "$error_count" -gt 100 ]; then
        echo "⚠️  HIGH ERROR RATE: $error_count errors in the last 24 hours"
        echo "   Action: Review error sources and implement fixes"
        echo ""
    fi
    
    # Check for specific issues
    if journalctl --since "24 hours ago" 2>/dev/null | grep -qi "out of memory"; then
        echo "⚠️  MEMORY ISSUES DETECTED"
        echo "   Action: Consider increasing RAM or optimizing applications"
        echo ""
    fi
    
    if journalctl --since "24 hours ago" 2>/dev/null | grep -qi "disk full\|no space"; then
        echo "⚠️  DISK SPACE ISSUES DETECTED"
        echo "   Action: Clean up logs, remove unused files, add storage"
        echo ""
    fi
}

# ═══════════════════════════════════════════════════════════════
# MAIN EXECUTION
# ═══════════════════════════════════════════════════════════════

{
    generate_header
    analyze_error_frequency
    analyze_top_error_sources
    analyze_failed_logins
    analyze_disk_warnings
    generate_recommendations
} | tee "$REPORT_FILE"

echo ""
echo "Report saved to: $REPORT_FILE"
```

### Script 3: Real-Time Alert System

```bash
#!/bin/bash
# File: ~/scripts/log-alerter.sh
# Purpose: Real-time alerting system with severity levels

# ═══════════════════════════════════════════════════════════════
# CONFIGURATION
# ═══════════════════════════════════════════════════════════════

# Slack webhook (optional)
SLACK_WEBHOOK=""

# Alert thresholds
CRITICAL_KEYWORDS="kernel panic|out of memory|disk full|segfault|fatal"
ERROR_KEYWORDS="error|failed|failure|exception"
WARNING_KEYWORDS="warning|deprecated|slow|timeout"

# Cooldown to prevent alert flooding (seconds)
ALERT_COOLDOWN=300

# ═══════════════════════════════════════════════════════════════
# ALERT FUNCTIONS
# ═══════════════════════════════════════════════════════════════

declare -A LAST_ALERT

send_alert() {
    local severity=$1
    local message=$2
    local key=$(echo "$message" | md5sum | cut -d' ' -f1)
    local now=$(date +%s)
    
    # Check cooldown
    if [ -n "${LAST_ALERT[$key]}" ]; then
        local diff=$((now - LAST_ALERT[$key]))
        if [ "$diff" -lt "$ALERT_COOLDOWN" ]; then
            return 0
        fi
    fi
    
    LAST_ALERT[$key]=$now
    
    # Terminal notification
    case $severity in
        "CRITICAL")
            echo -e "\033[41m\033[1;37m [CRITICAL] \033[0m $message"
            ;;
        "ERROR")
            echo -e "\033[0;31m [ERROR] \033[0m $message"
            ;;
        "WARNING")
            echo -e "\033[1;33m [WARNING] \033[0m $message"
            ;;
    esac
    
    # Desktop notification (if available)
    if command -v notify-send &> /dev/null; then
        notify-send -u critical "Log Alert: $severity" "$message"
    fi
    
    # Slack notification
    if [ -n "$SLACK_WEBHOOK" ]; then
        local color="warning"
        [ "$severity" = "CRITICAL" ] && color="danger"
        [ "$severity" = "ERROR" ] && color="danger"
        
        curl -s -X POST "$SLACK_WEBHOOK" \
            -H 'Content-type: application/json' \
            --data "{
                \"attachments\": [{
                    \"color\": \"$color\",
                    \"title\": \"Log Alert: $severity\",
                    \"text\": \"$message\",
                    \"footer\": \"$(hostname) | $(date)\"
                }]
            }" > /dev/null
    fi
}

# ═══════════════════════════════════════════════════════════════
# MAIN MONITORING LOOP
# ═══════════════════════════════════════════════════════════════

echo "Starting real-time log alerter..."
echo "Monitoring journalctl for alerts..."
echo "Press Ctrl+C to stop"
echo ""

journalctl -f -p warning --no-pager | while read line; do
    # Check for critical patterns
    if echo "$line" | grep -qiE "$CRITICAL_KEYWORDS"; then
        send_alert "CRITICAL" "$line"
    # Check for error patterns
    elif echo "$line" | grep -qiE "$ERROR_KEYWORDS"; then
        send_alert "ERROR" "$line"
    # Check for warning patterns
    elif echo "$line" | grep -qiE "$WARNING_KEYWORDS"; then
        send_alert "WARNING" "$line"
    fi
done
```

## 📊 Log Parsing with awk and sed

### Common Parsing Patterns

```bash
# ═══════════════════════════════════════════════════════════════
# AWK PATTERNS FOR LOG ANALYSIS
# ═══════════════════════════════════════════════════════════════

# Extract timestamp and message
journalctl -o short --since today | \
    awk '{print $1, $2, $3, $NF}'

# Count occurrences by service
journalctl --since today -o json | \
    jq -r '._SYSTEMD_UNIT // "unknown"' | \
    sort | uniq -c | sort -rn

# Average response time from nginx
awk '{
    sum += $(NF-1);  # Response time is usually second to last
    count++
} END {
    print "Average response time:", sum/count, "ms"
}' /var/log/nginx/access.log

# Count HTTP status codes
awk '{print $9}' /var/log/nginx/access.log | \
    sort | uniq -c | sort -rn

# Find requests taking more than 1 second
awk '$NF > 1.0 {print}' /var/log/nginx/access.log

# ═══════════════════════════════════════════════════════════════
# SED PATTERNS FOR LOG CLEANING
# ═══════════════════════════════════════════════════════════════

# Remove ANSI color codes
sed 's/\x1b\[[0-9;]*m//g' log.txt

# Extract IP addresses
sed -n 's/.*\([0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\).*/\1/p' /var/log/auth.log

# Convert timestamp format
sed 's/\([0-9]\{4\}\)-\([0-9]\{2\}\)-\([0-9]\{2\}\)/\2\/\3\/\1/g' log.txt
```

### Creating a Log Parser Tool

```bash
#!/bin/bash
# File: ~/scripts/logparse.sh
# Purpose: Swiss-army knife for log parsing

usage() {
    cat << EOF
Usage: logparse.sh [OPTIONS] [LOGFILE]

OPTIONS:
    -e, --errors          Show only errors
    -w, --warnings        Show warnings and above
    -i, --ip              Extract IP addresses
    -t, --top N           Show top N occurrences
    -s, --since TIME      Filter by time (e.g., "1 hour ago")
    -g, --grep PATTERN    Filter by pattern
    -c, --count           Count lines matching criteria
    -j, --json            Output as JSON
    -h, --help            Show this help

EXAMPLES:
    logparse.sh -e /var/log/syslog
    logparse.sh --top 10 --ip /var/log/auth.log
    logparse.sh -s "1 hour ago" -g "nginx" 
EOF
}

# Parse arguments
ERRORS_ONLY=false
WARNINGS=false
EXTRACT_IP=false
TOP_N=0
SINCE_TIME=""
GREP_PATTERN=""
COUNT_ONLY=false
JSON_OUTPUT=false

while [[ $# -gt 0 ]]; do
    case $1 in
        -e|--errors)
            ERRORS_ONLY=true
            shift
            ;;
        -w|--warnings)
            WARNINGS=true
            shift
            ;;
        -i|--ip)
            EXTRACT_IP=true
            shift
            ;;
        -t|--top)
            TOP_N="$2"
            shift 2
            ;;
        -s|--since)
            SINCE_TIME="$2"
            shift 2
            ;;
        -g|--grep)
            GREP_PATTERN="$2"
            shift 2
            ;;
        -c|--count)
            COUNT_ONLY=true
            shift
            ;;
        -j|--json)
            JSON_OUTPUT=true
            shift
            ;;
        -h|--help)
            usage
            exit 0
            ;;
        *)
            LOGFILE="$1"
            shift
            ;;
    esac
done

# Build and execute the pipeline
build_pipeline() {
    local cmd=""
    
    # Input source
    if [ -n "$SINCE_TIME" ]; then
        cmd="journalctl --since \"$SINCE_TIME\" --no-pager"
    elif [ -n "$LOGFILE" ]; then
        cmd="cat \"$LOGFILE\""
    else
        cmd="journalctl --since today --no-pager"
    fi
    
    # Filters
    if [ "$ERRORS_ONLY" = true ]; then
        cmd="$cmd | grep -iE 'error|fail|critical|fatal'"
    elif [ "$WARNINGS" = true ]; then
        cmd="$cmd | grep -iE 'error|fail|critical|fatal|warning|warn'"
    fi
    
    if [ -n "$GREP_PATTERN" ]; then
        cmd="$cmd | grep -iE \"$GREP_PATTERN\""
    fi
    
    # IP extraction
    if [ "$EXTRACT_IP" = true ]; then
        cmd="$cmd | grep -oE '\\b([0-9]{1,3}\\.){3}[0-9]{1,3}\\b'"
    fi
    
    # Counting
    if [ "$COUNT_ONLY" = true ]; then
        cmd="$cmd | wc -l"
    elif [ "$TOP_N" -gt 0 ]; then
        cmd="$cmd | sort | uniq -c | sort -rn | head -$TOP_N"
    fi
    
    # JSON output
    if [ "$JSON_OUTPUT" = true ]; then
        cmd="$cmd | jq -R -s 'split(\"\n\") | map(select(length > 0))'"
    fi
    
    echo "$cmd"
}

# Execute
eval "$(build_pipeline)"
```

---

# Part 5: 5-Minute Root Cause Analysis

## 🎯 The RCA Framework

```
┌─────────────────────────────────────────────────────────────────────────┐
│            5-MINUTE ROOT CAUSE ANALYSIS FRAMEWORK                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   MINUTE 1: TRIAGE                                                      │
│   ├── What service is affected?                                         │
│   ├── When did it start?                                                │
│   └── What's the error message?                                         │
│                                                                         │
│   MINUTE 2: TIMELINE                                                    │
│   ├── What happened just before the issue?                              │
│   ├── Were there any deployments?                                       │
│   └── Did anything else change?                                         │
│                                                                         │
│   MINUTE 3: SCOPE                                                       │
│   ├── Is this affecting other services?                                 │
│   ├── Is it isolated to one server?                                     │
│   └── Are there related errors?                                         │
│                                                                         │
│   MINUTE 4: EVIDENCE                                                    │
│   ├── Collect relevant log excerpts                                     │
│   ├── Note error codes and messages                                     │
│   └── Document timeline of events                                       │
│                                                                         │
│   MINUTE 5: HYPOTHESIS                                                  │
│   ├── Form initial theory                                               │
│   ├── Identify verification steps                                       │
│   └── Plan immediate mitigation                                         │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## 🚀 Quick Diagnosis Commands

### The RCA Command Toolkit

```bash
#!/bin/bash
# File: ~/scripts/rca.sh
# Purpose: Rapid Root Cause Analysis toolkit

# ═══════════════════════════════════════════════════════════════
# MINUTE 1: TRIAGE
# ═══════════════════════════════════════════════════════════════

triage() {
    echo "╔═══════════════════════════════════════════════════════════════╗"
    echo "║                   MINUTE 1: TRIAGE                            ║"
    echo "╚═══════════════════════════════════════════════════════════════╝"
    echo ""
    
    # Most recent errors
    echo "📍 RECENT ERRORS (last 5 minutes):"
    echo "─────────────────────────────────────"
    journalctl --since "5 minutes ago" -p err --no-pager | tail -20
    echo ""
    
    # Failed services
    echo "📍 FAILED SERVICES:"
    echo "───────────────────"
    systemctl --failed
    echo ""
    
    # High-level system health
    echo "📍 SYSTEM HEALTH:"
    echo "─────────────────"
    echo "  Load: $(uptime | awk -F'load average:' '{print $2}')"
    echo "  Memory: $(free -h | awk '/^Mem:/ {print $3 "/" $2}')"
    echo "  Disk: $(df -h / | awk 'NR==2 {print $5 " used"}')"
    echo ""
}

# ═══════════════════════════════════════════════════════════════
# MINUTE 2: TIMELINE
# ═══════════════════════════════════════════════════════════════

timeline() {
    local service=$1
    local timeframe=${2:-"30 minutes ago"}
    
    echo "╔═══════════════════════════════════════════════════════════════╗"
    echo "║                   MINUTE 2: TIMELINE                          ║"
    echo "╚═══════════════════════════════════════════════════════════════╝"
    echo ""
    
    if [ -n "$service" ]; then
        echo "📍 TIMELINE FOR: $service"
        echo "────────────────────────────"
        journalctl -u "$service" --since "$timeframe" -o short-iso --no-pager
    else
        echo "📍 SYSTEM TIMELINE (all warnings+):"
        echo "────────────────────────────────────"
        journalctl --since "$timeframe" -p warning -o short-iso --no-pager | tail -50
    fi
    echo ""
    
    # Check for recent deployments/changes
    echo "📍 RECENT PACKAGE CHANGES:"
    echo "──────────────────────────"
    grep "$(date +%Y-%m-%d)" /var/log/dpkg.log 2>/dev/null | tail -10
    echo ""
}

# ═══════════════════════════════════════════════════════════════
# MINUTE 3: SCOPE
# ═══════════════════════════════════════════════════════════════

scope() {
    local pattern=$1
    
    echo "╔═══════════════════════════════════════════════════════════════╗"
    echo "║                    MINUTE 3: SCOPE                            ║"
    echo "╚═══════════════════════════════════════════════════════════════╝"
    echo ""
    
    echo "📍 ERROR DISTRIBUTION BY SERVICE:"
    echo "──────────────────────────────────"
    journalctl --since "30 minutes ago" -p err -o json 2>/dev/null | \
        jq -r '._SYSTEMD_UNIT // ._COMM // "kernel"' 2>/dev/null | \
        sort | uniq -c | sort -rn
    echo ""
    
    if [ -n "$pattern" ]; then
        echo "📍 RELATED ERRORS (pattern: $pattern):"
        echo "──────────────────────────────────────"
        journalctl --since "1 hour ago" -g "$pattern" --no-pager | tail -20
    fi
    echo ""
    
    # Check kernel messages
    echo "📍 KERNEL MESSAGES (OOM, hardware, etc.):"
    echo "──────────────────────────────────────────"
    journalctl -k --since "30 minutes ago" -p warning --no-pager | tail -10
    echo ""
}

# ═══════════════════════════════════════════════════════════════
# MINUTE 4: EVIDENCE COLLECTION
# ═══════════════════════════════════════════════════════════════

evidence() {
    local service=$1
    local output_file="/tmp/rca-evidence-$(date +%Y%m%d-%H%M%S).txt"
    
    echo "╔═══════════════════════════════════════════════════════════════╗"
    echo "║                   MINUTE 4: EVIDENCE                          ║"
    echo "╚═══════════════════════════════════════════════════════════════╝"
    echo ""
    
    {
        echo "RCA Evidence Collection"
        echo "Generated: $(date)"
        echo "Hostname: $(hostname)"
        echo "========================================"
        echo ""
        
        echo "=== SYSTEM INFO ==="
        uname -a
        echo ""
        
        echo "=== UPTIME ==="
        uptime
        echo ""
        
        echo "=== MEMORY ==="
        free -h
        echo ""
        
        echo "=== DISK ==="
        df -h
        echo ""
        
        echo "=== FAILED SERVICES ==="
        systemctl --failed
        echo ""
        
        echo "=== RECENT ERRORS (last hour) ==="
        journalctl --since "1 hour ago" -p err --no-pager
        echo ""
        
        if [ -n "$service" ]; then
            echo "=== SERVICE LOG: $service ==="
            journalctl -u "$service" --since "1 hour ago" --no-pager
            echo ""
        fi
        
    } | tee "$output_file"
    
    echo ""
    echo "📁 Evidence saved to: $output_file"
}

# ═══════════════════════════════════════════════════════════════
# MINUTE 5: QUICK CHECKS
# ═══════════════════════════════════════════════════════════════

quick_checks() {
    echo "╔═══════════════════════════════════════════════════════════════╗"
    echo "║              MINUTE 5: QUICK DIAGNOSTIC CHECKS                ║"
    echo "╚═══════════════════════════════════════════════════════════════╝"
    echo ""
    
    # Disk space
    echo "📍 DISK SPACE CHECK:"
    df -h | awk '$5 ~ /[8-9][0-9]%|100%/ {print "  ⚠️ WARNING: "$0}'
    df -h | awk '$5 !~ /[8-9][0-9]%|100%/ && NR>1 {print "  ✓ OK: "$6" at "$5}'
    echo ""
    
    # Memory
    echo "📍 MEMORY CHECK:"
    local mem_used=$(free | awk '/^Mem:/ {printf "%.0f", $3/$2 * 100}')
    if [ "$mem_used" -gt 90 ]; then
        echo "  ⚠️ WARNING: Memory at ${mem_used}%"
    else
        echo "  ✓ OK: Memory at ${mem_used}%"
    fi
    echo ""
    
    # Load
    echo "📍 LOAD CHECK:"
    local cpus=$(nproc)
    local load=$(uptime | awk -F'load average:' '{print $2}' | cut -d, -f1 | tr -d ' ')
    local load_int=${load%.*}
    if [ "$load_int" -gt "$cpus" ]; then
        echo "  ⚠️ WARNING: Load ($load) exceeds CPU count ($cpus)"
    else
        echo "  ✓ OK: Load ($load) within normal range"
    fi
    echo ""
    
    # Network
    echo "📍 NETWORK CHECK:"
    if ping -c 1 8.8.8.8 &>/dev/null; then
        echo "  ✓ OK: Internet connectivity"
    else
        echo "  ⚠️ WARNING: No internet connectivity"
    fi
    echo ""
    
    # Recent restarts
    echo "📍 RECENT SERVICE RESTARTS:"
    journalctl --since "1 hour ago" -g "Started|Stopped|Restarted" --no-pager | \
        grep -E "Started|Stopped" | tail -10
    echo ""
}

# ═══════════════════════════════════════════════════════════════
# FULL RCA WORKFLOW
# ═══════════════════════════════════════════════════════════════

full_rca() {
    local service=$1
    
    clear
    echo "╔═══════════════════════════════════════════════════════════════════════════╗"
    echo "║               5-MINUTE ROOT CAUSE ANALYSIS                                ║"
    echo "║                   Started: $(date '+%H:%M:%S')                                       ║"
    echo "╚═══════════════════════════════════════════════════════════════════════════╝"
    echo ""
    
    echo "Press any key to proceed to each minute..."
    echo ""
    
    read -n 1 -s -r
    triage
    
    read -n 1 -s -r -p "Press any key for Minute 2 (Timeline)..."
    echo ""
    timeline "$service"
    
    read -n 1 -s -r -p "Press any key for Minute 3 (Scope)..."
    echo ""
    scope
    
    read -n 1 -s -r -p "Press any key for Minute 4 (Evidence)..."
    echo ""
    evidence "$service"
    
    read -n 1 -s -r -p "Press any key for Minute 5 (Quick Checks)..."
    echo ""
    quick_checks
    
    echo ""
    echo "╔═══════════════════════════════════════════════════════════════════════════╗"
    echo "║                     RCA COMPLETE: $(date '+%H:%M:%S')                                ║"
    echo "╚═══════════════════════════════════════════════════════════════════════════╝"
}

# ═══════════════════════════════════════════════════════════════
# MAIN
# ═══════════════════════════════════════════════════════════════

case ${1:-full} in
    triage)     triage ;;
    timeline)   timeline "$2" "$3" ;;
    scope)      scope "$2" ;;
    evidence)   evidence "$2" ;;
    checks)     quick_checks ;;
    full)       full_rca "$2" ;;
    *)
        echo "Usage: rca.sh [triage|timeline|scope|evidence|checks|full] [service]"
        ;;
esac
```

## 🔍 Scenario-Based RCA Examples

### Scenario 1: Web Server Not Responding

```bash
# Step 1: Check nginx status
systemctl status nginx

# Step 2: Check recent nginx logs
journalctl -u nginx --since "10 minutes ago" -p warning

# Step 3: Check for port conflicts
sudo ss -tlnp | grep :80
sudo ss -tlnp | grep :443

# Step 4: Check for resource issues
journalctl -k --since "10 minutes ago" | grep -i "memory\|oom\|kill"

# Step 5: Check config validity
sudo nginx -t

# Common findings and fixes:
# ┌────────────────────────────┬───────────────────────────────────────┐
# │ Finding                    │ Fix                                   │
# ├────────────────────────────┼───────────────────────────────────────┤
# │ Port already in use        │ Kill conflicting process              │
# │ Config syntax error        │ Fix config, reload nginx              │
# │ Upstream timeout           │ Check backend servers                 │
# │ Out of memory              │ Increase memory, reduce workers       │
# │ Permission denied          │ Check file/socket permissions         │
# └────────────────────────────┴───────────────────────────────────────┘
```

### Scenario 2: High Server Load

```bash
# Step 1: Check current load
uptime
top -bn1 | head -20

# Step 2: Find resource-hungry processes
journalctl --since "30 minutes ago" -k | grep -iE "oom|memory|kill"

# Step 3: Check for runaway processes
ps aux --sort=-%cpu | head -10
ps aux --sort=-%mem | head -10

# Step 4: Check for I/O issues
journalctl --since "30 minutes ago" -k | grep -iE "io|disk|block"

# Step 5: Review service logs for errors
journalctl --since "30 minutes ago" -p err | head -50
```

### Scenario 3: Database Connection Failures

```bash
# Step 1: Check MySQL/PostgreSQL status
systemctl status mysql
# or
systemctl status postgresql

# Step 2: Check connection logs
journalctl -u mysql --since "10 minutes ago" | grep -iE "connection|refused|timeout"

# Step 3: Check current connections
mysqladmin status
# or
sudo -u postgres psql -c "SELECT count(*) FROM pg_stat_activity;"

# Step 4: Check for disk space (databases need space)
df -h /var/lib/mysql
df -h /var/lib/postgresql

# Step 5: Check for lock issues
journalctl -u mysql --since "30 minutes ago" | grep -iE "lock|wait|timeout"
```

### Scenario 4: SSH Connection Problems

```bash
# Step 1: Check SSH service
systemctl status ssh

# Step 2: Check authentication logs
journalctl -u ssh --since "10 minutes ago" -p warning

# Step 3: Check for fail2ban blocks
sudo fail2ban-client status sshd 2>/dev/null

# Step 4: Check SSH configuration
sudo sshd -T | head -20

# Step 5: Check for PAM issues
journalctl --since "10 minutes ago" | grep -i "pam"
```

---

# Part 6: Muscle Memory Drills

## 🏋️ Daily Practice Exercises

### Exercise Set 1: Basic Navigation (5 minutes daily)

```bash
# ═══════════════════════════════════════════════════════════════
# DRILL 1: Quick View Commands
# Goal: Execute each in under 3 seconds
# ═══════════════════════════════════════════════════════════════

# View last 50 system log entries
journalctl -n 50

# View today's logs
journalctl --since today

# View last hour's errors
journalctl --since "1 hour ago" -p err

# View kernel messages
journalctl -k

# Follow logs in real-time
journalctl -f

# ═══════════════════════════════════════════════════════════════
# DRILL 2: Service Filtering
# Goal: Type service filter commands without looking
# ═══════════════════════════════════════════════════════════════

# Practice these 10 times each:
journalctl -u nginx
journalctl -u ssh
journalctl -u mysql
journalctl -u docker

# Combine with time filters (practice 5 times each):
journalctl -u nginx --since today
journalctl -u ssh --since "1 hour ago" -p warning
```

### Exercise Set 2: Speed Typing Challenges

```bash
# ═══════════════════════════════════════════════════════════════
# CHALLENGE 1: Type these commands in under 10 seconds
# ═══════════════════════════════════════════════════════════════

# Level 1 (Beginner):
journalctl -f
journalctl -b
tail -f /var/log/syslog

# Level 2 (Intermediate):
journalctl -u nginx --since "1 hour ago"
journalctl -p err --since today
tail -f /var/log/nginx/error.log | grep error

# Level 3 (Advanced):
journalctl -u nginx --since "2024-01-15 08:00" --until "2024-01-15 12:00" -p warning -o json
journalctl --since "30 minutes ago" -p err -o json | jq -r '.MESSAGE'

# ═══════════════════════════════════════════════════════════════
# CHALLENGE 2: Keyboard Shortcut Speed Run
# Navigate through journalctl output using only keyboard
# ═══════════════════════════════════════════════════════════════

# Start with:
journalctl --since today

# Then practice:
# G     - Jump to end (3 times)
# g     - Jump to start (3 times)
# /error - Search for "error" (5 times)
# n     - Next result (10 times)
# N     - Previous result (5 times)
# q     - Quit

```

### Exercise Set 3: Problem-Solving Drills

```bash
# ═══════════════════════════════════════════════════════════════
# SCENARIO DRILL 1: "Website is down"
# Complete in under 2 minutes
# ═══════════════════════════════════════════════════════════════

# Step 1: Check service status
systemctl status nginx

# Step 2: View recent errors
journalctl -u nginx --since "5 minutes ago" -p err

# Step 3: Check for patterns
journalctl -u nginx --since "10 minutes ago" | grep -iE "error|fail|crash"

# Step 4: Check upstream services
journalctl -u php-fpm --since "5 minutes ago" -p warning

# ═══════════════════════════════════════════════════════════════
# SCENARIO DRILL 2: "Server is slow"
# Complete in under 2 minutes
# ═══════════════════════════════════════════════════════════════

# Step 1: Check for OOM kills
journalctl -k --since "30 minutes ago" | grep -i "oom\|kill"

# Step 2: Check high-error services
journalctl --since "30 minutes ago" -p err -o json | jq -r '._SYSTEMD_UNIT' | sort | uniq -c | sort -rn | head -5

# Step 3: Check disk I/O issues
journalctl -k --since "30 minutes ago" | grep -iE "io|disk|block"

# Step 4: Check for service restarts
journalctl --since "30 minutes ago" | grep -E "Started|Stopped|Restarting"
```

### Exercise Set 4: Finger Memory Patterns

```bash
# ═══════════════════════════════════════════════════════════════
# Pattern 1: The Quick Check
# Type this sequence 10 times until it's automatic
# ═══════════════════════════════════════════════════════════════

journalctl --since "5 minutes ago" -p err
# ↑ This should become your instinctive first response

# ═══════════════════════════════════════════════════════════════
# Pattern 2: The Service Deep Dive
# Practice this sequence for any service name
# ═══════════════════════════════════════════════════════════════

# Replace SERVICE with: nginx, mysql, ssh, docker
journalctl -u SERVICE --since "1 hour ago" -p warning -o short-iso

# ═══════════════════════════════════════════════════════════════
# Pattern 3: The Time Window Investigation
# Practice with different time windows
# ═══════════════════════════════════════════════════════════════

journalctl --since "YYYY-MM-DD HH:MM" --until "YYYY-MM-DD HH:MM" -p err
# Practice filling in times quickly

# ═══════════════════════════════════════════════════════════════
# Pattern 4: The Count and Aggregate
# Essential for understanding scope
# ═══════════════════════════════════════════════════════════════

journalctl --since "1 hour ago" -p err | wc -l
journalctl --since "1 hour ago" -p err -o json | jq -r '._SYSTEMD_UNIT' | sort | uniq -c | sort -rn
```

## 📊 Progress Tracking

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    SKILL PROGRESS TRACKER                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  BASIC COMMANDS:                                      Target: < 5s      │
│  □ journalctl -f                                      Your time: ___    │
│  □ journalctl -b                                      Your time: ___    │
│  □ journalctl --since today                           Your time: ___    │
│  □ tail -f /var/log/syslog                           Your time: ___    │
│                                                                         │
│  FILTERED QUERIES:                                    Target: < 10s     │
│  □ journalctl -u nginx --since "1 hour ago"          Your time: ___    │
│  □ journalctl -p err --since today                   Your time: ___    │
│  □ journalctl -k --since "30 min ago" | grep error   Your time: ___    │
│                                                                         │
│  COMPLEX QUERIES:                                     Target: < 20s     │
│  □ Full service investigation with time range        Your time: ___    │
│  □ JSON output with jq processing                    Your time: ___    │
│  □ Multi-service correlation