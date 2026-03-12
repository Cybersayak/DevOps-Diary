# User Management Automation Mastery Guide

## Learning Objectives
By the end of this 1-hour session, you will:
- Automate user creation with complete home directory setup
- Implement group-based access control
- Configure granular sudo permissions
- Build a complete user provisioning system

---

## Part 1: User Management Fundamentals (15 mins)

### Core Concepts

**What defines a Linux user?**

```bash
# User information stored in these files:
/etc/passwd    # User accounts (readable by all)
/etc/shadow    # Encrypted passwords (root only)
/etc/group     # Group definitions
/etc/gshadow   # Group passwords (rarely used)
```

**Anatomy of /etc/passwd entry:**
```
username:x:UID:GID:comment:home_directory:shell
   │     │  │   │     │          │          │
   │     │  │   │     │          │          └─ Login shell
   │     │  │   │     │          └─ Home directory path
   │     │  │   │     └─ GECOS field (full name, info)
   │     │  │   └─ Primary group ID
   │     │  └─ User ID
   │     └─ Password placeholder (actual in /etc/shadow)
   └─ Username
```

**Example:**
```bash
grep "john" /etc/passwd
# john:x:1001:1001:John Smith,DevOps:/home/john:/bin/bash
```

### UID/GID Ranges

| Range | Purpose |
|-------|---------|
| 0 | Root user |
| 1-999 | System/service accounts |
| 1000+ | Regular users |
| 65534 | nobody (special) |

### Essential Commands Overview

```bash
# User management
useradd     # Create user (low-level)
adduser     # Create user (interactive, Debian-friendly)
usermod     # Modify existing user
userdel     # Delete user
passwd      # Set/change password

# Group management
groupadd    # Create group
groupmod    # Modify group
groupdel    # Delete group
gpasswd     # Manage group membership

# Information
id          # Show user/group IDs
groups      # Show group membership
getent      # Query user/group databases
```

---

## Part 2: Automated User Creation (15 mins)

### useradd Deep Dive

**Basic syntax:**
```bash
useradd [OPTIONS] USERNAME
```

**Critical options:**

| Option | Purpose | Example |
|--------|---------|---------|
| `-m` | Create home directory | `-m` |
| `-d` | Custom home path | `-d /data/users/john` |
| `-s` | Login shell | `-s /bin/bash` |
| `-g` | Primary group | `-g developers` |
| `-G` | Supplementary groups | `-G docker,sudo` |
| `-c` | Comment/full name | `-c "John Smith"` |
| `-e` | Account expiry date | `-e 2025-12-31` |
| `-u` | Specific UID | `-u 1500` |
| `-r` | System account | `-r` (no home, UID < 1000) |

### Complete User Creation Script

