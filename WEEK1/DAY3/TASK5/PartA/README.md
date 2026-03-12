# Production-Ready Automation Scripts Mastery Guide

## Overview

This comprehensive guide covers two critical production systems:
- **Task A**: Enterprise Backup & Recovery System
- **Task B**: Intelligent Log Analysis & Alerting System

---

# TASK A: System Backup & Recovery Script (2.5 hrs)

## Learning Objectives
- Implement incremental backups with deduplication
- Ensure database consistency during backups
- Configure multi-destination replication
- Automate disaster recovery testing
- Set up comprehensive notifications

---

## Part 1: Backup Fundamentals (30 mins)

### Backup Types Explained

```
Full Backup
└── Complete copy of all data
    ├── Pros: Simple recovery, self-contained
    └── Cons: Slow, storage-intensive

Incremental Backup
└── Only changes since LAST backup
    ├── Pros: Fast, minimal storage
    └── Cons: Recovery needs full chain

Differential Backup
└── Changes since LAST FULL backup
    ├── Pros: Faster recovery than incremental
    └── Cons: Grows larger over time

Synthetic Full
└── Combines full + incrementals into new full
    ├── Pros: Best of both worlds
    └── Cons: Requires processing
```

### The 3-2-1 Backup Rule

```
3 copies of data
├── Original
├── Local backup
└── Off-site backup

2 different media types
├── Disk/SSD
└── Cloud/Tape

1 copy off-site
└── Different geographic location
```

### Key Tools Comparison

| Tool | Use Case | Incremental | Dedup | Speed |
|------|----------|-------------|-------|-------|
| rsync | File sync | ✅ (manual) | ❌ | Fast |
| tar | Archives | ❌ | ❌ | Medium |
| borg | Dedup backup | ✅ | ✅ | Fast |
| restic | Cloud backup | ✅ | ✅ | Fast |
| rclone | Cloud sync | ✅ | ❌ | Fast |

---

## Part 2: Core Backup Implementation (45 mins)

### Complete Backup System Script

