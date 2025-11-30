# File Operations Bootcamp: Complete Mastery Guide

## Table of Contents
1. [Foundation Concepts](#foundation-concepts)
2. [Creating 1000 Sequential Files](#creating-1000-sequential-files)
3. [Mass Rename Operations](#mass-rename-operations)
4. [Symbolic and Hard Links](#symbolic-and-hard-links)
5. [Managing 100+ Files Simultaneously](#managing-100-files-simultaneously)
6. [Muscle Memory Drills](#muscle-memory-drills)
7. [Real-World DevOps Scenarios](#real-world-devops-scenarios)
8. [Advanced Techniques](#advanced-techniques)

---

## Foundation Concepts

### Why File Operations Matter in DevOps

```
┌─────────────────────────────────────────────────────────┐
│  Daily DevOps File Operation Tasks:                     │
├─────────────────────────────────────────────────────────┤
│  ✓ Batch process log files (rename, archive, analyze)  │
│  ✓ Deploy configuration files across environments      │
│  ✓ Create symbolic links for version management        │
│  ✓ Manage backup files with sequential naming          │
│  ✓ Organize artifacts from CI/CD pipelines             │
│  ✓ Handle log rotation and archival                    │
│  ✓ Version control for configuration management        │
└─────────────────────────────────────────────────────────┘
```

### The Three Core File Operation Commands

```
┌──────────────────────────────────────────────────────────┐
│  COMMAND   │  PURPOSE           │  CREATES/MODIFIES      │
├──────────────────────────────────────────────────────────┤
│  touch     │  Create empty file │  File metadata         │
│            │  Update timestamps │  (access/modify time)  │
│            │                    │                        │
│  rename    │  Batch rename      │  Filename changes      │
│  (rename)  │  Pattern matching  │  (perl-based)          │
│            │                    │                        │
│  ln        │  Create links      │  Hard/symbolic links   │
│            │  File references   │  (pointers to files)   │
└──────────────────────────────────────────────────────────┘
```

### File System Concepts Visual

```
INODE TABLE (File Metadata)
┌─────────────────────────────────┐
│ Inode #123                      │
│ ├─ Permissions: -rw-r--r--      │
│ ├─ Owner: user                  │
│ ├─ Size: 4096 bytes             │
│ ├─ Created: 2024-01-15 10:00    │
│ ├─ Modified: 2024-01-15 14:30   │
│ └─ Data Block Pointers: →→→     │
└─────────────────────────────────┘
                ↓
        DIRECTORY ENTRY
┌─────────────────────────────────┐
│ filename.txt → Inode #123       │  ← Hard link (direct reference)
│ symlink.txt  → filename.txt     │  ← Symbolic link (name reference)
└─────────────────────────────────┘
                ↓
         ACTUAL DATA BLOCKS
┌─────────────────────────────────┐
│ [File content lives here...]    │
└─────────────────────────────────┘
```

---

## Creating 1000 Sequential Files

### Part 1: Understanding the Touch Command

#### Basic Touch Syntax

```bash
# Create a single file
touch file.txt

# What happens internally:
# 1. Check if file exists
#    - If YES: Update timestamps (access time, modification time)
#    - If NO:  Create empty file (0 bytes)
# 2. Update directory entry
# 3. Allocate inode (but no data blocks until content written)
```

**Detailed Breakdown:**

```bash
# Check before touch
ls -l file.txt
# Output: ls: cannot access 'file.txt': No such file or directory

# Create file
touch file.txt

# Check after touch
ls -l file.txt
# Output: -rw-r--r-- 1 user user 0 Jan 15 10:30 file.txt
#         └─┬──┘ └┬┘ └─┬─┘ └─┬─┘ └┬┘ └────┬─────┘ └───┬────┘
#           │     │    │      │    │       │            │
#      Permissions Links Owner Group Size  Time        Name
```

**Why Touch is Used:**

```
┌────────────────────────────────────────────────────┐
│  TOUCH COMMAND USE CASES:                          │
├────────────────────────────────────────────────────┤
│  1. Create placeholder files                       │
│     (e.g., lock files, flag files)                 │
│                                                     │
│  2. Update modification time                       │
│     (trigger make/build systems)                   │
│                                                     │
│  3. Create test files for scripts                  │
│     (testing file processing pipelines)            │
│                                                     │
│  4. Batch file creation                            │
│     (our current task: 1000 files)                 │
└────────────────────────────────────────────────────┘
```

### Part 2: Understanding Brace Expansion for Sequences

#### How Brace Expansion Works

```bash
# Simple sequence
echo {1..5}
# Shell expands to: 1 2 3 4 5
# Output: 1 2 3 4 5

# With prefix
echo file{1..5}.txt
# Shell expands to: file1.txt file2.txt file3.txt file4.txt file5.txt
# Output: file1.txt file2.txt file3.txt file4.txt file5.txt

# With zero-padding
echo file{01..05}.txt
# Shell expands to: file01.txt file02.txt file03.txt file04.txt file05.txt
# Output: file01.txt file02.txt file03.txt file04.txt file05.txt
```

**Expansion Happens BEFORE Command Execution:**

```
┌─────────────────────────────────────────────────────┐
│  SHELL PROCESSING ORDER:                            │
├─────────────────────────────────────────────────────┤
│  1. User types: touch file{1..3}.txt               │
│                                                     │
│  2. Shell expands braces:                          │
│     touch file1.txt file2.txt file3.txt            │
│                                                     │
│  3. Shell executes touch with 3 arguments:         │
│     arg1: file1.txt                                │
│     arg2: file2.txt                                │
│     arg3: file3.txt                                │
│                                                     │
│  4. Touch creates each file sequentially           │
└─────────────────────────────────────────────────────┘
```

### Part 3: Creating 1000 Files

#### Method 1: Single Command (Recommended)

```bash
# Create practice directory
mkdir -p ~/file-bootcamp
cd ~/file-bootcamp

# Create 1000 files with zero-padded sequential names
touch file{0001..1000}.txt

# Why this works:
# {0001..1000} expands to: 0001, 0002, 0003, ..., 0999, 1000
# Zero-padding ensures proper alphabetical sorting
```

**Step-by-Step Breakdown:**

```bash
# What the shell does:
# 1. Parse command: touch file{0001..1000}.txt
# 2. Expand brace expression to 1000 arguments
# 3. Execute: touch file0001.txt file0002.txt ... file1000.txt
# 4. Touch creates each file (1000 operations)

# Time this operation:
time touch file{0001..1000}.txt

# Expected output:
# real    0m0.234s   (actual time elapsed)
# user    0m0.052s   (CPU time in user mode)
# sys     0m0.182s   (CPU time in kernel mode)
#
# Why so fast?
# - No data written, only metadata
# - Sequential inode allocation
# - Filesystem caching
```

**Verification:**

```bash
# Count created files
ls -1 | wc -l
# Output: 1000

# Show first 10 files
ls | head -10
# Output:
# file0001.txt
# file0002.txt
# file0003.txt
# ...
# file0010.txt

# Show last 10 files
ls | tail -10
# Output:
# file0991.txt
# file0992.txt
# ...
# file1000.txt

# Check file sizes (should all be 0)
ls -lh file0001.txt
# Output: -rw-r--r-- 1 user user 0 Jan 15 10:30 file0001.txt
#                                └─ 0 bytes (empty file)
```

#### Method 2: Using a Loop (Educational)

```bash
# Bash for loop approach
for i in {0001..1000}; do
    touch "file${i}.txt"
done

# Breakdown:
# for i in {0001..1000}  → Loop variable i takes values 0001, 0002, ..., 1000
# do                     → Start of loop body
#     touch "file${i}.txt"  → Create file with interpolated variable
# done                   → End of loop

# Why slower than Method 1?
# - Executes touch 1000 times (not once with 1000 args)
# - More process overhead
# - Less efficient

# Time comparison:
time touch file{0001..1000}.txt    # ~0.2 seconds
time for i in {0001..1000}; do touch "file${i}.txt"; done  # ~2.5 seconds
# Loop is ~12x slower!
```

#### Method 3: Using seq with xargs (Advanced)

```bash
# Generate sequence and pipe to xargs
seq -w 1 1000 | xargs -I {} touch "file{}.txt"

# Breakdown:
# seq -w 1 1000          → Generate 0001, 0002, ..., 1000 (-w = zero-pad)
# |                      → Pipe output to next command
# xargs -I {}            → Replace {} with each input line
# touch "file{}.txt"     → Create file with sequence number

# Why use this?
# - More flexible for complex naming patterns
# - Can be combined with other commands
# - Useful when brace expansion has limitations
```

### Part 4: Understanding Zero-Padding

**Why Zero-Padding Matters:**

```bash
# WITHOUT zero-padding:
touch file{1..1000}.txt
ls | head -20

# Output (alphabetical sorting):
# file1.txt
# file10.txt      ← 10 comes BEFORE 2 (alphabetically)
# file100.txt
# file1000.txt
# file101.txt
# ...
# file2.txt       ← 2 comes AFTER 100 (alphabetically!)
# file20.txt

# This breaks:
# - Sequential processing scripts
# - Human readability
# - Batch operations expecting order

# WITH zero-padding:
touch file{0001..1000}.txt
ls | head -20

# Output (correct sequential order):
# file0001.txt
# file0002.txt
# file0003.txt
# ...
# file0020.txt

# Why this matters in DevOps:
# - Log files: app.log.0001, app.log.0002
# - Backup files: backup-0001.tar.gz
# - Batch processing: process files in order
```

**Padding Rules:**

```
┌────────────────────────────────────────────────────┐
│  ZERO-PADDING PATTERNS:                            │
├────────────────────────────────────────────────────┤
│  {01..99}      → 2 digits (01, 02, ..., 99)        │
│  {001..999}    → 3 digits (001, 002, ..., 999)     │
│  {0001..9999}  → 4 digits (0001, 0002, ..., 9999)  │
│                                                     │
│  Rule: Leading zeros in start number define width  │
│                                                     │
│  {0001..100}   → 0001, 0002, ..., 0100 (4 digits)  │
│  {1..1000}     → 1, 2, ..., 1000 (variable width)  │
└────────────────────────────────────────────────────┘
```

### Part 5: Adding Content to Files

```bash
# Create 1000 files with sequential content
for i in {0001..1000}; do
    echo "This is file number $i" > "file${i}.txt"
done

# Verify content
cat file0001.txt
# Output: This is file number 0001

cat file0500.txt
# Output: This is file number 0500

# Check file sizes now (no longer 0 bytes)
ls -lh file0001.txt
# Output: -rw-r--r-- 1 user user 26 Jan 15 10:30 file0001.txt
#                                └─ 26 bytes (has content)
```

**Advanced Content Patterns:**

```bash
# Create files with timestamps
for i in {0001..1000}; do
    echo "Created: $(date)" > "file${i}.txt"
done

# Create files with random data (for testing)
for i in {0001..1000}; do
    echo $RANDOM > "file${i}.txt"
done

# Create files with specific sizes (for disk space testing)
for i in {0001..1000}; do
    dd if=/dev/zero of="file${i}.txt" bs=1M count=1 2>/dev/null
done
# Creates 1000 files, each exactly 1MB
```

---

## Mass Rename Operations

### Part 1: Understanding the Rename Command

#### Rename Command Variants

```
┌────────────────────────────────────────────────────┐
│  TWO DIFFERENT RENAME COMMANDS EXIST:              │
├────────────────────────────────────────────────────┤
│  1. Perl-based rename (Debian/Ubuntu)              │
│     - rename 's/old/new/' files                    │
│     - Uses Perl regex                              │
│     - More powerful                                │
│                                                     │
│  2. util-linux rename (RedHat/CentOS)              │
│     - rename old new files                         │
│     - Simple string replacement                    │
│     - Less powerful                                │
│                                                     │
│  Check which you have:                             │
│  rename --version                                  │
└────────────────────────────────────────────────────┘
```

**Installing Perl Rename:**

```bash
# Debian/Ubuntu
sudo apt-get install rename

# RedHat/CentOS (install perl-rename)
sudo yum install prename

# macOS (via Homebrew)
brew install rename

# Verify installation
which rename
# Output: /usr/bin/rename

rename --version
# Output: /usr/bin/rename using File::Rename version 1.10
```

#### Perl Rename Syntax

```bash
# Basic structure:
rename [OPTIONS] 's/PATTERN/REPLACEMENT/FLAGS' FILES

# Breakdown:
# s/           → Substitute command (Perl regex)
# PATTERN      → What to find (regex)
# REPLACEMENT  → What to replace with
# FLAGS        → Modifiers (g=global, i=ignore case)
# FILES        → Target files

# Example:
rename 's/file/document/' file*.txt

# What happens:
# file0001.txt → document0001.txt
# file0002.txt → document0002.txt
# ...
```

**Rename Command Options:**

```bash
# -n or --dry-run (TEST without actually renaming)
rename -n 's/file/doc/' file*.txt
# Output: rename(file0001.txt, doc0001.txt)
#         rename(file0002.txt, doc0002.txt)
# Files NOT renamed, only shows what WOULD happen

# -v or --verbose (show what's being renamed)
rename -v 's/file/doc/' file*.txt
# Output: file0001.txt renamed as doc0001.txt
#         file0002.txt renamed as doc0002.txt

# -f or --force (overwrite existing files)
rename -f 's/file/doc/' file*.txt
# Overwrites doc0001.txt if it already exists
```

### Part 2: Basic Rename Patterns

#### Pattern 1: Simple Text Replacement

```bash
# Setup: Create test files
mkdir -p ~/rename-practice
cd ~/rename-practice
touch file{0001..0100}.txt

# Rename "file" to "document"
rename -n 's/file/document/' file*.txt
# Output preview:
# rename(file0001.txt, document0001.txt)
# rename(file0002.txt, document0002.txt)
# ...

# Actually rename
rename -v 's/file/document/' file*.txt

# Verify
ls | head -5
# Output:
# document0001.txt
# document0002.txt
# document0003.txt
# document0004.txt
# document0005.txt
```

**Why This Works:**

```
Step 1: Shell expands file*.txt → file0001.txt file0002.txt ... file0100.txt

Step 2: Rename processes each filename:
        file0001.txt → Apply s/file/document/ → document0001.txt
        file0002.txt → Apply s/file/document/ → document0002.txt
        ...

Step 3: Filesystem renames each file (updates directory entry)
```

#### Pattern 2: Changing File Extensions

```bash
# Change .txt to .log
rename -v 's/\.txt$/.log/' document*.txt

# Breakdown:
# \.       → Escaped dot (literal dot, not "any character")
# txt      → The text "txt"
# $        → End of string anchor
# /.log/   → Replacement (dot + log)

# Result:
# document0001.txt → document0001.log
# document0002.txt → document0002.log

# Why the $ is important:
# Without $:
# filename.txt.backup → filename.log.backup (WRONG!)
# With $:
# filename.txt.backup → filename.txt.backup (CORRECT - no change)
```

#### Pattern 3: Adding Prefixes/Suffixes

```bash
# Add prefix
rename -v 's/^/backup-/' document*.log
# ^ means "start of string"
# Result: document0001.log → backup-document0001.log

# Add suffix before extension
rename -v 's/\.log$/-old.log/' backup-document*.log
# Result: backup-document0001.log → backup-document0001-old.log

# Add date prefix
rename -v 's/^/2024-01-15-/' backup-*.log
# Result: backup-document0001-old.log → 2024-01-15-backup-document0001-old.log
```

#### Pattern 4: Removing Parts of Filenames

```bash
# Remove prefix
rename -v 's/^2024-01-15-//' 2024-01-15-*.log
# Result: 2024-01-15-backup-document0001-old.log → backup-document0001-old.log

# Remove suffix
rename -v 's/-old//' backup-*.log
# Result: backup-document0001-old.log → backup-document0001.log

# Remove all numbers
rename -v 's/\d+//' backup-*.log
# \d+ means "one or more digits"
# Result: backup-document0001.log → backup-document.log
# Warning: This makes all files have the same name! (only last rename succeeds)
```

#### Pattern 5: Case Conversion

```bash
# Lowercase all
rename -v 'y/A-Z/a-z/' *
# y/// is transliteration operator
# Result: Document0001.txt → document0001.txt

# Uppercase all
rename -v 'y/a-z/A-Z/' *
# Result: document0001.txt → DOCUMENT0001.txt

# Capitalize first letter (advanced)
rename -v 's/^(.)/\U$1/' document*.txt
# \U means uppercase next character
# Result: document0001.txt → Document0001.txt
```

### Part 3: Advanced Rename Patterns

#### Using Capture Groups

```bash
# Extract and rearrange parts
# Original: file0001.txt
# Goal: 0001-file.txt

rename -v 's/file(\d+)\.txt/$1-file.txt/' file*.txt

# Breakdown:
# file     → Literal "file"
# (\d+)    → Capture group 1: one or more digits
# \.txt    → Literal ".txt"
# $1       → Reference to capture group 1
# -file.txt → Literal suffix

# Result: file0001.txt → 0001-file.txt
```

**Complex Example:**

```bash
# Original: app-2024-01-15-production.log
# Goal: production-20240115-app.log

rename -v 's/app-(\d{4})-(\d{2})-(\d{2})-(\w+)\.log/$4-$1$2$3-app.log/' app-*.log

# Breakdown:
# (\d{4})  → Capture group 1: year (4 digits)
# (\d{2})  → Capture group 2: month (2 digits)
# (\d{2})  → Capture group 3: day (2 digits)
# (\w+)    → Capture group 4: environment (one or more word characters)
# $4       → Reference environment
# $1$2$3   → Concatenate year+month+day (no dashes)

# Result: app-2024-01-15-production.log → production-20240115-app.log
```

#### Conditional Renaming

```bash
# Only rename files containing "test"
rename -v 's/test/prod/' *test*.txt
# Files without "test" in name are unchanged

# Only rename files with specific extension
rename -v 's/(\d+)\.txt$/log-$1.txt/' *.txt
# Only .txt files affected

# Skip files matching pattern
ls | grep -v "important" | xargs -I {} rename 's/old/new/' {}
# Renames all except files with "important" in name
```

### Part 4: Bash Parameter Expansion (Alternative to Rename)

#### Understanding Parameter Expansion

```bash
# Parameter expansion syntax:
${variable/pattern/replacement}

# Example:
filename="file0001.txt"
echo ${filename/file/document}
# Output: document0001.txt
# Original $filename unchanged
```

**Common Parameter Expansion Patterns:**

```
┌────────────────────────────────────────────────────────┐
│  PARAMETER EXPANSION PATTERNS:                         │
├────────────────────────────────────────────────────────┤
│  ${var#pattern}    → Remove shortest match from start  │
│  ${var##pattern}   → Remove longest match from start   │
│  ${var%pattern}    → Remove shortest match from end    │
│  ${var%%pattern}   → Remove longest match from end     │
│  ${var/pat/rep}    → Replace first match               │
│  ${var//pat/rep}   → Replace all matches               │
│  ${var^}           → Uppercase first character         │
│  ${var^^}          → Uppercase all                     │
│  ${var,}           → Lowercase first character         │
│  ${var,,}          → Lowercase all                     │
└────────────────────────────────────────────────────────┘
```

#### Batch Rename with Parameter Expansion

```bash
# Change .txt to .log for all files
for file in *.txt; do
    mv "$file" "${file%.txt}.log"
done

# Breakdown:
# for file in *.txt       → Loop through all .txt files
# ${file%.txt}            → Remove .txt from end
# "${file%.txt}.log"      → Add .log extension
# mv "$file" "newname"    → Rename file

# Example:
# file = "document0001.txt"
# ${file%.txt} = "document0001"
# "${file%.txt}.log" = "document0001.log"
```

**Advanced Example:**

```bash
# Add date prefix to all log files
for file in *.log; do
    mv "$file" "$(date +%Y%m%d)-$file"
done

# Result: app.log → 20240115-app.log

# Add suffix before extension
for file in *.txt; do
    base="${file%.txt}"  # Remove .txt
    mv "$file" "${base}-backup.txt"
done

# Result: file0001.txt → file0001-backup.txt
```

#### Real-World Example: Organize by Date

```bash
# Organize files by modification date into directories
for file in *.log; do
    # Get modification date
    date=$(stat -c %y "$file" | cut -d' ' -f1)  # YYYY-MM-DD
    
    # Create directory if needed
    mkdir -p "$date"
    
    # Move file
    mv "$file" "$date/"
done

# Result:
# app.log (modified 2024-01-15) → 2024-01-15/app.log
# error.log (modified 2024-01-14) → 2024-01-14/error.log
```

### Part 5: Common Rename Mistakes

```
┌────────────────────────────────────────────────────────┐
│  COMMON MISTAKES AND FIXES:                            │
├────────────────────────────────────────────────────────┤
│  ❌ MISTAKE 1: Not using dry-run                       │
│     rename 's/important/deleted/' *                    │
│     (Oops! Renamed production files)                   │
│                                                         │
│  ✅ FIX: Always test first                             │
│     rename -n 's/important/deleted/' *                 │
│     (Preview changes before applying)                  │
│                                                         │
│  ❌ MISTAKE 2: Forgetting to escape dots               │
│     rename 's/.txt/.log/' *                            │
│     (Dot matches ANY character)                        │
│                                                         │
│  ✅ FIX: Escape special characters                     │
│     rename 's/\.txt$/.log/' *                          │
│     (Literal dot, end of string)                       │
│                                                         │
│  ❌ MISTAKE 3: Overwriting files                       │
│     rename 's/\d+/001/' file*.txt                      │
│     (All become file001.txt - only last survives!)     │
│                                                         │
│  ✅ FIX: Ensure unique names                           │
│     Check output of -n before running                  │
│                                                         │
│  ❌ MISTAKE 4: Not quoting variables                   │
│     mv $file $newname  # Breaks with spaces            │
│                                                         │
│  ✅ FIX: Always quote                                  │
│     mv "$file" "$newname"                              │
└────────────────────────────────────────────────────────┘
```

---

## Symbolic and Hard Links

### Part 1: Understanding Links Conceptually

#### Visual Representation

```
HARD LINK SCENARIO:
┌──────────────────────────────────────────────────┐
│  INODE #12345 (Actual file data)                 │
│  ├─ Permissions: -rw-r--r--                      │
│  ├─ Size: 1024 bytes                             │
│  ├─ Data blocks: → [Block 100, Block 101]       │
│  └─ Link count: 3  ← Number of directory entries│
└──────────────────────────────────────────────────┘
         ↑           ↑           ↑
         │           │           │
    original.txt  backup.txt  archive.txt
    
    All three names point to SAME inode
    Deleting one does NOT delete the file
    File deleted only when link count = 0
```

```
SYMBOLIC LINK SCENARIO:
┌──────────────────────────────────────────────────┐
│  INODE #12345 (Actual file)                      │
│  └─ Data: "Hello World"                          │
└──────────────────────────────────────────────────┘
         ↑
         │
    original.txt ← Real file
         ↑
         │ (points to name)
    ┌────────────────────┐
    │  INODE #54321      │
    │  (Symbolic link)   │
    │  Data: "original.txt" ← Stores path as text
    └────────────────────┘
         ↑
         │
    symlink.txt ← Symbolic link
    
    Symbolic link is separate file containing path
    If original.txt deleted → symlink becomes broken
```

### Part 2: Hard Links

#### Creating Hard Links

```bash
# Create test file
echo "Original content" > original.txt

# Check inode number
ls -li original.txt
# Output: 123456 -rw-r--r-- 1 user user 17 Jan 15 10:30 original.txt
#         └──┬──┘                └─ Link count = 1
#          Inode number

# Create hard link
ln original.txt hardlink.txt

# Check both files
ls -li original.txt hardlink.txt
# Output:
# 123456 -rw-r--r-- 2 user user 17 Jan 15 10:30 original.txt
# 123456 -rw-r--r-- 2 user user 17 Jan 15 10:30 hardlink.txt
#        └─────────┘ └─ Link count = 2 (same inode!)
#        Same inode number
```

**Hard Link Syntax:**

```bash
# Basic syntax:
ln SOURCE LINK_NAME

# Create multiple hard links
ln original.txt link1.txt
ln original.txt link2.txt
ln original.txt link3.txt

# Verify all point to same inode
ls -li *.txt
# All show same inode number and link count = 4
```

**Hard Link Properties:**

```bash
# Modify content via any link
echo "Changed via hardlink" > hardlink.txt

# All links reflect the change
cat original.txt
# Output: Changed via hardlink

cat link1.txt
# Output: Changed via hardlink

# Why? They all point to the same data blocks!

# Delete original file
rm original.txt

# Other links still work!
cat hardlink.txt
# Output: Changed via hardlink

# File data persists until last link deleted
ls -li hardlink.txt
# Link count decreased by 1
```

**Hard Link Limitations:**

```
┌────────────────────────────────────────────────────┐
│  HARD LINK LIMITATIONS:                            │
├────────────────────────────────────────────────────┤
│  ✗ Cannot link directories (prevents cycles)      │
│  ✗ Cannot cross filesystem boundaries             │
│  ✗ Cannot link to non-existent files              │
│  ✓ Same permissions as original (shared inode)    │
│  ✓ Updates reflect across all links               │
│  ✓ File persists until all links deleted          │
└────────────────────────────────────────────────────┘
```

### Part 3: Symbolic (Soft) Links

#### Creating Symbolic Links

```bash
# Create test file
echo "Original content" > original.txt

# Create symbolic link
ln -s original.txt symlink.txt

# Check with ls -l
ls -l symlink.txt
# Output: lrwxrwxrwx 1 user user 12 Jan 15 10:30 symlink.txt -> original.txt
#         └─ 'l' indicates symbolic link     └─ Shows target

# Check inode (different from original)
ls -li original.txt symlink.txt
# Output:
# 123456 -rw-r--r-- 1 user user 17 Jan 15 10:30 original.txt
# 789012 lrwxrwxrwx 1 user user 12 Jan 15 10:30 symlink.txt -> original.txt
#        └─ Different inode
```

**Symbolic Link Syntax:**

```bash
# Basic syntax:
ln -s TARGET LINK_NAME

# -s means "symbolic" (soft link)

# Absolute path symlink
ln -s /home/user/original.txt /tmp/symlink.txt

# Relative path symlink
ln -s ../docs/file.txt ./shortcut.txt

# Link to directory (allowed for symlinks!)
ln -s /var/log logs
```

**Symbolic Link Properties:**

```bash
# Follow the symlink to read content
cat symlink.txt
# Output: Original content

# Modify via symlink
echo "Changed via symlink" > symlink.txt

# Original file modified
cat original.txt
# Output: Changed via symlink

# Delete original file
rm original.txt

# Symlink now broken (dangling)
cat symlink.txt
# Output: cat: symlink.txt: No such file or directory

# Symlink still exists but points nowhere
ls -l symlink.txt
# Output: lrwxrwxrwx 1 user user 12 Jan 15 10:30 symlink.txt -> original.txt
#         └─ Red color in terminal = broken link
```

**Finding Broken Symlinks:**

```bash
# Find all broken symbolic links
find . -type l ! -exec test -e {} \; -print

# Breakdown:
# -type l              → Only symbolic links
# ! -exec test -e {}   → NOT exists (! = NOT, test -e = exists test)
# -print               → Show the broken links

# Delete broken symlinks
find . -type l ! -exec test -e {} \; -delete
```

### Part 4: Complex Link Scenarios

#### Scenario 1: Application Deployment with Symlinks

```bash
# Common deployment pattern: zero-downtime releases

# Directory structure:
# /app/
# ├── releases/
# │   ├── release-001/
# │   ├── release-002/
# │   └── release-003/
# ├── shared/
# │   ├── logs/
# │   └── uploads/
# └── current → releases/release-003 (symlink)

# Setup:
mkdir -p /app/releases/{release-001,release-002,release-003}
mkdir -p /app/shared/{logs,uploads}

# Deploy release-001
ln -s /app/releases/release-001 /app/current

# Check what's live
ls -l /app/current
# Output: /app/current -> /app/releases/release-001

# Deploy release-002 (atomic switchover)
ln -sfn /app/releases/release-002 /app/current
# -f = force (overwrite existing symlink)
# -n = no-dereference (treat destination as normal file)

# Now current points to release-002
ls -l /app/current
# Output: /app/current -> /app/releases/release-002

# Rollback to release-001 instantly
ln -sfn /app/releases/release-001 /app/current

# Why this works:
# - Changing symlink is atomic (single filesystem operation)
# - No downtime during switch
# - Easy rollback
# - Keep multiple releases for quick switching
```

#### Scenario 2: Managing Configuration Versions

```bash
# Setup configuration versions
mkdir -p ~/configs/{production,staging,development}

# Create config files
echo "DB_HOST=prod.db.example.com" > ~/configs/production/database.conf
echo "DB_HOST=staging.db.example.com" > ~/configs/staging/database.conf
echo "DB_HOST=localhost" > ~/configs/development/database.conf

# Application reads from /etc/myapp/database.conf
# Use symlink to point to current environment

# Development environment
sudo ln -s ~/configs/development/database.conf /etc/myapp/database.conf

# Switch to staging
sudo ln -sfn ~/configs/staging/database.conf /etc/myapp/database.conf

# Switch to production
sudo ln -sfn ~/configs/production/database.conf /etc/myapp/database.conf

# Application always reads /etc/myapp/database.conf
# But actual config changes based on symlink target
```

#### Scenario 3: Shared Log Directories

```bash
# Multiple applications sharing log directory

# Setup:
mkdir -p /var/log/shared/{app1,app2,app3}
mkdir -p /app/{app1,app2,app3}

# Create symlinks from app dirs to shared log dir
ln -s /var/log/shared/app1 /app/app1/logs
ln -s /var/log/shared/app2 /app/app2/logs
ln -s /var/log/shared/app3 /app/app3/logs

# Now apps write to their local logs/ directory
echo "App1 log entry" >> /app/app1/logs/app.log

# But files actually stored in /var/log/shared/app1/
ls /var/log/shared/app1/
# Output: app.log

# Benefits:
# - Centralized log management
# - Apps don't need to know about /var/log structure
# - Easy to change log location (just update symlink)
# - Log rotation scripts can work on /var/log/shared/
```

#### Scenario 4: Version Management with Hard Links

```bash
# Efficient backups using hard links (same data, multiple names)

# Create original file (large)
dd if=/dev/urandom of=database.db bs=1M count=100
# Creates 100MB file

# Create "backup" using hard link
ln database.db database.db.backup-$(date +%Y%m%d)

# Check disk usage
du -sh database.db*
# Output:
# 100M database.db
# 100M database.db.backup-20240115
# Total: 100M (NOT 200M!)

# Both names point to same data blocks
# Disk space only used once

# Modify original
echo "New data" >> database.db

# Both files show update (same inode)
ls -li database.db*
# Same inode number, same size
```

### Part 5: Link Management Commands

#### Finding Links

```bash
# Find all symbolic links
find /path -type l

# Find all symbolic links pointing to specific file
find /path -lname "*original.txt"

# Find files with link count > 1 (hard links exist)
find /path -type f -links +1

# Show where symlink points
readlink symlink.txt
# Output: original.txt

# Show full path of symlink target
readlink -f symlink.txt
# Output: /home/user/original.txt
```

#### Updating Links

```bash
# Update symlink to point to new target
ln -sfn new-target.txt symlink.txt

# Create relative symlink
ln -sr /path/to/target /path/to/link
# -r = relative (creates relative path automatically)

# Force overwrite existing link
ln -sf target link  # Overwrites link if exists
```

#### Removing Links

```bash
# Remove symbolic link (does NOT remove target)
rm symlink.txt
# OR
unlink symlink.txt

# Remove hard link (decreases link count)
rm hardlink.txt
# File data deleted only when link count reaches 0

# Remove all hard links to a file
find . -samefile original.txt -delete
# Finds all links to same inode and deletes them
```

---

## Managing 100+ Files Simultaneously

### Part 1: Batch Selection Techniques

#### Using Wildcards Effectively

```bash
# Create test environment
mkdir -p ~/batch-test
cd ~/batch-test
touch file{001..200}.txt

# Select specific ranges
ls file{001..050}.txt  # First 50 files
ls file{051..100}.txt  # Next 50 files
ls file{101..150}.txt  # Files 101-150

# Select by pattern
ls file*1.txt  # All files ending in 1 (file001, file011, file021, ...)
ls file*5*.txt # All files containing 5

# Select by wildcards
ls file???.txt # Exactly 3 characters between "file" and ".txt"
ls file???[0-9].txt # Last character is a digit
```

#### Using Find for Complex Selection

```bash
# Select files by modification time
find . -name "file*.txt" -mtime -7  # Modified in last 7 days

# Select files by size
find . -name "file*.txt" -size +1k  # Larger than 1KB

# Select every Nth file (e.g., every 10th)
find . -name "file*.txt" | sed -n '0~10p'
# sed -n '0~10p' prints every 10th line

# Select random 50 files
find . -name "file*.txt" | shuf -n 50
# shuf = shuffle, -n 50 = pick 50
```

### Part 2: Batch Operations

#### Operation 1: Batch Delete

```bash
# Delete specific range
rm file{001..050}.txt

# Delete by pattern
rm file*1.txt

# Delete with confirmation
rm -i file{001..010}.txt
# -i prompts for each file

# Safe delete: move to trash directory first
mkdir -p ~/.trash
mv file{001..050}.txt ~/.trash/

# Find and delete (dry-run first!)
find . -name "file*.txt" -mtime +30  # Preview
find . -name "file*.txt" -mtime +30 -delete  # Actually delete
```

#### Operation 2: Batch Copy/Move

```bash
# Copy range to another directory
mkdir -p ~/backup
cp file{001..100}.txt ~/backup/

# Copy with structure preservation
find . -name "file*.txt" -exec cp --parents {} ~/backup/ \;
# --parents preserves directory structure

# Move files to organized directories by range
mkdir -p {batch1,batch2,batch3,batch4}
mv file{001..050}.txt batch1/
mv file{051..100}.txt batch2/
mv file{101..150}.txt batch3/
mv file{151..200}.txt batch4/

# Move files based on condition
find . -name "file*.txt" -size +10k -exec mv {} large-files/ \;
```

#### Operation 3: Batch Rename

```bash
# Add prefix to range
rename 's/^/processed-/' file{001..100}.txt

# Add date suffix
rename 's/$/-2024-01-15/' file{001..050}.txt

# Sequential renumbering
counter=1
for file in file*.txt; do
    mv "$file" "$(printf "newfile%04d.txt" $counter)"
    ((counter++))
done
# Result: file001.txt → newfile0001.txt, etc.
```

#### Operation 4: Batch Content Modification

```bash
# Add line to all files
for file in file{001..100}.txt; do
    echo "Processed: $(date)" >> "$file"
done

# Replace text in all files
find . -name "file*.txt" -exec sed -i 's/old/new/g' {} +
# sed -i = in-place edit

# Add header to all files
for file in file*.txt; do
    echo "=== Header ===" | cat - "$file" > temp && mv temp "$file"
done
```

### Part 3: Parallel Processing

#### Using xargs for Parallelization

```bash
# Process files in parallel (4 at a time)
find . -name "file*.txt" | xargs -P 4 -I {} gzip {}
# -P 4 = 4 parallel processes
# -I {} = placeholder for each file

# Batch operations in parallel
ls file*.txt | xargs -P 4 -n 25 tar czf batch.tar.gz
# -n 25 = 25 files per tar command
# Processes 4 batches simultaneously
```

#### Using GNU Parallel

```bash
# Install GNU parallel
sudo apt-get install parallel  # Debian/Ubuntu

# Process files in parallel
parallel gzip ::: file{001..100}.txt
# ::: separates command from arguments

# With progress bar
parallel --bar gzip ::: file{001..100}.txt

# Advanced: process with custom function
process_file() {
    file=$1
    echo "Processing $file"
    # Do complex operations
    gzip "$file"
}
export -f process_file
parallel process_file ::: file{001..100}.txt
```

### Part 4: Monitoring and Logging

#### Track Progress

```bash
# Count files before operation
total=$(ls file*.txt | wc -l)
echo "Total files: $total"

# Process with progress
counter=0
for file in file*.txt; do
    # Operation here
    gzip "$file"
    
    ((counter++))
    echo -ne "Progress: $counter/$total\r"
done
echo # Newline after progress

# Using pv (pipe viewer) for progress
ls file*.txt | pv -l -s $total | xargs -I {} gzip {}
# -l = line mode, -s = size (number of lines)
```

#### Logging Operations

```bash
# Log all operations
log_file="operations-$(date +%Y%m%d-%H%M%S).log"

for file in file*.txt; do
    echo "$(date): Processing $file" >> "$log_file"
    
    if gzip "$file" 2>>"$log_file"; then
        echo "$(date): Success - $file" >> "$log_file"
    else
        echo "$(date): Failed - $file" >> "$log_file"
    fi
done

# Review log
cat "$log_file"
```

### Part 5: Error Handling

#### Safe Batch Operations

```bash
# Check before deleting
files_to_delete=(file{001..050}.txt)
echo "About to delete ${#files_to_delete[@]} files:"
printf '%s\n' "${files_to_delete[@]}"
read -p "Continue? (y/n) " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
    rm "${files_to_delete[@]}"
fi

# Atomic operations with temporary directory
temp_dir=$(mktemp -d)
trap "rm -rf $temp_dir" EXIT  # Cleanup on exit

# Work in temp directory
cp file{001..100}.txt "$temp_dir/"
cd "$temp_dir"
# Do operations...
# If successful:
cp * ~/final-destination/
```

#### Handling Failures

```bash
# Continue on error
for file in file*.txt; do
    if ! gzip "$file" 2>/dev/null; then
        echo "Failed: $file" >> failed.log
    fi
done

# Stop on first error
set -e  # Exit on any error
for file in file*.txt; do
    gzip "$file"  # Will exit if fails
done
```

---

## Muscle Memory Drills

### Drill 1: File Creation Speed Test (5 minutes)

```bash
# Create clean environment
mkdir -p ~/drill1 && cd ~/drill1

# Drill: Create 100 files in under 5 seconds
time touch file{001..100}.txt

# Goal: < 0.5 seconds
# Expert: < 0.1 seconds

# Variations:
# 1. Different patterns
time touch {app,web,db}-{dev,prod}-{01..10}.log

# 2. Nested directories
time mkdir -p project/{src,test}/{js,css} && \
     touch project/{src,test}/{js,css}/file{1..5}.txt

# Repeat until commands flow naturally
```

### Drill 2: Rename Mastery (10 minutes)

```bash
# Setup
mkdir -p ~/drill2 && cd ~/drill2
touch file{001..100}.txt

# Round 1: Basic rename (30 seconds)
rename -n 's/file/doc/' file*.txt  # Preview
rename 's/file/doc/' file*.txt     # Execute

# Round 2: Extension change (30 seconds)
rename 's/\.txt$/.log/' doc*.txt

# Round 3: Add prefix (30 seconds)
rename 's/^/backup-/' doc*.log

# Round 4: Complex pattern (60 seconds)
rename 's/backup-doc(\d+)\.log/$1-doc-backup.log/' backup-*.log

# Round 5: Case conversion (30 seconds)
rename 'y/a-z/A-Z/' *.log

# Repeat until you can do all 5 rounds in under 3 minutes
```

### Drill 3: Link Creation Circuit (10 minutes)

```bash
# Setup
mkdir -p ~/drill3 && cd ~/drill3
echo "Original content" > original.txt

# Round 1: Hard links (15 seconds)
ln original.txt hard1.txt
ln original.txt hard2.txt
ln original.txt hard3.txt
ls -li *.txt  # Verify same inode

# Round 2: Symbolic links (15 seconds)
ln -s original.txt sym1.txt
ln -s original.txt sym2.txt
ln -s original.txt sym3.txt
ls -l sym*.txt  # Verify symlinks

# Round 3: Complex scenario (60 seconds)
mkdir -p {releases/v{1,2,3},shared}
echo "v1" > releases/v1/app.txt
echo "v2" > releases/v2/app.txt
echo "v3" > releases/v3/app.txt
ln -s releases/v3/app.txt current.txt
readlink current.txt  # Verify

# Round 4: Update links (30 seconds)
ln -sfn releases/v2/app.txt current.txt  # Switch version
ln -sfn releases/v1/app.txt current.txt  # Rollback

# Goal: Complete all rounds without errors in under 2 minutes
```

### Drill 4: Batch Operations (15 minutes)

```bash
# Setup
mkdir -p ~/drill4 && cd ~/drill4
touch file{0001..0200}.txt

# Task 1: Organize into 4 batches (60 seconds)
mkdir -p batch{1..4}
mv file{0001..0050}.txt batch1/
mv file{0051..0100}.txt batch2/
mv file{0101..0150}.txt batch3/
mv file{0151..0200}.txt batch4/

# Task 2: Rename all files in batch1 (45 seconds)
rename 's/^/processed-/' batch1/file*.txt

# Task 3: Create backups using hard links (60 seconds)
mkdir backups
find batch1 -name "*.txt" -exec ln {} backups/ \;

# Task 4: Create symlinks to current batch (45 seconds)
mkdir current
find batch4 -name "*.txt" -exec ln -s ../{} current/ \;

# Task 5: Clean up old batches (30 seconds)
rm -rf batch{1..3}

# Total time goal: < 4 minutes
```

### Drill 5: Real-World Scenario Simulation (20 minutes)

```bash
# Scenario: Production deployment with rollback capability

# Setup (60 seconds)
mkdir -p ~/deployment/{releases,shared/logs,backups}
cd ~/deployment

# Task 1: Create release structure (60 seconds)
release_id="release-$(date +%Y%m%d-%H%M%S)"
mkdir -p "releases/$release_id"/{app,config,data}

# Task 2: Deploy files (90 seconds)
touch "releases/$release_id"/app/app{1..50}.txt
touch "releases/$release_id"/config/config{1..10}.yml

# Task 3: Link shared resources (45 seconds)
ln -s ../../shared/logs "releases/$release_id/logs"

# Task 4: Activate release (30 seconds)
ln -sfn "releases/$release_id" current

# Task 5: Verify deployment (30 seconds)
ls -l current
find current -type f | wc -l  # Should be 60

# Task 6: Create backup (60 seconds)
backup_name="backup-$(date +%Y%m%d-%H%M%S)"
find "releases/$release_id" -type f -exec ln {} "backups/$backup_name-{}" \;

# Task 7: Simulate rollback (45 seconds)
# Assuming previous release exists
mkdir -p releases/release-previous/{app,config}
touch releases/release-previous/app/app{1..50}.txt
ln -sfn releases/release-previous current

# Total time goal: < 7 minutes
# Expert goal: < 5 minutes
```

---

## Real-World DevOps Use Cases

### Use Case 1: Log File Management

**Scenario:** Manage application logs across environments

```bash
# Setup log structure
mkdir -p /var/log/myapp/{production,staging,development}

# Generate log files (simulating applications)
for env in production staging development; do
    for i in {01..30}; do
        echo "Log entry $(date)" > "/var/log/myapp/$env/app-2024-01-$i.log"
    done
done

# Task 1: Compress old logs (older than 7 days)
find /var/log/myapp -name "*.log" -mtime +7 -exec gzip {} \;

# Task 2: Organize logs by date
for env in production staging development; do
    cd "/var/log/myapp/$env"
    for log in *.log 2>/dev/null; do
        date=$(echo "$log" | grep -oP '\d{4}-\d{2}-\d{2}')
        mkdir -p "$date"
        mv "$log" "$date/"
    done
done

# Task 3: Create current log symlinks
for env in production staging development; do
    latest=$(ls -t "/var/log/myapp/$env"/*.log 2>/dev/null | head -1)
    ln -sfn "$latest" "/var/log/myapp/$env/current.log"
done

# Task 4: Archive old logs
tar czf "/var/log/myapp/archive-$(date +%Y%m).tar.gz" \
    /var/log/myapp/*/????-??-??/

# Task 5: Clean up archived logs
find /var/log/myapp -type d -name "????-??-??" -mtime +90 -exec rm -rf {} +
```

**Why This Matters:**
- Prevents disk space exhaustion
- Maintains organized log history
- Quick access to current logs via symlinks
- Automated archival for compliance

### Use Case 2: Blue-Green Deployment

**Scenario:** Zero-downtime application deployment

```bash
# Setup deployment structure
mkdir -p /app/{blue,green,shared/{logs,uploads,config}}

# Initial deployment (Blue)
cp -r /tmp/app-v1/* /app/blue/
ln -sfn /app/blue /app/current

# Link shared resources
ln -s /app/shared/logs /app/blue/logs
ln -s /app/shared/uploads /app/blue/uploads
ln -s /app/shared/config /app/blue/config

# Deploy new version (Green) - while Blue is serving traffic
cp -r /tmp/app-v2/* /app/green/
ln -s /app/shared/logs /app/green/logs
ln -s /app/shared/uploads /app/green/uploads
ln -s /app/shared/config /app/green/config

# Test Green deployment
# ... run tests ...

# Switch traffic to Green (atomic operation)
ln -sfn /app/green /app/current

# If issues detected, instant rollback
ln -sfn /app/blue /app/current

# After successful Green deployment, prepare Blue for next release
rm -rf /app/blue/*
# Blue is now ready for next version
```

**Why This Matters:**
- Zero downtime during deployment
- Instant rollback capability
- Blue/Green stay synchronized via shared resources
- Clear separation of old and new versions

### Use Case 3: Configuration Management

**Scenario:** Manage configurations across multiple environments

```bash
# Setup configuration repository
mkdir -p ~/configs/{production,staging,development}/{app,db,web}

# Create environment-specific configs
cat > ~/configs/production/db/database.yml << EOF
host: prod.db.example.com
port: 5432
database: production_db
EOF

cat > ~/configs/staging/db/database.yml << EOF
host: staging.db.example.com
port: 5432
database: staging_db
EOF

cat > ~/configs/development/db/database.yml << EOF
host: localhost
port: 5432
database: dev_db
EOF

# Create versioned config links
for env in production staging development; do
    mkdir -p /etc/myapp/$env
    ln -s ~/configs/$env/db/database.yml /etc/myapp/$env/database.yml
done

# Switch environment by changing single symlink
ln -sfn /etc/myapp/production /etc/myapp/current

# Application reads from /etc/myapp/current/database.yml
# Change environment:
ln -sfn /etc/myapp/staging /etc/myapp/current

# Backup configs before changes
backup_dir=~/config-backups/$(date +%Y%m%d-%H%M%S)
mkdir -p "$backup_dir"
find ~/configs -name "*.yml" -exec cp --parents {} "$backup_dir" \;
```

### Use Case 4: Batch File Processing Pipeline

**Scenario:** Process uploaded files from users

```bash
# Setup processing pipeline
mkdir -p ~/pipeline/{incoming,processing,processed,failed}

# Simulate incoming files
for i in {001..100}; do
    echo "User data $i" > ~/pipeline/incoming/upload-$i.txt
done

# Process files in batches
batch_size=10
cd ~/pipeline/incoming

ls *.txt | while read -r file; do
    # Move to processing
    mv "$file" ../processing/
    
    # Simulate processing
    if process_file "../processing/$file"; then
        # Success - move to processed
        mv "../processing/$file" ../processed/
        
        # Create compressed archive
        gzip "../processed/$file"
        
        # Create link in archive directory
        mkdir -p ../archive/$(date +%Y-%m-%d)
        ln "../processed/$file.gz" "../archive/$(date +%Y-%m-%d)/"
    else
        # Failure - move to failed
        mv "../processing/$file" ../failed/
        
        # Log failure
        echo "$(date): Failed to process $file" >> ../failed.log
    fi
done

# Cleanup old processed files
find ~/pipeline/processed -name "*.gz" -mtime +30 -delete

# Generate processing report
echo "Processing Report - $(date)"
echo "Incoming: $(ls ~/pipeline/incoming/*.txt 2>/dev/null | wc -l)"
echo "Processing: $(ls ~/pipeline/processing/*.txt 2>/dev/null | wc -l)"
echo "Processed: $(ls ~/pipeline/processed/*.gz 2>/dev/null | wc -l)"
echo "Failed: $(ls ~/pipeline/failed/*.txt 2>/dev/null | wc -l)"
```

### Use Case 5: Backup and Disaster Recovery

**Scenario:** Incremental backups using hard links

```bash
# Setup backup structure
mkdir -p /backups/{daily,weekly,monthly}

# Initial full backup
backup_date=$(date +%Y-%m-%d)
mkdir -p "/backups/daily/$backup_date"
cp -r /data/* "/backups/daily/$backup_date/"

# Next day: incremental backup using hard links
previous_backup="/backups/daily/$backup_date"
new_backup_date=$(date +%Y-%m-%d)
new_backup="/backups/daily/$new_backup_date"

# Create new backup directory
mkdir -p "$new_backup"

# Copy only changed files, hard link unchanged
rsync -a --link-dest="$previous_backup" /data/ "$new_backup/"

# Why hard links save space:
# Day 1 backup: 10GB of data
# Day 2 backup: Only 100MB changed
# Disk usage: 10.1GB total (not 20GB!)
# Unchanged files hard-linked to Day 1 backup

# Weekly backup: promote daily to weekly
weekly_backup="/backups/weekly/week-$(date +%U)"
cp -rl "/backups/daily/$backup_date" "$weekly_backup"
# -rl = recursive + preserve links

# Monthly backup: promote weekly to monthly
monthly_backup="/backups/monthly/$(date +%Y-%m)"
cp -rl "$weekly_backup" "$monthly_backup"

# Cleanup old daily backups (keep last 7)
find /backups/daily -maxdepth 1 -type d -mtime +7 -exec rm -rf {} +

# Verify backup integrity
find "$new_backup" -type f -exec md5sum {} + > "$new_backup/checksums.md5"
```

### Use Case 6: CI/CD Artifact Management

**Scenario:** Manage build artifacts from CI/CD pipeline

```bash
# Setup artifact repository
mkdir -p ~/artifacts/{builds,releases,latest}

# Simulate CI/CD build creating artifacts
build_number="build-$(date +%Y%m%d-%H%M%S)"
build_dir=~/artifacts/builds/$build_number

mkdir -p "$build_dir"
# Copy build artifacts
cp /tmp/build-output/* "$build_dir/"

# Create versioned artifacts
for artifact in "$build_dir"/*; do
    filename=$(basename "$artifact")
    # Add version number
    versioned="${filename%.*}-v${build_number#build-}.${filename##*.}"
    cp "$artifact" "~/artifacts/releases/$versioned"
done

# Update "latest" symlinks
for artifact in "$build_dir"/*; do
    filename=$(basename "$artifact")
    ln -sfn "../builds/$build_number/$filename" \
            "~/artifacts/latest/$filename"
done

# Tag successful builds
mkdir -p ~/artifacts/tagged
ln -s "../builds/$build_number" \
      "~/artifacts/tagged/production-ready-$build_number"

# Cleanup old builds (keep last 50)
cd ~/artifacts/builds
ls -t | tail -n +51 | xargs rm -rf

# Generate artifact manifest
cat > "$build_dir/MANIFEST.txt" << EOF
Build: $build_number
Date: $(date)
Files:
$(find "$build_dir" -type f -exec ls -lh {} \;)
Checksums:
$(find "$build_dir" -type f -exec sha256sum {} \;)
EOF
```

---

## Advanced Techniques

### Technique 1: Atomic File Operations

```bash
# Problem: Moving files isn't atomic across filesystems
# Solution: Write to temp, then rename

# Wrong way (non-atomic):
cp source.txt /destination/source.txt  # Can fail mid-copy

# Right way (atomic):
temp_file=$(mktemp -p /destination)
cp source.txt "$temp_file"
mv "$temp_file" /destination/source.txt  # mv is atomic on same filesystem

# For critical operations:
atomic_write() {
    local content=$1
    local target=$2
    local temp=$(mktemp -p "$(dirname "$target")")
    
    echo "$content" > "$temp"
    chmod --reference="$target" "$temp" 2>/dev/null || chmod 644 "$temp"
    mv "$temp" "$target"
}

# Usage:
atomic_write "New config" /etc/app/config.yml
```

### Technique 2: Parallel Processing Patterns

```bash
# Process files in parallel with controlled concurrency
process_in_parallel() {
    local max_jobs=$1
    shift
    local files=("$@")
    
    for file in "${files[@]}"; do
        # Wait if max jobs running
        while [ $(jobs -r | wc -l) -ge $max_jobs ]; do
            sleep 0.1
        done
        
        # Process in background
        {
            echo "Processing $file"
            # Your processing here
            gzip "$file"
        } &
    done
    
    # Wait for all background jobs
    wait
}

# Usage:
process_in_parallel 4 file{001..100}.txt
```

### Technique 3: Safe Batch Deletion

```bash
# Never delete directly - use trash directory
safe_delete() {
    local trash_dir=~/.trash/$(date +%Y%m%d)
    mkdir -p "$trash_dir"
    
    for file in "$@"; do
        if [ -e "$file" ]; then
            mv "$file" "$trash_dir/"
            echo "Moved to trash: $file"
        fi
    done
}

# Restore from trash
restore_from_trash() {
    local trash_dir=~/.trash/$(date +%Y%m%d)
    local file=$1
    
    if [ -e "$trash_dir/$file" ]; then
        mv "$trash_dir/$file" .
        echo "Restored: $file"
    fi
}

# Auto-cleanup trash (older than 30 days)
find ~/.trash -type d -mtime +30 -exec rm -rf {} +
```

### Technique 4: Checksumming and Verification

```bash
# Create checksums for files
create_checksums() {
    local dir=$1
    find "$dir" -type f -exec sha256sum {} + > "$dir/checksums.sha256"
}

# Verify checksums
verify_checksums() {
    local dir=$1
    if [ -f "$dir/checksums.sha256" ]; then
        cd "$dir"
        sha256sum -c checksums.sha256
    fi
}

# Usage:
create_checksums /important-data
verify_checksums /important-data
```

### Technique 5: File Locking for Concurrent Access

```bash
# Prevent concurrent modifications
with_lock() {
    local lockfile=$1
    local command=$2
    
    # Create lock
    exec 200>"$lockfile"
    flock -n 200 || {
        echo "Another instance is running"
        return 1
    }
    
    # Execute command
    eval "$command"
    
    # Release lock
    flock -u 200
}

# Usage:
with_lock /tmp/myapp.lock "touch file{001..100}.txt"
```

---

## Keyboard Shortcuts Reference

```
┌────────────────────────────────────────────────────────┐
│  ESSENTIAL KEYBOARD SHORTCUTS                          │
├────────────────────────────────────────────────────────┤
│  FILE CREATION:                                        │
│    touch file{1..100}.txt           (15 keystrokes)    │
│    Alt+T (macro: touch file{1..100}.txt)               │
│                                                         │
│  NAVIGATION:                                           │
│    Ctrl+A  → Beginning of line                         │
│    Ctrl+E  → End of line                               │
│    Alt+B   → Back one word                             │
│    Alt+F   → Forward one word                          │
│                                                         │
│  EDITING:                                              │
│    Ctrl+U  → Delete to beginning                       │
│    Ctrl+K  → Delete to end                             │
│    Ctrl+W  → Delete word backward                      │
│    Alt+D   → Delete word forward                       │
│                                                         │
│  HISTORY:                                              │
│    Ctrl+R  → Reverse search                            │
│    Ctrl+P  → Previous command                          │
│    Ctrl+N  → Next command                              │
│    !!      → Repeat last command                       │
│    !$      → Last argument                             │
│                                                         │
│  COMPLETION:                                           │
│    Tab     → Complete filename                         │
│    Tab Tab → Show all completions                      │
│    Alt+*   → Insert all completions                    │
└────────────────────────────────────────────────────────┘
```

---

## Common Mistakes and Solutions

```
┌────────────────────────────────────────────────────────────┐
│  MISTAKE 1: Creating files with spaces in names            │
│  ❌ touch file 001.txt                                     │
│     Creates: "file" and "001.txt" (two files!)             │
│  ✅ touch "file 001.txt"                                   │
│     Creates: "file 001.txt" (one file)                     │
│  Better: Avoid spaces, use underscores/hyphens             │
│                                                             │
│  MISTAKE 2: Overwriting files with rename                  │
│  ❌ rename 's/\d+/001/' file*.txt                          │
│     All become file001.txt - data lost!                    │
│  ✅ rename -n 's/\d+/001/' file*.txt                       │
│     Test first with --dry-run                              │
│                                                             │
│  MISTAKE 3: Breaking symlinks                              │
│  ❌ mv original.txt /new/location/                         │
│     Symlinks still point to old location - now broken      │
│  ✅ Update symlinks after moving:                          │
│     ln -sfn /new/location/original.txt symlink.txt         │
│                                                             │
│  MISTAKE 4: Deleting hard link thinking it's a copy        │
│  ❌ rm hardlink.txt  # Decreases link count               │
│     Data persists (link count still > 0)                   │
│  ✅ Check link count:                                      │
│     ls -l original.txt  # Look at link count column        │
│                                                             │
│  MISTAKE 5: Not quoting variables with filenames           │
│  ❌ for file in *.txt; do mv $file processed-$file; done   │
│     Breaks with spaces in filenames                        │
│  ✅ for file in *.txt; do mv "$file" "processed-$file"; done│
│     Always quote variables                                 │
│                                                             │
│  MISTAKE 6: Using rm instead of safer alternatives         │
│  ❌ rm -rf important-files/                                │
│     No recovery possible                                   │
│  ✅ mv important-files ~/.trash/                           │
│     Can recover if needed                                  │
│                                                             │
│  MISTAKE 7: Forgetting to test find -delete                │
│  ❌ find . -name "*.log" -delete                           │
│     Immediately deletes - no undo                          │
│  ✅ find . -name "*.log" -print  # Test first              │
│     find . -name "*.log" -delete # Then delete             │
└────────────────────────────────────────────────────────────┘
```

---

## Cheat Sheet

```bash
# FILE CREATION
touch file{001..100}.txt              # 100 sequential files
touch {app,web,db}-{dev,prod}.log     # Combination (6 files)
seq -w 1 100 | xargs -I{} touch file{}.txt  # Alternative

# BATCH RENAME
rename 's/old/new/' *.txt             # Simple replacement
rename 's/\.txt$/.log/' *.txt         # Change extension
rename 's/^/prefix-/' *.txt           # Add prefix
rename 's/$/-suffix/' *.txt           # Add suffix (before ext)
rename 'y/a-z/A-Z/' *.txt             # Uppercase all

# PARAMETER EXPANSION
${file%.txt}                          # Remove .txt from end
${file#prefix-}                       # Remove prefix- from start
${file/old/new}                       # Replace first occurrence
${file//old/new}                      # Replace all occurrences
${file^^}                             # Uppercase all
${file,,}                             # Lowercase all

# HARD LINKS
ln source.txt link.txt                # Create hard link
ls -li *.txt                          # Show inode numbers
find . -samefile original.txt         # Find all hard links

# SYMBOLIC LINKS
ln -s target link                     # Create symlink
ln -sfn new-target link               # Update symlink
readlink link                         # Show where symlink points
find . -type l                        # Find all symlinks
find . -type l ! -exec test -e {} \; -print  # Find broken symlinks

# BATCH OPERATIONS
cp file{001..050}.txt ~/backup/       # Copy range
mv file{051..100}.txt ~/archive/      # Move range
rm file{001..010}.txt                 # Delete range
find . -name "*.txt" -exec gzip {} +  # Compress all

# PARALLEL PROCESSING
xargs -P 4 -I {} command {}           # 4 parallel jobs
parallel command ::: file*.txt        # GNU parallel

# MONITORING
ls *.txt | wc -l                      # Count files
du -sh directory/                     # Directory size
find . -name "*.txt" | wc -l          # Count specific files
```

---

## Final Mastery Test (30 minutes)

```bash
# Test yourself - complete all tasks, time yourself

# SETUP
mkdir -p ~/mastery-test && cd ~/mastery-test

# TASK 1 (2 minutes): Create 500 files with zero-padded names
touch file{0001..0500}.txt
# Verify: ls | wc -l  # Should output 500

# TASK 2 (3 minutes): Rename half with prefix "processed-"
rename 's/^/processed-/' file{0001..0250}.txt
# Verify: ls processed-* | wc -l  # Should output 250

# TASK 3 (3 minutes): Change extension of remaining files
rename 's/\.txt$/.log/' file{0251..0500}.txt
# Verify: ls *.log | wc -l  # Should output 250

# TASK 4 (4 minutes): Create deployment structure
mkdir -p deploy/releases/{v1,v2,v3}
cp processed-file{0001..0100}.txt deploy/releases/v1/
cp processed-file{0101..0200}.txt deploy/releases/v2/
cp processed-file{0201..0250}.txt deploy/releases/v3/

# TASK 5 (2 minutes): Create symlink to active version
ln -s releases/v3 deploy/current

# TASK 6 (3 minutes): Create hard link backups
mkdir backups
find deploy/releases/v3 -type f -exec ln {} backups/ \;

# TASK 7 (4 minutes): Organize logs by pattern
mkdir -p logs/{app,system,error}
mv file{0251..0350}.log logs/app/
mv file{0351..0450}.log logs/system/
mv file{0451..0500}.log logs/error/

# TASK 8 (3 minutes): Batch rename logs with date prefix
for dir in logs/*; do
    rename "s/^/$(date +%Y%m%d)-/" "$dir"/*.log
done

# TASK 9 (3 minutes): Create archive of each log category
cd logs
for dir in */; do
    tar czf "${dir%/}-$(date +%Y%m%d).tar.gz" "$dir"
done
cd ..

# TASK 10 (3 minutes): Cleanup and verification
# Switch deployment to v2
ln -sfn releases/v2 deploy/current
# Verify backups
ls -li backups/ | head -5
# Count total files created
find . -type f | wc -l  # Should be reasonable number

# SCORING:
# < 20 min: Master level
# 20-25 min: Advanced
# 25-30 min: Intermediate
# > 30 min: Need more practice
```

---

## Conclusion and Next Steps

### You've Mastered:
1. ✅ Creating 1000+ files efficiently
2. ✅ Mass renaming with rename and parameter expansion
3. ✅ Complex hard link and symbolic link scenarios
4. ✅ Managing 100+ files simultaneously

### Daily Practice Routine (15 minutes):

```bash
# Day 1-7: File creation drills
touch file{0001..1000}.txt && rm *.txt  # Repeat 5 times

# Day 8-14: Rename mastery
# Create, rename with different patterns, verify

# Day 15-21: Link scenarios
# Practice hard/soft links, verify behavior, break/fix links

# Day 22-28: Batch operations
# Create complex scenarios, practice parallel processing

# Day 29+: Real-world simulations
# Combine all skills in deployment/backup/log management scenarios
```

### Advanced Learning Path:

```bash
# Learn rsync for advanced file synchronization
man rsync

# Study inotify for file system monitoring
man inotifywait

# Explore advanced parallel processing
man parallel

# Master filesystem tools
man find xargs sed awk
```

### Track Your Progress:

```bash
# Create practice log
echo "$(date): Completed file operations bootcamp" >> ~/devops-practice.log

# Set up weekly challenges
# Week 1: Complete mastery test in < 20 minutes
# Week 2: Create custom deployment automation
# Week 3: Build log processing pipeline
# Week 4: Implement backup system with hard links
```

**Remember:** File operations are fundamental to DevOps. Master these skills, and complex deployment, backup, and management tasks become simple, repeatable, and reliable.

**Pro Tip:** Real mastery comes from handling edge cases gracefully. Always test operations with `-n` (dry-run), verify results, and maintain backups before bulk operations.