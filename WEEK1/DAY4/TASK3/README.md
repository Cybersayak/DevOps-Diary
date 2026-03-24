# **RAID Configuration & Recovery Mastery Guide**

## **📚 PART 1: RAID FUNDAMENTALS**

### **What is RAID and Why Use It?**

**RAID (Redundant Array of Independent Disks)** combines multiple physical disks into a single logical unit for **performance**, **redundancy**, or **both**.

**Without RAID (Single Disk):**
```bash
/dev/sda → Single point of failure
         → Limited by single disk speed
         → Data loss if disk fails
```

**With RAID:**
```bash
/dev/md0 → Multiple disks working together
         → Fault tolerance (except RAID 0)
         → Improved performance
         → Hot-swap capability
```

### **RAID Levels Explained**

```bash
# RAID 0 (Striping) - Speed, No Redundancy
Data: ABCDEFGH
Disk1: A C E G    # Data split across disks
Disk2: B D F H    # 2x read/write speed
Risk: Any disk failure = total data loss

# RAID 1 (Mirroring) - Redundancy, No Speed Gain
Data: ABCDEFGH
Disk1: ABCDEFGH   # Complete copy
Disk2: ABCDEFGH   # Mirror copy
Benefit: Can lose n-1 disks

# RAID 5 (Striping + Parity) - Balance
Data: ABCDEFGH
Disk1: A D G P₂   # Data + parity
Disk2: B E P₁ H   # Distributed parity
Disk3: C P₀ F I   # Can lose 1 disk

# RAID 10 (1+0) - Mirrored Stripes
Stripe1: [Disk1+Disk2] mirrored to
Stripe2: [Disk3+Disk4]
Best of both worlds: Speed + redundancy
```

## **🔧 PART 2: ENVIRONMENT SETUP**

### **Prerequisites Installation**
```bash
#!/bin/bash
# setup_raid_env.sh - Prepare RAID testing environment

# Install mdadm (software RAID management)
sudo apt update
sudo apt install -y mdadm

# Install monitoring tools
sudo apt install -y smartmontools sysstat

# Create loop devices for testing (8 x 1GB disks)
for i in {1..8}; do
    sudo dd if=/dev/zero of=/tmp/raid_disk${i}.img bs=1G count=1
    sudo losetup /dev/loop$((i+10)) /tmp/raid_disk${i}.img
done

# Verify loop devices
lsblk | grep loop
```

### **Clear Previous RAID Configurations**
```bash
# Important: Clean any existing RAID metadata
for i in {11..18}; do
    sudo mdadm --zero-superblock /dev/loop${i} 2>/dev/null
done

# Stop all RAID arrays
sudo mdadm --stop /dev/md* 2>/dev/null
```

## **🎯 PART 3: RAID CONFIGURATION GUIDE**

### **LEVEL 1: RAID 0 Configuration (Striping)**

```bash
#!/bin/bash
# raid0_setup.sh - Performance-focused striping

create_raid0() {
    echo "=== Creating RAID 0 Array ==="
    
    # Create RAID 0 with 2 disks
    sudo mdadm --create /dev/md0 \
        --level=0 \
        --raid-devices=2 \
        /dev/loop11 /dev/loop12
    
    # Why these parameters:
    # --level=0: RAID 0 for pure performance
    # --raid-devices=2: Minimum for striping
    
    # Check array status
    cat /proc/mdstat
    
    # Format and mount
    sudo mkfs.ext4 -F /dev/md0
    sudo mkdir -p /mnt/raid0
    sudo mount /dev/md0 /mnt/raid0
    
    # Verify stripe performance
    echo "Testing write speed..."
    dd if=/dev/zero of=/mnt/raid0/test bs=1M count=100 2>&1 | grep -o '[0-9.]* MB/s'
    
    # Show detailed info
    sudo mdadm --detail /dev/md0
}

# Check chunk size impact on performance
optimize_raid0_chunk() {
    local chunk_size=$1  # 64K, 128K, 256K, 512K
    
    # Recreate with specific chunk size
    sudo umount /mnt/raid0 2>/dev/null
    sudo mdadm --stop /dev/md0
    
    sudo mdadm --create /dev/md0 \
        --level=0 \
        --chunk=$chunk_size \
        --raid-devices=2 \
        /dev/loop11 /dev/loop12 \
        --assume-clean
    
    sudo mkfs.ext4 -F -E stride=$((chunk_size/4)),stripe-width=$((chunk_size/2)) /dev/md0
    # stride: chunk_size / block_size (4K)
    # stripe-width: stride * num_data_disks
    
    echo "RAID 0 with ${chunk_size}K chunks created"
}

create_raid0
```

### **LEVEL 2: RAID 1 Configuration (Mirroring)**

```bash
#!/bin/bash
# raid1_setup.sh - Redundancy-focused mirroring

create_raid1() {
    echo "=== Creating RAID 1 Array ==="
    
    # Create RAID 1 with 2 disks
    sudo mdadm --create /dev/md1 \
        --level=1 \
        --raid-devices=2 \
        /dev/loop13 /dev/loop14 \
        --bitmap=internal
    
    # --bitmap=internal: Speeds up resync after failures
    
    # Monitor sync progress
    watch -n1 'cat /proc/mdstat'
    
    # Wait for sync to complete
    while [ $(cat /proc/mdstat | grep -c "resync") -gt 0 ]; do
        sleep 2
    done
    
    # Create filesystem with journal on both mirrors
    sudo mkfs.ext4 -F /dev/md1
    sudo mkdir -p /mnt/raid1
    sudo mount /dev/md1 /mnt/raid1
    
    # Test redundancy
    echo "Testing redundancy..."
    echo "Original data" > /mnt/raid1/test.txt
    
    # Simulate disk failure
    sudo mdadm --fail /dev/md1 /dev/loop14
    
    # Verify data still accessible
    cat /mnt/raid1/test.txt
    
    # Show degraded status
    sudo mdadm --detail /dev/md1 | grep -A2 "State"
}

# Advanced: Add hot spare
add_raid1_spare() {
    # Add spare disk
    sudo mdadm --add /dev/md1 /dev/loop15
    
    # Verify spare status
    sudo mdadm --detail /dev/md1 | grep spare
    
    # Spare automatically replaces failed disk
    echo "Hot spare configured"
}

create_raid1
```