```bash
#!/bin/bash
# create_user.sh - Production-ready user creation

set -euo pipefail

# ============ Configuration ============
SKEL_DIR="/etc/skel"
DEFAULT_SHELL="/bin/bash"
DEFAULT_GROUPS="users"
MIN_PASSWORD_LENGTH=12

# ============ Functions ============
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

error_exit() {
    log "ERROR: $1" >&2
    exit 1
}

validate_username() {
    local username="$1"
    
    # Check format (lowercase, numbers, underscore, hyphen)
    if [[ ! "$username" =~ ^[a-z_][a-z0-9_-]{2,31}$ ]]; then
        error_exit "Invalid username format: $username"
    fi
    
    # Check if exists
    if id "$username" &>/dev/null; then
        error_exit "User already exists: $username"
    fi
}

generate_password() {
    # Generate secure random password
    openssl rand -base64 16 | tr -dc 'a-zA-Z0-9!@#$%' | head -c "$MIN_PASSWORD_LENGTH"
}

create_user() {
    local username="$1"
    local fullname="$2"
    local groups="${3:-$DEFAULT_GROUPS}"
    local shell="${4:-$DEFAULT_SHELL}"
    
    log "Creating user: $username"
    
    # Create user with home directory
    useradd \
        --create-home \
        --shell "$shell" \
        --comment "$fullname" \
        --groups "$groups" \
        "$username"
    
    # Generate and set password
    local password
    password=$(generate_password)
    echo "${username}:${password}" | chpasswd
    
    # Force password change on first login
    chage -d 0 "$username"
    
    # Set proper home directory permissions
    chmod 750 "/home/$username"
    
    log "User created successfully"
    echo "USERNAME: $username"
    echo "PASSWORD: $password"
    echo "NOTE: Password change required on first login"
}

setup_ssh_key() {
    local username="$1"
    local pubkey="$2"
    local ssh_dir="/home/$username/.ssh"
    
    log "Setting up SSH key for $username"
    
    mkdir -p "$ssh_dir"
    echo "$pubkey" >> "$ssh_dir/authorized_keys"
    
    # Critical: Fix permissions
    chmod 700 "$ssh_dir"
    chmod 600 "$ssh_dir/authorized_keys"
    chown -R "${username}:${username}" "$ssh_dir"
    
    log "SSH key configured"
}

# ============ Main ============
main() {
    # Check root
    [[ $EUID -eq 0 ]] || error_exit "Must run as root"
    
    # Parse arguments
    if [[ $# -lt 2 ]]; then
        echo "Usage: $0 <username> <fullname> [groups] [shell]"
        echo "Example: $0 jsmith 'John Smith' 'developers,docker' /bin/bash"
        exit 1
    fi
    
    local username="$1"
    local fullname="$2"
    local groups="${3:-}"
    local shell="${4:-}"
    
    validate_username "$username"
    create_user "$username" "$fullname" "$groups" "$shell"
}

main "$@"
```

**Usage:**
```bash
sudo ./create_user.sh jsmith "John Smith" "developers,docker"
# Output:
# [2025-01-15 10:30:00] Creating user: jsmith
# [2025-01-15 10:30:00] User created successfully
# USERNAME: jsmith
# PASSWORD: xK9#mPq2vR7nL4
# NOTE: Password change required on first login
```

### Home Directory Skeleton Setup

```bash
# /etc/skel contains files copied to new home directories
ls -la /etc/skel/
# .bash_logout  .bashrc  .profile

# Customize skeleton for your organization
sudo mkdir -p /etc/skel/{.config,.local/bin,projects}

# Add custom .bashrc additions
sudo tee -a /etc/skel/.bashrc << 'EOF'

# Company customizations
export EDITOR=vim
export PATH="$HOME/.local/bin:$PATH"
alias ll='ls -alF'

# Load local environment
[[ -f ~/.env.local ]] && source ~/.env.local
EOF

# Add welcome message
sudo tee /etc/skel/.config/welcome.txt << 'EOF'
Welcome to the development server!
Documentation: https://wiki.company.com
Support: helpdesk@company.com
EOF
```

---

## Part 3: Group-Based Permission Management (15 mins)

### Group Hierarchy Design

**Common organizational structure:**
```
all-staff           # Base group for all employees
├── developers      # Development team
│   ├── dev-frontend
│   ├── dev-backend
│   └── dev-devops
├── operations      # Ops team
└── contractors     # External contractors (limited access)
```

### Group Management Script