```bash
#!/bin/bash
#===============================================================================
# backup_system.sh - Enterprise Backup & Recovery System
# Version: 2.0
# Author: DevOps Team
# Description: Automated incremental backups with multi-destination support
#===============================================================================

set -euo pipefail
IFS=$'\n\t'

#===============================================================================
# CONFIGURATION
#===============================================================================

# Paths
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly CONFIG_FILE="${SCRIPT_DIR}/backup.conf"
readonly LOG_DIR="/var/log/backup"
readonly LOCK_FILE="/var/run/backup.lock"
readonly STATE_DIR="/var/lib/backup"

# Load configuration
load_config() {
    if [[ -f "$CONFIG_FILE" ]]; then
        # shellcheck source=/dev/null
        source "$CONFIG_FILE"
    else
        # Default configuration
        BACKUP_NAME="${BACKUP_NAME:-$(hostname -s)}"
        BACKUP_ROOT="${BACKUP_ROOT:-/backup}"
        RETENTION_DAYS="${RETENTION_DAYS:-30}"
        RETENTION_WEEKS="${RETENTION_WEEKS:-12}"
        RETENTION_MONTHS="${RETENTION_MONTHS:-12}"
        
        # Sources to backup
        BACKUP_SOURCES=(
            "/etc"
            "/home"
            "/var/www"
            "/opt"
        )
        
        # Exclusions
        BACKUP_EXCLUDES=(
            "*.tmp"
            "*.log"
            "*.cache"
            "*/.cache/*"
            "*/node_modules/*"
            "*/__pycache__/*"
            "*.sock"
        )
        
        # Remote destinations (space-separated: type:path)
        REMOTE_DESTINATIONS=(
            # "s3:s3.amazonaws.com/mybucket/backup"
            # "sftp:backup@remote.server.com:/backup"
            # "b2:mybucket/backup"
        )
        
        # Database settings
        DB_BACKUP_ENABLED="${DB_BACKUP_ENABLED:-true}"
        MYSQL_DATABASES="${MYSQL_DATABASES:-all}"
        POSTGRES_DATABASES="${POSTGRES_DATABASES:-all}"
        
        # Notifications
        NOTIFICATION_EMAIL="${NOTIFICATION_EMAIL:-}"
        SLACK_WEBHOOK_URL="${SLACK_WEBHOOK_URL:-}"
        
        # Encryption
        ENCRYPTION_ENABLED="${ENCRYPTION_ENABLED:-true}"
        ENCRYPTION_KEY_FILE="${ENCRYPTION_KEY_FILE:-/etc/backup/encryption.key}"
    fi
}

#===============================================================================
# LOGGING SYSTEM
#===============================================================================

readonly LOG_FILE="${LOG_DIR}/backup_$(date +%Y%m%d).log"
declare -A LOG_LEVELS=([DEBUG]=0 [INFO]=1 [WARN]=2 [ERROR]=3 [FATAL]=4)
LOG_LEVEL="${LOG_LEVEL:-INFO}"

init_logging() {
    mkdir -p "$LOG_DIR"
    exec 3>&1  # Save stdout
    exec 4>&2  # Save stderr
}

log() {
    local level="$1"
    shift
    local message="$*"
    local timestamp
    timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    
    # Check log level
    [[ ${LOG_LEVELS[$level]} -ge ${LOG_LEVELS[$LOG_LEVEL]} ]] || return 0
    
    # Color codes
    local color=""
    local reset="\033[0m"
    case "$level" in
        DEBUG) color="\033[36m" ;;  # Cyan
        INFO)  color="\033[32m" ;;  # Green
        WARN)  color="\033[33m" ;;  # Yellow
        ERROR) color="\033[31m" ;;  # Red
        FATAL) color="\033[35m" ;;  # Magenta
    esac
    
    # Log to file (no color)
    echo "[$timestamp] [$level] $message" >> "$LOG_FILE"
    
    # Log to console (with color)
    echo -e "${color}[$timestamp] [$level]${reset} $message" >&3
}

log_debug() { log DEBUG "$@"; }
log_info()  { log INFO "$@"; }
log_warn()  { log WARN "$@"; }
log_error() { log ERROR "$@"; }
log_fatal() { log FATAL "$@"; exit 1; }

#===============================================================================
# LOCKING MECHANISM
#===============================================================================

acquire_lock() {
    exec 200>"$LOCK_FILE"
    if ! flock -n 200; then
        log_fatal "Another backup process is running (lock file: $LOCK_FILE)"
    fi
    echo $$ >&200
    log_debug "Lock acquired (PID: $$)"
}

release_lock() {
    flock -u 200 2>/dev/null || true
    rm -f "$LOCK_FILE"
    log_debug "Lock released"
}

#===============================================================================
# BACKUP STATE MANAGEMENT
#===============================================================================

init_state() {
    mkdir -p "$STATE_DIR"
    
    # Initialize state file
    STATE_FILE="$STATE_DIR/backup_state.json"
    if [[ ! -f "$STATE_FILE" ]]; then
        cat > "$STATE_FILE" << 'EOF'
{
    "last_full_backup": null,
    "last_incremental_backup": null,
    "backup_count": 0,
    "total_size_bytes": 0,
    "history": []
}
EOF
    fi
}

update_state() {
    local backup_type="$1"
    local backup_path="$2"
    local backup_size="$3"
    local status="$4"
    
    local timestamp
    timestamp=$(date -Iseconds)
    
    # Update using jq if available, otherwise simple append
    if command -v jq &>/dev/null; then
        local tmp_file
        tmp_file=$(mktemp)
        
        jq --arg type "$backup_type" \
           --arg path "$backup_path" \
           --arg size "$backup_size" \
           --arg status "$status" \
           --arg timestamp "$timestamp" \
           '.history += [{
               "timestamp": $timestamp,
               "type": $type,
               "path": $path,
               "size": ($size | tonumber),
               "status": $status
           }] |
           .backup_count += 1 |
           .total_size_bytes += ($size | tonumber) |
           if $type == "full" then .last_full_backup = $timestamp
           else .last_incremental_backup = $timestamp end' \
           "$STATE_FILE" > "$tmp_file"
        
        mv "$tmp_file" "$STATE_FILE"
    fi
}

get_last_backup_time() {
    local backup_type="${1:-any}"
    
    if command -v jq &>/dev/null && [[ -f "$STATE_FILE" ]]; then
        case "$backup_type" in
            full)
                jq -r '.last_full_backup // empty' "$STATE_FILE"
                ;;
            incremental)
                jq -r '.last_incremental_backup // empty' "$STATE_FILE"
                ;;
            *)
                jq -r '[.last_full_backup, .last_incremental_backup] | map(select(. != null)) | max // empty' "$STATE_FILE"
                ;;
        esac
    fi
}

#===============================================================================
# FILESYSTEM BACKUP
#===============================================================================

create_exclusion_file() {
    local exclude_file
    exclude_file=$(mktemp)
    
    for pattern in "${BACKUP_EXCLUDES[@]}"; do
        echo "$pattern" >> "$exclude_file"
    done
    
    echo "$exclude_file"
}

backup_filesystem_rsync() {
    local backup_type="$1"
    local destination="$2"
    
    log_info "Starting filesystem backup (type: $backup_type)"
    
    local exclude_file
    exclude_file=$(create_exclusion_file)
    
    local rsync_opts=(
        --archive
        --verbose
        --human-readable
        --progress
        --stats
        --delete
        --delete-excluded
        --exclude-from="$exclude_file"
        --compress
        --partial
        --timeout=3600
    )
    
    # Add incremental options
    if [[ "$backup_type" == "incremental" ]]; then
        local link_dest
        link_dest=$(find "$BACKUP_ROOT/filesystem" -maxdepth 1 -type d -name "20*" | sort -r | head -1)
        
        if [[ -n "$link_dest" ]]; then
            rsync_opts+=(--link-dest="$link_dest")
            log_info "Using link-dest: $link_dest"
        fi
    fi
    
    # Execute backup for each source
    local total_size=0
    local failed_sources=()
    
    for source in "${BACKUP_SOURCES[@]}"; do
        if [[ ! -e "$source" ]]; then
            log_warn "Source does not exist: $source"
            continue
        fi
        
        local source_name
        source_name=$(basename "$source")
        local target_path="$destination/$source_name"
        
        log_info "Backing up: $source -> $target_path"
        
        mkdir -p "$target_path"
        
        if rsync "${rsync_opts[@]}" "$source/" "$target_path/" 2>&1 | tee -a "$LOG_FILE"; then
            local source_size
            source_size=$(du -sb "$target_path" 2>/dev/null | cut -f1)
            total_size=$((total_size + source_size))
            log_info "Completed: $source ($source_size bytes)"
        else
            log_error "Failed: $source"
            failed_sources+=("$source")
        fi
    done
    
    rm -f "$exclude_file"
    
    if [[ ${#failed_sources[@]} -gt 0 ]]; then
        log_error "Failed sources: ${failed_sources[*]}"
        return 1
    fi
    
    echo "$total_size"
}

backup_filesystem_borg() {
    local backup_type="$1"
    local repo="$2"
    
    log_info "Starting Borg backup"
    
    # Initialize repo if needed
    if [[ ! -d "$repo" ]]; then
        log_info "Initializing Borg repository: $repo"
        
        if [[ "$ENCRYPTION_ENABLED" == "true" ]]; then
            BORG_PASSPHRASE=$(cat "$ENCRYPTION_KEY_FILE") \
            borg init --encryption=repokey "$repo"
        else
            borg init --encryption=none "$repo"
        fi
    fi
    
    local archive_name="${BACKUP_NAME}-$(date +%Y%m%d-%H%M%S)"
    local exclude_args=()
    
    for pattern in "${BACKUP_EXCLUDES[@]}"; do
        exclude_args+=(--exclude "$pattern")
    done
    
    local borg_opts=(
        --verbose
        --stats
        --show-rc
        --compression lz4
        --checkpoint-interval 300
    )
    
    # Execute backup
    local output
    if [[ "$ENCRYPTION_ENABLED" == "true" ]]; then
        output=$(BORG_PASSPHRASE=$(cat "$ENCRYPTION_KEY_FILE") \
            borg create "${borg_opts[@]}" "${exclude_args[@]}" \
            "${repo}::${archive_name}" \
            "${BACKUP_SOURCES[@]}" 2>&1)
    else
        output=$(borg create "${borg_opts[@]}" "${exclude_args[@]}" \
            "${repo}::${archive_name}" \
            "${BACKUP_SOURCES[@]}" 2>&1)
    fi
    
    log_info "Borg output: $output"
    
    # Get backup size
    local backup_info
    if [[ "$ENCRYPTION_ENABLED" == "true" ]]; then
        backup_info=$(BORG_PASSPHRASE=$(cat "$ENCRYPTION_KEY_FILE") \
            borg info "${repo}::${archive_name}" 2>&1)
    else
        backup_info=$(borg info "${repo}::${archive_name}" 2>&1)
    fi
    
    echo "$backup_info" | grep -oP 'This archive:\s+\K[\d.]+\s+\w+' || echo "0"
}

#===============================================================================
# DATABASE BACKUP
#===============================================================================

backup_mysql() {
    local backup_dir="$1"
    local timestamp
    timestamp=$(date +%Y%m%d_%H%M%S)
    
    log_info "Starting MySQL backup"
    
    mkdir -p "$backup_dir/mysql"
    
    # Get MySQL credentials
    local mysql_opts=()
    if [[ -f /etc/mysql/debian.cnf ]]; then
        mysql_opts+=(--defaults-file=/etc/mysql/debian.cnf)
    elif [[ -f ~/.my.cnf ]]; then
        mysql_opts+=(--defaults-file=~/.my.cnf)
    fi
    
    # Get database list
    local databases
    if [[ "$MYSQL_DATABASES" == "all" ]]; then
        databases=$(mysql "${mysql_opts[@]}" -N -e "SHOW DATABASES" | \
            grep -Ev '^(information_schema|performance_schema|sys)$')
    else
        databases="$MYSQL_DATABASES"
    fi
    
    local failed_dbs=()
    
    for db in $databases; do
        log_info "Backing up MySQL database: $db"
        
        local dump_file="$backup_dir/mysql/${db}_${timestamp}.sql"
        local compressed_file="${dump_file}.gz"
        
        # Dump with consistency
        if mysqldump "${mysql_opts[@]}" \
            --single-transaction \
            --routines \
            --triggers \
            --events \
            --quick \
            --lock-tables=false \
            "$db" 2>>"$LOG_FILE" | gzip > "$compressed_file"; then
            
            # Verify dump
            if gzip -t "$compressed_file" 2>/dev/null; then
                local size
                size=$(stat -c%s "$compressed_file")
                log_info "MySQL backup completed: $db ($size bytes)"
                
                # Create checksum
                sha256sum "$compressed_file" > "${compressed_file}.sha256"
            else
                log_error "MySQL backup corrupted: $db"
                failed_dbs+=("$db")
                rm -f "$compressed_file"
            fi
        else
            log_error "MySQL backup failed: $db"
            failed_dbs+=("$db")
        fi
    done
    
    if [[ ${#failed_dbs[@]} -gt 0 ]]; then
        return 1
    fi
    
    return 0
}

backup_postgresql() {
    local backup_dir="$1"
    local timestamp
    timestamp=$(date +%Y%m%d_%H%M%S)
    
    log_info "Starting PostgreSQL backup"
    
    mkdir -p "$backup_dir/postgresql"
    
    # Get database list
    local databases
    if [[ "$POSTGRES_DATABASES" == "all" ]]; then
        databases=$(sudo -u postgres psql -t -c \
            "SELECT datname FROM pg_database WHERE datistemplate = false AND datname != 'postgres'")
    else
        databases="$POSTGRES_DATABASES"
    fi
    
    local failed_dbs=()
    
    for db in $databases; do
        db=$(echo "$db" | xargs)  # Trim whitespace
        [[ -z "$db" ]] && continue
        
        log_info "Backing up PostgreSQL database: $db"
        
        local dump_file="$backup_dir/postgresql/${db}_${timestamp}.sql"
        local compressed_file="${dump_file}.gz"
        
        # Custom format for better restore options
        local custom_dump="$backup_dir/postgresql/${db}_${timestamp}.dump"
        
        # Dump with consistency
        if sudo -u postgres pg_dump \
            --format=custom \
            --verbose \
            --file="$custom_dump" \
            "$db" 2>>"$LOG_FILE"; then
            
            # Also create SQL dump for readability
            sudo -u postgres pg_dump \
                --format=plain \
                "$db" 2>>"$LOG_FILE" | gzip > "$compressed_file"
            
            local size
            size=$(stat -c%s "$custom_dump")
            log_info "PostgreSQL backup completed: $db ($size bytes)"
            
            # Create checksum
            sha256sum "$custom_dump" > "${custom_dump}.sha256"
        else
            log_error "PostgreSQL backup failed: $db"
            failed_dbs+=("$db")
        fi
    done
    
    # Global objects backup (roles, tablespaces)
    log_info "Backing up PostgreSQL global objects"
    sudo -u postgres pg_dumpall --globals-only 2>>"$LOG_FILE" | \
        gzip > "$backup_dir/postgresql/globals_${timestamp}.sql.gz"
    
    if [[ ${#failed_dbs[@]} -gt 0 ]]; then
        return 1
    fi
    
    return 0
}

backup_mongodb() {
    local backup_dir="$1"
    local timestamp
    timestamp=$(date +%Y%m%d_%H%M%S)
    
    log_info "Starting MongoDB backup"
    
    mkdir -p "$backup_dir/mongodb"
    
    local mongo_dump_dir="$backup_dir/mongodb/dump_${timestamp}"
    
    # Dump all databases
    if mongodump --out="$mongo_dump_dir" 2>>"$LOG_FILE"; then
        # Compress
        tar -czf "${mongo_dump_dir}.tar.gz" -C "$backup_dir/mongodb" "dump_${timestamp}"
        rm -rf "$mongo_dump_dir"
        
        local size
        size=$(stat -c%s "${mongo_dump_dir}.tar.gz")
        log_info "MongoDB backup completed ($size bytes)"
        
        sha256sum "${mongo_dump_dir}.tar.gz" > "${mongo_dump_dir}.tar.gz.sha256"
    else
        log_error "MongoDB backup failed"
        return 1
    fi
    
    return 0
}

#===============================================================================
# BACKUP VERIFICATION
#===============================================================================

verify_backup() {
    local backup_path="$1"
    
    log_info "Verifying backup: $backup_path"
    
    local verification_passed=true
    local issues=()
    
    # Check if backup exists
    if [[ ! -d "$backup_path" ]]; then
        log_error "Backup path does not exist: $backup_path"
        return 1
    fi
    
    # Check backup size
    local total_size
    total_size=$(du -sb "$backup_path" | cut -f1)
    
    if [[ $total_size -lt 1024 ]]; then
        issues+=("Backup suspiciously small: $total_size bytes")
        verification_passed=false
    fi
    
    # Verify checksums
    find "$backup_path" -name "*.sha256" | while read -r checksum_file; do
        local data_file="${checksum_file%.sha256}"
        
        if [[ -f "$data_file" ]]; then
            if ! sha256sum -c "$checksum_file" &>/dev/null; then
                issues+=("Checksum mismatch: $data_file")
                verification_passed=false
            fi
        fi
    done
    
    # Verify compressed files
    find "$backup_path" -name "*.gz" | while read -r gz_file; do
        if ! gzip -t "$gz_file" 2>/dev/null; then
            issues+=("Corrupted archive: $gz_file")
            verification_passed=false
        fi
    done
    
    # Test database dump restoration (dry run)
    find "$backup_path" -name "*.dump" -type f | head -1 | while read -r dump_file; do
        if [[ -f "$dump_file" ]]; then
            if ! pg_restore --list "$dump_file" &>/dev/null; then
                issues+=("Invalid PostgreSQL dump: $dump_file")
                verification_passed=false
            fi
        fi
    done
    
    # Report results
    if [[ "$verification_passed" == "true" ]]; then
        log_info "Backup verification PASSED"
        return 0
    else
        log_error "Backup verification FAILED"
        for issue in "${issues[@]}"; do
            log_error "  - $issue"
        done
        return 1
    fi
}

#===============================================================================
# REMOTE SYNC
#===============================================================================

sync_to_remote() {
    local source="$1"
    local destination="$2"
    
    log_info "Syncing to remote: $destination"
    
    local dest_type="${destination%%:*}"
    local dest_path="${destination#*:}"
    
    case "$dest_type" in
        s3)
            sync_to_s3 "$source" "$dest_path"
            ;;
        sftp)
            sync_to_sftp "$source" "$dest_path"
            ;;
        b2)
            sync_to_b2 "$source" "$dest_path"
            ;;
        rsync)
            sync_to_rsync "$source" "$dest_path"
            ;;
        *)
            log_error "Unknown remote type: $dest_type"
            return 1
            ;;
    esac
}

sync_to_s3() {
    local source="$1"
    local bucket="$2"
    
    if ! command -v aws &>/dev/null; then
        log_error "AWS CLI not installed"
        return 1
    fi
    
    aws s3 sync "$source" "s3://$bucket" \
        --storage-class STANDARD_IA \
        --delete \
        --only-show-errors \
        2>&1 | tee -a "$LOG_FILE"
}

sync_to_sftp() {
    local source="$1"
    local destination="$2"
    
    local user_host="${destination%%:*}"
    local remote_path="${destination#*:}"
    
    rsync -avz --progress --delete \
        -e "ssh -o StrictHostKeyChecking=no -o BatchMode=yes" \
        "$source/" "${user_host}:${remote_path}/" \
        2>&1 | tee -a "$LOG_FILE"
}

sync_to_b2() {
    local source="$1"
    local bucket="$2"
    
    if ! command -v b2 &>/dev/null; then
        log_error "Backblaze B2 CLI not installed"
        return 1
    fi
    
    b2 sync --delete "$source" "b2://$bucket" \
        2>&1 | tee -a "$LOG_FILE"
}

sync_to_rsync() {
    local source="$1"
    local destination="$2"
    
    rsync -avz --progress --delete \
        "$source/" "$destination/" \
        2>&1 | tee -a "$LOG_FILE"
}

sync_all_remotes() {
    local source="$1"
    
    local failed_destinations=()
    
    for destination in "${REMOTE_DESTINATIONS[@]}"; do
        if ! sync_to_remote "$source" "$destination"; then
            failed_destinations+=("$destination")
        fi
    done
    
    if [[ ${#failed_destinations[@]} -gt 0 ]]; then
        log_error "Failed remote destinations: ${failed_destinations[*]}"
        return 1
    fi
    
    return 0
}

#===============================================================================
# RETENTION MANAGEMENT
#===============================================================================

apply_retention_policy() {
    local backup_root="$1"
    
    log_info "Applying retention policy"
    
    local now
    now=$(date +%s)
    
    # Daily retention
    log_info "Cleaning daily backups older than $RETENTION_DAYS days"
    find "$backup_root" -maxdepth 2 -type d -name "20*" -mtime "+$RETENTION_DAYS" | \
    while read -r old_backup; do
        # Keep weekly backups (Sunday)
        local backup_date
        backup_date=$(basename "$old_backup" | cut -c1-8)
        local day_of_week
        day_of_week=$(date -d "$backup_date" +%u 2>/dev/null || echo "")
        
        if [[ "$day_of_week" != "7" ]]; then
            log_info "Removing old daily backup: $old_backup"
            rm -rf "$old_backup"
        fi
    done
    
    # Weekly retention
    local weekly_cutoff=$((RETENTION_WEEKS * 7))
    log_info "Cleaning weekly backups older than $weekly_cutoff days"
    find "$backup_root" -maxdepth 2 -type d -name "20*" -mtime "+$weekly_cutoff" | \
    while read -r old_backup; do
        local backup_date
        backup_date=$(basename "$old_backup" | cut -c1-8)
        local day_of_month
        day_of_month=$(date -d "$backup_date" +%d 2>/dev/null || echo "")
        
        # Keep first of month
        if [[ "$day_of_month" != "01" ]]; then
            log_info "Removing old weekly backup: $old_backup"
            rm -rf "$old_backup"
        fi
    done
    
    # Monthly retention
    local monthly_cutoff=$((RETENTION_MONTHS * 30))
    log_info "Cleaning monthly backups older than $monthly_cutoff days"
    find "$backup_root" -maxdepth 2 -type d -name "20*" -mtime "+$monthly_cutoff" -exec rm -rf {} \;
    
    # Clean Borg repository
    if [[ -d "$backup_root/borg" ]]; then
        log_info "Pruning Borg repository"
        
        local borg_prune_opts=(
            --keep-daily="$RETENTION_DAYS"
            --keep-weekly="$RETENTION_WEEKS"
            --keep-monthly="$RETENTION_MONTHS"
            --stats
        )
        
        if [[ "$ENCRYPTION_ENABLED" == "true" ]]; then
            BORG_PASSPHRASE=$(cat "$ENCRYPTION_KEY_FILE") \
            borg prune "${borg_prune_opts[@]}" "$backup_root/borg" 2>&1 | tee -a "$LOG_FILE"
        else
            borg prune "${borg_prune_opts[@]}" "$backup_root/borg" 2>&1 | tee -a "$LOG_FILE"
        fi
    fi
    
    log_info "Retention policy applied"
}

#===============================================================================
# DISASTER RECOVERY TESTING
#===============================================================================

test_disaster_recovery() {
    local backup_path="$1"
    local test_dir="/tmp/dr_test_$(date +%s)"
    
    log_info "Starting disaster recovery test"
    
    mkdir -p "$test_dir"
    
    local test_results=()
    local all_passed=true
    
    # Test 1: Filesystem restore
    log_info "Test 1: Filesystem restore"
    local fs_backup
    fs_backup=$(find "$backup_path/filesystem" -maxdepth 1 -type d -name "20*" | sort -r | head -1)
    
    if [[ -n "$fs_backup" ]]; then
        local test_file
        test_file=$(find "$fs_backup" -type f | head -1)
        
        if [[ -n "$test_file" ]]; then
            local relative_path="${test_file#$fs_backup/}"
            local restore_path="$test_dir/fs_restore"
            
            mkdir -p "$restore_path"
            if cp "$test_file" "$restore_path/"; then
                test_results+=("Filesystem restore: PASSED")
            else
                test_results+=("Filesystem restore: FAILED")
                all_passed=false
            fi
        fi
    else
        test_results+=("Filesystem restore: SKIPPED (no backup found)")
    fi
    
    # Test 2: Database restore (PostgreSQL)
    log_info "Test 2: PostgreSQL restore"
    local pg_dump
    pg_dump=$(find "$backup_path/postgresql" -name "*.dump" -type f | head -1)
    
    if [[ -n "$pg_dump" ]]; then
        local test_db="dr_test_$(date +%s)"
        
        # Create test database
        if sudo -u postgres createdb "$test_db" 2>/dev/null; then
            # Restore
            if sudo -u postgres pg_restore -d "$test_db" "$pg_dump" 2>/dev/null; then
                # Verify tables exist
                local table_count
                table_count=$(sudo -u postgres psql -t -d "$test_db" -c \
                    "SELECT COUNT(*) FROM information_schema.tables WHERE table_schema = 'public'" | xargs)
                
                if [[ $table_count -gt 0 ]]; then
                    test_results+=("PostgreSQL restore: PASSED ($table_count tables)")
                else
                    test_results+=("PostgreSQL restore: FAILED (no tables)")
                    all_passed=false
                fi
            else
                test_results+=("PostgreSQL restore: FAILED (restore error)")
                all_passed=false
            fi
            
            # Cleanup
            sudo -u postgres dropdb "$test_db" 2>/dev/null || true
        else
            test_results+=("PostgreSQL restore: SKIPPED (cannot create test db)")
        fi
    else
        test_results+=("PostgreSQL restore: SKIPPED (no dump found)")
    fi
    
    # Test 3: MySQL restore
    log_info "Test 3: MySQL restore"
    local mysql_dump
    mysql_dump=$(find "$backup_path/mysql" -name "*.sql.gz" -type f | head -1)
    
    if [[ -n "$mysql_dump" ]]; then
        local test_db="dr_test_$(date +%s)"
        local mysql_opts=()
        
        if [[ -f /etc/mysql/debian.cnf ]]; then
            mysql_opts+=(--defaults-file=/etc/mysql/debian.cnf)
        fi
        
        # Create test database
        if mysql "${mysql_opts[@]}" -e "CREATE DATABASE $test_db" 2>/dev/null; then
            # Restore
            if gunzip -c "$mysql_dump" | mysql "${mysql_opts[@]}" "$test_db" 2>/dev/null; then
                local table_count
                table_count=$(mysql "${mysql_opts[@]}" -N -e "SELECT COUNT(*) FROM information_schema.tables WHERE table_schema = '$test_db'" | xargs)
                
                if [[ $table_count -gt 0 ]]; then
                    test_results+=("MySQL restore: PASSED ($table_count tables)")
                else
                    test_results+=("MySQL restore: WARNING (no tables, might be empty db)")
                fi
            else
                test_results+=("MySQL restore: FAILED (restore error)")
                all_passed=false
            fi
            
            # Cleanup
            mysql "${mysql_opts[@]}" -e "DROP DATABASE $test_db" 2>/dev/null || true
        else
            test_results+=("MySQL restore: SKIPPED (cannot create test db)")
        fi
    else
        test_results+=("MySQL restore: SKIPPED (no dump found)")
    fi
    
    # Test 4: Checksum verification
    log_info "Test 4: Checksum verification"
    local checksum_failures=0
    
    find "$backup_path" -name "*.sha256" | while read -r checksum_file; do
        local data_file="${checksum_file%.sha256}"
        if [[ -f "$data_file" ]]; then
            if ! sha256sum -c "$checksum_file" &>/dev/null; then
                ((checksum_failures++))
            fi
        fi
    done
    
    if [[ $checksum_failures -eq 0 ]]; then
        test_results+=("Checksum verification: PASSED")
    else
        test_results+=("Checksum verification: FAILED ($checksum_failures failures)")
        all_passed=false
    fi
    
    # Cleanup
    rm -rf "$test_dir"
    
    # Report results
    log_info "========== DISASTER RECOVERY TEST RESULTS =========="
    for result in "${test_results[@]}"; do
        log_info "  $result"
    done
    log_info "===================================================="
    
    if [[ "$all_passed" == "true" ]]; then
        log_info "DR TEST: ALL PASSED"
        return 0
    else
        log_error "DR TEST: SOME TESTS FAILED"
        return 1
    fi
}

#===============================================================================
# NOTIFICATIONS
#===============================================================================

send_email_notification() {
    local subject="$1"
    local body="$2"
    
    [[ -n "$NOTIFICATION_EMAIL" ]] || return 0
    
    if command -v mail &>/dev/null; then
        echo -e "$body" | mail -s "$subject" "$NOTIFICATION_EMAIL"
        log_info "Email notification sent to $NOTIFICATION_EMAIL"
    elif command -v sendmail &>/dev/null; then
        {
            echo "To: $NOTIFICATION_EMAIL"
            echo "Subject: $subject"
            echo "Content-Type: text/plain; charset=UTF-8"
            echo ""
            echo -e "$body"
        } | sendmail -t
        log_info "Email notification sent to $NOTIFICATION_EMAIL"
    else
        log_warn "No mail command available"
    fi
}

send_slack_notification() {
    local message="$1"
    local status="${2:-info}"
    
    [[ -n "$SLACK_WEBHOOK_URL" ]] || return 0
    
    local color
    case "$status" in
        success) color="good" ;;
        warning) color="warning" ;;
        error)   color="danger" ;;
        *)       color="#439FE0" ;;
    esac
    
    local payload
    payload=$(cat << EOF
{
    "attachments": [
        {
            "color": "$color",
            "title": "Backup System Notification",
            "text": "$message",
            "footer": "$(hostname -f)",
            "ts": $(date +%s)
        }
    ]
}
EOF
)
    
    curl -s -X POST -H 'Content-type: application/json' \
        --data "$payload" \
        "$SLACK_WEBHOOK_URL" >/dev/null
    
    log_info "Slack notification sent"
}

generate_backup_report() {
    local backup_path="$1"
    local backup_type="$2"
    local duration="$3"
    local status="$4"
    
    local total_size
    total_size=$(du -sh "$backup_path" 2>/dev/null | cut -f1)
    
    local report
    report=$(cat << EOF
================================================================================
                         BACKUP REPORT
================================================================================

Server:         $(hostname -f)
Date:           $(date '+%Y-%m-%d %H:%M:%S %Z')
Backup Type:    $backup_type
Status:         $status
Duration:       ${duration}s
Total Size:     $total_size
Backup Path:    $backup_path

DETAILS
-------
EOF
)
    
    # Add filesystem details
    if [[ -d "$backup_path/filesystem" ]]; then
        report+=$'\n'"Filesystem Backup:"
        for source in "${BACKUP_SOURCES[@]}"; do
            local source_name
            source_name=$(basename "$source")
            local size
            size=$(du -sh "$backup_path/filesystem/$source_name" 2>/dev/null | cut -f1 || echo "N/A")
            report+=$'\n'"  - $source: $size"
        done
    fi
    
    # Add database details
    if [[ -d "$backup_path/mysql" ]]; then
        report+=$'\n\n'"MySQL Backups:"
        for dump in "$backup_path/mysql"/*.sql.gz; do
            [[ -f "$dump" ]] || continue
            local size
            size=$(du -sh "$dump" 2>/dev/null | cut -f1)
            report+=$'\n'"  - $(basename "$dump"): $size"
        done
    fi
    
    if [[ -d "$backup_path/postgresql" ]]; then
        report+=$'\n\n'"PostgreSQL Backups:"
        for dump in "$backup_path/postgresql"/*.dump; do
            [[ -f "$dump" ]] || continue
            local size
            size=$(du -sh "$dump" 2>/dev/null | cut -f1)
            report+=$'\n'"  - $(basename "$dump"): $size"
        done
    fi
    
    report+=$'\n\n'"================================================================================"
    
    echo "$report"
}

#===============================================================================
# MAIN BACKUP ORCHESTRATION
#===============================================================================

run_backup() {
    local backup_type="${1:-incremental}"
    local start_time
    start_time=$(date +%s)
    
    log_info "========== STARTING BACKUP =========="
    log_info "Type: $backup_type"
    log_info "Host: $(hostname -f)"
    
    # Create backup directory structure
    local timestamp
    timestamp=$(date +%Y%m%d_%H%M%S)
    local backup_path="$BACKUP_ROOT/$timestamp"
    
    mkdir -p "$backup_path"/{filesystem,mysql,postgresql}
    
    local backup_status="SUCCESS"
    local failed_components=()
    
    # 1. Filesystem backup
    log_info "Phase 1: Filesystem backup"
    if command -v borg &>/dev/null && [[ "$backup_type" != "simple" ]]; then
        if ! backup_filesystem_borg "$backup_type" "$BACKUP_ROOT/borg"; then
            failed_components+=("filesystem")
            backup_status="PARTIAL"
        fi
    else
        if ! backup_filesystem_rsync "$backup_type" "$backup_path/filesystem"; then
            failed_components+=("filesystem")
            backup_status="PARTIAL"
        fi
    fi
    
    # 2. Database backups
    if [[ "$DB_BACKUP_ENABLED" == "true" ]]; then
        log_info "Phase 2: Database backup"
        
        # MySQL
        if command -v mysql &>/dev/null; then
            if ! backup_mysql "$backup_path"; then
                failed_components+=("mysql")
                backup_status="PARTIAL"
            fi
        fi
        
        # PostgreSQL
        if command -v psql &>/dev/null; then
            if ! backup_postgresql "$backup_path"; then
                failed_components+=("postgresql")
                backup_status="PARTIAL"
            fi
        fi
        
        # MongoDB
        if command -v mongodump &>/dev/null; then
            if ! backup_mongodb "$backup_path"; then
                failed_components+=("mongodb")
                backup_status="PARTIAL"
            fi
        fi
    fi
    
    # 3. Verification
    log_info "Phase 3: Verification"
    if ! verify_backup "$backup_path"; then
        backup_status="VERIFICATION_FAILED"
    fi
    
    # 4. Remote sync
    if [[ ${#REMOTE_DESTINATIONS[@]} -gt 0 ]]; then
        log_info "Phase 4: Remote sync"
        if ! sync_all_remotes "$backup_path"; then
            failed_components+=("remote_sync")
            backup_status="PARTIAL"
        fi
    fi
    
    # 5. Retention
    log_info "Phase 5: Retention policy"
    apply_retention_policy "$BACKUP_ROOT"
    
    # Calculate duration
    local end_time
    end_time=$(date +%s)
    local duration=$((end_time - start_time))
    
    # Update state
    local backup_size
    backup_size=$(du -sb "$backup_path" 2>/dev/null | cut -f1)
    update_state "$backup_type" "$backup_path" "$backup_size" "$backup_status"
    
    # Generate report
    local report
    report=$(generate_backup_report "$backup_path" "$backup_type" "$duration" "$backup_status")
    
    # Send notifications
    if [[ "$backup_status" == "SUCCESS" ]]; then
        send_slack_notification "Backup completed successfully\nDuration: ${duration}s\nSize: $(du -sh "$backup_path" | cut -f1)" "success"
        send_email_notification "[SUCCESS] Backup Complete - $(hostname -s)" "$report"
    else
        send_slack_notification "Backup completed with issues: ${failed_components[*]}\nStatus: $backup_status" "warning"
        send_email_notification "[WARNING] Backup Issues - $(hostname -s)" "$report"
    fi
    
    log_info "========== BACKUP COMPLETE =========="
    log_info "Status: $backup_status"
    log_info "Duration: ${duration}s"
    
    if [[ ${#failed_components[@]} -gt 0 ]]; then
        log_warn "Failed components: ${failed_components[*]}"
        return 1
    fi
    
    return 0
}

#===============================================================================
# RESTORE FUNCTIONS
#===============================================================================

restore_filesystem() {
    local backup_path="$1"
    local target_path="${2:-/}"
    local specific_path="${3:-}"
    
    log_info "Starting filesystem restore"
    log_info "Source: $backup_path"
    log_info "Target: $target_path"
    
    # Safety check
    if [[ "$target_path" == "/" ]]; then
        read -rp "WARNING: Restoring to root. Continue? (yes/no): " confirm
        [[ "$confirm" == "yes" ]] || { log_info "Restore cancelled"; return 1; }
    fi
    
    local rsync_opts=(
        --archive
        --verbose
        --progress
        --human-readable
    )
    
    if [[ -n "$specific_path" ]]; then
        rsync "${rsync_opts[@]}" "$backup_path/$specific_path" "$target_path/"
    else
        rsync "${rsync_opts[@]}" "$backup_path/" "$target_path/"
    fi
    
    log_info "Filesystem restore completed"
}

restore_database() {
    local db_type="$1"
    local dump_file="$2"
    local target_db="${3:-}"
    
    log_info "Starting $db_type restore"
    log_info "Dump: $dump_file"
    
    case "$db_type" in
        mysql)
            local mysql_opts=()
            [[ -f /etc/mysql/debian.cnf ]] && mysql_opts+=(--defaults-file=/etc/mysql/debian.cnf)
            
            if [[ -n "$target_db" ]]; then
                mysql "${mysql_opts[@]}" -e "CREATE DATABASE IF NOT EXISTS $target_db"
                gunzip -c "$dump_file" | mysql "${mysql_opts[@]}" "$target_db"
            else
                gunzip -c "$dump_file" | mysql "${mysql_opts[@]}"
            fi
            ;;
        postgresql)
            if [[ -n "$target_db" ]]; then
                sudo -u postgres createdb "$target_db" 2>/dev/null || true
                sudo -u postgres pg_restore -d "$target_db" "$dump_file"
            else
                sudo -u postgres pg_restore "$dump_file"
            fi
            ;;
        *)
            log_error "Unknown database type: $db_type"
            return 1
            ;;
    esac
    
    log_info "Database restore completed"
}

#===============================================================================
# CLI INTERFACE
#===============================================================================

show_usage() {
    cat << EOF
Backup System - Enterprise Backup & Recovery

Usage: $(basename "$0") <command> [options]

Commands:
    backup [type]       Run backup (full|incremental|simple)
    restore             Restore from backup
    verify <path>       Verify backup integrity
    list                List available backups
    status              Show backup status
    test-dr <path>      Run disaster recovery test
    prune               Apply retention policy
    sync <path> <dest>  Sync to remote destination

Options:
    -c, --config FILE   Use alternate config file
    -v, --verbose       Enable verbose output
    -h, --help          Show this help

Examples:
    $(basename "$0") backup full
    $(basename "$0") backup incremental
    $(basename "$0") restore /backup/20250115_020000
    $(basename "$0") test-dr /backup/20250115_020000
    $(basename "$0") verify /backup/20250115_020000

EOF
}

list_backups() {
    echo "Available Backups:"
    echo "=================="
    
    find "$BACKUP_ROOT" -maxdepth 1 -type d -name "20*" | sort -r | while read -r backup; do
        local name
        name=$(basename "$backup")
        local size
        size=$(du -sh "$backup" 2>/dev/null | cut -f1)
        local date_str
        date_str=$(echo "$name" | sed 's/_/ /' | sed 's/\([0-9]\{4\}\)\([0-9]\{2\}\)\([0-9]\{2\}\)/\1-\2-\3/')
        
        printf "  %-20s  %8s  %s\n" "$name" "$size" "$backup"
    done
}

show_status() {
    echo "Backup System Status"
    echo "===================="
    echo
    
    echo "Configuration:"
    echo "  Backup Root:     $BACKUP_ROOT"
    echo "  Retention Days:  $RETENTION_DAYS"
    echo "  Retention Weeks: $RETENTION_WEEKS"
    echo "  Encryption:      $ENCRYPTION_ENABLED"
    echo
    
    echo "Last Backups:"
    local last_full
    last_full=$(get_last_backup_time full)
    local last_incr
    last_incr=$(get_last_backup_time incremental)
    
    echo "  Last Full:        ${last_full:-Never}"
    echo "  Last Incremental: ${last_incr:-Never}"
    echo
    
    echo "Storage Usage:"
    du -sh "$BACKUP_ROOT"/* 2>/dev/null || echo "  No backups found"
    echo
    
    echo "Disk Space:"
    df -h "$BACKUP_ROOT" | tail -1 | awk '{print "  Total: " $2 "  Used: " $3 "  Available: " $4 "  Usage: " $5}'
}

#===============================================================================
# MAIN
#===============================================================================

main() {
    # Initialize
    init_logging
    load_config
    init_state
    
    # Parse arguments
    local command="${1:-}"
    shift || true
    
    case "$command" in
        backup)
            acquire_lock
            trap release_lock EXIT
            run_backup "${1:-incremental}"
            ;;
        restore)
            restore_filesystem "$@"
            ;;
        verify)
            verify_backup "${1:?Backup path required}"
            ;;
        list)
            list_backups
            ;;
        status)
            show_status
            ;;
        test-dr)
            test_disaster_recovery "${1:?Backup path required}"
            ;;
        prune)
            apply_retention_policy "$BACKUP_ROOT"
            ;;
        sync)
            sync_to_remote "${1:?Source required}" "${2:?Destination required}"
            ;;
        -h|--help|help|"")
            show_usage
            ;;
        *)
            echo "Unknown command: $command"
            show_usage
            exit 1
            ;;
    esac
}

main "$@"
```