### **LEVEL 3: RAID 5 Configuration (Striping + Parity)**

```bash
#!/bin/bash
# raid5_setup.sh - Balanced performance and redundancy

create_raid5() {
    echo "=== Creating RAID 5 Array ==="
    
    # Minimum 3 disks for RAID 5
    sudo mdadm --create /dev/md5 \
        --level=5 \
        --raid-devices=3 \
        /dev/loop16 /dev/loop17 /dev/loop18 \
        --chunk=256
    
    # Monitor initialization (parity calculation)
    echo "Calculating parity blocks..."
    while grep -q resync /proc/mdstat; do
        progress=$(grep md5 /proc/mdstat | grep -o '[0-9.]*%')
        echo -ne "\rProgress: $progress"
        sleep 1
    done
    echo -e "\nRAID 5 initialization complete"
    
    # Optimize filesystem for RAID 5
    sudo mkfs.ext4 -F -E stride=64,stripe-width=128 /dev/md5
    # stride = chunk_size(256K) / block_size(4K) = 64
    # stripe-width = stride * (num_disks - 1) = 64 * 2 = 128
    
    sudo mkdir -p /mnt/raid5
    sudo mount /dev/md5 /mnt/raid5
    
    # Show capacity (n-1 disks worth)
    df -h /mnt/raid5
}

# Advanced: Grow RAID 5 array
grow_raid5() {
    echo "Adding disk to RAID 5..."
    
    # Add new disk
    sudo mdadm --add /dev/md5 /dev/loop11
    
    # Grow array
    sudo mdadm --grow /dev/md5 --raid-devices=4
    
    # Monitor reshape progress
    while grep -q reshape /proc/mdstat; do
        progress=$(grep md5 /proc/mdstat | grep -o '[0-9.]*%')
        echo -ne "\rReshape progress: $progress"
        sleep 2
    done
    
    # Resize filesystem
    sudo resize2fs /dev/md5
    echo "RAID 5 grown to 4 disks"
}

create_raid5
```

### **LEVEL 4: RAID 10 Configuration (Mirror + Stripe)**

```bash
#!/bin/bash
# raid10_setup.sh - Maximum performance and redundancy

create_raid10() {
    echo "=== Creating RAID 10 Array ==="
    
    # Need even number of disks (minimum 4)
    sudo mdadm --create /dev/md10 \
        --level=10 \
        --raid-devices=4 \
        --layout=n2 \
        /dev/loop11 /dev/loop12 /dev/loop13 /dev/loop14
    
    # --layout=n2: Near layout, 2 copies
    # Alternatives: f2 (far), o2 (offset)
    
    # Wait for sync
    while grep -q resync /proc/mdstat; do
        sync_status=$(grep md10 /proc/mdstat)
        echo -ne "\r$sync_status"
        sleep 1
    done
    
    # Create optimized filesystem
    sudo mkfs.ext4 -F -E stride=128,stripe-width=256 /dev/md10
    sudo mkdir -p /mnt/raid10
    sudo mount /dev/md10 /mnt/raid10
    
    # Performance test
    echo -e "\n=== Performance Test ==="
    echo "Sequential write:"
    dd if=/dev/zero of=/mnt/raid10/test bs=1M count=500 2>&1 | tail -1
    
    echo "Random read (4K blocks):"
    dd if=/mnt/raid10/test of=/dev/null bs=4K count=10000 2>&1 | tail -1
}

# Layout comparison
compare_raid10_layouts() {
    for layout in n2 f2 o2; do
        echo "Testing layout: $layout"
        
        sudo umount /mnt/raid10 2>/dev/null
        sudo mdadm --stop /dev/md10
        sudo mdadm --zero-superblock /dev/loop{11..14}
        
        sudo mdadm --create /dev/md10 \
            --level=10 \
            --raid-devices=4 \
            --layout=$layout \
            /dev/loop{11..14} \
            --assume-clean
        
        sudo mkfs.ext4 -F /dev/md10 >/dev/null 2>&1
        sudo mount /dev/md10 /mnt/raid10
        
        # Test performance
        write_speed=$(dd if=/dev/zero of=/mnt/raid10/test bs=1M count=100 2>&1 | grep -o '[0-9.]* MB/s' | tail -1)
        echo "  Write speed: $write_speed"
        
        read_speed=$(dd if=/mnt/raid10/test of=/dev/null bs=1M 2>&1 | grep -o '[0-9.]* MB/s' | tail -1)
        echo "  Read speed: $read_speed"
    done
}

create_raid10
```

## **🛠️ PART 4: FAILURE & RECOVERY PROCEDURES**

### **Disk Failure Simulation & Recovery**

