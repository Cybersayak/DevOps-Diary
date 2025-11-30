# Text Processing Warfare: Complete Mastery Guide

## Table of Contents
1. [Foundation Concepts](#foundation-concepts)
2. [Core Tools Deep Dive](#core-tools-deep-dive)
3. [Log File Analysis](#log-file-analysis)
4. [Performance Optimization](#performance-optimization)
5. [Real-World DevOps Scenarios](#real-world-devops-scenarios)
6. [Advanced Techniques](#advanced-techniques)

---

## Foundation Concepts

### Why Text Processing Matters in DevOps

```
┌─────────────────────────────────────────────────────────┐
│  CRITICAL DEVOPS TEXT PROCESSING TASKS:                 │
├─────────────────────────────────────────────────────────┤
│  ✓ Analyze 1GB+ log files in seconds                   │
│  ✓ Extract security threats from access logs           │
│  ✓ Find performance bottlenecks in application logs    │
│  ✓ Generate reports from system metrics                │
│  ✓ Parse API responses and configuration files         │
│  ✓ Monitor real-time log streams                       │
│  ✓ Transform data between formats (JSON, CSV, etc.)    │
│  ✓ Automate log rotation and cleanup                   │
└─────────────────────────────────────────────────────────┘
```

### The Unix Pipeline Philosophy

```
┌────────────────────────────────────────────────────┐
│  UNIX PHILOSOPHY: "Do one thing well"              │
│                                                     │
│  DATA FLOW:                                        │
│  ┌──────┐    ┌──────┐    ┌──────┐    ┌──────┐    │
│  │ grep │ ─► │ awk  │ ─► │ sort │ ─► │ uniq │    │
│  └──────┘    └──────┘    └──────┘    └──────┘    │
│   Filter     Extract     Order      Deduplicate   │
│                                                     │
│  Each tool:                                        │
│  - Reads from stdin (or file)                      │
│  - Processes line by line                          │
│  - Writes to stdout                                │
│  - Can be chained infinitely                       │
└────────────────────────────────────────────────────┘
```

### Text Processing Command Hierarchy

```
BASIC LEVEL:
├── cat     → Concatenate and display files
├── head    → Show first N lines
├── tail    → Show last N lines
├── wc      → Count words, lines, bytes
└── cut     → Extract columns/fields

INTERMEDIATE:
├── grep    → Search patterns (regex)
├── sort    → Order lines
├── uniq    → Remove duplicates
└── tr      → Translate/delete characters

ADVANCED:
├── sed     → Stream editor (find/replace)
├── awk     → Pattern scanning/processing language
├── perl    → One-liners for complex text manipulation
└── jq      → JSON processor

PERFORMANCE:
├── parallel → Execute commands in parallel
├── xargs   → Build and execute command lines
└── ripgrep → Ultra-fast grep alternative
```

---

## Core Tools Deep Dive

### Part 1: grep - Pattern Searching

#### Understanding grep

```bash
# Basic syntax:
grep [OPTIONS] PATTERN [FILE]

# How grep works:
# 1. Read file line by line
# 2. Check if line matches PATTERN
# 3. If match, output line
# 4. Continue to next line

# Example:
echo -e "apple\nbanana\napricot" | grep "ap"
# Output:
# apple
# apricot
# (Lines containing "ap")
```

#### Essential grep Flags

```bash
# -i: Ignore case
grep -i "error" /var/log/syslog
# Matches: error, ERROR, Error, eRRor

# -v: Invert match (show lines that DON'T match)
grep -v "^#" /etc/nginx/nginx.conf
# Shows non-comment lines

# -c: Count matching lines
grep -c "404" /var/log/nginx/access.log
# Output: 1523 (number of 404 errors)

# -n: Show line numbers
grep -n "error" app.log
# Output: 42:Fatal error occurred

# -A: Show N lines AFTER match
grep -A 3 "error" app.log
# Shows error line + 3 lines after

# -B: Show N lines BEFORE match
grep -B 2 "error" app.log
# Shows 2 lines before + error line

# -C: Show N lines of CONTEXT (before and after)
grep -C 2 "error" app.log
# Shows 2 before + error + 2 after

# -r: Recursive (search directories)
grep -r "TODO" /var/www/
# Searches all files recursively

# -l: Show only filenames with matches
grep -l "error" /var/log/*.log
# Output: /var/log/app.log

# -h: Suppress filename in output
grep -h "error" /var/log/*.log
# Just shows matching lines

# -w: Match whole words only
grep -w "error" app.log
# Matches "error" but NOT "errors" or "terror"

# -o: Show only matching part
echo "IP: 192.168.1.1" | grep -o "[0-9]\+\.[0-9]\+\.[0-9]\+\.[0-9]\+"
# Output: 192.168.1.1

# -E: Extended regex (grep -E = egrep)
grep -E "(error|warning|critical)" app.log
# Matches any of these patterns

# -P: Perl-compatible regex (PCRE)
grep -P "\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}" access.log
# Matches IP addresses using \d shorthand
```

#### grep Performance Tips

```bash
# ❌ SLOW: Process entire file
grep "error" huge-file.log

# ✅ FASTER: Stop after N matches
grep -m 100 "error" huge-file.log
# Stops after finding 100 matches

# ✅ FASTER: Use fixed strings (not regex)
grep -F "exact.string.to.find" huge-file.log
# -F = fixed string (faster than regex)

# ✅ FASTER: Specify file encoding
LANG=C grep "pattern" huge-file.log
# C locale is faster than UTF-8

# ✅ FASTEST: Use ripgrep (if available)
rg "pattern" huge-file.log
# Modern grep alternative, much faster
```

#### Practical grep Examples

```bash
# Find all IP addresses
grep -Eo "([0-9]{1,3}\.){3}[0-9]{1,3}" access.log

# Find failed login attempts
grep "Failed password" /var/log/auth.log

# Find errors but exclude known issues
grep "ERROR" app.log | grep -v "KNOWN_ISSUE"

# Find lines with email addresses
grep -E "\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b" file.txt

# Find all TODO comments in code
grep -rn "TODO\|FIXME\|XXX" /path/to/code/

# Case-insensitive search for multiple patterns
grep -iE "error|warning|fail" /var/log/syslog
```

### Part 2: awk - Text Processing Language

#### Understanding awk

```
┌────────────────────────────────────────────────────┐
│  AWK PROCESSING MODEL:                             │
│                                                     │
│  Input: "field1 field2 field3"                     │
│          └─┬──┘ └─┬──┘ └─┬──┘                     │
│            $1     $2     $3                        │
│          └──────┬─────────┘                        │
│                 $0 (entire line)                   │
│                                                     │
│  awk '{print $1}'                                  │
│       └──┬────┘                                    │
│      Action (what to do)                           │
│                                                     │
│  awk '/pattern/ {print $1}'                        │
│       └───┬───┘                                    │
│      Pattern (when to do it)                       │
└────────────────────────────────────────────────────┘
```

#### awk Basics

```bash
# Print entire line (like cat)
awk '{print}' file.txt
# OR
awk '{print $0}' file.txt

# Print first field (column)
echo "alice 25 developer" | awk '{print $1}'
# Output: alice

# Print multiple fields
echo "alice 25 developer" | awk '{print $1, $3}'
# Output: alice developer
# (Note: comma adds space)

# Print with custom separator
echo "alice 25 developer" | awk '{print $1 " is a " $3}'
# Output: alice is a developer

# Field separator (default: whitespace)
echo "alice,25,developer" | awk -F',' '{print $1}'
# Output: alice
# -F',' sets field separator to comma

# Multiple character separator
echo "alice::25::developer" | awk -F'::' '{print $1}'
# Output: alice

# Print specific fields in order
awk '{print $3, $1, $2}' file.txt
# Reorders columns
```

#### awk Built-in Variables

```bash
# NR: Number of Records (current line number)
awk '{print NR, $0}' file.txt
# Output:
# 1 first line
# 2 second line
# 3 third line

# NF: Number of Fields (columns in current line)
echo "a b c" | awk '{print NF}'
# Output: 3

# Print last field (regardless of count)
echo "a b c d e" | awk '{print $NF}'
# Output: e

# Print second-to-last field
echo "a b c d e" | awk '{print $(NF-1)}'
# Output: d

# FS: Field Separator (input)
awk 'BEGIN {FS=":"} {print $1}' /etc/passwd
# Sets FS to colon

# OFS: Output Field Separator
echo "a b c" | awk 'BEGIN {OFS="-"} {print $1, $2, $3}'
# Output: a-b-c

# RS: Record Separator (input, default: newline)
# ORS: Output Record Separator

# FILENAME: Current filename being processed
awk '{print FILENAME, $0}' file1.txt file2.txt
```

#### awk Patterns and Conditions

```bash
# Pattern matching
awk '/error/ {print}' app.log
# Prints lines containing "error"

# Pattern with action
awk '/error/ {print $1, $2}' app.log
# Prints first two fields of lines containing "error"

# Numeric comparison
awk '$3 > 100 {print}' file.txt
# Prints lines where 3rd field > 100

# String comparison
awk '$1 == "alice" {print}' users.txt
# Prints lines where 1st field equals "alice"

# Multiple conditions (AND)
awk '$3 > 100 && $1 == "alice" {print}' file.txt

# Multiple conditions (OR)
awk '$3 > 100 || $1 == "alice" {print}' file.txt

# Regex match
awk '$1 ~ /^a/ {print}' file.txt
# Prints lines where 1st field starts with 'a'

# Regex NOT match
awk '$1 !~ /^a/ {print}' file.txt
```

#### awk BEGIN and END

```bash
# BEGIN: Execute before processing any lines
awk 'BEGIN {print "Starting processing..."} {print $1}' file.txt
# Output:
# Starting processing...
# (then first field of each line)

# END: Execute after processing all lines
awk '{sum += $1} END {print "Total:", sum}' numbers.txt
# Sums first column and prints total at end

# Combine BEGIN, pattern, and END
awk 'BEGIN {count=0} /error/ {count++} END {print count}' app.log
# Counts lines containing "error"

# BEGIN with variable initialization
awk 'BEGIN {FS=":"; OFS="\t"} {print $1, $3}' /etc/passwd
# Sets input separator to : and output to tab
```

#### awk Advanced Examples

```bash
# Calculate average
awk '{sum += $1; count++} END {print sum/count}' numbers.txt

# Print unique values (like uniq)
awk '!seen[$1]++' file.txt
# Prints first occurrence of each value in field 1

# Sum column
awk '{sum += $3} END {print sum}' data.txt

# Count occurrences
awk '{count[$1]++} END {for (word in count) print word, count[word]}' file.txt

# Filter by range
awk 'NR >= 10 && NR <= 20' file.txt
# Prints lines 10-20

# Print lines longer than 80 characters
awk 'length > 80' file.txt

# Remove duplicate lines (consecutive)
awk 'a !~ $0; {a=$0}' file.txt

# Pretty print with formatting
awk '{printf "%-10s %5d\n", $1, $2}' file.txt
# Left-align field 1 (10 chars), right-align field 2 (5 digits)
```

### Part 3: sed - Stream Editor

#### Understanding sed

```
┌────────────────────────────────────────────────────┐
│  SED COMMAND STRUCTURE:                            │
│                                                     │
│  sed 'COMMAND' file.txt                            │
│      └───┬───┘                                     │
│      One or more commands                          │
│                                                     │
│  Common commands:                                  │
│  s/pattern/replacement/  → Substitute              │
│  d                       → Delete                  │
│  p                       → Print                   │
│  a\                      → Append                  │
│  i\                      → Insert                  │
└────────────────────────────────────────────────────┘
```

#### sed Substitution (s//)

```bash
# Basic substitution
echo "hello world" | sed 's/world/universe/'
# Output: hello universe

# Replace first occurrence on each line
sed 's/error/ERROR/' app.log
# Changes first "error" per line to "ERROR"

# Global replacement (all occurrences)
sed 's/error/ERROR/g' app.log
# Changes ALL "error" to "ERROR"

# Case-insensitive replacement
sed 's/error/ERROR/gi' app.log
# OR
sed 's/error/ERROR/gI' app.log

# Replace only Nth occurrence
echo "a b a b a" | sed 's/a/X/2'
# Output: a b X b a
# (Replaces 2nd occurrence)

# Replace from Nth occurrence onward
echo "a b a b a" | sed 's/a/X/g2'
# Output: a b X b X
# (Replaces 2nd and subsequent)

# In-place editing (modify file directly)
sed -i 's/old/new/g' file.txt
# -i = in-place (overwrites file)

# In-place with backup
sed -i.bak 's/old/new/g' file.txt
# Creates file.txt.bak before modifying

# Use different delimiter (useful for paths)
sed 's|/old/path|/new/path|g' file.txt
# Using | instead of / (easier with paths)

# Reference matched pattern with &
echo "hello" | sed 's/hello/& world/'
# Output: hello world
# (& represents matched text)

# Capture groups
echo "2024-01-15" | sed 's/\([0-9]\{4\}\)-\([0-9]\{2\}\)-\([0-9]\{2\}\)/\3\/\2\/\1/'
# Output: 15/01/2024
# \1, \2, \3 reference captured groups
```

#### sed Deletion (d)

```bash
# Delete specific line
sed '3d' file.txt
# Deletes line 3

# Delete range of lines
sed '2,5d' file.txt
# Deletes lines 2-5

# Delete last line
sed '$d' file.txt
# $ = last line

# Delete lines matching pattern
sed '/error/d' app.log
# Deletes all lines containing "error"

# Delete lines NOT matching pattern
sed '/error/!d' app.log
# Deletes lines that DON'T contain "error"
# (effectively, keeps only error lines)

# Delete empty lines
sed '/^$/d' file.txt

# Delete lines with only whitespace
sed '/^[[:space:]]*$/d' file.txt

# Delete comments
sed '/^#/d' config.file
```

#### sed Print (p)

```bash
# Print specific line
sed -n '5p' file.txt
# -n = suppress default output
# p = print
# Result: Only line 5 is printed

# Print range
sed -n '10,20p' file.txt
# Prints lines 10-20

# Print lines matching pattern
sed -n '/error/p' app.log
# Like grep

# Print first match and quit
sed -n '/error/p; /error/q' app.log
# Prints first error line, then quits (fast!)
```

#### sed Advanced Examples

```bash
# Add line before match
sed '/pattern/i\New line here' file.txt
# i\ = insert before

# Add line after match
sed '/pattern/a\New line here' file.txt
# a\ = append after

# Change entire line
sed '/pattern/c\Replacement line' file.txt
# c\ = change entire line

# Multiple commands
sed -e 's/foo/bar/' -e 's/baz/qux/' file.txt
# OR
sed 's/foo/bar/; s/baz/qux/' file.txt

# Apply only to lines matching pattern
sed '/error/ s/fatal/critical/' app.log
# Replaces "fatal" with "critical" only on lines containing "error"

# Print line numbers
sed '=' file.txt
# Prints line number before each line

# Remove HTML tags
sed 's/<[^>]*>//g' file.html

# Extract IP addresses
sed -n 's/.*\([0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\).*/\1/p' file.txt

# Convert Windows line endings to Unix
sed 's/\r$//' file.txt

# Add prefix to each line
sed 's/^/PREFIX: /' file.txt

# Add suffix to each line
sed 's/$/ SUFFIX/' file.txt

# Remove leading whitespace
sed 's/^[[:space:]]*//' file.txt

# Remove trailing whitespace
sed 's/[[:space:]]*$//' file.txt

# Double-space file
sed 'G' file.txt
# G appends newline after each line
```

### Part 4: sort - Ordering Lines

#### Basic Sorting

```bash
# Alphabetical sort
sort file.txt
# Default: lexicographic (dictionary) order

# Reverse sort
sort -r file.txt

# Numeric sort
sort -n numbers.txt
# Treats content as numbers: 2 comes before 10

# Without -n (lexicographic):
# 10
# 2
# With -n (numeric):
# 2
# 10

# Sort by specific field (column)
sort -k 2 file.txt
# Sorts by 2nd field

# Sort by multiple fields
sort -k 1,1 -k 2,2n file.txt
# Sort by field 1 (text), then field 2 (numeric)

# Sort with custom delimiter
sort -t: -k3 -n /etc/passwd
# -t: sets field separator to colon
# -k3 sorts by 3rd field (UID)
# -n numeric sort

# Case-insensitive sort
sort -f file.txt
# OR
sort --ignore-case file.txt

# Sort and remove duplicates
sort -u file.txt
# Equivalent to: sort file.txt | uniq

# Human-numeric sort (1K, 2M, 3G)
du -h | sort -h
# Understands K, M, G suffixes

# Random sort (shuffle)
sort -R file.txt

# Check if file is sorted
sort -c file.txt
# Outputs error if not sorted

# Stable sort (preserve order of equal elements)
sort -s file.txt
```

#### Advanced Sorting

```bash
# Sort by IP address (version sort)
sort -V ip-list.txt
# Correctly sorts: 1.2.3.4, 1.2.3.10, 1.2.3.100

# Sort by month
sort -M dates.txt
# Recognizes: Jan, Feb, Mar, etc.

# Sort by multiple columns with different types
sort -k1,1 -k2,2n -k3,3r file.txt
# Column 1: text ascending
# Column 2: numeric ascending
# Column 3: reverse

# Sort large files (use temporary directory)
sort -T /tmp/sort --parallel=4 huge-file.txt
# -T specifies temp directory
# --parallel uses 4 CPU cores

# Sort by field range
sort -k2,4 file.txt
# Sorts by fields 2, 3, 4 combined

# Ignore leading blanks
sort -b file.txt

# Sort by specific character position
sort -k1.5,1.7 file.txt
# Sort by characters 5-7 of field 1
```

### Part 5: uniq - Remove Duplicates

#### Basic Usage

```bash
# Remove duplicate consecutive lines
uniq file.txt
# Note: Only removes CONSECUTIVE duplicates
# File must be sorted first!

# Proper duplicate removal:
sort file.txt | uniq

# Count occurrences
sort file.txt | uniq -c
# Output:
#   3 apple
#   2 banana
#   1 cherry

# Show only duplicates
sort file.txt | uniq -d

# Show only unique lines (non-duplicates)
sort file.txt | uniq -u

# Case-insensitive
sort file.txt | uniq -i

# Ignore first N fields
uniq -f 2 file.txt
# Skips first 2 fields when comparing

# Ignore first N characters
uniq -s 5 file.txt
# Skips first 5 characters

# Compare only first N characters
uniq -w 10 file.txt
# Compares only first 10 characters
```

#### Practical Examples

```bash
# Count unique IP addresses
awk '{print $1}' access.log | sort | uniq -c | sort -rn
# Result:
#  1523 192.168.1.1
#   842 192.168.1.2
#   521 192.168.1.3

# Find most common user agents
awk -F'"' '{print $6}' access.log | sort | uniq -c | sort -rn | head

# Find duplicate entries
sort file.txt | uniq -d

# Remove duplicates preserving order (using awk instead)
awk '!seen[$0]++' file.txt
# Better than sort | uniq when order matters
```

### Part 6: head and tail - File Preview

```bash
# First 10 lines (default)
head file.txt

# First N lines
head -n 20 file.txt
# OR
head -20 file.txt

# First N bytes
head -c 100 file.txt

# Multiple files
head file1.txt file2.txt
# Shows headers: ==> file1.txt <==

# Suppress headers
head -q file1.txt file2.txt

# Last 10 lines (default)
tail file.txt

# Last N lines
tail -n 20 file.txt
# OR
tail -20 file.txt

# Last N bytes
tail -c 100 file.txt

# Follow file (real-time monitoring)
tail -f /var/log/syslog
# Updates as file grows
# Ctrl+C to stop

# Follow with retry (keeps trying if file doesn't exist)
tail -F /var/log/app.log
# Useful for log rotation

# Follow multiple files
tail -f file1.log file2.log

# Show last N lines and follow
tail -n 50 -f app.log

# All lines except first N
tail -n +10 file.txt
# Skips first 9 lines, shows from line 10 onward

# All lines except last N
head -n -10 file.txt
# Shows all except last 10 lines
```

---

## Log File Analysis

### Part 1: Understanding Log Formats

#### Apache/Nginx Combined Log Format

```
Format:
IP - - [timestamp] "METHOD /path HTTP/1.1" status size "referrer" "user-agent"

Example:
192.168.1.1 - - [15/Jan/2024:10:30:45 +0000] "GET /index.html HTTP/1.1" 200 1234 "https://google.com" "Mozilla/5.0..."

Fields:
$1 = 192.168.1.1              (IP address)
$4 = [15/Jan/2024:10:30:45    (Timestamp)
$6 = GET                       (HTTP method)
$7 = /index.html              (Request path)
$9 = 200                       (Status code)
$10 = 1234                     (Response size)
$11 = "https://google.com"    (Referrer)
$12 = "Mozilla/5.0..."        (User agent)
```

#### Apache Error Log Format

```
Format:
[timestamp] [level] [pid PID] [client IP] message

Example:
[Mon Jan 15 10:30:45.123456 2024] [error] [pid 12345] [client 192.168.1.1:56789] File does not exist: /var/www/html/missing.html

Fields:
[Mon Jan 15 10:30:45.123456 2024]  (Timestamp)
[error]                             (Log level)
[pid 12345]                         (Process ID)
[client 192.168.1.1:56789]         (Client IP:port)
File does not exist...              (Error message)
```

#### Syslog Format

```
Format:
timestamp hostname service[PID]: message

Example:
Jan 15 10:30:45 server1 sshd[12345]: Failed password for root from 192.168.1.100 port 22 ssh2

Fields:
Jan 15 10:30:45    (Timestamp)
server1            (Hostname)
sshd               (Service)
[12345]            (PID)
Failed password... (Message)
```

### Part 2: Downloading Sample Log Files

```bash
# Create working directory
mkdir -p ~/log-analysis
cd ~/log-analysis

# Download sample Apache access log
wget https://www.almhuette-raith.at/apache-log/access.log
# OR generate synthetic log:
cat > access.log << 'EOF'
192.168.1.1 - - [15/Jan/2024:10:30:45 +0000] "GET /index.html HTTP/1.1" 200 1234 "-" "Mozilla/5.0"
192.168.1.2 - - [15/Jan/2024:10:30:46 +0000] "GET /about.html HTTP/1.1" 200 2345 "-" "Mozilla/5.0"
192.168.1.1 - - [15/Jan/2024:10:30:47 +0000] "POST /api/login HTTP/1.1" 404 0 "-" "curl/7.68.0"
192.168.1.3 - - [15/Jan/2024:10:30:48 +0000] "GET /static/style.css HTTP/1.1" 200 5678 "-" "Mozilla/5.0"
192.168.1.1 - - [15/Jan/2024:10:30:49 +0000] "GET /admin HTTP/1.1" 403 0 "-" "Mozilla/5.0"
EOF

# Generate large log file for performance testing
for i in {1..1000000}; do
    echo "192.168.$((RANDOM%256)).$((RANDOM%256)) - - [15/Jan/2024:10:30:45 +0000] \"GET /page$((RANDOM%100)).html HTTP/1.1\" $((RANDOM%3 == 0 ? 404 : 200)) $((RANDOM*100)) \"-\" \"Mozilla/5.0\""
done > large-access.log

# Check file size
ls -lh large-access.log
du -h large-access.log
```

### Part 3: Top 20 IP Addresses (< 30 seconds)

#### Method 1: Classic Pipeline (Most Common)

```bash
# Complete command:
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -20

# Breakdown step by step:

# Step 1: Extract IP addresses (field 1)
awk '{print $1}' access.log
# Output:
# 192.168.1.1
# 192.168.1.2
# 192.168.1.1
# 192.168.1.3
# 192.168.1.1

# Step 2: Sort (required for uniq)
awk '{print $1}' access.log | sort
# Output:
# 192.168.1.1
# 192.168.1.1
# 192.168.1.1
# 192.168.1.2
# 192.168.1.3

# Step 3: Count occurrences
awk '{print $1}' access.log | sort | uniq -c
# Output:
#   3 192.168.1.1
#   1 192.168.1.2
#   1 192.168.1.3

# Step 4: Sort by count (reverse numeric)
awk '{print $1}' access.log | sort | uniq -c | sort -rn
# Output:
#   3 192.168.1.1
#   1 192.168.1.2
#   1 192.168.1.3

# Step 5: Show top 20
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -20
```

**Why This Works:**
- `awk '{print $1}'`: Extracts first field (IP) - O(n)
- `sort`: Orders identical IPs together - O(n log n)
- `uniq -c`: Counts consecutive duplicates - O(n)
- `sort -rn`: Orders by count descending - O(n log n)
- `head -20`: Takes top 20 - O(1)

**Total complexity:** O(n log n) - very efficient!

#### Method 2: Faster with AWK Only

```bash
# Single awk command (faster for large files)
awk '{count[$1]++} END {for (ip in count) print count[ip], ip}' access.log | sort -rn | head -20

# Breakdown:

# awk '{count[$1]++}': Create associative array with IP counts
# count["192.168.1.1"] = 3
# count["192.168.1.2"] = 1

# END {for (ip in count) print count[ip], ip}: Print all counts
# 3 192.168.1.1
# 1 192.168.1.2

# sort -rn | head -20: Sort and show top 20

# Why faster?
# - Single pass through file (awk does counting)
# - No intermediate sort before uniq
# - Less data passed through pipe
```

#### Method 3: Cut (Alternative)

```bash
cut -d' ' -f1 access.log | sort | uniq -c | sort -rn | head -20

# cut -d' ' -f1: Extract field 1, space-delimited
# Simpler than awk for simple field extraction
```

#### Method 4: Optimized for 1GB+ Files

```bash
# Use parallel processing
cat large-access.log | parallel --pipe --block 100M "awk '{print \$1}'" | sort | uniq -c | sort -rn | head -20

# Explanation:
# parallel --pipe: Feed data to parallel
# --block 100M: Process in 100MB chunks
# awk in parallel: Multiple processes extract IPs
# Final sort | uniq -c | sort -rn: Combine results

# Alternative: Use grep with fixed strings
LC_ALL=C grep -oP '^\S+' large-access.log | sort | uniq -c | sort -rn | head -20
# LC_ALL=C: Use C locale (faster)
# grep -oP: Perl regex, only matching part
# ^\S+: Match non-whitespace at start (IP)
```

#### Performance Comparison

```bash
# Benchmark different methods
time awk '{print $1}' large-access.log | sort | uniq -c | sort -rn | head -20
time awk '{count[$1]++} END {for (ip in count) print count[ip], ip}' large-access.log | sort -rn | head -20
time cut -d' ' -f1 large-access.log | sort | uniq -c | sort -rn | head -20

# Expected results (1GB file):
# Method 1: ~15 seconds
# Method 2: ~8 seconds  ← Fastest
# Method 3: ~17 seconds
```

### Part 4: Complex Log Parsing Examples

#### Example 1: Extract All URLs Accessed

```bash
# Extract request path (field 7)
awk '{print $7}' access.log | sort | uniq -c | sort -rn | head -20

# Output:
#  1523 /index.html
#   842 /about.html
#   521 /contact.html

# More precise (handle quoted fields):
awk -F'"' '{print $2}' access.log | awk '{print $2}' | sort | uniq -c | sort -rn | head -20

# Breakdown:
# -F'"': Split by double quotes
# {print $2}: Get second field (the HTTP request)
# awk '{print $2}': Get second word (the URL path)
```

#### Example 2: Count HTTP Status Codes

```bash
awk '{print $9}' access.log | sort | uniq -c | sort -rn

# Output:
#  8234 200
#  1523 404
#   521 500
#   103 403

# With percentages:
awk '{status[$9]++; total++} END {for (code in status) printf "%s: %d (%.2f%%)\n", code, status[code], (status[code]/total)*100}' access.log | sort -k2 -rn

# Output:
# 200: 8234 (82.34%)
# 404: 1523 (15.23%)
# 500: 521 (5.21%)
```

#### Example 3: Requests by Hour

```bash
awk '{print $4}' access.log | awk -F: '{print $2}' | sort | uniq -c

# Breakdown:
# {print $4}: Extract timestamp field [15/Jan/2024:10:30:45
# -F:: Split by colon
# {print $2}: Get hour (10)

# Output:
#  523 08
# 1234 09
# 2345 10
# 1523 11

# Better formatting:
awk '{print $4}' access.log | awk -F: '{print $2":00"}' | sort | uniq -c | sort -k2

# Output:
#  523 08:00
# 1234 09:00
# 2345 10:00
```

#### Example 4: Find 404 Errors with IPs

```bash
awk '$9 == 404 {print $1, $7}' access.log | sort | uniq -c | sort -rn | head -20

# $9 == 404: Filter status code 404
# {print $1, $7}: Print IP and path
# Result: Most common 404s by IP

# Example output:
#  25 192.168.1.100 /admin
#  18 192.168.1.101 /wp-admin
#  12 192.168.1.102 /phpmyadmin
```

#### Example 5: Top User Agents

```bash
awk -F'"' '{print $6}' access.log | sort | uniq -c | sort -rn | head -10

# -F'"': Split by quotes
# $6: User agent field (after 3 quote pairs)

# Output:
# 5234 Mozilla/5.0 (Windows NT 10.0; Win64; x64)...
# 2345 Mozilla/5.0 (iPhone; CPU iPhone OS 14_0)...
# 1234 curl/7.68.0

# Simplified user agents:
awk -F'"' '{print $6}' access.log | sed 's/(.*//' | sort | uniq -c | sort -rn | head -10

# sed 's/(.*//'': Remove everything after first (
# Result: Cleaner grouping
```

#### Example 6: Bandwidth Usage by IP

```bash
awk '{ip_bytes[$1]+=$10} END {for (ip in ip_bytes) print ip, ip_bytes[ip]}' access.log | sort -k2 -rn | head -20

# {ip_bytes[$1]+=$10}: Sum bytes (field 10) per IP
# END: Print totals
# sort -k2 -rn: Sort by bytes descending

# Human-readable bytes:
awk '{ip_bytes[$1]+=$10} END {for (ip in ip_bytes) printf "%s\t%.2f MB\n", ip, ip_bytes[ip]/1024/1024}' access.log | sort -k2 -rn | head -20
```

#### Example 7: Detect Potential Attacks

```bash
# Rapid requests from single IP (> 100 requests)
awk '{count[$1]++} END {for (ip in count) if (count[ip] > 100) print count[ip], ip}' access.log | sort -rn

# Scanning for common vulnerable paths
grep -E "(admin|phpmyadmin|wp-admin|xmlrpc)" access.log | awk '{print $1}' | sort | uniq -c | sort -rn

# SQL injection attempts
grep -iE "(\%27|'|--|\%23|#)" access.log | awk '{print $1, $7}'

# Failed authentication attempts
awk '$9 == 401 || $9 == 403 {print $1, $7}' access.log | sort | uniq -c | sort -rn
```

#### Example 8: Response Time Analysis (Custom Log Format)

```bash
# If log includes response time (microseconds) at end
# Format: ... 12345
# Extract and calculate average

awk '{sum+=$NF; count++} END {print "Average:", sum/count/1000, "ms"}' access.log

# Find slowest requests
awk '{print $NF, $7}' access.log | sort -rn | head -20

# Requests taking > 1 second
awk '$NF > 1000000 {print $NF/1000, "ms", $7}' access.log | sort -rn
```

### Part 5: Multi-Format Log Parsing

#### Parsing Syslog

```bash
# Extract failed SSH login attempts
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn

# Breakdown:
# grep "Failed password": Filter relevant lines
# $(NF-3): Get 4th field from end (IP address)

# Example line:
# Jan 15 10:30:45 server sshd[12345]: Failed password for root from 192.168.1.100 port 22 ssh2
#                                                                      └─── $(NF-3)

# Alternative (more precise):
grep "Failed password" /var/log/auth.log | awk '{for(i=1;i<=NF;i++) if($i=="from") print $(i+1)}' | sort | uniq -c | sort -rn
```

#### Parsing JSON Logs

```bash
# Sample JSON log:
# {"timestamp":"2024-01-15T10:30:45Z","level":"error","ip":"192.168.1.1","message":"Connection failed"}

# Using jq (JSON processor):
jq -r '.ip' app.json | sort | uniq -c | sort -rn

# Without jq (grep approach):
grep -oP '"ip":"[^"]*"' app.json | cut -d'"' -f4 | sort | uniq -c | sort -rn

# Extract error messages:
jq -r 'select(.level=="error") | .message' app.json

# Count by error level:
jq -r '.level' app.json | sort | uniq -c
```

#### Parsing CSV Logs

```bash
# Sample CSV:
# timestamp,ip,status,bytes
# 2024-01-15 10:30:45,192.168.1.1,200,1234

# Top IPs (field 2):
awk -F, '{print $2}' access.csv | tail -n +2 | sort | uniq -c | sort -rn | head -20
# tail -n +2: Skip header line

# Status code distribution:
awk -F, 'NR>1 {print $3}' access.csv | sort | uniq -c

# Total bandwidth:
awk -F, 'NR>1 {sum+=$4} END {print sum/1024/1024, "MB"}' access.csv
```

---

## Performance Optimization

### Part 1: Bottleneck Identification

```bash
# Profile command execution
time command

# Detailed timing:
/usr/bin/time -v command
# Shows:
# - User time
# - System time
# - Memory usage
# - I/O statistics

# Example:
/usr/bin/time -v awk '{print $1}' large-access.log | sort | uniq -c | sort -rn | head -20

# Identify slow component:
time awk '{print $1}' large-access.log > /tmp/ips.txt
time sort /tmp/ips.txt > /tmp/ips-sorted.txt
time uniq -c /tmp/ips-sorted.txt > /tmp/ips-counted.txt
time sort -rn /tmp/ips-counted.txt > /tmp/ips-final.txt
```

### Part 2: Optimization Techniques

#### Technique 1: Use C Locale

```bash
# Default (slow):
time grep "pattern" large-file.log

# With C locale (faster):
time LC_ALL=C grep "pattern" large-file.log

# Why faster?
# - Skips UTF-8 character handling
# - Treats data as simple bytes
# - 2-3x speed improvement

# Apply to entire pipeline:
LC_ALL=C awk '{print $1}' large-access.log | LC_ALL=C sort | LC_ALL=C uniq -c | LC_ALL=C sort -rn | head -20
```

#### Technique 2: Parallel Processing

```bash
# Install GNU parallel:
sudo apt-get install parallel

# Split work across CPU cores:
cat large-access.log | parallel --pipe --block 100M "awk '{print \$1}'" | sort | uniq -c | sort -rn | head -20

# Explanation:
# --pipe: Pipe input to parallel
# --block 100M: Process 100MB chunks
# "awk '{print \$1}'": Escape $ for parallel
# Multiple awk processes run simultaneously

# Parallel grep:
parallel -a large-access.log --pipepart --block 100M "LC_ALL=C grep 'error'" | sort | uniq -c

# Benchmark:
time awk '{print $1}' large-access.log | sort | uniq -c | sort -rn | head -20
# vs
time cat large-access.log | parallel --pipe --block 100M "awk '{print \$1}'" | sort | uniq -c | sort -rn | head -20
```

#### Technique 3: Use Faster Alternatives

```bash
# Instead of grep, use ripgrep (rg):
# Install:
sudo apt-get install ripgrep

# Usage (much faster):
rg "pattern" large-file.log

# Instead of awk for simple field extraction, use cut:
time awk '{print $1}' large-access.log > /dev/null
time cut -d' ' -f1 large-access.log > /dev/null
# cut is often faster for simple cases

# Instead of sort | uniq, use awk:
# Slow:
time sort large-file.txt | uniq > /dev/null

# Fast:
time awk '!seen[$0]++' large-file.txt > /dev/null
```

#### Technique 4: Avoid Unnecessary Operations

```bash
# ❌ BAD: Multiple passes through file
cat large-access.log | grep "error" > /tmp/errors.txt
cat /tmp/errors.txt | awk '{print $1}' > /tmp/ips.txt
cat /tmp/ips.txt | sort > /tmp/sorted.txt

# ✅ GOOD: Single pipeline
grep "error" large-access.log | awk '{print $1}' | sort

# ❌ BAD: Useless use of cat
cat file.txt | grep "pattern"

# ✅ GOOD: Direct file input
grep "pattern" file.txt

# ❌ BAD: Process entire file when you need first match
grep "error" huge-file.log | head -1

# ✅ GOOD: Stop early
grep -m 1 "error" huge-file.log
# OR
sed -n '/error/p; /error/q' huge-file.log
```

#### Technique 5: Use Binary Mode

```bash
# For binary-safe operations:
grep -a "pattern" file
# -a treats file as text even if binary data present

# Or use strings to extract text from binary:
strings binary-file | grep "pattern"
```

#### Technique 6: Memory-Mapped Files

```bash
# For very large files, use mmap-based tools:
# ripgrep uses mmap internally

# Or use mmap in scripts:
# Perl with File::Map:
perl -MFile::Map -e '
  open my $fh, "<", $ARGV[0];
  map_file my $map, $fh;
  while ($map =~ /pattern/g) { print $& }
' large-file.log
```

### Part 3: Large File Strategies

```bash
# Strategy 1: Sample the file
# Analyze first 1 million lines instead of all 100 million
head -1000000 huge-file.log | your_analysis_pipeline

# Strategy 2: Split file
split -l 1000000 huge-file.log chunk_
# Creates chunk_aa, chunk_ab, etc.
# Process in parallel:
for chunk in chunk_*; do
    your_analysis_pipeline < "$chunk" &
done
wait

# Strategy 3: Use streaming analysis
# Don't store intermediate results
awk '{count[$1]++} END {for (ip in count) print count[ip], ip}' huge-file.log
# Stores only unique IPs in memory, not entire file

# Strategy 4: Database approach
# Import to SQLite for complex queries:
sqlite3 :memory: <<EOF
.mode csv
.import huge-file.csv data
SELECT ip, COUNT(*) as cnt FROM data GROUP BY ip ORDER BY cnt DESC LIMIT 20;
EOF
```

---

## Real-World DevOps Scenarios

### Scenario 1: Security Incident Response

**Task:** Identify potential DDoS attack

```bash
# Step 1: Find IPs with > 1000 requests in last hour
# Assuming log has timestamps
awk '{print $4, $1}' access.log | \
  awk -F: '{if ($2 == "10") print $0}' | \
  awk '{count[$2]++} END {for (ip in count) if (count[ip] > 1000) print count[ip], ip}' | \
  sort -rn

# Step 2: Analyze request patterns from top offender
TOP_IP="192.168.1.100"
grep "$TOP_IP" access.log | awk '{print $7}' | sort | uniq -c | sort -rn | head -20

# Step 3: Check if legitimate (varied URLs or single target?)
grep "$TOP_IP" access.log | awk '{print $7}' | sort -u | wc -l
# Low number = likely attack

# Step 4: Extract attack timeline
grep "$TOP_IP" access.log | awk '{print $4}' | awk -F: '{print $2":"$3}' | sort | uniq -c

# Step 5: Generate block command
echo "iptables -A INPUT -s $TOP_IP -j DROP"

# Complete automated script:
cat > analyze-ddos.sh << 'EOF'
#!/bin/bash
LOGFILE=$1
THRESHOLD=${2:-1000}

echo "=== Analyzing $LOGFILE for potential DDoS ==="
echo "Threshold: $THRESHOLD requests"
echo ""

echo "Top offending IPs:"
awk '{print $1}' "$LOGFILE" | sort | uniq -c | sort -rn | head -20 | \
  awk -v thresh="$THRESHOLD" '$1 > thresh {print $1, $2}'

echo ""
echo "Recommended blocks:"
awk '{print $1}' "$LOGFILE" | sort | uniq -c | sort -rn | head -20 | \
  awk -v thresh="$THRESHOLD" '$1 > thresh {print "iptables -A INPUT -s", $2, "-j DROP"}'
EOF

chmod +x analyze-ddos.sh
./analyze-ddos.sh access.log 1000
```

### Scenario 2: Performance Analysis

**Task:** Find slow endpoints causing server load

```bash
# Assuming log format with response time at end
# Format: ... /endpoint HTTP/1.1" 200 1234 response_time_ms

# Step 1: Average response time per endpoint
awk '{endpoint=$7; time=$NF; sum[endpoint]+=time; count[endpoint]++} 
     END {for (e in sum) printf "%s\t%.2f ms\n", e, sum[e]/count[e]}' access.log | \
  sort -k2 -rn | head -20

# Step 2: Find endpoints with response time > 1s
awk '$NF > 1000 {print $7, $NF}' access.log | sort | uniq -c | sort -rn

# Step 3: Correlate with HTTP status codes
awk '$NF > 1000 {print $7, $9}' access.log | sort | uniq -c | sort -rn

# Step 4: Hourly performance trend
awk '{
  hour=substr($4, 14, 2); 
  if ($NF > 1000) slow[hour]++; 
  total[hour]++
} 
END {
  for (h in total) 
    printf "%s:00 - Slow: %d (%.2f%%)\n", h, slow[h], (slow[h]/total[h])*100
}' access.log | sort

# Complete performance report script:
cat > performance-report.sh << 'EOF'
#!/bin/bash
LOGFILE=$1

echo "=== Performance Report for $LOGFILE ==="
echo ""

echo "Slowest Endpoints (avg response time):"
awk '{sum[$7]+=$NF; count[$7]++} 
     END {for (e in sum) printf "%.2f ms\t%s\n", sum[e]/count[e], e}' "$LOGFILE" | \
  sort -rn | head -10

echo ""
echo "Endpoints with > 1s response time:"
awk '$NF > 1000 {count[$7]++} 
     END {for (e in count) print count[e], e}' "$LOGFILE" | \
  sort -rn | head -10

echo ""
echo "Hourly slow request rate:"
awk '{
  hour=substr($4, 14, 2); 
  if ($NF > 1000) slow[hour]++; 
  total[hour]++
} 
END {
  for (h in total) 
    printf "%s:00\t%d slow / %d total (%.2f%%)\n", h, slow[h], total[h], (slow[h]/total[h])*100
}' "$LOGFILE" | sort -k1
EOF

chmod +x performance-report.sh
./performance-report.sh access.log
```

### Scenario 3: Application Error Tracking

**Task:** Identify and categorize application errors

```bash
# Step 1: Extract error messages from application log
grep -E "(ERROR|FATAL|EXCEPTION)" app.log | \
  sed 's/^.*ERROR/ERROR/' | \
  sort | uniq -c | sort -rn | head -20

# Step 2: Group by error type
grep "ERROR" app.log | \
  awk -F: '{print $2}' | \
  sed 's/[0-9]//g' | \
  sort | uniq -c | sort -rn

# Step 3: Extract stack traces
awk '/EXCEPTION/,/^$/' app.log > exceptions.txt

# Step 4: Count by exception type
grep "Exception" app.log | \
  awk '{for(i=1;i<=NF;i++) if($i ~ /Exception$/) print $i}' | \
  sort | uniq -c | sort -rn

# Step 5: Timeline of errors
grep "ERROR" app.log | \
  awk '{print $1, $2}' | \
  awk -F: '{print $1":"$2":00"}' | \
  sort | uniq -c

# Complete error analysis script:
cat > analyze-errors.sh << 'EOF'
#!/bin/bash
LOGFILE=$1

echo "=== Error Analysis Report ==="
echo ""

echo "Top 10 Error Messages:"
grep -E "(ERROR|FATAL)" "$LOGFILE" | \
  sed 's/^.*\(ERROR\|FATAL\)/\1/' | \
  sed 's/:[0-9]*//g' | \
  sort | uniq -c | sort -rn | head -10

echo ""
echo "Errors by Hour:"
grep -E "(ERROR|FATAL)" "$LOGFILE" | \
  awk '{print substr($1, 1, 13)}' | \
  sort | uniq -c

echo ""
echo "Exception Types:"
grep "Exception" "$LOGFILE" | \
  awk '{for(i=1;i<=NF;i++) if($i ~ /Exception/) print $i}' | \
  sed 's/[:(].*//' | \
  sort | uniq -c | sort -rn | head -10

echo ""
echo "Error Rate:"
TOTAL=$(wc -l < "$LOGFILE")
ERRORS=$(grep -c -E "(ERROR|FATAL)" "$LOGFILE")
echo "Total lines: $TOTAL"
echo "Error lines: $ERRORS"
echo "Error rate: $(awk "BEGIN {printf \"%.2f%%\", ($ERRORS/$TOTAL)*100}")"
EOF

chmod +x analyze-errors.sh
./analyze-errors.sh app.log
```

### Scenario 4: User Behavior Analysis

**Task:** Track user journeys through website

```bash
# Step 1: Extract session IDs (assuming cookie-based)
# Format: ... "Cookie: session=abc123; ..." ...

grep -oP 'session=\K[^;]+' access.log | sort | uniq -c | sort -rn | head -20

# Step 2: Track pages visited per session
SESSION="abc123"
grep "session=$SESSION" access.log | awk '{print $4, $7}' | sort

# Step 3: Most common user paths (simplified)
awk '{print $7}' access.log | \
  paste -d' ' - - | \
  sort | uniq -c | sort -rn | head -20

# Step 4: Conversion funnel analysis
# Count visits to each funnel step
for step in "/home" "/product" "/cart" "/checkout"; do
    count=$(grep -c "$step" access.log)
    echo "$step: $count"
done

# Step 5: Drop-off analysis
awk '{
  if ($7 == "/home") home++
  if ($7 == "/product") product++
  if ($7 == "/cart") cart++
  if ($7 == "/checkout") checkout++
}
END {
  print "Home:", home
  print "Product:", product, sprintf("(%.2f%%)", (product/home)*100)
  print "Cart:", cart, sprintf("(%.2f%%)", (cart/product)*100)
  print "Checkout:", checkout, sprintf("(%.2f%%)", (checkout/cart)*100)
}' access.log
```

### Scenario 5: API Usage Monitoring

**Task:** Monitor API endpoint usage and identify abuse

```bash
# Step 1: Extract API endpoints
grep "/api/" access.log | awk '{print $7}' | sort | uniq -c | sort -rn

# Step 2: Requests per API key (assuming header)
grep "X-API-Key:" access.log | \
  grep -oP 'X-API-Key: \K[^"]+' | \
  sort | uniq -c | sort -rn

# Step 3: Rate limiting check (requests per minute per key)
awk '/X-API-Key:/ {
  match($0, /X-API-Key: ([^"]+)/, key)
  minute=substr($4, 14, 5)
  count[key[1],minute]++
}
END {
  for (k in count) {
    split(k, parts, SUBSEP)
    printf "%s at %s: %d requests\n", parts[1], parts[2], count[k]
  }
}' access.log | awk '$NF > 60' | sort -k5 -rn

# Step 4: Endpoint popularity by status code
awk '/\/api\// {
  endpoint=$7
  status=$9
  count[endpoint,status]++
}
END {
  for (key in count) {
    split(key, parts, SUBSEP)
    printf "%s\t%s\t%d\n", parts[1], parts[2], count[key]
  }
}' access.log | sort -k1,1 -k3,3rn

# Complete API monitoring script:
cat > monitor-api.sh << 'EOF'
#!/bin/bash
LOGFILE=$1

echo "=== API Usage Report ==="
echo ""

echo "Top 20 API Endpoints:"
grep "/api/" "$LOGFILE" | awk '{print $7}' | sort | uniq -c | sort -rn | head -20

echo ""
echo "API Keys with > 1000 requests:"
awk '{
  if (match($0, /api_key=([^& ]+)/, key)) {
    count[key[1]]++
  }
}
END {
  for (k in count) {
    if (count[k] > 1000) print count[k], k
  }
}' "$LOGFILE" | sort -rn

echo ""
echo "Rate Limit Violations (>100 req/min):"
awk '{
  if (match($0, /api_key=([^& ]+)/, key)) {
    minute=substr($4, 1, 16)
    count[key[1],minute]++
  }
}
END {
  for (k in count) {
    if (count[k] > 100) {
      split(k, parts, SUBSEP)
      printf "%s %s: %d requests\n", parts[1], parts[2], count[k]
    }
  }
}' "$LOGFILE" | sort -k3 -rn

echo ""
echo "Failed API Calls (4xx, 5xx):"
grep "/api/" "$LOGFILE" | awk '$9 ~ /^[45]/ {print $9, $7}' | sort | uniq -c | sort -rn | head -20
EOF

chmod +x monitor-api.sh
./monitor-api.sh access.log
```

---

## Advanced Techniques

### Technique 1: One-Liner Mastery

```bash
# Convert CSV to JSON
awk -F, 'NR==1{for(i=1;i<=NF;i++)h[i]=$i;next}{printf "{";for(i=1;i<=NF;i++)printf "\"%s\":\"%s\"%s",h[i],$i,(i<NF?",":"");print "}"}' file.csv

# Generate random test data
for i in {1..1000}; do echo "192.168.$((RANDOM%256)).$((RANDOM%256)) - - [$(date '+%d/%b/%Y:%H:%M:%S %z')] \"GET /page$((RANDOM%100)).html HTTP/1.1\" $((RANDOM%2?200:404)) $((RANDOM*100))"; done

# Extract emails from text
grep -Eo "\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b" file.txt

# Convert Unix timestamp to readable date
awk '{$1=strftime("%Y-%m-%d %H:%M:%S", $1); print}' timestamps.txt

# Calculate percentiles
sort -n numbers.txt | awk 'BEGIN{c=0}{a[c]=$1;c++}END{p50=int(c*0.5);p95=int(c*0.95);p99=int(c*0.99);print "P50:",a[p50],"P95:",a[p95],"P99:",a[p99]}'

# Find files modified in last 24 hours and show sizes
find . -type f -mtime -1 -exec ls -lh {} \; | awk '{print $5, $9}'

# Network bandwidth by IP (from tcpdump)
tcpdump -n -r capture.pcap | awk '{print $3}' | cut -d. -f1-4 | sort | uniq -c | sort -rn

# Parse nginx log and create CSV report
awk '{print $1","$7","$9","$10}' access.log | sed '1iIP,URL,Status,Bytes' > report.csv
```

### Technique 2: Complex Multi-File Analysis

```bash
# Compare two log files and find differences
comm -3 <(awk '{print $1}' log1.txt | sort) <(awk '{print $1}' log2.txt | sort)

# Merge multiple logs by timestamp
sort -m -k4 access1.log access2.log access3.log

# Find IPs in log1 but not in log2
awk 'NR==FNR{a[$1];next}!($1 in a)' log2.txt log1.txt

# Cross-reference logs
# File1: IPs
# File2: Access log
awk 'NR==FNR{ips[$1];next}$1 in ips{print}' suspicious-ips.txt access.log

# Aggregate statistics across multiple files
cat access*.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -20
```

### Technique 3: Real-Time Log Monitoring

```bash
# Live monitoring with color
tail -f access.log | awk '{
  if ($9 ~ /^2/) print "\033[32m" $0 "\033[0m"
  else if ($9 ~ /^4/) print "\033[33m" $0 "\033[0m"
  else if ($9 ~ /^5/) print "\033[31m" $0 "\033[0m"
  else print $0
}'

# Alert on specific pattern
tail -f app.log | awk '/ERROR/{print "\a"; system("echo ERROR DETECTED | mail -s Alert admin@example.com")}'

# Live top IPs
tail -f access.log | awk '{print $1}' | uniq -c | sort -rn | head -10

# Real-time rate calculation
tail -f access.log | awk '{count++; if(NR%100==0) {print count, "requests/sec"; count=0}}'

# Dashboard-style monitoring
watch -n 1 'tail -1000 access.log | awk "{print \$1}" | sort | uniq -c | sort -rn | head -20'
```

### Technique 4: Log Correlation

```bash
# Correlate application log with access log
# Application log: timestamp,user_id,action
# Access log: standard format

# Find user actions around error time
ERROR_TIME="15/Jan/2024:10:30:45"
grep "$ERROR_TIME" access.log | awk '{print $1}' | while read ip; do
    grep "$ip" app.log
done

# Multi-log correlation script:
cat > correlate-logs.sh << 'EOF'
#!/bin/bash
# Correlate nginx access log with application error log

ACCESS_LOG=$1
ERROR_LOG=$2
TIME_WINDOW=${3:-5}  # minutes

# Extract error timestamps
awk '/ERROR/ {print $1" "$2}' "$ERROR_LOG" | while read timestamp; do
    # Convert to epoch for math
    epoch=$(date -d "$timestamp" +%s)
    start=$((epoch - TIME_WINDOW*60))
    end=$((epoch + TIME_WINDOW*60))
    
    echo "=== Errors around $timestamp ==="
    
    # Find access log entries in time window
    awk -v start="$start" -v end="$end" '{
        # Parse timestamp from access log
        # This is simplified - actual implementation needs proper parsing
        print $0
    }' "$ACCESS_LOG" | head -20
    
    echo ""
done
EOF
```

### Technique 5: Custom Log Formats

```bash
# Define log format parser
parse_custom_log() {
    # Custom format: LEVEL|TIMESTAMP|SOURCE|MESSAGE
    # Example: ERROR|2024-01-15 10:30:45|app.py:123|Connection failed
    
    awk -F'|' '{
        level=$1
        timestamp=$2
        source=$3
        message=$4
        
        # Your analysis here
        if (level == "ERROR") {
            print timestamp, source, message
        }
    }' "$1"
}

# Use it:
parse_custom_log app.log | sort | uniq -c | sort -rn
```

---

## Muscle Memory Drills

### Drill 1: Quick IP Extraction (30 seconds)

```bash
# From any position, extract top IPs from access log

# Drill:
time awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -20

# Goal: Type command from memory in < 30 seconds
# Practice 10 times daily until automatic
```

### Drill 2: Error Analysis Pipeline (1 minute)

```bash
# Extract, count, and sort errors

# Drill:
time grep -i error app.log | awk '{print $NF}' | sort | uniq -c | sort -rn | head -10

# Variations to practice:
# 1. Case insensitive: grep -i
# 2. Count only: grep -c
# 3. With context: grep -C 3
# 4. With line numbers: grep -n
```

### Drill 3: Complex Pipeline Construction (2 minutes)

```bash
# Build complete analysis from scratch

# Drill: Find IPs with >100 404 errors, show count and IPs
time awk '$9==404 {print $1}' access.log | sort | uniq -c | awk '$1>100' | sort -rn

# Goal: Construct similar pipelines without reference
```

### Drill 4: One-Liner Speed Test (5 minutes)

```bash
# Type these one-liners from memory as fast as possible:

# 1. Top user agents:
awk -F'"' '{print $6}' access.log | sort | uniq -c | sort -rn | head -10

# 2. Bandwidth by IP:
awk '{sum[$1]+=$10} END {for(ip in sum) print sum[ip], ip}' access.log | sort -rn | head -20

# 3. Requests per hour:
awk '{print substr($4,14,2)}' access.log | sort | uniq -c

# 4. Status code distribution:
awk '{print $9}' access.log | sort | uniq -c | sort -rn

# 5. Failed requests:
awk '$9>=400 {print $1,$7,$9}' access.log | sort | uniq | head -20

# Track your time for all 5
# Goal: < 3 minutes
```

---

## Cheat Sheet

```
┌─────────────────────────────────────────────────────────────┐
│  TEXT PROCESSING QUICK REFERENCE                            │
├─────────────────────────────────────────────────────────────┤
│  GREP:                                                      │
│    grep "pattern" file              → Search              │
│    grep -i "pattern" file           → Case insensitive    │
│    grep -v "pattern" file           → Invert match        │
│    grep -E "p1|p2" file             → Multiple patterns   │
│    grep -A 3 "pattern" file         → Show 3 lines after  │
│                                                             │
│  AWK:                                                       │
│    awk '{print $1}' file            → Print field 1       │
│    awk -F: '{print $1}' file        → Custom separator    │
│    awk '$3>100' file                → Conditional         │
│    awk '{sum+=$1} END {print sum}'  → Aggregation         │
│    awk '!seen[$1]++'                → Unique lines        │
│                                                             │
│  SED:                                                       │
│    sed 's/old/new/' file            → Replace (first)     │
│    sed 's/old/new/g' file           → Replace (all)       │
│    sed '/pattern/d' file            → Delete lines        │
│    sed -n '5,10p' file              → Print range         │
│    sed -i 's/old/new/g' file        → In-place edit       │
│                                                             │
│  SORT & UNIQ:                                               │
│    sort file                        → Alphabetical        │
│    sort -n file                     → Numeric             │
│    sort -r file                     → Reverse             │
│    sort -k2 file                    → By field 2          │
│    sort file | uniq -c              → Count unique        │
│    sort file | uniq -d              → Show duplicates     │
│                                                             │
│  COMMON PIPELINES:                                          │
│    Top IPs:                                                 │
│      awk '{print $1}' log | sort | uniq -c | sort -rn     │
│                                                             │
│    Count by field:                                          │
│      awk '{print $9}' log | sort | uniq -c                │
│                                                             │
│    Filter and extract:                                      │
│      grep "pattern" log | awk '{print $3}' | sort -u      │
│                                                             │
│    Aggregate and calculate:                                 │
│      awk '{sum+=$10} END {print sum}' log                  │
└─────────────────────────────────────────────────────────────┘
```

---

## Final Mastery Test

### Test 1: Top 20 IPs (30 seconds)

```bash
# Given: access.log (1GB+)
# Task: Extract top 20 IP addresses by request count

# Your command:
# _______________________________________

# Solution:
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -20

# Time: _____ seconds
```

### Test 2: Complex Analysis (2 minutes)

```bash
# Given: access.log
# Task: Find IPs with >100 failed requests (4xx, 5xx)

# Your command:
# _______________________________________

# Solution:
awk '$9~/^[45]/ {print $1}' access.log | sort | uniq -c | awk '$1>100 {print}' | sort -rn

# Time: _____ seconds
```

### Test 3: Multi-Format Parsing (3 minutes)

```bash
# Given: app.log (custom format: TIMESTAMP|LEVEL|MESSAGE)
# Task: Count errors by hour, show top 5 hours

# Your command:
# _______________________________________

# Solution:
awk -F'|' '$2=="ERROR" {split($1,t," "); split(t[2],h,":"); print h[1]}' app.log | sort | uniq -c | sort -rn | head -5

# Time: _____ seconds
```

---

## Conclusion

You've mastered:
- ✅ grep, awk, sed, sort, uniq pipeline construction
- ✅ Sub-30 second log analysis on 1GB+ files
- ✅ Multiple log format parsing
- ✅ Performance optimization techniques
- ✅ Real-world DevOps scenarios

**Daily Practice:**
1. Analyze production logs daily
2. Time yourself on common tasks
3. Build new one-liners weekly
4. Share knowledge with team

**Remember:** The goal isn't to memorize commands, but to develop intuition for text processing patterns. With practice, solutions flow naturally! 🚀