# System Monitor Dashboard - Complete Mastery Guide

## Table of Contents
1. [Foundation & Prerequisites](#foundation)
2. [Core Monitoring Components](#core-components)
3. [Building the Dashboard](#building-dashboard)
4. [Alerting System](#alerting)
5. [Service Auto-Recovery](#service-recovery)
6. [Cross-Distribution Compatibility](#compatibility)
7. [Cheat Sheets](#cheat-sheets)
8. [Hands-On Exercises](#exercises)

---

## 1. Foundation & Prerequisites {#foundation}

### Why System Monitoring?
**Real-world scenario**: Production server crashes at 3 AM. No one noticed CPU spiked to 100% for 2 hours before failure. Cost: $50K in lost revenue.

**Solution**: Automated monitoring catches issues before they become disasters.

### Essential Concepts

```
┌─────────────────────────────────────────────────────┐
│           SYSTEM MONITORING PYRAMID                 │
├─────────────────────────────────────────────────────┤
│  Level 4: Automation (Auto-restart services)        │
│  Level 3: Alerting (Notify when thresholds hit)     │
│  Level 2: Logging (Historical data)                 │
│  Level 1: Real-time Display (Current metrics)       │
│  Level 0: Data Collection (Read system stats)       │
└─────────────────────────────────────────────────────┘
```

### Installation

```bash
# Update system
sudo apt update

# Core monitoring tools
sudo apt install -y \
  sysstat \        # sar, iostat, mpstat
  procps \         # ps, top, free, vmstat
  net-tools \      # netstat (legacy)
  iproute2 \       # ss, ip (modern)
  bc \             # Calculator for percentage calculations
  jq \             # JSON processing for structured logs
  curl \           # For webhook alerts
  mailutils        # For email alerts

# Optional but recommended
sudo apt install -y \
  htop \           # Better than top
  iotop \          # I/O monitoring
  ncdu \           # Disk usage analyzer
  glances          # All-in-one monitoring
```

**Breakdown**:
- `sysstat`: System performance tools
- `procps`: Process utilities
- `bc`: Floating-point math (CPU percentages)
- `jq`: Parse JSON logs
- `mailutils`: Send email alerts

---

## 2. Core Monitoring Components {#core-components}

### CPU Monitoring

#### Method 1: /proc/stat (Most Portable)

```bash
#!/bin/bash
# File: get_cpu_usage.sh

get_cpu_usage() {
    # Read /proc/stat twice with 1-second interval
    read cpu user1 nice1 system1 idle1 iowait1 irq1 softirq1 steal1 <<< $(grep '^cpu ' /proc/stat)
    sleep 1
    read cpu user2 nice2 system2 idle2 iowait2 irq2 softirq2 steal2 <<< $(grep '^cpu ' /proc/stat)
    
    # Calculate deltas
    user=$((user2 - user1))
    nice=$((nice2 - nice1))
    system=$((system2 - system1))
    idle=$((idle2 - idle1))
    iowait=$((iowait2 - iowait1))
    irq=$((irq2 - irq1))
    softirq=$((softirq2 - softirq1))
    steal=$((steal2 - steal1))
    
    # Total time
    total=$((user + nice + system + idle + iowait + irq + softirq + steal))
    
    # CPU usage percentage
    usage=$((100 * (total - idle) / total))
    
    echo "$usage"
}

# Test
CPU_USAGE=$(get_cpu_usage)
echo "CPU Usage: ${CPU_USAGE}%"
```

**Explanation**:
- `/proc/stat` shows cumulative CPU time in jiffies (clock ticks)
- First line: `cpu <user> <nice> <system> <idle> <iowait> <irq> <softirq> <steal>`
- Take two readings 1 second apart
- Calculate difference (delta)
- `usage = 100 * (total - idle) / total`

**Expected Output**:
```
CPU Usage: 23%
```

---

#### Method 2: top/mpstat (More Features)

```bash
#!/bin/bash
# File: cpu_advanced.sh

# Using top (1 iteration, batch mode)
cpu_top() {
    top -bn1 | grep "Cpu(s)" | awk '{print 100 - $8}' | cut -d. -f1
}

# Using mpstat (requires sysstat)
cpu_mpstat() {
    mpstat 1 1 | awk '/Average/ {print 100 - $NF}' | cut -d. -f1
}

# Per-core usage
cpu_per_core() {
    mpstat -P ALL 1 1 | awk '/Average/ && $2 ~ /[0-9]/ {printf "CPU%s: %.0f%% ", $2, 100-$NF}'
    echo
}

echo "Method 1 (top): $(cpu_top)%"
echo "Method 2 (mpstat): $(cpu_mpstat)%"
echo "Per-core: $(cpu_per_core)"
```

**Breakdown**:
- `top -bn1`: Batch mode, 1 iteration
- `grep "Cpu(s)"`: Extract CPU line
- `awk '{print 100 - $8}'`: $8 is idle%, so 100 - idle = used
- `mpstat -P ALL`: Show all CPUs individually

---

### Memory Monitoring

```bash
#!/bin/bash
# File: get_memory.sh

get_memory_usage() {
    # Method 1: Using free (most common)
    free -m | awk 'NR==2 {printf "Used: %dMB (%.0f%%), Free: %dMB, Total: %dMB\n", $3, $3*100/$2, $4, $2}'
    
    # Method 2: Using /proc/meminfo (most accurate)
    local mem_total=$(awk '/MemTotal/ {print $2}' /proc/meminfo)
    local mem_available=$(awk '/MemAvailable/ {print $2}' /proc/meminfo)
    local mem_used=$((mem_total - mem_available))
    local mem_percent=$((100 * mem_used / mem_total))
    
    echo "Memory: ${mem_percent}% (${mem_used}KB used / ${mem_total}KB total)"
    
    # Swap usage
    local swap_total=$(awk '/SwapTotal/ {print $2}' /proc/meminfo)
    local swap_free=$(awk '/SwapFree/ {print $2}' /proc/meminfo)
    local swap_used=$((swap_total - swap_free))
    
    if [ $swap_total -gt 0 ]; then
        local swap_percent=$((100 * swap_used / swap_total))
        echo "Swap: ${swap_percent}% (${swap_used}KB used / ${swap_total}KB total)"
    fi
}

get_memory_usage
```

**Why MemAvailable instead of MemFree?**
- `MemFree`: Actually unused memory
- `MemAvailable`: Memory available for applications (includes cache that can be freed)
- `MemAvailable` is more accurate for "usable" memory

**Expected Output**:
```
Used: 3847MB (48%), Free: 1234MB, Total: 8096MB
Memory: 48% (3944532KB used / 8290816KB total)
Swap: 5% (51200KB used / 1048576KB total)
```

---

### Disk Monitoring

```bash
#!/bin/bash
# File: get_disk.sh

get_disk_usage() {
    echo "=== Disk Usage by Filesystem ==="
    df -h | awk 'NR==1 || /^\/dev\// {printf "%-20s %5s %5s %5s %4s %s\n", $1, $2, $3, $4, $5, $6}'
    
    echo -e "\n=== Disk I/O Stats ==="
    iostat -dx 1 2 | awk '/^[a-z]/ && NR>3 {printf "%-10s r/s: %5.1f w/s: %5.1f util: %5.1f%%\n", $1, $4, $5, $NF}'
    
    echo -e "\n=== Inode Usage ==="
    df -i / | awk 'NR==2 {printf "Inodes: %s used (%.0f%%)\n", $3, $3*100/$2}'
}

# Check specific partition threshold
check_disk_threshold() {
    local partition=${1:-/}
    local threshold=${2:-80}
    
    local usage=$(df "$partition" | awk 'NR==2 {print $5}' | sed 's/%//')
    
    if [ "$usage" -gt "$threshold" ]; then
        echo "ALERT: $partition is ${usage}% full (threshold: ${threshold}%)"
        return 1
    else
        echo "OK: $partition is ${usage}% full"
        return 0
    fi
}

get_disk_usage
check_disk_threshold "/" 80
```

**Breakdown**:
- `df -h`: Human-readable disk usage
- `/^\/dev\//`: Filter only physical devices
- `iostat -dx 1 2`: Disk I/O, 2 samples 1 second apart (first is discarded)
- `df -i`: Inode usage (file count limit)

**Expected Output**:
```
=== Disk Usage by Filesystem ===
Filesystem             Size  Used Avail  Use Mounted
/dev/sda1              50G   30G   18G   63% /
/dev/sdb1             100G   45G   50G   47% /data

=== Disk I/O Stats ===
sda         r/s:  12.3 w/s:  45.6 util:  15.2%
sdb         r/s:   5.1 w/s:  23.4 util:   8.5%

=== Inode Usage ===
Inodes: 345678 used (12%)

OK: / is 63% full
```

---

### Network Monitoring

```bash
#!/bin/bash
# File: get_network.sh

get_network_usage() {
    # Method 1: Using /proc/net/dev
    local interface=${1:-eth0}
    
    # First reading
    read rx1 tx1 <<< $(awk -v iface="$interface:" '$1 == iface {print $2, $10}' /proc/net/dev)
    sleep 1
    # Second reading
    read rx2 tx2 <<< $(awk -v iface="$interface:" '$1 == iface {print $2, $10}' /proc/net/dev)
    
    # Calculate bytes per second
    local rx_bytes=$((rx2 - rx1))
    local tx_bytes=$((tx2 - tx1))
    
    # Convert to human-readable
    local rx_mbps=$(echo "scale=2; $rx_bytes * 8 / 1000000" | bc)
    local tx_mbps=$(echo "scale=2; $tx_bytes * 8 / 1000000" | bc)
    
    echo "Interface: $interface"
    echo "  RX: ${rx_mbps} Mbps (${rx_bytes} bytes/sec)"
    echo "  TX: ${tx_mbps} Mbps (${tx_bytes} bytes/sec)"
}

# Get all active interfaces
get_all_interfaces() {
    ip -o link show | awk -F': ' '{print $2}' | grep -v "lo"
}

# Network connections count
get_connection_stats() {
    echo "=== Connection Statistics ==="
    ss -s
    
    echo -e "\n=== Established Connections ==="
    ss -tan | awk '$1 == "ESTAB" {print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn | head -10
}

# Auto-detect primary interface
PRIMARY_IF=$(ip route | awk '/default/ {print $5; exit}')
echo "Primary interface: $PRIMARY_IF"
get_network_usage "$PRIMARY_IF"
get_connection_stats
```

**Explanation**:
- `/proc/net/dev`: Network interface statistics
- Columns: Interface, RX bytes, RX packets, ... TX bytes, TX packets
- `$2` = RX bytes, `$10` = TX bytes
- Calculate delta over 1 second = bytes/second
- Multiply by 8 for bits, divide by 1,000,000 for Mbps

**Expected Output**:
```
Primary interface: eth0
Interface: eth0
  RX: 5.23 Mbps (654321 bytes/sec)
  TX: 2.15 Mbps (268750 bytes/sec)

=== Connection Statistics ===
Total: 234 (kernel 256)
TCP:   123 (estab 45, closed 12, orphaned 0, synrecv 0, timewait 12/0)

=== Established Connections ===
     15 192.168.1.100
     10 10.0.0.5
      8 172.16.0.20
```

---

## 3. Building the Dashboard {#building-dashboard}

### Complete Monitoring Dashboard Script

```bash
#!/bin/bash
# File: system_monitor.sh
# Purpose: Real-time system monitoring dashboard
# Author: DevOps Team
# Version: 2.0

#=============================================================================
# CONFIGURATION
#=============================================================================

# Thresholds
CPU_THRESHOLD=80
MEMORY_THRESHOLD=85
DISK_THRESHOLD=90
NETWORK_THRESHOLD=100  # Mbps

# Logging
LOG_DIR="/var/log/system_monitor"
METRICS_LOG="$LOG_DIR/metrics.log"
ALERT_LOG="$LOG_DIR/alerts.log"

# Alert settings
ALERT_EMAIL="admin@example.com"
WEBHOOK_URL="https://hooks.slack.com/services/YOUR/WEBHOOK/URL"

# Services to monitor
SERVICES=("nginx" "mysql" "redis")

# Refresh interval (seconds)
REFRESH_INTERVAL=2

# Colors for terminal output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color
BOLD='\033[1m'

#=============================================================================
# INITIALIZATION
#=============================================================================

# Create log directory
mkdir -p "$LOG_DIR"

# Trap Ctrl+C for clean exit
trap cleanup INT TERM

cleanup() {
    echo -e "\n${YELLOW}Monitoring stopped.${NC}"
    tput cnorm  # Show cursor
    exit 0
}

#=============================================================================
# UTILITY FUNCTIONS
#=============================================================================

log_metric() {
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "$timestamp | $1" | tee -a "$METRICS_LOG"
}

log_alert() {
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    local message="$timestamp | ALERT | $1"
    echo "$message" | tee -a "$ALERT_LOG"
}

send_email_alert() {
    local subject="$1"
    local body="$2"
    
    if command -v mail &>/dev/null; then
        echo "$body" | mail -s "$subject" "$ALERT_EMAIL"
    fi
}

send_webhook_alert() {
    local message="$1"
    
    if [ -n "$WEBHOOK_URL" ]; then
        curl -X POST -H 'Content-type: application/json' \
            --data "{\"text\":\"$message\"}" \
            "$WEBHOOK_URL" &>/dev/null
    fi
}

get_timestamp() {
    date '+%Y-%m-%d %H:%M:%S'
}

#=============================================================================
# MONITORING FUNCTIONS
#=============================================================================

get_cpu_usage() {
    # Read /proc/stat twice
    read cpu user1 nice1 system1 idle1 iowait1 irq1 softirq1 steal1 <<< $(grep '^cpu ' /proc/stat)
    sleep 0.5
    read cpu user2 nice2 system2 idle2 iowait2 irq2 softirq2 steal2 <<< $(grep '^cpu ' /proc/stat)
    
    # Calculate deltas
    local user=$((user2 - user1))
    local nice=$((nice2 - nice1))
    local system=$((system2 - system1))
    local idle=$((idle2 - idle1))
    local iowait=$((iowait2 - iowait1))
    local total=$((user + nice + system + idle + iowait + irq + softirq + steal))
    
    # Calculate percentages
    CPU_USAGE=$((100 * (total - idle) / total))
    CPU_USER=$((100 * user / total))
    CPU_SYSTEM=$((100 * system / total))
    CPU_IOWAIT=$((100 * iowait / total))
}

get_memory_usage() {
    local mem_total=$(awk '/MemTotal/ {print $2}' /proc/meminfo)
    local mem_available=$(awk '/MemAvailable/ {print $2}' /proc/meminfo)
    local mem_used=$((mem_total - mem_available))
    
    MEM_TOTAL_MB=$((mem_total / 1024))
    MEM_USED_MB=$((mem_used / 1024))
    MEM_USAGE=$((100 * mem_used / mem_total))
    
    # Swap
    local swap_total=$(awk '/SwapTotal/ {print $2}' /proc/meminfo)
    local swap_free=$(awk '/SwapFree/ {print $2}' /proc/meminfo)
    
    if [ "$swap_total" -gt 0 ]; then
        SWAP_USED=$((swap_total - swap_free))
        SWAP_USAGE=$((100 * SWAP_USED / swap_total))
    else
        SWAP_USAGE=0
    fi
}

get_disk_usage() {
    # Get root partition usage
    DISK_USAGE=$(df / | awk 'NR==2 {print $5}' | sed 's/%//')
    DISK_USED=$(df -h / | awk 'NR==2 {print $3}')
    DISK_TOTAL=$(df -h / | awk 'NR==2 {print $2}')
    
    # Get highest partition usage
    MAX_DISK_USAGE=$(df | awk 'NR>1 && /^\/dev\// {print $5}' | sed 's/%//' | sort -rn | head -1)
}

get_network_usage() {
    local interface=$(ip route | awk '/default/ {print $5; exit}')
    NETWORK_INTERFACE="$interface"
    
    if [ -z "$interface" ]; then
        NETWORK_RX_MBPS=0
        NETWORK_TX_MBPS=0
        return
    fi
    
    # First reading
    read rx1 tx1 <<< $(awk -v iface="$interface:" '$1 == iface {print $2, $10}' /proc/net/dev)
    sleep 0.5
    # Second reading
    read rx2 tx2 <<< $(awk -v iface="$interface:" '$1 == iface {print $2, $10}' /proc/net/dev)
    
    local rx_bytes=$(( (rx2 - rx1) * 2 ))  # *2 because we sleep 0.5s
    local tx_bytes=$(( (tx2 - tx1) * 2 ))
    
    NETWORK_RX_MBPS=$(echo "scale=2; $rx_bytes * 8 / 1000000" | bc)
    NETWORK_TX_MBPS=$(echo "scale=2; $tx_bytes * 8 / 1000000" | bc)
}

get_load_average() {
    read LOAD_1MIN LOAD_5MIN LOAD_15MIN <<< $(uptime | awk -F'load average:' '{print $2}' | tr -d ' ' | tr ',' ' ')
}

get_process_count() {
    PROCESS_COUNT=$(ps aux | wc -l)
    PROCESS_RUNNING=$(ps aux | awk '$8 == "R" {count++} END {print count+0}')
}

check_service_status() {
    local service=$1
    
    # Try systemctl first (systemd)
    if command -v systemctl &>/dev/null; then
        systemctl is-active --quiet "$service"
        return $?
    # Try service command (init.d)
    elif command -v service &>/dev/null; then
        service "$service" status &>/dev/null
        return $?
    # Try direct init script
    elif [ -f "/etc/init.d/$service" ]; then
        /etc/init.d/"$service" status &>/dev/null
        return $?
    fi
    
    return 1
}

#=============================================================================
# ALERT FUNCTIONS
#=============================================================================

check_thresholds() {
    local alerts=()
    
    # CPU alert
    if [ "$CPU_USAGE" -gt "$CPU_THRESHOLD" ]; then
        local msg="CPU usage ${CPU_USAGE}% exceeds threshold ${CPU_THRESHOLD}%"
        alerts+=("$msg")
        log_alert "$msg"
    fi
    
    # Memory alert
    if [ "$MEM_USAGE" -gt "$MEMORY_THRESHOLD" ]; then
        local msg="Memory usage ${MEM_USAGE}% exceeds threshold ${MEMORY_THRESHOLD}%"
        alerts+=("$msg")
        log_alert "$msg"
    fi
    
    # Disk alert
    if [ "$DISK_USAGE" -gt "$DISK_THRESHOLD" ]; then
        local msg="Disk usage ${DISK_USAGE}% exceeds threshold ${DISK_THRESHOLD}%"
        alerts+=("$msg")
        log_alert "$msg"
    fi
    
    # Send consolidated alert if any
    if [ ${#alerts[@]} -gt 0 ]; then
        local alert_message="System Alert on $(hostname):\n$(printf '%s\n' "${alerts[@]}")"
        send_webhook_alert "$alert_message"
        # Uncomment to enable email alerts
        # send_email_alert "System Alert" "$alert_message"
    fi
}

#=============================================================================
# SERVICE AUTO-RECOVERY
#=============================================================================

monitor_services() {
    for service in "${SERVICES[@]}"; do
        if ! check_service_status "$service"; then
            log_alert "Service $service is down - attempting restart"
            restart_service "$service"
        fi
    done
}

restart_service() {
    local service=$1
    
    if command -v systemctl &>/dev/null; then
        sudo systemctl restart "$service"
    elif command -v service &>/dev/null; then
        sudo service "$service" restart
    elif [ -f "/etc/init.d/$service" ]; then
        sudo /etc/init.d/"$service" restart
    fi
    
    sleep 2
    
    if check_service_status "$service"; then
        log_alert "Service $service restarted successfully"
        send_webhook_alert "✅ Service $service restarted on $(hostname)"
    else
        log_alert "CRITICAL: Failed to restart service $service"
        send_webhook_alert "❌ CRITICAL: Failed to restart $service on $(hostname)"
    fi
}

#=============================================================================
# DISPLAY FUNCTIONS
#=============================================================================

draw_bar() {
    local percentage=$1
    local width=50
    local filled=$((percentage * width / 100))
    local empty=$((width - filled))
    
    # Color based on percentage
    local color=$GREEN
    if [ "$percentage" -gt 80 ]; then
        color=$RED
    elif [ "$percentage" -gt 60 ]; then
        color=$YELLOW
    fi
    
    printf "${color}"
    printf '█%.0s' $(seq 1 $filled)
    printf "${NC}"
    printf '░%.0s' $(seq 1 $empty)
    printf " %3d%%" "$percentage"
}

display_dashboard() {
    # Clear screen and hide cursor
    clear
    tput civis
    
    # Header
    echo -e "${BOLD}${BLUE}"
    echo "╔═══════════════════════════════════════════════════════════════════════════╗"
    echo "║                    SYSTEM MONITORING DASHBOARD                            ║"
    echo "╚═══════════════════════════════════════════════════════════════════════════╝"
    echo -e "${NC}"
    
    echo -e "${BOLD}Server:${NC} $(hostname) | ${BOLD}Time:${NC} $(get_timestamp) | ${BOLD}Uptime:${NC} $(uptime -p)"
    echo ""
    
    # CPU Section
    echo -e "${BOLD}${BLUE}┌─ CPU ────────────────────────────────────────────────────────────────────┐${NC}"
    echo -n "  Total:  "
    draw_bar "$CPU_USAGE"
    echo ""
    echo "  User: ${CPU_USER}% | System: ${CPU_SYSTEM}% | IOWait: ${CPU_IOWAIT}%"
    echo "  Load Average: $LOAD_1MIN (1m) | $LOAD_5MIN (5m) | $LOAD_15MIN (15m)"
    echo -e "${BLUE}└──────────────────────────────────────────────────────────────────────────┘${NC}"
    echo ""
    
    # Memory Section
    echo -e "${BOLD}${BLUE}┌─ MEMORY ─────────────────────────────────────────────────────────────────┐${NC}"
    echo -n "  RAM:   "
    draw_bar "$MEM_USAGE"
    echo " (${MEM_USED_MB}MB / ${MEM_TOTAL_MB}MB)"
    echo -n "  Swap:  "
    draw_bar "$SWAP_USAGE"
    echo ""
    echo -e "${BLUE}└──────────────────────────────────────────────────────────────────────────┘${NC}"
    echo ""
    
    # Disk Section
    echo -e "${BOLD}${BLUE}┌─ DISK ───────────────────────────────────────────────────────────────────┐${NC}"
    echo -n "  Root:  "
    draw_bar "$DISK_USAGE"
    echo " (${DISK_USED} / ${DISK_TOTAL})"
    echo "  Max partition usage: ${MAX_DISK_USAGE}%"
    echo -e "${BLUE}└──────────────────────────────────────────────────────────────────────────┘${NC}"
    echo ""
    
    # Network Section
    echo -e "${BOLD}${BLUE}┌─ NETWORK ────────────────────────────────────────────────────────────────┐${NC}"
    echo "  Interface: $NETWORK_INTERFACE"
    echo "  RX: ${NETWORK_RX_MBPS} Mbps ↓ | TX: ${NETWORK_TX_MBPS} Mbps ↑"
    echo -e "${BLUE}└──────────────────────────────────────────────────────────────────────────┘${NC}"
    echo ""
    
    # Process Section
    echo -e "${BOLD}${BLUE}┌─ PROCESSES ──────────────────────────────────────────────────────────────┐${NC}"
    echo "  Total: $PROCESS_COUNT | Running: $PROCESS_RUNNING"
    echo -e "${BLUE}└──────────────────────────────────────────────────────────────────────────┘${NC}"
    echo ""
    
    # Services Section
    echo -e "${BOLD}${BLUE}┌─ MONITORED SERVICES ─────────────────────────────────────────────────────┐${NC}"
    for service in "${SERVICES[@]}"; do
        if check_service_status "$service"; then
            echo -e "  ${GREEN}●${NC} $service - Running"
        else
            echo -e "  ${RED}●${NC} $service - Stopped"
        fi
    done
    echo -e "${BLUE}└──────────────────────────────────────────────────────────────────────────┘${NC}"
    echo ""
    
    # Footer
    echo -e "${YELLOW}Press Ctrl+C to stop monitoring${NC}"
}

#=============================================================================
# LOGGING FUNCTIONS
#=============================================================================

log_metrics() {
    # JSON format for easy parsing
    local json=$(cat <<EOF
{
  "timestamp": "$(get_timestamp)",
  "hostname": "$(hostname)",
  "cpu": {
    "total": $CPU_USAGE,
    "user": $CPU_USER,
    "system": $CPU_SYSTEM,
    "iowait": $CPU_IOWAIT
  },
  "memory": {
    "usage_percent": $MEM_USAGE,
    "used_mb": $MEM_USED_MB,
    "total_mb": $MEM_TOTAL_MB,
    "swap_percent": $SWAP_USAGE
  },
  "disk": {
    "usage_percent": $DISK_USAGE,
    "used": "$DISK_USED",
    "total": "$DISK_TOTAL"
  },
  "network": {
    "interface": "$NETWORK_INTERFACE",
    "rx_mbps": $NETWORK_RX_MBPS,
    "tx_mbps": $NETWORK_TX_MBPS
  },
  "load": {
    "1min": $LOAD_1MIN,
    "5min": $LOAD_5MIN,
    "15min": $LOAD_15MIN
  }
}
EOF
)
    
    echo "$json" >> "$METRICS_LOG"
}

#=============================================================================
# MAIN LOOP
#=============================================================================

main() {
    echo "Starting System Monitor Dashboard..."
    echo "Logs: $METRICS_LOG"
    echo "Alerts: $ALERT_LOG"
    sleep 2
    
    while true; do
        # Collect all metrics
        get_cpu_usage
        get_memory_usage
        get_disk_usage
        get_network_usage
        get_load_average
        get_process_count
        
        # Display dashboard
        display_dashboard
        
        # Log metrics
        log_metrics
        
        # Check thresholds and alert
        check_thresholds
        
        # Monitor services
        monitor_services
        
        # Wait before next iteration
        sleep "$REFRESH_INTERVAL"
    done
}

# Run main function
main
```

**Usage**:
```bash
chmod +x system_monitor.sh
./system_monitor.sh
```

---

## 4. Alerting System {#alerting}

### Email Alerts

```bash
#!/bin/bash
# File: email_alerts.sh

setup_email_alerts() {
    # Install postfix/sendmail
    sudo apt install -y mailutils postfix
    
    # Configure postfix (choose "Internet Site")
    sudo dpkg-reconfigure postfix
}

send_email() {
    local subject="$1"
    local body="$2"
    local to="${3:-admin@example.com}"
    
    echo "$body" | mail -s "$subject" "$to"
}

# HTML Email with metrics
send_html_email() {
    local subject="$1"
    local to="${2:-admin@example.com}"
    
    local html_body=$(cat <<EOF
<!DOCTYPE html>
<html>
<head>
    <style>
        body { font-family: Arial, sans-serif; }
        .alert { background-color: #ff6b6b; color: white; padding: 10px; }
        .warning { background-color: #ffd93d; color: black; padding: 10px; }
        .ok { background-color: #6bcf7f; color: white; padding: 10px; }
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #4CAF50; color: white; }
    </style>
</head>
<body>
    <h2>System Alert: $(hostname)</h2>
    <div class="alert">
        <strong>Alert Time:</strong> $(date)<br>
        <strong>Server:</strong> $(hostname)
    </div>
    
    <h3>System Metrics</h3>
    <table>
        <tr><th>Metric</th><th>Value</th><th>Status</th></tr>
        <tr><td>CPU Usage</td><td>${CPU_USAGE}%</td><td>$([[ $CPU_USAGE -gt 80 ]] && echo "⚠️ High" || echo "✅ OK")</td></tr>
        <tr><td>Memory Usage</td><td>${MEM_USAGE}%</td><td>$([[ $MEM_USAGE -gt 85 ]] && echo "⚠️ High" || echo "✅ OK")</td></tr>
        <tr><td>Disk Usage</td><td>${DISK_USAGE}%</td><td>$([[ $DISK_USAGE -gt 90 ]] && echo "⚠️ High" || echo "✅ OK")</td></tr>
    </table>
</body>
</html>
EOF
)
    
    echo "$html_body" | mail -s "$subject" -a "Content-type: text/html" "$to"
}
```

---

### Slack/Webhook Alerts

```bash
#!/bin/bash
# File: webhook_alerts.sh

SLACK_WEBHOOK="https://hooks.slack.com/services/YOUR/WEBHOOK/URL"

send_slack_alert() {
    local message="$1"
    local color="${2:-danger}"  # danger (red), warning (yellow), good (green)
    
    local payload=$(cat <<EOF
{
    "attachments": [
        {
            "color": "$color",
            "title": "System Alert: $(hostname)",
            "text": "$message",
            "fields": [
                {
                    "title": "Server",
                    "value": "$(hostname)",
                    "short": true
                },
                {
                    "title": "Time",
                    "value": "$(date '+%Y-%m-%d %H:%M:%S')",
                    "short": true
                }
            ],
            "footer": "System Monitor",
            "ts": $(date +%s)
        }
    ]
}
EOF
)
    
    curl -X POST -H 'Content-type: application/json' \
        --data "$payload" \
        "$SLACK_WEBHOOK"
}

# Advanced Slack notification with metrics
send_slack_metrics() {
    local payload=$(cat <<EOF
{
    "blocks": [
        {
            "type": "header",
            "text": {
                "type": "plain_text",
                "text": "🚨 System Alert: $(hostname)"
            }
        },
        {
            "type": "section",
            "fields": [
                {
                    "type": "mrkdwn",
                    "text": "*CPU Usage:*\n${CPU_USAGE}% $([[ $CPU_USAGE -gt 80 ]] && echo "⚠️" || echo "✅")"
                },
                {
                    "type": "mrkdwn",
                    "text": "*Memory Usage:*\n${MEM_USAGE}% $([[ $MEM_USAGE -gt 85 ]] && echo "⚠️" || echo "✅")"
                },
                {
                    "type": "mrkdwn",
                    "text": "*Disk Usage:*\n${DISK_USAGE}% $([[ $DISK_USAGE -gt 90 ]] && echo "⚠️" || echo "✅")"
                },
                {
                    "type": "mrkdwn",
                    "text": "*Load Average:*\n${LOAD_1MIN} (1m)"
                }
            ]
        },
        {
            "type": "context",
            "elements": [
                {
                    "type": "mrkdwn",
                    "text": "Triggered at $(date '+%Y-%m-%d %H:%M:%S')"
                }
            ]
        }
    ]
}
EOF
)
    
    curl -X POST -H 'Content-type: application/json' \
        --data "$payload" \
        "$SLACK_WEBHOOK"
}

# Test alerts
send_slack_alert "This is a test alert" "warning"
```

---

### PagerDuty Integration

```bash
#!/bin/bash
# File: pagerduty_alerts.sh

PAGERDUTY_KEY="your_integration_key_here"

send_pagerduty_alert() {
    local severity="$1"  # critical, error, warning, info
    local summary="$2"
    local details="$3"
    
    local payload=$(cat <<EOF
{
  "routing_key": "$PAGERDUTY_KEY",
  "event_action": "trigger",
  "payload": {
    "summary": "$summary",
    "severity": "$severity",
    "source": "$(hostname)",
    "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%S.000Z)",
    "custom_details": {
      "details": "$details",
      "cpu_usage": "${CPU_USAGE}%",
      "memory_usage": "${MEM_USAGE}%",
      "disk_usage": "${DISK_USAGE}%"
    }
  }
}
EOF
)
    
    curl -X POST https://events.pagerduty.com/v2/enqueue \
        -H 'Content-Type: application/json' \
        -d "$payload"
}
```

---

## 5. Service Auto-Recovery {#service-recovery}

### Service Manager (Cross-Distribution)

```bash
#!/bin/bash
# File: service_manager.sh

# Detect init system
detect_init_system() {
    if command -v systemctl &>/dev/null && systemctl --version &>/dev/null; then
        echo "systemd"
    elif [ -f /sbin/init ] && /sbin/init --version 2>/dev/null | grep -q upstart; then
        echo "upstart"
    elif [ -f /etc/init.d/cron ] && [ ! -h /etc/init.d/cron ]; then
        echo "sysvinit"
    else
        echo "unknown"
    fi
}

# Universal service status check
check_service() {
    local service=$1
    local init_system=$(detect_init_system)
    
    case "$init_system" in
        systemd)
            systemctl is-active --quiet "$service"
            ;;
        upstart)
            status "$service" | grep -q "start/running"
            ;;
        sysvinit)
            service "$service" status &>/dev/null
            ;;
        *)
            # Fallback: check if process exists
            pgrep -x "$service" &>/dev/null
            ;;
    esac
}

# Universal service restart
restart_service() {
    local service=$1
    local init_system=$(detect_init_system)
    
    echo "Restarting $service using $init_system..."
    
    case "$init_system" in
        systemd)
            sudo systemctl restart "$service"
            ;;
        upstart)
            sudo restart "$service"
            ;;
        sysvinit)
            sudo service "$service" restart
            ;;
        *)
            # Fallback: kill and start
            sudo pkill -x "$service"
            sleep 2
            sudo "$service" &
            ;;
    esac
}

# Advanced: Service with retry logic
restart_service_with_retry() {
    local service=$1
    local max_attempts=3
    local attempt=1
    
    while [ $attempt -le $max_attempts ]; do
        echo "Restart attempt $attempt/$max_attempts for $service..."
        
        restart_service "$service"
        sleep 5
        
        if check_service "$service"; then
            echo "✅ Service $service restarted successfully"
            log_event "SUCCESS" "Service $service restarted on attempt $attempt"
            return 0
        fi
        
        attempt=$((attempt + 1))
        sleep 10
    done
    
    echo "❌ Failed to restart $service after $max_attempts attempts"
    log_event "CRITICAL" "Failed to restart $service after $max_attempts attempts"
    send_critical_alert "$service failed to restart"
    return 1
}

# Health check function
health_check() {
    local service=$1
    local check_type="${2:-process}"  # process, port, http
    local check_value="$3"
    
    case "$check_type" in
        process)
            pgrep -x "$service" &>/dev/null
            ;;
        port)
            netstat -tuln | grep -q ":$check_value "
            ;;
        http)
            curl -f -s -o /dev/null "$check_value"
            ;;
        *)
            check_service "$service"
            ;;
    esac
}

# Service dependency check
check_dependencies() {
    local service=$1
    local dependencies=()
    
    # Get dependencies from systemd
    if command -v systemctl &>/dev/null; then
        dependencies=($(systemctl list-dependencies --plain "$service" | grep -v "^$service$"))
    fi
    
    for dep in "${dependencies[@]}"; do
        if ! check_service "$dep"; then
            echo "⚠️  Dependency $dep is not running"
            restart_service "$dep"
        fi
    done
}

# Example monitoring loop
monitor_services() {
    local services=("nginx" "mysql" "redis")
    
    while true; do
        for service in "${services[@]}"; do
            if ! check_service "$service"; then
                echo "🔴 $service is down!"
                
                # Check dependencies first
                check_dependencies "$service"
                
                # Restart with retry
                restart_service_with_retry "$service"
                
            else
                echo "✅ $service is running"
            fi
        done
        
        sleep 60
    done
}
```

---

### Advanced Service Recovery with Backoff

```bash
#!/bin/bash
# File: advanced_recovery.sh

declare -A RESTART_COUNTS
declare -A LAST_RESTART_TIME

RESTART_THRESHOLD=5  # Max restarts in time window
TIME_WINDOW=300      # 5 minutes

should_restart_service() {
    local service=$1
    local current_time=$(date +%s)
    local last_time=${LAST_RESTART_TIME[$service]:-0}
    local count=${RESTART_COUNTS[$service]:-0}
    
    # Reset counter if outside time window
    if [ $((current_time - last_time)) -gt $TIME_WINDOW ]; then
        RESTART_COUNTS[$service]=0
        count=0
    fi
    
    # Check if exceeded restart threshold
    if [ $count -ge $RESTART_THRESHOLD ]; then
        echo "Service $service has been restarted $count times in last $TIME_WINDOW seconds"
        echo "Possible flapping - manual intervention required"
        send_critical_alert "Service $service is flapping (restarted $count times)"
        return 1
    fi
    
    return 0
}

restart_with_backoff() {
    local service=$1
    
    if ! should_restart_service "$service"; then
        return 1
    fi
    
    # Increment restart counter
    RESTART_COUNTS[$service]=$((${RESTART_COUNTS[$service]:-0} + 1))
    LAST_RESTART_TIME[$service]=$(date +%s)
    
    # Calculate backoff delay
    local count=${RESTART_COUNTS[$service]}
    local delay=$((2 ** count))  # Exponential backoff: 2, 4, 8, 16...
    
    echo "Waiting ${delay}s before restart (attempt #$count)..."
    sleep $delay
    
    restart_service "$service"
}
```

---

## 6. Cross-Distribution Compatibility {#compatibility}

### Distribution Detection

```bash
#!/bin/bash
# File: distro_detect.sh

detect_distro() {
    if [ -f /etc/os-release ]; then
        . /etc/os-release
        echo "$ID"
    elif [ -f /etc/lsb-release ]; then
        . /etc/lsb-release
        echo "$DISTRIB_ID" | tr '[:upper:]' '[:lower:]'
    elif [ -f /etc/debian_version ]; then
        echo "debian"
    elif [ -f /etc/redhat-release ]; then
        echo "rhel"
    else
        echo "unknown"
    fi
}

# Package manager detection
detect_package_manager() {
    if command -v apt-get &>/dev/null; then
        echo "apt"
    elif command -v yum &>/dev/null; then
        echo "yum"
    elif command -v dnf &>/dev/null; then
        echo "dnf"
    elif command -v pacman &>/dev/null; then
        echo "pacman"
    elif command -v zypper &>/dev/null; then
        echo "zypper"
    else
        echo "unknown"
    fi
}

# Install package universally
install_package() {
    local package=$1
    local pm=$(detect_package_manager)
    
    case "$pm" in
        apt)
            sudo apt-get update && sudo apt-get install -y "$package"
            ;;
        yum)
            sudo yum install -y "$package"
            ;;
        dnf)
            sudo dnf install -y "$package"
            ;;
        pacman)
            sudo pacman -S --noconfirm "$package"
            ;;
        zypper)
            sudo zypper install -y "$package"
            ;;
        *)
            echo "Unknown package manager"
            return 1
            ;;
    esac
}

# Service name mapping (different names on different distros)
map_service_name() {
    local service=$1
    local distro=$(detect_distro)
    
    case "$service" in
        apache)
            case "$distro" in
                ubuntu|debian) echo "apache2" ;;
                rhel|centos|fedora) echo "httpd" ;;
                *) echo "apache2" ;;
            esac
            ;;
        mysql)
            case "$distro" in
                ubuntu|debian) echo "mysql" ;;
                rhel|centos) echo "mysqld" ;;
                *) echo "mysql" ;;
            esac
            ;;
        *)
            echo "$service"
            ;;
    esac
}
```

---

### Universal System Monitor

```bash
#!/bin/bash
# File: universal_monitor.sh
# Works on: Ubuntu, Debian, CentOS, RHEL, Fedora, Arch, SUSE

# Source distro detection
DISTRO=$(detect_distro)
PKG_MANAGER=$(detect_package_manager)

# Install dependencies based on distro
install_dependencies() {
    local packages=()
    
    case "$PKG_MANAGER" in
        apt)
            packages=("sysstat" "bc" "curl" "net-tools")
            ;;
        yum|dnf)
            packages=("sysstat" "bc" "curl" "net-tools")
            ;;
        pacman)
            packages=("sysstat" "bc" "curl" "net-tools")
            ;;
    esac
    
    for pkg in "${packages[@]}"; do
        install_package "$pkg"
    done
}

# Get CPU - works on all distros
get_cpu_universal() {
    # /proc/stat is standard across all Linux
    read cpu user1 nice1 system1 idle1 rest <<< $(grep '^cpu ' /proc/stat)
    sleep 1
    read cpu user2 nice2 system2 idle2 rest <<< $(grep '^cpu ' /proc/stat)
    
    total1=$((user1 + nice1 + system1 + idle1))
    total2=$((user2 + nice2 + system2 + idle2))
    
    idle_delta=$((idle2 - idle1))
    total_delta=$((total2 - total1))
    
    usage=$((100 * (total_delta - idle_delta) / total_delta))
    echo "$usage"
}

# Get memory - works on all distros
get_memory_universal() {
    # /proc/meminfo is standard
    awk '/MemTotal/ {total=$2}
         /MemAvailable/ {avail=$2}
         END {used=total-avail; print int(used*100/total)}' /proc/meminfo
}

# Network interface - works on all distros
get_primary_interface() {
    # Try ip route first (modern)
    if command -v ip &>/dev/null; then
        ip route | awk '/default/ {print $5; exit}'
    # Fallback to route (older systems)
    elif command -v route &>/dev/null; then
        route -n | awk '/^0.0.0.0/ {print $8; exit}'
    # Fallback to /proc/net/route
    else
        awk '$2 == "00000000" {print $1; exit}' /proc/net/route
    fi
}
```

---

## 7. Cheat Sheets {#cheat-sheets}

### Quick Reference Card

```
╔═══════════════════════════════════════════════════════════════════╗
║                    SYSTEM MONITORING CHEAT SHEET                  ║
╠═══════════════════════════════════════════════════════════════════╣
║ QUICK CHECKS                                                      ║
║ ─────────────────────────────────────────────────────────────     ║
║ CPU:      top -bn1 | grep "Cpu(s)"                                ║
║ Memory:   free -h                                                 ║
║ Disk:     df -h                                                   ║
║ Network:  ip -s link show                                         ║
║ Load:     uptime                                                  ║
║                                                                   ║
║ METRICS FILES                                                     ║
║ ─────────────────────────────────────────────────────────────     ║
║ CPU:      /proc/stat, /proc/loadavg                               ║
║ Memory:   /proc/meminfo                                           ║
║ Disk:     /proc/diskstats                                         ║
║ Network:  /proc/net/dev                                           ║
║ Process:  /proc/<PID>/status, /proc/<PID>/stat                    ║
║                                                                   ║
║ ONE-LINERS                                                        ║
║ ─────────────────────────────────────────────────────────────     ║
║ # CPU %                                                           ║
║ top -bn1 | grep "Cpu(s)" | awk '{print 100-$8"%"}'                ║
║                                                                   ║
║ # Memory %                                                        ║
║ free | awk '/Mem/ {printf "%.0f%%", $3/$2*100}'                   ║
║                                                                   ║
║ # Disk % (root)                                                   ║
║ df / | awk 'NR==2 {print $5}'                                     ║
║                                                                   ║
║ # Top 5 CPU processes                                             ║
║ ps aux --sort=-%cpu | head -6                                     ║
║                                                                   ║
║ # Top 5 memory processes                                          ║
║ ps aux --sort=-%mem | head -6                                     ║
║                                                                   ║
║ # Network connections count                                       ║
║ ss -tan | wc -l                                                   ║
║                                                                   ║
║ # Listening ports                                                 ║
║ ss -tuln                                                          ║
╚═══════════════════════════════════════════════════════════════════╝
```

---

### Alert Thresholds Reference

```
┌─────────────────────────────────────────────────────────┐
│              RECOMMENDED THRESHOLDS                     │
├────────────┬──────────┬───────────┬─────────────────────┤
│ Metric     │ Warning  │ Critical  │ Notes               │
├────────────┼──────────┼───────────┼─────────────────────┤
│ CPU        │ 70%      │ 90%       │ Sustained usage     │
│ Memory     │ 80%      │ 95%       │ Physical RAM only   │
│ Swap       │ 20%      │ 50%       │ Indicates RAM issue │
│ Disk (/)   │ 80%      │ 90%       │ Root partition      │
│ Disk (/var)│ 85%      │ 95%       │ Logs partition      │
│ Inodes     │ 80%      │ 90%       │ File count limit    │
│ Load Avg   │ #CPU*1.5 │ #CPU*2    │ 5-min average       │
│ Network    │ 70% BW   │ 90% BW    │ Link bandwidth      │
└────────────┴──────────┴───────────┴─────────────────────┘

Production Adjustments:
- Database servers: Lower memory threshold (75%)
- Web servers: Higher CPU threshold acceptable (85%)
- File servers: Lower disk threshold (75%)
```

---

### Keyboard Shortcuts (Terminal Monitoring)

```
╔═══════════════════════════════════════════════════════════╗
║              TERMINAL SHORTCUTS FOR MONITORING            ║
╠═══════════════════════════════════════════════════════════╣
║ GENERAL                                                   ║
║ ─────────────────────────────────────────────────────     ║
║ Ctrl+C         Kill current process                       ║
║ Ctrl+Z         Suspend to background                      ║
║ Ctrl+L         Clear screen (same as 'clear')             ║
║ Ctrl+R         Reverse search command history             ║
║                                                           ║
║ TOP COMMANDS                                              ║
║ ─────────────────────────────────────────────────────     ║
║ 1              Toggle individual CPU cores                ║
║ P              Sort by CPU %                              ║
║ M              Sort by Memory %                           ║
║ T              Sort by running time                       ║
║ k              Kill process (enter PID)                   ║
║ r              Renice (change priority)                   ║
║ f              Field management (add/remove columns)      ║
║ W              Write config to ~/.toprc                   ║
║ q              Quit                                       ║
║                                                           ║
║ HTOP COMMANDS                                             ║
║ ─────────────────────────────────────────────────────     ║
║ F1             Help                                       ║
║ F2             Setup                                      ║
║ F3 (/)         Search                                     ║
║ F4 (\)         Filter                                     ║
║ F5 (t)         Tree view                                  ║
║ F6 (<>)        Sort by column                             ║
║ F9 (k)         Kill process                               ║
║ F10 (q)        Quit                                       ║
║ Space          Tag process                                ║
║ u              Filter by user                             ║
║                                                           ║
║ SCREEN/TMUX (For persistent monitoring)                   ║
║ ─────────────────────────────────────────────────────     ║
║ screen -S monitor    Start named session                  ║
║ Ctrl+A, D            Detach session                       ║
║ screen -r monitor    Reattach session                     ║
║ Ctrl+A, C            New window                           ║
║ Ctrl+A, N            Next window                          ║
║                                                           ║
║ tmux new -s monitor  Start tmux session                   ║
║ Ctrl+B, D            Detach tmux                          ║
║ tmux attach -t mon   Reattach                             ║
║ Ctrl+B, %            Split vertical                       ║
║ Ctrl+B, "            Split horizontal                     ║
╚═══════════════════════════════════════════════════════════╝
```

---

### Service Management Commands

```bash
# Systemd (Ubuntu 16.04+, CentOS 7+, Debian 8+)
systemctl status nginx
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl reload nginx
systemctl enable nginx    # Start on boot
systemctl disable nginx
systemctl is-active nginx
systemctl is-enabled nginx

# SysVinit (Older systems)
service nginx status
service nginx start
/etc/init.d/nginx restart

# Check all failed services
systemctl list-units --state=failed

# View service logs
journalctl -u nginx -n 50 -f  # Last 50 lines, follow
```

---

## 8. Hands-On Exercises {#exercises}

### Exercise 1: Basic Dashboard Setup

**Objective**: Create a minimal monitoring dashboard

```bash
#!/bin/bash
# File: exercise1_basic_dashboard.sh

# TODO: Implement these functions
get_cpu() {
    # HINT: Use top or /proc/stat
    echo "0"
}

get_memory() {
    # HINT: Use free command
    echo "0"
}

get_disk() {
    # HINT: Use df command
    echo "0"
}

# Display function
display() {
    clear
    echo "=== Basic System Monitor ==="
    echo "CPU:    $(get_cpu)%"
    echo "Memory: $(get_memory)%"
    echo "Disk:   $(get_disk)%"
}

# Main loop
while true; do
    display
    sleep 2
done
```

**Tasks**:
1. Implement `get_cpu()` to return actual CPU usage
2. Implement `get_memory()` to return memory percentage
3. Implement `get_disk()` to return disk usage for `/`
4. Add color coding (green < 70%, yellow < 90%, red >= 90%)

**Solution**:
```bash
get_cpu() {
    top -bn1 | grep "Cpu(s)" | awk '{print 100-$8}' | cut -d. -f1
}

get_memory() {
    free | awk '/Mem/ {printf "%.0f", $3/$2*100}'
}

get_disk() {
    df / | awk 'NR==2 {print $5}' | sed 's/%//'
}
```

---

### Exercise 2: Threshold Alerting

**Objective**: Implement alert system

```bash
#!/bin/bash
# File: exercise2_alerts.sh

CPU_THRESHOLD=80
MEM_THRESHOLD=85
ALERT_LOG="alerts.log"

check_cpu() {
    local usage=$(get_cpu)
    
    if [ "$usage" -gt "$CPU_THRESHOLD" ]; then
        # TODO: Log alert and send notification
        echo "CPU alert: ${usage}%"
    fi
}

check_memory() {
    # TODO: Implement memory threshold check
    :
}

# TODO: Implement send_alert function
send_alert() {
    local message="$1"
    # HINT: Use echo to file, mail command, or curl for webhook
}

# Main monitoring loop
while true; do
    check_cpu
    check_memory
    sleep 10
done
```

**Tasks**:
1. Complete `check_memory()` function
2. Implement `send_alert()` to write to log file
3. Add timestamp to alerts
4. Prevent alert spam (max 1 alert per 5 minutes)

**Advanced**: Send webhook notification when threshold exceeded

---

### Exercise 3: Service Auto-Restart

**Objective**: Monitor and auto-restart services

```bash
#!/bin/bash
# File: exercise3_service_recovery.sh

SERVICES=("nginx" "mysql")

check_service() {
    local service=$1
    systemctl is-active --quiet "$service"
    return $?
}

restart_service() {
    local service=$1
    # TODO: Implement restart with logging
}

monitor_services() {
    for service in "${SERVICES[@]}"; do
        if ! check_service "$service"; then
            echo "$service is down!"
            # TODO: Restart service
            # TODO: Verify it started
            # TODO: Send alert if restart failed
        fi
    done
}

while true; do
    monitor_services
    sleep 30
done
```

**Tasks**:
1. Implement service restart with verification
2. Add retry logic (3 attempts)
3. Log all restart attempts
4. Send alert if service fails to start after 3 attempts

**Testing**:
```bash
# Stop nginx to test
sudo systemctl stop nginx

# Run script - it should auto-restart
./exercise3_service_recovery.sh

# Verify nginx is running
systemctl status nginx
```

---

### Exercise 4: Historical Data Logging

**Objective**: Log metrics for trend analysis

```bash
#!/bin/bash
# File: exercise4_historical_logging.sh

LOG_FILE="metrics_$(date +%Y%m%d).log"

log_metrics() {
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    local cpu=$(get_cpu)
    local mem=$(get_memory)
    local disk=$(get_disk)
    
    # TODO: Log in CSV format
    # Format: timestamp,cpu,memory,disk
}

# TODO: Implement log rotation (keep last 7 days)
rotate_logs() {
    # HINT: Use find to delete old logs
    :
}

# Main loop
while true; do
    log_metrics
    rotate_logs
    sleep 60
done
```

**Tasks**:
1. Implement CSV logging
2. Add log rotation (delete logs older than 7 days)
3. Create a function to analyze trends
4. Generate daily summary report

**Analysis Script**:
```bash
#!/bin/bash
# File: analyze_metrics.sh

# TODO: Calculate average CPU for last 24 hours
# TODO: Find peak memory usage
# TODO: Identify usage patterns
```

---

### Exercise 5: Complete Production Dashboard

**Objective**: Combine all features

**Requirements**:
- Real-time display with refresh
- CPU, Memory, Disk, Network monitoring
- Alert on thresholds
- Service monitoring and auto-restart
- Historical logging (JSON format)
- Webhook notifications
- Works on Ubuntu and CentOS

**Testing Scenarios**:

1. **CPU Spike Test**:
```bash
# Generate CPU load
yes > /dev/null &
PID=$!

# Watch dashboard detect and alert
# Kill load generator
kill $PID
```

2. **Memory Test**:
```bash
# Allocate memory
stress --vm 1 --vm-bytes 2G --timeout 60s

# Dashboard should alert
```

3. **Service Failure Test**:
```bash
# Stop a service
sudo systemctl stop nginx

# Dashboard should:
# 1. Detect failure
# 2. Attempt restart
# 3. Log event
# 4. Send alert
```

4. **Disk Fill Test**:
```bash
# Fill disk (safely in /tmp)
dd if=/dev/zero of=/tmp/bigfile bs=1M count=1000

# Dashboard should alert when threshold hit

# Cleanup
rm /tmp/bigfile
```

---

### Exercise 6: Multi-Server Monitoring

**Objective**: Monitor multiple servers from central location

```bash
#!/bin/bash
# File: multi_server_monitor.sh

SERVERS=("192.168.1.10" "192.168.1.11" "192.168.1.12")
SSH_USER="monitor"

get_remote_cpu() {
    local server=$1
    ssh "${SSH_USER}@${server}" "top -bn1 | grep 'Cpu(s)' | awk '{print 100-\$8}'"
}

monitor_all_servers() {
    for server in "${SERVERS[@]}"; do
        echo "=== Server: $server ==="
        
        # TODO: Get metrics from remote server
        # TODO: Display in dashboard
        # TODO: Alert if thresholds exceeded
    done
}

# Main loop
while true; do
    clear
    monitor_all_servers
    sleep 5
done
```

**Setup**:
```bash
# Generate SSH key
ssh-keygen -t rsa -b 4096 -f ~/.ssh/monitor_key

# Copy to servers
for server in "${SERVERS[@]}"; do
    ssh-copy-id -i ~/.ssh/monitor_key "${SSH_USER}@${server}"
done
```

---

## 9. Muscle Memory Drills

### Drill 1: Quick System Check (30 seconds)

**Practice this sequence until automatic**:

```bash
# 1. CPU (5s)
top -bn1 | head -n 5

# 2. Memory (5s)
free -h

# 3. Disk (5s)
df -h | grep -E "^/dev"

# 4. Network (5s)
ip -s link

# 5. Services (5s)
systemctl status nginx mysql redis --no-pager

# 6. Load (5s)
uptime
```

**Goal**: Complete all checks in < 30 seconds

---

### Drill 2: Emergency Response (1 minute)

**Scenario**: High load alert received

```bash
# 1. Identify top CPU processes (10s)
ps aux --sort=-%cpu | head -5

# 2. Check memory hogs (10s)
ps aux --sort=-%mem | head -5

# 3. Disk I/O check (10s)
iostat -x 1 2

# 4. Network connections (10s)
ss -tan | wc -l
ss -tan | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn | head -5

# 5. Check logs for errors (10s)
journalctl -p err -n 20 --no-pager

# 6. System load and uptime (10s)
uptime && w
```

**Practice**: Run through this sequence 10 times, reducing time each iteration

---

### Drill 3: Service Recovery Workflow

```bash
# Memorize this exact sequence:

SERVICE="nginx"

# 1. Check status
systemctl status $SERVICE

# 2. If failed, check logs
journalctl -u $SERVICE -n 50 --no-pager

# 3. Attempt restart
sudo systemctl restart $SERVICE

# 4. Verify
systemctl is-active $SERVICE && echo "✅ Running" || echo "❌ Failed"

# 5. If still failed, check dependencies
systemctl list-dependencies $SERVICE

# 6. Check configuration
sudo nginx -t  # For nginx
# sudo mysql --version  # For mysql

# 7. Last resort: full stop/start
sudo systemctl stop $SERVICE
sleep 2
sudo systemctl start $SERVICE
```

**Practice**: Perform this workflow on different services until it becomes automatic

---

## 10. Common Mistakes & Solutions

### Mistake 1: Not Handling Missing Commands

**Wrong**:
```bash
CPU=$(top -bn1 | grep "Cpu(s)" | awk '{print 100-$8}')
# Fails if top not installed
```

**Right**:
```bash
get_cpu() {
    if command -v top &>/dev/null; then
        top -bn1 | grep "Cpu(s)" | awk '{print 100-$8}'
    elif [ -r /proc/stat ]; then
        # Fallback to /proc/stat
        read cpu user1 nice1 system1 idle1 rest <<< $(grep '^cpu ' /proc/stat)
        sleep 1
        read cpu user2 nice2 system2 idle2 rest <<< $(grep '^cpu ' /proc/stat)
        echo $(( 100 - (idle2-idle1)*100/(user2-user1+system2-system1+idle2-idle1) ))
    else
        echo "0"
    fi
}
```

---

### Mistake 2: Alert Spam

**Wrong**:
```bash
if [ $CPU_USAGE -gt 80 ]; then
    send_alert "High CPU"  # Sends every iteration!
fi
```

**Right**:
```bash
declare -A LAST_ALERT_TIME

should_send_alert() {
    local alert_type=$1
    local cooldown=300  # 5 minutes
    local current_time=$(date +%s)
    local last_time=${LAST_ALERT_TIME[$alert_type]:-0}
    
    if [ $((current_time - last_time)) -gt $cooldown ]; then
        LAST_ALERT_TIME[$alert_type]=$current_time
        return 0
    fi
    return 1
}

if [ $CPU_USAGE -gt 80 ] && should_send_alert "cpu"; then
    send_alert "High CPU"
fi
```

---

### Mistake 3: Ignoring Exit Codes

**Wrong**:
```bash
systemctl restart nginx
echo "Service restarted"
```

**Right**:
```bash
if systemctl restart nginx; then
    echo "Service restarted successfully"
    log_event "SUCCESS" "nginx restarted"
else
    echo "Failed to restart service"
    log_event "ERROR" "nginx restart failed"
    send_alert "Critical: nginx restart failed"
fi
```

---

### Mistake 4: Hardcoded Paths

**Wrong**:
```bash
LOG_FILE="/var/log/monitor.log"  # May not be writable
```

**Right**:
```bash
# Try multiple locations
for dir in "/var/log" "$HOME/logs" "/tmp"; do
    if [ -w "$dir" ]; then
        LOG_DIR="$dir"
        break
    fi
done

LOG_FILE="${LOG_DIR}/monitor.log"
mkdir -p "$(dirname "$LOG_FILE")"
```

---

### Mistake 5: Not Handling Service Name Variations

**Wrong**:
```bash
systemctl status apache  # Fails on Ubuntu (apache2)
```

**Right**:
```bash
get_apache_name() {
    if systemctl list-units --all | grep -q "apache2"; then
        echo "apache2"
    elif systemctl list-units --all | grep -q "httpd"; then
        echo "httpd"
    else
        echo "apache"
    fi
}

APACHE=$(get_apache_name)
systemctl status "$APACHE"
```

---

## 11. Production-Ready Example

### Complete Enterprise Dashboard

```bash
#!/bin/bash
#
# Enterprise System Monitor Dashboard
# Author: DevOps Team
# Version: 3.0
# Compatible: Ubuntu 16.04+, CentOS 7+, Debian 9+
#
# Features:
# - Real-time system monitoring
# - Multi-threshold alerting
# - Service auto-recovery
# - Historical logging (JSON)
# - Cross-distribution support
# - Webhook/Email notifications
# - Log rotation
# - PID file locking (single instance)
#

set -euo pipefail

#=============================================================================
# CONFIGURATION
#=============================================================================

# Script metadata
SCRIPT_NAME="system-monitor"
SCRIPT_VERSION="3.0"
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
PID_FILE="/var/run/${SCRIPT_NAME}.pid"

# Logging
LOG_DIR="/var/log/${SCRIPT_NAME}"
METRICS_LOG="${LOG_DIR}/metrics.jsonl"  # JSON Lines format
ALERT_LOG="${LOG_DIR}/alerts.log"
ERROR_LOG="${LOG_DIR}/error.log"
LOG_RETENTION_DAYS=30

# Thresholds
declare -A THRESHOLDS=(
    [cpu_warning]=70
    [cpu_critical]=90
    [mem_warning]=80
    [mem_critical]=95
    [disk_warning]=80
    [disk_critical]=90
    [load_warning]=8
    [load_critical]=16
)

# Services to monitor
MONITORED_SERVICES=()
# Add your services here
# MONITORED_SERVICES=("nginx" "mysql" "redis")

# Alert configuration
ALERT_EMAIL="${ALERT_EMAIL:-}"
SLACK_WEBHOOK="${SLACK_WEBHOOK:-}"
ALERT_COOLDOWN=300  # 5 minutes between same alerts

# Monitoring settings
REFRESH_INTERVAL=5
MAX_RESTART_ATTEMPTS=3
RESTART_BACKOFF_BASE=10

# Feature flags
ENABLE_ALERTS=true
ENABLE_SERVICE_RECOVERY=true
ENABLE_METRICS_LOGGING=true

# Terminal colors
if [ -t 1 ]; then
    RED='\033[0;31m'
    GREEN='\033[0;32m'
    YELLOW='\033[1;33m'
    BLUE='\033[0;34m'
    MAGENTA='\033[0;35m'
    CYAN='\033[0;36m'
    BOLD='\033[1m'
    NC='\033[0m'
else
    RED='' GREEN='' YELLOW='' BLUE='' MAGENTA='' CYAN='' BOLD='' NC=''
fi

#=============================================================================
# INITIALIZATION
#=============================================================================

# Create directories
mkdir -p "$LOG_DIR"
chmod 755 "$LOG_DIR"

# PID file check (prevent multiple instances)
if [ -f "$PID_FILE" ]; then
    OLD_PID=$(cat "$PID_FILE")
    if ps -p "$OLD_PID" > /dev/null 2>&1; then
        echo "ERROR: Another instance is already running (PID: $OLD_PID)"
        exit 1
    else
        echo "Removing stale PID file"
        rm -f "$PID_FILE"
    fi
fi

# Write PID
echo $$ > "$PID_FILE"

# Cleanup on exit
cleanup() {
    local exit_code=$?
    rm -f "$PID_FILE"
    tput cnorm  # Show cursor
    echo -e "\n${YELLOW}Monitoring stopped (exit code: $exit_code)${NC}"
    exit $exit_code
}

trap cleanup EXIT INT TERM

# Error handler
error_exit() {
    echo "ERROR: $1" | tee -a "$ERROR_LOG"
    exit 1
}

# Logging functions
log_error() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] ERROR: $*" >> "$ERROR_LOG"
}

log_alert() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" >> "$ALERT_LOG"
}

#=============================================================================
# DISTRIBUTION DETECTION
#=============================================================================

detect_distro() {
    if [ -f /etc/os-release ]; then
        . /etc/os-release
        echo "$ID"
    else
        echo "unknown"
    fi
}

DISTRO=$(detect_distro)

#=============================================================================
# DEPENDENCY CHECKS
#=============================================================================

check_dependencies() {
    local missing_deps=()
    local required_commands=("awk" "grep" "bc" "curl" "date")
    
    for cmd in "${required_commands[@]}"; do
        if ! command -v "$cmd" &>/dev/null; then
            missing_deps+=("$cmd")
        fi
    done
    
    if [ ${#missing_deps[@]} -gt 0 ]; then
        echo "ERROR: Missing required commands: ${missing_deps[*]}"
        echo "Please install: bc curl"
        exit 1
    fi
}

check_dependencies

#=============================================================================
# METRICS COLLECTION
#=============================================================================

# State variables
declare -A LAST_ALERT_TIME
declare -A SERVICE_RESTART_COUNT
declare -A SERVICE_LAST_RESTART

# CPU monitoring
get_cpu_metrics() {
    local stat1 stat2
    stat1=$(grep '^cpu ' /proc/stat)
    sleep 0.5
    stat2=$(grep '^cpu ' /proc/stat)
    
    read -r _ user1 nice1 system1 idle1 iowait1 irq1 softirq1 steal1 <<< "$stat1"
    read -r _ user2 nice2 system2 idle2 iowait2 irq2 softirq2 steal2 <<< "$stat2"
    
    local user=$((user2 - user1))
    local nice=$((nice2 - nice1))
    local system=$((system2 - system1))
    local idle=$((idle2 - idle1))
    local iowait=$((iowait2 - iowait1))
    local total=$((user + nice + system + idle + iowait + irq + softirq + steal))
    
    CPU_TOTAL=$((100 * (total - idle) / total))
    CPU_USER=$((100 * user / total))
    CPU_SYSTEM=$((100 * system / total))
    CPU_IOWAIT=$((100 * iowait / total))
}

# Memory monitoring
get_memory_metrics() {
    local mem_info
    mem_info=$(</proc/meminfo)
    
    local mem_total=$(echo "$mem_info" | awk '/MemTotal/ {print $2}')
    local mem_available=$(echo "$mem_info" | awk '/MemAvailable/ {print $2}')
    local mem_used=$((mem_total - mem_available))
    
    MEM_TOTAL_MB=$((mem_total / 1024))
    MEM_USED_MB=$((mem_used / 1024))
    MEM_AVAILABLE_MB=$((mem_available / 1024))
    MEM_PERCENT=$((100 * mem_used / mem_total))
    
    local swap_total=$(echo "$mem_info" | awk '/SwapTotal/ {print $2}')
    local swap_free=$(echo "$mem_info" | awk '/SwapFree/ {print $2}')
    
    if [ "$swap_total" -gt 0 ]; then
        SWAP_USED_MB=$(( (swap_total - swap_free) / 1024 ))
        SWAP_PERCENT=$((100 * (swap_total - swap_free) / swap_total))
    else
        SWAP_USED_MB=0
        SWAP_PERCENT=0
    fi
}

# Disk monitoring
get_disk_metrics() {
    DISK_USAGE=$(df / | awk 'NR==2 {print $5}' | tr -d '%')
    DISK_USED=$(df -h / | awk 'NR==2 {print $3}')
    DISK_TOTAL=$(df -h / | awk 'NR==2 {print $2}')
    DISK_AVAILABLE=$(df -h / | awk 'NR==2 {print $4}')
    
    # Find partition with highest usage
    MAX_DISK_USAGE=$(df | awk 'NR>1 && /^\/dev\// {gsub("%","",$5); print $5}' | sort -rn | head -1)
    MAX_DISK_PARTITION=$(df | awk 'NR>1 && /^\/dev\// {gsub("%","",$5); if($5==max){print $6; exit}}' max="$MAX_DISK_USAGE")
}

# Network monitoring
get_network_metrics() {
    local interface
    interface=$(ip route | awk '/default/ {print $5; exit}')
    NETWORK_INTERFACE="${interface:-none}"
    
    if [ "$NETWORK_INTERFACE" = "none" ]; then
        NETWORK_RX_MBPS=0
        NETWORK_TX_MBPS=0
        return
    fi
    
    local rx1 tx1 rx2 tx2
    read -r rx1 tx1 <<< "$(awk -v iface="${interface}:" '$1 == iface {print $2, $10}' /proc/net/dev)"
    sleep 0.5
    read -r rx2 tx2 <<< "$(awk -v iface="${interface}:" '$1 == iface {print $2, $10}' /proc/net/dev)"
    
    local rx_bytes=$(( (rx2 - rx1) * 2 ))
    local tx_bytes=$(( (tx2 - tx1) * 2 ))
    
    NETWORK_RX_MBPS=$(echo "scale=2; $rx_bytes * 8 / 1000000" | bc)
    NETWORK_TX_MBPS=$(echo "scale=2; $tx_bytes * 8 / 1000000" | bc)
}

# Load average
get_load_metrics() {
    read -r LOAD_1MIN LOAD_5MIN LOAD_15MIN <<< "$(awk '{print $1, $2, $3}' /proc/loadavg)"
    CPU_COUNT=$(nproc)
}

# Process stats
get_process_metrics() {
    PROCESS_TOTAL=$(ps aux | wc -l)
    PROCESS_RUNNING=$(ps aux | awk '$8 == "R" {count++} END {print count+0}')
    PROCESS_SLEEPING=$(ps aux | awk '$8 == "S" {count++} END {print count+0}')
}

# Collect all metrics
collect_all_metrics() {
    get_cpu_metrics
    get_memory_metrics
    get_disk_metrics
    get_network_metrics
    get_load_metrics
    get_process_metrics
}

#=============================================================================
# SERVICE MONITORING
#=============================================================================

check_service_status() {
    local service=$1
    
    if command -v systemctl &>/dev/null; then
        systemctl is-active --quiet "$service" 2>/dev/null
    elif command -v service &>/dev/null; then
        service "$service" status &>/dev/null
    else
        pgrep -x "$service" &>/dev/null
    fi
}

restart_service() {
    local service=$1
    local timestamp=$(date +%s)
    
    # Check restart count
    local restart_count=${SERVICE_RESTART_COUNT[$service]:-0}
    local last_restart=${SERVICE_LAST_RESTART[$service]:-0}
    
    # Reset counter if more than 1 hour passed
    if [ $((timestamp - last_restart)) -gt 3600 ]; then
        restart_count=0
    fi
    
    # Check if exceeded max attempts
    if [ $restart_count -ge $MAX_RESTART_ATTEMPTS ]; then
        log_alert "Service $service exceeded max restart attempts ($MAX_RESTART_ATTEMPTS)"
        send_critical_alert "Service $service has failed $restart_count times - manual intervention required"
        return 1
    fi
    
    # Calculate backoff delay
    local backoff=$((RESTART_BACKOFF_BASE * (2 ** restart_count)))
    log_alert "Restarting $service (attempt $((restart_count + 1))/$MAX_RESTART_ATTEMPTS, backoff ${backoff}s)"
    sleep "$backoff"
    
    # Attempt restart
    if command -v systemctl &>/dev/null; then
        systemctl restart "$service" 2>>"$ERROR_LOG"
    elif command -v service &>/dev/null; then
        service "$service" restart 2>>"$ERROR_LOG"
    fi
    
    sleep 3
    
    # Verify
    if check_service_status "$service"; then
        log_alert "Service $service restarted successfully"
        SERVICE_RESTART_COUNT[$service]=0
        return 0
    else
        SERVICE_RESTART_COUNT[$service]=$((restart_count + 1))
        SERVICE_LAST_RESTART[$service]=$timestamp
        log_alert "Service $service restart failed"
        return 1
    fi
}

monitor_services() {
    [ "$ENABLE_SERVICE_RECOVERY" = false ] && return
    
    for service in "${MONITORED_SERVICES[@]}"; do
        if ! check_service_status "$service"; then
            log_alert "Service $service is down"
            
            if restart_service "$service"; then
                send_alert "Service $service was down and has been restarted" "warning"
            else
                send_alert "CRITICAL: Service $service failed to restart" "danger"
            fi
        fi
    done
}

#=============================================================================
# ALERTING SYSTEM
#=============================================================================

should_send_alert() {
    local alert_key=$1
    local current_time=$(date +%s)
    local last_time=${LAST_ALERT_TIME[$alert_key]:-0}
    
    if [ $((current_time - last_time)) -gt $ALERT_COOLDOWN ]; then
        LAST_ALERT_TIME[$alert_key]=$current_time
        return 0
    fi
    return 1
}

send_email_alert() {
    [ -z "$ALERT_EMAIL" ] && return
    
    local subject=$1
    local body=$2
    
    if command -v mail &>/dev/null; then
        echo "$body" | mail -s "$subject" "$ALERT_EMAIL"
    fi
}

send_slack_alert() {
    [ -z "$SLACK_WEBHOOK" ] && return
    
    local message=$1
    local color=${2:-warning}
    
    local payload
    payload=$(cat <<EOF
{
    "attachments": [{
        "color": "$color",
        "title": "System Alert: $(hostname)",
        "text": "$message",
        "fields": [
            {"title": "Server", "value": "$(hostname)", "short": true},
            {"title": "Time", "value": "$(date '+%Y-%m-%d %H:%M:%S')", "short": true},
            {"title": "CPU", "value": "${CPU_TOTAL}%", "short": true},
            {"title": "Memory", "value": "${MEM_PERCENT}%", "short": true}
        ],
        "footer": "System Monitor v${SCRIPT_VERSION}",
        "ts": $(date +%s)
    }]
}
EOF
)
    
    curl -X POST -H 'Content-type: application/json' \
        --data "$payload" \
        "$SLACK_WEBHOOK" &>/dev/null
}

send_alert() {
    local message=$1
    local severity=${2:-warning}
    
    log_alert "$message"
    send_email_alert "System Alert" "$message"
    send_slack_alert "$message" "$severity"
}

send_critical_alert() {
    send_alert "$1" "danger"
}

check_thresholds() {
    [ "$ENABLE_ALERTS" = false ] && return
    
    # CPU alerts
    if [ "$CPU_TOTAL" -ge "${THRESHOLDS[cpu_critical]}" ]; then
        should_send_alert "cpu_critical" && \
            send_critical_alert "CPU usage critical: ${CPU_TOTAL}% (threshold: ${THRESHOLDS[cpu_critical]}%)"
    elif [ "$CPU_TOTAL" -ge "${THRESHOLDS[cpu_warning]}" ]; then
        should_send_alert "cpu_warning" && \
            send_alert "CPU usage warning: ${CPU_TOTAL}% (threshold: ${THRESHOLDS[cpu_warning]}%)" "warning"
    fi
    
    # Memory alerts
    if [ "$MEM_PERCENT" -ge "${THRESHOLDS[mem_critical]}" ]; then
        should_send_alert "mem_critical" && \
            send_critical_alert "Memory usage critical: ${MEM_PERCENT}% (threshold: ${THRESHOLDS[mem_critical]}%)"
    elif [ "$MEM_PERCENT" -ge "${THRESHOLDS[mem_warning]}" ]; then
        should_send_alert "mem_warning" && \
            send_alert "Memory usage warning: ${MEM_PERCENT}% (threshold: ${THRESHOLDS[mem_warning]}%)" "warning"
    fi
    
    # Disk alerts
    if [ "$DISK_USAGE" -ge "${THRESHOLDS[disk_critical]}" ]; then
        should_send_alert "disk_critical" && \
            send_critical_alert "Disk usage critical: ${DISK_USAGE}% on / (threshold: ${THRESHOLDS[disk_critical]}%)"
    elif [ "$DISK_USAGE" -ge "${THRESHOLDS[disk_warning]}" ]; then
        should_send_alert "disk_warning" && \
            send_alert "Disk usage warning: ${DISK_USAGE}% on / (threshold: ${THRESHOLDS[disk_warning]}%)" "warning"
    fi
}

#=============================================================================
# LOGGING
#=============================================================================

log_metrics_json() {
    [ "$ENABLE_METRICS_LOGGING" = false ] && return
    
    local json
    json=$(cat <<EOF
{"timestamp":"$(date -u +%Y-%m-%dT%H:%M:%SZ)","hostname":"$(hostname)","cpu":{"total":${CPU_TOTAL},"user":${CPU_USER},"system":${CPU_SYSTEM},"iowait":${CPU_IOWAIT}},"memory":{"percent":${MEM_PERCENT},"used_mb":${MEM_USED_MB},"total_mb":${MEM_TOTAL_MB},"swap_percent":${SWAP_PERCENT}},"disk":{"percent":${DISK_USAGE},"used":"${DISK_USED}","total":"${DISK_TOTAL}"},"network":{"interface":"${NETWORK_INTERFACE}","rx_mbps":${NETWORK_RX_MBPS},"tx_mbps":${NETWORK_TX_MBPS}},"load":{"1min":${LOAD_1MIN},"5min":${LOAD_5MIN},"15min":${LOAD_15MIN},"cpu_count":${CPU_COUNT}}}
EOF
)
    
    echo "$json" >> "$METRICS_LOG"
}

rotate_logs() {
    # Rotate if logs are older than retention period
    find "$LOG_DIR" -name "*.log*" -type f -mtime +${LOG_RETENTION_DAYS} -delete
}

#=============================================================================
# DISPLAY
#=============================================================================

draw_progress_bar() {
    local percent=$1
    local width=40
    local filled=$((percent * width / 100))
    local empty=$((width - filled))
    
    local color=$GREEN
    [ "$percent" -gt 80 ] && color=$RED
    [ "$percent" -gt 60 ] && [ "$percent" -le 80 ] && color=$YELLOW
    
    printf "${color}%-${filled}s${NC}%-${empty}s %3d%%" \
        "$(printf '█%.0s' $(seq 1 $filled))" \
        "$(printf '░%.0s' $(seq 1 $empty))" \
        "$percent"
}

display_dashboard() {
    clear
    tput civis  # Hide cursor
    
    # Header
    echo -e "${BOLD}${BLUE}"
    cat << "EOF"
╔══════════════════════════════════════════════════════════════════════════════╗
║                     SYSTEM MONITORING DASHBOARD                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
EOF
    echo -e "${NC}"
    
    printf "${BOLD}Server:${NC} %s | ${BOLD}Time:${NC} %s | ${BOLD}Uptime:${NC} %s\n" \
        "$(hostname)" "$(date '+%Y-%m-%d %H:%M:%S')" "$(uptime -p 2>/dev/null || uptime | sed 's/.*up //')"
    echo ""
    
    # CPU Section
    echo -e "${BOLD}${CYAN}┌─ CPU ────────────────────────────────────────────────────────────────────────┐${NC}"
    printf "  Total:  "
    draw_progress_bar "$CPU_TOTAL"
    echo ""
    printf "  User: %3d%% | System: %3d%% | IOWait: %3d%%\n" "$CPU_USER" "$CPU_SYSTEM" "$CPU_IOWAIT"
    printf "  Load Average: %s (1m) | %s (5m) | %s (15m) | CPUs: %d\n" \
        "$LOAD_1MIN" "$LOAD_5MIN" "$LOAD_15MIN" "$CPU_COUNT"
    echo -e "${CYAN}└──────────────────────────────────────────────────────────────────────────────┘${NC}"
    echo ""
    
    # Memory Section
    echo -e "${BOLD}${CYAN}┌─ MEMORY ─────────────────────────────────────────────────────────────────────┐${NC}"
    printf "  RAM:   "
    draw_progress_bar "$MEM_PERCENT"
    printf " (%dMB / %dMB)\n" "$MEM_USED_MB" "$MEM_TOTAL_MB"
    printf "  Swap:  "
    draw_progress_bar "$SWAP_PERCENT"
    printf " (%dMB used)\n" "$SWAP_USED_MB"
    echo -e "${CYAN}└──────────────────────────────────────────────────────────────────────────────┘${NC}"
    echo ""
    
    # Disk Section
    echo -e "${BOLD}${CYAN}┌─ DISK ───────────────────────────────────────────────────────────────────────┐${NC}"
    printf "  Root:  "
    draw_progress_bar "$DISK_USAGE"
    printf " (%s / %s used, %s available)\n" "$DISK_USED" "$DISK_TOTAL" "$DISK_AVAILABLE"
    printf "  Highest: %d%% on %s\n" "${MAX_DISK_USAGE:-0}" "${MAX_DISK_PARTITION:-/}"
    echo -e "${CYAN}└──────────────────────────────────────────────────────────────────────────────┘${NC}"
    echo ""
    
    # Network Section
    echo -e "${BOLD}${CYAN}┌─ NETWORK ────────────────────────────────────────────────────────────────────┐${NC}"
    printf "  Interface: %s\n" "$NETWORK_INTERFACE"
    printf "  RX: %s Mbps ↓ | TX: %s Mbps ↑\n" "$NETWORK_RX_MBPS" "$NETWORK_TX_MBPS"
    echo -e "${CYAN}└──────────────────────────────────────────────────────────────────────────────┘${NC}"
    echo ""
    
    # Process Section
    echo -e "${BOLD}${CYAN}┌─ PROCESSES ──────────────────────────────────────────────────────────────────┐${NC}"
    printf "  Total: %d | Running: %d | Sleeping: %d\n" \
        "$PROCESS_TOTAL" "$PROCESS_RUNNING" "$PROCESS_SLEEPING"
    echo -e "${CYAN}└──────────────────────────────────────────────────────────────────────────────┘${NC}"
    echo ""
    
    # Services Section
    if [ ${#MONITORED_SERVICES[@]} -gt 0 ]; then
        echo -e "${BOLD}${CYAN}┌─ MONITORED SERVICES ─────────────────────────────────────────────────────────┐${NC}"
        for service in "${MONITORED_SERVICES[@]}"; do
            if check_service_status "$service"; then
                echo -e "  ${GREEN}●${NC} $service - Running"
            else
                echo -e "  ${RED}●${NC} $service - Stopped"
            fi
        done
        echo -e "${CYAN}└──────────────────────────────────────────────────────────────────────────────┘${NC}"
        echo ""
    fi
    
    # Footer
    echo -e "${YELLOW}Press Ctrl+C to stop | Refresh: ${REFRESH_INTERVAL}s | Version: ${SCRIPT_VERSION}${NC}"
}

#=============================================================================
# MAIN LOOP
#=============================================================================

main() {
    echo "Starting System Monitor Dashboard v${SCRIPT_VERSION}"
    echo "Distribution: $DISTRO"
    echo "PID: $$"
    echo "Logs: $LOG_DIR"
    sleep 2
    
    while true; do
        collect_all_metrics
        display_dashboard
        log_metrics_json
        check_thresholds
        monitor_services
        rotate_logs
        
        sleep "$REFRESH_INTERVAL"
    done
}

# Start monitoring
main
```

Save as `system-monitor.sh`, make executable, and run:

```bash
chmod +x system-monitor.sh
sudo ./system-monitor.sh
```

---

## Final Summary

**What You've Learned**:

1. ✅ **Metrics Collection**: CPU, Memory, Disk, Network from `/proc`
2. ✅ **Real-time Display**: Color-coded dashboard with progress bars
3. ✅ **Alerting**: Email, Slack, webhook notifications with cooldowns
4. ✅ **Service Recovery**: Auto-restart with exponential backoff
5. ✅ **Historical Logging**: JSON format for easy parsing
6. ✅ **Cross-Distribution**: Works on Ubuntu, CentOS, Debian, etc.
7. ✅ **Production-Ready**: PID locking, error handling, log rotation

**Next Steps**:
- Add more services to monitor
- Integrate with Prometheus/Grafana
- Create alerts dashboard
- Add predictive analytics
- Implement auto-scaling triggers

**Production Deployment**:

```bash
# Install as systemd service
sudo cp system-monitor.sh /usr/local/bin/
sudo chmod +x /usr/local/bin/system-monitor.sh

# Create service file
sudo tee /etc/systemd/system/system-monitor.service << 'EOF'
[Unit]
Description=System Monitoring Dashboard
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/system-monitor.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable system-monitor
sudo systemctl start system-monitor

# Check status
sudo systemctl status system-monitor

# View logs
sudo journalctl -u system-monitor -f
```

Now we have enterprise-grade system monitoring! 🚀