---

## Part 3: Configuration File

```bash
# /opt/backup/backup.conf

# Backup identification
BACKUP_NAME="production-server"
BACKUP_ROOT="/backup"

# Retention policy
RETENTION_DAYS=7
RETENTION_WEEKS=4
RETENTION_MONTHS=12

# Sources
BACKUP_SOURCES=(
    "/etc"
    "/home"
    "/var/www"
    "/opt/apps"
    "/var/lib/docker/volumes"
)

# Exclusions
BACKUP_EXCLUDES=(
    "*.tmp"
    "*.log"
    "*.cache"
    "*/.cache/*"
    "*/node_modules/*"
    "*/__pycache__/*"
    "*.sock"
    "*/tmp/*"
)

# Remote destinations
REMOTE_DESTINATIONS=(
    "s3:mybucket/backups"
    "sftp:backup@offsite.server.com:/backups"
)

# Database settings
DB_BACKUP_ENABLED=true
MYSQL_DATABASES="all"
POSTGRES_DATABASES="all"

# Encryption
ENCRYPTION_ENABLED=true
ENCRYPTION_KEY_FILE="/etc/backup/encryption.key"

# Notifications
NOTIFICATION_EMAIL="admin@company.com"
SLACK_WEBHOOK_URL="https://hooks.slack.com/services/xxx/yyy/zzz"
```