```bash
#!/bin/bash
# setup_groups.sh - Organizational group structure

set -euo pipefail

# ============ Group Definitions ============
declare -A GROUPS=(
    ["all-staff"]="1100"
    ["developers"]="1200"
    ["dev-frontend"]="1201"
    ["dev-backend"]="1202"
    ["dev-devops"]="1203"
    ["operations"]="1300"
    ["contractors"]="1400"
    ["docker"]="1500"
    ["database-admins"]="1600"
)

# Shared directory structure
declare -A SHARED_DIRS=(
    ["/srv/projects"]="developers:2775"
    ["/srv/shared"]="all-staff:2775"
    ["/srv/ops"]="operations:2770"
    ["/srv/databases"]="database-admins:2750"
)

# ============ Functions ============
create_groups() {
    echo "Creating organizational groups..."
    
    for group in "${!GROUPS[@]}"; do
        local gid="${GROUPS[$group]}"
        
        if getent group "$group" &>/dev/null; then
            echo "  Group exists: $group"
        else
            groupadd --gid "$gid" "$group"
            echo "  Created: $group (GID: $gid)"
        fi
    done
}

setup_shared_directories() {
    echo "Setting up shared directories..."
    
    for dir in "${!SHARED_DIRS[@]}"; do
        IFS=':' read -r group perms <<< "${SHARED_DIRS[$dir]}"
        
        mkdir -p "$dir"
        chgrp "$group" "$dir"
        chmod "$perms" "$dir"
        
        # Set default ACL for new files
        setfacl -d -m g:"$group":rwx "$dir"
        
        echo "  $dir -> $group ($perms)"
    done
}

# Understanding permissions
explain_permissions() {
    cat << 'EOF'
Permission Reference:
  2775 = setgid + rwxrwxr-x
         │       │││││││││
         │       │││││││││── others: read + execute
         │       ││││││└└─── group: read + write + execute  
         │       │││└└└───── owner: read + write + execute
         └─────────────────── setgid: new files inherit group

  2770 = setgid + rwxrwx--- (no other access)
  2750 = setgid + rwxr-x--- (group read-only, no others)
EOF
}

# ============ Main ============
main() {
    [[ $EUID -eq 0 ]] || { echo "Run as root"; exit 1; }
    
    create_groups
    setup_shared_directories
    explain_permissions
    
    echo "Group setup complete!"
}

main "$@"
```

### User-Group Assignment

```bash
#!/bin/bash
# assign_groups.sh - Assign users to appropriate groups

assign_user_groups() {
    local username="$1"
    local role="$2"
    
    # Base group for all
    local groups="all-staff"
    
    # Role-based group assignment
    case "$role" in
        "frontend-dev")
            groups+=",developers,dev-frontend"
            ;;
        "backend-dev")
            groups+=",developers,dev-backend,docker"
            ;;
        "devops")
            groups+=",developers,dev-devops,docker,operations"
            ;;
        "dba")
            groups+=",database-admins"
            ;;
        "ops")
            groups+=",operations"
            ;;
        "contractor")
            groups="contractors"  # Note: no all-staff
            ;;
        *)
            echo "Unknown role: $role"
            return 1
            ;;
    esac
    
    usermod -aG "$groups" "$username"
    echo "Assigned $username to groups: $groups"
}

# Usage
assign_user_groups "jsmith" "devops"
```

### ACL (Access Control Lists) for Fine-Grained Control

```bash
# View current ACLs
getfacl /srv/projects

# Grant specific user access
setfacl -m u:contractor1:rx /srv/projects

# Grant group access
setfacl -m g:dev-frontend:rwx /srv/projects/frontend

# Set default ACL (for new files)
setfacl -d -m g:developers:rw /srv/projects

# Remove ACL
setfacl -x u:contractor1 /srv/projects

# Remove all ACLs
setfacl -b /srv/projects

# Copy ACL from one file to another
getfacl file1 | setfacl --set-file=- file2
```

---

## Part 4: Sudo Configuration - Least Privilege (10 mins)

### Sudoers File Structure

```bash
# /etc/sudoers structure
# NEVER edit directly - always use visudo

# Format:
# WHO  WHERE=(AS_WHOM)  WHAT
# user host=(runas)     commands
```

### Safe Sudoers Management