```bash
#!/bin/bash
# raid_recovery.sh - Handle disk failures

# RAID 1 Failure & Recovery
raid1_failure_recovery() {
    echo "=== RAID 1 Failure Simulation ==="
    
    # Create test data
    echo "Critical data before failure" > /mnt/raid1/important.txt
    md5_before=$(md5sum /mnt/raid1/important.txt)
    
    # Simulate disk failure
    failed_disk="/dev/loop14"
    sudo mdadm --fail /dev/md1 $failed_disk
    
    # Check degraded status
    sudo mdadm --detail /dev/md1 | grep -E "State|Failed|Degraded"
    
    # Remove failed disk
    sudo mdadm --remove /dev/md1 $failed_disk
    
    # Prepare replacement disk (zero superblock)
    sudo mdadm --zero-superblock $failed_disk
    
    # Add replacement disk
    sudo mdadm --add /dev/md1 $failed_disk
    
    # Monitor rebuild
    while grep -q recovery /proc/mdstat; do
        recovery=$(grep md1 /proc/mdstat | grep -o '[0-9.]*%' | head -1)
        echo -ne "\rRecovery progress: $recovery"
        sleep 1
    done
    echo -e "\nRecovery complete!"
    
    # Verify data integrity
    md5_after=$(md5sum /mnt/raid1/important.txt)
    if [ "$md5_before" = "$md5_after" ]; then
        echo "✓ Data integrity verified"
    else
        echo "✗ Data corruption detected!"
    fi
}

# RAID 5 Double Failure Recovery
raid5_catastrophic_recovery() {
    echo "=== RAID 5 Catastrophic Recovery ==="
    
    # Backup metadata first
    sudo mdadm --detail --scan > /tmp/mdadm_backup.conf
    sudo mdadm --examine /dev/loop{16..18} > /tmp/raid5_metadata.txt
    
    # Simulate first disk failure
    sudo mdadm --fail /dev/md5 /dev/loop16
    echo "First disk failed - array degraded but operational"
    
    # Simulate second failure (catastrophic)
    sudo mdadm --fail /dev/md5 /dev/loop17
    echo "Second disk failed - array failed!"
    
    # Force assembly with available disks
    sudo mdadm --stop /dev/md5
    
    # Attempt recovery with --force
    sudo mdadm --assemble --force /dev/md5 /dev/loop16 /dev/loop17 /dev/loop18
    
    if [ $? -eq 0 ]; then
        echo "Array assembled in degraded mode"
        
        # Immediately backup data
        sudo mkdir -p /mnt/emergency_backup
        sudo rsync -av /mnt/raid5/ /mnt/emergency_backup/
        
        # Rebuild array
        sudo mdadm --add /dev/md5 /dev/loop11
        sudo mdadm --add /dev/md5 /dev/loop12
    else
        echo "Manual intervention required - check metadata backup"
    fi
}

# RAID 10 Multiple Failure Handling
raid10_smart_recovery() {
    echo "=== RAID 10 Smart Recovery ==="
    
    # Identify mirror pairs
    sudo mdadm --detail /dev/md10 | grep -A20 "Number.*Major.*Minor"
    
    # Safe failure (different mirror groups)
    echo "Simulating safe failures (different mirrors)..."
    sudo mdadm --fail /dev/md10 /dev/loop11  # Mirror 1
    sudo mdadm --fail /dev/md10 /dev/loop13  # Mirror 2
    
    # Array still operational
    echo "Array status after safe failures:"
    sudo mdadm --detail /dev/md10 | grep State
    
    # Recovery procedure
    for disk in /dev/loop11 /dev/loop13; do
        sudo mdadm --remove /dev/md10 $disk
        sudo mdadm --zero-superblock $disk
        sudo mdadm --add /dev/md10 $disk
    done
    
    # Monitor parallel rebuild
    while grep -q recovery /proc/mdstat; do
        cat /proc/mdstat | grep md10
        sleep 2
    done
}

raid1_failure_recovery
```

## **📊 PART 5: MONITORING & PERFORMANCE**

### **Comprehensive RAID Monitoring**

