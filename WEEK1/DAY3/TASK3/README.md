# systemd Service Engineering Mastery Guide

## Learning Objectives

By the end of this 1.5-hour session, you will:
- Create production-ready systemd services with proper dependencies
- Implement health checks, watchdogs, and auto-restart mechanisms
- Replace cron jobs with systemd timers
- Convert legacy SysV init scripts to modern systemd services

---

## Part 1: systemd Fundamentals (20 mins)

### What is systemd?

**systemd** is the init system (PID 1) for modern Linux distributions. It manages:

```
System Boot Process
├── Kernel loads
├── systemd starts (PID 1)
│   ├── Mounts filesystems
│   ├── Starts services (in parallel)
│   ├── Manages dependencies
│   └── Reaches target state
└── System ready
```

### Why systemd Matters for DevOps

| Feature | Benefit |
|---------|---------|
| Parallel startup | Faster boot times (dependency-based) |
| Socket activation | Services start on-demand |
| Cgroups integration | Resource control per service |
| Unified logging | journald captures all output |
| Declarative config | Infrastructure as code |
| Automatic restart | Self-healing services |

### Core Concepts

```
Units = Configuration files that systemd manages

Unit Types:
├── .service    → Daemons and processes
├── .socket     → IPC/network sockets
├── .timer      → Scheduled tasks (cron replacement)
├── .target     → Groups of units (runlevels)
├── .mount      → Filesystem mount points
├── .path       → Path-based activation
├── .slice      → Resource management groups
└── .device     → Kernel device exposure
```

### Directory Hierarchy

```bash
# Priority (highest to lowest):
/etc/systemd/system/          # Local admin customizations (USE THIS)
/run/systemd/system/          # Runtime units (transient)
/usr/lib/systemd/system/      # Package-installed units (DON'T EDIT)

# User services:
~/.config/systemd/user/       # Per-user services
```

**Golden Rule**: Always create/modify units in `/etc/systemd/system/`

### Essential Commands Cheat Sheet

```bash
# Service Management
systemctl start|stop|restart|reload <unit>
systemctl enable|disable <unit>           # Boot persistence
systemctl enable --now <unit>             # Enable AND start
systemctl status <unit>                   # Detailed status
systemctl is-active <unit>                # Check if running
systemctl is-enabled <unit>               # Check boot status
systemctl mask|unmask <unit>              # Prevent/allow starting

# System State
systemctl list-units                      # All loaded units
systemctl list-units --failed             # Failed units only
systemctl list-unit-files                 # All available units
systemctl list-dependencies <unit>        # Dependency tree

# Unit File Operations
systemctl cat <unit>                      # View unit file
systemctl show <unit>                     # All properties
systemctl edit <unit>                     # Create override
systemctl edit --full <unit>              # Edit complete file
systemctl daemon-reload                   # Reload after changes

# Logs
journalctl -u <unit>                      # Unit logs
journalctl -u <unit> -f                   # Follow logs
journalctl -u <unit> --since "10 min ago"
journalctl -u <unit> -p err               # Errors only

# Analysis
systemd-analyze                           # Boot time
systemd-analyze blame                     # Slow units
systemd-analyze critical-chain <unit>     # Startup chain
systemd-analyze verify <unit-file>        # Validate syntax
```

---

## Part 2: Service Unit Deep Dive (25 mins)

### Complete Unit File Anatomy