```bash
#!/bin/bash
# configure_sudo.sh - Least privilege sudo setup

SUDOERS_DIR="/etc/sudoers.d"

# ============ Sudo Rules ============

# Developers - limited sudo
create_dev_sudoers() {
    local file="$SUDOERS_DIR/10-developers"
    
    cat > "$file" << 'EOF'
# Developers sudo rules
# Allow service management for specific services
%developers ALL=(root) NOPASSWD: /bin/systemctl restart nginx
%developers ALL=(root) NOPASSWD: /bin/systemctl restart php-fpm
%developers ALL=(root) NOPASSWD: /bin/systemctl status *

# Allow viewing logs
%developers ALL=(root) NOPASSWD: /bin/journalctl -u nginx*
%developers ALL=(root) NOPASSWD: /bin/journalctl -u php-fpm*
%developers ALL=(root) NOPASSWD: /usr/bin/tail -f /var/log/nginx/*

# Allow docker commands without sudo
%developers ALL=(root) NOPASSWD: /usr/bin/docker *
%developers ALL=(root) NOPASSWD: /usr/bin/docker-compose *
EOF
    
    chmod 440 "$file"
    visudo -cf "$file" && echo "Developers sudo configured"
}

# DevOps - broader but controlled access
create_devops_sudoers() {
    local file="$SUDOERS_DIR/20-devops"
    
    cat > "$file" << 'EOF'
# DevOps sudo rules
# Full systemctl access
%dev-devops ALL=(root) NOPASSWD: /bin/systemctl *

# Package management
%dev-devops ALL=(root) NOPASSWD: /usr/bin/apt update
%dev-devops ALL=(root) NOPASSWD: /usr/bin/apt install *
%dev-devops ALL=(root) NOPASSWD: /usr/bin/apt upgrade -y

# User management (cannot modify root or sudo group)
%dev-devops ALL=(root) NOPASSWD: /usr/sbin/useradd *
%dev-devops ALL=(root) NOPASSWD: /usr/sbin/usermod *
%dev-devops ALL=(root) NOPASSWD: /usr/bin/passwd [a-z]*

# Networking
%dev-devops ALL=(root) NOPASSWD: /sbin/iptables -L *
%dev-devops ALL=(root) NOPASSWD: /bin/netstat *
%dev-devops ALL=(root) NOPASSWD: /bin/ss *

# DENIED: Cannot edit sudoers
%dev-devops ALL=(root) !NOPASSWD: /usr/sbin/visudo
%dev-devops ALL=(root) !NOPASSWD: /usr/bin/vim /etc/sudoers*
EOF
    
    chmod 440 "$file"
    visudo -cf "$file" && echo "DevOps sudo configured"
}

# Database admins - database-specific
create_dba_sudoers() {
    local file="$SUDOERS_DIR/30-dba"
    
    cat > "$file" << 'EOF'
# DBA sudo rules
%database-admins ALL=(root) NOPASSWD: /bin/systemctl * postgresql*
%database-admins ALL=(root) NOPASSWD: /bin/systemctl * mysql*
%database-admins ALL=(root) NOPASSWD: /bin/systemctl * mariadb*
%database-admins ALL=(root) NOPASSWD: /usr/bin/pg_dump *
%database-admins ALL=(root) NOPASSWD: /usr/bin/mysqldump *

# Allow switching to postgres user
%database-admins ALL=(postgres) NOPASSWD: ALL
EOF
    
    chmod 440 "$file"
    visudo -cf "$file" && echo "DBA sudo configured"
}

# Contractors - very limited
create_contractor_sudoers() {
    local file="$SUDOERS_DIR/90-contractors"
    
    cat > "$file" << 'EOF'
# Contractors - minimal access
# Only status commands, no changes
%contractors ALL=(root) NOPASSWD: /bin/systemctl status *
%contractors ALL=(root) NOPASSWD: /bin/journalctl --no-pager -n 100 *
EOF
    
    chmod 440 "$file"
    visudo -cf "$file" && echo "Contractor sudo configured"
}

# ============ Main ============
main() {
    [[ $EUID -eq 0 ]] || { echo "Run as root"; exit 1; }
    
    # Validate main sudoers file
    visudo -c || { echo "sudoers file corrupted!"; exit 1; }
    
    # Create group-specific rules
    create_dev_sudoers
    create_devops_sudoers
    create_dba_sudoers
    create_contractor_sudoers
    
    echo "Sudo configuration complete"
    echo "Files created in: $SUDOERS_DIR"
    ls -la "$SUDOERS_DIR"
}

main "$@"
```

### Sudoers Syntax Reference