```bash
#!/bin/bash
# raid_monitor.sh - Real-time RAID health monitoring

# Email alert configuration
ALERT_EMAIL="admin@example.com"
SMTP_SERVER="localhost"

# Main monitoring function
monitor_raid_health() {
    local check_interval=${1:-60}  # Default 60 seconds
    
    while true; do
        clear
        echo "=== RAID Health Monitor - $(date) ==="
        
        # Check all arrays
        for md in $(ls /dev/md* 2>/dev/null); do
            array_name=$(basename $md)
            echo -e "\n[$array_name Status]"
            
            # Get array details
            detail=$(sudo mdadm --detail $md 2>/dev/null)
            
            if [ $? -ne 0 ]; then
                echo "ERROR: Cannot read array $md"
                continue
            fi
            
            # Extract key metrics
            state=$(echo "$detail" | grep "State :" | awk '{print $3}')
            active=$(echo "$detail" | grep "Active Devices" | awk '{print $4}')
            working=$(echo "$detail" | grep "Working Devices" | awk '{print $4}')
            failed=$(echo "$detail" | grep "Failed Devices" | awk '{print $4}')
            spare=$(echo "$detail" | grep "Spare Devices" | awk '{print $4}')
            
            # Display status with color coding
            case $state in
                "clean"|"active")
                    echo -e "\e[32m✓ State: $state\e[0m"  # Green
                    ;;
                "degraded")
                    echo -e "\e[33m⚠ State: $state\e[0m"  # Yellow
                    send_alert "WARNING" "$array_name is degraded"
                    ;;
                *)
                    echo -e "\e[31m✗ State: $state\e[0m"  # Red
                    send_alert "CRITICAL" "$array_name has failed"
                    ;;
            esac
            
            echo "  Active/Working: $active/$working"
            echo "  Failed: $failed, Spares: $spare"
            
            # Check rebuild progress
            if grep -q "${array_name#md}" /proc/mdstat; then
                if grep -q recovery /proc/mdstat; then
                    progress=$(grep "${array_name#md}" /proc/mdstat | grep -o '[0-9.]*%' | head -1)
                    echo -e "  \e[33mRebuild Progress: $progress\e[0m"
                fi
            fi
            
            # Disk health check
            echo "  Disk Health:"
            for disk in $(echo "$detail" | grep "/dev/" | awk '{print $7}' | grep "^/dev"); do
                if [ -b "$disk" ]; then
                    # Check SMART status if available
                    if sudo smartctl -H $disk &>/dev/null; then
                        health=$(sudo smartctl -H $disk | grep "SMART overall" | awk '{print $NF}')
                        if [ "$health" = "PASSED" ]; then
                            echo "    $disk: ✓ Healthy"
                        else
                            echo "    $disk: ✗ FAILING"
                            send_alert "CRITICAL" "Disk $disk is failing in $array_name"
                        fi
                    fi
                fi
            done
        done
        
        # Performance metrics
        echo -e "\n[Performance Metrics]"
        for md in $(ls /sys/block/md*/md/sync_speed 2>/dev/null); do
            array=$(echo $md | cut -d'/' -f4)
            speed=$(cat $md 2>/dev/null)
            if [ "$speed" != "0" ]; then
                echo "  $array sync speed: ${speed} KB/s"
            fi
        done
        
        # Check mdadm monitor daemon
        if pgrep mdadm >/dev/null; then
            echo -e "\n\e[32m✓ mdadm monitor daemon running\e[0m"
        else
            echo -e "\n\e[31m✗ mdadm monitor daemon not running\e[0m"
        fi
        
        sleep $check_interval
    done
}

# Send email alerts
send_alert() {
    local severity=$1
    local message=$2
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    
    # Log to syslog
    logger -t "RAID-Monitor" -p user.err "[$severity] $message"
    
    # Send email (requires mail command)
    if command -v mail >/dev/null 2>&1; then
        echo "RAID Alert [$severity] at $timestamp: $message" | \
            mail -s "RAID Alert: $severity" $ALERT_EMAIL
    fi
    
    # Write to alert file
    echo "[$timestamp] $severity: $message" >> /var/log/raid_alerts.log
}

# Performance testing function
test_raid_performance() {
    local device=$1
    local mount_point=$2
    
    echo "=== RAID Performance Test for $device ==="
    
    # Sequential write test
    echo "Sequential Write (1GB):"
    sync && echo 3 > /proc/sys/vm/drop_caches
    time dd if=/dev/zero of=$mount_point/test_write bs=1M count=1024 oflag=direct
    
    # Sequential read test
    echo -e "\nSequential Read (1GB):"
    sync && echo 3 > /proc/sys/vm/drop_caches
    time dd if=$mount_point/test_write of=/dev/null bs=1M iflag=direct
    
    # Random 4K write IOPS
    echo -e "\nRandom 4K Write IOPS:"
    fio --name=rand_write --ioengine=libaio --rw=randwrite \
        --bs=4k --direct=1 --size=100M --numjobs=4 \
        --runtime=10 --group_reporting --filename=$mount_point/test_fio
    
    # Cleanup
    rm -f $mount_point/test_write $mount_point/test_fio
}

# Start monitoring
monitor_raid_health
```

### **Advanced Performance Tuning**

```bash
#!/bin/bash
# raid_tuning.sh - Optimize RAID performance

optimize_raid_performance() {
    local device=$1
    local raid_level=$2
    
    echo "=== Optimizing $device (RAID $raid_level) ==="
    
    # Set stripe cache size (improves sequential performance)
    if [ -f /sys/block/${device#/dev/}/md/stripe_cache_size ]; then
        current=$(cat /sys/block/${device#/dev/}/md/stripe_cache_size)
        echo "Current stripe cache: $current"
        
        # Increase for better performance (uses more RAM)
        echo 8192 > /sys/block/${device#/dev/}/md/stripe_cache_size
        echo "Stripe cache set to 8192"
    fi
    
    # Set read-ahead (improves sequential reads)
    current_ra=$(blockdev --getra $device)
    echo "Current read-ahead: $current_ra sectors"
    
    # Set based on RAID level
    case $raid_level in
        0|10)
            # Higher read-ahead for striped arrays
            sudo blockdev --setra 8192 $device
            ;;
        1)
            # Moderate read-ahead for mirrors
            sudo blockdev --setra 2048 $device
            ;;
        5|6)
            # Calculate based on stripe size
            sudo blockdev --setra 4096 $device
            ;;
    esac
    
    # Set scheduler for SSDs vs HDDs
    for disk in $(mdadm --detail $device | grep "/dev/" | awk '{print $7}' | grep "^/dev"); do
        if lsblk -d -o name,rota $disk | grep -q " 0$"; then
            # SSD detected (rotational=0)
            echo noop > /sys/block/$(basename $disk)/queue/scheduler
            echo "Set noop scheduler for SSD $disk"
        else
            # HDD detected
            echo deadline > /sys/block/$(basename $disk)/queue/scheduler
            echo "Set deadline scheduler for HDD $disk"
        fi
    done
    
    # Set bitmap chunk size for faster resync
    if mdadm --detail $device | grep -q "Intent Bitmap"; then
        mdadm --grow $device --bitmap-chunk=64M
        echo "Bitmap chunk size set to 64M"
    fi
    
    # Enable write-intent bitmap for faster recovery
    if ! mdadm --detail $device | grep -q "Intent Bitmap"; then
        mdadm --grow $device --bitmap=internal
        echo "Write-intent bitmap enabled"
    fi
}

# System-wide RAID tuning
tune_system_for_raid() {
    echo "=== System-wide RAID Optimization ==="
    
    # Increase minimum and maximum resync speed
    echo 50000 > /proc/sys/dev/raid/speed_limit_min
    echo 200000 > /proc/sys/dev/raid/speed_limit_max
    echo "Resync speed limits: 50-200 MB/s"
    
    # Tune VM settings for better I/O
    echo 10 > /proc/sys/vm/dirty_ratio
    echo 5 > /proc/sys/vm/dirty_background_ratio
    echo "VM dirty ratios optimized"
    
    # Set swappiness low for database servers
    echo 10 > /proc/sys/vm/swappiness
    echo "Swappiness set to 10"
    
    # Make settings persistent
    cat >> /etc/sysctl.d/99-raid-tuning.conf <<EOF
# RAID Performance Tuning
vm.dirty_ratio = 10
vm.dirty_background_ratio = 5
vm.swappiness = 10
EOF
    
    sysctl -p /etc/sysctl.d/99-raid-tuning.conf
}
```

