# System Audit Script Mastery Guide for Ubuntu

## Table of Contents
1. [Foundation Concepts](#foundation-concepts)
2. [Script Structure & Planning](#script-structure--planning)
3. [User Audit Module](#user-audit-module)
4. [Permission Security Module](#permission-security-module)
5. [Disk Space Analysis Module](#disk-space-analysis-module)
6. [Process & Network Security Module](#process--network-security-module)
7. [Complete Script](#complete-script)
8. [Testing & Validation](#testing--validation)
9. [Advanced Techniques](#advanced-techniques)

---

## Foundation Concepts

### Why System Auditing Matters in DevOps

```
┌─────────────────────────────────────────────────────────┐
│  CRITICAL DEVOPS SCENARIOS REQUIRING SYSTEM AUDITS:     │
├─────────────────────────────────────────────────────────┤
│  ✓ Security compliance (PCI-DSS, HIPAA, SOC2)          │
│  ✓ Incident response (breach investigation)            │
│  ✓ Performance troubleshooting (disk space, processes) │
│  ✓ Access control verification (who has access?)       │
│  ✓ Vulnerability scanning (dangerous permissions)      │
│  ✓ Capacity planning (disk usage trends)               │
│  ✓ Baseline establishment (normal vs abnormal)         │
│  ✓ Automated compliance reporting                      │
└─────────────────────────────────────────────────────────┘
```

### Security Audit Components

```
┌──────────────────────────────────────────────────────────┐
│                  AUDIT ARCHITECTURE                       │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  MODULE 1: USER AUDIT                                    │
│  ├─ Active users                                         │
│  ├─ Last login times                                     │
│  ├─ Dormant accounts (security risk)                     │
│  ├─ Users with shell access                              │
│  └─ UID 0 accounts (root equivalents)                    │
│                                                           │
│  MODULE 2: PERMISSION AUDIT                              │
│  ├─ World-writable files (777)                           │
│  ├─ SUID files (run as owner)                            │
│  ├─ SGID files (run with group privileges)               │
│  ├─ No-owner files (orphaned)                            │
│  └─ Suspicious executable locations                      │
│                                                           │
│  MODULE 3: DISK SPACE AUDIT                              │
│  ├─ Large files (>100MB)                                 │
│  ├─ Directory space usage                                │
│  ├─ Filesystem capacity                                  │
│  ├─ Inode usage                                          │
│  └─ Old files (potential cleanup)                        │
│                                                           │
│  MODULE 4: PROCESS & NETWORK AUDIT                       │
│  ├─ Unusual processes (high CPU/memory)                  │
│  ├─ Processes running as root                            │
│  ├─ Listening ports                                      │
│  ├─ Established connections                              │
│  └─ Unknown/suspicious binaries                          │
└──────────────────────────────────────────────────────────┘
```

---

## Script Structure & Planning

### Part 1: Script Template Architecture

**Why This Structure?**
```
Modular design allows:
  ✓ Easy testing of individual components
  ✓ Simple maintenance and updates
  ✓ Reusable functions
  ✓ Clear error handling
  ✓ Portable across distributions
```

**Basic Script Template:**

```bash
#!/bin/bash
# system-audit.sh - Comprehensive system security audit
# Author: DevOps Team
# Version: 1.0
# Description: Audits users, permissions, disk usage, processes, and network

###########################################
# CONFIGURATION
###########################################

# Script metadata
SCRIPT_NAME="System Audit Script"
VERSION="1.0"
AUDIT_DATE=$(date '+%Y-%m-%d %H:%M:%S')

# Output configuration
OUTPUT_DIR="/var/log/system-audit"
REPORT_FILE="${OUTPUT_DIR}/audit-$(date '+%Y%m%d-%H%M%S').txt"

# Thresholds (configurable)
LARGE_FILE_SIZE="100M"          # Files larger than this
DISK_WARN_THRESHOLD=80          # Disk usage warning %
OLD_FILE_DAYS=365               # Files older than this
MAX_SUID_FILES=100              # Alert if more SUID files found

# Color codes for output
RED='\033[0;31m'
YELLOW='\033[1;33m'
GREEN='\033[0;32m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

###########################################
# HELPER FUNCTIONS
###########################################

# Print section header
print_header() {
    echo ""
    echo "=========================================="
    echo "$1"
    echo "=========================================="
    echo ""
}

# Print success message
print_success() {
    echo -e "${GREEN}✓ $1${NC}"
}

# Print warning message
print_warning() {
    echo -e "${YELLOW}⚠ $1${NC}"
}

# Print error message
print_error() {
    echo -e "${RED}✗ $1${NC}"
}

# Print info message
print_info() {
    echo -e "${BLUE}ℹ $1${NC}"
}

# Check if script is run as root
check_root() {
    if [[ $EUID -ne 0 ]]; then
        print_error "This script must be run as root (sudo)"
        exit 1
    fi
}

# Create output directory
setup_output() {
    if [[ ! -d "$OUTPUT_DIR" ]]; then
        mkdir -p "$OUTPUT_DIR"
        print_success "Created output directory: $OUTPUT_DIR"
    fi
}

# Write to both console and report file
log_output() {
    echo "$1" | tee -a "$REPORT_FILE"
}

###########################################
# MAIN EXECUTION
###########################################

main() {
    print_header "$SCRIPT_NAME v$VERSION"
    print_info "Audit started: $AUDIT_DATE"
    
    check_root
    setup_output
    
    # Initialize report file
    {
        echo "================================================================"
        echo "  SYSTEM SECURITY AUDIT REPORT"
        echo "================================================================"
        echo "Generated: $AUDIT_DATE"
        echo "Hostname: $(hostname)"
        echo "OS: $(cat /etc/os-release | grep PRETTY_NAME | cut -d'"' -f2)"
        echo "Kernel: $(uname -r)"
        echo "================================================================"
        echo ""
    } > "$REPORT_FILE"
    
    # Run audit modules
    audit_users
    audit_permissions
    audit_disk_space
    audit_processes_network
    
    # Summary
    print_header "AUDIT COMPLETE"
    print_success "Report saved to: $REPORT_FILE"
}

# Run main function
main "$@"
```

**Understanding the Template:**

1. **Shebang (`#!/bin/bash`)**: Tells system to use bash interpreter
2. **Configuration Section**: All customizable parameters in one place
3. **Helper Functions**: Reusable code for output formatting
4. **Main Function**: Orchestrates the entire audit
5. **Dual Output**: Results shown on screen AND saved to file

### Part 2: Cross-Distribution Compatibility

**Why Compatibility Matters:**
```
DevOps environments often have:
  - Ubuntu servers
  - CentOS/RHEL servers
  - Debian servers
  - Alpine containers

Script must work everywhere!
```

**Compatibility Techniques:**

```bash
# Detect distribution
detect_distro() {
    if [[ -f /etc/os-release ]]; then
        . /etc/os-release
        DISTRO=$ID
        VERSION=$VERSION_ID
    elif [[ -f /etc/redhat-release ]]; then
        DISTRO="rhel"
    else
        DISTRO="unknown"
    fi
    
    print_info "Detected distribution: $DISTRO $VERSION"
}

# Use portable commands
# ✓ Use: command -v instead of which
# ✓ Use: [[ ]] instead of [ ] (bash-specific but more robust)
# ✓ Use: $(command) instead of `command`

# Check command availability
check_command() {
    local cmd=$1
    if ! command -v "$cmd" &> /dev/null; then
        print_warning "Command '$cmd' not found, some features may be limited"
        return 1
    fi
    return 0
}

# Distribution-agnostic package check
check_package() {
    local package=$1
    
    case $DISTRO in
        ubuntu|debian)
            dpkg -l | grep -q "^ii.*$package"
            ;;
        centos|rhel|fedora)
            rpm -qa | grep -q "$package"
            ;;
        alpine)
            apk info | grep -q "$package"
            ;;
        *)
            print_warning "Unknown distribution, cannot check packages"
            return 1
            ;;
    esac
}
```

---

## User Audit Module

### Part 1: Understanding User Information Sources

**Critical Files:**
```
/etc/passwd     - User account information
/etc/shadow     - Encrypted passwords (root only)
/var/log/wtmp   - Login history (binary format)
/var/log/lastlog - Last login times (binary format)
/var/run/utmp   - Currently logged in users
```

**File Format - /etc/passwd:**
```
username:x:UID:GID:comment:home:shell
│        │ │   │   │       │    └─ Login shell
│        │ │   │   │       └─ Home directory
│        │ │   │   └─ GECOS field (full name, etc)
│        │ │   └─ Primary group ID
│        │ └─ User ID
│        └─ Password (x = in /etc/shadow)
└─ Username

Example:
alice:x:1001:1001:Alice Smith:/home/alice:/bin/bash
```

### Part 2: User Audit Implementation

```bash
###########################################
# USER AUDIT MODULE
###########################################

audit_users() {
    print_header "USER AUDIT"
    
    {
        echo "USER AUDIT"
        echo "=========================================="
        echo ""
        
        # Total user count
        local total_users=$(wc -l < /etc/passwd)
        echo "Total users in system: $total_users"
        echo ""
        
        # Users with login shells (potential interactive access)
        echo "Users with Login Shells:"
        echo "------------------------"
        awk -F: '$7 !~ /nologin|false/ {print $1 " (UID: " $3 ", Shell: " $7 ")"}' /etc/passwd
        echo ""
        
        # Users with UID 0 (root equivalent)
        echo "Users with UID 0 (Root Privileges):"
        echo "-----------------------------------"
        awk -F: '$3 == 0 {print $1}' /etc/passwd
        local uid0_count=$(awk -F: '$3 == 0' /etc/passwd | wc -l)
        if [[ $uid0_count -gt 1 ]]; then
            echo "⚠ WARNING: Multiple UID 0 accounts detected!"
        fi
        echo ""
        
        # Last login times
        echo "Last Login Times:"
        echo "-----------------"
        # lastlog shows all users, filter for actual logins
        lastlog | awk 'NR==1 || $2 != "**Never" {print}'
        echo ""
        
        # Users who never logged in
        echo "Users Who Never Logged In:"
        echo "--------------------------"
        lastlog | awk '$2 == "**Never" && NR > 1 {print $1}'
        echo ""
        
        # Currently logged in users
        echo "Currently Logged In:"
        echo "-------------------"
        who
        echo ""
        
        # Users with recent activity (last 7 days)
        echo "Users Active in Last 7 Days:"
        echo "----------------------------"
        last -F -n 50 | awk '{print $1}' | sort -u | head -20
        echo ""
        
        # Check for suspicious usernames
        echo "Suspicious Username Check:"
        echo "-------------------------"
        awk -F: '{
            # Check for hidden users (UID < 1000 but not system users)
            if ($3 < 1000 && $3 > 0 && $7 !~ /nologin|false/) {
                print "⚠ Non-system user with low UID: " $1 " (UID: " $3 ")"
            }
            # Check for users with suspicious names
            if ($1 ~ /^[.]/ || $1 ~ /test|temp|guest/) {
                print "⚠ Suspicious username: " $1
            }
        }' /etc/passwd
        echo ""
        
        # Password aging information (if accessible)
        if [[ -r /etc/shadow ]]; then
            echo "Password Aging Information:"
            echo "---------------------------"
            awk -F: '{
                # Field 5 is maximum password age
                if ($5 != "" && $5 == "99999") {
                    print "⚠ " $1 ": Password never expires"
                }
            }' /etc/shadow | head -20
            echo ""
        fi
        
        # Check for accounts with empty passwords
        echo "Security Check - Empty Passwords:"
        echo "--------------------------------"
        if [[ -r /etc/shadow ]]; then
            awk -F: '($2 == "" || $2 == "!") {print $1}' /etc/shadow
            local empty_pass=$(awk -F: '$2 == ""' /etc/shadow | wc -l)
            if [[ $empty_pass -gt 0 ]]; then
                echo "🚨 CRITICAL: $empty_pass accounts with empty passwords!"
            else
                echo "✓ No empty passwords found"
            fi
        else
            echo "⚠ Cannot access /etc/shadow - run as root"
        fi
        echo ""
        
    } | tee -a "$REPORT_FILE"
    
    print_success "User audit completed"
}
```

**Why Each Check Matters:**

1. **UID 0 accounts**: Only root should have UID 0
   - Multiple UID 0 = backdoor risk
   - Attack: Create user with UID 0 for persistent access

2. **Never-logged-in users**: Potential stale accounts
   - Security risk: Forgotten accounts with weak passwords
   - Compliance: Remove unused accounts

3. **Empty passwords**: Critical vulnerability
   - Anyone can login without authentication
   - Automated attacks will find these

4. **Low UID non-system users**: Suspicious
   - UIDs < 1000 usually reserved for system
   - Could indicate tampering

### Part 3: Advanced User Analysis

```bash
# Enhanced user audit with grouping and sudo access

audit_users_advanced() {
    print_header "ADVANCED USER ANALYSIS"
    
    {
        echo "SUDO/ADMIN ACCESS AUDIT"
        echo "======================="
        echo ""
        
        # Check sudo group members
        echo "Users in 'sudo' group:"
        getent group sudo | cut -d: -f4 | tr ',' '\n'
        echo ""
        
        # Check sudoers file (if readable)
        if [[ -r /etc/sudoers ]]; then
            echo "Custom sudoers entries:"
            grep -v '^#' /etc/sudoers | grep -v '^$' | grep 'ALL=(ALL)'
            echo ""
        fi
        
        # Users with valid shells grouped by shell type
        echo "Users Grouped by Shell:"
        echo "----------------------"
        awk -F: '$7 !~ /nologin|false/ {shells[$7]++; users[$7]=users[$7] $1 ", "} 
                 END {for (shell in shells) print shell ": " shells[shell] " users (" users[shell] ")"
        }' /etc/passwd
        echo ""
        
        # Detect service accounts that shouldn't have shells
        echo "Service Accounts with Shells (Suspicious):"
        echo "------------------------------------------"
        awk -F: '
            # System UIDs (usually < 1000) with valid shells
            $3 < 1000 && $3 > 0 && $7 !~ /nologin|false/ {
                print $1 " (UID: " $3 ", Shell: " $7 ")"
            }
        ' /etc/passwd
        echo ""
        
    } | tee -a "$REPORT_FILE"
}
```

---

## Permission Security Module

### Part 1: Understanding Dangerous Permissions

**Security Risk Matrix:**

```
┌─────────────────────────────────────────────────────┐
│  PERMISSION RISK LEVELS                             │
├─────────────────────────────────────────────────────┤
│                                                      │
│  🚨 CRITICAL (Immediate Action Required)            │
│  ├─ World-writable files: /etc/passwd, /etc/shadow │
│  ├─ SUID root binaries in /tmp or /dev/shm         │
│  └─ Writable startup scripts                        │
│                                                      │
│  ⚠️  HIGH (Review Required)                          │
│  ├─ World-writable files in /usr or /bin           │
│  ├─ SUID files owned by regular users              │
│  ├─ SGID directories in sensitive locations        │
│  └─ Files with no owner (orphaned)                 │
│                                                      │
│  ⚡ MEDIUM (Monitor)                                │
│  ├─ Excessive SUID/SGID binaries                   │
│  ├─ World-readable sensitive files                 │
│  └─ Unusual file permissions in system dirs        │
└─────────────────────────────────────────────────────┘
```

**Why These Are Dangerous:**

```
World-Writable (777):
  - Anyone can modify file
  - Attacker can replace legitimate binary with malware
  - Example: writable /etc/passwd = add user with UID 0

SUID (Set User ID):
  - File executes with OWNER's permissions
  - SUID root = runs as root when executed
  - Vulnerability = instant root access
  - Example: SUID shell = permanent root access

SGID (Set Group ID):
  - File executes with GROUP's permissions
  - Can access group-restricted resources
  - Directory SGID = new files inherit group

No Owner:
  - User deleted but files remain
  - Can't properly manage permissions
  - May be exploitable
```

### Part 2: Permission Audit Implementation

```bash
###########################################
# PERMISSION SECURITY MODULE
###########################################

audit_permissions() {
    print_header "PERMISSION SECURITY AUDIT"
    
    {
        echo "PERMISSION SECURITY AUDIT"
        echo "=========================================="
        echo ""
        
        # World-writable files (777 or XX7)
        echo "World-Writable Files:"
        echo "--------------------"
        print_info "Searching for world-writable files (this may take a few minutes)..."
        
        # Search common directories (exclude /proc, /sys to speed up)
        find /etc /home /root /usr /var /opt /tmp 2>/dev/null \
            -type f -perm -002 ! -type l \
            -exec ls -l {} \; | head -100
        
        local ww_count=$(find /etc /home /root /usr /var /opt /tmp 2>/dev/null \
            -type f -perm -002 ! -type l | wc -l)
        
        echo ""
        echo "Total world-writable files found: $ww_count"
        
        if [[ $ww_count -gt 0 ]]; then
            echo "🚨 CRITICAL: World-writable files detected!"
            echo "   These files can be modified by any user on the system."
            echo "   Recommendation: Review and restrict permissions."
        else
            echo "✓ No world-writable files found in common directories"
        fi
        echo ""
        
        # SUID files
        echo "SUID Files (Set User ID):"
        echo "------------------------"
        print_info "Searching for SUID files..."
        
        find / -type f -perm -4000 2>/dev/null -exec ls -l {} \; | \
            awk '{print $1, $3, $9}' | column -t
        
        local suid_count=$(find / -type f -perm -4000 2>/dev/null | wc -l)
        echo ""
        echo "Total SUID files: $suid_count"
        
        if [[ $suid_count -gt $MAX_SUID_FILES ]]; then
            echo "⚠ WARNING: Unusually high number of SUID files"
            echo "   Expected: ~$MAX_SUID_FILES, Found: $suid_count"
        fi
        
        # Check for suspicious SUID files
        echo ""
        echo "Suspicious SUID Files:"
        echo "---------------------"
        find / -type f -perm -4000 2>/dev/null | \
        while read -r file; do
            # Check if in unusual location
            if [[ "$file" =~ /tmp/|/dev/shm/|/home/ ]]; then
                echo "🚨 CRITICAL: SUID file in suspicious location: $file"
                ls -l "$file"
            fi
            
            # Check if owned by regular user
            owner=$(stat -c '%U' "$file" 2>/dev/null)
            uid=$(stat -c '%u' "$file" 2>/dev/null)
            if [[ $uid -ge 1000 ]]; then
                echo "⚠ WARNING: SUID file owned by regular user: $file (owner: $owner)"
            fi
        done
        echo ""
        
        # SGID files
        echo "SGID Files (Set Group ID):"
        echo "-------------------------"
        find / -type f -perm -2000 2>/dev/null -exec ls -l {} \; | head -50
        
        local sgid_count=$(find / -type f -perm -2000 2>/dev/null | wc -l)
        echo ""
        echo "Total SGID files: $sgid_count"
        echo ""
        
        # Files with no owner
        echo "Orphaned Files (No Owner):"
        echo "-------------------------"
        find / -nouser -o -nogroup 2>/dev/null | head -50
        
        local orphan_count=$(find / -nouser -o -nogroup 2>/dev/null | wc -l)
        echo ""
        echo "Total orphaned files: $orphan_count"
        
        if [[ $orphan_count -gt 0 ]]; then
            echo "⚠ WARNING: Orphaned files found"
            echo "   These files have no valid owner/group (user may have been deleted)"
            echo "   Recommendation: Review and assign proper ownership or remove"
        fi
        echo ""
        
        # Critical system files check
        echo "Critical System Files Permission Check:"
        echo "---------------------------------------"
        
        critical_files=(
            "/etc/passwd:644:root:root"
            "/etc/shadow:640:root:shadow"
            "/etc/group:644:root:root"
            "/etc/sudoers:440:root:root"
            "/etc/ssh/sshd_config:600:root:root"
        )
        
        for entry in "${critical_files[@]}"; do
            IFS=':' read -r file expected_perm expected_owner expected_group <<< "$entry"
            
            if [[ -f "$file" ]]; then
                actual_perm=$(stat -c '%a' "$file" 2>/dev/null)
                actual_owner=$(stat -c '%U' "$file" 2>/dev/null)
                actual_group=$(stat -c '%G' "$file" 2>/dev/null)
                
                printf "%-25s " "$file"
                
                if [[ "$actual_perm" == "$expected_perm" ]] && \
                   [[ "$actual_owner" == "$expected_owner" ]] && \
                   [[ "$actual_group" == "$expected_group" ]]; then
                    echo "✓ OK ($actual_perm $actual_owner:$actual_group)"
                else
                    echo "🚨 MISMATCH!"
                    echo "   Expected: $expected_perm $expected_owner:$expected_group"
                    echo "   Actual:   $actual_perm $actual_owner:$actual_group"
                fi
            else
                echo "⚠ File not found: $file"
            fi
        done
        echo ""
        
        # World-writable directories without sticky bit
        echo "World-Writable Directories Without Sticky Bit:"
        echo "----------------------------------------------"
        find / -type d -perm -002 ! -perm -1000 2>/dev/null | head -20
        
        echo ""
        echo "⚠ These directories allow any user to create/delete files"
        echo "   Recommendation: Add sticky bit (+t) or restrict permissions"
        echo ""
        
    } | tee -a "$REPORT_FILE"
    
    print_success "Permission audit completed"
}
```

**Understanding the Find Commands:**

```bash
# Find world-writable files
find / -type f -perm -002 2>/dev/null
#      │      │     │       └─ Suppress error messages
#      │      │     └─ Permission: others have write (bit 1)
#      │      └─ Only files (not directories)
#      └─ Start from root

# Find SUID files
find / -type f -perm -4000 2>/dev/null
#                    └─ SUID bit (octal 4000)

# Find SGID files  
find / -type f -perm -2000 2>/dev/null
#                    └─ SGID bit (octal 2000)

# Find files with no owner
find / -nouser -o -nogroup 2>/dev/null
#      └─nouser  │  └─ no group
#              OR (either condition)

# Permission notation:
# -perm -002  : At least these permissions (others write)
# -perm 002   : Exactly these permissions
# -perm /002  : Any of these permissions
```

### Part 3: Advanced Permission Analysis

```bash
# Check for recently modified SUID/SGID files (potential backdoor)
check_recent_suid() {
    echo "Recently Modified SUID/SGID Files (Last 30 Days):"
    echo "------------------------------------------------"
    
    find / -type f \( -perm -4000 -o -perm -2000 \) -mtime -30 2>/dev/null | \
    while read -r file; do
        echo "$(stat -c '%y %n' "$file")"
    done | sort -r
    echo ""
}

# Check for hidden SUID files (starting with .)
check_hidden_suid() {
    echo "Hidden SUID Files:"
    echo "-----------------"
    
    find / -type f -perm -4000 -name ".*" 2>/dev/null
    echo ""
}

# Verify integrity of common SUID binaries
verify_suid_integrity() {
    echo "Common SUID Binary Verification:"
    echo "--------------------------------"
    
    # List of expected SUID binaries
    expected_suid=(
        "/usr/bin/passwd"
        "/usr/bin/sudo"
        "/usr/bin/su"
        "/bin/ping"
        "/bin/mount"
        "/bin/umount"
    )
    
    for binary in "${expected_suid[@]}"; do
        if [[ -f "$binary" ]]; then
            perm=$(stat -c '%a' "$binary")
            if [[ $perm =~ ^[4567] ]]; then
                echo "✓ $binary: $perm (SUID as expected)"
            else
                echo "⚠ $binary: $perm (SUID bit NOT set - unusual!)"
            fi
        fi
    done
    echo ""
}
```

---

## Disk Space Analysis Module

### Part 1: Understanding Disk Space Issues

**Common Disk Space Problems:**

```
┌─────────────────────────────────────────────────────┐
│  DISK SPACE ISSUES IN PRODUCTION:                   │
├─────────────────────────────────────────────────────┤
│                                                      │
│  Log Files:                                         │
│  ├─ Unrotated application logs (GB sized)          │
│  ├─ Core dumps from crashes                        │
│  └─ Debug logs left on in production               │
│                                                      │
│  Cache/Temp:                                        │
│  ├─ Package manager cache (/var/cache)             │
│  ├─ Temp files (/tmp, /var/tmp)                    │
│  └─ Build artifacts                                 │
│                                                      │
│  Databases:                                         │
│  ├─ Uncleaned transaction logs                     │
│  ├─ Old backups not purged                         │
│  └─ Database growth without monitoring             │
│                                                      │
│  Miscellaneous:                                     │
│  ├─ Orphaned Docker images/containers               │
│  ├─ Old kernel versions                             │
│  └─ Deleted files still held open by processes     │
└─────────────────────────────────────────────────────┘
```

### Part 2: Disk Space Audit Implementation

```bash
###########################################
# DISK SPACE ANALYSIS MODULE
###########################################

audit_disk_space() {
    print_header "DISK SPACE ANALYSIS"
    
    {
        echo "DISK SPACE ANALYSIS"
        echo "=========================================="
        echo ""
        
        # Filesystem usage overview
        echo "Filesystem Usage:"
        echo "----------------"
        df -h | grep -v "tmpfs\|devtmpfs"
        echo ""
        
        # Identify filesystems over threshold
        echo "Filesystems Over ${DISK_WARN_THRESHOLD}% Full:"
        echo "--------------------------------------------"
        df -h | awk -v thresh="$DISK_WARN_THRESHOLD" '
            NR>1 && $5+0 > thresh {
                print "⚠ " $6 " is " $5 " full (" $4 " available)"
            }
        '
        echo ""
        
        # Inode usage (can fill up even with free space)
        echo "Inode Usage:"
        echo "-----------"
        df -i | grep -v "tmpfs\|devtmpfs"
        echo ""
        
        # Check for inode exhaustion
        df -i | awk 'NR>1 && $5+0 > 80 {
            print "⚠ " $6 " inode usage: " $5
        }'
        echo ""
        
        # Top 20 largest files
        echo "Top 20 Largest Files:"
        echo "--------------------"
        print_info "Searching for large files (this may take several minutes)..."
        
        find / -type f -size +${LARGE_FILE_SIZE} 2>/dev/null \
            -exec du -h {} \; | sort -rh | head -20
        echo ""
        
        # Top 20 largest directories
        echo "Top 20 Largest Directories:"
        echo "--------------------------"
        du -h --max-depth=2 / 2>/dev/null | sort -rh | head -20
        echo ""
        
        # Specific directory analysis
        echo "Common Space-Consuming Directories:"
        echo "----------------------------------"
        
        dirs_to_check=(
            "/var/log"
            "/var/cache"
            "/tmp"
            "/var/tmp"
            "/home"
            "/root"
            "/var/lib/docker"
            "/opt"
        )
        
        for dir in "${dirs_to_check[@]}"; do
            if [[ -d "$dir" ]]; then
                size=$(du -sh "$dir" 2>/dev/null | awk '{print $1}')
                echo "$dir: $size"
            fi
        done
        echo ""
        
        # Old files (potential cleanup candidates)
        echo "Files Not Accessed in $OLD_FILE_DAYS Days:"
        echo "--------------------------------------------"
        find /var/log /tmp /var/tmp -type f -atime +${OLD_FILE_DAYS} 2>/dev/null \
            -exec ls -lh {} \; | head -20
        echo ""
        
        # Large log files
        echo "Large Log Files (>100MB):"
        echo "------------------------"
        find /var/log -type f -size +100M 2>/dev/null -exec ls -lh {} \;
        echo ""
        
        # Deleted files still held open (lsof required)
        if command -v lsof &> /dev/null; then
            echo "Deleted Files Still Open (Space Not Released):"
            echo "----------------------------------------------"
            lsof 2>/dev/null | grep deleted | \
                awk '{print $1, $2, $7, $9}' | head -20
            echo ""
            echo "⚠ These processes are holding deleted files"
            echo "   Space will not be freed until processes restart"
            echo ""
        fi
        
        # Core dumps
        echo "Core Dump Files:"
        echo "---------------"
        find / -name "core.*" -o -name "core" 2>/dev/null | head -20
        local core_count=$(find / -name "core.*" -o -name "core" 2>/dev/null | wc -l)
        if [[ $core_count -gt 0 ]]; then
            echo "⚠ Found $core_count core dump files"
            echo "   These are crash dumps and can be large"
        fi
        echo ""
        
        # Package manager cache
        echo "Package Manager Cache:"
        echo "---------------------"
        if [[ -d /var/cache/apt ]]; then
            apt_cache=$(du -sh /var/cache/apt 2>/dev/null | awk '{print $1}')
            echo "APT cache: $apt_cache"
            echo "   Clean with: sudo apt-get clean"
        fi
        
        if [[ -d /var/cache/yum ]]; then
            yum_cache=$(du -sh /var/cache/yum 2>/dev/null | awk '{print $1}')
            echo "YUM cache: $yum_cache"
            echo "   Clean with: sudo yum clean all"
        fi
        echo ""
        
        # Disk I/O statistics (if iostat available)
        if command -v iostat &> /dev/null; then
            echo "Disk I/O Statistics:"
            echo "-------------------"
            iostat -x 1 2 | tail -n +4
            echo ""
        fi
        
    } | tee -a "$REPORT_FILE"
    
    print_success "Disk space audit completed"
}
```

**Understanding Disk Commands:**

```bash
# df - Disk Free (filesystem level)
df -h                    # Human readable (GB, MB)
df -i                    # Inode usage
df -T                    # Show filesystem type

# du - Disk Usage (directory level)
du -h /var/log          # Human readable
du -sh /var/log         # Summary only
du -h --max-depth=1     # Limit depth

# find by size
find / -size +100M      # Larger than 100MB
find / -size -1M        # Smaller than 1MB
find / -size 50M        # Exactly 50MB

# find by time
find / -mtime -7        # Modified in last 7 days
find / -atime +365      # Accessed more than 365 days ago
find / -ctime -1        # Changed in last 24 hours
```

### Part 3: Advanced Disk Analysis

```bash
# Analyze directory growth over time
analyze_directory_growth() {
    local dir=$1
    
    echo "Directory Growth Analysis: $dir"
    echo "================================"
    echo ""
    
    # Files by age distribution
    echo "Files by Age:"
    echo "-------------"
    for days in 7 30 90 180 365; do
        count=$(find "$dir" -type f -mtime -$days 2>/dev/null | wc -l)
        size=$(find "$dir" -type f -mtime -$days 2>/dev/null -exec du -ch {} + | tail -1 | awk '{print $1}')
        echo "Last $days days: $count files, $size total"
    done
    echo ""
    
    # Largest subdirectories
    echo "Largest Subdirectories:"
    echo "----------------------"
    du -h --max-depth=1 "$dir" 2>/dev/null | sort -rh | head -10
    echo ""
}

# Identify duplicate files
find_duplicate_files() {
    echo "Potential Duplicate Files (by size):"
    echo "------------------------------------"
    
    find "$1" -type f -exec du -b {} \; 2>/dev/null | \
        sort -n | \
        awk 'prev==$1 {print $2} {prev=$1}' | \
        head -20
    echo ""
}

# Check for orphaned Docker artifacts
check_docker_space() {
    if command -v docker &> /dev/null; then
        echo "Docker Space Usage:"
        echo "------------------"
        docker system df 2>/dev/null || echo "Docker not running or no permission"
        echo ""
        
        echo "Dangling Images:"
        docker images -f "dangling=true" 2>/dev/null | wc -l
        echo ""
    fi
}
```

---

## Process & Network Security Module

### Part 1: Understanding Process Security

**Suspicious Process Indicators:**

```
┌─────────────────────────────────────────────────────┐
│  SUSPICIOUS PROCESS CHARACTERISTICS:                 │
├─────────────────────────────────────────────────────┤
│                                                      │
│  Resource Anomalies:                                │
│  ├─ Unexpected high CPU usage                      │
│  ├─ Unusual memory consumption                     │
│  └─ Process running longer than expected           │
│                                                      │
│  Permission Issues:                                 │
│  ├─ Regular user process running as root           │
│  ├─ Processes from /tmp or /dev/shm                │
│  └─ Deleted binary still running                   │
│                                                      │
│  Network Activity:                                  │
│  ├─ Unexpected listening ports                     │
│  ├─ Connections to unknown IPs                     │
│  ├─ High network traffic from user process         │
│  └─ Processes with many open connections           │
│                                                      │
│  Behavioral:                                        │
│  ├─ Renamed system utilities                       │
│  ├─ Multiple instances of same process             │
│  ├─ Processes with suspicious names                │
│  └─ Hidden processes (not in ps output)            │
└─────────────────────────────────────────────────────┘
```

### Part 2: Process & Network Audit Implementation

```bash
###########################################
# PROCESS & NETWORK SECURITY MODULE
###########################################

audit_processes_network() {
    print_header "PROCESS & NETWORK SECURITY AUDIT"
    
    {
        echo "PROCESS & NETWORK SECURITY AUDIT"
        echo "=========================================="
        echo ""
        
        # Top CPU consuming processes
        echo "Top 10 CPU-Consuming Processes:"
        echo "------------------------------"
        ps aux --sort=-%cpu | head -11 | \
            awk 'NR==1 {print $0} NR>1 {printf "%-10s %5s%% %5s%% %-10s %s\n", $1, $3, $4, $2, $11}'
        echo ""
        
        # Top memory consuming processes
        echo "Top 10 Memory-Consuming Processes:"
        echo "---------------------------------"
        ps aux --sort=-%mem | head -11 | \
            awk 'NR==1 {print $0} NR>1 {printf "%-10s %5s%% %5s%% %-10s %s\n", $1, $3, $4, $2, $11}'
        echo ""
        
        # Processes running as root
        echo "Processes Running as Root:"
        echo "-------------------------"
        ps aux | awk '$1 == "root" {print $2, $11}' | head -20
        echo ""
        echo "Total root processes: $(ps aux | awk '$1 == "root"' | wc -l)"
        echo ""
        
        # Suspicious process locations
        echo "Processes Running from Suspicious Locations:"
        echo "--------------------------------------------"
        ps aux | awk '{print $2, $11}' | \
        while read -r pid cmd; do
            if [[ "$cmd" =~ /tmp/|/dev/shm/|/var/tmp/ ]]; then
                echo "⚠ PID $pid: $cmd"
            fi
        done
        echo ""
        
        # Deleted binaries still running
        echo "Processes with Deleted Binaries:"
        echo "-------------------------------"
        if command -v lsof &> /dev/null; then
            lsof 2>/dev/null | grep DEL | grep bin | \
                awk '{print $2, $1, $9}' | sort -u | head -20
            echo ""
            echo "⚠ These processes are running deleted executables"
            echo "   Could indicate rootkit or improper update"
        else
            echo "lsof not available, skipping"
        fi
        echo ""
        
        # Network listening ports
        echo "Listening Network Ports:"
        echo "-----------------------"
        if command -v ss &> /dev/null; then
            ss -tlnp 2>/dev/null | grep LISTEN | \
                awk '{print $4, $6}' | column -t
        elif command -v netstat &> /dev/null; then
            netstat -tlnp 2>/dev/null | grep LISTEN | \
                awk '{print $4, $7}' | column -t
        else
            echo "Neither ss nor netstat available"
        fi
        echo ""
        
        # Established network connections
        echo "Active Network Connections:"
        echo "--------------------------"
        if command -v ss &> /dev/null; then
            ss -tnp 2>/dev/null | grep ESTAB | \
                awk '{print $4, $5, $6}' | head -20 | column -t
        else
            netstat -tnp 2>/dev/null | grep ESTABLISHED | \
                awk '{print $4, $5, $7}' | head -20 | column -t
        fi
        echo ""
        
        # Check for unexpected listening ports
        echo "Unexpected Listening Ports:"
        echo "--------------------------"
        
        # Common legitimate ports
        known_ports=(22 80 443 3306 5432 6379 27017)
        
        if command -v ss &> /dev/null; then
            ss -tlnp 2>/dev/null | grep LISTEN | \
            while read -r line; do
                port=$(echo "$line" | awk '{print $4}' | rev | cut -d: -f1 | rev)
                
                # Check if port is in known list
                is_known=0
                for known in "${known_ports[@]}"; do
                    if [[ "$port" == "$known" ]]; then
                        is_known=1
                        break
                    fi
                done
                
                if [[ $is_known -eq 0 ]]; then
                    echo "$line"
                fi
            done
        fi
        echo ""
        
        # Process with most open connections
        echo "Processes with Most Network Connections:"
        echo "---------------------------------------"
        if command -v lsof &> /dev/null; then
            lsof -i 2>/dev/null | awk 'NR>1 {print $1}' | \
                sort | uniq -c | sort -rn | head -10
        else
            echo "lsof not available, skipping"
        fi
        echo ""
        
        # Check for processes listening on all interfaces
        echo "Processes Listening on All Interfaces (0.0.0.0):"
        echo "-----------------------------------------------"
        if command -v ss &> /dev/null; then
            ss -tlnp 2>/dev/null | grep "0.0.0.0" | \
                awk '{print $4, $6}'
        fi
        echo ""
        echo "⚠ Listening on 0.0.0.0 exposes service to all networks"
        echo "   Consider binding to specific interfaces for security"
        echo ""
        
        # Unusual process names
        echo "Processes with Unusual Names:"
        echo "----------------------------"
        ps aux | awk '{print $11}' | sort -u | \
        while read -r proc; do
            # Check for single character names (often suspicious)
            if [[ "${#proc}" -eq 1 ]] || [[ "$proc" =~ ^[[:punct:]] ]]; then
                echo "⚠ Suspicious: $proc"
            fi
        done
        echo ""
        
        # Long-running processes (potential zombie or hung)
        echo "Long-Running Processes (>7 days):"
        echo "--------------------------------"
        ps -eo pid,etime,user,command --sort=-etime | head -20
        echo ""
        
    } | tee -a "$REPORT_FILE"
    
    print_success "Process and network audit completed"
}
```

**Understanding Network Commands:**

```bash
# ss (Socket Statistics) - modern replacement for netstat
ss -t        # TCP sockets
ss -u        # UDP sockets
ss -l        # Listening sockets
ss -n        # Numeric (don't resolve names)
ss -p        # Show process using socket

# Combined:
ss -tlnp     # TCP, listening, numeric, process

# netstat (older, but still common)
netstat -t   # TCP
netstat -l   # Listening
netstat -n   # Numeric
netstat -p   # Process

# lsof (List Open Files) - includes network
lsof -i      # Internet connections
lsof -i :80  # Specific port
lsof -u user # By user
```

### Part 3: Advanced Process Analysis

```bash
# Check for hidden processes (rootkit detection)
check_hidden_processes() {
    echo "Hidden Process Detection:"
    echo "------------------------"
    
    # Compare ps output with /proc directory
    ps_pids=$(ps aux | awk 'NR>1 {print $2}' | sort -n)
    proc_pids=$(ls -1 /proc | grep -E '^[0-9]+$' | sort -n)
    
    # Find PIDs in /proc but not in ps (potentially hidden)
    hidden=$(comm -13 <(echo "$ps_pids") <(echo "$proc_pids"))
    
    if [[ -n "$hidden" ]]; then
        echo "🚨 CRITICAL: Hidden processes detected!"
        echo "$hidden"
    else
        echo "✓ No hidden processes detected"
    fi
    echo ""
}

# Analyze process trees
analyze_process_tree() {
    echo "Suspicious Process Trees:"
    echo "------------------------"
    
    # Find processes spawned from unusual parents
    ps -ef --forest | \
    awk '{
        if ($3 == 1 && $8 !~ /systemd|init|cron/) {
            print "⚠ Unusual parent: " $0
        }
    }'
    echo ""
}

# Check for cryptocurrency miners (CPU-intensive, network-heavy)
check_crypto_miners() {
    echo "Potential Cryptocurrency Miner Detection:"
    echo "----------------------------------------"
    
    # Common miner names
    miner_patterns="xmrig|minerd|cpuminer|ccminer|cryptonight"
    
    ps aux | grep -iE "$miner_patterns" | grep -v grep
    
    # High CPU processes with network connections
    ps aux --sort=-%cpu | head -20 | \
    while read -r line; do
        pid=$(echo "$line" | awk '{print $2}')
        cpu=$(echo "$line" | awk '{print $3}')
        
        if (( $(echo "$cpu > 50" | bc -l) )); then
            connections=$(lsof -i -a -p "$pid" 2>/dev/null | wc -l)
            if [[ $connections -gt 0 ]]; then
                echo "⚠ High CPU ($cpu%) with network activity: $line"
            fi
        fi
    done
    echo ""
}
```

---

## Complete Script

### Final Production-Ready Script

```bash
#!/bin/bash
###########################################
# SYSTEM SECURITY AUDIT SCRIPT
# Version: 2.0
# Description: Comprehensive system security audit
# Usage: sudo ./system-audit.sh
###########################################

set -euo pipefail  # Exit on error, undefined vars, pipe failures

###########################################
# CONFIGURATION
###########################################

SCRIPT_NAME="System Security Audit"
VERSION="2.0"
AUDIT_DATE=$(date '+%Y-%m-%d %H:%M:%S')

# Output
OUTPUT_DIR="/var/log/system-audit"
REPORT_FILE="${OUTPUT_DIR}/audit-$(date '+%Y%m%d-%H%M%S').txt"

# Thresholds
LARGE_FILE_SIZE="100M"
DISK_WARN_THRESHOLD=80
OLD_FILE_DAYS=365
MAX_SUID_FILES=100

# Colors
RED='\033[0;31m'
YELLOW='\033[1;33m'
GREEN='\033[0;32m'
BLUE='\033[0;34m'
NC='\033[0m'

###########################################
# HELPER FUNCTIONS
###########################################

print_header() {
    echo ""
    echo "=========================================="
    echo "$1"
    echo "=========================================="
    echo ""
}

print_success() { echo -e "${GREEN}✓ $1${NC}"; }
print_warning() { echo -e "${YELLOW}⚠ $1${NC}"; }
print_error() { echo -e "${RED}✗ $1${NC}"; }
print_info() { echo -e "${BLUE}ℹ $1${NC}"; }

check_root() {
    if [[ $EUID -ne 0 ]]; then
        print_error "This script must be run as root (sudo)"
        exit 1
    fi
}

setup_output() {
    mkdir -p "$OUTPUT_DIR"
    print_success "Output directory: $OUTPUT_DIR"
}

log_output() {
    echo "$1" | tee -a "$REPORT_FILE"
}

###########################################
# AUDIT MODULES
###########################################

audit_users() {
    print_header "USER AUDIT"
    
    {
        echo "USER AUDIT"
        echo "=========================================="
        echo ""
        
        total_users=$(wc -l < /etc/passwd)
        echo "Total users: $total_users"
        echo ""
        
        echo "Users with Login Shells:"
        awk -F: '$7 !~ /nologin|false/ {print $1 " (UID: " $3 ", Shell: " $7 ")"}' /etc/passwd
        echo ""
        
        echo "Users with UID 0 (Root Privileges):"
        awk -F: '$3 == 0 {print $1}' /etc/passwd
        uid0_count=$(awk -F: '$3 == 0' /etc/passwd | wc -l)
        [[ $uid0_count -gt 1 ]] && echo "⚠ WARNING: Multiple UID 0 accounts!"
        echo ""
        
        echo "Last Login Times:"
        lastlog | awk 'NR==1 || $2 != "**Never"'
        echo ""
        
        echo "Currently Logged In:"
        who
        echo ""
        
        if [[ -r /etc/shadow ]]; then
            echo "Password Security:"
            awk -F: '$2 == "" {print "🚨 " $1 ": Empty password!"}' /etc/shadow
            echo ""
        fi
        
    } | tee -a "$REPORT_FILE"
    
    print_success "User audit completed"
}

audit_permissions() {
    print_header "PERMISSION SECURITY AUDIT"
    
    {
        echo "PERMISSION SECURITY AUDIT"
        echo "=========================================="
        echo ""
        
        echo "World-Writable Files:"
        find /etc /home /root /usr /var /opt /tmp 2>/dev/null \
            -type f -perm -002 ! -type l -ls | head -50
        
        ww_count=$(find /etc /home /root /usr /var /opt /tmp 2>/dev/null \
            -type f -perm -002 ! -type l | wc -l)
        echo ""
        echo "Total: $ww_count files"
        [[ $ww_count -gt 0 ]] && echo "🚨 CRITICAL: World-writable files found!"
        echo ""
        
        echo "SUID Files:"
        find / -type f -perm -4000 2>/dev/null -ls | head -50
        suid_count=$(find / -type f -perm -4000 2>/dev/null | wc -l)
        echo ""
        echo "Total: $suid_count files"
        [[ $suid_count -gt $MAX_SUID_FILES ]] && echo "⚠ WARNING: High SUID count!"
        echo ""
        
        echo "SGID Files:"
        find / -type f -perm -2000 2>/dev/null -ls | head -50
        echo ""
        
        echo "Orphaned Files:"
        find / -nouser -o -nogroup 2>/dev/null | head -50
        orphan_count=$(find / -nouser -o -nogroup 2>/dev/null | wc -l)
        echo ""
        echo "Total: $orphan_count files"
        echo ""
        
        echo "Critical System Files Check:"
        for file in /etc/passwd /etc/shadow /etc/sudoers; do
            if [[ -f "$file" ]]; then
                ls -l "$file"
            fi
        done
        echo ""
        
    } | tee -a "$REPORT_FILE"
    
    print_success "Permission audit completed"
}

audit_disk_space() {
    print_header "DISK SPACE ANALYSIS"
    
    {
        echo "DISK SPACE ANALYSIS"
        echo "=========================================="
        echo ""
        
        echo "Filesystem Usage:"
        df -h | grep -v "tmpfs\|devtmpfs"
        echo ""
        
        echo "Filesystems Over ${DISK_WARN_THRESHOLD}%:"
        df -h | awk -v thresh="$DISK_WARN_THRESHOLD" \
            'NR>1 && $5+0 > thresh {print "⚠ " $6 " is " $5 " full"}'
        echo ""
        
        echo "Top 20 Largest Files:"
        find / -type f -size +${LARGE_FILE_SIZE} 2>/dev/null \
            -exec du -h {} \; | sort -rh | head -20
        echo ""
        
        echo "Top 20 Largest Directories:"
        du -h --max-depth=2 / 2>/dev/null | sort -rh | head -20
        echo ""
        
        echo "Large Log Files:"
        find /var/log -type f -size +100M 2>/dev/null -ls
        echo ""
        
    } | tee -a "$REPORT_FILE"
    
    print_success "Disk space audit completed"
}

audit_processes_network() {
    print_header "PROCESS & NETWORK AUDIT"
    
    {
        echo "PROCESS & NETWORK AUDIT"
        echo "=========================================="
        echo ""
        
        echo "Top CPU Processes:"
        ps aux --sort=-%cpu | head -11
        echo ""
        
        echo "Top Memory Processes:"
        ps aux --sort=-%mem | head -11
        echo ""
        
        echo "Root Processes:"
        ps aux | awk '$1 == "root"' | head -20
        echo ""
        
        echo "Listening Ports:"
        if command -v ss &> /dev/null; then
            ss -tlnp 2>/dev/null | grep LISTEN
        elif command -v netstat &> /dev/null; then
            netstat -tlnp 2>/dev/null | grep LISTEN
        fi
        echo ""
        
        echo "Active Connections:"
        if command -v ss &> /dev/null; then
            ss -tnp 2>/dev/null | grep ESTAB | head -20
        fi
        echo ""
        
    } | tee -a "$REPORT_FILE"
    
    print_success "Process/network audit completed"
}

###########################################
# MAIN EXECUTION
###########################################

main() {
    print_header "$SCRIPT_NAME v$VERSION"
    print_info "Audit started: $AUDIT_DATE"
    
    check_root
    setup_output
    
    # Initialize report
    {
        echo "================================================================"
        echo "  SYSTEM SECURITY AUDIT REPORT"
        echo "================================================================"
        echo "Generated: $AUDIT_DATE"
        echo "Hostname: $(hostname)"
        echo "OS: $(cat /etc/os-release 2>/dev/null | grep PRETTY_NAME | cut -d'"' -f2)"
        echo "Kernel: $(uname -r)"
        echo "================================================================"
        echo ""
    } > "$REPORT_FILE"
    
    # Run audits
    audit_users
    audit_permissions
    audit_disk_space
    audit_processes_network
    
    # Summary
    print_header "AUDIT COMPLETE"
    print_success "Report saved to: $REPORT_FILE"
    
    # Generate summary
    {
        echo ""
        echo "SUMMARY & RECOMMENDATIONS"
        echo "=========================================="
        echo ""
        
        # Critical findings
        ww_count=$(find /etc /home /root /usr /var /opt /tmp 2>/dev/null -type f -perm -002 ! -type l | wc -l)
        uid0_count=$(awk -F: '$3 == 0' /etc/passwd | wc -l)
        
        echo "Critical Findings:"
        [[ $ww_count -gt 0 ]] && echo "  🚨 $ww_count world-writable files"
        [[ $uid0_count -gt 1 ]] && echo "  🚨 Multiple UID 0 accounts detected"
        
        echo ""
        echo "Recommendations:"
        echo "  1. Review and fix world-writable files"
        echo "  2. Audit SUID/SGID binaries"
        echo "  3. Remove orphaned files"
        echo "  4. Monitor large files and disk usage"
        echo "  5. Review listening network ports"
        echo "  6. Schedule regular audits"
        echo ""
        
    } | tee -a "$REPORT_FILE"
}

main "$@"
```

---

## Testing & Validation

### Part 1: Creating Test Scenarios

```bash
#!/bin/bash
# test-scenarios.sh - Create test security issues

# Create world-writable file
sudo touch /tmp/world-writable-test.txt
sudo chmod 777 /tmp/world-writable-test.txt

# Create SUID test file
sudo cp /bin/bash /tmp/suid-test
sudo chmod 4755 /tmp/suid-test

# Create orphaned file
sudo useradd testuser
sudo -u testuser touch /tmp/orphan-test.txt
sudo userdel testuser

# Create large file
dd if=/dev/zero of=/tmp/large-file-test.dat bs=1M count=500

# Create old file
sudo touch -d "2020-01-01" /tmp/old-file-test.txt

echo "Test scenarios created!"
echo "Run audit script to detect these issues"
```

### Part 2: Validation Checklist

```bash
# Validation script
#!/bin/bash
# validate-audit.sh - Verify audit script detects issues

echo "Running validation tests..."

# Test 1: Root check
echo "Test 1: Non-root execution should fail"
./system-audit.sh 2>&1 | grep -q "must be run as root" && echo "✓ PASS" || echo "✗ FAIL"

# Test 2: World-writable detection
echo "Test 2: Detect world-writable files"
sudo ./system-audit.sh 2>&1 | grep -q "world-writable-test" && echo "✓ PASS" || echo "✗ FAIL"

# Test 3: SUID detection
echo "Test 3: Detect SUID files"
sudo ./system-audit.sh 2>&1 | grep -q "suid-test" && echo "✓ PASS" || echo "✗ FAIL"

# Test 4: Large file detection
echo "Test 4: Detect large files"
sudo ./system-audit.sh 2>&1 | grep -q "large-file-test" && echo "✓ PASS" || echo "✗ FAIL"

# Test 5: Report file creation
echo "Test 5: Report file created"
[[ -f /var/log/system-audit/audit-*.txt ]] && echo "✓ PASS" || echo "✗ FAIL"

echo "Validation complete!"
```

---

## Advanced Techniques

### Part 1: Automated Scheduling

```bash
# Schedule weekly audits with cron
# Edit crontab: sudo crontab -e

# Run every Monday at 2 AM
0 2 * * 1 /usr/local/bin/system-audit.sh > /dev/null 2>&1

# Run daily at midnight, email results
0 0 * * * /usr/local/bin/system-audit.sh | mail -s "Daily Audit Report" admin@example.com
```

### Part 2: Alert Integration

```bash
# Add to script for Slack notifications
send_slack_alert() {
    local message=$1
    local webhook_url="YOUR_SLACK_WEBHOOK_URL"
    
    curl -X POST -H 'Content-type: application/json' \
        --data "{\"text\":\"$message\"}" \
        "$webhook_url"
}

# Use in script:
if [[ $ww_count -gt 0 ]]; then
    send_slack_alert "🚨 Security Alert: $ww_count world-writable files detected!"
fi
```

### Part 3: Baseline Comparison

```bash
# Create baseline
sudo ./system-audit.sh
cp "$REPORT_FILE" /var/log/system-audit/baseline.txt

# Compare against baseline
diff -u /var/log/system-audit/baseline.txt "$REPORT_FILE" > /var/log/system-audit/changes.txt

# Alert on changes
if [[ -s /var/log/system-audit/changes.txt ]]; then
    echo "⚠ Changes detected since baseline!"
    cat /var/log/system-audit/changes.txt
fi
```

---

## Cheat Sheet

```bash
# QUICK REFERENCE

# Run full audit
sudo ./system-audit.sh

# Find world-writable files
find / -type f -perm -002 2>/dev/null

# Find SUID files
find / -type f -perm -4000 2>/dev/null

# Find SGID files
find / -type f -perm -2000 2>/dev/null

# Find orphaned files
find / -nouser -o -nogroup 2>/dev/null

# Check user logins
lastlog

# Check currently logged in
who

# Check UID 0 accounts
awk -F: '$3 == 0 {print $1}' /etc/passwd

# Disk usage top 10
du -h / --max-depth=1 2>/dev/null | sort -rh | head -10

# Large files
find / -size +100M 2>/dev/null -ls

# Listening ports
ss -tlnp

# Top processes
ps aux --sort=-%cpu | head -10

# Network connections
ss -tnp | grep ESTAB
```

---

## Conclusion

You now have:
- ✅ Complete system audit script
- ✅ User security analysis
- ✅ Permission vulnerability detection
- ✅ Disk space monitoring
- ✅ Process and network security
- ✅ Automated reporting
- ✅ Testing and validation

**Next Steps:**
1. Install script: `sudo cp system-audit.sh /usr/local/bin/`
2. Make executable: `sudo chmod +x /usr/local/bin/system-audit.sh`
3. Run first audit: `sudo system-audit.sh`
4. Review report: `less /var/log/system-audit/audit-*.txt`
5. Schedule regular audits: `sudo crontab -e`
6. Create baseline: `cp report baseline.txt`

**Remember:** Security is ongoing. Run audits regularly and investigate anomalies immediately! 