```bash
# Allow commands (whitelist approach)
user ALL=(ALL) /path/to/command

# Deny commands (use ! prefix)
user ALL=(ALL) ALL, !/bin/su, !/bin/bash

# No password required
user ALL=(ALL) NOPASSWD: /path/to/command

# Require password
user ALL=(ALL) PASSWD: /path/to/command

# Run as specific user
user ALL=(appuser) /path/to/command

# Command aliases
Cmnd_Alias SERVICES = /bin/systemctl start *, /bin/systemctl stop *
%operators ALL=(root) NOPASSWD: SERVICES

# Host aliases (for multi-server)
Host_Alias WEBSERVERS = web1, web2, web3
%webadmins WEBSERVERS=(root) NOPASSWD: /bin/systemctl restart nginx
```

---

## Part 5: Complete User Provisioning System (5 mins)

### Production-Ready Provisioning Script

```bash
#!/bin/bash
# provision_user.sh - Complete user provisioning system
# Usage: ./provision_user.sh <config_file>
# Config format: username,fullname,email,role,ssh_pubkey

set -euo pipefail

# ============ Configuration ============
LOG_FILE="/var/log/user_provisioning.log"
REPORT_DIR="/var/reports/provisioning"
ADMIN_EMAIL="admin@company.com"

# Role definitions
declare -A ROLE_GROUPS=(
    ["frontend-dev"]="all-staff,developers,dev-frontend"
    ["backend-dev"]="all-staff,developers,dev-backend,docker"
    ["devops"]="all-staff,developers,dev-devops,docker,operations"
    ["dba"]="all-staff,database-admins"
    ["ops"]="all-staff,operations"
    ["contractor"]="contractors"
)

declare -A ROLE_SHELL=(
    ["frontend-dev"]="/bin/bash"
    ["backend-dev"]="/bin/bash"
    ["devops"]="/bin/bash"
    ["dba"]="/bin/bash"
    ["ops"]="/bin/bash"
    ["contractor"]="/bin/rbash"  # Restricted shell
)

# ============ Logging ============
log() {
    local level="$1"
    shift
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$level] $*" | tee -a "$LOG_FILE"
}

# ============ Validation ============
validate_input() {
    local username="$1"
    local email="$2"
    local role="$3"
    
    # Username format
    [[ "$username" =~ ^[a-z][a-z0-9_-]{2,31}$ ]] || {
        log "ERROR" "Invalid username: $username"
        return 1
    }
    
    # Email format
    [[ "$email" =~ ^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$ ]] || {
        log "ERROR" "Invalid email: $email"
        return 1
    }
    
    # Valid role
    [[ -v ROLE_GROUPS[$role] ]] || {
        log "ERROR" "Unknown role: $role"
        return 1
    }
    
    # User doesn't exist
    ! id "$username" &>/dev/null || {
        log "ERROR" "User exists: $username"
        return 1
    }
    
    return 0
}

# ============ User Creation ============
create_user_account() {
    local username="$1"
    local fullname="$2"
    local email="$3"
    local role="$4"
    
    local groups="${ROLE_GROUPS[$role]}"
    local shell="${ROLE_SHELL[$role]}"
    
    log "INFO" "Creating user: $username (role: $role)"
    
    # Create user
    useradd \
        --create-home \
        --shell "$shell" \
        --comment "$fullname <$email>" \
        --groups "$groups" \
        "$username"
    
    # Generate password
    local password
    password=$(openssl rand -base64 16 | tr -dc 'a-zA-Z0-9!@#$%' | head -c 16)
    echo "${username}:${password}" | chpasswd
    
    # Force password change
    chage -d 0 "$username"
    
    # Set account expiry for contractors (90 days)
    if [[ "$role" == "contractor" ]]; then
        local expiry_date
        expiry_date=$(date -d "+90 days" +%Y-%m-%d)
        chage -E "$expiry_date" "$username"
        log "INFO" "Contractor account expires: $expiry_date"
    fi
    
    echo "$password"
}

setup_ssh_access() {
    local username="$1"
    local pubkey="$2"
    
    [[ -n "$pubkey" ]] || return 0
    
    local ssh_dir="/home/$username/.ssh"
    
    mkdir -p "$ssh_dir"
    echo "$pubkey" > "$ssh_dir/authorized_keys"
    chmod 700 "$ssh_dir"
    chmod 600 "$ssh_dir/authorized_keys"
    chown -R "$username:$username" "$ssh_dir"
    
    log "INFO" "SSH key configured for $username"
}

setup_home_directory() {
    local username="$1"
    local role="$2"
    local home_dir="/home/$username"
    
    # Create standard directories
    sudo -u "$username" mkdir -p "$home_dir"/{projects,.local/bin,.config}
    
    # Role-specific setup
    case "$role" in
        "devops"|"backend-dev")
            # Git configuration
            sudo -u "$username" tee "$home_dir/.gitconfig" > /dev/null << EOF
[user]
    name = $(getent passwd "$username" | cut -d: -f5 | cut -d'<' -f1 | xargs)
    email = $(getent passwd "$username" | cut -d: -f5 | grep -oP '(?<=<)[^>]+')
[core]
    editor = vim
[init]
    defaultBranch = main
EOF
            ;;
        "contractor")
            # Restricted environment
            chmod 750 "$home_dir"
            # Remove write access to profile files
            chattr +i "$home_dir/.bashrc" "$home_dir/.profile" 2>/dev/null || true
            ;;
    esac
    
    log "INFO" "Home directory configured for $username"
}

# ============ Reporting ============
generate_welcome_email() {
    local username="$1"
    local fullname="$2"
    local email="$3"
    local password="$4"
    local role="$5"
    
    mkdir -p "$REPORT_DIR"
    local report_file="$REPORT_DIR/${username}_welcome.txt"
    
    cat > "$report_file" << EOF
================================================================================
                        WELCOME TO COMPANY SYSTEMS
================================================================================

Hello $fullname,

Your account has been created on our systems.

ACCOUNT DETAILS
===============
Username: $username
Role: $role
Server: $(hostname -f)

INITIAL LOGIN
=============
1. SSH to the server:
   ssh ${username}@$(hostname -f)

2. Use this temporary password: $password

3. You will be prompted to change your password on first login.

IMPORTANT SECURITY NOTES
========================
- Never share your password
- Use SSH keys for authentication when possible
- Report any security concerns to: security@company.com

RESOURCES
=========
- Documentation: https://wiki.company.com
- Support: helpdesk@company.com
- Emergency: +1-XXX-XXX-XXXX

================================================================================
Generated: $(date)
================================================================================
EOF
    
    chmod 600 "$report_file"
    log "INFO" "Welcome email generated: $report_file"
    
    # Send email (if mail is configured)
    if command -v mail &>/dev/null; then
        mail -s "Welcome to Company Systems" "$email" < "$report_file"
        log "INFO" "Welcome email sent to $email"
    fi
}

generate_admin_report() {
    local username="$1"
    local role="$2"
    
    local report="$REPORT_DIR/admin_report_$(date +%Y%m%d).txt"
    
    cat >> "$report" << EOF
---
User: $username
Role: $role
Created: $(date)
UID: $(id -u "$username")
Groups: $(id -Gn "$username" | tr ' ' ',')
Home: $(getent passwd "$username" | cut -d: -f6)
Shell: $(getent passwd "$username" | cut -d: -f7)
---

EOF
    
    log "INFO" "Admin report updated: $report"
}

# ============ Bulk Processing ============
process_config_file() {
    local config_file="$1"
    
    [[ -f "$config_file" ]] || {
        log "ERROR" "Config file not found: $config_file"
        exit 1
    }
    
    local success=0
    local failed=0
    
    while IFS=',' read -r username fullname email role ssh_pubkey; do
        # Skip header and comments
        [[ "$username" =~ ^#.*$ || "$username" == "username" ]] && continue
        [[ -z "$username" ]] && continue
        
        log "INFO" "Processing: $username"
        
        if validate_input "$username" "$email" "$role"; then
            local password
            password=$(create_user_account "$username" "$fullname" "$email" "$role")
            setup_ssh_access "$username" "$ssh_pubkey"
            setup_home_directory "$username" "$role"
            generate_welcome_email "$username" "$fullname" "$email" "$password" "$role"
            generate_admin_report "$username" "$role"
            ((success++))
            log "INFO" "Successfully provisioned: $username"
        else
            ((failed++))
            log "ERROR" "Failed to provision: $username"
        fi
        
    done < "$config_file"
    
    log "INFO" "Provisioning complete: $success success, $failed failed"
}

# ============ Interactive Mode ============
interactive_provision() {
    echo "=== User Provisioning System ==="
    echo
    
    read -rp "Username: " username
    read -rp "Full Name: " fullname
    read -rp "Email: " email
    
    echo "Available roles:"
    for role in "${!ROLE_GROUPS[@]}"; do
        echo "  - $role"
    done
    read -rp "Role: " role
    
    read -rp "SSH Public Key (optional, press Enter to skip): " ssh_pubkey
    
    echo
    echo "Review:"
    echo "  Username: $username"
    echo "  Full Name: $fullname"
    echo "  Email: $email"
    echo "  Role: $role"
    echo "  Groups: ${ROLE_GROUPS[$role]}"
    echo
    
    read -rp "Proceed? (y/N): " confirm
    [[ "$confirm" =~ ^[Yy]$ ]] || { echo "Cancelled"; exit 0; }
    
    if validate_input "$username" "$email" "$role"; then
        local password
        password=$(create_user_account "$username" "$fullname" "$email" "$role")
        setup_ssh_access "$username" "$ssh_pubkey"
        setup_home_directory "$username" "$role"
        generate_welcome_email "$username" "$fullname" "$email" "$password" "$role"
        
        echo
        echo "=== User Created Successfully ==="
        echo "Username: $username"
        echo "Password: $password"
        echo "NOTE: User must change password on first login"
    else
        echo "Provisioning failed"
        exit 1
    fi
}

# ============ Main ============
main() {
    [[ $EUID -eq 0 ]] || { echo "Run as root"; exit 1; }
    
    mkdir -p "$(dirname "$LOG_FILE")" "$REPORT_DIR"
    
    case "${1:-}" in
        -f|--file)
            [[ -n "${2:-}" ]] || { echo "Usage: $0 -f <config_file>"; exit 1; }
            process_config_file "$2"
            ;;
        -i|--interactive)
            interactive_provision
            ;;
        -h|--help)
            cat << EOF
User Provisioning System

Usage:
  $0 -i                     Interactive mode
  $0 -f <config_file>       Batch mode from CSV file

Config file format (CSV):
  username,fullname,email,role,ssh_pubkey

Available roles: ${!ROLE_GROUPS[*]}

Examples:
  $0 -i
  $0 -f new_users.csv
EOF
            ;;
        *)
            echo "Usage: $0 {-i|--interactive|-f|--file <config>|-h|--help}"
            exit 1
            ;;
    esac
}

main "$@"
```