## **🐛 PART 6: ERROR HANDLING & DEBUGGING**

### **Common RAID Problems and Solutions**

```bash
#!/bin/bash
# raid_troubleshoot.sh - Diagnose and fix RAID issues

# Problem 1: Array won't assemble
fix_assembly_issues() {
    echo "=== Troubleshooting Array Assembly ==="
    
    # Check for existing arrays
    cat /proc/mdstat
    
    # Scan for all arrays
    sudo mdadm --assemble --scan --verbose
    
    # If that fails, try manual assembly
    echo "Attempting manual assembly..."
    
    # Examine all potential member disks
    for disk in /dev/loop{11..18}; do
        echo "Examining $disk:"
        sudo mdadm --examine $disk 2>/dev/null | grep -E "Array UUID|Device Role"
    done
    
    # Force assembly with UUID
    read -p "Enter Array UUID: " uuid
    sudo mdadm --assemble --force --uuid=$uuid /dev/md127 /dev/loop*
    
    # Update mdadm.conf
    sudo mdadm --detail --scan >> /etc/mdadm/mdadm.conf
}

# Problem 2: Slow rebuild/resync
fix_slow_resync() {
    echo "=== Fixing Slow Resync ==="
    
    # Check current speed
    current_speed=$(cat /proc/sys/dev/raid/speed_limit_min)
    echo "Current minimum speed: $current_speed KB/s"
    
    # Temporarily boost resync speed
    echo 100000 > /proc/sys/dev/raid/speed_limit_min
    echo 300000 > /proc/sys/dev/raid/speed_limit_max
    
    # Check I/O scheduler
    for disk in $(mdadm --detail /dev/md* | grep "/dev/" | awk '{print $7}' | grep "^/dev"); do
        scheduler=$(cat /sys/block/$(basename $disk)/queue/scheduler)
        echo "$disk: $scheduler"
    done
    
    # Monitor speed
    watch -n1 'cat /proc/mdstat | grep -E "speed|finish"'
}

# Problem 3: Degraded array won't accept new disk
fix_degraded_array() {
    local array=$1
    local new_disk=$2
    
    echo "=== Fixing Degraded Array ==="
    
    # Check array state
    sudo mdadm --detail $array | grep -E "State|Update Time"
    
    # Check disk for old metadata
    if sudo mdadm --examine $new_disk 2>/dev/null | grep -q "Array UUID"; then
        echo "Disk has old metadata, clearing..."
        sudo mdadm --zero-superblock $new_disk
    fi
    
    # Check for disk issues
    if ! sudo fdisk -l $new_disk >/dev/null 2>&1; then
        echo "ERROR: Disk $new_disk not accessible"
        return 1
    fi
    
    # Try to add with force
    sudo mdadm --add $array $new_disk --force
    
    if [ $? -ne 0 ]; then
        echo "Attempting metadata recreation..."
        sudo mdadm --create $array --level=1 --raid-devices=2 \
              --assume-clean --metadata=1.2 \
              missing $new_disk
    fi
}

# Problem 4: Data corruption detection
check_data_integrity() {
    local array=$1
    local mount_point=$2
    
    echo "=== Checking Data Integrity ==="
    
    # Run array consistency check
    echo check > /sys/block/${array#/dev/}/md/sync_action
    
    # Monitor check progress
    while [ "$(cat /sys/block/${array#/dev/}/md/sync_action)" = "check" ]; do
        progress=$(cat /proc/mdstat | grep ${array#/dev/} | grep -o '[0-9.]*%')
        echo -ne "\rCheck progress: $progress"
        sleep 2
    done
    
    # Check for mismatches
    mismatches=$(cat /sys/block/${array#/dev/}/md/mismatch_cnt)
    if [ "$mismatches" -gt 0 ]; then
        echo -e "\n⚠️  WARNING: $mismatches mismatches found!"
        echo "Running repair..."
        echo repair > /sys/block/${array#/dev/}/md/sync_action
    else
        echo -e "\n✓ No mismatches found"
    fi
    
    # Filesystem check
    sudo umount $mount_point
    sudo e2fsck -f $array
    sudo mount $array $mount_point
}
```

## **📝 PART 7: BEST PRACTICES & COMMON MISTAKES**

### **Best Practices Implementation**