```ini
# /etc/systemd/system/myapp.service

#===============================================================================
# [Unit] Section - Identity and Dependencies
#===============================================================================
[Unit]
# Human-readable description
Description=My Application Server

# Documentation links
Documentation=https://docs.myapp.com
Documentation=man:myapp(8)

# Dependency ordering (WHEN to start)
After=network.target postgresql.service redis.service
Before=nginx.service

# Dependency requirements (WHETHER to start)
Requires=postgresql.service          # Hard dependency - fails if dep fails
Wants=redis.service                   # Soft dependency - continues if dep fails
BindsTo=docker.service                # Strongest - stops if dep stops

# Conditional execution
ConditionPathExists=/etc/myapp/config.yaml
ConditionPathIsDirectory=/var/lib/myapp
AssertFileIsExecutable=/usr/bin/myapp

# Conflict handling
Conflicts=myapp-legacy.service

#===============================================================================
# [Service] Section - Process Management
#===============================================================================
[Service]
# Service type (CRITICAL - determines how systemd tracks the process)
Type=simple                           # Default: ExecStart IS the main process

# Execution
User=appuser
Group=appgroup
WorkingDirectory=/opt/myapp

# Environment
Environment=NODE_ENV=production
Environment=PORT=3000
EnvironmentFile=/etc/myapp/env        # Load from file
EnvironmentFile=-/etc/myapp/local.env # Optional file (- prefix)

# Commands
ExecStartPre=/opt/myapp/pre-start.sh  # Run before main process
ExecStart=/opt/myapp/bin/server       # Main process command
ExecStartPost=/opt/myapp/post-start.sh # Run after main starts
ExecReload=/bin/kill -HUP $MAINPID    # Reload command
ExecStop=/opt/myapp/graceful-stop.sh  # Custom stop
ExecStopPost=/opt/myapp/cleanup.sh    # Cleanup after stop

# Restart behavior
Restart=on-failure                    # When to restart
RestartSec=5                          # Delay between restarts
StartLimitBurst=5                     # Max restarts in interval
StartLimitIntervalSec=60              # Interval for burst limit

# Timeouts
TimeoutStartSec=30                    # Max startup time
TimeoutStopSec=30                     # Max stop time
TimeoutSec=30                         # Both start and stop

# Watchdog
WatchdogSec=30                        # Health check interval

# Resource limits
LimitNOFILE=65535                     # Max open files
LimitNPROC=4096                       # Max processes
MemoryMax=2G                          # Memory limit
CPUQuota=80%                          # CPU limit

# Security hardening
NoNewPrivileges=true
ProtectSystem=strict                  # Read-only /usr, /boot
ProtectHome=true                      # No access to /home
PrivateTmp=true                       # Isolated /tmp
ReadOnlyPaths=/etc
ReadWritePaths=/var/lib/myapp /var/log/myapp

# Logging
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp

#===============================================================================
# [Install] Section - Boot Integration
#===============================================================================
[Install]
# Which target activates this service
WantedBy=multi-user.target            # Non-graphical multi-user

# Aliases
Alias=app.service

# Additional units to enable
Also=myapp-worker.service
```

### Service Types Explained

```ini
# Type=simple (Default)
# - ExecStart process IS the main service process
# - systemd considers service started immediately
# Use for: Node.js, Python, Go binaries, anything that doesn't fork

[Service]
Type=simple
ExecStart=/usr/bin/node /app/server.js


# Type=forking
# - ExecStart process forks and parent exits
# - Child becomes main service process
# - MUST specify PIDFile for proper tracking
# Use for: Traditional daemons (Apache, nginx in some configs)

[Service]
Type=forking
PIDFile=/var/run/myapp.pid
ExecStart=/usr/sbin/myapp --daemon


# Type=oneshot
# - Process runs and exits
# - Service considered active after completion
# - Use RemainAfterExit=yes to keep "active" status
# Use for: Initialization scripts, setup tasks

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/opt/scripts/initialize-database.sh


# Type=notify
# - Service sends notification to systemd when ready
# - Most accurate startup detection
# - Requires sd_notify() in code or systemd-notify wrapper
# Use for: Services with slow startup, custom readiness

[Service]
Type=notify
ExecStart=/usr/bin/myapp --systemd-notify


# Type=dbus
# - Ready when specified D-Bus name is acquired
# Use for: D-Bus activated services

[Service]
Type=dbus
BusName=org.example.MyApp
ExecStart=/usr/bin/myapp


# Type=idle
# - Like simple, but waits for all jobs to complete
# Use for: Services that should start last
```

### Practical Example: Node.js Application

```ini
# /etc/systemd/system/webapp.service
[Unit]
Description=Node.js Web Application
Documentation=https://github.com/company/webapp
After=network.target mongodb.service
Wants=mongodb.service

[Service]
Type=simple
User=webapp
Group=webapp
WorkingDirectory=/opt/webapp

# Environment
Environment=NODE_ENV=production
Environment=PORT=3000
EnvironmentFile=/etc/webapp/env

# Process management
ExecStart=/usr/bin/node /opt/webapp/server.js
ExecReload=/bin/kill -USR2 $MAINPID

# Restart policy
Restart=always
RestartSec=10
StartLimitBurst=5
StartLimitIntervalSec=120

# Timeouts
TimeoutStartSec=30
TimeoutStopSec=30

# Logging
StandardOutput=journal
StandardError=journal
SyslogIdentifier=webapp

# Security
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=true
PrivateTmp=true
ReadWritePaths=/opt/webapp/logs /opt/webapp/uploads

# Resource limits
LimitNOFILE=65535
MemoryMax=1G

[Install]
WantedBy=multi-user.target
```