### Sample Configuration File

```csv
# new_users.csv
username,fullname,email,role,ssh_pubkey
jsmith,John Smith,jsmith@company.com,devops,ssh-rsa AAAA...
agarcia,Ana Garcia,agarcia@company.com,backend-dev,ssh-ed25519 AAAA...
contractor1,External Dev,ext@vendor.com,contractor,
```

---

## Quick Reference Cheat Sheet

### User Commands
```bash
# Create
useradd -m -s /bin/bash -G group1,group2 -c "Full Name" username
adduser username                    # Interactive (Debian)

# Modify
usermod -aG newgroup username       # Add to group
usermod -s /bin/zsh username        # Change shell
usermod -L username                 # Lock account
usermod -U username                 # Unlock account

# Delete
userdel username                    # Keep home
userdel -r username                 # Remove home

# Password
passwd username                     # Set password
chage -l username                   # View password policy
chage -d 0 username                 # Force password change
chage -E 2025-12-31 username        # Set expiry
```

### Group Commands
```bash
groupadd groupname                  # Create
groupadd -g 1500 groupname          # With specific GID
groupdel groupname                  # Delete
gpasswd -a user group               # Add user to group
gpasswd -d user group               # Remove from group
```

### Permission Commands
```bash
chmod 750 /path                     # rwxr-x---
chmod 2775 /path                    # setgid + rwxrwxr-x
chown user:group /path              # Change ownership
setfacl -m u:user:rwx /path         # Add ACL
getfacl /path                       # View ACL
```