```bash
#!/bin/bash
# raid_best_practices.sh - Production-ready RAID setup

implement_best_practices() {
    echo "=== RAID Best Practices Implementation ==="
    
    # 1. Always use write-intent bitmaps
    for md in /dev/md*; do
        if ! mdadm --detail $md 2>/dev/null | grep -q "Intent Bitmap"; then
            mdadm --grow $md --bitmap=internal
            echo "✓ Bitmap enabled for $md"
        fi
    done
    
    # 2. Set up email notifications
    cat > /etc/mdadm/mdadm.conf <<EOF
# mdadm.conf
DEVICE /dev/loop[11-18]
ARRAY /dev/md0 metadata=1.2
ARRAY /dev/md1 metadata=1.2
MAILADDR sysadmin@example.com
MAILFROM raid-monitor@$(hostname)
PROGRAM /usr/local/bin/raid-alert.sh
EOF
    
    # 3. Create monitoring script
    cat > /usr/local/bin/raid-alert.sh <<'EOF'
#!/bin/bash
event=$1
device=$2
component=$3

case $event in
    Fail)
        echo "CRITICAL: Device $component failed in $device" | \
            mail -s "RAID FAILURE on $(hostname)" sysadmin@example.com
        ;;
    DegradedArray)
        echo "WARNING: Array $device is degraded" | \
            mail -s "RAID Degraded on $(hostname)" sysadmin@example.com
        ;;
esac
EOF
    chmod +x /usr/local/bin/raid-alert.sh
    
    # 4. Set up regular scrubbing
    cat > /etc/cron.d/raid-scrub <<EOF
# RAID array scrubbing - monthly
0 2 1 * * root /usr/local/bin/raid-scrub.sh
EOF
    
    # 5. Create scrub script
    cat > /usr/local/bin/raid-scrub.sh <<'EOF'
#!/bin/bash
for md in /sys/block/md*/md/sync_action; do
    echo check > $md
done
EOF
    chmod +x /usr/local/bin/raid-scrub.sh
    
    # 6. Implement SMART monitoring
    for disk in $(mdadm --detail /dev/md* 2>/dev/null | grep "/dev/" | awk '{print $7}' | sort -u | grep "^/dev"); do
        smartctl -t short $disk
        echo "✓ SMART test scheduled for $disk"
    done
    
    # 7. Document array configuration
    mdadm --detail --scan > /root/mdadm.conf.backup
    for md in /dev/md*; do
        mdadm --detail $md > /root/raid-${md#/dev/}-details.txt
    done
    
    echo "Best practices implemented successfully"
}

# Common mistakes to avoid
demonstrate_common_mistakes() {
    echo "=== Common RAID Mistakes to Avoid ==="
    
    cat <<'MISTAKES'
1. ❌ Using RAID 0 for important data
   ✓ Use RAID 1/5/10 for data that matters

2. ❌ Not monitoring array health
   ✓ Set up mdadm monitoring daemon

3. ❌ Ignoring SMART warnings
   ✓ Replace disks proactively

4. ❌ No spare disks
   ✓ Keep hot spares for critical arrays

5. ❌ Same batch/model disks
   ✓ Mix purchase dates to avoid simultaneous failures

6. ❌ No backup of RAID config
   ✓ Backup mdadm.conf and array metadata

7. ❌ Assuming RAID = Backup
   ✓ RAID is for uptime, not backup!

8. ❌ Wrong chunk size
   ✓ Large chunks for sequential, small for random I/O

9. ❌ Not testing recovery procedures
   ✓ Practice failure scenarios regularly

10. ❌ Rebuilding during peak hours
    ✓ Schedule rebuilds during maintenance windows
MISTAKES
}
```

## **🎯 PART 8: PRACTICE CHALLENGES**

### **Challenge 1: Build Resilient Storage System**

```bash
#!/bin/bash
# challenge1_resilient_storage.sh

# Challenge: Create a multi-tier storage system
# Requirements:
# - Fast tier: RAID 10 for databases
# - Standard tier: RAID 5 for files  
# - Archive tier: RAID 1 for backups
# - Implement monitoring for all tiers

build_storage_tiers() {
    echo "=== Challenge 1: Multi-tier Storage ==="
    
    # Your implementation here
    # TODO: Create the three tiers
    # TODO: Set appropriate chunk sizes
    # TODO: Configure monitoring
    
    # Validation
    validate_challenge1
}

validate_challenge1() {
    score=0
    
    # Check RAID 10 exists
    if mdadm --detail /dev/md10 2>/dev/null | grep -q "Raid Level : raid10"; then
        echo "✓ RAID 10 created"
        ((score+=33))
    fi
    
    # Check RAID 5 exists
    if mdadm --detail /dev/md5 2>/dev/null | grep -q "Raid Level : raid5"; then
        echo "✓ RAID 5 created"
        ((score+=33))
    fi
    
    # Check RAID 1 exists
    if mdadm --detail /dev/md1 2>/dev/null | grep -q "Raid Level : raid1"; then
        echo "✓ RAID 1 created"
        ((score+=34))
    fi
    
    echo "Score: $score/100"
}
```

### **Challenge 2: Automated Recovery System**

```bash
#!/bin/bash
# challenge2_auto_recovery.sh

# Challenge: Create automated failure recovery
# Requirements:
# - Detect disk failures automatically
# - Replace with spare automatically
# - Send notifications
# - Log all actions

create_auto_recovery() {
    echo "=== Challenge 2: Automated Recovery ==="
    
    # Your implementation here
    # TODO: Monitor daemon setup
    # TODO: Spare disk management
    # TODO: Alert system
    # TODO: Recovery automation
    
    # Validation
    validate_challenge2
}

validate_challenge2() {
    score=0
    
    # Check monitoring daemon
    if pgrep mdadm >/dev/null; then
        echo "✓ Monitoring active"
        ((score+=25))
    fi
    
    # Check spare configuration
    if mdadm --detail /dev/md* 2>/dev/null | grep -q "Spare Devices"; then
        echo "✓ Spare configured"
        ((score+=25))
    fi
    
    # Check alert script
    if [ -x /usr/local/bin/raid-alert.sh ]; then
        echo "✓ Alert script ready"
        ((score+=25))
    fi
    
    # Check logging
    if [ -f /var/log/raid_alerts.log ]; then
        echo "✓ Logging configured"
        ((score+=25))
    fi
    
    echo "Score: $score/100"
}
```

## **🚀 PART 9: FINAL TEST - MULTI-DISK FAILURE RECOVERY**