### Deploy and Test

```bash
# 1. Create the service file
sudo vim /etc/systemd/system/webapp.service

# 2. Reload systemd
sudo systemctl daemon-reload

# 3. Enable and start
sudo systemctl enable --now webapp.service

# 4. Verify
sudo systemctl status webapp.service

# 5. Check logs
sudo journalctl -u webapp.service -f

# 6. Test restart behavior
sudo kill -9 $(pgrep -f "node.*server.js")
# Watch it auto-restart!
sudo systemctl status webapp.service
```

---

## Part 3: Dependencies and Ordering (15 mins)

### Dependency Types Matrix

| Directive | Effect | Use When |
|-----------|--------|----------|
| `After=` | Start after X | Ordering only, no requirement |
| `Before=` | Start before X | Reverse ordering |
| `Requires=` | Must have X running | Hard dependency, fail if X fails |
| `Wants=` | Would like X running | Soft dependency, continue if X fails |
| `BindsTo=` | Tied to X lifecycle | Stop when X stops |
| `PartOf=` | Affected by X operations | Restart/stop with X |
| `Conflicts=` | Cannot run with X | Stops X when starting |

### Dependency Examples

```ini
# Example 1: Web app needing database
[Unit]
Description=Web Application
After=network.target postgresql.service
Requires=postgresql.service
# Meaning: Start after network and postgres, FAIL if postgres fails

# Example 2: Cache is optional
[Unit]
Description=API Server  
After=network.target mysql.service redis.service
Requires=mysql.service
Wants=redis.service
# Meaning: Must have mysql, redis is nice-to-have

# Example 3: Container-dependent service
[Unit]
Description=App in Docker
After=docker.service
BindsTo=docker.service
# Meaning: If docker stops, this stops too

# Example 4: Multiple instances conflict
[Unit]
Description=Production App
Conflicts=myapp-staging.service myapp-dev.service
# Meaning: Cannot run alongside staging or dev versions
```

### Network Dependency Best Practices

```ini
# Basic network (interfaces up)
After=network.target

# Network fully online (IPs assigned, routes available)
After=network-online.target
Wants=network-online.target

# Specific port available
After=network.target
ExecStartPre=/bin/sh -c 'until nc -z localhost 5432; do sleep 1; done'
```

### Creating Service Groups with Targets

```ini
# /etc/systemd/system/myapp.target
[Unit]
Description=MyApp Application Stack
Requires=myapp-web.service myapp-worker.service myapp-scheduler.service
After=myapp-web.service myapp-worker.service myapp-scheduler.service

[Install]
WantedBy=multi-user.target
```

```bash
# Manage entire stack
sudo systemctl start myapp.target
sudo systemctl stop myapp.target
sudo systemctl status myapp.target
```

---

## Part 4: Health Checks and Auto-Restart (20 mins)

### Restart Policies

```ini
# Restart options
Restart=no            # Never restart (default)
Restart=always        # Always restart
Restart=on-success    # Only if clean exit (code 0)
Restart=on-failure    # Non-zero exit, signal, timeout
Restart=on-abnormal   # Signal, timeout, watchdog
Restart=on-abort      # Uncaught signal
Restart=on-watchdog   # Watchdog timeout only
```

### Complete Auto-Restart Configuration

```ini
[Service]
# Restart on any failure
Restart=on-failure
RestartSec=5

# Prevent restart loops
StartLimitBurst=5              # Max 5 restarts...
StartLimitIntervalSec=300      # ...within 5 minutes

# What to do when limit reached
StartLimitAction=none          # Just stop trying (default)
# StartLimitAction=reboot      # Reboot the system
# StartLimitAction=reboot-force # Immediate reboot

# Exit codes that should NOT trigger restart
RestartPreventExitStatus=0 1 255
# SuccessExitStatus=143        # Add SIGTERM as success
```

### Watchdog Implementation

The watchdog ensures the service is actually functioning, not just running:

```ini
[Service]
Type=notify
WatchdogSec=30                 # Service must ping every 30s
WatchdogSignal=SIGABRT         # Signal sent on timeout (default)
Restart=on-watchdog
```

**Application Integration (Python example):**

