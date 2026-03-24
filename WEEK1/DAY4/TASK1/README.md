# **DISK MANAGEMENT MASTERY GUIDE**
*Complete Ubuntu/Linux Scripting & Administration Path*

## **📚 TABLE OF CONTENTS**

1. [Foundation Knowledge](#foundation)
2. [Tool Mastery: fdisk, parted, gdisk](#tools)
3. [Filesystem Creation & Management](#filesystems)
4. [Automatic Mounting with fstab](#fstab)
5. [Advanced Techniques & Optimization](#advanced)
6. [Error Handling & Recovery](#errors)
7. [Practice Labs & Challenges](#practice)
8. [15-Minute Speed Test](#speedtest)
9. [Learning Timeline](#timeline)

---

## **1. FOUNDATION KNOWLEDGE** {#foundation}

### **Understanding Block Devices**

#### **What Are Block Devices?**
Block devices are hardware or virtual devices that provide buffered access to data in fixed-size blocks. Think of them as containers that hold your data.

```bash
# List all block devices
lsblk

# Expected Output:
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0    20G  0 disk 
├─sda1   8:1    0    19G  0 part /
└─sda2   8:2    0     1G  0 part [SWAP]
sdb      8:16   0    10G  0 disk 
```

**Key Concepts:**
- **sda, sdb**: Physical disks (a=first disk, b=second disk)
- **sda1, sda2**: Partitions on disk sda
- **MAJ:MIN**: Major and minor device numbers
- **TYPE**: disk (physical), part (partition), lvm (logical volume)

#### **Device Naming Conventions**

```bash
# Traditional naming
/dev/sda    # First SATA/SCSI disk
/dev/sdb    # Second SATA/SCSI disk
/dev/hda    # IDE disk (legacy)

# NVMe naming
/dev/nvme0n1    # First NVMe disk
/dev/nvme0n1p1  # First partition on first NVMe

# Virtual machines
/dev/vda    # First virtio disk
/dev/xvda   # Xen virtual disk
```

### **Partition Tables: MBR vs GPT**

#### **MBR (Master Boot Record)**
```bash
# Check if disk uses MBR
sudo fdisk -l /dev/sdb | grep "Disklabel type"
# Output: Disklabel type: dos
```

**MBR Limitations:**
- Max 4 primary partitions
- Max 2TB disk size
- Legacy BIOS boot

#### **GPT (GUID Partition Table)**
```bash
# Check if disk uses GPT
sudo gdisk -l /dev/sdb | grep "GPT"
# Output: Partition table scan: GPT: present
```

**GPT Advantages:**
- 128 partitions (default)
- Supports disks >2TB
- UEFI boot compatible
- Backup partition table

### **Essential Pre-Partitioning Commands**

```bash
# 1. Identify available disks
lsblk -f    # Shows filesystem info

# 2. Check disk details
sudo hdparm -I /dev/sdb    # Detailed disk info

# 3. Verify disk is not in use
sudo lsof /dev/sdb    # Check open files

# 4. Backup partition table (CRITICAL!)
sudo sfdisk -d /dev/sda > partition_backup.txt

# 5. Wipe disk signatures (CAUTION!)
sudo wipefs -a /dev/sdb    # Remove all signatures
```

---

## **2. TOOL MASTERY** {#tools}

### **📦 FDISK - The Classic Tool**

#### **Basic fdisk Operations**

```bash
# Start fdisk on target disk
sudo fdisk /dev/sdb

# Inside fdisk interactive mode:
Command (m for help): m

# Common commands:
# n - new partition
# d - delete partition
# p - print partition table
# t - change partition type
# w - write changes
# q - quit without saving
```

#### **Complete fdisk Workflow Example**

```bash
#!/bin/bash
# fdisk_partition.sh - Automated partitioning script

DISK="/dev/sdb"

# Wipe existing partitions
echo "Wiping $DISK..."
sudo wipefs -a $DISK

# Create partitions using fdisk
echo "Creating partitions..."
(
echo n # New partition
echo p # Primary
echo 1 # Partition number
echo   # Default first sector
echo +2G # 2GB size
echo n # New partition
echo p # Primary
echo 2 # Partition number
echo   # Default first sector
echo +3G # 3GB size
echo n # New partition
echo p # Primary
echo 3 # Partition number
echo   # Default first sector
echo   # Use remaining space
echo w # Write changes
) | sudo fdisk $DISK

# Verify
lsblk $DISK
```

**Expected Output:**
```
sdb      8:16   0   10G  0 disk 
├─sdb1   8:17   0    2G  0 part 
├─sdb2   8:18   0    3G  0 part 
└─sdb3   8:19   0    5G  0 part
```

#### **Advanced fdisk Techniques**

```bash
# Change partition type to Linux LVM
echo -e "t\n1\n8e\nw" | sudo fdisk /dev/sdb

# Create extended partition for logical drives
echo -e "n\ne\n4\n\n\nw" | sudo fdisk /dev/sdb

# Set bootable flag
echo -e "a\n1\nw" | sudo fdisk /dev/sdb
```

### **📦 PARTED - The Modern Alternative**

#### **Why parted?**
- Supports both MBR and GPT
- Non-interactive mode for scripting
- Handles disks >2TB
- Immediate write (no separate save step)

#### **Basic parted Commands**

```bash
# View disk information
sudo parted /dev/sdb print

# Create GPT partition table
sudo parted /dev/sdb mklabel gpt

# Create partitions (non-interactive)
sudo parted /dev/sdb mkpart primary ext4 0% 20%
sudo parted /dev/sdb mkpart primary xfs 20% 50%
sudo parted /dev/sdb mkpart primary 50% 100%

# Set partition names
sudo parted /dev/sdb name 1 'System'
sudo parted /dev/sdb name 2 'Data'
sudo parted /dev/sdb name 3 'Backup'
```

#### **Complete parted Automation Script**

```bash
#!/bin/bash
# parted_automation.sh - Advanced partition creation

set -e  # Exit on error

DISK="/dev/sdb"
LOGFILE="/var/log/partition_$(date +%Y%m%d).log"

# Function: Log messages
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a $LOGFILE
}

# Function: Create GPT layout
create_gpt_layout() {
    local disk=$1
    
    log_message "Creating GPT on $disk"
    
    # Create GPT label
    sudo parted -s $disk mklabel gpt
    
    # Create partitions
    sudo parted -s $disk mkpart primary fat32 1MiB 512MiB
    sudo parted -s $disk set 1 esp on
    sudo parted -s $disk mkpart primary ext4 512MiB 4608MiB
    sudo parted -s $disk mkpart primary xfs 4608MiB 100%
    
    # Name partitions
    sudo parted -s $disk name 1 'EFI'
    sudo parted -s $disk name 2 'ROOT'
    sudo parted -s $disk name 3 'DATA'
    
    log_message "Partitions created successfully"
}

# Main execution
if [ ! -b "$DISK" ]; then
    log_message "ERROR: $DISK not found"
    exit 1
fi

create_gpt_layout $DISK

# Verify
sudo parted $DISK print
```

### **📦 GDISK - GPT Specialist**

#### **gdisk Advantages**
- GPT-focused (better GPT handling than fdisk)
- Backup/restore GPT tables
- Convert MBR to GPT
- Repair corrupted GPT

#### **Interactive gdisk Usage**

```bash
# Start gdisk
sudo gdisk /dev/sdb

# Commands:
# n - new partition
# d - delete partition
# p - print partition table
# i - show partition info
# v - verify disk
# w - write table to disk
# q - quit without saving
```

#### **gdisk Scripting Example**

```bash
#!/bin/bash
# gdisk_advanced.sh - GPT partition with specific types

DISK="/dev/sdb"

# Create GPT partitions with specific types
sudo sgdisk --zap-all $DISK  # Clear everything
sudo sgdisk --new=1:0:+1G --typecode=1:ef00 $DISK  # EFI partition
sudo sgdisk --new=2:0:+8G --typecode=2:8300 $DISK  # Linux filesystem
sudo sgdisk --new=3:0:0 --typecode=3:8e00 $DISK    # Linux LVM

# Set partition names
sudo sgdisk --change-name=1:EFI $DISK
sudo sgdisk --change-name=2:ROOT $DISK
sudo sgdisk --change-name=3:LVM $DISK

# Verify
sudo sgdisk --print $DISK
```

---

## **3. FILESYSTEM CREATION & MANAGEMENT** {#filesystems}

### **📂 Filesystem Types Overview**

| Filesystem | Use Case | Max File Size | Max Volume Size | Features |
|------------|----------|---------------|-----------------|----------|
| ext4 | General Linux | 16TB | 1EB | Journaling, extents |
| XFS | Large files/performance | 8EB | 8EB | High performance, scalable |
| Btrfs | Advanced features | 16EB | 16EB | Snapshots, compression |
| F2FS | Flash storage | 4TB | 16TB | Optimized for SSD/flash |
| VFAT/FAT32 | Cross-platform | 4GB | 2TB | Windows compatible |

### **Creating Filesystems**

#### **EXT4 - The Linux Standard**

```bash
#!/bin/bash
# ext4_creation.sh - Complete ext4 setup

PARTITION="/dev/sdb1"

# Basic ext4 creation
sudo mkfs.ext4 $PARTITION

# Advanced ext4 with options
sudo mkfs.ext4 -L "DataDrive" \
    -m 1 \                      # 1% reserved blocks
    -O dir_index,extent \       # Enable features
    -E stride=16,stripe_width=64 \  # RAID optimization
    $PARTITION

# Tune ext4 after creation
sudo tune2fs -c 0 -i 0 $PARTITION  # Disable fsck based on count/time
sudo tune2fs -o journal_data $PARTITION  # Full journaling

# Check filesystem
sudo e2fsck -f $PARTITION
```

#### **XFS - High Performance**

```bash
#!/bin/bash
# xfs_creation.sh - XFS filesystem setup

PARTITION="/dev/sdb2"

# Create XFS with custom parameters
sudo mkfs.xfs -f \
    -L "XFS_Data" \
    -b size=4096 \              # Block size
    -d agcount=4 \              # Allocation groups
    -l size=128m,lazy-count=1 \ # Log size
    $PARTITION

# XFS-specific operations
sudo xfs_info $PARTITION       # Show XFS info
sudo xfs_repair -n $PARTITION  # Check without repair
sudo xfs_admin -L "NewLabel" $PARTITION  # Change label
```

#### **Btrfs - Next Generation**

```bash
#!/bin/bash
# btrfs_advanced.sh - Btrfs with advanced features

DEVICE1="/dev/sdb3"
DEVICE2="/dev/sdc1"  # For RAID example

# Single device Btrfs
sudo mkfs.btrfs -L "BtrfsVolume" \
    -n 16384 \           # Node size
    -d single \          # Data profile
    -m single \          # Metadata profile
    $DEVICE1

# Btrfs with RAID1
sudo mkfs.btrfs -L "BtrfsRAID" \
    -d raid1 \
    -m raid1 \
    $DEVICE1 $DEVICE2

# Mount and create subvolumes
sudo mount $DEVICE1 /mnt
sudo btrfs subvolume create /mnt/@home
sudo btrfs subvolume create /mnt/@snapshots

# Create snapshot
sudo btrfs subvolume snapshot /mnt/@home /mnt/@snapshots/home_backup

# Enable compression
sudo mount -o compress=zstd $DEVICE1 /mnt
```

### **Filesystem Management Operations**

```bash
#!/bin/bash
# fs_management.sh - Common filesystem operations

# Function: Check filesystem usage
check_fs_usage() {
    local mount_point=$1
    df -h $mount_point
    df -i $mount_point  # Check inode usage
}

# Function: Resize filesystem
resize_filesystem() {
    local device=$1
    local fs_type=$(blkid -o value -s TYPE $device)
    
    case $fs_type in
        ext4)
            sudo e2fsck -f $device
            sudo resize2fs $device
            ;;
        xfs)
            # XFS can only grow, not shrink
            sudo xfs_growfs $device
            ;;
        btrfs)
            sudo btrfs filesystem resize max $device
            ;;
    esac
}

# Function: Filesystem maintenance
maintain_filesystem() {
    local device=$1
    local fs_type=$(blkid -o value -s TYPE $device)
    
    case $fs_type in
        ext4)
            sudo e2fsck -p $device  # Auto repair
            ;;
        xfs)
            sudo xfs_repair $device
            ;;
        btrfs)
            sudo btrfs check $device
            sudo btrfs balance start $device
            ;;
    esac
}
```

---

## **4. AUTOMATIC MOUNTING WITH /etc/fstab** {#fstab}

### **Understanding fstab Structure**

```bash
# fstab format:
# <device> <mount_point> <fs_type> <options> <dump> <pass>

# Example entry:
UUID=abc123 /data ext4 defaults,noatime 0 2
```

**Field Explanations:**
1. **Device**: UUID, LABEL, or device path
2. **Mount Point**: Where to mount
3. **Filesystem Type**: ext4, xfs, btrfs, etc.
4. **Options**: Mount options (comma-separated)
5. **Dump**: Backup utility flag (0=skip, 1=backup)
6. **Pass**: fsck order (0=skip, 1=root, 2=other)

### **Best Practices for fstab**

```bash
#!/bin/bash
# fstab_setup.sh - Safe fstab configuration

# Always use UUID instead of device names
get_uuid() {
    local device=$1
    blkid -o value -s UUID $device
}

# Function: Add fstab entry safely
add_fstab_entry() {
    local device=$1
    local mount_point=$2
    local fs_type=$3
    local options=${4:-"defaults"}
    
    # Get UUID
    local uuid=$(get_uuid $device)
    
    # Create mount point
    sudo mkdir -p $mount_point
    
    # Backup fstab
    sudo cp /etc/fstab /etc/fstab.backup.$(date +%Y%m%d)
    
    # Add entry
    echo "UUID=$uuid $mount_point $fs_type $options 0 2" | sudo tee -a /etc/fstab
    
    # Test mount
    sudo mount -a
    if [ $? -eq 0 ]; then
        echo "✓ Fstab entry added successfully"
    else
        echo "✗ Error in fstab! Restoring backup..."
        sudo cp /etc/fstab.backup.$(date +%Y%m%d) /etc/fstab
    fi
}

# Example usage
add_fstab_entry /dev/sdb1 /data ext4 "defaults,noatime,nodiratime"
```

### **Advanced fstab Options**

```bash
# Performance options
defaults,noatime,nodiratime  # Reduce disk writes
data=writeback               # Faster but less safe
commit=60                    # Delay writes (seconds)

# Security options
ro                          # Read-only
noexec                      # No execution
nosuid                      # No setuid
nodev                       # No device files

# Network filesystems
_netdev                     # Wait for network
soft,timeo=5,retrans=3      # NFS timeouts

# Special filesystems
tmpfs /tmp tmpfs size=2G,mode=1777 0 0  # RAM disk
```

### **Complete fstab Automation Script**

```bash
#!/bin/bash
# fstab_automation.sh - Complete disk setup with fstab

set -euo pipefail

# Configuration
DISK="/dev/sdb"
MOUNT_BASE="/mnt"

# Function: Setup partition with fstab
setup_partition() {
    local partition=$1
    local mount_point=$2
    local fs_type=$3
    local label=$4
    
    echo "Setting up $partition..."
    
    # Create filesystem
    case $fs_type in
        ext4)
            sudo mkfs.ext4 -L "$label" $partition
            ;;
        xfs)
            sudo mkfs.xfs -L "$label" $partition
            ;;
        btrfs)
            sudo mkfs.btrfs -L "$label" $partition
            ;;
    esac
    
    # Get UUID
    local uuid=$(blkid -o value -s UUID $partition)
    
    # Create mount point
    sudo mkdir -p $mount_point
    
    # Add to fstab
    echo "# $label partition" | sudo tee -a /etc/fstab
    echo "UUID=$uuid $mount_point $fs_type defaults,noatime 0 2" | sudo tee -a /etc/fstab
    
    # Mount
    sudo mount $mount_point
    
    # Set permissions
    sudo chmod 755 $mount_point
    
    echo "✓ $partition mounted at $mount_point"
}

# Main execution
echo "=== Disk Setup Automation ==="

# Create partitions
sudo parted -s $DISK mklabel gpt
sudo parted -s $DISK mkpart primary ext4 0% 33%
sudo parted -s $DISK mkpart primary xfs 33% 66%
sudo parted -s $DISK mkpart primary btrfs 66% 100%

# Setup each partition
setup_partition "${DISK}1" "$MOUNT_BASE/data1" "ext4" "DATA1"
setup_partition "${DISK}2" "$MOUNT_BASE/data2" "xfs" "DATA2"
setup_partition "${DISK}3" "$MOUNT_BASE/data3" "btrfs" "DATA3"

# Verify
echo -e "\n=== Verification ==="
mount | grep $DISK
df -h | grep $DISK
```

---

## **5. ADVANCED TECHNIQUES & OPTIMIZATION** {#advanced}

### **Performance Tuning**

```bash
#!/bin/bash
# performance_tuning.sh - Optimize disk performance

# Function: Optimize ext4 performance
optimize_ext4() {
    local device=$1
    
    # Enable writeback mode
    sudo tune2fs -o journal_data_writeback $device
    
    # Disable access time updates
    sudo tune2fs -O ^has_journal $device  # Remove journal (risky!)
    
    # Set reserved blocks to 0% for data partitions
    sudo tune2fs -m 0 $device
    
    # Enable directory indexing
    sudo tune2fs -O dir_index $device
}

# Function: Optimize XFS performance
optimize_xfs() {
    local mount_point=$1
    
    # Remount with optimizations
    sudo mount -o remount,nobarrier,noatime,nodiratime,logbufs=8 $mount_point
}

# Function: Set I/O scheduler
set_io_scheduler() {
    local device=$1
    local scheduler=$2  # noop, deadline, cfq, bfq
    
    echo $scheduler > /sys/block/$(basename $device)/queue/scheduler
    
    # Make persistent
    echo "echo $scheduler > /sys/block/$(basename $device)/queue/scheduler" >> /etc/rc.local
}

# Function: Optimize read-ahead
optimize_readahead() {
    local device=$1
    local kb=${2:-256}
    
    sudo blockdev --setra $((kb*2)) $device  # Convert KB to 512-byte sectors
}
```

### **RAID Configuration**

```bash
#!/bin/bash
# raid_setup.sh - Software RAID configuration

# Install mdadm
sudo apt-get install -y mdadm

# Create RAID 1 (mirror)
create_raid1() {
    local dev1=$1
    local dev2=$2
    local raid_device="/dev/md0"
    
    # Create RAID
    sudo mdadm --create $raid_device \
        --level=1 \
        --raid-devices=2 \
        $dev1 $dev2
    
    # Wait for sync
    cat /proc/mdstat
    
    # Create filesystem
    sudo mkfs.ext4 -L "RAID1" $raid_device
    
    # Save configuration
    sudo mdadm --detail --scan >> /etc/mdadm/mdadm.conf
    
    # Update initramfs
    sudo update-initramfs -u
}

# Create RAID 5 (striping with parity)
create_raid5() {
    local raid_device="/dev/md1"
    shift
    local devices="$@"
    
    sudo mdadm --create $raid_device \
        --level=5 \
        --raid-devices=$(echo $devices | wc -w) \
        $devices
}
```

### **LVM Integration**

```bash
#!/bin/bash
# lvm_setup.sh - LVM on top of partitions

# Physical Volume creation
sudo pvcreate /dev/sdb1 /dev/sdb2

# Volume Group creation
sudo vgcreate data_vg /dev/sdb1 /dev/sdb2

# Logical Volume creation
sudo lvcreate -L 5G -n lv_data data_vg
sudo lvcreate -l 100%FREE -n lv_backup data_vg

# Create filesystems
sudo mkfs.ext4 /dev/data_vg/lv_data
sudo mkfs.xfs /dev/data_vg/lv_backup

# Extend logical volume
sudo lvextend -L +2G /dev/data_vg/lv_data
sudo resize2fs /dev/data_vg/lv_data
```

### **Encryption with LUKS**

```bash
#!/bin/bash
# luks_encryption.sh - Encrypted partition setup

DEVICE="/dev/sdb1"
MAPPER_NAME="encrypted_data"

# Setup LUKS
setup_luks() {
    # Format with LUKS
    sudo cryptsetup luksFormat --type luks2 $DEVICE
    
    # Open encrypted device
    sudo cryptsetup open $DEVICE $MAPPER_NAME
    
    # Create filesystem on encrypted device
    sudo mkfs.ext4 /dev/mapper/$MAPPER_NAME
    
    # Mount
    sudo mkdir -p /mnt/encrypted
    sudo mount /dev/mapper/$MAPPER_NAME /mnt/encrypted
    
    # Auto-unlock with keyfile
    sudo dd if=/dev/urandom of=/root/keyfile bs=1024 count=4
    sudo chmod 0400 /root/keyfile
    sudo cryptsetup luksAddKey $DEVICE /root/keyfile
    
    # Add to crypttab
    echo "$MAPPER_NAME $DEVICE /root/keyfile luks" | sudo tee -a /etc/crypttab
}
```

---

## **6. ERROR HANDLING & RECOVERY** {#errors}

### **Common Errors and Solutions**

```bash
#!/bin/bash
# error_handling.sh - Comprehensive error handling

# Function: Safe partition operations
safe_partition() {
    local device=$1
    
    # Check if device exists
    if [ ! -b "$device" ]; then
        echo "ERROR: Device $device not found"
        return 1
    fi
    
    # Check if mounted
    if mount | grep -q "$device"; then
        echo "ERROR: Device $device is mounted"
        # Attempt to unmount
        sudo umount $device 2>/dev/null || {
            echo "Failed to unmount. Processes using device:"
            sudo lsof $device
            return 1
        }
    fi
    
    # Check for open processes
    if sudo lsof $device &>/dev/null; then
        echo "ERROR: Device $device is in use"
        return 1
    fi
    
    # Backup partition table
    sudo sfdisk -d $device > /tmp/partition_backup_$(basename $device).txt
    
    return 0
}

# Function: Recover from fstab errors
recover_fstab() {
    # Boot in emergency mode if fstab is broken
    # Add to kernel parameters: systemd.unit=emergency.target
    
    # Remount root as read-write
    sudo mount -o remount,rw /
    
    # Fix fstab
    sudo nano /etc/fstab
    
    # Or restore from backup
    if [ -f /etc/fstab.backup ]; then
        sudo cp /etc/fstab.backup /etc/fstab
    fi
    
    # Test
    sudo mount -a
}

# Function: Fix filesystem errors
fix_filesystem() {
    local device=$1
    local fs_type=$(blkid -o value -s TYPE $device)
    
    case $fs_type in
        ext4)
            # Force check
            sudo e2fsck -f $device
            # Automatic repair
            sudo e2fsck -p $device
            # Aggressive repair
            sudo e2fsck -y $device
            ;;
        xfs)
            # Check only
            sudo xfs_repair -n $device
            # Repair
            sudo xfs_repair $device
            # Force repair
            sudo xfs_repair -L $device  # Clears log
            ;;
        btrfs)
            # Check
            sudo btrfs check $device
            # Repair
            sudo btrfs check --repair $device
            # Scrub mounted filesystem
            sudo btrfs scrub start /mount/point
            ;;
    esac
}
```

### **Disaster Recovery Procedures**

```bash
#!/bin/bash
# disaster_recovery.sh - Recovery procedures

# Function: Backup partition table
backup_partition_table() {
    local device=$1
    local backup_dir="/var/backups/partitions"
    
    sudo mkdir -p $backup_dir
    
    # MBR backup
    sudo dd if=$device of=$backup_dir/mbr_$(basename $device).bin bs=512 count=1
    
    # Full partition table
    sudo sfdisk -d $device > $backup_dir/partition_$(basename $device).txt
    
    # GPT backup
    sudo sgdisk --backup=$backup_dir/gpt_$(basename $device).bin $device
}

# Function: Restore partition table
restore_partition_table() {
    local device=$1
    local backup_file=$2
    
    # Restore MBR
    if [[ $backup_file == *.bin ]] && [[ $backup_file == *mbr* ]]; then
        sudo dd if=$backup_file of=$device bs=512 count=1
    fi
    
    # Restore sfdisk backup
    if [[ $backup_file == *.txt ]]; then
        sudo sfdisk $device < $backup_file
    fi
    
    # Restore GPT
    if [[ $backup_file == *gpt*.bin ]]; then
        sudo sgdisk --load-backup=$backup_file $device
    fi
}

# Function: Emergency data recovery
emergency_recovery() {
    local failed_device=$1
    local recovery_dir="/mnt/recovery"
    
    sudo mkdir -p $recovery_dir
    
    # Try to mount read-only
    sudo mount -o ro $failed_device $recovery_dir 2>/dev/null || {
        echo "Normal mount failed, trying recovery tools..."
        
        # Use ddrescue for damaged disks
        sudo ddrescue -f -n $failed_device /tmp/recovery.img /tmp/recovery.log
        
        # Try TestDisk
        sudo testdisk $failed_device
        
        # PhotoRec for file recovery
        sudo photorec $failed_device
    }
}
```

---

## **7. PRACTICE LABS & CHALLENGES** {#practice}

### **Lab 1: Basic Partitioning**
**Time Limit: 5 minutes**

```bash
#!/bin/bash
# lab1_basic.sh - Basic partitioning challenge

echo "LAB 1: Create 3 equal partitions on /dev/sdb"
echo "Requirements:"
echo "- Use GPT partition table"
echo "- Each partition exactly 1/3 of disk"
echo "- Name them: BOOT, ROOT, DATA"
echo ""
echo "Start timer now!"

# Solution
solution_lab1() {
    local disk="/dev/sdb"
    
    # Get disk size in sectors
    local disk_size=$(sudo blockdev --getsz $disk)
    local part_size=$((disk_size / 3))
    
    # Clear and create GPT
    sudo sgdisk --zap-all $disk
    
    # Create partitions
    sudo sgdisk -n 1:0:+${part_size}s -c 1:"BOOT" $disk
    sudo sgdisk -n 2:0:+${part_size}s -c 2:"ROOT" $disk
    sudo sgdisk -n 3:0:0 -c 3:"DATA" $disk
    
    # Verify
    sudo sgdisk -p $disk
}
```

### **Lab 2: Filesystem Creation**
**Time Limit: 5 minutes**

```bash
#!/bin/bash
# lab2_filesystems.sh - Multiple filesystem challenge

echo "LAB 2: Create different filesystems"
echo "Requirements:"
echo "- sdb1: ext4 with label 'SYSTEM'"
echo "- sdb2: XFS with label 'DATA'"
echo "- sdb3: Btrfs with compression enabled"
echo ""

# Solution
solution_lab2() {
    # EXT4
    sudo mkfs.ext4 -L "SYSTEM" /dev/sdb1
    
    # XFS
    sudo mkfs.xfs -L "DATA" /dev/sdb2
    
    # Btrfs with compression
    sudo mkfs.btrfs -L "COMPRESSED" /dev/sdb3
    sudo mount -o compress=zstd /dev/sdb3 /mnt/compressed
}
```

### **Lab 3: Complex fstab Configuration**
**Time Limit: 5 minutes**

```bash
#!/bin/bash
# lab3_fstab.sh - fstab configuration challenge

echo "LAB 3: Configure automatic mounting"
echo "Requirements:"
echo "- Mount sdb1 at /data with noatime"
echo "- Mount sdb2 at /backup read-only"
echo "- Mount tmpfs at /tmp with 2G size limit"
echo "- All using UUID in fstab"

# Solution
solution_lab3() {
    # Get UUIDs
    UUID1=$(blkid -o value -s UUID /dev/sdb1)
    UUID2=$(blkid -o value -s UUID /dev/sdb2)
    
    # Create mount points
    sudo mkdir -p /data /backup
    
    # Add to fstab
    cat << EOF | sudo tee -a /etc/fstab
UUID=$UUID1 /data ext4 defaults,noatime 0 2
UUID=$UUID2 /backup xfs ro 0 2
tmpfs /tmp tmpfs size=2G,mode=1777 0 0
EOF
    
    # Mount all
    sudo mount -a
}
```

### **Lab 4: RAID + LVM + Encryption**
**Time Limit: 10 minutes**

```bash
#!/bin/bash
# lab4_advanced.sh - Advanced storage stack

echo "LAB 4: Build complete storage stack"
echo "Requirements:"
echo "- Create RAID 1 with sdb and sdc"
echo "- Setup LVM on RAID device"
echo "- Create encrypted logical volume"
echo "- Mount with specific options"

# Solution
solution_lab4() {
    # Create RAID 1
    sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
    
    # Setup LVM
    sudo pvcreate /dev/md0
    sudo vgcreate storage_vg /dev/md0
    sudo lvcreate -L 2G -n secure_lv storage_vg
    
    # Encrypt
    echo "password123" | sudo cryptsetup luksFormat /dev/storage_vg/secure_lv -
    echo "password123" | sudo cryptsetup open /dev/storage_vg/secure_lv secure_data -
    
    # Create filesystem and mount
    sudo mkfs.ext4 /dev/mapper/secure_data
    sudo mkdir -p /secure
    sudo mount -o noexec,nosuid /dev/mapper/secure_data /secure
}
```

---

## **8. 15-MINUTE SPEED TEST** {#speedtest}

### **The Ultimate Challenge**

```bash
#!/bin/bash
# speed_test.sh - 15-minute complex disk setup

# CHALLENGE REQUIREMENTS:
# ========================
# 1. Clear existing partitions on /dev/sdb
# 2. Create GPT partition scheme:
#    - 512MB EFI partition (FAT32)
#    - 4GB root partition (ext4)
#    - 2GB swap partition
#    - Remaining space for LVM
# 3. Setup LVM with:
#    - Volume group named "data_vg"
#    - 3 logical volumes: projects(2G), docker(3G), backups(remaining)
# 4. Create appropriate filesystems
# 5. Configure /etc/fstab for all partitions
# 6. Mount everything and verify
# 7. Create RAID 1 on sdc and sdd for /backup2
# 8. Setup quota on projects volume
# 9. Create automated backup script

# Timer function
start_timer() {
    START_TIME=$(date +%s)
    echo "Timer started at $(date)"
}

end_timer() {
    END_TIME=$(date +%s)
    ELAPSED=$((END_TIME - START_TIME))
    echo "Time taken: $((ELAPSED / 60)) minutes $((ELAPSED % 60)) seconds"
}

# Complete solution
complete_speed_test() {
    start_timer
    
    # 1. Clear disk
    sudo wipefs -a /dev/sdb
    
    # 2. Create GPT partitions
    sudo parted -s /dev/sdb mklabel gpt
    sudo parted -s /dev/sdb mkpart ESP fat32 1MiB 513MiB
    sudo parted -s /dev/sdb set 1 esp on
    sudo parted -s /dev/sdb mkpart root ext4 513MiB 4609MiB
    sudo parted -s /dev/sdb mkpart swap linux-swap 4609MiB 6657MiB
    sudo parted -s /dev/sdb mkpart lvm 6657MiB 100%
    sudo parted -s /dev/sdb set 4 lvm on
    
    # 3. Create filesystems
    sudo mkfs.fat -F32 /dev/sdb1
    sudo mkfs.ext4 -L "ROOT" /dev/sdb2
    sudo mkswap -L "SWAP" /dev/sdb3
    
    # 4. Setup LVM
    sudo pvcreate /dev/sdb4
    sudo vgcreate data_vg /dev/sdb4
    sudo lvcreate -L 2G -n projects data_vg
    sudo lvcreate -L 3G -n docker data_vg
    sudo lvcreate -l 100%FREE -n backups data_vg
    
    # 5. Create filesystems on LVM
    sudo mkfs.ext4 -L "PROJECTS" /dev/data_vg/projects
    sudo mkfs.ext4 -L "DOCKER" /dev/data_vg/docker
    sudo mkfs.xfs -L "BACKUPS" /dev/data_vg/backups
    
    # 6. Create mount points
    sudo mkdir -p /boot/efi /mnt/{root,projects,docker,backups}
    
    # 7. Setup fstab
    cat << 'EOF' | sudo tee /tmp/fstab_entries
# Speed test entries
UUID=$(blkid -o value -s UUID /dev/sdb1) /boot/efi vfat umask=0077 0 1
UUID=$(blkid -o value -s UUID /dev/sdb2) /mnt/root ext4 defaults 0 2
UUID=$(blkid -o value -s UUID /dev/sdb3) none swap sw 0 0
UUID=$(blkid -o value -s UUID /dev/data_vg/projects) /mnt/projects ext4 defaults,usrquota,grpquota 0 2
UUID=$(blkid -o value -s UUID /dev/data_vg/docker) /mnt/docker ext4 defaults 0 2
UUID=$(blkid -o value -s UUID /dev/data_vg/backups) /mnt/backups xfs defaults 0 2
EOF
    
    # Process and add to fstab
    while IFS= read -r line; do
        if [[ $line == UUID=* ]]; then
            eval "echo \"$line\"" | sudo tee -a /etc/fstab
        else
            echo "$line" | sudo tee -a /etc/fstab
        fi
    done < /tmp/fstab_entries
    
    # 8. Mount all
    sudo swapon /dev/sdb3
    sudo mount -a
    
    # 9. Setup RAID 1 (if sdc and sdd exist)
    if [ -b /dev/sdc ] && [ -b /dev/sdd ]; then
        sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdc /dev/sdd --assume-clean
        sudo mkfs.ext4 -L "RAID_BACKUP" /dev/md0
        sudo mkdir -p /backup2
        echo "/dev/md0 /backup2 ext4 defaults 0 2" | sudo tee -a /etc/fstab
        sudo mount /backup2
    fi
    
    # 10. Setup quotas
    sudo quotacheck -cug /mnt/projects
    sudo quotaon /mnt/projects
    
    # 11. Create backup script
    cat << 'BACKUP_SCRIPT' | sudo tee /usr/local/bin/backup.sh
#!/bin/bash
rsync -av --delete /mnt/projects/ /mnt/backups/projects/
rsync -av --delete /mnt/docker/ /mnt/backups/docker/
echo "Backup completed at $(date)" >> /var/log/backup.log
BACKUP_SCRIPT
    
    sudo chmod +x /usr/local/bin/backup.sh
    
    # 12. Verify everything
    echo "=== VERIFICATION ==="
    lsblk /dev/sdb
    df -h | grep -E "sdb|data_vg"
    swapon --show
    cat /proc/mdstat
    
    end_timer
}

# Run the test
complete_speed_test
```

---

## **9. LEARNING TIMELINE** {#timeline}

### **Week 1: Foundation (Days 1-7)**

**Day 1-2: Block Devices & Concepts**
```bash
# Practice commands
lsblk -f
fdisk -l
blkid
df -h
# Goal: Understand device naming, identify all disks
```

**Day 3-4: Basic fdisk Operations**
```bash
# Create/delete partitions
# Practice with loop devices for safety
dd if=/dev/zero of=disk.img bs=1G count=10
sudo losetup /dev/loop0 disk.img
sudo fdisk /dev/loop0
```

**Day 5-7: Filesystem Basics**
```bash
# Create ext4, xfs, btrfs
# Mount/unmount operations
# Basic fstab entries
```

### **Week 2: Intermediate (Days 8-14)**

**Day 8-9: parted Mastery**
```bash
# GPT partitioning
# Scripted partitioning
# Partition alignment
```

**Day 10-11: Advanced Filesystems**
```bash
# Filesystem features
# Tuning parameters
# Compression, quotas
```

**Day 12-14: fstab Automation**
```bash
# UUID usage
# Mount options optimization
# Network filesystems
```

### **Week 3: Advanced (Days 15-21)**

**Day 15-16: LVM Operations**
```bash
# Physical volumes, volume groups
# Logical volume management
# Snapshots
```

**Day 17-18: RAID Configuration**
```bash
# Software RAID levels
# mdadm operations
# RAID monitoring
```

**Day 19-21: Integration**
```bash
# RAID + LVM + Encryption
# Performance optimization
# Disaster recovery
```

### **Week 4: Mastery (Days 22-30)**

**Day 22-24: Speed Optimization**
```bash
# Practice speed tests
# Memorize command sequences
# Build muscle memory
```

**Day 25-27: Real-world Scenarios**
```bash
# DevOps automation scripts
# CI/CD storage setup
# Container storage
```

**Day 28-30: Final Assessment**
```bash
# Complete 15-minute challenge
# Debug complex issues
# Performance tuning
```

### **Daily Practice Routine**

```bash
#!/bin/bash
# daily_practice.sh - 30-minute daily practice

echo "=== Daily Disk Management Practice ==="
echo "Start time: $(date)"

# Warm-up (5 min)
echo "1. System check..."
lsblk
df -h
cat /proc/mdstat

# Skill practice (15 min)
echo "2. Today's focus..."
# Rotate through: fdisk, parted, gdisk, filesystem creation, fstab editing

# Challenge (10 min)
echo "3. Mini challenge..."
# Random scenario from challenge bank

echo "End time: $(date)"
echo "Log your progress!"
```

### **Progress Tracking**

```bash
#!/bin/bash
# track_progress.sh - Track learning progress

PROGRESS_FILE="$HOME/.disk_management_progress"

log_progress() {
    local skill=$1
    local level=$2  # 1-5
    local notes=$3
    
    echo "$(date),${skill},${level},${notes}" >> $PROGRESS_FILE
}

show_progress() {
    echo "=== Your Progress ==="
    echo "Skill,Latest Level,Last Practice"
    
    for skill in fdisk parted gdisk filesystems fstab lvm raid; do
        grep $skill $PROGRESS_FILE | tail -1
    done
}

# Usage
log_progress "fdisk" "4" "Completed advanced partitioning"
show_progress
```

## **🎯 FINAL MASTERY CHECKLIST**

- [ ] Can identify any disk/partition in under 5 seconds
- [ ] Can create GPT/MBR partitions with fdisk/parted/gdisk
- [ ] Can create and optimize ext4/XFS/Btrfs filesystems
- [ ] Can write complex fstab entries from memory
- [ ] Can setup LVM with multiple logical volumes
- [ ] Can create and manage software RAID
- [ ] Can encrypt partitions with LUKS
- [ ] Can recover from partition table corruption
- [ ] Can complete 15-minute speed test successfully
- [ ] Can automate entire disk setup via scripts

**Remember:** Practice with virtual machines or loop devices first. Never practice on production systems. Always have backups before any disk operation!