---

## Part 4: Systemd Integration (15 mins)

### Service and Timer Files

```ini
# /etc/systemd/system/backup.service
[Unit]
Description=System Backup Service
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/opt/backup/backup_system.sh backup incremental
TimeoutStartSec=3600
Nice=19
IOSchedulingClass=best-effort
IOSchedulingPriority=7

# Resource limits
MemoryMax=2G
CPUQuota=50%

# Logging
StandardOutput=journal
StandardError=journal
SyslogIdentifier=backup

[Install]
WantedBy=multi-user.target
```

```ini
# /etc/systemd/system/backup.timer
[Unit]
Description=Daily Backup Timer

[Timer]
OnCalendar=*-*-* 02:00:00
RandomizedDelaySec=900
Persistent=true

[Install]
WantedBy=timers.target
```

```ini
# /etc/systemd/system/backup-weekly.timer
[Unit]
Description=Weekly Full Backup Timer

[Timer]
OnCalendar=Sun *-*-* 01:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

```ini
# /etc/systemd/system/backup-weekly.service
[Unit]
Description=Weekly Full Backup Service
After=network-online.target

[Service]
Type=oneshot
ExecStart=/opt/backup/backup_system.sh backup full
TimeoutStartSec=7200
```

### Enable Timers

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now backup.timer
sudo systemctl enable --now backup-weekly.timer
systemctl list-timers | grep backup
```