```python
#!/usr/bin/env python3
# app_with_watchdog.py

import os
import time
import threading
from systemd.daemon import notify, Notification

def watchdog_ping():
    """Send periodic watchdog pings to systemd."""
    watchdog_usec = os.environ.get('WATCHDOG_USEC')
    if not watchdog_usec:
        return  # Watchdog not enabled
    
    # Ping at half the watchdog interval
    interval = int(watchdog_usec) / 1_000_000 / 2
    
    while True:
        # Only ping if application is healthy
        if health_check():
            notify(Notification.WATCHDOG)
        time.sleep(interval)

def health_check():
    """Check if application is healthy."""
    try:
        # Add your health checks here
        # - Database connection
        # - Queue connectivity
        # - Memory usage
        return True
    except Exception:
        return False

def main():
    # Signal ready to systemd
    notify(Notification.READY)
    
    # Start watchdog thread
    watchdog_thread = threading.Thread(target=watchdog_ping, daemon=True)
    watchdog_thread.start()
    
    # Main application loop
    while True:
        do_work()

if __name__ == '__main__':
    main()
```

**Bash Watchdog Wrapper:**

```bash
#!/bin/bash
# watchdog_wrapper.sh - For apps that can't integrate directly

# Start the actual application in background
/opt/myapp/bin/server &
APP_PID=$!

# Health check loop
while kill -0 $APP_PID 2>/dev/null; do
    if curl -sf http://localhost:8080/health > /dev/null; then
        systemd-notify WATCHDOG=1
    fi
    sleep 10
done

# App died, exit with error
exit 1
```

```ini
# Service using wrapper
[Service]
Type=notify
ExecStart=/opt/myapp/watchdog_wrapper.sh
WatchdogSec=30
NotifyAccess=all
```

### External Health Check Service

```ini
# /etc/systemd/system/myapp-health.service
[Unit]
Description=MyApp Health Monitor
BindsTo=myapp.service
After=myapp.service

[Service]
Type=simple
ExecStart=/usr/local/bin/health-monitor.sh myapp 8080 /health
Restart=always
RestartSec=5

[Install]
WantedBy=myapp.service
```

```bash
#!/bin/bash
# /usr/local/bin/health-monitor.sh

SERVICE="$1"
PORT="$2"
ENDPOINT="${3:-/health}"
CHECK_INTERVAL="${4:-30}"
FAIL_THRESHOLD="${5:-3}"

failures=0

while true; do
    if curl -sf "http://localhost:${PORT}${ENDPOINT}" > /dev/null 2>&1; then
        failures=0
        logger -t health-monitor "$SERVICE: healthy"
    else
        ((failures++))
        logger -t health-monitor "$SERVICE: unhealthy (failures: $failures)"
        
        if [[ $failures -ge $FAIL_THRESHOLD ]]; then
            logger -t health-monitor "$SERVICE: restarting due to health failures"
            systemctl restart "$SERVICE"
            failures=0
            sleep 60  # Grace period after restart
        fi
    fi
    
    sleep "$CHECK_INTERVAL"
done
```

### Socket Activation (On-Demand Starting)

```ini
# /etc/systemd/system/myapp.socket
[Unit]
Description=MyApp Socket

[Socket]
ListenStream=8080
Accept=no

[Install]
WantedBy=sockets.target
```

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=MyApp Service
Requires=myapp.socket
After=myapp.socket

[Service]
Type=simple
ExecStart=/opt/myapp/bin/server
# Socket passed as file descriptor
# StandardInput=socket (for inetd-style)

[Install]
WantedBy=multi-user.target
```

```bash
# Enable socket, not service
sudo systemctl enable myapp.socket
sudo systemctl start myapp.socket

# Service starts automatically on first connection
curl http://localhost:8080
```

---

## Part 5: systemd Timers (15 mins)

### Timer vs Cron Comparison

| Feature | Cron | systemd Timer |
|---------|------|---------------|
| Dependencies | ❌ | ✅ Full systemd deps |
| Missed runs | ❌ Lost | ✅ `Persistent=true` |
| Randomization | ❌ Manual | ✅ `RandomizedDelaySec` |
| Logging | Custom | ✅ journald integrated |
| Resources | N/A | ✅ Full cgroup control |
| Boot timing | ❌ | ✅ `OnBootSec` |
| Monitoring | External | ✅ `systemctl status` |

### Timer Anatomy

Timers require TWO files:
1. `mybackup.timer` - When to run
2. `mybackup.service` - What to run

```ini
# /etc/systemd/system/mybackup.service
[Unit]
Description=Daily Backup Job
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
User=backup
ExecStart=/opt/backup/run-backup.sh
StandardOutput=journal
StandardError=journal

# Don't install - timer activates this
```

```ini
# /etc/systemd/system/mybackup.timer
[Unit]
Description=Run Backup Daily at 2 AM

