# Permission Engineering Mastery Guide for Ubuntu

## Table of Contents
1. [Foundation Concepts](#foundation-concepts)
2. [Basic Permission System](#basic-permission-system)
3. [Advanced ACLs](#advanced-acls)
4. [Multi-User Project Setup](#multi-user-project-setup)
5. [Real-World Scenarios](#real-world-scenarios)
6. [Troubleshooting & Speed Drills](#troubleshooting-speed-drills)

---

## Foundation Concepts

### The Permission Problem in DevOps

```
┌────────────────────────────────────────────────────┐
│  REAL-WORLD SCENARIO:                              │
│                                                     │
│  Team: 3 developers, 2 DevOps engineers           │
│  Need: Shared project directory                    │
│  Requirements:                                      │
│    ✓ Everyone can create files                     │
│    ✓ Can't delete others' files                    │
│    ✓ Specific users get special access            │
│    ✓ Audit who changed what                       │
│    ✓ Prevent accidental data loss                 │
└────────────────────────────────────────────────────┘
```

### Permission Layers in Linux

```
┌─────────────────────────────────────────────────────┐
│                 PERMISSION LAYERS                    │
├─────────────────────────────────────────────────────┤
│                                                      │
│  LAYER 1: Basic Permissions (chmod/chown)           │
│  ├─ Owner (u)                                       │
│  ├─ Group (g)                                       │
│  └─ Others (o)                                      │
│                                                      │
│  LAYER 2: Special Bits                              │
│  ├─ SUID (Set User ID)                              │
│  ├─ SGID (Set Group ID)                             │
│  └─ Sticky Bit (t)                                  │
│                                                      │
│  LAYER 3: Access Control Lists (ACLs)               │
│  ├─ User-specific permissions                       │
│  ├─ Group-specific permissions                      │
│  ├─ Default permissions                             │
│  └─ Mask controls                                   │
│                                                      │
│  LAYER 4: Extended Attributes (xattr)               │
│  └─ File capabilities, SELinux contexts             │
└─────────────────────────────────────────────────────┘
```

### Visual Permission Breakdown

```
FILE: -rwxr-xr--  2 alice  devteam  4096  Jan 15 10:30  deploy.sh
       │││││││││  │   │      │       │       │           │
       │││││││││  │   │      │       │       │           └─ Filename
       │││││││││  │   │      │       │       └─ Modified time
       │││││││││  │   │      │       └─ Size in bytes
       │││││││││  │   │      └─ Group owner
       │││││││││  │   └─ User owner
       │││││││││  └─ Link count
       │││││││││
       ││││││││└─ Others: r-- (read only)
       │││││││└── Others: x (no execute)
       ││││││└─── Group: r-- (read only)
       │││││└──── Group: x (execute)
       ││││└───── Group: - (no write)
       │││└────── Owner: r (read)
       ││└─────── Owner: w (write)
       │└──────── Owner: x (execute)
       └───────── File type: - (regular file)
                              d (directory)
                              l (symlink)
```

---

## Basic Permission System

### Part 1: Understanding Permission Numbers

#### The Octal System Explained

```
┌──────────────────────────────────────────────────┐
│  PERMISSION CALCULATION:                         │
├──────────────────────────────────────────────────┤
│                                                   │
│  Each permission has a value:                    │
│    r (read)    = 4                               │
│    w (write)   = 2                               │
│    x (execute) = 1                               │
│    - (none)    = 0                               │
│                                                   │
│  Add them up:                                    │
│    rwx = 4+2+1 = 7                               │
│    rw- = 4+2+0 = 6                               │
│    r-x = 4+0+1 = 5                               │
│    r-- = 4+0+0 = 4                               │
│    -wx = 0+2+1 = 3                               │
│    -w- = 0+2+0 = 2                               │
│    --x = 0+0+1 = 1                               │
│    --- = 0+0+0 = 0                               │
│                                                   │
│  Three digits: owner group others                │
│    755 = rwxr-xr-x                               │
│    644 = rw-r--r--                               │
│    600 = rw-------                               │
└──────────────────────────────────────────────────┘
```

#### Common Permission Patterns

```bash
# View permissions
ls -l filename

# Output example:
-rw-r--r--  1 alice devteam 1234 Jan 15 10:30 config.yml
```

**Understanding each part:**
```
- : File type (- = regular, d = directory, l = link)
rw- : Owner can read and write
r-- : Group can only read
r-- : Others can only read
1 : Number of hard links
alice : Owner username
devteam : Group name
1234 : File size in bytes
Jan 15 10:30 : Last modification time
config.yml : Filename
```

### Part 2: chmod - Change Permissions

#### Numeric Method

```bash
# Syntax: chmod [permissions] [file]

# Set exact permissions
chmod 755 script.sh
# Result: rwxr-xr-x
# Owner: read, write, execute (7)
# Group: read, execute (5)
# Others: read, execute (5)

chmod 644 config.txt
# Result: rw-r--r--
# Owner: read, write (6)
# Group: read (4)
# Others: read (4)

chmod 600 secret.key
# Result: rw-------
# Owner: read, write (6)
# Group: nothing (0)
# Others: nothing (0)

# Recursive (all files/subdirs)
chmod -R 755 /var/www/html/
# -R flag applies to all contents
```

**Why these numbers?**
```
755 - Standard for directories and executables
      (Owner full access, others read/execute)
644 - Standard for regular files
      (Owner read/write, others read-only)
600 - Private files (credentials, keys)
      (Only owner can access)
700 - Private directories
      (Only owner can access)
```

#### Symbolic Method

```bash
# Syntax: chmod [who][operation][permission] [file]
# who: u(user/owner), g(group), o(others), a(all)
# operation: +(add), -(remove), =(set exactly)
# permission: r(read), w(write), x(execute)

# Add execute for owner
chmod u+x script.sh
# Before: rw-r--r--
# After:  rwxr--r--

# Remove write for group and others
chmod go-w file.txt
# Before: rw-rw-rw-
# After:  rw-r--r--

# Set exact permissions for all
chmod a=rx script.sh
# After: r-xr-xr-x (all have read+execute, no write)

# Add execute for owner, remove write for others
chmod u+x,o-w file.txt

# Make file executable by everyone
chmod a+x deploy.sh
```

**Common Symbolic Patterns:**
```bash
chmod u+x file      # Make executable for owner
chmod g+w file      # Add write for group
chmod o-rwx file    # Remove all permissions for others
chmod a+r file      # Make readable by everyone
chmod ug+rw file    # Owner and group: read+write
```

### Part 3: chown - Change Ownership

```bash
# Syntax: chown [owner]:[group] [file]

# Change owner only
sudo chown alice file.txt
# User owner changes to 'alice'
# Group stays the same

# Change owner and group
sudo chown alice:devteam file.txt
# User: alice
# Group: devteam

# Change group only (colon prefix)
sudo chown :devteam file.txt
# User stays the same
# Group: devteam

# Alternative: use chgrp
sudo chgrp devteam file.txt

# Recursive ownership change
sudo chown -R alice:devteam /project/
# Changes owner/group for directory and all contents

# Change with reference to another file
sudo chown --reference=template.txt newfile.txt
# Copies ownership from template.txt to newfile.txt
```

**Real-world example:**
```bash
# Web server files
sudo chown -R www-data:www-data /var/www/html/
# www-data is the user that Apache/Nginx runs as
# This ensures the web server can read/write its files
```

### Part 4: Special Permission Bits

#### SUID (Set User ID) - 4

```bash
# When set on executable: runs with owner's permissions
# Represented by 's' in owner execute position

# Set SUID
chmod u+s program
# OR
chmod 4755 program

# View SUID files
ls -l /usr/bin/passwd
# -rwsr-xr-x (note the 's' instead of 'x')

# Why: /usr/bin/passwd needs to modify /etc/shadow (root-only)
# Regular users run passwd with root permissions temporarily
```

**Example:**
```bash
# File owned by root with SUID
-rwsr-xr-x  1 root root 54256 Mar 23  2023 /usr/bin/passwd

# When alice runs it:
# - Process runs with root privileges (not alice's)
# - Can modify /etc/shadow
# - Returns to alice's privileges when complete
```

**Security Warning:**
```
⚠️  SUID is dangerous if misused!
Never set SUID on:
  - Shell scripts (bash, sh)
  - Text editors (vim, nano)
  - Interpreters (python, perl)
Why: Could allow privilege escalation attacks
```

#### SGID (Set Group ID) - 2

**On Files:**
```bash
# Executable runs with group's permissions
chmod g+s program
# OR
chmod 2755 program

# Represented by 's' in group execute position
-rwxr-sr-x
```

**On Directories (MORE COMMON):**
```bash
# New files inherit directory's group (not creator's primary group)
chmod g+s /shared/project/

# Example:
mkdir /shared/project
chgrp devteam /shared/project
chmod 2775 /shared/project

# alice creates file:
touch /shared/project/alice-file.txt
ls -l /shared/project/alice-file.txt
# -rw-r--r-- 1 alice devteam 0 Jan 15 10:30 alice-file.txt
#                    ^^^^^^^ Inherited from directory!
```

**Why SGID on directories matters:**
```
WITHOUT SGID:
  alice (primary group: alice) creates file
  → File group: alice
  → Bob (in devteam) can't access it

WITH SGID:
  alice creates file in SGID directory
  → File group: devteam
  → Bob (in devteam) can access it ✓
```

#### Sticky Bit - 1

```bash
# On directories: only file owner can delete their files
# Even if directory is world-writable!

chmod +t /shared/uploads/
# OR
chmod 1777 /shared/uploads/

# Represented by 't' in others execute position
drwxrwxrwt

# Classic example: /tmp
ls -ld /tmp
# drwxrwxrwt 20 root root 4096 Jan 15 10:30 /tmp
```

**Sticky Bit Demonstration:**
```bash
# Create shared upload directory
sudo mkdir /shared/uploads
sudo chmod 1777 /shared/uploads

# alice creates file:
sudo -u alice touch /shared/uploads/alice-upload.txt

# bob tries to delete alice's file:
sudo -u bob rm /shared/uploads/alice-upload.txt
# rm: cannot remove 'alice-upload.txt': Operation not permitted

# alice can delete her own file:
sudo -u alice rm /shared/uploads/alice-upload.txt
# Success!
```

**Combined Special Bits:**
```bash
# All three: SUID + SGID + Sticky
chmod 7755 file
# -rwsr-sr-t

# SGID + Sticky (common for shared dirs)
chmod 3775 /shared/
# drwxrwsr-t
```

### Part 5: Default Permissions (umask)

```bash
# View current umask
umask
# Output: 0022

# Umask calculation:
# Files start at:       666 (rw-rw-rw-)
# Directories start at: 777 (rwxrwxrwx)
# Subtract umask:      -022
# File result:          644 (rw-r--r--)
# Directory result:     755 (rwxr-xr-x)
```

**Visual Calculation:**
```
File default:    666
Umask:          -022
Result:          644

  6   6   6
- 0   2   2
---------
  6   4   4  = rw-r--r--

Directory:       777
Umask:          -022
Result:          755

  7   7   7
- 0   2   2
---------
  7   5   5  = rwxr-xr-x
```

**Setting umask:**
```bash
# More restrictive (private files)
umask 077
# Files: 600 (rw-------)
# Dirs:  700 (rwx------)

# Standard (group readable)
umask 022
# Files: 644 (rw-r--r--)
# Dirs:  755 (rwxr-xr-x)

# Permissive (group writable)
umask 002
# Files: 664 (rw-rw-r--)
# Dirs:  775 (rwxrwxr-x)

# Make permanent (add to ~/.bashrc)
echo "umask 027" >> ~/.bashrc
```

---

## Advanced ACLs

### Part 1: Understanding ACLs

```
┌────────────────────────────────────────────────────┐
│  WHY ACLs?                                         │
├────────────────────────────────────────────────────┤
│                                                     │
│  Problem: Basic permissions only support:          │
│    1 owner + 1 group + others                      │
│                                                     │
│  Scenario:                                         │
│    File owned by alice, group devteam              │
│    Need: Bob (not in devteam) should read         │
│          Charlie (in devteam) should NOT read      │
│                                                     │
│  Solution: ACLs provide per-user/per-group control│
└────────────────────────────────────────────────────┘
```

### Part 2: Installing ACL Support

```bash
# Install ACL tools
sudo apt-get update
sudo apt-get install acl

# Check if filesystem supports ACLs
mount | grep acl
# Should show 'acl' in mount options

# If not, enable ACLs on filesystem
sudo tune2fs -o acl /dev/sda1
# Or add 'acl' to /etc/fstab and remount

# Verify installation
which getfacl setfacl
# Should show paths: /usr/bin/getfacl and /usr/bin/setfacl
```

### Part 3: Viewing ACLs (getfacl)

```bash
# View ACLs on file
getfacl filename

# Example output:
# file: config.yml
# owner: alice
# group: devteam
# user::rw-           ← Owner permissions
# user:bob:r--        ← User bob has read
# group::r--          ← Group permissions
# mask::r--           ← Effective permissions mask
# other::---          ← Others permissions

# View ACLs on directory (recursive)
getfacl -R /project/

# Compact output (one line per file)
getfacl -c filename

# Skip base entries (show only extended ACLs)
getfacl --omit-header filename
```

**Understanding ACL Output:**
```
user::rw-          ← user:: = owner (alice)
user:bob:r--       ← user:bob = specific user ACL
user:charlie:rw-   ← user:charlie = another specific user
group::r--         ← group:: = owning group (devteam)
group:ops:rwx      ← group:ops = specific group ACL
mask::rw-          ← Maximum effective permissions
other::---         ← Everyone else
```

### Part 4: Setting ACLs (setfacl)

#### Basic Syntax

```bash
# setfacl -m [entry] [file]
# -m = modify
# entry format: [type]:[name]:[permissions]

# Grant user bob read access
setfacl -m u:bob:r file.txt

# Grant group ops read+write
setfacl -m g:ops:rw file.txt

# Remove specific ACL entry
setfacl -x u:bob file.txt

# Remove all ACLs (keep base permissions)
setfacl -b file.txt

# Recursive application
setfacl -R -m u:bob:rx /project/
```

**Complete Example:**
```bash
# Create test file
touch report.txt
ls -l report.txt
# -rw-r--r-- 1 alice alice 0 Jan 15 10:30 report.txt

# Add ACL for bob (read+write)
setfacl -m u:bob:rw report.txt

# Add ACL for charlie (read only)
setfacl -m u:charlie:r report.txt

# View result
ls -l report.txt
# -rw-rw-r--+ 1 alice alice 0 Jan 15 10:30 report.txt
#           ^ Plus sign indicates ACLs present

getfacl report.txt
# user::rw-
# user:bob:rw-
# user:charlie:r--
# group::r--
# mask::rw-
# other::r--
```

#### Permission Symbols in ACLs

```bash
# Read, Write, Execute
setfacl -m u:bob:rwx file

# Individual permissions
setfacl -m u:bob:r file    # Read only
setfacl -m u:bob:w file    # Write only
setfacl -m u:bob:x file    # Execute only

# Combinations
setfacl -m u:bob:rw file   # Read + Write
setfacl -m u:bob:rx file   # Read + Execute

# No permissions
setfacl -m u:bob:- file    # Explicitly deny (sets to ---)

# Numeric format (like chmod)
setfacl -m u:bob:6 file    # 6 = rw- (4+2)
setfacl -m u:bob:7 file    # 7 = rwx (4+2+1)
setfacl -m u:bob:5 file    # 5 = r-x (4+1)
```

### Part 5: Default ACLs

**Understanding Default ACLs:**
```
Default ACLs apply to DIRECTORIES only
  → Set permissions for NEW files/subdirectories created inside
  → Inherited by children
  → Don't affect existing files
```

**Setting Default ACLs:**
```bash
# Set default ACL on directory
setfacl -d -m u:bob:rwx /project/
# -d = default

# Now create file in that directory
touch /project/newfile.txt

# Check ACLs
getfacl /project/newfile.txt
# user::rw-
# user:bob:rwx      ← Inherited from default ACL!
# group::r--
# mask::rwx
# other::r--

# View default ACLs on directory
getfacl /project/
# Output shows both access and default:
# user::rwx
# group::r-x
# other::r-x
# default:user::rwx
# default:user:bob:rwx    ← Default ACL
# default:group::r-x
# default:other::r-x
```

**Practical Example:**
```bash
# Setup shared project directory
sudo mkdir /shared/project
sudo chgrp devteam /shared/project
sudo chmod 2775 /shared/project

# Set default ACLs (all new files accessible to team)
sudo setfacl -d -m u::rwx /shared/project/
sudo setfacl -d -m g::rwx /shared/project/
sudo setfacl -d -m o::r-x /shared/project/

# Set specific user access
sudo setfacl -d -m u:bob:rwx /shared/project/
sudo setfacl -d -m u:charlie:r-x /shared/project/

# Now alice creates file:
sudo -u alice touch /shared/project/newfile.txt

# Check permissions:
getfacl /shared/project/newfile.txt
# user::rw-         (alice - owner)
# user:bob:rwx      (bob - from default ACL)
# user:charlie:r-x  (charlie - from default ACL)
# group::rwx        (devteam - from default ACL)
```

### Part 6: ACL Mask

**What is the Mask?**
```
The mask defines MAXIMUM effective permissions for:
  - Named users (user:name:...)
  - Named groups (group:name:...)
  - Owning group (group::...)

Does NOT affect:
  - Owner (user::...)
  - Others (other::...)
```

**Mask Calculation:**
```bash
# Example ACL:
user:bob:rwx     (permissions granted: rwx)
mask::r--        (maximum allowed: r--)
# Effective: r-- (rwx & r-- = r--)

# View effective permissions
getfacl file.txt
# user:bob:rwx    #effective:r--
#                 ^^^^^^^^^^^^^^ Shows actual permissions after mask
```

**Setting the Mask:**
```bash
# Set mask explicitly
setfacl -m m::rx file.txt
# Now all ACL users/groups limited to read+execute max

# Recalculate mask automatically
setfacl -n -m u:bob:rwx file.txt
# -n = don't recalculate mask

# Without -n, setfacl recalculates mask to allow new permissions
setfacl -m u:bob:rwx file.txt
# Mask automatically adjusted to rwx
```

**Why Mask Matters:**
```bash
# Scenario: Temporarily restrict all ACL access

# Current state:
# user:bob:rwx
# user:charlie:rw-
# user:dave:r--

# Set restrictive mask
setfacl -m m::r-- file.txt

# All users now limited to read:
# user:bob:rwx      #effective:r--
# user:charlie:rw-  #effective:r--
# user:dave:r--     #effective:r--

# Later, restore access by removing mask restriction
setfacl -m m::rwx file.txt
```

### Part 7: Copying and Moving ACLs

```bash
# Copy ACLs from one file to another
getfacl file1.txt | setfacl --set-file=- file2.txt

# Copy ACLs to multiple files
getfacl template.txt | setfacl --set-file=- file1.txt file2.txt file3.txt

# Backup ACLs to file
getfacl -R /project/ > /backup/project-acls.txt

# Restore ACLs from backup
setfacl --restore=/backup/project-acls.txt

# Apply ACLs from file
setfacl --set-file=acl-template.txt /project/
```

**ACL Template File Format:**
```bash
# Create ACL template
cat > acl-template.txt << EOF
user::rwx
user:bob:rwx
user:charlie:r-x
group::r-x
group:devteam:rwx
mask::rwx
other::r-x
EOF

# Apply template
setfacl --set-file=acl-template.txt /project/
```

---

## Multi-User Project Setup

### Part 1: Planning the Structure

**Requirements Analysis:**
```
┌────────────────────────────────────────────────────┐
│  PROJECT REQUIREMENTS:                             │
├────────────────────────────────────────────────────┤
│  Users:                                            │
│    - alice (developer)                             │
│    - bob (developer)                               │
│    - charlie (ops engineer)                        │
│    - dave (manager - read only)                    │
│                                                     │
│  Directories:                                      │
│    /project/code         - devs: rw, ops: r       │
│    /project/deploy       - devs: r, ops: rw       │
│    /project/shared       - all: rw, can't delete  │
│                           others' files            │
│    /project/secrets      - ops only               │
│                                                     │
│  Goals:                                            │
│    ✓ Developers collaborate on code               │
│    ✓ Ops deploy but don't modify code             │
│    ✓ Everyone shares documents safely             │
│    ✓ Secrets isolated to ops team                 │
└────────────────────────────────────────────────────┘
```

### Part 2: User and Group Setup

```bash
# Create groups
sudo groupadd developers
sudo groupadd ops
sudo groupadd project-users

# Create users
sudo useradd -m -s /bin/bash -G developers,project-users alice
sudo useradd -m -s /bin/bash -G developers,project-users bob
sudo useradd -m -s /bin/bash -G ops,project-users charlie
sudo useradd -m -s /bin/bash -G project-users dave

# Set passwords (for testing)
echo "alice:password123" | sudo chpasswd
echo "bob:password123" | sudo chpasswd
echo "charlie:password123" | sudo chpasswd
echo "dave:password123" | sudo chpasswd

# Verify group memberships
groups alice
# alice : alice developers project-users

id alice
# uid=1001(alice) gid=1001(alice) groups=1001(alice),1002(developers),1004(project-users)
```

### Part 3: Directory Structure Creation

```bash
# Create base directory
sudo mkdir -p /project/{code,deploy,shared,secrets}

# Set base permissions
sudo chmod 755 /project

# View structure
tree -L 2 -p /project
# /project
# ├── [drwxr-xr-x]  code
# ├── [drwxr-xr-x]  deploy
# ├── [drwxr-xr-x]  shared
# └── [drwxr-xr-x]  secrets
```

### Part 4: Implementing /project/code

**Requirements:**
- Developers: read + write + execute
- Ops: read + execute
- Manager: read only
- New files inherit group

```bash
# Set ownership and base permissions
sudo chown root:developers /project/code
sudo chmod 2775 /project/code
# 2 = SGID (new files inherit 'developers' group)
# 775 = rwxrwxr-x

# Set default ACLs for new files
sudo setfacl -d -m u::rwx /project/code
sudo setfacl -d -m g::rwx /project/code
sudo setfacl -d -m o::r-x /project/code

# Grant charlie (ops) read access
sudo setfacl -m u:charlie:r-x /project/code
sudo setfacl -d -m u:charlie:r-x /project/code

# Grant dave (manager) read access
sudo setfacl -m u:dave:r-- /project/code
sudo setfacl -d -m u:dave:r-- /project/code

# Verify
getfacl /project/code
```

**Expected Output:**
```
# file: project/code
# owner: root
# group: developers
# flags: -s-           ← SGID flag
user::rwx
user:charlie:r-x
user:dave:r--
group::rwx
mask::rwx
other::r-x
default:user::rwx
default:user:charlie:r-x
default:user:dave:r--
default:group::rwx
default:mask::rwx
default:other::r-x
```

**Test:**
```bash
# alice creates file
sudo -u alice touch /project/code/app.py

# Check ownership
ls -l /project/code/app.py
# -rw-rwxr--+ 1 alice developers 0 Jan 15 10:30 app.py
#                   ^^^^^^^^^^ SGID worked!

# Check ACLs
getfacl /project/code/app.py
# user::rw-
# user:charlie:r-x
# user:dave:r--
# group::rwx
# mask::rwx
# other::r-x

# Verify charlie can read
sudo -u charlie cat /project/code/app.py
# Success!

# Verify charlie can't write
sudo -u charlie echo "test" >> /project/code/app.py
# Permission denied ✓
```

### Part 5: Implementing /project/deploy

**Requirements:**
- Ops: read + write + execute
- Developers: read only
- New files owned by ops

```bash
# Set ownership and permissions
sudo chown root:ops /project/deploy
sudo chmod 2770 /project/deploy
# 2 = SGID
# 770 = rwxrwx--- (ops full access, others none)

# Set default ACLs
sudo setfacl -d -m u::rwx /project/deploy
sudo setfacl -d -m g::rwx /project/deploy
sudo setfacl -d -m o::--- /project/deploy

# Grant developers read access
sudo setfacl -m g:developers:r-x /project/deploy
sudo setfacl -d -m g:developers:r-x /project/deploy

# Verify
getfacl /project/deploy
```

**Test:**
```bash
# charlie creates deployment script
sudo -u charlie touch /project/deploy/deploy.sh
sudo -u charlie chmod +x /project/deploy/deploy.sh

# alice (developer) can read
sudo -u alice ls /project/deploy/
# deploy.sh

# alice can't write
sudo -u alice touch /project/deploy/test.txt
# Permission denied ✓
```

### Part 6: Implementing /project/shared (THE CHALLENGE!)

**Requirements:**
- Everyone can create files
- Everyone can modify their own files
- Can read others' files
- **CANNOT delete others' files**

**Solution: Sticky Bit + SGID + ACLs**

```bash
# Set ownership
sudo chown root:project-users /project/shared

# Set permissions with SGID + Sticky Bit
sudo chmod 3775 /project/shared
# 3 = SGID (2) + Sticky Bit (1)
# 775 = rwxrwxr-x

# Set default ACLs
sudo setfacl -d -m u::rw /project/shared
sudo setfacl -d -m g::rw /project/shared
sudo setfacl -d -m o::r-- /project/shared

# Verify
ls -ld /project/shared
# drwxrwsr-t 2 root project-users 4096 Jan 15 10:30 /project/shared
#       ^ ^  ^
#       │ │  └─ Sticky bit (t)
#       │ └──── SGID (s)
#       └────── Full permissions for owner
```

**Understanding the Magic:**
```
┌────────────────────────────────────────────────────┐
│  HOW THIS WORKS:                                   │
├────────────────────────────────────────────────────┤
│                                                     │
│  SGID (s in group position):                       │
│    → New files inherit 'project-users' group       │
│    → All team members in this group                │
│    → Everyone can modify files (group has write)   │
│                                                     │
│  Sticky Bit (t in others position):                │
│    → Only file OWNER can delete                    │
│    → Even though directory is writable by all      │
│    → Prevents accidental deletion                  │
│                                                     │
│  Result:                                           │
│    ✓ alice creates file → alice owns it            │
│    ✓ bob can edit (group has write)                │
│    ✓ Bob CAN'T delete (sticky bit prevents)        │
│    ✓ alice CAN delete (owner)                      │
└────────────────────────────────────────────────────┘
```

**Testing the Setup:**
```bash
# alice creates file
sudo -u alice touch /project/shared/alice-doc.txt
sudo -u alice echo "Alice's content" > /project/shared/alice-doc.txt

# Check ownership
ls -l /project/shared/alice-doc.txt
# -rw-rw-r-- 1 alice project-users 16 Jan 15 10:30 alice-doc.txt
#                   ^^^^^^^^^^^^^^ Inherited from directory!

# bob can read
sudo -u bob cat /project/shared/alice-doc.txt
# Alice's content ✓

# bob can modify (group has write)
sudo -u bob echo "Bob's addition" >> /project/shared/alice-doc.txt
# Success! ✓

# bob tries to delete
sudo -u bob rm /project/shared/alice-doc.txt
# rm: cannot remove 'alice-doc.txt': Operation not permitted ✓✓✓

# alice (owner) can delete
sudo -u alice rm /project/shared/alice-doc.txt
# Success! ✓
```

### Part 7: Implementing /project/secrets

**Requirements:**
- Only ops team can access
- Developers can't even see files
- Root can always access

```bash
# Set ownership
sudo chown root:ops /project/secrets

# Restrictive permissions
sudo chmod 2770 /project/secrets
# 2 = SGID
# 770 = rwxrwx--- (owner and group only)

# Set default ACLs (ensure new files stay restricted)
sudo setfacl -d -m u::rwx /project/secrets
sudo setfacl -d -m g::rwx /project/secrets
sudo setfacl -d -m o::--- /project/secrets

# Verify
ls -ld /project/secrets
# drwxrws--- 2 root ops 4096 Jan 15 10:30 /project/secrets

# Test
sudo -u charlie touch /project/secrets/api-key.txt
sudo -u charlie echo "secret123" > /project/secrets/api-key.txt

# alice (developer) tries to access
sudo -u alice ls /project/secrets
# ls: cannot open directory '/project/secrets': Permission denied ✓

# charlie (ops) can access
sudo -u charlie cat /project/secrets/api-key.txt
# secret123 ✓
```

### Part 8: Complete Setup Script

```bash
#!/bin/bash
# project-setup.sh - Complete multi-user project setup

set -e  # Exit on error

echo "=== Project Permission Setup ==="

# Create groups
echo "Creating groups..."
sudo groupadd -f developers
sudo groupadd -f ops
sudo groupadd -f project-users

# Create users
echo "Creating users..."
for user in alice bob charlie dave; do
    if ! id "$user" &>/dev/null; then
        sudo useradd -m -s /bin/bash "$user"
        echo "$user:password123" | sudo chpasswd
    fi
done

# Add users to groups
sudo usermod -aG developers,project-users alice
sudo usermod -aG developers,project-users bob
sudo usermod -aG ops,project-users charlie
sudo usermod -aG project-users dave

# Create directory structure
echo "Creating directories..."
sudo mkdir -p /project/{code,deploy,shared,secrets}
sudo chmod 755 /project

# Setup /project/code
echo "Configuring /project/code..."
sudo chown root:developers /project/code
sudo chmod 2775 /project/code
sudo setfacl -d -m u::rwx /project/code
sudo setfacl -d -m g::rwx /project/code
sudo setfacl -d -m o::r-x /project/code
sudo setfacl -m u:charlie:r-x /project/code
sudo setfacl -d -m u:charlie:r-x /project/code
sudo setfacl -m u:dave:r-- /project/code
sudo setfacl -d -m u:dave:r-- /project/code

# Setup /project/deploy
echo "Configuring /project/deploy..."
sudo chown root:ops /project/deploy
sudo chmod 2770 /project/deploy
sudo setfacl -d -m u::rwx /project/deploy
sudo setfacl -d -m g::rwx /project/deploy
sudo setfacl -d -m o::--- /project/deploy
sudo setfacl -m g:developers:r-x /project/deploy
sudo setfacl -d -m g:developers:r-x /project/deploy

# Setup /project/shared
echo "Configuring /project/shared..."
sudo chown root:project-users /project/shared
sudo chmod 3775 /project/shared
sudo setfacl -d -m u::rw /project/shared
sudo setfacl -d -m g::rw /project/shared
sudo setfacl -d -m o::r-- /project/shared

# Setup /project/secrets
echo "Configuring /project/secrets..."
sudo chown root:ops /project/secrets
sudo chmod 2770 /project/secrets
sudo setfacl -d -m u::rwx /project/secrets
sudo setfacl -d -m g::rwx /project/secrets
sudo setfacl -d -m o::--- /project/secrets

echo "=== Setup Complete ==="
echo ""
echo "Directory Structure:"
tree -L 2 -p /project

echo ""
echo "Run tests with: sudo bash project-test.sh"
```

### Part 9: Testing Script

```bash
#!/bin/bash
# project-test.sh - Test permission setup

set +e  # Don't exit on error (we expect some to fail)

RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m' # No Color

pass() {
    echo -e "${GREEN}✓ PASS:${NC} $1"
}

fail() {
    echo -e "${RED}✗ FAIL:${NC} $1"
}

echo "=== Testing /project/code ==="

# Test 1: alice can create file
if sudo -u alice touch /project/code/test1.txt 2>/dev/null; then
    pass "alice can create files"
else
    fail "alice cannot create files"
fi

# Test 2: charlie (ops) can read
if sudo -u charlie cat /project/code/test1.txt &>/dev/null; then
    pass "charlie can read files"
else
    fail "charlie cannot read files"
fi

# Test 3: charlie cannot write
if ! sudo -u charlie touch /project/code/test2.txt 2>/dev/null; then
    pass "charlie cannot write (as expected)"
else
    fail "charlie can write (should not be able to)"
fi

echo ""
echo "=== Testing /project/shared ==="

# Test 4: alice creates file
sudo -u alice touch /project/shared/alice-file.txt
sudo -u alice echo "alice data" > /project/shared/alice-file.txt

# Test 5: bob can modify
if sudo -u bob echo "bob addition" >> /project/shared/alice-file.txt 2>/dev/null; then
    pass "bob can modify alice's file"
else
    fail "bob cannot modify alice's file"
fi

# Test 6: bob cannot delete
if ! sudo -u bob rm /project/shared/alice-file.txt 2>/dev/null; then
    pass "bob cannot delete alice's file (sticky bit works!)"
else
    fail "bob can delete alice's file (sticky bit failed!)"
fi

# Test 7: alice can delete her own file
if sudo -u alice rm /project/shared/alice-file.txt 2>/dev/null; then
    pass "alice can delete her own file"
else
    fail "alice cannot delete her own file"
fi

echo ""
echo "=== Testing /project/secrets ==="

# Test 8: charlie (ops) can access
if sudo -u charlie touch /project/secrets/secret.txt 2>/dev/null; then
    pass "charlie (ops) can create secrets"
else
    fail "charlie (ops) cannot create secrets"
fi

# Test 9: alice (dev) cannot access
if ! sudo -u alice ls /project/secrets 2>/dev/null; then
    pass "alice (dev) cannot access secrets"
else
    fail "alice (dev) can access secrets (security breach!)"
fi

echo ""
echo "=== Test Summary ==="
echo "If all tests passed, permissions are correctly configured!"
```

---

## Real-World Scenarios

### Scenario 1: Web Application Deployment

**Setup:**
```bash
# Web server runs as www-data
# Developers need to upload files
# Web server needs read access to files

# Create deployment directory
sudo mkdir -p /var/www/app
sudo chown www-data:www-data /var/www/app
sudo chmod 2755 /var/www/app

# Allow developers to upload (ACL)
sudo setfacl -m g:developers:rwx /var/www/app
sudo setfacl -d -m g:developers:rwx /var/www/app
sudo setfacl -d -m u::rw /var/www/app
sudo setfacl -d -m g::r-x /var/www/app
sudo setfacl -d -m o::r-- /var/www/app

# Test
sudo -u alice touch /var/www/app/index.html
ls -l /var/www/app/index.html
# -rw-r-xr-- 1 alice www-data 0 Jan 15 10:30 index.html

# www-data can read
sudo -u www-data cat /var/www/app/index.html
# Success!
```

### Scenario 2: Log File Management

**Setup:**
```bash
# Application writes logs as appuser
# Developers need read access
# Ops need read+write for troubleshooting

sudo mkdir -p /var/log/app
sudo chown appuser:appuser /var/log/app
sudo chmod 2755 /var/log/app

# Grant developers read access
sudo setfacl -m g:developers:r-x /var/log/app
sudo setfacl -d -m g:developers:r-- /var/log/app

# Grant ops read+write
sudo setfacl -m g:ops:rwx /var/log/app
sudo setfacl -d -m g:ops:rw- /var/log/app

# Prevent accidental deletion
sudo chmod +t /var/log/app

# Verify
getfacl /var/log/app
```

### Scenario 3: Database Backup Directory

**Setup:**
```bash
# Only database user and ops can access
# Strict permissions for security

sudo mkdir -p /backup/database
sudo chown postgres:postgres /backup/database
sudo chmod 700 /backup/database

# Grant ops access via ACL
sudo setfacl -m g:ops:rwx /backup/database
sudo setfacl -d -m g:ops:rwx /backup/database

# Verify no one else can access
sudo -u alice ls /backup/database
# Permission denied ✓

# postgres and ops can access
sudo -u postgres ls /backup/database
sudo -u charlie ls /backup/database
# Success!
```

### Scenario 4: CI/CD Pipeline Artifacts

**Setup:**
```bash
# Jenkins user creates artifacts
# Developers can download (read-only)
# Cleanup jobs can delete old artifacts

sudo mkdir -p /var/jenkins/artifacts
sudo chown jenkins:jenkins /var/jenkins/artifacts
sudo chmod 2755 /var/jenkins/artifacts

# Developers can read
sudo setfacl -m g:developers:r-x /var/jenkins/artifacts
sudo setfacl -d -m g:developers:r-- /var/jenkins/artifacts

# Cleanup user can delete
sudo setfacl -m u:cleanup:rwx /var/jenkins/artifacts
sudo setfacl -d -m u:cleanup:rwx /var/jenkins/artifacts

# Test
sudo -u jenkins touch /var/jenkins/artifacts/build-123.tar.gz
sudo -u alice cat /var/jenkins/artifacts/build-123.tar.gz
# Can read ✓
sudo -u alice rm /var/jenkins/artifacts/build-123.tar.gz
# Cannot delete ✓
sudo -u cleanup rm /var/jenkins/artifacts/build-123.tar.gz
# Can delete ✓
```

---

## Troubleshooting & Speed Drills

### Part 1: Common Permission Problems

#### Problem 1: "Permission Denied" on File Access

```bash
# Diagnostic workflow:

# 1. Check basic permissions
ls -l filename
# -rw------- 1 alice alice 123 Jan 15 10:30 filename

# 2. Check ACLs
getfacl filename
# user::rw-
# group::---
# other::---

# 3. Check your user/groups
id
# uid=1002(bob) gid=1002(bob) groups=1002(bob),1001(developers)

# 4. Check parent directory permissions
ls -ld /path/to/
# drwx------ 2 alice alice 4096 Jan 15 10:30 /path/to

# Solution: Need execute on parent directory!
sudo chmod o+x /path/to/
# OR add ACL:
sudo setfacl -m u:bob:x /path/to/
```

#### Problem 2: Can't Create File in Directory

```bash
# Diagnostic:
ls -ld /project/code
# drwxr-xr-x 2 root developers 4096 Jan 15 10:30 /project/code
#    └─ Group needs write!

# Check groups
groups
# alice : alice developers

# Solution: Add write for group
sudo chmod g+w /project/code
# OR use ACL:
sudo setfacl -m g:developers:rwx /project/code
```

#### Problem 3: ACLs Not Inherited

```bash
# Problem: New files don't have expected ACLs

# Check default ACLs
getfacl /project/code
# (No default entries shown)

# Solution: Set default ACLs
sudo setfacl -d -m u:bob:rwx /project/code
sudo setfacl -d -m g::rwx /project/code

# Verify
getfacl /project/code
# Should show "default:" entries
```

#### Problem 4: Can Delete Others' Files (Sticky Bit Not Working)

```bash
# Check sticky bit
ls -ld /project/shared
# drwxrwxrwx 2 root project-users 4096 Jan 15 10:30 /project/shared
#          ^ Should be 't', not 'x'!

# Solution: Add sticky bit
sudo chmod +t /project/shared

# Verify
ls -ld /project/shared
# drwxrwxrwt 2 root project-users 4096 Jan 15 10:30 /project/shared
#          ^ Now has 't'
```

### Part 2: Speed Drill - 2 Minute Challenge

**Scenario:** Fix permission issues quickly

```bash
# Setup (run this first):
sudo mkdir -p /test/drill
sudo touch /test/drill/file1.txt
sudo chmod 000 /test/drill/file1.txt
sudo chown root:root /test/drill/file1.txt

# CHALLENGE: Make file accessible to user "alice" in under 2 minutes
# Requirements:
#   - alice can read and write
#   - Other users cannot access
#   - Don't change ownership

# Timer starts... GO!

# Solution 1: ACL (fastest)
sudo setfacl -m u:alice:rw /test/drill/file1.txt
# Time: ~5 seconds

# Solution 2: Group-based
sudo chgrp alice /test/drill/file1.txt
sudo chmod 060 /test/drill/file1.txt
# Time: ~10 seconds

# Verify
sudo -u alice cat /test/drill/file1.txt
# Should work!
```

### Part 3: Permission Debugging Checklist

```bash
# Copy this script for quick diagnostics

#!/bin/bash
# check-perms.sh <file>

FILE=$1

echo "=== Permission Diagnostic for $FILE ==="
echo ""

# 1. File exists?
if [ ! -e "$FILE" ]; then
    echo "❌ File does not exist!"
    exit 1
fi

# 2. Basic permissions
echo "Basic Permissions:"
ls -ld "$FILE"
echo ""

# 3. ACLs
echo "ACLs:"
getfacl "$FILE"
echo ""

# 4. Parent directory
PARENT=$(dirname "$FILE")
echo "Parent Directory ($PARENT):"
ls -ld "$PARENT"
echo ""

# 5. Parent ACLs
echo "Parent ACLs:"
getfacl "$PARENT"
echo ""

# 6. Current user context
echo "Your User Context:"
id
echo ""

# 7. Test access
echo "Access Test:"
if [ -r "$FILE" ]; then
    echo "✓ Can read"
else
    echo "✗ Cannot read"
fi

if [ -w "$FILE" ]; then
    echo "✓ Can write"
else
    echo "✗ Cannot write"
fi

if [ -x "$FILE" ]; then
    echo "✓ Can execute"
else
    echo "✗ Cannot execute"
fi
```

### Part 4: Muscle Memory Drills

**Drill 1: Quick ACL Setup (30 seconds)**

```bash
# Practice this sequence 10 times:

# 1. Create file
touch testfile.txt

# 2. View current ACLs
getfacl testfile.txt

# 3. Add user ACL
setfacl -m u:bob:rw testfile.txt

# 4. Add group ACL
setfacl -m g:developers:r testfile.txt

# 5. Verify
getfacl testfile.txt

# 6. Remove ACL
setfacl -x u:bob testfile.txt

# 7. Clean up
rm testfile.txt

# Goal: Complete in under 30 seconds
```

**Drill 2: Directory Setup (1 minute)**

```bash
# Practice creating secure shared directory:

# 1. Create directory
mkdir -p /tmp/shared-drill

# 2. Set SGID + Sticky
chmod 3775 /tmp/shared-drill

# 3. Set group
chgrp developers /tmp/shared-drill

# 4. Default ACLs
setfacl -d -m u::rw /tmp/shared-drill
setfacl -d -m g::rw /tmp/shared-drill
setfacl -d -m o::r-- /tmp/shared-drill

# 5. Verify
ls -ld /tmp/shared-drill
getfacl /tmp/shared-drill

# 6. Test
touch /tmp/shared-drill/test.txt
ls -l /tmp/shared-drill/test.txt
getfacl /tmp/shared-drill/test.txt

# 7. Clean up
rm -rf /tmp/shared-drill

# Goal: Complete in under 1 minute
```

**Drill 3: Permission Troubleshooting (2 minutes)**

```bash
# Scenario: User reports "Permission Denied"
# Find and fix the issue

# Setup:
sudo mkdir -p /tmp/broken
sudo touch /tmp/broken/file.txt
sudo chmod 000 /tmp/broken
sudo chmod 000 /tmp/broken/file.txt

# GO! Fix access for current user

# Solution steps:
# 1. Check file
ls -l /tmp/broken/file.txt  # Will fail - no access to parent

# 2. Check parent
ls -ld /tmp/broken  # Aha! drwx------

# 3. Fix parent first
sudo chmod 755 /tmp/broken

# 4. Now check file
ls -l /tmp/broken/file.txt  # ----------

# 5. Fix file
sudo chmod 644 /tmp/broken/file.txt

# 6. Verify
cat /tmp/broken/file.txt  # Works!

# Goal: Complete in under 2 minutes
```

---

## Cheat Sheet

### Quick Reference

```bash
# BASIC PERMISSIONS
chmod 755 file              # rwxr-xr-x
chmod 644 file              # rw-r--r--
chmod +x file               # Add execute
chmod u+w,go-w file        # User write, others no write
chown user:group file       # Change owner
chgrp group file           # Change group only

# SPECIAL BITS
chmod u+s file             # SUID (4)
chmod g+s dir              # SGID (2)
chmod +t dir               # Sticky bit (1)
chmod 2775 dir             # SGID + rwxrwxr-x
chmod 3775 dir             # SGID + Sticky + rwxrwxr-x

# ACLs - VIEWING
getfacl file               # Show ACLs
getfacl -R dir            # Recursive
ls -l                      # '+' indicates ACLs present

# ACLs - SETTING
setfacl -m u:bob:rw file   # Add user ACL
setfacl -m g:dev:rx file   # Add group ACL
setfacl -x u:bob file      # Remove user ACL
setfacl -b file            # Remove all ACLs
setfacl -d -m u:bob:rw dir # Default ACL (inheritance)

# ACLs - COPYING
getfacl file1 | setfacl --set-file=- file2
getfacl -R /dir > backup.acl
setfacl --restore=backup.acl

# DEBUGGING
namei -l /path/to/file     # Check all parent permissions
stat file                  # Detailed file info
id                         # Your user/group info
groups user                # User's groups

# COMMON PATTERNS
# Shared directory (create but not delete others' files):
chmod 3775 dir             # SGID + Sticky
setfacl -d -m u::rw dir
setfacl -d -m g::rw dir

# Read-only for specific user:
setfacl -m u:bob:r file

# Private directory (owner only):
chmod 700 dir

# Web server files:
chown www-data:www-data /var/www
chmod 755 /var/www         # Directories
chmod 644 /var/www/*.html  # Files
```

### Visual Decision Tree

```
Permission Problem?
│
├─ Need basic permissions?
│  └─ Use chmod/chown
│
├─ Need per-user access?
│  └─ Use ACLs (setfacl)
│
├─ Shared directory?
│  ├─ Everyone creates files → chmod g+s (SGID)
│  └─ Can't delete others' → chmod +t (sticky)
│
└─ Can't access file?
   ├─ Check file permissions (ls -l)
   ├─ Check ACLs (getfacl)
   ├─ Check parent directory (namei -l)
   └─ Check your groups (id)
```

---

## Final Mastery Test

### Test 1: Basic Setup (5 minutes)

```bash
# Task: Create project directory with these requirements:
# - Directory: /test/project
# - Group: developers
# - Developers: full access
# - User 'bob': read-only
# - New files inherit group
# - Sticky bit prevents deletion

# Your commands:
# _______________________________________________

# Solution:
sudo mkdir -p /test/project
sudo chgrp developers /test/project
sudo chmod 3775 /test/project
sudo setfacl -m u:bob:r-x /test/project
sudo setfacl -d -m u:bob:r-- /test/project
sudo setfacl -d -m u::rw /test/project
sudo setfacl -d -m g::rw /test/project

# Verify:
getfacl /test/project
ls -ld /test/project
```

### Test 2: Troubleshooting (2 minutes)

```bash
# Problem: User alice reports "Permission Denied" on /data/file.txt

# Your diagnostic commands:
# _______________________________________________

# Solution:
ls -l /data/file.txt        # Check file perms
getfacl /data/file.txt      # Check ACLs
id alice                    # Check alice's groups
namei -l /data/file.txt     # Check parent perms
sudo setfacl -m u:alice:rw /data/file.txt  # Fix

# Verify:
sudo -u alice cat /data/file.txt
```

### Test 3: Complex Scenario (10 minutes)

```bash
# Task: Set up CI/CD artifact directory
# Requirements:
# - Jenkins user creates artifacts
# - Developers can read
# - Cleanup script can delete old files
# - Nobody else can access

# Your complete setup:
# _______________________________________________

# Solution:
sudo mkdir -p /var/ci/artifacts
sudo chown jenkins:jenkins /var/ci/artifacts
sudo chmod 2750 /var/ci/artifacts
sudo setfacl -m g:developers:r-x /var/ci/artifacts
sudo setfacl -d -m g:developers:r-- /var/ci/artifacts
sudo setfacl -m u:cleanup:rwx /var/ci/artifacts
sudo setfacl -d -m u:cleanup:rwx /var/ci/artifacts

# Test:
sudo -u jenkins touch /var/ci/artifacts/build.tar.gz
sudo -u alice cat /var/ci/artifacts/build.tar.gz  # Can read
sudo -u alice rm /var/ci/artifacts/build.tar.gz   # Can't delete
sudo -u cleanup rm /var/ci/artifacts/build.tar.gz # Can delete
```

---

## Conclusion

You now have complete mastery of:
- ✅ Basic permission system (rwx, chmod, chown)
- ✅ Special bits (SUID, SGID, sticky)
- ✅ Advanced ACLs (setfacl, getfacl)
- ✅ Multi-user project setups
- ✅ Real-world DevOps scenarios
- ✅ 2-minute troubleshooting

**Daily Practice:**
1. Run the speed drills
2. Solve one test scenario
3. Review the cheat sheet
4. Apply to real projects

**Remember:** Permissions are about **security** and **collaboration**. Get them right once, and your team operates smoothly! 🚀