```bash
#!/bin/bash
# final_test_catastrophic_recovery.sh
# Goal: Recover from multi-disk RAID failure scenario

set -euo pipefail

# Configuration
LOG_FILE="/var/log/raid_recovery_$(date +%Y%m%d_%H%M%S).log"
BACKUP_DIR="/mnt/raid_backup"
RECOVERY_REPORT="/tmp/recovery_report_$(date +%Y%m%d_%H%M%S).txt"

# Logging
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a $LOG_FILE
}

# Setup test environment
setup_test_environment() {
    log_message "=== Setting up test environment ==="
    
    # Create test arrays
    log_message "Creating RAID 5 array for testing..."
    sudo mdadm --create /dev/md100 --level=5 --raid-devices=4 \
        /dev/loop11 /dev/loop12 /dev/loop13 /dev/loop14 \
        --spare-devices=1 /dev/loop15 \
        --bitmap=internal \
        --assume-clean
    
    # Create filesystem and mount
    sudo mkfs.ext4 -F /dev/md100
    sudo mkdir -p /mnt/raid_test
    sudo mount /dev/md100 /mnt/raid_test
    
    # Create critical test data
    log_message "Creating test data..."
    for i in {1..10}; do
        dd if=/dev/urandom of=/mnt/raid_test/critical_data_$i.bin bs=1M count=10
        md5sum /mnt/raid_test/critical_data_$i.bin >> /tmp/checksums_original.txt
    done
    
    log_message "Test environment ready"
}

# Simulate catastrophic failure
simulate_catastrophic_failure() {
    log_message "=== Simulating catastrophic failure ==="
    
    # First disk fails
    log_message "Disk 1 failing..."
    sudo mdadm --fail /dev/md100 /dev/loop11
    sleep 2
    
    # Spare takes over automatically
    log_message "Spare rebuilding..."
    while grep -q recovery /proc/mdstat; do
        progress=$(grep md100 /proc/mdstat | grep -o '[0-9.]*%' | head -1)
        log_message "Recovery progress: $progress"
        sleep 5
    done
    
    # Second disk fails during operation
    log_message "CRITICAL: Second disk failing!"
    sudo mdadm --fail /dev/md100 /dev/loop12
    
    # Array is now degraded with no spares
    state=$(mdadm --detail /dev/md100 | grep "State :" | awk '{print $3}')
    log_message "Array state: $state"
    
    # Third disk fails - catastrophic!
    log_message "CATASTROPHIC: Third disk failing!"
    sudo mdadm --fail /dev/md100 /dev/loop13
    
    # Array should be failed/inactive now
    if ! mount | grep -q /mnt/raid_test; then
        log_message "File system automatically unmounted"
    fi
}

# Recovery procedure
perform_recovery() {
    log_message "=== Starting recovery procedure ==="
    
    # Step 1: Stop the failed array
    log_message "Stopping failed array..."
    sudo umount /mnt/raid_test 2>/dev/null || true
    sudo mdadm --stop /dev/md100
    
    # Step 2: Examine all disks
    log_message "Examining disk metadata..."
    for disk in /dev/loop{11..15}; do
        if sudo mdadm --examine $disk &>/dev/null; then
            log_message "$disk: Metadata present"
            sudo mdadm --examine $disk | grep -E "Events|Update Time|State" | \
                sed 's/^/  /' >> $LOG_FILE
        fi
    done
    
    # Step 3: Identify most recent good disks
    log_message "Identifying recoverable disks..."
    
    # Find disk with highest event count
    highest_events=0
    best_disks=""
    for disk in /dev/loop{11..15}; do
        events=$(sudo mdadm --examine $disk 2>/dev/null | \
                grep Events | awk '{print $3}')
        if [ -n "$events" ]; then
            if [ "$events" -ge "$highest_events" ]; then
                highest_events=$events
                best_disks="$best_disks $disk"
            fi
        fi
    done
    
    log_message "Best disks for recovery: $best_disks"
    
    # Step 4: Force assembly with available disks
    log_message "Attempting forced assembly..."
    if sudo mdadm --assemble --force /dev/md100 $best_disks; then
        log_message "SUCCESS: Array assembled in degraded mode!"
        
        # Mount read-only first
        sudo mount -o ro /dev/md100 /mnt/raid_test
        
        # Step 5: Immediate data backup
        log_message "Backing up data immediately..."
        sudo mkdir -p $BACKUP_DIR
        
        if sudo rsync -av /mnt/raid_test/ $BACKUP_DIR/; then
            log_message "Data backed up successfully to $BACKUP_DIR"
            
            # Verify backup
            for file in $BACKUP_DIR/critical_data_*.bin; do
                md5sum "$file" >> /tmp/checksums_recovered.txt
            done
            
            # Compare checksums
            if diff /tmp/checksums_original.txt /tmp/checksums_recovered.txt >/dev/null; then
                log_message "✓ Data integrity verified - all files recovered intact!"
                recovery_status="SUCCESS"
            else
                log_message "⚠ Some data corruption detected"
                recovery_status="PARTIAL"
            fi
        else
            log_message "✗ Backup failed!"
            recovery_status="FAILED"
        fi
        
        # Step 6: Rebuild array properly
        log_message "Rebuilding array..."
        sudo umount /mnt/raid_test
        
        # Add new disks to rebuild
        for disk in /dev/loop{16..18}; do
            if [ -b "$disk" ]; then
                sudo mdadm --add /dev/md100 $disk
                log_message "Added $disk for rebuild"
                break
            fi
        done
        
        # Monitor rebuild
        while grep -q recovery /proc/mdstat; do
            progress=$(grep md100 /proc/mdstat | grep -o '[0-9.]*%' | head -1)
            log_message "Rebuild progress: $progress"
            sleep 5
        done
        
        log_message "Array rebuild complete"
        
    else
        log_message "✗ Forced assembly failed - trying alternate recovery"
        recovery_status="FAILED"
        
        # Alternative: Create new array with --assume-clean
        log_message "Attempting overlay assembly..."
        sudo mdadm --create /dev/md101 --level=5 --raid-devices=4 \
            --assume-clean \
            /dev/loop14 /dev/loop15 missing missing
        
        if sudo mount -o ro /dev/md101 /mnt/raid_test 2>/dev/null; then
            log_message "Partial data may be recoverable"
            recovery_status="PARTIAL"
        fi
    fi
    
    return 0
}

# Generate recovery report
generate_report() {
    log_message "=== Generating recovery report ==="
    
    cat > $RECOVERY_REPORT <<EOF
=====================================
RAID Catastrophic Recovery Report
=====================================
Date: $(date)
Hostname: $(hostname)
Scenario: Multi-disk RAID 5 failure

Initial Configuration:
- RAID Level: 5
- Devices: 4 + 1 spare
- Total Failures: 3 disks

Recovery Status: $recovery_status

Recovery Steps Taken:
1. Failed array stopped
2. Disk metadata examined
3. Forced assembly attempted
4. Data backup performed
5. Array rebuilt

Data Recovery:
- Original files: $(ls /mnt/raid_test 2>/dev/null | wc -l)
- Recovered files: $(ls $BACKUP_DIR 2>/dev/null | wc -l)
- Integrity check: $recovery_status

Lessons Learned:
- Write-intent bitmap enabled: YES
- Spare disk available: YES
- Recovery time: $(grep "Starting recovery" $LOG_FILE | head -1 | cut -d' ' -f1-2) to $(date '+%Y-%m-%d %H:%M:%S')

Recommendations:
1. Maintain multiple spare disks
2. Regular backup verification
3. Monitor disk health proactively
4. Test recovery procedures quarterly
5. Document array configurations

Log file: $LOG_FILE
=====================================
EOF
    
    cat $RECOVERY_REPORT
    log_message "Report saved to $RECOVERY_REPORT"
}

# Cleanup function
cleanup() {
    log_message "Cleaning up test environment..."
    sudo umount /mnt/raid_test 2>/dev/null || true
    sudo mdadm --stop /dev/md100 2>/dev/null || true
    sudo mdadm --stop /dev/md101 2>/dev/null || true
    
    for disk in /dev/loop{11..18}; do
        sudo mdadm --zero-superblock $disk 2>/dev/null || true
    done
}

# Main execution
main() {
    log_message "===== Starting RAID Catastrophic Recovery Test ====="
    
    # Initialize recovery status
    recovery_status="UNKNOWN"
    
    # Setup
    setup_test_environment
    
    # Create failure scenario
    simulate_catastrophic_failure
    
    # Perform recovery
    perform_recovery
    
    # Generate report
    generate_report
    
    # Cleanup
    read -p "Press Enter to cleanup test environment..."
    cleanup
    
    log_message "===== Test Complete ====="
    
    # Final verdict
    case $recovery_status in
        SUCCESS)
            echo -e "\n\e[32m✓ TEST PASSED: Full recovery achieved!\e[0m"
            exit 0
            ;;
        PARTIAL)
            echo -e "\n\e[33m⚠ TEST PARTIAL: Some data recovered\e[0m"
            exit 1
            ;;
        FAILED)
            echo -e "\n\e[31m✗ TEST FAILED: Recovery unsuccessful\e[0m"
            exit 2
            ;;
    esac
}

# Error handling
trap 'log_message "ERROR: Script failed at line $LINENO"' ERR
trap cleanup EXIT

# Run the test
main "$@"
```