[Timer]
# Calendar-based scheduling
OnCalendar=*-*-* 02:00:00

# Catch up on missed runs
Persistent=true

# Random delay to prevent thundering herd
RandomizedDelaySec=300

# Accuracy (default 1min, can be more precise)
AccuracySec=1s

[Install]
WantedBy=timers.target
```

### OnCalendar Syntax

```bash
# Format: DayOfWeek Year-Month-Day Hour:Minute:Second

# Examples:
OnCalendar=hourly                    # Every hour at :00
OnCalendar=daily                     # Every day at 00:00:00
OnCalendar=weekly                    # Every Monday at 00:00:00
OnCalendar=monthly                   # First day of month at 00:00:00

OnCalendar=*-*-* 04:00:00           # Daily at 4 AM
OnCalendar=Mon *-*-* 09:00:00       # Every Monday at 9 AM
OnCalendar=*-*-01 00:00:00          # First of every month at midnight
OnCalendar=*-01,07-01 00:00:00      # Jan 1 and Jul 1 at midnight

OnCalendar=Mon..Fri *-*-* 09:00:00  # Weekdays at 9 AM
OnCalendar=*-*-* *:00/15:00         # Every 15 minutes
OnCalendar=*-*-* 00/2:00:00         # Every 2 hours

# Test your expression:
systemd-analyze calendar "Mon..Fri *-*-* 09:00:00"
```

### Monotonic Timers (Relative to Events)

```ini
[Timer]
# Relative to system boot
OnBootSec=5min              # 5 minutes after boot

# Relative to timer activation
OnActiveSec=1h              # 1 hour after timer starts

# Relative to last service run
OnUnitActiveSec=30min       # 30 min after service last started
OnUnitInactiveSec=1h        # 1 hour after service stopped

# Combined example: boot + repeating
OnBootSec=5min              # First run 5 min after boot
OnUnitActiveSec=1h          # Then every hour after that
```

### Complete Timer Examples

**Log Rotation Timer:**

```ini
# /etc/systemd/system/logrotate.service
[Unit]
Description=Rotate Logs

[Service]
Type=oneshot
ExecStart=/usr/sbin/logrotate /etc/logrotate.conf
Nice=19
IOSchedulingClass=best-effort
IOSchedulingPriority=7
```

```ini
# /etc/systemd/system/logrotate.timer
[Unit]
Description=Daily Log Rotation

[Timer]
OnCalendar=daily
AccuracySec=1h
Persistent=true

[Install]
WantedBy=timers.target
```

**Database Cleanup Timer:**

```ini
# /etc/systemd/system/db-cleanup.service
[Unit]
Description=Database Cleanup Job
After=postgresql.service
Requires=postgresql.service

[Service]
Type=oneshot
User=postgres
ExecStart=/opt/scripts/db-cleanup.sh
TimeoutStartSec=3600
```

```ini
# /etc/systemd/system/db-cleanup.timer
[Unit]
Description=Weekly Database Cleanup

[Timer]
OnCalendar=Sun *-*-* 03:00:00
Persistent=true
RandomizedDelaySec=1800

[Install]
WantedBy=timers.target
```

### Managing Timers

```bash
# Enable and start timer
sudo systemctl daemon-reload
sudo systemctl enable --now mybackup.timer

# List all timers
systemctl list-timers --all

# Check specific timer
systemctl status mybackup.timer

# See next run times
systemd-analyze calendar "*-*-* 02:00:00" --iterations=5

# Manually trigger the associated service
sudo systemctl start mybackup.service

# View timer logs
journalctl -u mybackup.timer
journalctl -u mybackup.service
```

---

## Part 6: Legacy Init Script Conversion (15 mins)

### Understanding SysV Init Scripts

```bash
#!/bin/bash
# /etc/init.d/myapp - Traditional init script

### BEGIN INIT INFO
# Provides:          myapp
# Required-Start:    $network $local_fs
# Required-Stop:     $network $local_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: MyApp Service
### END INIT INFO

. /lib/lsb/init-functions

NAME=myapp
DAEMON=/usr/bin/myapp
PIDFILE=/var/run/$NAME.pid
LOGFILE=/var/log/$NAME.log
USER=myapp
DAEMON_OPTS="--config /etc/myapp/config.yaml"