### Sudo Quick Reference
```bash
visudo                              # Edit sudoers safely
visudo -cf /etc/sudoers.d/file      # Check syntax
sudo -l                             # List user's sudo rights
sudo -u otheruser command           # Run as other user
```

---

## Common Mistakes & Fixes

| Mistake | Problem | Fix |
|---------|---------|-----|
| `usermod -G` without `-a` | Replaces all groups | Always use `usermod -aG` |
| Editing `/etc/sudoers` directly | Syntax error locks sudo | Always use `visudo` |
| Wrong SSH permissions | Key authentication fails | `chmod 700 ~/.ssh; chmod 600 ~/.ssh/*` |
| Forgetting `daemon-reload` | Service changes not applied | Not applicable here |
| Creating user without `-m` | No home directory | Use `-m` or `mkhomedir_helper` |

---

## Mini Challenges

### Challenge 1 (3 min)
Create a user `webdev` with home directory, bash shell, member of `www-data` group.

```bash
# Solution
sudo useradd -m -s /bin/bash -G www-data -c "Web Developer" webdev
sudo passwd webdev
```

### Challenge 2 (3 min)
Create sudo rule allowing `developers` group to restart nginx without password.

```bash
# Solution - /etc/sudoers.d/developers
%developers ALL=(root) NOPASSWD: /bin/systemctl restart nginx
```

