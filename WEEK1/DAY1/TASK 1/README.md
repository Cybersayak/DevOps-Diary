# Filesystem Navigation Mastery Guide

## Table of Contents
1. [Foundation Concepts](#foundation-concepts)
2. [Keyboard-Only Navigation](#keyboard-only-navigation)
3. [Directory Structure Creation](#directory-structure-creation)
4. [Find Command Mastery](#find-command-mastery)
5. [Speed Navigation Test](#speed-navigation-test)
6. [Muscle Memory Drills](#muscle-memory-drills)
7. [Real-World DevOps Use Cases](#real-world-devops-use-cases)

---

## Foundation Concepts

### Why Filesystem Navigation Matters in DevOps

```
┌─────────────────────────────────────────────────────┐
│  DevOps Engineer's Daily Navigation Tasks:          │
├─────────────────────────────────────────────────────┤
│  ✓ Deploy applications to different environments   │
│  ✓ Troubleshoot logs across multiple directories   │
│  ✓ Manage configuration files                      │
│  ✓ Search for specific files in large codebases    │
│  ✓ Create repeatable directory structures          │
│  ✓ Audit file permissions and ownership            │
└─────────────────────────────────────────────────────┘
```

### Linux Filesystem Hierarchy (Visual Map)

```
/
├── bin/        → Essential user binaries
├── boot/       → Boot loader files
├── dev/        → Device files
├── etc/        → Configuration files
├── home/       → User home directories
├── lib/        → System libraries
├── opt/        → Optional software
├── root/       → Root user's home
├── tmp/        → Temporary files
├── usr/        → User programs
│   ├── bin/
│   ├── local/
│   └── share/
└── var/        → Variable data
    ├── log/    → Log files (CRITICAL for DevOps)
    ├── www/    → Web server files
    └── tmp/    → Temporary files
```

---

## Keyboard-Only Navigation

### Part 1: Essential Terminal Shortcuts

#### Basic Movement Commands

```bash
# Command Structure:
# COMMAND [OPTIONS] [ARGUMENTS]

# pwd - Print Working Directory
pwd
# Output: /home/username
# Why: Always know your current location before navigating
```

**Detailed Breakdown:**
- **p**rint **w**orking **d**irectory
- No flags needed for basic use
- Returns absolute path
- Critical for orientation in complex directory trees

```bash
# cd - Change Directory
cd /path/to/directory

# Examples with explanations:
cd /                    # Go to root (topmost directory)
cd ~                    # Go to home directory (~/ is shorthand)
cd -                    # Go to previous directory (toggle between two locations)
cd ..                   # Go up one level
cd ../..                # Go up two levels
cd ../../etc            # Go up two, then into etc/
```

**Why Each Matters:**
- `cd /` - Reset to known starting point
- `cd ~` - Quick return to your workspace
- `cd -` - Bounce between working directories (HUGE time saver)
- `cd ..` - Navigate up the tree structure

#### Keyboard Shortcuts for Terminal

```
┌────────────────────────────────────────────────────┐
│  NAVIGATION SHORTCUTS (Muscle Memory Required)    │
├────────────────────────────────────────────────────┤
│  Ctrl + A  → Jump to beginning of line            │
│  Ctrl + E  → Jump to end of line                  │
│  Ctrl + U  → Clear line before cursor             │
│  Ctrl + K  → Clear line after cursor              │
│  Ctrl + W  → Delete word before cursor            │
│  Ctrl + L  → Clear screen (same as 'clear')       │
│  Ctrl + R  → Reverse search command history       │
│  Ctrl + C  → Cancel current command               │
│  Ctrl + D  → Exit terminal (or EOF)               │
│  Alt  + B  → Move backward one word               │
│  Alt  + F  → Move forward one word                │
│  !!        → Repeat last command                  │
│  !$        → Last argument of previous command    │
└────────────────────────────────────────────────────┘
```

**Practical Example:**

```bash
# Scenario: You typed a long path incorrectly
cd /var/log/apache2/sites-available/old-configs/2023/

# Instead of retyping:
# 1. Press Ctrl + A (jump to beginning)
# 2. Fix the command
# 3. Press Ctrl + E (jump to end) to continue

# Or use Ctrl + U to clear and start over
```

### Part 2: Advanced Navigation Techniques

#### Using ls Effectively

```bash
# Basic listing
ls
# Output: file1.txt  file2.txt  directory1/

# Detailed listing (-l = long format)
ls -l
# Output:
# drwxr-xr-x 2 user group 4096 Jan 15 10:30 directory1
# -rw-r--r-- 1 user group  123 Jan 15 10:25 file1.txt
#
# Breakdown:
# d         → directory (- for files, l for symlinks)
# rwxr-xr-x → permissions (owner, group, others)
# 2         → number of links
# user      → owner
# group     → group
# 4096      → size in bytes
# Jan 15... → modification time
# directory1→ name
```

**Essential ls Flags:**

```bash
ls -a        # Show hidden files (starting with .)
ls -l        # Long format with details
ls -h        # Human-readable sizes (KB, MB, GB)
ls -t        # Sort by modification time (newest first)
ls -r        # Reverse order
ls -S        # Sort by size
ls -R        # Recursive (show subdirectories)

# Combine flags:
ls -lah      # Long format, all files, human-readable
ls -ltr      # Long format, sorted by time, reversed (oldest first)
ls -lhS      # Long format, human-readable, sorted by size
```

**DevOps Use Case:**

```bash
# Find recently modified configuration files
cd /etc/nginx/
ls -lt | head -10

# Output: Shows 10 most recently modified files
# Why: Quickly identify what changed during troubleshooting
```

#### Tab Completion Mastery

```
┌────────────────────────────────────────────────────┐
│  TAB COMPLETION WORKFLOW                           │
├────────────────────────────────────────────────────┤
│  1. Type first few characters                      │
│  2. Press TAB once    → Auto-complete if unique    │
│  3. Press TAB twice   → Show all possibilities     │
│  4. Type more chars   → Narrow down options        │
│  5. Press TAB again   → Complete                   │
└────────────────────────────────────────────────────┘
```

**Practical Exercise:**

```bash
# Navigate to a directory using only TAB
cd /v[TAB]        # Completes to /var/
cd /var/l[TAB]    # Completes to /var/log/
cd /var/log/a[TAB][TAB]  # Shows all directories starting with 'a'

# Expected output after double TAB:
# alternatives/  apache2/  apt/  auth.log
```

#### Using pushd and popd (Directory Stack)

```bash
# pushd - Push directory onto stack and change to it
# popd  - Pop directory from stack and change to it

# Example workflow:
pwd
# /home/user

pushd /var/log
# /var/log /home/user
# Now in /var/log, /home/user is saved

pushd /etc/nginx
# /etc/nginx /var/log /home/user
# Now in /etc/nginx

popd
# Back to /var/log

popd
# Back to /home/user

# View directory stack:
dirs -v
# Output:
# 0  /home/user
# 1  /var/log
# 2  /etc/nginx
```

**Why This Matters:**
- Navigate between multiple locations without memorizing paths
- Essential for complex deployment workflows
- Faster than repeated `cd` commands

---

## Directory Structure Creation

### Part 1: Understanding Brace Expansion

**Concept:**
```bash
# Brace expansion creates multiple arguments from a pattern
echo {1,2,3}
# Output: 1 2 3

echo {a,b,c}{1,2,3}
# Output: a1 a2 a3 b1 b2 b3 c1 c2 c3
```

**How It Works:**
```
{frontend,backend,database}  →  frontend backend database
{dev,staging,prod}           →  dev staging prod

Nested:
{A,B}/{1,2}  →  A/1 A/2 B/1 B/2
```

### Part 2: Creating Project Structure

#### Target Structure Visualization

```
project/
├── frontend/
│   ├── dev/
│   ├── staging/
│   └── prod/
├── backend/
│   ├── dev/
│   ├── staging/
│   └── prod/
└── database/
    ├── dev/
    ├── staging/
    └── prod/
```

#### Method 1: Single Command (mkdir -p with Brace Expansion)

```bash
# Create the entire structure in ONE command
mkdir -p /project/{frontend,backend,database}/{dev,staging,prod}

# Breakdown:
# mkdir              → make directory command
# -p                 → create parent directories as needed, no error if exists
# /project/          → base path
# {frontend,backend,database}  → expands to 3 directories
# /                  → separator
# {dev,staging,prod} → expands to 3 subdirectories for EACH above
```

**Step-by-Step Expansion:**

```
Step 1: Brace expansion happens BEFORE mkdir runs
{frontend,backend,database}/{dev,staging,prod}

Step 2: Shell expands to:
frontend/dev
frontend/staging
frontend/prod
backend/dev
backend/staging
backend/prod
database/dev
database/staging
database/prod

Step 3: mkdir -p creates all these paths under /project/
```

**Verification:**

```bash
# View the structure
tree /project
# OR if tree isn't installed:
find /project -type d

# Expected output:
# /project
# /project/frontend
# /project/frontend/dev
# /project/frontend/staging
# /project/frontend/prod
# /project/backend
# /project/backend/dev
# /project/backend/staging
# /project/backend/prod
# /project/database
# /project/database/dev
# /project/database/staging
# /project/database/prod
```

#### Method 2: Step-by-Step (Understanding Each Layer)

```bash
# Create base directory
mkdir /project

# Create top-level directories
mkdir /project/frontend /project/backend /project/database

# Create environment subdirectories (method 1: one by one)
mkdir /project/frontend/dev
mkdir /project/frontend/staging
mkdir /project/frontend/prod

# OR (method 2: using brace expansion per directory)
mkdir /project/backend/{dev,staging,prod}

# OR (method 3: all at once)
mkdir /project/database/{dev,staging,prod}
```

#### Method 3: Advanced Patterns

```bash
# More complex DevOps structure
mkdir -p /projects/{web,api,mobile}/{src,tests,docs}/{js,css,assets}

# Creates:
# /projects/web/src/js
# /projects/web/src/css
# /projects/web/src/assets
# /projects/web/tests/js
# ... (27 directories total)

# With specific environment configs
mkdir -p /deploy/{app1,app2,app3}/config/{dev,staging,prod}/{nginx,ssl,env}

# Real-world Kubernetes structure
mkdir -p /k8s/{namespaces,deployments,services,configmaps}/{dev,staging,prod}
```

### Part 3: Common Mistakes and Fixes

```
┌────────────────────────────────────────────────────┐
│  COMMON MISTAKES                                   │
├────────────────────────────────────────────────────┤
│  ❌ mkdir /project/frontend/dev (without -p)       │
│     Error: mkdir: cannot create directory          │
│     '/project/frontend/dev': No such file...       │
│                                                     │
│  ✅ mkdir -p /project/frontend/dev                 │
│     Creates /project, then /frontend, then /dev    │
│                                                     │
│  ❌ mkdir /project/{frontend backend}              │
│     Error: No comma, creates one dir with spaces   │
│                                                     │
│  ✅ mkdir /project/{frontend,backend}              │
│     Comma-separated, no spaces                     │
│                                                     │
│  ❌ Forgetting quotes with spaces:                 │
│     mkdir /project/my directory                    │
│     Creates 'my' and 'directory' separately        │
│                                                     │
│  ✅ mkdir "/project/my directory"                  │
│     OR: mkdir /project/my\ directory               │
└────────────────────────────────────────────────────┘
```

#### Hands-On Exercise 1: Create Complex Structure

```bash
# Task: Create a microservices project structure
# Services: auth, payments, notifications
# Environments: dev, staging, prod
# Each environment needs: config, logs, data

# Solution:
mkdir -p /microservices/{auth,payments,notifications}/{dev,staging,prod}/{config,logs,data}

# Verify:
tree /microservices -L 3

# Expected output: 27 directories organized hierarchically
```

---

## Find Command Mastery

### Part 1: Understanding Find Syntax

```bash
# Basic structure:
find [PATH] [OPTIONS] [TESTS] [ACTIONS]

# Example:
find /var/log -name "*.log" -type f -mtime -7
#    └─────┬─┘ └────┬─────┘ └──┬──┘ └───┬────┘
#      PATH     TEST(name)  TEST  TEST(modified)
```

**Component Breakdown:**

```
PATH        → Where to search (default: current directory)
OPTIONS     → How to search (e.g., -maxdepth, -mindepth)
TESTS       → What to match (e.g., -name, -type, -size)
ACTIONS     → What to do with results (e.g., -print, -exec, -delete)
```

### Part 2: Find by Name

```bash
# Find exact filename
find /home -name "config.yml"
# Output: /home/user/project/config.yml

# Case-insensitive search
find /home -iname "CONFIG.yml"
# Matches: config.yml, CONFIG.YML, Config.Yml

# Wildcard patterns (* = any characters)
find /var/log -name "*.log"
# Matches: error.log, access.log, system.log

find /etc -name "nginx*"
# Matches: nginx.conf, nginx_backup, nginx/

# Multiple patterns (OR logic)
find /home -name "*.js" -o -name "*.ts"
# Matches: app.js OR index.ts
```

**Detailed Example:**

```bash
# Find all Docker-related files
find /etc -name "*docker*"

# Output explanation:
# /etc/docker/               → directory
# /etc/docker/daemon.json    → file
# /etc/systemd/system/docker.service  → service file

# Why this matters:
# Quickly locate all Docker configurations during troubleshooting
```

### Part 3: Find by Type

```bash
# File types:
# f = regular file
# d = directory
# l = symbolic link
# s = socket
# p = named pipe
# b = block device
# c = character device

# Find only directories
find /var -type d -name "log*"
# Output: /var/log, /var/log/nginx, etc.

# Find only files
find /etc -type f -name "*.conf"
# Output: /etc/nginx/nginx.conf, etc.

# Find symbolic links
find /usr/bin -type l
# Output: Lists all symlinked commands
```

**Practical DevOps Example:**

```bash
# Find all broken symbolic links (common issue)
find /etc -type l ! -exec test -e {} \; -print

# Breakdown:
# -type l           → only symbolic links
# ! -exec test -e   → NOT exists test (! means NOT)
# {} \;             → placeholder for found file
# -print            → display result

# Why: Broken symlinks can break application startup
```

### Part 4: Find by Size

```bash
# Size units:
# c = bytes
# k = kilobytes
# M = megabytes
# G = gigabytes

# Find files larger than 100MB
find /var/log -type f -size +100M

# Find files smaller than 1KB
find /tmp -type f -size -1k

# Find files exactly 512 bytes
find /boot -type f -size 512c

# Range: between 1MB and 10MB
find /home -type f -size +1M -size -10M
```

**Real-World Scenario:**

```bash
# Find large log files consuming disk space
find /var/log -type f -size +500M -exec ls -lh {} \;

# Output:
# -rw-r--r-- 1 root root 723M Jan 15 10:30 /var/log/syslog
# -rw-r--r-- 1 root root 612M Jan 14 08:15 /var/log/apache2/access.log

# Action: Identify logs to rotate or compress
```

### Part 5: Find by Modification Time

```bash
# Time units:
# -mtime n  → modified exactly n days ago
# -mtime +n → modified more than n days ago
# -mtime -n → modified less than n days ago (within last n days)

# Modified in last 7 days
find /etc -type f -mtime -7

# Modified more than 30 days ago
find /tmp -type f -mtime +30

# Modified exactly 10 days ago
find /var -type f -mtime 10

# Modified in last 24 hours (use -mmin for minutes)
find /var/log -type f -mmin -1440
# 1440 minutes = 24 hours

# Modified in last hour
find /var/log -type f -mmin -60
```

**Detailed Time Breakdown:**

```
-mtime -1  →  0-24 hours ago (today)
-mtime 0   →  exactly today
-mtime +0  →  more than 24 hours ago (yesterday or earlier)
-mtime -7  →  last 7 days
-mtime 7   →  exactly 7 days ago
-mtime +7  →  more than 7 days ago
```

**DevOps Use Case:**

```bash
# Find recently modified config files (potential recent changes)
find /etc -type f -mtime -1 -name "*.conf"

# Find old temporary files to clean up
find /tmp -type f -mtime +7 -delete
# ⚠️ Warning: -delete is destructive! Test without it first
```

### Part 6: Find by Permissions

```bash
# Permission format: -perm MODE

# Exact permissions (must match exactly)
find /home -type f -perm 0644

# At least these permissions (can have more)
find /usr/bin -type f -perm -u+x  # user executable

# Any of these permissions
find /var -type f -perm /u+w  # user writable

# Security audit: world-writable files (security risk!)
find /etc -type f -perm -002  # writable by others

# SUID files (run with owner's permissions)
find / -type f -perm -4000 -ls 2>/dev/null
```

**Permission Syntax Explained:**

```
0644  → Exact: rw-r--r--
-0644 → At least these bits set
/0644 → Any of these bits set

Common permission values:
644 → rw-r--r-- (regular file)
755 → rwxr-xr-x (executable)
600 → rw------- (private file)
777 → rwxrwxrwx (world writable - dangerous!)
```

**Critical Security Check:**

```bash
# Find files with dangerous permissions
find /var/www -type f -perm -002 -ls

# Why: Web files should NOT be world-writable
# Attackers could modify them

# Find SUID/SGID files (potential privilege escalation)
find / -type f \( -perm -4000 -o -perm -2000 \) -ls 2>/dev/null

# Output shows files that run with elevated privileges
```

### Part 7: Find with Actions

```bash
# -print (default, shows path)
find /etc -name "*.conf" -print

# -ls (detailed listing)
find /var/log -name "*.log" -ls

# -exec (execute command on each result)
find /tmp -name "*.tmp" -exec rm {} \;
#                              └─┬─┘ └┬┘
#                          placeholder  terminator

# -exec with confirmation
find /old-logs -name "*.log" -ok rm {} \;
# Asks for confirmation before each deletion

# -exec with + (runs command once with all results)
find /docs -name "*.txt" -exec grep "error" {} +
# More efficient than running grep separately for each file

# Delete files (USE CAREFULLY!)
find /tmp -type f -mtime +30 -delete
```

**-exec Detailed Breakdown:**

```bash
# Structure:
# -exec COMMAND {} \;
#       └──┬──┘ └┬┘ └┬┘
#       command  placeholder  terminator

# Example: Change ownership of all .log files
find /var/log -name "*.log" -exec chown syslog:adm {} \;

# How it works:
# 1. find locates: error.log
# 2. Executes: chown syslog:adm error.log
# 3. find locates: access.log
# 4. Executes: chown syslog:adm access.log
# (repeats for each file)

# Using + instead of \; (more efficient):
find /var/log -name "*.log" -exec chown syslog:adm {} +

# Executes once: chown syslog:adm error.log access.log system.log ...
```

### Part 8: Combining Multiple Tests

```bash
# AND logic (default, both conditions must be true)
find /var/log -name "*.log" -size +10M
# Files named *.log AND larger than 10MB

# OR logic (use -o)
find /etc -name "*.conf" -o -name "*.cfg"
# Files named *.conf OR *.cfg

# NOT logic (use !)
find /home -type f ! -name "*.txt"
# Files that are NOT .txt

# Complex example: Grouping with parentheses
find /var/log \( -name "*.log" -o -name "*.txt" \) -size +1M
# (*.log OR *.txt) AND larger than 1MB

# Exclude directories
find /var -name "cache" -prune -o -name "*.log" -print
# Find *.log but skip 'cache' directories
```

**Advanced Example:**

```bash
# Find large, old log files not accessed recently
find /var/log \
  -type f \
  -name "*.log" \
  -size +100M \
  -mtime +30 \
  -atime +60 \
  -ls

# Breakdown:
# -type f       → only files
# -name "*.log" → log files
# -size +100M   → larger than 100MB
# -mtime +30    → modified more than 30 days ago
# -atime +60    → accessed more than 60 days ago
# -ls           → detailed listing

# Why: Identify log files safe to archive/delete
```

### Part 9: Find Command Cheat Sheet

```
┌─────────────────────────────────────────────────────────────┐
│  FIND COMMAND QUICK REFERENCE                               │
├─────────────────────────────────────────────────────────────┤
│  BY NAME:                                                   │
│    find /path -name "filename"      → exact match          │
│    find /path -iname "filename"     → case-insensitive     │
│    find /path -name "*.ext"         → wildcard             │
│                                                             │
│  BY TYPE:                                                   │
│    find /path -type f               → files                │
│    find /path -type d               → directories          │
│    find /path -type l               → symbolic links       │
│                                                             │
│  BY SIZE:                                                   │
│    find /path -size +100M           → larger than 100MB    │
│    find /path -size -1k             → smaller than 1KB     │
│    find /path -size 512c            → exactly 512 bytes    │
│                                                             │
│  BY TIME:                                                   │
│    find /path -mtime -7             → modified last 7 days │
│    find /path -mtime +30            → modified >30 days ago│
│    find /path -mmin -60             → modified last hour   │
│                                                             │
│  BY PERMISSIONS:                                            │
│    find /path -perm 0644            → exact permissions    │
│    find /path -perm -002            → world-writable       │
│    find /path -perm /u+x            → user executable      │
│                                                             │
│  ACTIONS:                                                   │
│    find /path -name "x" -print      → show results         │
│    find /path -name "x" -ls         → detailed listing     │
│    find /path -name "x" -exec cmd{}\;→ run command         │
│    find /path -name "x" -delete     → delete (careful!)    │
│                                                             │
│  COMBINING:                                                 │
│    find /path TEST1 TEST2           → AND (both)           │
│    find /path TEST1 -o TEST2        → OR (either)          │
│    find /path ! TEST                → NOT (negate)         │
└─────────────────────────────────────────────────────────────┘
```

---

## Speed Navigation Test

### Objective: Navigate to any system directory in under 5 seconds

#### Setup the Test Environment

```bash
# Create test script
cat > /tmp/nav_test.sh << 'EOF'
#!/bin/bash

# Array of target directories
targets=(
  "/var/log"
  "/etc/nginx"
  "/usr/local/bin"
  "/home/$USER"
  "/tmp"
  "/opt"
  "/var/www"
  "/etc/systemd/system"
)

# Pick random target
target=${targets[$RANDOM % ${#targets[@]}]}

echo "Navigate to: $target"
echo "Starting in: $(pwd)"
echo ""
echo "Press ENTER when in position..."

start=$(date +%s)
read
end=$(date +%s)

# Check if in correct directory
if [ "$(pwd)" == "$target" ]; then
  elapsed=$((end - start))
  echo "✅ SUCCESS in ${elapsed} seconds!"
  
  if [ $elapsed -le 5 ]; then
    echo "🏆 EXCELLENT! Under 5 seconds!"
  elif [ $elapsed -le 10 ]; then
    echo "👍 Good! Keep practicing."
  else
    echo "⏱️  Need more practice."
  fi
else
  echo "❌ Wrong directory. You're in $(pwd)"
  echo "Should be in: $target"
fi
EOF

chmod +x /tmp/nav_test.sh
```

#### Speed Navigation Techniques

**Technique 1: Absolute Paths**
```bash
# Always fastest for known locations
cd /var/log              # 10 keystrokes
cd /etc/nginx            # 13 keystrokes

# Use tab completion:
cd /v[TAB]l[TAB]        # ~8 keystrokes
cd /e[TAB]n[TAB]        # ~8 keystrokes
```

**Technique 2: Environment Variables**
```bash
# Add to ~/.bashrc for frequent locations
export LOGS="/var/log"
export NGINX="/etc/nginx"
export WWW="/var/www/html"

# Usage:
cd $LOGS                 # 8 keystrokes
cd $NGINX                # 9 keystrokes

# Even better with aliases:
alias clog='cd /var/log'
alias cnginx='cd /etc/nginx'

# Usage:
clog                     # 4 keystrokes!
cnginx                   # 6 keystrokes!
```

**Technique 3: CDPATH**
```bash
# Add to ~/.bashrc
export CDPATH=".:~:/var:/etc"

# Now cd searches these paths automatically:
cd log                   # Finds /var/log
cd nginx                 # Finds /etc/nginx

# Without CDPATH:
cd /var/log              # 11 keystrokes

# With CDPATH:
cd log                   # 6 keystrokes (45% faster!)
```

**Technique 4: Bookmarks with bd/bd**
```bash
# Install bd (quickly go back to parent directory)
# Add to ~/.bashrc:
source /usr/share/bd/bd

# Usage when in: /var/www/html/project/src/components
bd html                  # Jump to /var/www/html
bd www                   # Jump to /var/www
bd var                   # Jump to /var
```

**Technique 5: Z / Autojump (Most Powerful)**
```bash
# Install z (tracks frequently visited directories)
# Add to ~/.bashrc:
. /usr/share/z/z.sh

# After visiting directories a few times:
z log                    # Jumps to most frequent match (likely /var/log)
z nginx                  # Jumps to /etc/nginx
z www                    # Jumps to /var/www

# Why so fast:
# - Learns your habits
# - 2-4 keystrokes to any directory
# - Fuzzy matching
```

#### Speed Test Practice Routine

```bash
# Week 1: Basic Speed Drills (5 minutes daily)
cd /; cd /var/log; cd /etc; cd /usr/bin; cd ~; cd /tmp; cd /opt; cd /var/www

# Week 2: Tab Completion Drills (5 minutes daily)
cd /v[TAB]l[TAB]; cd /e[TAB]n[TAB]; cd /u[TAB]b[TAB]

# Week 3: Keyboard Shortcuts (5 minutes daily)
cd /var/log; cd -; cd ~; cd /etc; cd -; cd /tmp; cd -

# Week 4: Full Speed Test
/tmp/nav_test.sh
```

#### Speed Benchmarks

```
┌────────────────────────────────────────────────────┐
│  NAVIGATION SPEED TARGETS                          │
├────────────────────────────────────────────────────┤
│  Beginner (Week 1):                                │
│    → Any directory in 15-20 seconds                │
│                                                     │
│  Intermediate (Week 2):                            │
│    → Common directories in 10 seconds              │
│    → Use tab completion naturally                  │
│                                                     │
│  Advanced (Week 3):                                │
│    → Any directory in 5-8 seconds                  │
│    → Use cd -, pushd/popd effectively              │
│                                                     │
│  Expert (Week 4):                                  │
│    → Any directory in under 5 seconds              │
│    → Use CDPATH, aliases, z/autojump               │
│    → Navigate without looking at keyboard          │
└────────────────────────────────────────────────────┘
```

---

## Muscle Memory Drills

### Drill 1: Directory Navigation Circuit (10 minutes)

```bash
# Set timer for 10 minutes
# Complete circuit as many times as possible

# Round 1: Basic Navigation
cd /
pwd
cd /var
pwd
cd log
pwd
cd /etc
pwd
cd nginx
pwd
cd ~
pwd

# Count completions:
# Beginner: 3-4 rounds
# Intermediate: 5-7 rounds
# Expert: 8+ rounds
```

### Drill 2: Keyboard Shortcuts Circuit (5 minutes)

```bash
# Type this command:
cd /var/log/nginx/error.log.1.gz

# Then perform these actions WITHOUT retyping:
# 1. Ctrl+A → jump to beginning
# 2. Ctrl+E → jump to end
# 3. Ctrl+W → delete last word (.1.gz)
# 4. Ctrl+W → delete next word (error.log)
# 5. Type: access.log
# 6. Press Enter

# Repeat 20 times, building speed
```

### Drill 3: Find Command Speed Drill (10 minutes)

```bash
# Memorize these patterns, then type from memory:

# Round 1:
find /var/log -name "*.log"
find /etc -type d -name "nginx*"
find /home -type f -size +1M

# Round 2:
find /var/log -name "*.log" -mtime -7
find /tmp -type f -size +100M -delete  # Don't press Enter!
find /etc -name "*.conf" -exec grep "error" {} +

# Round 3 (Complex):
find /var/log -type f -name "*.log" -size +10M -mtime +30
find /etc \( -name "*.conf" -o -name "*.cfg" \) -type f
find / -type f -perm -4000 -ls 2>/dev/null

# Goal: Type each without errors within 10 seconds
```

### Drill 4: Directory Creation Speed Drill (5 minutes)

```bash
# Create these structures from memory:

# Round 1:
mkdir -p /tmp/drill1/{a,b,c}/{1,2,3}

# Round 2:
mkdir -p /tmp/drill2/{frontend,backend}/{dev,prod}

# Round 3:
mkdir -p /tmp/drill3/{web,api,db}/{src,tests,docs}/{js,css,html}

# Round 4:
mkdir -p /tmp/drill4/{app{1..5}}/{config,logs,data}/{dev,staging,prod}

# Verify each with: tree /tmp/drill[1-4]
# Goal: Create correctly in under 15 seconds each
```

### Drill 5: Real-World Scenario Simulation (15 minutes)

```bash
# Scenario: Production incident - corrupted log files

# Task 1: Navigate to log directory (target: 3 seconds)
cd /var/log

# Task 2: Find all log files modified today (target: 8 seconds)
find . -name "*.log" -mtime 0

# Task 3: Find large log files (>100MB) (target: 8 seconds)
find . -type f -size +100M

# Task 4: Create backup directory structure (target: 10 seconds)
mkdir -p /var/log/backup/$(date +%Y%m%d)/{nginx,apache,system}

# Task 5: Find and list suspicious files (target: 12 seconds)
find . -type f -name "*.log" -mtime -1 -size +500M -ls

# Total time goal: Under 45 seconds
# Repeat until comfortable
```

---

## Real-World DevOps Use Cases

### Use Case 1: Log Investigation

**Scenario:** Application experiencing errors in production

```bash
# Step 1: Navigate to log directory
cd /var/log/nginx

# Step 2: Find recent error logs
find . -name "*error*" -mtime -1

# Output:
# ./error.log
# ./error.log.1

# Step 3: Check file sizes
ls -lh error.log*
# -rw-r--r-- 1 www-data www-data 234M Jan 15 14:23 error.log
# -rw-r--r-- 1 www-data www-data 156M Jan 14 23:59 error.log.1

# Step 4: Find specific error pattern
find . -name "error.log*" -exec grep -l "500 Internal" {} +

# Step 5: Examine last 100 errors
tail -100 error.log | grep "ERROR"

# Why this workflow:
# - Quickly locate relevant logs
# - Identify which logs contain errors
# - Focus investigation on recent issues
```

### Use Case 2: Configuration Management

**Scenario:** Audit all Nginx configurations across environments

```bash
# Step 1: Create audit directory
mkdir -p ~/audit/nginx/{dev,staging,prod}

# Step 2: Find all Nginx config files
find /etc/nginx -name "*.conf" -type f > ~/audit/config_list.txt

# Step 3: Verify configurations
find /etc/nginx -name "*.conf" -exec nginx -t -c {} \; 2>&1 | grep -v "successful"

# Step 4: Find recently modified configs (potential changes)
find /etc/nginx -name "*.conf" -mtime -7 -ls

# Step 5: Find configs with specific settings
find /etc/nginx -name "*.conf" -exec grep -l "ssl_certificate" {} +

# Output: List of SSL-enabled sites

# Step 6: Check permissions (security audit)
find /etc/nginx -type f ! -perm 0644

# Any output = incorrect permissions!
```

### Use Case 3: Disk Space Management

**Scenario:** Server running out of disk space

```bash
# Step 1: Find directories using most space
du -sh /* 2>/dev/null | sort -rh | head -10

# Output:
# 45G  /var
# 12G  /usr
# 5.2G /home
# ...

# Step 2: Navigate to largest directory
cd /var

# Step 3: Find large files
find . -type f -size +100M -exec ls -lh {} \; | sort -k5 -rh | head -20

# Step 4: Find old log files safe to delete
find /var/log -name "*.log" -mtime +90 -ls

# Step 5: Find compressed logs
find /var/log -name "*.gz" -mtime +60 -printf "%p %s\n" | awk '{total+=$2; print} END {print "Total:", total/1024/1024 "MB"}'

# Step 6: Clean up old temporary files
find /tmp -type f -mtime +30 -user root -ls

# Step 7: Create cleanup script
cat > /tmp/cleanup.sh << 'EOF'
#!/bin/bash
# Dry run - shows what would be deleted
find /var/log -name "*.gz" -mtime +90 -ls
find /tmp -type f -mtime +30 -ls

# Uncomment to actually delete:
# find /var/log -name "*.gz" -mtime +90 -delete
# find /tmp -type f -mtime +30 -delete
EOF
```

### Use Case 4: Security Audit

**Scenario:** Check for security vulnerabilities

```bash
# Step 1: Find world-writable files (major security risk)
find /var/www -type f -perm -002 ! -type l -ls

# Any output = security issue!

# Step 2: Find SUID/SGID files
find / -type f \( -perm -4000 -o -perm -2000 \) -ls 2>/dev/null

# Review list for unauthorized SUID binaries

# Step 3: Find files owned by specific user (after user termination)
find /home -user oldemployee -ls

# Step 4: Find recently modified system files
find /etc -type f -mtime -7 ! -user root -ls

# Non-root modifications to /etc = investigate!

# Step 5: Find files with no owner (orphaned)
find /var/www -nouser -o -nogroup

# Step 6: Find files accessible by group 'www-data' (web server)
find /var/www -group www-data -type f ! -perm 0640 -ls

# Any output = incorrect web permissions

# Step 7: Generate security report
cat > /tmp/security_audit.sh << 'EOF'
#!/bin/bash
echo "=== Security Audit Report ==="
echo "Date: $(date)"
echo ""
echo "World-writable files:"
find /var/www -type f -perm -002 2>/dev/null | wc -l
echo ""
echo "SUID files:"
find / -type f -perm -4000 2>/dev/null | wc -l
echo ""
echo "Files modified in last 24 hours:"
find /etc -type f -mtime -1 2>/dev/null | wc -l
EOF

chmod +x /tmp/security_audit.sh
/tmp/security_audit.sh
```

### Use Case 5: Application Deployment

**Scenario:** Deploy new application version to multiple environments

```bash
# Step 1: Create deployment structure
mkdir -p /deploy/myapp/{dev,staging,prod}/{releases,shared/{logs,config,uploads}}

# Verify structure
tree /deploy/myapp -L 3

# Step 2: Find previous releases
find /deploy/myapp/*/releases -maxdepth 1 -type d -name "20*" | sort

# Output:
# /deploy/myapp/dev/releases/20240115_120000
# /deploy/myapp/staging/releases/20240114_150000
# /deploy/myapp/prod/releases/20240110_100000

# Step 3: Find old releases to clean (keep last 5)
find /deploy/myapp/prod/releases -maxdepth 1 -type d -name "20*" | sort -r | tail -n +6

# Step 4: Create new release directory
release_dir="/deploy/myapp/prod/releases/$(date +%Y%m%d_%H%M%S)"
mkdir -p "$release_dir"

# Step 5: Copy application files
cp -r /tmp/app_build/* "$release_dir/"

# Step 6: Link shared directories
ln -s /deploy/myapp/prod/shared/logs "$release_dir/logs"
ln -s /deploy/myapp/prod/shared/uploads "$release_dir/uploads"

# Step 7: Find and verify environment configs
find "$release_dir" -name "*.env*" -type f

# Step 8: Atomic switchover (symlink)
ln -sfn "$release_dir" /deploy/myapp/prod/current

# Step 9: Verify deployment
ls -la /deploy/myapp/prod/current
```

### Use Case 6: Backup and Recovery

**Scenario:** Backup critical configuration files before maintenance

```bash
# Step 1: Create backup structure
backup_dir="/backup/$(hostname)/$(date +%Y%m%d_%H%M%S)"
mkdir -p "$backup_dir"/{etc,var,home}

# Step 2: Find all config files modified in last 30 days
find /etc -name "*.conf" -mtime -30 -type f > "$backup_dir/config_list.txt"

# Step 3: Copy configs while preserving structure
cd /
find etc -name "*.conf" -mtime -30 -type f -exec cp --parents {} "$backup_dir" \;

# --parents preserves directory structure

# Step 4: Backup application-specific configs
find /opt /var/www -name "config.*" -o -name "*.yml" -o -name "*.yaml" | \
  xargs tar czf "$backup_dir/app_configs.tar.gz"

# Step 5: Find and backup database configs
find /etc -name "my.cnf" -o -name "postgresql.conf" | \
  xargs cp -t "$backup_dir/etc/"

# Step 6: Create backup manifest
cat > "$backup_dir/MANIFEST.txt" << EOF
Backup Date: $(date)
Hostname: $(hostname)
Files backed up:
$(find "$backup_dir" -type f -ls | wc -l) files
$(du -sh "$backup_dir" | cut -f1) total size
EOF

# Step 7: Verify backup integrity
find "$backup_dir" -type f -name "*.conf" | wc -l
# Should match count from config_list.txt
```

---

## Advanced Tips and Tricks

### Tip 1: Command History Search (Ctrl+R)

```bash
# Instead of retyping complex find commands:

# 1. Press Ctrl+R
# 2. Type: find
# 3. Press Ctrl+R again to cycle through previous find commands
# 4. Press Enter to execute or Ctrl+E to edit

# Make it even better - add to ~/.bashrc:
export HISTSIZE=10000
export HISTFILESIZE=20000
export HISTCONTROL=ignoredups:erasedups
shopt -s histappend

# Now you have 10,000 commands in searchable history!
```

### Tip 2: Create Reusable Find Templates

```bash
# Add to ~/.bashrc:

# Find large files
alias findlarge='find . -type f -size +100M -exec ls -lh {} \; | sort -k5 -rh'

# Find recent files
alias findrecent='find . -type f -mtime -7 -ls | sort -k10'

# Find world-writable
alias findww='find . -type f -perm -002 -ls'

# Find by extension
findext() {
  find . -type f -name "*.$1"
}
# Usage: findext log

# Usage examples:
cd /var/log
findlarge        # Shows large log files
findrecent       # Shows recent changes
findext conf     # Shows all .conf files
```

### Tip 3: Combining Find with Parallel Processing

```bash
# Install GNU parallel (optional but powerful)
# Process find results in parallel:

# Convert all .log files to .gz in parallel
find /var/log -name "*.log" -type f | parallel gzip {}

# Check syntax of all Nginx configs in parallel
find /etc/nginx -name "*.conf" | parallel 'nginx -t -c {}'

# Why: 4x faster on quad-core systems!
```

### Tip 4: Find Excluding Directories

```bash
# Exclude node_modules, .git, etc. (huge time saver)
find . -name "*.js" \
  -not -path "*/node_modules/*" \
  -not -path "*/.git/*" \
  -type f

# Better approach: use -prune
find . -type d \( -name node_modules -o -name .git \) -prune -o -name "*.js" -print

# Why -prune is better:
# - Doesn't enter excluded directories at all
# - Much faster on large codebases
```

### Tip 5: Create Custom Keyboard Shortcuts

```bash
# Add to ~/.inputrc (readline configuration)

# Navigate word-by-word with Ctrl+Arrow
"\e[1;5C": forward-word
"\e[1;5D": backward-word

# Clear screen with Ctrl+O
"\C-o": clear-screen

# List directory with Ctrl+L
"\C-l": "ls -la\n"

# After editing, reload:
bind -f ~/.inputrc
```

---

## Common Mistakes and How to Avoid Them

### Mistake 1: Using Find Delete Without Testing

```bash
# ❌ WRONG - Immediately deletes files
find /var/log -name "*.log" -delete

# ✅ CORRECT - Test first, then delete
# Step 1: See what would be deleted
find /var/log -name "*.log" -print

# Step 2: Verify the list
find /var/log -name "*.log" -ls

# Step 3: Only then delete
find /var/log -name "*.log" -delete

# Even better: Create backup first
find /var/log -name "*.log" -exec cp {} /backup/ \; && \
find /var/log -name "*.log" -delete
```

### Mistake 2: Incorrect Brace Expansion

```bash
# ❌ WRONG - Space before comma
mkdir /project/{frontend , backend}
# Creates: /project/{frontend (with space in name!)

# ❌ WRONG - Space after comma
mkdir /project/{frontend, backend}
# Creates: /project/ backend (two directories!)

# ✅ CORRECT - No spaces
mkdir /project/{frontend,backend}
```

### Mistake 3: Forgetting -type with Find

```bash
# ❌ RISKY - Includes directories named *.log
find /var/log -name "*.log" -delete
# Might delete: system.log/ (directory!)

# ✅ SAFE - Only files
find /var/log -type f -name "*.log" -delete
```

### Mistake 4: Wrong Time Syntax

```bash
# ❌ WRONG - Thinks it means "within 7 days"
find /var/log -mtime 7
# Actually means: exactly 7 days ago (7*24 to 8*24 hours ago)

# ✅ CORRECT - Within last 7 days
find /var/log -mtime -7

# ✅ CORRECT - Older than 7 days
find /var/log -mtime +7
```

### Mistake 5: Not Using -print0 with xargs

```bash
# ❌ BREAKS WITH SPACES - Filenames with spaces cause errors
find /var/log -name "*.log" | xargs rm
# Fails on: "error log.txt" (treated as two files)

# ✅ CORRECT - Handles spaces/special characters
find /var/log -name "*.log" -print0 | xargs -0 rm

# Or better, use -exec:
find /var/log -name "*.log" -exec rm {} +
```

---

## Final Mastery Checklist

```
┌─────────────────────────────────────────────────────┐
│  FILESYSTEM NAVIGATION MASTERY CHECKLIST            │
├─────────────────────────────────────────────────────┤
│  LEVEL 1: BEGINNER (Week 1)                         │
│  ☐ Navigate using absolute paths                    │
│  ☐ Use cd, pwd, ls comfortably                      │
│  ☐ Create simple directory structures               │
│  ☐ Find files by name                               │
│  ☐ Know 5 terminal keyboard shortcuts               │
│                                                      │
│  LEVEL 2: INTERMEDIATE (Week 2)                     │
│  ☐ Navigate using tab completion fluently           │
│  ☐ Create nested structures with brace expansion    │
│  ☐ Find files by size, type, time                   │
│  ☐ Use cd -, pushd/popd                             │
│  ☐ Know 10 terminal keyboard shortcuts              │
│                                                      │
│  LEVEL 3: ADVANCED (Week 3)                         │
│  ☐ Navigate to any directory under 5 seconds        │
│  ☐ Complex find with multiple conditions            │
│  ☐ Use find with -exec safely                       │
│  ☐ Set up CDPATH and aliases                        │
│  ☐ Navigate without looking at keyboard             │
│                                                      │
│  LEVEL 4: EXPERT (Week 4)                           │
│  ☐ Solve real-world scenarios in under 60 seconds   │
│  ☐ Create custom find aliases                       │
│  ☐ Use z/autojump for instant navigation            │
│  ☐ Teach these concepts to others                   │
│  ☐ Build automation scripts using find              │
└─────────────────────────────────────────────────────┘
```

## Practice Exam (30 minutes)

```bash
# Complete all tasks, time yourself

# TASK 1 (5 points): Create structure in 30 seconds
mkdir -p /tmp/exam/{web,api,db}/{src,tests,config}/{dev,staging,prod}

# TASK 2 (5 points): Navigate and verify in 20 seconds
cd /tmp/exam/web/src/dev
pwd

# TASK 3 (10 points): Find all directories named 'dev' in 15 seconds
find /tmp/exam -type d -name "dev"

# TASK 4 (10 points): Find files larger than 1MB in /var in 20 seconds
find /var -type f -size +1M 2>/dev/null | head -5

# TASK 5 (15 points): Find .conf files modified in last 7 days in 25 seconds
find /etc -type f -name "*.conf" -mtime -7 2>/dev/null

# TASK 6 (15 points): Create backup of configs in 40 seconds
mkdir -p /tmp/backup
find /etc -name "*.conf" -mtime -7 2>/dev/null -exec cp --parents {} /tmp/backup \;

# TASK 7 (20 points): Security audit in 60 seconds
find /var/www -type f -perm -002 2>/dev/null
find / -type f -perm -4000 2>/dev/null | head -5

# TASK 8 (20 points): Complex find with cleanup in 90 seconds
find /var/log -type f -name "*.log" -size +100M -mtime +30 -ls

# SCORING:
# 90-100: Master level
# 75-89:  Advanced
# 60-74:  Intermediate
# <60:    Need more practice
```

---

## Conclusion and Next Steps

### You've Mastered:
1. ✅ Keyboard-only filesystem navigation
2. ✅ Complex directory structure creation
3. ✅ Advanced find command usage
4. ✅ Speed navigation under 5 seconds
5. ✅ Real-world DevOps scenarios

### Continue Improving:
```bash
# Daily practice routine (10 minutes)
# Day 1-7:   Keyboard shortcuts drill
# Day 8-14:  Speed navigation test
# Day 15-21: Find command variations
# Day 22-28: Real-world scenarios
# Day 29+:   Teach others (best way to solidify knowledge)
```

### Additional Resources:
```bash
# Read manual pages
man find
man bash
man readline

# Practice playgrounds
mkdir ~/devops-practice
# Use this for daily drills

# Track your progress
echo "$(date): Completed navigation drill" >> ~/practice-log.txt
```

**Remember:** Muscle memory takes time. Practice daily for 10-15 minutes for 30 days, and these commands will become second nature.

**Pro tip:** The fastest DevOps engineers don't memorize commands—they build muscle memory through repetition. Start slow, focus on accuracy, speed will follow naturally.