case "$1" in
    start)
        log_daemon_msg "Starting $NAME"
        start-stop-daemon --start --quiet --pidfile $PIDFILE \
            --chuid $USER --background --make-pidfile \
            --exec $DAEMON -- $DAEMON_OPTS >> $LOGFILE 2>&1
        log_end_msg $?
        ;;
    stop)
        log_daemon_msg "Stopping $NAME"
        start-stop-daemon --stop --quiet --pidfile $PIDFILE
        log_end_msg $?
        rm -f $PIDFILE
        ;;
    restart)
        $0 stop
        sleep 2
        $0 start
        ;;
    status)
        status_of_proc -p $PIDFILE "$DAEMON" "$NAME" && exit 0 || exit $?
        ;;
    *)
        echo "Usage: $0 {start|stop|restart|status}"
        exit 1
        ;;
esac
```

### Conversion Mapping Table

| Init Script Pattern | systemd Equivalent |
|--------------------|-------------------|
| `start-stop-daemon --background` | Remove (Type=simple handles it) |
| `--make-pidfile` | Remove (Type=simple doesn't need PID file) |
| `--chuid $USER` | `User=` / `Group=` |
| `--exec $DAEMON` | `ExecStart=` |
| `>> $LOGFILE 2>&1` | `StandardOutput=journal` |
| `DAEMON_OPTS=` | `ExecStart=` arguments |
| `Required-Start: $network` | `After=network.target` |
| `sleep 2` | `RestartSec=2` |
| `/etc/default/myapp` source | `EnvironmentFile=/etc/default/myapp` |
| `ulimit -n 65535` | `LimitNOFILE=65535` |

### Converted systemd Service

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=MyApp Service
Documentation=man:myapp(8)
After=network.target local-fs.target
Wants=network.target

[Service]
Type=simple
User=myapp
Group=myapp

# Environment (replaces /etc/default/myapp source)
EnvironmentFile=-/etc/default/myapp

# Command (no --background, no pidfile)
ExecStart=/usr/bin/myapp --config /etc/myapp/config.yaml

# Restart (replaces manual restart logic)
Restart=on-failure
RestartSec=2

# Logging (replaces >> $LOGFILE)
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp

# From ulimit in init script
LimitNOFILE=65535

# Working directory if needed
WorkingDirectory=/opt/myapp

[Install]
WantedBy=multi-user.target
```

### Handling Forking Daemons

If the application truly daemonizes (forks and exits):

```ini
[Service]
Type=forking
PIDFile=/var/run/myapp.pid
ExecStart=/usr/bin/myapp --daemon --pidfile /var/run/myapp.pid
```

### Complete Conversion Script

```bash
#!/bin/bash
# convert_init_to_systemd.sh - Automated init script converter

set -euo pipefail

INPUT_SCRIPT="$1"
OUTPUT_DIR="${2:-/etc/systemd/system}"

[[ -f "$INPUT_SCRIPT" ]] || { echo "Usage: $0 <init-script> [output-dir]"; exit 1; }

# Extract information from init script
extract_value() {
    grep -oP "(?<=$1=)[^\s]+" "$INPUT_SCRIPT" 2>/dev/null | head -1 || echo ""
}

extract_lsb() {
    grep -oP "(?<=# $1:)[^\n]+" "$INPUT_SCRIPT" 2>/dev/null | xargs || echo ""
}

# Parse init script
NAME=$(extract_value "NAME" || basename "$INPUT_SCRIPT")
DAEMON=$(extract_value "DAEMON")
USER=$(extract_value "USER")
GROUP=$(extract_value "GROUP" || echo "$USER")
PIDFILE=$(extract_value "PIDFILE")
DAEMON_OPTS=$(extract_value "DAEMON_OPTS")

# LSB headers
DESCRIPTION=$(extract_lsb "Short-Description")
REQUIRED_START=$(extract_lsb "Required-Start")

# Detect if forking
FORKING=$(grep -q "\-\-background\|\-\-daemon" "$INPUT_SCRIPT" && echo "simple" || echo "forking")

# Map dependencies
AFTER="network.target"
[[ "$REQUIRED_START" =~ "\$remote_fs" ]] && AFTER="$AFTER remote-fs.target"
[[ "$REQUIRED_START" =~ "mysql" ]] && AFTER="$AFTER mysql.service"
[[ "$REQUIRED_START" =~ "postgresql" ]] && AFTER="$AFTER postgresql.service"

# Generate unit file
OUTPUT_FILE="$OUTPUT_DIR/${NAME}.service"

cat > "$OUTPUT_FILE" << EOF
# Converted from: $INPUT_SCRIPT
# Generated: $(date)

[Unit]
Description=${DESCRIPTION:-$NAME Service}
After=$AFTER

[Service]
Type=simple
EOF

[[ -n "$USER" ]] && echo "User=$USER" >> "$OUTPUT_FILE"
[[ -n "$GROUP" ]] && echo "Group=$GROUP" >> "$OUTPUT_FILE"

cat >> "$OUTPUT_FILE" << EOF
ExecStart=$DAEMON $DAEMON_OPTS
Restart=on-failure
RestartSec=5
StandardOutput=journal
StandardError=journal
SyslogIdentifier=$NAME

[Install]
WantedBy=multi-user.target
EOF

echo "Created: $OUTPUT_FILE"
echo
echo "Next steps:"
echo "1. Review and edit: sudo vim $OUTPUT_FILE"
echo "2. Reload systemd: sudo systemctl daemon-reload"
echo "3. Disable old init: sudo update-rc.d $NAME disable"
echo "4. Enable new service: sudo systemctl enable --now $NAME"
```