### Challenge 3 (4 min)
Set up shared directory `/srv/dev` where all files created belong to `developers` group.

```bash
# Solution
sudo mkdir -p /srv/dev
sudo chgrp developers /srv/dev
sudo chmod 2775 /srv/dev
sudo setfacl -d -m g:developers:rwx /srv/dev
```

---

## Final Test: User Provisioning System Validation

Run this test to verify your provisioning system:

```bash
#!/bin/bash
# test_provisioning.sh

echo "=== Testing User Provisioning System ==="

# Test 1: Create test user
echo "Test 1: Creating test user..."
./provision_user.sh -i << EOF
testuser123
Test User
testuser@example.com
backend-dev

y
EOF

# Verify
echo "Test 2: Verifying user creation..."
id testuser123 || { echo "FAIL: User not created"; exit 1; }

echo "Test 3: Checking groups..."
groups testuser123 | grep -q "developers" || { echo "FAIL: Not in developers"; exit 1; }
groups testuser123 | grep -q "docker" || { echo "FAIL: Not in docker"; exit 1; }

echo "Test 4: Checking home directory..."
[[ -d /home/testuser123 ]] || { echo "FAIL: No home dir"; exit 1; }
[[ -d /home/testuser123/.ssh ]] || echo "NOTE: No SSH dir (expected if no key provided)"

echo "Test 5: Checking shell..."
getent passwd testuser123 | grep -q "/bin/bash" || { echo "FAIL: Wrong shell"; exit 1; }

# Cleanup
echo "Cleaning up..."
sudo userdel -r testuser123 2>/dev/null

echo "=== All Tests Passed ==="
```

---

## Time-Based Learning Plan

| Time | Activity | Checkpoint |
|------|----------|------------|
| 0:00-0:15 | Part 1-2: Fundamentals & User Creation | Create single user with script |
| 0:15-0:30 | Part 3: Group Management | Set up group hierarchy + shared dirs |
| 0:30-0:40 | Part 4: Sudo Configuration | Create role-based sudo rules |
| 0:40-0:55 | Part 5: Complete System | Run full provisioning script |
| 0:55-1:00 | Final Test & Review | Pass all verification tests |

---

## Key Takeaways

1. **Always use `-aG`** with usermod to append groups
2. **Never edit sudoers directly** - use `visudo`
3. **SSH permissions are critical**: 700 for `.ssh`, 600 for keys
4. **Use setgid (2xxx)** on shared directories
5. **Implement least privilege** - grant minimum required access
6. **Log everything** for audit trails
7. **Automate consistently** - manual changes lead to drift