## **📅 LEARNING TIMELINE & CHECKPOINTS**

### **Week 1: Foundation (Days 1-7)**

**Days 1-2: RAID Concepts & Basic Setup**
- Understand RAID levels (0,1,5,10)
- Install mdadm and create loop devices
- Create first RAID 1 array

**Days 3-4: Multiple RAID Types**
- Build RAID 0, 5, and 10 arrays
- Understand chunk sizes and performance impact
- Practice with different filesystem types

**Days 5-6: Monitoring & Management**
- Set up monitoring with mdadm
- Configure email alerts
- Use mdstat and mdadm commands fluently

**Day 7: Review & Practice**
- Complete Challenge 1
- Document all arrays created

### **Week 2: Advanced Operations (Days 8-14)**

**Days 8-9: Failure Simulation**
- Practice single disk failures
- Master the recovery workflow
- Understand degraded states

**Days 10-11: Performance Optimization**
- Tune chunk sizes
- Optimize for workload types
- Implement caching strategies

**Days 12-13: Complex Scenarios**
- Multi-disk failures
- Array migration and reshaping
- Complete Challenge 2

**Day 14: Final Test**
- Execute catastrophic recovery test
- Document recovery procedures
- Achieve successful recovery

### **Validation Checkpoints**

**After Day 3:** Create and verify all 4 RAID types
```bash
for level in 0 1 5 10; do
    mdadm --detail /dev/md$level | grep "Raid Level"
done
```

**After Day 7:** Handle single disk failure without data loss
```bash
# Test: Fail disk, recover, verify data integrity
```

**After Day 14:** Successfully recover from multi-disk failure
```bash
# Test: Pass the catastrophic recovery scenario
```

## **🎖️ MASTERY INDICATORS**

You've mastered RAID when you can:

1. ✅ Create any RAID level from memory
2. ✅ Calculate usable space and redundancy for any configuration  
3. ✅ Diagnose and fix degraded arrays without documentation
4. ✅ Optimize RAID performance for specific workloads
5. ✅ Implement comprehensive monitoring and alerting
6. ✅ Recover from catastrophic multi-disk failures
7. ✅ Automate RAID management tasks
8. ✅ Make informed decisions on RAID levels for production
9. ✅ Perform online array reshaping and migration
10. ✅ Document and maintain RAID configurations properly

**Final Advice:** Practice failure scenarios repeatedly in test environments. Real confidence comes from successfully recovering from disasters multiple times. Always remember: **RAID is not a backup solution** - it's for availability and performance!