### Conversion Checklist

```bash
#!/bin/bash
# conversion_checklist.sh

SERVICE_NAME="$1"

echo "=== Init to systemd Conversion Checklist ==="
echo

echo "1. PRE-CONVERSION"
echo "   [ ] Document current init script behavior"
echo "   [ ] Note any custom start/stop logic"
echo "   [ ] Identify environment variables used"
echo "   [ ] Check for PID file usage"
echo "   [ ] List dependencies"
echo

echo "2. CREATE SERVICE"
echo "   [ ] Create /etc/systemd/system/${SERVICE_NAME}.service"
echo "   [ ] Set correct Type= (simple/forking/oneshot)"
echo "   [ ] Configure User= and Group="
echo "   [ ] Set ExecStart= with full path"
echo "   [ ] Add EnvironmentFile= if needed"
echo "   [ ] Configure After= and Wants="
echo

echo "3. VALIDATE"
echo "   [ ] systemd-analyze verify ${SERVICE_NAME}.service"
echo "   [ ] systemctl daemon-reload"
echo "   [ ] systemctl start ${SERVICE_NAME}"
echo "   [ ] systemctl status ${SERVICE_NAME}"
echo "   [ ] journalctl -u ${SERVICE_NAME}"
echo

echo "4. MIGRATE"
echo "   [ ] Stop old service: service ${SERVICE_NAME} stop"
echo "   [ ] Disable old service: update-rc.d ${SERVICE_NAME} disable"
echo "   [ ] Enable new service: systemctl enable ${SERVICE_NAME}"
echo "   [ ] Test restart: systemctl restart ${SERVICE_NAME}"
echo "   [ ] Test auto-restart: kill the process"
echo

echo "5. CLEANUP"
echo "   [ ] Backup and remove old init script"
echo "   [ ] Update deployment scripts"
echo "   [ ] Update monitoring"
echo "   [ ] Document changes"
```

---

## Quick Reference Cheat Sheet

### Unit File Template

```ini
[Unit]
Description=My Service
After=network.target
Wants=dependency.service

[Service]
Type=simple
User=serviceuser
WorkingDirectory=/opt/app
EnvironmentFile=/etc/app/env
ExecStart=/opt/app/bin/server
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### Timer File Template

```ini
[Unit]
Description=My Timer

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true
RandomizedDelaySec=300

[Install]
WantedBy=timers.target
```

### Essential Commands

```bash
# After any unit file change
sudo systemctl daemon-reload

# Service management
sudo systemctl start|stop|restart|reload SERVICE
sudo systemctl enable|disable SERVICE
sudo systemctl enable --now SERVICE  # Enable AND start

# Inspection
systemctl status SERVICE
systemctl show SERVICE
systemctl cat SERVICE
journalctl -u SERVICE -f

# Debugging
systemctl list-units --failed
systemd-analyze verify /path/to/unit
systemd-analyze blame
systemd-analyze critical-chain SERVICE
```

---

## Common Mistakes & Fixes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Forgot `daemon-reload` | Changes not applied | `systemctl daemon-reload` |
| Wrong `Type=` | Service starts/stops incorrectly | Match Type to process behavior |
| Background in ExecStart | Exits immediately (Type=simple) | Remove `&` from command |
| Relative path in ExecStart | 203/EXEC error | Use absolute paths |
| Missing executable permission | 203/EXEC error | `chmod +x /path/to/binary` |
| Wrong user permissions | Permission denied | Check file ownership |
| Circular dependencies | Deadlock at boot | Audit After=/Requires= chain |
| EnvironmentFile not found | Service fails | Use `-` prefix for optional files |

### Debugging 203/EXEC Errors

```bash
# Error 203 = Cannot execute