---

## Part 5: Quick Restore Procedures (10 mins)

### Emergency Restore Script

```bash
#!/bin/bash
# quick_restore.sh - Fast disaster recovery

set -euo pipefail

BACKUP_ROOT="${BACKUP_ROOT:-/backup}"

show_menu() {
    echo "=== Quick Restore Menu ==="
    echo
    echo "Available backups:"
    
    local i=1
    declare -a backups
    
    while IFS= read -r backup; do
        backups+=("$backup")
        local size
        size=$(du -sh "$backup" 2>/dev/null | cut -f1)
        echo "  $i) $(basename "$backup") ($size)"
        ((i++))
    done < <(find "$BACKUP_ROOT" -maxdepth 1 -type d -name "20*" | sort -r | head -10)
    
    echo
    read -rp "Select backup (1-${#backups[@]}): " selection
    
    if [[ $selection -ge 1 && $selection -le ${#backups[@]} ]]; then
        selected_backup="${backups[$((selection-1))]}"
        restore_menu "$selected_backup"
    else
        echo "Invalid selection"
        exit 1
    fi
}

restore_menu() {
    local backup_path="$1"
    
    echo
    echo "Selected: $backup_path"
    echo
    echo "Restore options:"
    echo "  1) Full system restore"
    echo "  2) Specific directory"
    echo "  3) MySQL database"
    echo "  4) PostgreSQL database"
    echo "  5) Cancel"
    echo
    
    read -rp "Select option: " option
    
    case "$option" in
        1)
            echo "WARNING: This will overwrite system files!"
            read -rp "Type 'CONFIRM' to proceed: " confirm
            [[ "$confirm" == "CONFIRM" ]] || exit 0
            
            restore_full_system "$backup_path"
            ;;
        2)
            read -rp "Directory to restore: " dir
            restore_directory "$backup_path" "$dir"
            ;;
        3)
            restore_mysql_menu "$backup_path"
            ;;
        4)
            restore_postgresql_menu "$backup_path"
            ;;
        5)
            echo "Cancelled"
            exit 0
            ;;
    esac
}

restore_full_system() {
    local backup_path="$1"
    
    echo "Starting full system restore..."
    
    rsync -avz --progress \
        "$backup_path/filesystem/" "/" \
        --exclude="/proc/*" \
        --exclude="/sys/*" \
        --exclude="/dev/*" \
        --exclude="/run/*" \
        --exclude="/tmp/*"
    
    echo "System restore completed"
    echo "IMPORTANT: Reboot recommended"
}

restore_directory() {
    local backup_path="$1"
    local target_dir="$2"
    
    local source_dir="$backup_path/filesystem$target_dir"
    
    if [[ ! -d "$source_dir" ]]; then
        echo "Directory not found in backup: $target_dir"
        return 1
    fi
    
    echo "Restoring $target_dir..."
    rsync -avz --progress "$source_dir/" "$target_dir/"
    echo "Restore completed"
}

restore_mysql_menu() {
    local backup_path="$1"
    
    echo "Available MySQL dumps:"
    local i=1
    declare -a dumps
    
    for dump in "$backup_path/mysql"/*.sql.gz; do
        [[ -f "$dump" ]] || continue
        dumps+=("$dump")
        echo "  $i) $(basename "$dump")"
        ((i++))
    done
    
    read -rp "Select dump: " selection
    
    if [[ $selection -ge 1 && $selection -le ${#dumps[@]} ]]; then
        local dump_file="${dumps[$((selection-1))]}"
        read -rp "Target database (leave empty for original): " target_db
        
        if [[ -n "$target_db" ]]; then
            mysql -e "CREATE DATABASE IF NOT EXISTS $target_db"
            gunzip -c "$dump_file" | mysql "$target_db"
        else
            gunzip -c "$dump_file" | mysql
        fi
        
        echo "MySQL restore completed"
    fi
}

restore_postgresql_menu() {
    local backup_path="$1"
    
    echo "Available PostgreSQL dumps:"
    local i=1
    declare -a dumps
    
    for dump in "$backup_path/postgresql"/*.dump; do
        [[ -f "$dump" ]] || continue
        dumps+=("$dump")
        echo "  $i) $(basename "$dump")"
        ((i++))
    done
    
    read -rp "Select dump: " selection
    
    if [[ $selection -ge 1 && $selection -le ${#dumps[@]} ]]; then
        local dump_file="${dumps[$((selection-1))]}"
        read -rp "Target database: " target_db
        
        sudo -u postgres createdb "$target_db" 2>/dev/null || true
        sudo -u postgres pg_restore -d "$target_db" "$dump_file"
        
        echo "PostgreSQL restore completed"
    fi
}

# Main
[[ $EUID -eq 0 ]] || { echo "Run as root"; exit 1; }
show_menu
```

---