# Check 1: File exists?
ls -la /path/to/binary

# Check 2: Executable?
file /path/to/binary

# Check 3: Architecture match?
uname -m
file /path/to/binary

# Check 4: Dependencies available?
ldd /path/to/binary

# Check 5: Can user execute?
sudo -u serviceuser /path/to/binary --version

# Check 6: SELinux/AppArmor?
getenforce
aa-status
```

---

## Mini Challenges

### Challenge 1: Basic Service (5 min)
Create a service that runs `python3 -m http.server 8080` in `/var/www/html`.

<details>
<summary>Solution</summary>

```ini
# /etc/systemd/system/simple-http.service
[Unit]
Description=Simple HTTP Server
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/var/www/html
ExecStart=/usr/bin/python3 -m http.server 8080
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
</details>

### Challenge 2: Service with Health Check (5 min)
Add watchdog and restart limits to Challenge 1.

<details>
<summary>Solution</summary>

```ini
[Service]
Type=simple
User=www-data
WorkingDirectory=/var/www/html
ExecStart=/usr/bin/python3 -m http.server 8080

# Health check script
ExecStartPost=/bin/sh -c 'sleep 2 && curl -sf http://localhost:8080/ || exit 1'

Restart=on-failure
RestartSec=5
StartLimitBurst=5
StartLimitIntervalSec=300
```
</details>

### Challenge 3: Timer Task (5 min)
Create a timer that logs "Heartbeat" every 5 minutes.

<details>
<summary>Solution</summary>

```ini
# /etc/systemd/system/heartbeat.service
[Unit]
Description=Heartbeat Logger

[Service]
Type=oneshot
ExecStart=/usr/bin/logger "Heartbeat at $(date)"
```

```ini
# /etc/systemd/system/heartbeat.timer
[Unit]
Description=Heartbeat Timer

[Timer]
OnCalendar=*:0/5
Persistent=true

[Install]
WantedBy=timers.target
```
</details>

---

## Final Test: Complete Conversion

Convert this init script to systemd with health checks and a timer:

```bash
#!/bin/bash
# /etc/init.d/webapp
source /etc/default/webapp
cd /var/www/webapp
ulimit -n 4096
su - webuser -c "node server.js >> /var/log/webapp.log 2>&1 &"
echo $! > /var/run/webapp.pid
```

### Expected Solution

```ini
# /etc/systemd/system/webapp.service
[Unit]
Description=WebApp Node.js Server
After=network.target
Documentation=file:///var/www/webapp/README.md

[Service]
Type=simple
User=webuser
Group=webuser
WorkingDirectory=/var/www/webapp

# Environment (replaces source /etc/default/webapp)
EnvironmentFile=-/etc/default/webapp

# Process (no backgrounding, no pidfile)
ExecStart=/usr/bin/node server.js

# Health check
ExecStartPost=/bin/sh -c 'sleep 5 && curl -sf http://localhost:3000/health'

# Logging (replaces >> /var/log/webapp.log)
StandardOutput=journal
StandardError=journal
SyslogIdentifier=webapp

# Resource limits (replaces ulimit)
LimitNOFILE=4096

# Restart policy
Restart=on-failure
RestartSec=5
StartLimitBurst=5
StartLimitIntervalSec=300

# Security
NoNewPrivileges=true
ProtectSystem=strict
ReadWritePaths=/var/www/webapp/uploads

[Install]
WantedBy=multi-user.target
```

```ini
# /etc/systemd/system/webapp-cleanup.timer
[Unit]
Description=WebApp Daily Cleanup

[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

---

## Time-Based Learning Plan

| Time | Activity | Checkpoint |
|------|----------|------------|
| 0:00-0:20 | Part 1-2: Fundamentals & Service Anatomy | Create basic service |
| 0:20-0:35 | Part 3: Dependencies | Multi-service stack |
| 0:35-0:55 | Part 4: Health Checks | Watchdog working |
| 0:55-1:10 | Part 5: Timers | Replace a cron job |
| 1:10-1:25 | Part 6: Conversion | Convert init script |
| 1:25-1:30 | Final Test | All challenges complete |

---

## Key Takeaways

1. **Always use `/etc/systemd/system/`** for custom services
2. **Run `daemon-reload`** after any unit file change
3. **Match `Type=`** to how your process behaves
4. **Use absolute paths** in ExecStart
5. **Leverage journald** instead of custom log files
6. **Implement health checks** for production services
7. **Replace cron with timers** for better integration
8. **Test restart behavior** before going to production