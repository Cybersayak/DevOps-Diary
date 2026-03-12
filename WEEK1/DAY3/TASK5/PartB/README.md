# GOAL 5: Production-Ready Automation Scripts - TASK B

## Log Analysis & Alerting System (2.5 hrs)

***

# 📋 TABLE OF CONTENTS

1. [Concept Foundation](#section-1)
2. [Log Format Deep Dive](#section-2)
3. [Real-Time Parsing Engine](#section-3)
4. [Anomaly Detection System](#section-4)
5. [Alert Escalation Framework](#section-5)
6. [Executive Reporting](#section-6)
7. [Monitoring Integration](#section-7)
8. [Complete Production System](#section-8)
9. [Mini Challenges & Final Test](#section-9)
10. [Time-Based Learning Plan](#section-10)

***

<a name="section-1"></a>

# 1. CONCEPT FOUNDATION (15 minutes)

## 1.1 What is Log Analysis?

**Definition**: Log analysis is the systematic process of collecting, parsing, correlating, and interpreting log data to identify patterns, anomalies, security threats, and operational issues.

### The Log Analysis Pipeline

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  COLLECT    │───▶│   PARSE     │───▶│  ANALYZE    │───▶│   ALERT     │───▶│   REPORT    │
│  (gather)   │    │ (structure) │    │  (detect)   │    │  (notify)   │    │ (summarize) │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
     │                   │                  │                  │                  │
     ▼                   ▼                  ▼                  ▼                  ▼
  syslog            regex/awk          statistics         email/slack        PDF/HTML
  journald          grok patterns      ML models          PagerDuty          dashboards
  application       JSON parsing       thresholds         webhooks           metrics
```

### Why Log Analysis Matters in DevOps

| Scenario          | Without Analysis      | With Analysis       |
| ----------------- | --------------------- | ------------------- |
| Security Breach   | Discovered days later | Detected in seconds |
| Performance Issue | Users complain        | Proactive fix       |
| Disk Full         | System crash          | Early warning       |
| Attack Pattern    | Unnoticed             | Blocked immediately |
| Compliance Audit  | Manual review         | Automated reports   |

## 1.2 Log Severity Levels (RFC 5424)

```bash
#!/bin/bash
# Understanding syslog severity levels

declare -A SEVERITY_LEVELS=(
    [0]="EMERGENCY"   # System is unusable
    [1]="ALERT"       # Action must be taken immediately
    [2]="CRITICAL"    # Critical conditions
    [3]="ERROR"       # Error conditions
    [4]="WARNING"     # Warning conditions
    [5]="NOTICE"      # Normal but significant
    [6]="INFO"        # Informational messages
    [7]="DEBUG"       # Debug-level messages
)

# Color coding for terminal output
declare -A SEVERITY_COLORS=(
    [EMERGENCY]="\033[1;97;41m"  # White on Red (blinking)
    [ALERT]="\033[1;91m"          # Bright Red
    [CRITICAL]="\033[1;31m"       # Red
    [ERROR]="\033[0;31m"          # Dark Red
    [WARNING]="\033[1;33m"        # Yellow
    [NOTICE]="\033[1;36m"         # Cyan
    [INFO]="\033[0;32m"           # Green
    [DEBUG]="\033[0;37m"          # Gray
)
RESET="\033[0m"

# Display severity guide
echo "=== SYSLOG SEVERITY LEVELS ==="
for level in {0..7}; do
    severity="${SEVERITY_LEVELS[$level]}"
    color="${SEVERITY_COLORS[$severity]}"
    printf "${color}Level %d: %-12s${RESET}\n" "$level" "$severity"
done
```

**Expected Output:**

```
=== SYSLOG SEVERITY LEVELS ===
Level 0: EMERGENCY   (displayed in white on red)
Level 1: ALERT       (displayed in bright red)
Level 2: CRITICAL    (displayed in red)
Level 3: ERROR       (displayed in dark red)
Level 4: WARNING     (displayed in yellow)
Level 5: NOTICE      (displayed in cyan)
Level 6: INFO        (displayed in green)
Level 7: DEBUG       (displayed in gray)
```

***

<a name="section-2"></a>

# 2. LOG FORMAT DEEP DIVE (20 minutes)

## 2.1 Common Log Formats Explained

### Syslog Format (RFC 3164 & RFC 5424)

```bash
#!/bin/bash
# Syslog format parser with detailed breakdown

# RFC 3164 (BSD Syslog) Example:
# <34>Oct 11 22:14:15 mymachine su: 'su root' failed for user1 on /dev/pts/8

# RFC 5424 (Modern Syslog) Example:
# <165>1 2023-10-11T22:14:15.003Z mymachine evntslog - ID47 [exampleSDID@32473 iut="3"] Application startup

parse_syslog_rfc3164() {
    local line="$1"
    
    # Priority extraction: <PRI> where PRI = (Facility * 8) + Severity
    if [[ "$line" =~ ^\<([0-9]+)\>(.*)$ ]]; then
        local pri="${BASH_REMATCH[1]}"
        local message="${BASH_REMATCH[2]}"
        
        # Calculate facility and severity
        local facility=$((pri / 8))
        local severity=$((pri % 8))
        
        # Facility names
        local facilities=(
            "kern" "user" "mail" "daemon" "auth" "syslog" 
            "lpr" "news" "uucp" "cron" "authpriv" "ftp"
            "ntp" "audit" "alert" "clock" 
            "local0" "local1" "local2" "local3" 
            "local4" "local5" "local6" "local7"
        )
        
        echo "Priority: $pri"
        echo "Facility: ${facilities[$facility]} ($facility)"
        echo "Severity: $severity"
        echo "Message: $message"
    fi
}

# Test the parser
echo "=== RFC 3164 Syslog Parsing ==="
parse_syslog_rfc3164 "<34>Oct 11 22:14:15 server01 sshd[1234]: Failed password for root"

# Output:
# Priority: 34
# Facility: auth (4)
# Severity: 2 (CRITICAL)
# Message: Oct 11 22:14:15 server01 sshd[1234]: Failed password for root
```

### Apache/Nginx Access Log (Combined Format)

```bash
#!/bin/bash
# Apache/Nginx Combined Log Format Parser

# Format: %h %l %u %t "%r" %>s %b "%{Referer}i" "%{User-Agent}i"
# Example: 192.168.1.100 - john [10/Oct/2023:13:55:36 -0700] "GET /api/users HTTP/1.1" 200 2326 "https://example.com" "Mozilla/5.0"

parse_apache_combined() {
    local line="$1"
    
    # Regex pattern breakdown:
    # ^([0-9.]+)           - IP address
    # \s+-\s+              - Identity (usually -)
    # (\S+)                - User (or -)
    # \s+\[([^\]]+)\]      - Timestamp in brackets
    # \s+"([^"]+)"         - Request line in quotes
    # \s+(\d+)             - Status code
    # \s+(\d+|-)           - Bytes sent
    # \s+"([^"]*)"         - Referer
    # \s+"([^"]*)"         - User agent
    
    local regex='^([0-9.]+) - (\S+) \[([^\]]+)\] "([^"]+)" ([0-9]+) ([0-9-]+) "([^"]*)" "([^"]*)"'
    
    if [[ "$line" =~ $regex ]]; then
        echo "IP Address: ${BASH_REMATCH[1]}"
        echo "User: ${BASH_REMATCH[2]}"
        echo "Timestamp: ${BASH_REMATCH[3]}"
        echo "Request: ${BASH_REMATCH[4]}"
        echo "Status: ${BASH_REMATCH[5]}"
        echo "Bytes: ${BASH_REMATCH[6]}"
        echo "Referer: ${BASH_REMATCH[7]}"
        echo "User-Agent: ${BASH_REMATCH[8]}"
        
        # Parse request line
        local request="${BASH_REMATCH[4]}"
        if [[ "$request" =~ ^([A-Z]+)\ (.*)\ HTTP ]]; then
            echo "  Method: ${BASH_REMATCH[1]}"
            echo "  Path: ${BASH_REMATCH[2]}"
        fi
    else
        echo "ERROR: Could not parse line"
    fi
}

# Test
test_line='192.168.1.100 - john [10/Oct/2023:13:55:36 -0700] "GET /api/users HTTP/1.1" 200 2326 "https://example.com" "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"'
echo "=== Apache Combined Log Parsing ==="
parse_apache_combined "$test_line"
```

**Expected Output:**

```
=== Apache Combined Log Parsing ===
IP Address: 192.168.1.100
User: john
Timestamp: 10/Oct/2023:13:55:36 -0700
Request: GET /api/users HTTP/1.1
Status: 200
Bytes: 2326
Referer: https://example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
  Method: GET
  Path: /api/users
```

### JSON Log Format (Modern Applications)

```bash
#!/bin/bash
# JSON Log Parser using jq

# Example JSON log:
# {"timestamp":"2023-10-11T14:30:00Z","level":"ERROR","service":"api-gateway","message":"Connection timeout","details":{"host":"db-primary","port":5432,"timeout_ms":30000}}

parse_json_log() {
    local line="$1"
    
    # Validate JSON first
    if ! echo "$line" | jq -e . >/dev/null 2>&1; then
        echo "ERROR: Invalid JSON"
        return 1
    fi
    
    # Extract fields
    local timestamp=$(echo "$line" | jq -r '.timestamp // "N/A"')
    local level=$(echo "$line" | jq -r '.level // "INFO"')
    local service=$(echo "$line" | jq -r '.service // "unknown"')
    local message=$(echo "$line" | jq -r '.message // ""')
    
    # Check for nested details
    local has_details=$(echo "$line" | jq 'has("details")')
    
    echo "Timestamp: $timestamp"
    echo "Level: $level"
    echo "Service: $service"
    echo "Message: $message"
    
    if [[ "$has_details" == "true" ]]; then
        echo "Details:"
        echo "$line" | jq -r '.details | to_entries[] | "  \(.key): \(.value)"'
    fi
}

# Alternative: Pure Bash JSON parsing for simple cases (no jq dependency)
parse_simple_json() {
    local json="$1"
    
    # Extract values using grep and sed
    # WARNING: This only works for simple, non-nested JSON
    local timestamp=$(echo "$json" | grep -oP '"timestamp"\s*:\s*"\K[^"]+')
    local level=$(echo "$json" | grep -oP '"level"\s*:\s*"\K[^"]+')
    local message=$(echo "$json" | grep -oP '"message"\s*:\s*"\K[^"]+')
    
    echo "Timestamp: ${timestamp:-N/A}"
    echo "Level: ${level:-INFO}"
    echo "Message: ${message:-}"
}

# Test
json_log='{"timestamp":"2023-10-11T14:30:00Z","level":"ERROR","service":"api-gateway","message":"Connection timeout","details":{"host":"db-primary","port":5432}}'
echo "=== JSON Log Parsing (with jq) ==="
parse_json_log "$json_log"
```

## 2.2 Multi-Format Log Parser

```bash
#!/bin/bash
# Universal log format detector and parser

detect_and_parse_log() {
    local line="$1"
    
    # Format detection patterns
    local json_pattern='^\s*\{'
    local syslog_pattern='^\<[0-9]+\>'
    local apache_pattern='^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+ .* \[.*\] ".*"'
    local nginx_pattern='^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+ - .* \[.*\]'
    local journald_pattern='^[A-Z][a-z]{2} [0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2}'
    local iso_timestamp='^[0-9]{4}-[0-9]{2}-[0-9]{2}[T ][0-9]{2}:[0-9]{2}:[0-9]{2}'
    
    # Detect format
    if [[ "$line" =~ $json_pattern ]]; then
        echo "FORMAT: JSON"
        echo "$line" | jq -r 'to_entries[] | "\(.key): \(.value)"' 2>/dev/null || echo "Invalid JSON"
        
    elif [[ "$line" =~ $syslog_pattern ]]; then
        echo "FORMAT: Syslog (RFC 3164/5424)"
        parse_syslog_rfc3164 "$line"
        
    elif [[ "$line" =~ $apache_pattern ]] || [[ "$line" =~ $nginx_pattern ]]; then
        echo "FORMAT: Apache/Nginx Access Log"
        parse_apache_combined "$line"
        
    elif [[ "$line" =~ $journald_pattern ]]; then
        echo "FORMAT: Systemd Journal"
        parse_journald_line "$line"
        
    elif [[ "$line" =~ $iso_timestamp ]]; then
        echo "FORMAT: ISO Timestamp (Generic Application Log)"
        parse_generic_iso "$line"
        
    else
        echo "FORMAT: Unknown/Plain Text"
        echo "Content: $line"
    fi
}

parse_journald_line() {
    local line="$1"
    # Oct 11 14:30:00 hostname service[pid]: message
    if [[ "$line" =~ ^([A-Z][a-z]{2}\ [0-9]{2}\ [0-9:]+)\ ([^\ ]+)\ ([^\[]+)\[([0-9]+)\]:\ (.*)$ ]]; then
        echo "Timestamp: ${BASH_REMATCH[1]}"
        echo "Hostname: ${BASH_REMATCH[2]}"
        echo "Service: ${BASH_REMATCH[3]}"
        echo "PID: ${BASH_REMATCH[4]}"
        echo "Message: ${BASH_REMATCH[5]}"
    fi
}

parse_generic_iso() {
    local line="$1"
    # 2023-10-11 14:30:00 [LEVEL] [component] message
    if [[ "$line" =~ ^([0-9]{4}-[0-9]{2}-[0-9]{2}[T\ ][0-9:]+)\ *\[?([A-Z]+)\]?\ *\[?([^\]]*)\]?\ *(.*)$ ]]; then
        echo "Timestamp: ${BASH_REMATCH[1]}"
        echo "Level: ${BASH_REMATCH[2]}"
        echo "Component: ${BASH_REMATCH[3]}"
        echo "Message: ${BASH_REMATCH[4]}"
    fi
}

# Test with multiple formats
echo "=========================================="
echo "MULTI-FORMAT LOG DETECTION TEST"
echo "=========================================="

test_logs=(
    '{"timestamp":"2023-10-11T14:30:00Z","level":"ERROR","message":"Test"}'
    '<34>Oct 11 22:14:15 mymachine su: su failed for user1'
    '192.168.1.100 - - [10/Oct/2023:13:55:36 -0700] "GET / HTTP/1.1" 200 2326 "-" "curl/7.68.0"'
    'Oct 11 14:30:00 webserver nginx[1234]: Connection accepted'
    '2023-10-11 14:30:00 [ERROR] [database] Connection refused'
)

for log in "${test_logs[@]}"; do
    echo ""
    echo "Input: $log"
    echo "---"
    detect_and_parse_log "$log"
    echo ""
done
```

***

<a name="section-3"></a>

# 3. REAL-TIME PARSING ENGINE (25 minutes)

## 3.1 Stream Processing Fundamentals

### Why Real-Time Processing?

```
BATCH PROCESSING:                    REAL-TIME PROCESSING:
┌─────────────────┐                  ┌─────────────────┐
│  Collect logs   │                  │  Log generated  │
│  for 1 hour     │                  │        │        │
│        │        │                  │        ▼        │
│        ▼        │                  │  Process <1s    │
│  Process batch  │                  │        │        │
│        │        │                  │        ▼        │
│        ▼        │                  │  Alert <5s      │
│  Alert (delay!) │                  └─────────────────┘
└─────────────────┘

Time to detect issue:               Time to detect issue:
~30-60 minutes                      ~1-5 seconds
```

## 3.2 Tail-Based Real-Time Parser

```bash
#!/bin/bash
# Real-time log parsing engine with buffering and pattern matching

# Configuration
readonly CONFIG_FILE="/etc/log_analyzer/config.conf"
readonly STATE_DIR="/var/lib/log_analyzer"
readonly BUFFER_SIZE=100  # Lines to buffer before processing
readonly FLUSH_INTERVAL=5 # Seconds

# Initialize
declare -A LOG_SOURCES
declare -A PATTERNS
declare -a BUFFER

# Define log sources to monitor
LOG_SOURCES=(
    [syslog]="/var/log/syslog"
    [auth]="/var/log/auth.log"
    [nginx_access]="/var/log/nginx/access.log"
    [nginx_error]="/var/log/nginx/error.log"
    [app]="/var/log/myapp/application.log"
)

# Define patterns to detect
PATTERNS=(
    [failed_login]='[Ff]ailed (password|login)|authentication failure'
    [ssh_brute]='Failed password.*ssh'
    [error_spike]='\b(ERROR|FATAL|CRITICAL)\b'
    [sql_injection]='(UNION|SELECT|INSERT|DELETE|DROP|UPDATE).*--'
    [path_traversal]='\.\./|%2e%2e'
    [high_response_time]='response_time[=:]([5-9][0-9]{3}|[0-9]{5,})'
    [disk_full]='No space left on device|disk.*(full|100%)'
    [oom_killer]='Out of memory|oom-killer|Killed process'
    [service_down]='Connection refused|service.*(down|unavailable|failed)'
)

# Real-time log follower
follow_log() {
    local source_name="$1"
    local log_path="$2"
    
    # Check if file exists
    [[ -f "$log_path" ]] || {
        echo "WARNING: Log file not found: $log_path" >&2
        return 1
    }
    
    # Track file position for resume capability
    local state_file="${STATE_DIR}/${source_name}.pos"
    local start_pos=0
    
    if [[ -f "$state_file" ]]; then
        start_pos=$(cat "$state_file" 2>/dev/null || echo 0)
    fi
    
    # Use tail with follow, handling log rotation
    tail -F -n +$((start_pos + 1)) "$log_path" 2>/dev/null | while read -r line; do
        # Add source tag
        echo "${source_name}|$(date +%s.%N)|${line}"
        
        # Update position (approximate - for exact, use inotify)
        ((start_pos++))
        echo "$start_pos" > "$state_file"
    done
}

# Multi-source aggregator using named pipes
setup_aggregator() {
    local fifo="/tmp/log_aggregator_$$"
    mkfifo "$fifo" 2>/dev/null || true
    
    # Start followers for each source
    for source in "${!LOG_SOURCES[@]}"; do
        follow_log "$source" "${LOG_SOURCES[$source]}" >> "$fifo" &
    done
    
    # Process aggregated stream
    process_stream < "$fifo"
    
    # Cleanup
    rm -f "$fifo"
}

# Stream processor with pattern matching
process_stream() {
    local line_count=0
    local last_flush=$(date +%s)
    
    while IFS='|' read -r source timestamp content; do
        # Parse timestamp
        local event_time=$(date -d "@${timestamp%.*}" "+%Y-%m-%d %H:%M:%S" 2>/dev/null || echo "$timestamp")
        
        # Apply all patterns
        local matched=false
        local matches=()
        
        for pattern_name in "${!PATTERNS[@]}"; do
            if [[ "$content" =~ ${PATTERNS[$pattern_name]} ]]; then
                matched=true
                matches+=("$pattern_name")
            fi
        done
        
        # If matched, process immediately
        if [[ "$matched" == true ]]; then
            process_match "$source" "$event_time" "$content" "${matches[*]}"
        fi
        
        # Buffer for batch processing
        BUFFER+=("$source|$event_time|$content|${matches[*]}")
        ((line_count++))
        
        # Flush buffer if needed
        local current_time=$(date +%s)
        if ((line_count >= BUFFER_SIZE)) || ((current_time - last_flush >= FLUSH_INTERVAL)); then
            flush_buffer
            BUFFER=()
            line_count=0
            last_flush=$current_time
        fi
    done
}

# Process matched patterns
process_match() {
    local source="$1"
    local timestamp="$2"
    local content="$3"
    local patterns="$4"
    
    # Determine severity based on pattern
    local severity="INFO"
    for pattern in $patterns; do
        case "$pattern" in
            failed_login|ssh_brute)
                severity="WARNING"
                ;;
            sql_injection|path_traversal)
                severity="CRITICAL"
                ;;
            error_spike|oom_killer|service_down)
                severity="ERROR"
                ;;
            disk_full)
                severity="ALERT"
                ;;
        esac
    done
    
    # Log the detection
    log_detection "$severity" "$source" "$timestamp" "$patterns" "$content"
    
    # Trigger appropriate actions
    if [[ "$severity" == "CRITICAL" ]] || [[ "$severity" == "ALERT" ]]; then
        trigger_immediate_alert "$severity" "$source" "$patterns" "$content"
    fi
}

log_detection() {
    local severity="$1"
    local source="$2"
    local timestamp="$3"
    local patterns="$4"
    local content="$5"
    
    local log_line="[$(date '+%Y-%m-%d %H:%M:%S')] [$severity] Source: $source | Patterns: $patterns | Content: ${content:0:200}"
    echo "$log_line" >> /var/log/log_analyzer/detections.log
    
    # Also output to stdout for debugging
    echo "$log_line"
}

# Flush buffer for batch analysis
flush_buffer() {
    [[ ${#BUFFER[@]} -eq 0 ]] && return
    
    # Analyze buffer for patterns
    analyze_buffer_stats
}

analyze_buffer_stats() {
    local error_count=0
    local warning_count=0
    
    for entry in "${BUFFER[@]}"; do
        IFS='|' read -r source timestamp content patterns <<< "$entry"
        [[ "$patterns" == *"error_spike"* ]] && ((error_count++))
        [[ "$patterns" == *"warning"* ]] && ((warning_count++))
    done
    
    # Report buffer statistics
    local total=${#BUFFER[@]}
    if ((error_count > total / 10)); then
        echo "WARNING: High error rate detected: $error_count/$total ($(( error_count * 100 / total ))%)"
    fi
}

trigger_immediate_alert() {
    local severity="$1"
    local source="$2"
    local patterns="$3"
    local content="$4"
    
    # This will be expanded in the alerting section
    echo "IMMEDIATE ALERT: [$severity] $patterns detected in $source" >&2
}
```

## 3.3 High-Performance AWK Parser

```bash
#!/bin/bash
# AWK-based high-performance log parser
# AWK is significantly faster than bash for text processing

create_awk_parser() {
    cat << 'AWK_PARSER'
#!/usr/bin/awk -f

# Initialize counters and arrays
BEGIN {
    # Define severity weights
    severity["EMERGENCY"] = 0
    severity["ALERT"] = 1
    severity["CRITICAL"] = 2
    severity["ERROR"] = 3
    severity["WARNING"] = 4
    severity["NOTICE"] = 5
    severity["INFO"] = 6
    severity["DEBUG"] = 7
    
    # Counters
    total_lines = 0
    error_count = 0
    warning_count = 0
    
    # Time window for rate calculation (seconds)
    time_window = 60
    
    # Detection patterns
    patterns["FAILED_LOGIN"] = "([Ff]ailed|[Ii]nvalid).*(password|login|auth)"
    patterns["BRUTE_FORCE"] = "Failed password.*from.*repeated"
    patterns["SQL_INJECTION"] = "(UNION|SELECT|INSERT|DELETE|DROP|--)"
    patterns["XSS_ATTEMPT"] = "(<script|javascript:|onerror=)"
    patterns["ERROR"] = "\\b(ERROR|FATAL|CRITICAL|SEVERE)\\b"
    patterns["EXCEPTION"] = "(Exception|Traceback|Stack trace)"
    patterns["DISK_ISSUE"] = "(No space|disk full|I/O error)"
    patterns["MEMORY_ISSUE"] = "(OOM|Out of memory|Cannot allocate)"
    patterns["CONNECTION"] = "(Connection refused|timed out|reset by peer)"
    
    # Output format
    OFS = "|"
}

# Function to extract timestamp from various formats
function extract_timestamp(line) {
    # ISO 8601: 2023-10-11T14:30:00
    if (match(line, /[0-9]{4}-[0-9]{2}-[0-9]{2}[T ][0-9]{2}:[0-9]{2}:[0-9]{2}/)) {
        return substr(line, RSTART, RLENGTH)
    }
    # Syslog: Oct 11 14:30:00
    if (match(line, /[A-Z][a-z]{2} [ 0-9][0-9] [0-9]{2}:[0-9]{2}:[0-9]{2}/)) {
        return substr(line, RSTART, RLENGTH)
    }
    # Unix timestamp
    if (match(line, /[0-9]{10}/)) {
        return substr(line, RSTART, RLENGTH)
    }
    return "UNKNOWN"
}

# Function to extract IP address
function extract_ip(line) {
    if (match(line, /[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/)) {
        return substr(line, RSTART, RLENGTH)
    }
    return ""
}

# Function to determine severity
function get_severity(line) {
    if (line ~ /EMERGENCY|EMERG/) return "EMERGENCY"
    if (line ~ /ALERT/) return "ALERT"
    if (line ~ /CRITICAL|CRIT/) return "CRITICAL"
    if (line ~ /\bERROR\b|FATAL|SEVERE/) return "ERROR"
    if (line ~ /WARNING|WARN/) return "WARNING"
    if (line ~ /NOTICE/) return "NOTICE"
    if (line ~ /DEBUG/) return "DEBUG"
    return "INFO"
}

# Function to check patterns and return matches
function check_patterns(line,    matched, p) {
    matched = ""
    for (p in patterns) {
        if (line ~ patterns[p]) {
            if (matched != "") matched = matched ","
            matched = matched p
        }
    }
    return matched
}

# Main processing for each line
{
    total_lines++
    
    timestamp = extract_timestamp($0)
    ip_address = extract_ip($0)
    severity_level = get_severity($0)
    matched_patterns = check_patterns($0)
    
    # Update counters
    if (severity_level == "ERROR" || severity_level == "CRITICAL" || severity_level == "FATAL") {
        error_count++
    }
    if (severity_level == "WARNING") {
        warning_count++
    }
    
    # Track IPs for brute force detection
    if (ip_address != "") {
        ip_count[ip_address]++
        if (matched_patterns ~ /FAILED_LOGIN/) {
            failed_login_ip[ip_address]++
        }
    }
    
    # Output parsed line if it matches any pattern
    if (matched_patterns != "") {
        print timestamp, severity_level, ip_address, matched_patterns, substr($0, 1, 500)
    }
    
    # Output alerts for high-severity
    if (severity[severity_level] <= 2) {
        print "[ALERT]", timestamp, severity_level, $0 > "/dev/stderr"
    }
}

# Final summary
END {
    print "=" >> "/dev/stderr"
    print "PARSING SUMMARY:" >> "/dev/stderr"
    print "  Total lines processed: " total_lines >> "/dev/stderr"
    print "  Errors found: " error_count >> "/dev/stderr"
    print "  Warnings found: " warning_count >> "/dev/stderr"
    
    # Report potential brute force sources
    for (ip in failed_login_ip) {
        if (failed_login_ip[ip] >= 5) {
            print "  BRUTE FORCE CANDIDATE: " ip " (" failed_login_ip[ip] " failed logins)" >> "/dev/stderr"
        }
    }
}
AWK_PARSER
}

# Save AWK parser
create_awk_parser > /usr/local/bin/log_parser.awk
chmod +x /usr/local/bin/log_parser.awk

# Usage example
process_logs_with_awk() {
    local log_file="$1"
    
    # Process with AWK (much faster than bash)
    awk -f /usr/local/bin/log_parser.awk "$log_file"
}

# Real-time streaming with AWK
stream_logs_with_awk() {
    local log_file="$1"
    
    tail -F "$log_file" | awk -f /usr/local/bin/log_parser.awk
}
```

## 3.4 Parallel Processing for Multiple Logs

```bash
#!/bin/bash
# Parallel log processing using GNU parallel or background jobs

readonly MAX_PARALLEL=4
readonly WORK_DIR="/tmp/log_analysis_$$"

setup_parallel_processing() {
    mkdir -p "$WORK_DIR"
    
    # Create named pipes for inter-process communication
    mkfifo "${WORK_DIR}/aggregator_pipe"
    
    # Trap for cleanup
    trap 'cleanup_parallel' EXIT INT TERM
}

cleanup_parallel() {
    # Kill all background jobs
    jobs -p | xargs -r kill 2>/dev/null
    rm -rf "$WORK_DIR"
}

process_log_parallel() {
    local log_file="$1"
    local output_file="$2"
    
    # Use AWK for parsing
    awk -f /usr/local/bin/log_parser.awk "$log_file" > "$output_file" 2>&1
    
    # Signal completion
    echo "DONE:$log_file" >> "${WORK_DIR}/status"
}

run_parallel_analysis() {
    local log_files=("$@")
    
    setup_parallel_processing
    
    local job_count=0
    local running=0
    
    for log_file in "${log_files[@]}"; do
        # Wait if we have too many parallel jobs
        while ((running >= MAX_PARALLEL)); do
            wait -n 2>/dev/null || true
            ((running--))
        done
        
        # Start background job
        local output="${WORK_DIR}/output_${job_count}.txt"
        process_log_parallel "$log_file" "$output" &
        ((running++))
        ((job_count++))
    done
    
    # Wait for all jobs to complete
    wait
    
    # Aggregate results
    aggregate_results
}

aggregate_results() {
    echo "=== AGGREGATED RESULTS ==="
    
    local total_errors=0
    local total_warnings=0
    local total_critical=0
    
    for result in "${WORK_DIR}"/output_*.txt; do
        [[ -f "$result" ]] || continue
        
        local errors=$(grep -c "ERROR" "$result" 2>/dev/null || echo 0)
        local warnings=$(grep -c "WARNING" "$result" 2>/dev/null || echo 0)
        local critical=$(grep -c "CRITICAL" "$result" 2>/dev/null || echo 0)
        
        ((total_errors += errors))
        ((total_warnings += warnings))
        ((total_critical += critical))
    done
    
    echo "Total Errors: $total_errors"
    echo "Total Warnings: $total_warnings"
    echo "Total Critical: $total_critical"
    
    # Merge all detections
    cat "${WORK_DIR}"/output_*.txt | sort -t'|' -k1 > "${WORK_DIR}/merged_results.txt"
    echo "Merged results saved to: ${WORK_DIR}/merged_results.txt"
}
```

***

<a name="section-4"></a>

# 4. ANOMALY DETECTION SYSTEM (25 minutes)

## 4.1 Statistical Anomaly Detection

### Understanding Baselines and Thresholds

```bash
#!/bin/bash
# Statistical anomaly detection with baseline learning

readonly BASELINE_DIR="/var/lib/log_analyzer/baselines"
readonly ANOMALY_THRESHOLD=2.5  # Standard deviations

mkdir -p "$BASELINE_DIR"

# Calculate statistics for baseline
calculate_baseline() {
    local metric_name="$1"
    local values_file="$2"
    local output_file="${BASELINE_DIR}/${metric_name}.baseline"
    
    # Read values into array
    mapfile -t values < "$values_file"
    local count=${#values[@]}
    
    [[ $count -eq 0 ]] && {
        echo "ERROR: No values to calculate baseline"
        return 1
    }
    
    # Calculate mean
    local sum=0
    for val in "${values[@]}"; do
        sum=$(echo "$sum + $val" | bc -l)
    done
    local mean=$(echo "scale=4; $sum / $count" | bc -l)
    
    # Calculate standard deviation
    local sq_sum=0
    for val in "${values[@]}"; do
        local diff=$(echo "$val - $mean" | bc -l)
        local sq_diff=$(echo "$diff * $diff" | bc -l)
        sq_sum=$(echo "$sq_sum + $sq_diff" | bc -l)
    done
    local variance=$(echo "scale=4; $sq_sum / $count" | bc -l)
    local std_dev=$(echo "scale=4; sqrt($variance)" | bc -l)
    
    # Calculate percentiles
    local sorted=($(printf '%s\n' "${values[@]}" | sort -n))
    local p50_idx=$((count / 2))
    local p90_idx=$((count * 90 / 100))
    local p95_idx=$((count * 95 / 100))
    local p99_idx=$((count * 99 / 100))
    
    # Save baseline
    cat > "$output_file" << EOF
# Baseline for: $metric_name
# Generated: $(date -Iseconds)
# Sample size: $count
MEAN=$mean
STD_DEV=$std_dev
MIN=${sorted[0]}
MAX=${sorted[-1]}
P50=${sorted[$p50_idx]}
P90=${sorted[$p90_idx]}
P95=${sorted[$p95_idx]}
P99=${sorted[$p99_idx]}
UPPER_THRESHOLD=$(echo "$mean + $ANOMALY_THRESHOLD * $std_dev" | bc -l)
LOWER_THRESHOLD=$(echo "$mean - $ANOMALY_THRESHOLD * $std_dev" | bc -l)
EOF
    
    echo "Baseline saved: $output_file"
    cat "$output_file"
}

# Check if value is anomalous
check_anomaly() {
    local metric_name="$1"
    local current_value="$2"
    local baseline_file="${BASELINE_DIR}/${metric_name}.baseline"
    
    # Load baseline
    if [[ ! -f "$baseline_file" ]]; then
        echo "UNKNOWN"  # No baseline yet
        return 2
    fi
    
    source "$baseline_file"
    
    # Check thresholds
    local is_anomaly=false
    local anomaly_type=""
    local deviation=""
    
    if (( $(echo "$current_value > $UPPER_THRESHOLD" | bc -l) )); then
        is_anomaly=true
        anomaly_type="HIGH"
        deviation=$(echo "scale=2; ($current_value - $MEAN) / $STD_DEV" | bc -l)
    elif (( $(echo "$current_value < $LOWER_THRESHOLD" | bc -l) )); then
        is_anomaly=true
        anomaly_type="LOW"
        deviation=$(echo "scale=2; ($MEAN - $current_value) / $STD_DEV" | bc -l)
    fi
    
    if [[ "$is_anomaly" == true ]]; then
        echo "ANOMALY:$anomaly_type:${deviation}σ"
        return 1
    else
        echo "NORMAL"
        return 0
    fi
}

# Example: Monitor errors per minute
build_error_rate_baseline() {
    local log_file="$1"
    local output_file="${BASELINE_DIR}/error_rate_samples.txt"
    
    # Count errors per minute for the last 24 hours
    > "$output_file"
    
    # This simulates historical data collection
    # In production, you'd parse actual historical logs
    local current_time=$(date +%s)
    local day_ago=$((current_time - 86400))
    
    # Extract error counts per minute from logs
    awk -v start="$day_ago" '
    BEGIN { bucket = 0; count = 0 }
    {
        # Extract timestamp and check if ERROR
        if (/ERROR|CRITICAL|FATAL/) {
            # Get minute bucket
            if (match($0, /[0-9]{2}:[0-9]{2}/)) {
                minute = substr($0, RSTART, RLENGTH)
                if (minute != last_minute) {
                    if (last_minute != "") print count
                    count = 1
                    last_minute = minute
                } else {
                    count++
                }
            }
        }
    }
    END { print count }
    ' "$log_file" >> "$output_file"
    
    # Calculate baseline from samples
    calculate_baseline "error_rate" "$output_file"
}
```

## 4.2 Time-Series Anomaly Detection

```bash
#!/bin/bash
# Time-series based anomaly detection with sliding windows

declare -A SLIDING_WINDOW
readonly WINDOW_SIZE=60  # 60 data points
readonly SPIKE_THRESHOLD=3  # Standard deviations for spike detection

# Initialize time series tracking
init_time_series() {
    local metric_name="$1"
    SLIDING_WINDOW["${metric_name}_data"]=""
    SLIDING_WINDOW["${metric_name}_timestamps"]=""
    SLIDING_WINDOW["${metric_name}_count"]=0
}

# Add data point to sliding window
add_data_point() {
    local metric_name="$1"
    local value="$2"
    local timestamp="${3:-$(date +%s)}"
    
    local data_key="${metric_name}_data"
    local ts_key="${metric_name}_timestamps"
    local count_key="${metric_name}_count"
    
    # Append new value
    SLIDING_WINDOW[$data_key]="${SLIDING_WINDOW[$data_key]} $value"
    SLIDING_WINDOW[$ts_key]="${SLIDING_WINDOW[$ts_key]} $timestamp"
    ((SLIDING_WINDOW[$count_key]++))
    
    # Trim if window exceeded
    if ((SLIDING_WINDOW[$count_key] > WINDOW_SIZE)); then
        # Remove oldest value
        SLIDING_WINDOW[$data_key]=$(echo "${SLIDING_WINDOW[$data_key]}" | cut -d' ' -f2-)
        SLIDING_WINDOW[$ts_key]=$(echo "${SLIDING_WINDOW[$ts_key]}" | cut -d' ' -f2-)
        SLIDING_WINDOW[$count_key]=$WINDOW_SIZE
    fi
}

# Detect anomalies in sliding window
detect_window_anomaly() {
    local metric_name="$1"
    local data_key="${metric_name}_data"
    local count=${SLIDING_WINDOW["${metric_name}_count"]}
    
    [[ $count -lt 10 ]] && return 2  # Need minimum data
    
    # Get data as array
    read -ra data_points <<< "${SLIDING_WINDOW[$data_key]}"
    
    # Calculate statistics
    local sum=0
    for val in "${data_points[@]}"; do
        sum=$(echo "$sum + $val" | bc -l 2>/dev/null || echo "$sum")
    done
    local mean=$(echo "scale=4; $sum / $count" | bc -l)
    
    # Calculate standard deviation
    local sq_sum=0
    for val in "${data_points[@]}"; do
        local diff=$(echo "$val - $mean" | bc -l)
        sq_sum=$(echo "$sq_sum + $diff * $diff" | bc -l)
    done
    local std_dev=$(echo "scale=4; sqrt($sq_sum / $count)" | bc -l)
    
    # Check latest value
    local latest="${data_points[-1]}"
    
    # Avoid division by zero
    if (( $(echo "$std_dev == 0" | bc -l) )); then
        echo "STABLE"
        return 0
    fi
    
    local z_score=$(echo "scale=4; ($latest - $mean) / $std_dev" | bc -l)
    local abs_z=$(echo "$z_score" | tr -d '-')
    
    if (( $(echo "$abs_z > $SPIKE_THRESHOLD" | bc -l) )); then
        local direction="SPIKE"
        (( $(echo "$z_score < 0" | bc -l) )) && direction="DROP"
        echo "ANOMALY:$direction:z=${z_score}:mean=${mean}:std=${std_dev}:value=${latest}"
        return 1
    else
        echo "NORMAL:z=${z_score}:mean=${mean}:value=${latest}"
        return 0
    fi
}

# Trend detection
detect_trend() {
    local metric_name="$1"
    local data_key="${metric_name}_data"
    local count=${SLIDING_WINDOW["${metric_name}_count"]}
    
    [[ $count -lt 10 ]] && return 2
    
    read -ra data_points <<< "${SLIDING_WINDOW[$data_key]}"
    
    # Split into two halves
    local mid=$((count / 2))
    local first_half_sum=0
    local second_half_sum=0
    
    for ((i=0; i<mid; i++)); do
        first_half_sum=$(echo "$first_half_sum + ${data_points[$i]}" | bc -l)
    done
    
    for ((i=mid; i<count; i++)); do
        second_half_sum=$(echo "$second_half_sum + ${data_points[$i]}" | bc -l)
    done
    
    local first_half_avg=$(echo "scale=4; $first_half_sum / $mid" | bc -l)
    local second_half_avg=$(echo "scale=4; $second_half_sum / (($count - $mid))" | bc -l)
    
    local change_pct=$(echo "scale=2; (($second_half_avg - $first_half_avg) / $first_half_avg) * 100" | bc -l 2>/dev/null || echo "0")
    
    if (( $(echo "$change_pct > 20" | bc -l) )); then
        echo "TRENDING_UP:${change_pct}%"
    elif (( $(echo "$change_pct < -20" | bc -l) )); then
        echo "TRENDING_DOWN:${change_pct}%"
    else
        echo "STABLE:${change_pct}%"
    fi
}
```

## 4.3 Pattern-Based Anomaly Detection

```bash
#!/bin/bash
# Advanced pattern-based anomaly detection

# Pattern correlation engine
declare -A PATTERN_COUNTS
declare -A CORRELATION_MATRIX

# Track pattern occurrences
track_pattern() {
    local pattern="$1"
    local timestamp="${2:-$(date +%s)}"
    local source="${3:-unknown}"
    
    local key="${pattern}_${source}"
    local minute_bucket=$((timestamp / 60))
    
    # Increment count
    ((PATTERN_COUNTS["${key}_${minute_bucket}"]++))
    ((PATTERN_COUNTS["${key}_total"]++))
}

# Detect unusual pattern combinations
detect_pattern_correlation() {
    local pattern1="$1"
    local pattern2="$2"
    local time_window="${3:-300}"  # 5 minutes default
    
    local current_time=$(date +%s)
    local window_start=$((current_time - time_window))
    
    local count1=0
    local count2=0
    
    # Count patterns in window
    for ((t=window_start; t<=current_time; t+=60)); do
        local bucket=$((t / 60))
        ((count1 += ${PATTERN_COUNTS["${pattern1}_${bucket}"]:-0}))
        ((count2 += ${PATTERN_COUNTS["${pattern2}_${bucket}"]:-0}))
    done
    
    # Check correlation
    if ((count1 > 5 && count2 > 5)); then
        echo "CORRELATED:$pattern1($count1):$pattern2($count2)"
        return 0
    fi
    
    return 1
}

# Attack pattern detection
detect_attack_pattern() {
    local log_entries="$1"
    local result=""
    
    # Brute force detection
    local failed_logins=$(echo "$log_entries" | grep -c "Failed password")
    if ((failed_logins > 10)); then
        result+="BRUTE_FORCE:count=$failed_logins "
    fi
    
    # Port scanning detection (many connection attempts from same IP)
    local unique_ports=$(echo "$log_entries" | grep -oP 'port \K[0-9]+' | sort -u | wc -l)
    if ((unique_ports > 20)); then
        result+="PORT_SCAN:ports=$unique_ports "
    fi
    
    # SQL injection attempts
    local sqli_attempts=$(echo "$log_entries" | grep -cE "(UNION|SELECT.*FROM|INSERT.*INTO|DELETE.*FROM|DROP.*TABLE)")
    if ((sqli_attempts > 0)); then
        result+="SQL_INJECTION:count=$sqli_attempts "
    fi
    
    # Directory traversal
    local traversal=$(echo "$log_entries" | grep -cE "(\.\./|%2e%2e)")
    if ((traversal > 0)); then
        result+="PATH_TRAVERSAL:count=$traversal "
    fi
    
    # Command injection
    local cmd_injection=$(echo "$log_entries" | grep -cE "(\||;|\$\(|`|&&|\|\|)")
    if ((cmd_injection > 5)); then
        result+="COMMAND_INJECTION:count=$cmd_injection "
    fi
    
    [[ -n "$result" ]] && echo "$result" || echo "CLEAN"
}

# Error spike detection
detect_error_spike() {
    local metric_name="$1"
    local current_count="$2"
    local baseline_file="${BASELINE_DIR}/${metric_name}.baseline"
    
    # Load baseline
    if [[ -f "$baseline_file" ]]; then
        source "$baseline_file"
        
        # Check for spike (3x above P95)
        local spike_threshold=$(echo "$P95 * 3" | bc -l)
        
        if (( $(echo "$current_count > $spike_threshold" | bc -l) )); then
            local spike_factor=$(echo "scale=2; $current_count / $MEAN" | bc -l)
            echo "ERROR_SPIKE:current=$current_count:baseline_mean=$MEAN:spike_factor=${spike_factor}x"
            return 0
        fi
    fi
    
    return 1
}
```

## 4.4 Real-Time Anomaly Detector

```bash
#!/bin/bash
# Complete real-time anomaly detection system

readonly ANOMALY_LOG="/var/log/log_analyzer/anomalies.log"
readonly METRICS_DIR="/var/lib/log_analyzer/metrics"

mkdir -p "$(dirname "$ANOMALY_LOG")" "$METRICS_DIR"

# Anomaly detection engine
anomaly_detector() {
    local input_file="$1"
    local detection_mode="${2:-realtime}"  # realtime or batch
    
    # Initialize metrics
    local error_count=0
    local warning_count=0
    local auth_failures=0
    local http_errors=0
    local response_times=()
    
    declare -A ip_request_count
    declare -A ip_failure_count
    declare -A endpoint_errors
    declare -A last_seen
    
    while IFS= read -r line; do
        local timestamp=$(date +%s)
        
        # Extract components
        local log_timestamp=""
        local severity=""
        local source_ip=""
        local message=""
        
        # Parse based on format
        if [[ "$line" =~ ^\{.*\}$ ]]; then
            # JSON format
            severity=$(echo "$line" | jq -r '.level // "INFO"' 2>/dev/null)
            log_timestamp=$(echo "$line" | jq -r '.timestamp // ""' 2>/dev/null)
            source_ip=$(echo "$line" | jq -r '.client_ip // .ip // ""' 2>/dev/null)
            message=$(echo "$line" | jq -r '.message // ""' 2>/dev/null)
        else
            # Traditional format
            severity=$(get_severity "$line")
            source_ip=$(extract_ip "$line")
            message="$line"
        fi
        
        # === SEVERITY-BASED DETECTION ===
        case "$severity" in
            ERROR|FATAL|CRITICAL)
                ((error_count++))
                track_pattern "error" "$timestamp"
                ;;
            WARNING)
                ((warning_count++))
                ;;
        esac
        
        # === AUTHENTICATION FAILURE DETECTION ===
        if [[ "$line" =~ [Ff]ailed.*([Pp]assword|[Ll]ogin|[Aa]uth) ]]; then
            ((auth_failures++))
            [[ -n "$source_ip" ]] && ((ip_failure_count["$source_ip"]++))
            
            # Brute force threshold
            if [[ -n "$source_ip" ]] && ((ip_failure_count["$source_ip"] >= 5)); then
                log_anomaly "BRUTE_FORCE" "HIGH" \
                    "IP $source_ip has ${ip_failure_count[$source_ip]} failed auth attempts" \
                    "$line"
            fi
        fi
        
        # === RATE LIMITING DETECTION ===
        if [[ -n "$source_ip" ]]; then
            ((ip_request_count["$source_ip"]++))
            
            # High request rate from single IP
            if ((ip_request_count["$source_ip"] >= 100)); then
                # Check time window
                local now=$(date +%s)
                local first_seen="${last_seen[$source_ip]:-$now}"
                local duration=$((now - first_seen))
                
                if ((duration < 60 && duration > 0)); then
                    local rate=$((ip_request_count["$source_ip"] * 60 / duration))
                    if ((rate > 200)); then
                        log_anomaly "RATE_LIMIT" "MEDIUM" \
                            "High request rate from $source_ip: ~$rate req/min" \
                            "$line"
                    fi
                fi
            fi
            
            [[ -z "${last_seen[$source_ip]}" ]] && last_seen[$source_ip]=$(date +%s)
        fi
        
        # === HTTP ERROR DETECTION ===
        if [[ "$line" =~ HTTP/[0-9.]+\"\ ([4-5][0-9]{2}) ]]; then
            local status_code="${BASH_REMATCH[1]}"
            ((http_errors++))
            
            # Extract endpoint
            local endpoint=""
            if [[ "$line" =~ \"[A-Z]+\ ([^\ ]+)\ HTTP ]]; then
                endpoint="${BASH_REMATCH[1]}"
                ((endpoint_errors["$endpoint:$status_code"]++))
            fi
            
            # 500 error spike
            if [[ "$status_code" =~ ^5 ]] && ((http_errors >= 10)); then
                log_anomaly "HTTP_5XX_SPIKE" "HIGH" \
                    "High 5xx error rate: $http_errors errors" \
                    "$line"
            fi
        fi
        
        # === SECURITY PATTERN DETECTION ===
        # SQL Injection
        if [[ "$line" =~ (UNION|SELECT|INSERT|DELETE|DROP).*-- ]] || \
           [[ "$line" =~ (\'|%27)(or|and).*= ]]; then
            log_anomaly "SQL_INJECTION" "CRITICAL" \
                "SQL injection attempt detected from ${source_ip:-unknown}" \
                "$line"
        fi
        
        # XSS Attack
        if [[ "$line" =~ \<script ]] || \
           [[ "$line" =~ javascript: ]] || \
           [[ "$line" =~ onerror= ]]; then
            log_anomaly "XSS_ATTEMPT" "CRITICAL" \
                "XSS attempt detected from ${source_ip:-unknown}" \
                "$line"
        fi
        
        # Path Traversal
        if [[ "$line" =~ \.\./\.\. ]] || [[ "$line" =~ %2e%2e%2f ]]; then
            log_anomaly "PATH_TRAVERSAL" "HIGH" \
                "Path traversal attempt from ${source_ip:-unknown}" \
                "$line"
        fi
        
        # === SYSTEM HEALTH DETECTION ===
        # Disk space
        if [[ "$line" =~ [Nn]o\ space\ left ]] || \
           [[ "$line" =~ [Dd]isk.*full ]] || \
           [[ "$line" =~ [Dd]isk.*100% ]]; then
            log_anomaly "DISK_FULL" "CRITICAL" \
                "Disk space critical" \
                "$line"
        fi
        
        # Out of memory
        if [[ "$line" =~ [Oo]ut\ of\ memory ]] || \
           [[ "$line" =~ OOM.killer ]] || \
           [[ "$line" =~ [Kk]illed\ process ]]; then
            log_anomaly "OOM_KILL" "CRITICAL" \
                "Out of memory condition detected" \
                "$line"
        fi
        
        # Service down
        if [[ "$line" =~ [Cc]onnection\ refused ]] || \
           [[ "$line" =~ [Ss]ervice.*(down|unavailable|failed) ]]; then
            log_anomaly "SERVICE_DOWN" "HIGH" \
                "Service connectivity issue" \
                "$line"
        fi
        
        # === PERIODIC CHECKS ===
        local current_minute=$((timestamp / 60))
        if ((current_minute != last_minute_check)); then
            # Error rate check
            if ((error_count > 50)); then
                log_anomaly "ERROR_SPIKE" "HIGH" \
                    "High error rate: $error_count errors in last minute" \
                    "Aggregated metric"
            fi
            
            # Reset minute counters
            error_count=0
            warning_count=0
            http_errors=0
            last_minute_check=$current_minute
        fi
        
    done < "$input_file"
}

# Log anomaly to file and trigger alerts
log_anomaly() {
    local anomaly_type="$1"
    local severity="$2"
    local description="$3"
    local raw_log="$4"
    
    local timestamp=$(date -Iseconds)
    local anomaly_id=$(uuidgen 2>/dev/null || echo "$$-$(date +%s%N)")
    
    # Create structured anomaly record
    local record=$(cat << EOF
{
    "id": "$anomaly_id",
    "timestamp": "$timestamp",
    "type": "$anomaly_type",
    "severity": "$severity",
    "description": "$description",
    "raw_log": $(echo "$raw_log" | jq -Rs .),
    "hostname": "$(hostname)"
}
EOF
)
    
    # Write to log
    echo "$record" >> "$ANOMALY_LOG"
    
    # Trigger alert based on severity
    case "$severity" in
        CRITICAL)
            trigger_alert "CRITICAL" "$anomaly_type" "$description" "$anomaly_id" &
            ;;
        HIGH)
            trigger_alert "HIGH" "$anomaly_type" "$description" "$anomaly_id" &
            ;;
        MEDIUM)
            # Queue for aggregated alerting
            echo "$anomaly_id" >> /var/lib/log_analyzer/pending_alerts.txt
            ;;
    esac
}

# Helper functions
get_severity() {
    local line="$1"
    if [[ "$line" =~ (CRITICAL|FATAL|EMERGENCY) ]]; then echo "CRITICAL"
    elif [[ "$line" =~ ERROR ]]; then echo "ERROR"
    elif [[ "$line" =~ (WARNING|WARN) ]]; then echo "WARNING"
    elif [[ "$line" =~ INFO ]]; then echo "INFO"
    else echo "DEBUG"
    fi
}

extract_ip() {
    local line="$1"
    if [[ "$line" =~ ([0-9]+\.[0-9]+\.[0-9]+\.[0-9]+) ]]; then
        echo "${BASH_REMATCH[1]}"
    fi
}
```

***

<a name="section-5"></a>

# 5. ALERT ESCALATION FRAMEWORK (25 minutes)

## 5.1 Alert Configuration System

```bash
#!/bin/bash
# Alert escalation configuration

readonly ALERT_CONFIG="/etc/log_analyzer/alerts.conf"
readonly ALERT_QUEUE="/var/lib/log_analyzer/alert_queue"
readonly ALERT_HISTORY="/var/lib/log_analyzer/alert_history"

mkdir -p "$(dirname "$ALERT_CONFIG")" "$ALERT_QUEUE" "$ALERT_HISTORY"

# Default configuration
create_default_config() {
    cat > "$ALERT_CONFIG" << 'EOF'
# Log Analyzer Alert Configuration
# ================================

# Email Settings
EMAIL_ENABLED=true
EMAIL_SMTP_HOST="smtp.example.com"
EMAIL_SMTP_PORT="587"
EMAIL_FROM="log-analyzer@example.com"
EMAIL_USERNAME=""
EMAIL_PASSWORD=""
EMAIL_USE_TLS=true

# Slack Settings
SLACK_ENABLED=true
SLACK_WEBHOOK_URL=""
SLACK_CHANNEL="#alerts"
SLACK_USERNAME="Log Analyzer"

# PagerDuty Settings
PAGERDUTY_ENABLED=false
PAGERDUTY_SERVICE_KEY=""

# Telegram Settings
TELEGRAM_ENABLED=false
TELEGRAM_BOT_TOKEN=""
TELEGRAM_CHAT_ID=""

# Alert Thresholds
ALERT_COOLDOWN_SECONDS=300  # Don't repeat same alert within this period
ALERT_AGGREGATE_WINDOW=60   # Aggregate similar alerts within this window
MAX_ALERTS_PER_HOUR=50      # Rate limiting

# Escalation Settings
ESCALATION_ENABLED=true
ESCALATION_TIMEOUT_MINUTES=15  # Time before escalation

# On-Call Schedule (JSON format)
ONCALL_SCHEDULE='[
    {"day": "Mon-Fri", "start": "09:00", "end": "18:00", "team": "primary"},
    {"day": "Mon-Fri", "start": "18:00", "end": "09:00", "team": "oncall"},
    {"day": "Sat-Sun", "start": "00:00", "end": "23:59", "team": "oncall"}
]'

# Team Contacts
declare -A TEAM_CONTACTS
TEAM_CONTACTS[primary]="team@example.com,#team-alerts"
TEAM_CONTACTS[oncall]="oncall@example.com,#oncall-alerts"
TEAM_CONTACTS[security]="security@example.com,#security-incidents"
TEAM_CONTACTS[management]="cto@example.com"

# Severity to Team Mapping
declare -A SEVERITY_ROUTING
SEVERITY_ROUTING[CRITICAL]="primary,security,oncall"
SEVERITY_ROUTING[HIGH]="primary,oncall"
SEVERITY_ROUTING[MEDIUM]="primary"
SEVERITY_ROUTING[LOW]="primary"
EOF
}

# Load configuration
load_alert_config() {
    if [[ -f "$ALERT_CONFIG" ]]; then
        source "$ALERT_CONFIG"
    else
        echo "WARNING: Alert config not found, creating default"
        create_default_config
        source "$ALERT_CONFIG"
    fi
}
```

## 5.2 Multi-Channel Alert System

```bash
#!/bin/bash
# Multi-channel alerting system

load_alert_config

# Send alert through all configured channels
trigger_alert() {
    local severity="$1"
    local alert_type="$2"
    local description="$3"
    local alert_id="$4"
    
    local timestamp=$(date -Iseconds)
    
    # Check cooldown
    if is_in_cooldown "$alert_type" "$severity"; then
        echo "Alert suppressed (cooldown): $alert_type"
        return 0
    fi
    
    # Check rate limiting
    if is_rate_limited; then
        echo "Alert suppressed (rate limit): $alert_type"
        return 0
    fi
    
    # Create alert payload
    local alert_payload=$(cat << EOF
{
    "id": "$alert_id",
    "timestamp": "$timestamp",
    "severity": "$severity",
    "type": "$alert_type",
    "description": "$description",
    "hostname": "$(hostname)",
    "environment": "${ENVIRONMENT:-production}"
}
EOF
)
    
    # Determine recipients
    local routing="${SEVERITY_ROUTING[$severity]:-primary}"
    
    # Send to configured channels
    local status=0
    
    if [[ "$EMAIL_ENABLED" == "true" ]]; then
        send_email_alert "$severity" "$alert_type" "$description" "$routing" || ((status++))
    fi
    
    if [[ "$SLACK_ENABLED" == "true" ]]; then
        send_slack_alert "$severity" "$alert_type" "$description" "$alert_id" || ((status++))
    fi
    
    if [[ "$PAGERDUTY_ENABLED" == "true" ]] && [[ "$severity" =~ ^(CRITICAL|HIGH)$ ]]; then
        send_pagerduty_alert "$severity" "$alert_type" "$description" "$alert_id" || ((status++))
    fi
    
    if [[ "$TELEGRAM_ENABLED" == "true" ]]; then
        send_telegram_alert "$severity" "$alert_type" "$description" || ((status++))
    fi
    
    # Record alert
    record_alert "$alert_id" "$severity" "$alert_type" "$timestamp"
    
    # Schedule escalation if needed
    if [[ "$ESCALATION_ENABLED" == "true" ]] && [[ "$severity" == "CRITICAL" ]]; then
        schedule_escalation "$alert_id" "$severity" "$alert_type" "$description"
    fi
    
    return $status
}

# Email alerting
send_email_alert() {
    local severity="$1"
    local alert_type="$2"
    local description="$3"
    local routing="$4"
    
    # Build recipient list
    local recipients=""
    IFS=',' read -ra teams <<< "$routing"
    for team in "${teams[@]}"; do
        local contacts="${TEAM_CONTACTS[$team]:-}"
        for contact in ${contacts//,/ }; do
            [[ "$contact" =~ @ ]] && recipients+="$contact,"
        done
    done
    recipients="${recipients%,}"
    
    [[ -z "$recipients" ]] && return 1
    
    # Create email
    local subject="[$severity] $alert_type - $(hostname)"
    local body=$(cat << EOF
Log Analyzer Alert
==================

Severity: $severity
Type: $alert_type
Hostname: $(hostname)
Time: $(date)

Description:
$description

---
This is an automated alert from Log Analyzer.
EOF
)
    
    # Send using mail command or sendmail
    if command -v mail >/dev/null 2>&1; then
        echo "$body" | mail -s "$subject" "$recipients"
        return $?
    elif command -v sendmail >/dev/null 2>&1; then
        sendmail -t << EMAIL
To: $recipients
Subject: $subject
From: $EMAIL_FROM

$body
EMAIL
        return $?
    else
        # Use curl with SMTP
        send_email_via_curl "$recipients" "$subject" "$body"
        return $?
    fi
}

send_email_via_curl() {
    local to="$1"
    local subject="$2"
    local body="$3"
    
    local smtp_url="smtp://${EMAIL_SMTP_HOST}:${EMAIL_SMTP_PORT}"
    [[ "$EMAIL_USE_TLS" == "true" ]] && smtp_url="smtps://${EMAIL_SMTP_HOST}:${EMAIL_SMTP_PORT}"
    
    curl --ssl-reqd \
        --mail-from "$EMAIL_FROM" \
        --mail-rcpt "$to" \
        --url "$smtp_url" \
        --user "${EMAIL_USERNAME}:${EMAIL_PASSWORD}" \
        -T - << EMAIL
From: $EMAIL_FROM
To: $to
Subject: $subject

$body
EMAIL
}

# Slack alerting
send_slack_alert() {
    local severity="$1"
    local alert_type="$2"
    local description="$3"
    local alert_id="$4"
    
    [[ -z "$SLACK_WEBHOOK_URL" ]] && return 1
    
    # Color coding
    local color
    case "$severity" in
        CRITICAL) color="#dc3545" ;;  # Red
        HIGH)     color="#fd7e14" ;;  # Orange
        MEDIUM)   color="#ffc107" ;;  # Yellow
        LOW)      color="#28a745" ;;  # Green
        *)        color="#6c757d" ;;  # Gray
    esac
    
    # Build payload
    local payload=$(cat << EOF
{
    "channel": "$SLACK_CHANNEL",
    "username": "$SLACK_USERNAME",
    "icon_emoji": ":warning:",
    "attachments": [
        {
            "color": "$color",
            "title": "[$severity] $alert_type",
            "text": "$description",
            "fields": [
                {"title": "Hostname", "value": "$(hostname)", "short": true},
                {"title": "Time", "value": "$(date '+%Y-%m-%d %H:%M:%S')", "short": true},
                {"title": "Alert ID", "value": "\`$alert_id\`", "short": true},
                {"title": "Environment", "value": "${ENVIRONMENT:-production}", "short": true}
            ],
            "footer": "Log Analyzer",
            "ts": $(date +%s)
        }
    ]
}
EOF
)
    
    # Send to Slack
    curl -s -X POST \
        -H "Content-Type: application/json" \
        -d "$payload" \
        "$SLACK_WEBHOOK_URL" >/dev/null 2>&1
}

# PagerDuty alerting
send_pagerduty_alert() {
    local severity="$1"
    local alert_type="$2"
    local description="$3"
    local alert_id="$4"
    
    [[ -z "$PAGERDUTY_SERVICE_KEY" ]] && return 1
    
    # PagerDuty severity mapping
    local pd_severity
    case "$severity" in
        CRITICAL) pd_severity="critical" ;;
        HIGH)     pd_severity="error" ;;
        MEDIUM)   pd_severity="warning" ;;
        *)        pd_severity="info" ;;
    esac
    
    local payload=$(cat << EOF
{
    "routing_key": "$PAGERDUTY_SERVICE_KEY",
    "event_action": "trigger",
    "dedup_key": "${alert_type}_${alert_id}",
    "payload": {
        "summary": "[$severity] $alert_type on $(hostname)",
        "source": "$(hostname)",
        "severity": "$pd_severity",
        "custom_details": {
            "description": "$description",
            "alert_id": "$alert_id",
            "environment": "${ENVIRONMENT:-production}"
        }
    }
}
EOF
)
    
    curl -s -X POST \
        -H "Content-Type: application/json" \
        -d "$payload" \
        "https://events.pagerduty.com/v2/enqueue" >/dev/null 2>&1
}

# Telegram alerting
send_telegram_alert() {
    local severity="$1"
    local alert_type="$2"
    local description="$3"
    
    [[ -z "$TELEGRAM_BOT_TOKEN" ]] || [[ -z "$TELEGRAM_CHAT_ID" ]] && return 1
    
    # Emoji based on severity
    local emoji
    case "$severity" in
        CRITICAL) emoji="🔴" ;;
        HIGH)     emoji="🟠" ;;
        MEDIUM)   emoji="🟡" ;;
        LOW)      emoji="🟢" ;;
        *)        emoji="⚪" ;;
    esac
    
    local message="${emoji} *[$severity] $alert_type*

🖥️ Host: \`$(hostname)\`
⏰ Time: $(date '+%Y-%m-%d %H:%M:%S')

📝 *Description:*
$description"
    
    curl -s -X POST \
        "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
        -d "chat_id=${TELEGRAM_CHAT_ID}" \
        -d "text=${message}" \
        -d "parse_mode=Markdown" >/dev/null 2>&1
}
```

## 5.3 Alert Escalation Logic

```bash
#!/bin/bash
# Alert escalation system

readonly ESCALATION_DIR="/var/lib/log_analyzer/escalations"
mkdir -p "$ESCALATION_DIR"

# Schedule escalation
schedule_escalation() {
    local alert_id="$1"
    local severity="$2"
    local alert_type="$3"
    local description="$4"
    
    local escalation_file="${ESCALATION_DIR}/${alert_id}.escalation"
    local escalation_time=$(($(date +%s) + ESCALATION_TIMEOUT_MINUTES * 60))
    
    cat > "$escalation_file" << EOF
ALERT_ID=$alert_id
SEVERITY=$severity
ALERT_TYPE=$alert_type
DESCRIPTION=$description
CREATED=$(date +%s)
ESCALATE_AT=$escalation_time
LEVEL=1
ACKNOWLEDGED=false
EOF
    
    echo "Escalation scheduled for alert $alert_id at $(date -d "@$escalation_time")"
}

# Check and process escalations
process_escalations() {
    local current_time=$(date +%s)
    
    for escalation_file in "$ESCALATION_DIR"/*.escalation; do
        [[ -f "$escalation_file" ]] || continue
        
        source "$escalation_file"
        
        # Skip acknowledged alerts
        [[ "$ACKNOWLEDGED" == "true" ]] && {
            rm -f "$escalation_file"
            continue
        }
        
        # Check if escalation time reached
        if ((current_time >= ESCALATE_AT)); then
            escalate_alert "$ALERT_ID" "$SEVERITY" "$ALERT_TYPE" "$DESCRIPTION" "$LEVEL"
            
            # Schedule next level escalation
            ((LEVEL++))
            if ((LEVEL <= 3)); then
                local next_escalation=$((current_time + ESCALATION_TIMEOUT_MINUTES * 60))
                cat > "$escalation_file" << EOF
ALERT_ID=$ALERT_ID
SEVERITY=$SEVERITY
ALERT_TYPE=$ALERT_TYPE
DESCRIPTION=$DESCRIPTION
CREATED=$CREATED
ESCALATE_AT=$next_escalation
LEVEL=$LEVEL
ACKNOWLEDGED=false
EOF
            else
                # Max escalation reached
                log_final_escalation "$ALERT_ID"
                rm -f "$escalation_file"
            fi
        fi
    done
}

# Escalate to next level
escalate_alert() {
    local alert_id="$1"
    local severity="$2"
    local alert_type="$3"
    local description="$4"
    local level="$5"
    
    echo "ESCALATING: Alert $alert_id to level $level"
    
    local escalation_message="⚠️ ESCALATION LEVEL $level ⚠️

Original Alert: $alert_type ($severity)
Time Since Alert: $(($(date +%s) - CREATED)) seconds
Status: UNACKNOWLEDGED

$description

This alert has not been acknowledged and is being escalated."
    
    case $level in
        1)
            # Notify on-call team
            send_slack_alert "HIGH" "ESCALATION:$alert_type" "$escalation_message" "$alert_id"
            ;;
        2)
            # Add management
            send_slack_alert "CRITICAL" "ESCALATION:$alert_type" "$escalation_message" "$alert_id"
            send_email_alert "CRITICAL" "ESCALATION:$alert_type" "$escalation_message" "oncall,management"
            ;;
        3)
            # Final escalation - all hands
            send_slack_alert "CRITICAL" "FINAL_ESCALATION:$alert_type" "$escalation_message" "$alert_id"
            send_email_alert "CRITICAL" "FINAL_ESCALATION:$alert_type" "$escalation_message" "primary,oncall,security,management"
            send_pagerduty_alert "CRITICAL" "FINAL_ESCALATION:$alert_type" "$escalation_message" "$alert_id"
            ;;
    esac
}

# Acknowledge alert
acknowledge_alert() {
    local alert_id="$1"
    local acknowledged_by="${2:-unknown}"
    
    local escalation_file="${ESCALATION_DIR}/${alert_id}.escalation"
    
    if [[ -f "$escalation_file" ]]; then
        # Update escalation file
        sed -i "s/ACKNOWLEDGED=false/ACKNOWLEDGED=true/" "$escalation_file"
        
        # Log acknowledgment
        echo "$(date -Iseconds)|$alert_id|ACKNOWLEDGED|$acknowledged_by" >> "$ALERT_HISTORY/acknowledgments.log"
        
        # Send confirmation
        send_slack_alert "INFO" "ALERT_ACKNOWLEDGED" \
            "Alert $alert_id acknowledged by $acknowledged_by" \
            "$alert_id"
        
        echo "Alert $alert_id acknowledged by $acknowledged_by"
        return 0
    else
        echo "ERROR: Escalation file not found for alert $alert_id"
        return 1
    fi
}

# Resolve alert
resolve_alert() {
    local alert_id="$1"
    local resolved_by="${2:-system}"
    local resolution_notes="${3:-Auto-resolved}"
    
    local escalation_file="${ESCALATION_DIR}/${alert_id}.escalation"
    
    # Log resolution
    local resolution_record=$(cat << EOF
{
    "alert_id": "$alert_id",
    "resolved_at": "$(date -Iseconds)",
    "resolved_by": "$resolved_by",
    "notes": "$resolution_notes",
    "resolution_time_seconds": $(($(date +%s) - ${CREATED:-$(date +%s)}))
}
EOF
)
    
    echo "$resolution_record" >> "$ALERT_HISTORY/resolutions.log"
    
    # Remove escalation file
    rm -f "$escalation_file"
    
    # Send resolution notification
    send_slack_alert "INFO" "ALERT_RESOLVED" \
        "✅ Alert $alert_id resolved by $resolved_by: $resolution_notes" \
        "$alert_id"
    
    echo "Alert $alert_id resolved"
}

# Cooldown management
is_in_cooldown() {
    local alert_type="$1"
    local severity="$2"
    
    local cooldown_file="${ALERT_QUEUE}/cooldown_${alert_type}_${severity}"
    local current_time=$(date +%s)
    
    if [[ -f "$cooldown_file" ]]; then
        local last_alert=$(cat "$cooldown_file")
        local elapsed=$((current_time - last_alert))
        
        if ((elapsed < ALERT_COOLDOWN_SECONDS)); then
            return 0  # In cooldown
        fi
    fi
    
    # Update cooldown
    echo "$current_time" > "$cooldown_file"
    return 1  # Not in cooldown
}

# Rate limiting
is_rate_limited() {
    local rate_file="${ALERT_QUEUE}/rate_limit"
    local current_hour=$(date +%Y%m%d%H)
    local count=0
    
    if [[ -f "$rate_file" ]]; then
        local stored_hour=$(head -1 "$rate_file")
        if [[ "$stored_hour" == "$current_hour" ]]; then
            count=$(tail -1 "$rate_file")
        fi
    fi
    
    if ((count >= MAX_ALERTS_PER_HOUR)); then
        return 0  # Rate limited
    fi
    
    # Update count
    echo -e "${current_hour}\n$((count + 1))" > "$rate_file"
    return 1  # Not rate limited
}

# Record alert for history
record_alert() {
    local alert_id="$1"
    local severity="$2"
    local alert_type="$3"
    local timestamp="$4"
    
    local history_file="${ALERT_HISTORY}/$(date +%Y%m%d).log"
    
    echo "${timestamp}|${alert_id}|${severity}|${alert_type}" >> "$history_file"
}

# Escalation daemon
run_escalation_daemon() {
    echo "Starting escalation daemon..."
    
    while true; do
        process_escalations
        sleep 60  # Check every minute
    done
}
```

## 5.4 Alert Aggregation System

```bash
#!/bin/bash
# Alert aggregation to prevent alert fatigue

readonly AGGREGATION_WINDOW=60  # seconds
declare -A PENDING_ALERTS

aggregate_alerts() {
    local alert_type="$1"
    local severity="$2"
    local description="$3"
    local source="$4"
    
    local key="${alert_type}_${severity}"
    local current_time=$(date +%s)
    
    # Check if we have pending alerts of this type
    if [[ -n "${PENDING_ALERTS[$key]}" ]]; then
        IFS='|' read -r first_time count descriptions <<< "${PENDING_ALERTS[$key]}"
        
        # Still within aggregation window?
        if ((current_time - first_time < AGGREGATION_WINDOW)); then
            # Aggregate
            ((count++))
            descriptions+="\\n- $description"
            PENDING_ALERTS[$key]="$first_time|$count|$descriptions"
            return 0
        else
            # Window expired, flush and start new
            flush_aggregated_alert "$key"
        fi
    fi
    
    # Start new aggregation
    PENDING_ALERTS[$key]="$current_time|1|$description"
}

flush_aggregated_alert() {
    local key="$1"
    
    [[ -z "${PENDING_ALERTS[$key]}" ]] && return
    
    IFS='|' read -r first_time count descriptions <<< "${PENDING_ALERTS[$key]}"
    IFS='_' read -r alert_type severity <<< "$key"
    
    local alert_id=$(uuidgen 2>/dev/null || echo "agg-$$-$(date +%s)")
    
    if ((count > 1)); then
        # Send aggregated alert
        local agg_description="🔄 AGGREGATED: $count occurrences in ${AGGREGATION_WINDOW}s

Details:
$(echo -e "$descriptions")"
        
        trigger_alert "$severity" "${alert_type}_AGGREGATED" "$agg_description" "$alert_id"
    else
        # Single alert, send normally
        trigger_alert "$severity" "$alert_type" "$descriptions" "$alert_id"
    fi
    
    unset "PENDING_ALERTS[$key]"
}

# Flush all pending alerts
flush_all_aggregated() {
    for key in "${!PENDING_ALERTS[@]}"; do
        flush_aggregated_alert "$key"
    done
}

# Periodic flusher (run in background)
run_aggregation_flusher() {
    while true; do
        sleep "$AGGREGATION_WINDOW"
        flush_all_aggregated
    done
}
```

***

<a name="section-6"></a>

# 6. EXECUTIVE REPORTING (20 minutes)

## 6.1 Report Generation Framework

```bash
#!/bin/bash
# Executive summary report generator

readonly REPORT_DIR="/var/lib/log_analyzer/reports"
readonly REPORT_TEMPLATE_DIR="/etc/log_analyzer/templates"

mkdir -p "$REPORT_DIR" "$REPORT_TEMPLATE_DIR"

# Generate daily executive summary
generate_daily_report() {
    local report_date="${1:-$(date +%Y-%m-%d)}"
    local report_file="${REPORT_DIR}/daily_${report_date}.html"
    local data_file="${REPORT_DIR}/daily_${report_date}.json"
    
    echo "Generating daily report for $report_date..."
    
    # Collect metrics
    local metrics=$(collect_daily_metrics "$report_date")
    
    # Save raw data
    echo "$metrics" > "$data_file"
    
    # Generate HTML report
    generate_html_report "$metrics" "$report_file" "$report_date"
    
    # Generate PDF if wkhtmltopdf available
    if command -v wkhtmltopdf >/dev/null 2>&1; then
        wkhtmltopdf "$report_file" "${report_file%.html}.pdf" 2>/dev/null
    fi
    
    echo "Report generated: $report_file"
}

collect_daily_metrics() {
    local date="$1"
    local log_date=$(date -d "$date" +"%b %d" 2>/dev/null || echo "$date")
    
    # Initialize counters
    local total_events=0
    local error_count=0
    local warning_count=0
    local critical_count=0
    local auth_failures=0
    local security_incidents=0
    
    declare -A hourly_errors
    declare -A top_ips
    declare -A top_errors
    declare -A service_health
    
    # Process system logs
    if [[ -f /var/log/syslog ]]; then
        while IFS= read -r line; do
            ((total_events++))
            
            # Count by severity
            [[ "$line" =~ ERROR ]] && ((error_count++))
            [[ "$line" =~ WARNING ]] && ((warning_count++))
            [[ "$line" =~ CRITICAL ]] && ((critical_count++))
            
            # Extract hour for hourly breakdown
            if [[ "$line" =~ ([0-9]{2}):[0-9]{2}:[0-9]{2} ]]; then
                local hour="${BASH_REMATCH[1]}"
                ((hourly_errors[$hour]++))
            fi
        done < <(grep "^$log_date" /var/log/syslog 2>/dev/null)
    fi
    
    # Process auth logs
    if [[ -f /var/log/auth.log ]]; then
        auth_failures=$(grep -c "Failed password\|authentication failure" /var/log/auth.log 2>/dev/null || echo 0)
        
        # Get top offending IPs
        while IFS=' ' read -r count ip; do
            [[ -n "$ip" ]] && top_ips["$ip"]=$count
        done < <(grep -oP 'from \K[0-9.]+' /var/log/auth.log 2>/dev/null | sort | uniq -c | sort -rn | head -10)
    fi
    
    # Process anomaly log
    if [[ -f "$ANOMALY_LOG" ]]; then
        security_incidents=$(grep -c "SQL_INJECTION\|XSS\|PATH_TRAVERSAL\|BRUTE_FORCE" "$ANOMALY_LOG" 2>/dev/null || echo 0)
    fi
    
    # Get alert statistics
    local alerts_triggered=0
    local alerts_acknowledged=0
    local alerts_resolved=0
    local avg_resolution_time=0
    
    local alert_history_file="${ALERT_HISTORY}/${date//-/}.log"
    if [[ -f "$alert_history_file" ]]; then
        alerts_triggered=$(wc -l < "$alert_history_file")
    fi
    
    if [[ -f "${ALERT_HISTORY}/acknowledgments.log" ]]; then
        alerts_acknowledged=$(grep -c "$date" "${ALERT_HISTORY}/acknowledgments.log" 2>/dev/null || echo 0)
    fi
    
    if [[ -f "${ALERT_HISTORY}/resolutions.log" ]]; then
        alerts_resolved=$(grep -c "$date" "${ALERT_HISTORY}/resolutions.log" 2>/dev/null || echo 0)
        
        # Calculate average resolution time
        local total_time=0
        local count=0
        while IFS= read -r line; do
            local res_time=$(echo "$line" | jq -r '.resolution_time_seconds // 0' 2>/dev/null)
            ((total_time += res_time))
            ((count++))
        done < <(grep "$date" "${ALERT_HISTORY}/resolutions.log" 2>/dev/null)
        
        ((count > 0)) && avg_resolution_time=$((total_time / count))
    fi
    
    # Build JSON output
    cat << EOF
{
    "report_date": "$date",
    "generated_at": "$(date -Iseconds)",
    "summary": {
        "total_events": $total_events,
        "errors": $error_count,
        "warnings": $warning_count,
        "critical": $critical_count,
        "auth_failures": $auth_failures,
        "security_incidents": $security_incidents
    },
    "alerts": {
        "triggered": $alerts_triggered,
        "acknowledged": $alerts_acknowledged,
        "resolved": $alerts_resolved,
        "avg_resolution_seconds": $avg_resolution_time
    },
    "hourly_breakdown": $(printf '%s\n' "${!hourly_errors[@]}" | jq -Rn '[inputs | split(" ") | {(.[0]): (.[1] | tonumber)}] | add // {}' 2>/dev/null || echo '{}'),
    "top_offending_ips": $(for ip in "${!top_ips[@]}"; do echo "\"$ip\": ${top_ips[$ip]}"; done | jq -s 'add // {}' 2>/dev/null || echo '{}'),
    "system_health": {
        "cpu_avg": $(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1 2>/dev/null || echo 0),
        "memory_used_pct": $(free | awk '/Mem:/ {printf "%.1f", $3/$2 * 100}' 2>/dev/null || echo 0),
        "disk_used_pct": $(df / | awk 'NR==2 {print $5}' | tr -d '%' 2>/dev/null || echo 0)
    }
}
EOF
}

generate_html_report() {
    local metrics="$1"
    local output_file="$2"
    local report_date="$3"
    
    # Extract values from JSON
    local total_events=$(echo "$metrics" | jq -r '.summary.total_events')
    local errors=$(echo "$metrics" | jq -r '.summary.errors')
    local warnings=$(echo "$metrics" | jq -r '.summary.warnings')
    local critical=$(echo "$metrics" | jq -r '.summary.critical')
    local auth_failures=$(echo "$metrics" | jq -r '.summary.auth_failures')
    local security_incidents=$(echo "$metrics" | jq -r '.summary.security_incidents')
    local alerts_triggered=$(echo "$metrics" | jq -r '.alerts.triggered')
    local alerts_resolved=$(echo "$metrics" | jq -r '.alerts.resolved')
    local avg_resolution=$(echo "$metrics" | jq -r '.alerts.avg_resolution_seconds')
    
    # Calculate health score (0-100)
    local health_score=100
    ((errors > 100)) && ((health_score -= 20))
    ((critical > 10)) && ((health_score -= 30))
    ((security_incidents > 0)) && ((health_score -= 25))
    ((health_score < 0)) && health_score=0
    
    # Health color
    local health_color="green"
    ((health_score < 70)) && health_color="orange"
    ((health_score < 50)) && health_color="red"
    
    cat > "$output_file" << EOF
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Daily Security & Operations Report - $report_date</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { 
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; 
            background: #f5f6fa; 
            color: #2d3436;
            line-height: 1.6;
        }
        .container { max-width: 1200px; margin: 0 auto; padding: 20px; }
        header { 
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white; 
            padding: 30px; 
            border-radius: 10px;
            margin-bottom: 20px;
        }
        header h1 { font-size: 2em; margin-bottom: 5px; }
        header p { opacity: 0.9; }
        .health-score {
            display: inline-block;
            padding: 10px 20px;
            border-radius: 50px;
            font-size: 1.5em;
            font-weight: bold;
            background: white;
            color: ${health_color};
            margin-top: 15px;
        }
        .grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 20px; margin-bottom: 20px; }
        .card {
            background: white;
            border-radius: 10px;
            padding: 20px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        .card h3 { color: #636e72; font-size: 0.9em; text-transform: uppercase; margin-bottom: 10px; }
        .card .value { font-size: 2.5em; font-weight: bold; color: #2d3436; }
        .card.critical .value { color: #e74c3c; }
        .card.warning .value { color: #f39c12; }
        .card.success .value { color: #27ae60; }
        .section { background: white; border-radius: 10px; padding: 20px; margin-bottom: 20px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
        .section h2 { color: #2d3436; border-bottom: 2px solid #667eea; padding-bottom: 10px; margin-bottom: 20px; }
        table { width: 100%; border-collapse: collapse; }
        th, td { padding: 12px; text-align: left; border-bottom: 1px solid #dfe6e9; }
        th { background: #f8f9fa; font-weight: 600; }
        .status-badge {
            display: inline-block;
            padding: 4px 12px;
            border-radius: 20px;
            font-size: 0.85em;
            font-weight: 500;
        }
        .status-critical { background: #fee2e2; color: #dc2626; }
        .status-warning { background: #fef3c7; color: #d97706; }
        .status-ok { background: #d1fae5; color: #059669; }
        .chart-container { height: 200px; margin: 20px 0; }
        footer { text-align: center; padding: 20px; color: #636e72; }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>📊 Daily Security & Operations Report</h1>
            <p>Report Date: $report_date | Generated: $(date '+%Y-%m-%d %H:%M:%S')</p>
            <p>Hostname: $(hostname) | Environment: ${ENVIRONMENT:-Production}</p>
            <div class="health-score">System Health: ${health_score}%</div>
        </header>

        <div class="grid">
            <div class="card">
                <h3>Total Events</h3>
                <div class="value">$total_events</div>
            </div>
            <div class="card ${critical:+critical}">
                <h3>Critical Issues</h3>
                <div class="value">$critical</div>
            </div>
            <div class="card ${errors:+warning}">
                <h3>Errors</h3>
                <div class="value">$errors</div>
            </div>
            <div class="card">
                <h3>Warnings</h3>
                <div class="value">$warnings</div>
            </div>
        </div>

        <div class="grid">
            <div class="card ${security_incidents:+critical}">
                <h3>🔒 Security Incidents</h3>
                <div class="value">$security_incidents</div>
            </div>
            <div class="card ${auth_failures:+warning}">
                <h3>🔐 Auth Failures</h3>
                <div class="value">$auth_failures</div>
            </div>
            <div class="card">
                <h3>🚨 Alerts Triggered</h3>
                <div class="value">$alerts_triggered</div>
            </div>
            <div class="card success">
                <h3>✅ Alerts Resolved</h3>
                <div class="value">$alerts_resolved</div>
            </div>
        </div>

        <div class="section">
            <h2>🎯 Key Metrics Summary</h2>
            <table>
                <tr>
                    <th>Metric</th>
                    <th>Value</th>
                    <th>Status</th>
                </tr>
                <tr>
                    <td>Average Alert Resolution Time</td>
                    <td>$(format_duration $avg_resolution)</td>
                    <td><span class="status-badge $([ $avg_resolution -lt 300 ] && echo 'status-ok' || echo 'status-warning')">$([ $avg_resolution -lt 300 ] && echo 'Good' || echo 'Needs Improvement')</span></td>
                </tr>
                <tr>
                    <td>Error Rate</td>
                    <td>$(echo "scale=2; $errors * 100 / ($total_events + 1)" | bc)%</td>
                    <td><span class="status-badge $([ $errors -lt 100 ] && echo 'status-ok' || echo 'status-warning')">$([ $errors -lt 100 ] && echo 'Normal' || echo 'Elevated')</span></td>
                </tr>
                <tr>
                    <td>Security Posture</td>
                    <td>$security_incidents incidents</td>
                    <td><span class="status-badge $([ $security_incidents -eq 0 ] && echo 'status-ok' || echo 'status-critical')">$([ $security_incidents -eq 0 ] && echo 'Secure' || echo 'At Risk')</span></td>
                </tr>
            </table>
        </div>

        <div class="section">
            <h2>🔝 Top Offending IP Addresses</h2>
            <table>
                <tr>
                    <th>Rank</th>
                    <th>IP Address</th>
                    <th>Event Count</th>
                    <th>Threat Level</th>
                </tr>
$(echo "$metrics" | jq -r '.top_offending_ips | to_entries | sort_by(-.value) | .[:5] | to_entries[] | "<tr><td>\(.key + 1)</td><td>\(.value.key)</td><td>\(.value.value)</td><td><span class=\"status-badge \(if .value.value > 50 then \"status-critical\" elif .value.value > 20 then \"status-warning\" else \"status-ok\" end)\">\(if .value.value > 50 then \"High\" elif .value.value > 20 then \"Medium\" else \"Low\" end)</span></td></tr>"' 2>/dev/null || echo "<tr><td colspan='4'>No data available</td></tr>")
            </table>
        </div>

        <div class="section">
            <h2>💡 Recommendations</h2>
            <ul style="padding-left: 20px;">
$(generate_recommendations "$metrics")
            </ul>
        </div>

        <footer>
            <p>Generated by Log Analyzer | $(hostname) | $(date)</p>
        </footer>
    </div>
</body>
</html>
EOF
}

# Helper function to format duration
format_duration() {
    local seconds=$1
    if ((seconds < 60)); then
        echo "${seconds}s"
    elif ((seconds < 3600)); then
        echo "$((seconds / 60))m $((seconds % 60))s"
    else
        echo "$((seconds / 3600))h $((seconds % 3600 / 60))m"
    fi
}

# Generate recommendations based on metrics
generate_recommendations() {
    local metrics="$1"
    local recommendations=""
    
    local errors=$(echo "$metrics" | jq -r '.summary.errors')
    local security=$(echo "$metrics" | jq -r '.summary.security_incidents')
    local auth=$(echo "$metrics" | jq -r '.summary.auth_failures')
    
    if ((errors > 100)); then
        recommendations+="<li><strong>High Error Rate:</strong> Investigate application logs for recurring errors. Consider implementing circuit breakers.</li>\n"
    fi
    
    if ((security > 0)); then
        recommendations+="<li><strong>Security Incidents Detected:</strong> Review security incident logs immediately. Consider blocking suspicious IPs.</li>\n"
    fi
    
    if ((auth > 50)); then
        recommendations+="<li><strong>Authentication Issues:</strong> High number of auth failures detected. Consider implementing rate limiting or account lockout policies.</li>\n"
    fi
    
    if [[ -z "$recommendations" ]]; then
        recommendations="<li><strong>All Clear:</strong> No immediate action required. System operating normally.</li>\n"
    fi
    
    echo -e "$recommendations"
}
```

## 6.2 Scheduled Report Generation

```bash
#!/bin/bash
# Scheduled report generation with cron integration

setup_report_schedule() {
    local cron_file="/etc/cron.d/log_analyzer_reports"
    
    cat > "$cron_file" << 'EOF'
# Log Analyzer Report Schedule
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# Daily report at 6 AM
0 6 * * * root /usr/local/bin/log_analyzer_report.sh daily

# Weekly report on Monday at 7 AM
0 7 * * 1 root /usr/local/bin/log_analyzer_report.sh weekly

# Monthly report on 1st at 8 AM
0 8 1 * * root /usr/local/bin/log_analyzer_report.sh monthly
EOF

    chmod 644 "$cron_file"
    echo "Report schedule configured"
}

# Weekly report generator
generate_weekly_report() {
    local week_end="${1:-$(date +%Y-%m-%d)}"
    local week_start=$(date -d "$week_end -7 days" +%Y-%m-%d)
    local report_file="${REPORT_DIR}/weekly_${week_start}_to_${week_end}.html"
    
    echo "Generating weekly report: $week_start to $week_end"
    
    # Aggregate daily reports
    local total_events=0
    local total_errors=0
    local total_alerts=0
    local total_incidents=0
    
    for ((i=0; i<7; i++)); do
        local day=$(date -d "$week_end -$i days" +%Y-%m-%d)
        local daily_data="${REPORT_DIR}/daily_${day}.json"
        
        if [[ -f "$daily_data" ]]; then
            total_events=$((total_events + $(jq -r '.summary.total_events // 0' "$daily_data")))
            total_errors=$((total_errors + $(jq -r '.summary.errors // 0' "$daily_data")))
            total_alerts=$((total_alerts + $(jq -r '.alerts.triggered // 0' "$daily_data")))
            total_incidents=$((total_incidents + $(jq -r '.summary.security_incidents // 0' "$daily_data")))
        fi
    done
    
    # Generate weekly HTML (similar structure to daily)
    # ... (abbreviated for space, follows same pattern as daily report)
    
    echo "Weekly report generated: $report_file"
}

# Email report distribution
email_report() {
    local report_file="$1"
    local recipients="$2"
    local report_type="${3:-Daily}"
    
    local subject="[$report_type Report] Log Analysis Summary - $(hostname) - $(date +%Y-%m-%d)"
    
    if command -v mutt >/dev/null 2>&1; then
        mutt -e "set content_type=text/html" \
             -s "$subject" \
             -a "$report_file" \
             -- "$recipients" < "$report_file"
    elif command -v mail >/dev/null 2>&1; then
        mail -s "$subject" \
             -a "Content-Type: text/html" \
             "$recipients" < "$report_file"
    fi
}
```

***

<a name="section-7"></a>

# 7. MONITORING INTEGRATION (15 minutes)

## 7.1 Prometheus Metrics Exporter

```bash
#!/bin/bash
# Prometheus metrics exporter for log analyzer

readonly METRICS_PORT=9100
readonly METRICS_FILE="/var/lib/log_analyzer/prometheus_metrics"

# Generate Prometheus metrics
generate_prometheus_metrics() {
    local output=""
    
    # Help and type declarations
    output+="# HELP log_analyzer_events_total Total events processed\n"
    output+="# TYPE log_analyzer_events_total counter\n"
    output+="log_analyzer_events_total $(cat /var/lib/log_analyzer/metrics/total_events 2>/dev/null || echo 0)\n"
    
    output+="\n# HELP log_analyzer_errors_total Total errors detected\n"
    output+="# TYPE log_analyzer_errors_total counter\n"
    output+="log_analyzer_errors_total $(cat /var/lib/log_analyzer/metrics/error_count 2>/dev/null || echo 0)\n"
    
    output+="\n# HELP log_analyzer_alerts_total Total alerts triggered\n"
    output+="# TYPE log_analyzer_alerts_total counter\n"
    
    for severity in CRITICAL HIGH MEDIUM LOW; do
        local count=$(grep -c "\"severity\": \"$severity\"" "$ANOMALY_LOG" 2>/dev/null || echo 0)
        output+="log_analyzer_alerts_total{severity=\"$severity\"} $count\n"
    done
    
    output+="\n# HELP log_analyzer_security_incidents Security incidents detected\n"
    output+="# TYPE log_analyzer_security_incidents counter\n"
    
    for incident_type in SQL_INJECTION XSS_ATTEMPT PATH_TRAVERSAL BRUTE_FORCE; do
        local count=$(grep -c "$incident_type" "$ANOMALY_LOG" 2>/dev/null || echo 0)
        output+="log_analyzer_security_incidents{type=\"$incident_type\"} $count\n"
    done
    
    output+="\n# HELP log_analyzer_alert_resolution_seconds Alert resolution time\n"
    output+="# TYPE log_analyzer_alert_resolution_seconds gauge\n"
    local avg_resolution=$(tail -100 "${ALERT_HISTORY}/resolutions.log" 2>/dev/null | jq -r '.resolution_time_seconds' | awk '{sum+=$1; count++} END {print (count>0)?sum/count:0}')
    output+="log_analyzer_alert_resolution_seconds $avg_resolution\n"
    
    output+="\n# HELP log_analyzer_health_score System health score 0-100\n"
    output+="# TYPE log_analyzer_health_score gauge\n"
    output+="log_analyzer_health_score $(calculate_health_score)\n"
    
    echo -e "$output" > "$METRICS_FILE"
}

calculate_health_score() {
    local score=100
    local errors=$(cat /var/lib/log_analyzer/metrics/error_count 2>/dev/null || echo 0)
    local incidents=$(grep -c "CRITICAL\|SQL_INJECTION\|XSS" "$ANOMALY_LOG" 2>/dev/null || echo 0)
    
    ((errors > 100)) && ((score -= 20))
    ((errors > 500)) && ((score -= 20))
    ((incidents > 0)) && ((score -= 30))
    ((score < 0)) && score=0
    
    echo "$score"
}

# Simple HTTP server for metrics endpoint
start_metrics_server() {
    while true; do
        generate_prometheus_metrics
        
        {
            echo -e "HTTP/1.1 200 OK\r\nContent-Type: text/plain\r\n\r"
            cat "$METRICS_FILE"
        } | nc -l -p "$METRICS_PORT" -q 1
    done
}
```

## 7.2 Grafana Dashboard JSON

```bash
#!/bin/bash
# Generate Grafana dashboard configuration

generate_grafana_dashboard() {
    cat << 'EOF'
{
  "dashboard": {
    "title": "Log Analyzer Dashboard",
    "tags": ["logs", "security", "monitoring"],
    "timezone": "browser",
    "panels": [
      {
        "title": "System Health Score",
        "type": "gauge",
        "gridPos": {"x": 0, "y": 0, "w": 6, "h": 8},
        "targets": [
          {"expr": "log_analyzer_health_score", "legendFormat": "Health"}
        ],
        "options": {
          "reduceOptions": {"values": false, "calcs": ["lastNotNull"]},
          "showThresholdLabels": false,
          "showThresholdMarkers": true
        },
        "fieldConfig": {
          "defaults": {
            "thresholds": {
              "mode": "absolute",
              "steps": [
                {"color": "red", "value": null},
                {"color": "orange", "value": 50},
                {"color": "green", "value": 80}
              ]
            },
            "min": 0,
            "max": 100,
            "unit": "percent"
          }
        }
      },
      {
        "title": "Alerts by Severity",
        "type": "piechart",
        "gridPos": {"x": 6, "y": 0, "w": 6, "h": 8},
        "targets": [
          {"expr": "log_analyzer_alerts_total", "legendFormat": "{{severity}}"}
        ]
      },
      {
        "title": "Events Over Time",
        "type": "timeseries",
        "gridPos": {"x": 12, "y": 0, "w": 12, "h": 8},
        "targets": [
          {"expr": "rate(log_analyzer_events_total[5m])", "legendFormat": "Events/sec"}
        ]
      },
      {
        "title": "Security Incidents",
        "type": "stat",
        "gridPos": {"x": 0, "y": 8, "w": 24, "h": 4},
        "targets": [
          {"expr": "sum(log_analyzer_security_incidents)", "legendFormat": "Total Incidents"}
        ],
        "options": {"colorMode": "value", "graphMode": "area"}
      }
    ],
    "refresh": "30s",
    "schemaVersion": 30
  }
}
EOF
}
```

## 7.3 Webhook Integration

```bash
#!/bin/bash
# Generic webhook integration

send_webhook() {
    local webhook_url="$1"
    local payload="$2"
    local method="${3:-POST}"
    local content_type="${4:-application/json}"
    
    curl -s -X "$method" \
        -H "Content-Type: $content_type" \
        -d "$payload" \
        "$webhook_url"
}

# Integration with common tools
send_to_datadog() {
    local api_key="$DATADOG_API_KEY"
    local metric_name="$1"
    local value="$2"
    local tags="${3:-}"
    
    local payload=$(cat << EOF
{
    "series": [{
        "metric": "log_analyzer.$metric_name",
        "points": [[$(date +%s), $value]],
        "type": "gauge",
        "host": "$(hostname)",
        "tags": [$tags]
    }]
}
EOF
)
    
    curl -s -X POST \
        -H "DD-API-KEY: $api_key" \
        -H "Content-Type: application/json" \
        -d "$payload" \
        "https://api.datadoghq.com/api/v1/series"
}

send_to_splunk() {
    local hec_url="$SPLUNK_HEC_URL"
    local hec_token="$SPLUNK_HEC_TOKEN"
    local event="$1"
    
    local payload=$(cat << EOF
{
    "event": $event,
    "sourcetype": "log_analyzer",
    "host": "$(hostname)",
    "time": $(date +%s)
}
EOF
)
    
    curl -s -k -X POST \
        -H "Authorization: Splunk $hec_token" \
        -d "$payload" \
        "$hec_url"
}

send_to_elasticsearch() {
    local es_url="${ELASTICSEARCH_URL:-http://localhost:9200}"
    local index="log-analyzer-$(date +%Y.%m.%d)"
    local document="$1"
    
    curl -s -X POST \
        -H "Content-Type: application/json" \
        -d "$document" \
        "${es_url}/${index}/_doc"
}
```

***

<a name="section-8"></a>

# 8. COMPLETE PRODUCTION SYSTEM (20 minutes)

## 8.1 Full Implementation Script

```bash
#!/bin/bash
#===============================================================================
# LOG ANALYZER AND ALERTING SYSTEM
# Production-ready implementation
# Version: 2.0.0
#===============================================================================

set -euo pipefail

#-------------------------------------------------------------------------------
# CONFIGURATION
#-------------------------------------------------------------------------------

readonly VERSION="2.0.0"
readonly SCRIPT_NAME=$(basename "$0")
readonly SCRIPT_DIR=$(dirname "$(readlink -f "$0")")

# Directories
readonly BASE_DIR="/var/lib/log_analyzer"
readonly CONFIG_DIR="/etc/log_analyzer"
readonly LOG_DIR="/var/log/log_analyzer"
readonly RUN_DIR="/var/run/log_analyzer"

# Files
readonly PID_FILE="${RUN_DIR}/log_analyzer.pid"
readonly MAIN_LOG="${LOG_DIR}/analyzer.log"
readonly ANOMALY_LOG="${LOG_DIR}/anomalies.log"

# Create directories
mkdir -p "$BASE_DIR"/{metrics,baselines,reports,escalations,alert_history}
mkdir -p "$CONFIG_DIR" "$LOG_DIR" "$RUN_DIR"

#-------------------------------------------------------------------------------
# LOGGING
#-------------------------------------------------------------------------------

log() {
    local level="$1"
    shift
    local message="$*"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    
    echo "[$timestamp] [$level] $message" | tee -a "$MAIN_LOG"
}

log_info() { log "INFO" "$@"; }
log_warn() { log "WARN" "$@"; }
log_error() { log "ERROR" "$@"; }
log_debug() { [[ "${DEBUG:-false}" == "true" ]] && log "DEBUG" "$@"; }

#-------------------------------------------------------------------------------
# SIGNAL HANDLING
#-------------------------------------------------------------------------------

cleanup() {
    log_info "Shutting down log analyzer..."
    
    # Kill child processes
    jobs -p | xargs -r kill 2>/dev/null || true
    
    # Remove PID file
    rm -f "$PID_FILE"
    
    # Flush pending alerts
    flush_all_aggregated 2>/dev/null || true
    
    log_info "Shutdown complete"
    exit 0
}

trap cleanup SIGINT SIGTERM SIGHUP

#-------------------------------------------------------------------------------
# CONFIGURATION MANAGEMENT
#-------------------------------------------------------------------------------

load_configuration() {
    local config_file="${CONFIG_DIR}/analyzer.conf"
    
    # Default configuration
    declare -gA CONFIG=(
        [LOG_SOURCES]="/var/log/syslog,/var/log/auth.log"
        [ALERT_EMAIL]=""
        [SLACK_WEBHOOK]=""
        [ANOMALY_THRESHOLD]="2.5"
        [ALERT_COOLDOWN]="300"
        [BATCH_SIZE]="100"
        [FLUSH_INTERVAL]="5"
    )
    
    if [[ -f "$config_file" ]]; then
        while IFS='=' read -r key value; do
            [[ "$key" =~ ^[[:space:]]*# ]] && continue
            [[ -z "$key" ]] && continue
            CONFIG["$key"]="${value//\"/}"
        done < "$config_file"
    fi
    
    # Export for child processes
    export LOG_ANALYZER_CONFIG_LOADED=true
}

#-------------------------------------------------------------------------------
# LOG SOURCE MANAGEMENT
#-------------------------------------------------------------------------------

declare -A LOG_FOLLOWERS

start_log_followers() {
    log_info "Starting log followers..."
    
    IFS=',' read -ra sources <<< "${CONFIG[LOG_SOURCES]}"
    
    for source in "${sources[@]}"; do
        source=$(echo "$source" | xargs)  # Trim whitespace
        
        if [[ -f "$source" ]]; then
            local name=$(basename "$source" | tr '.' '_')
            follow_log_source "$name" "$source" &
            LOG_FOLLOWERS["$name"]=$!
            log_info "Started follower for $source (PID: ${LOG_FOLLOWERS[$name]})"
        else
            log_warn "Log source not found: $source"
        fi
    done
}

follow_log_source() {
    local name="$1"
    local path="$2"
    local state_file="${BASE_DIR}/state/${name}.pos"
    
    mkdir -p "$(dirname "$state_file")"
    
    # Start from end of file if no state
    local start_line=0
    [[ -f "$state_file" ]] && start_line=$(cat "$state_file")
    
    tail -F -n +$((start_line + 1)) "$path" 2>/dev/null | while read -r line; do
        process_log_line "$name" "$line"
        ((start_line++))
        echo "$start_line" > "$state_file"
    done
}

#-------------------------------------------------------------------------------
# LOG PROCESSING
#-------------------------------------------------------------------------------

declare -a PROCESSING_BUFFER
declare -i BUFFER_COUNT=0
declare -i LAST_FLUSH=$(date +%s)

process_log_line() {
    local source="$1"
    local line="$2"
    local timestamp=$(date +%s)
    
    # Detect log format and parse
    local parsed
    parsed=$(detect_and_parse "$line")
    
    # Check for anomalies
    local anomalies
    anomalies=$(detect_anomalies "$source" "$line" "$parsed")
    
    if [[ -n "$anomalies" ]]; then
        handle_anomalies "$source" "$timestamp" "$line" "$anomalies"
    fi
    
    # Buffer for batch processing
    PROCESSING_BUFFER+=("$source|$timestamp|$line|$anomalies")
    ((BUFFER_COUNT++))
    
    # Flush buffer if needed
    local current_time=$(date +%s)
    if ((BUFFER_COUNT >= ${CONFIG[BATCH_SIZE]})) || \
       ((current_time - LAST_FLUSH >= ${CONFIG[FLUSH_INTERVAL]})); then
        flush_buffer
    fi
    
    # Update metrics
    update_metrics "$source" "$anomalies"
}

detect_and_parse() {
    local line="$1"
    
    # JSON format
    if [[ "$line" =~ ^\{.*\}$ ]]; then
        echo "format=json"
        echo "$line" | jq -r 'to_entries | .[] | "\(.key)=\(.value)"' 2>/dev/null
        return
    fi
    
    # Syslog format
    if [[ "$line" =~ ^[A-Z][a-z]{2}\ [0-9 ]{2}\ [0-9:]+\ ([^\ ]+)\ ([^\[]+)\[([0-9]+)\]:\ (.*)$ ]]; then
        echo "format=syslog"
        echo "hostname=${BASH_REMATCH[1]}"
        echo "service=${BASH_REMATCH[2]}"
        echo "pid=${BASH_REMATCH[3]}"
        echo "message=${BASH_REMATCH[4]}"
        return
    fi
    
    # Apache/Nginx combined
    if [[ "$line" =~ ^([0-9.]+)\ -\ ([^\ ]+)\ \[([^\]]+)\]\ \"([^\"]+)\"\ ([0-9]+)\ ([0-9-]+) ]]; then
        echo "format=apache"
        echo "ip=${BASH_REMATCH[1]}"
        echo "user=${BASH_REMATCH[2]}"
        echo "time=${BASH_REMATCH[3]}"
        echo "request=${BASH_REMATCH[4]}"
        echo "status=${BASH_REMATCH[5]}"
        echo "bytes=${BASH_REMATCH[6]}"
        return
    fi
    
    echo "format=unknown"
    echo "raw=$line"
}

detect_anomalies() {
    local source="$1"
    local line="$2"
    local parsed="$3"
    local anomalies=""
    
    # Security patterns
    [[ "$line" =~ [Ff]ailed.*([Pp]assword|[Ll]ogin|[Aa]uth) ]] && anomalies+="FAILED_LOGIN,"
    [[ "$line" =~ (UNION|SELECT.*FROM|INSERT.*INTO|DELETE.*FROM|DROP.*TABLE) ]] && anomalies+="SQL_INJECTION,"
    [[ "$line" =~ (\<script|javascript:|onerror=) ]] && anomalies+="XSS_ATTEMPT,"
    [[ "$line" =~ (\.\./\.\.|%2e%2e) ]] && anomalies+="PATH_TRAVERSAL,"
    
    # System health
    [[ "$line" =~ [Nn]o\ space\ left|[Dd]isk.*full ]] && anomalies+="DISK_FULL,"
    [[ "$line" =~ [Oo]ut\ of\ memory|OOM ]] && anomalies+="OOM_KILL,"
    [[ "$line" =~ [Cc]onnection\ refused ]] && anomalies+="CONNECTION_REFUSED,"
    
    # Error levels
    [[ "$line" =~ (CRITICAL|FATAL|EMERGENCY) ]] && anomalies+="CRITICAL_ERROR,"
    [[ "$line" =~ \bERROR\b ]] && anomalies+="ERROR,"
    
    echo "${anomalies%,}"
}

handle_anomalies() {
    local source="$1"
    local timestamp="$2"
    local line="$3"
    local anomalies="$4"
    
    IFS=',' read -ra anomaly_list <<< "$anomalies"
    
    for anomaly in "${anomaly_list[@]}"; do
        local severity
        severity=$(get_anomaly_severity "$anomaly")
        
        # Log anomaly
        local record=$(cat << EOF
{"timestamp":"$(date -Iseconds)","source":"$source","type":"$anomaly","severity":"$severity","line":"${line:0:500}"}
EOF
)
        echo "$record" >> "$ANOMALY_LOG"
        
        # Aggregate or trigger alert
        if [[ "$severity" == "CRITICAL" ]]; then
            trigger_immediate_alert "$anomaly" "$severity" "$line" "$source"
        else
            aggregate_alert "$anomaly" "$severity" "$line" "$source"
        fi
    done
}

get_anomaly_severity() {
    local anomaly="$1"
    
    case "$anomaly" in
        SQL_INJECTION|XSS_ATTEMPT|OOM_KILL|CRITICAL_ERROR)
            echo "CRITICAL"
            ;;
        PATH_TRAVERSAL|DISK_FULL|FAILED_LOGIN)
            echo "HIGH"
            ;;
        CONNECTION_REFUSED|ERROR)
            echo "MEDIUM"
            ;;
        *)
            echo "LOW"
            ;;
    esac
}

#-------------------------------------------------------------------------------
# ALERTING
#-------------------------------------------------------------------------------

declare -A ALERT_AGGREGATION

aggregate_alert() {
    local type="$1"
    local severity="$2"
    local message="$3"
    local source="$4"
    
    local key="${type}_${severity}"
    local now=$(date +%s)
    
    if [[ -n "${ALERT_AGGREGATION[$key]:-}" ]]; then
        IFS='|' read -r first_time count messages <<< "${ALERT_AGGREGATION[$key]}"
        
        if ((now - first_time < ${CONFIG[ALERT_COOLDOWN]})); then
            ((count++))
            ALERT_AGGREGATION[$key]="$first_time|$count|$messages"
            return
        else
            flush_aggregated_alert "$key"
        fi
    fi
    
    ALERT_AGGREGATION[$key]="$now|1|$message"
}

flush_aggregated_alert() {
    local key="$1"
    [[ -z "${ALERT_AGGREGATION[$key]:-}" ]] && return
    
    IFS='|' read -r first_time count messages <<< "${ALERT_AGGREGATION[$key]}"
    IFS='_' read -r type severity <<< "$key"
    
    local alert_message
    if ((count > 1)); then
        alert_message="[AGGREGATED] $count occurrences of $type in last ${CONFIG[ALERT_COOLDOWN]}s"
    else
        alert_message="$messages"
    fi
    
    send_alert "$type" "$severity" "$alert_message"
    
    unset "ALERT_AGGREGATION[$key]"
}

flush_all_aggregated() {
    for key in "${!ALERT_AGGREGATION[@]}"; do
        flush_aggregated_alert "$key"
    done
}

trigger_immediate_alert() {
    local type="$1"
    local severity="$2"
    local message="$3"
    local source="$4"
    
    log_warn "IMMEDIATE ALERT: [$severity] $type from $source"
    send_alert "$type" "$severity" "$message" &
}

send_alert() {
    local type="$1"
    local severity="$2"
    local message="$3"
    
    local alert_id=$(date +%s%N | md5sum | head -c 12)
    
    # Email
    if [[ -n "${CONFIG[ALERT_EMAIL]:-}" ]]; then
        echo "$message" | mail -s "[$severity] $type - $(hostname)" "${CONFIG[ALERT_EMAIL]}" 2>/dev/null || true
    fi
    
    # Slack
    if [[ -n "${CONFIG[SLACK_WEBHOOK]:-}" ]]; then
        local color
        case "$severity" in
            CRITICAL) color="#dc3545" ;;
            HIGH)     color="#fd7e14" ;;
            MEDIUM)   color="#ffc107" ;;
            *)        color="#6c757d" ;;
        esac
        
        local payload=$(cat << EOF
{
    "attachments": [{
        "color": "$color",
        "title": "[$severity] $type",
        "text": "${message:0:500}",
        "footer": "$(hostname) | $alert_id",
        "ts": $(date +%s)
    }]
}
EOF
)
        curl -s -X POST -H "Content-Type: application/json" -d "$payload" "${CONFIG[SLACK_WEBHOOK]}" >/dev/null 2>&1 || true
    fi
    
    # Log alert
    echo "$(date -Iseconds)|$alert_id|$severity|$type|${message:0:200}" >> "${BASE_DIR}/alert_history/alerts.log"
}

#-------------------------------------------------------------------------------
# METRICS
#-------------------------------------------------------------------------------

update_metrics() {
    local source="$1"
    local anomalies="$2"
    
    # Increment counters
    local events_file="${BASE_DIR}/metrics/events_${source}"
    local count=$(($(cat "$events_file" 2>/dev/null || echo 0) + 1))
    echo "$count" > "$events_file"
    
    if [[ -n "$anomalies" ]]; then
        local anomaly_file="${BASE_DIR}/metrics/anomalies_total"
        local acount=$(($(cat "$anomaly_file" 2>/dev/null || echo 0) + 1))
        echo "$acount" > "$anomaly_file"
    fi
}

flush_buffer() {
    [[ ${#PROCESSING_BUFFER[@]} -eq 0 ]] && return
    
    # Batch analysis
    local error_count=0
    local total=${#PROCESSING_BUFFER[@]}
    
    for entry in "${PROCESSING_BUFFER[@]}"; do
        [[ "$entry" == *"ERROR"* ]] && ((error_count++))
    done
    
    # Check for error spike
    if ((total > 10 && error_count * 100 / total > 50)); then
        log_warn "Error spike detected: $error_count/$total ($(( error_count * 100 / total ))%)"
        trigger_immediate_alert "ERROR_SPIKE" "HIGH" "High error rate: $error_count errors in last batch" "aggregated"
    fi
    
    PROCESSING_BUFFER=()
    BUFFER_COUNT=0
    LAST_FLUSH=$(date +%s)
}

#-------------------------------------------------------------------------------
# REPORTING
#-------------------------------------------------------------------------------

generate_report() {
    local report_type="${1:-daily}"
    local report_date="${2:-$(date +%Y-%m-%d)}"
    
    log_info "Generating $report_type report for $report_date"
    
    local report_file="${BASE_DIR}/reports/${report_type}_${report_date}.html"
    
    # Collect metrics
    local total_events=$(cat "${BASE_DIR}"/metrics/events_* 2>/dev/null | awk '{sum+=$1} END {print sum+0}')
    local total_anomalies=$(cat "${BASE_DIR}/metrics/anomalies_total" 2>/dev/null || echo 0)
    local total_alerts=$(wc -l < "${BASE_DIR}/alert_history/alerts.log" 2>/dev/null || echo 0)
    
    # Generate HTML
    cat > "$report_file" << EOF
<!DOCTYPE html>
<html>
<head>
    <title>Log Analyzer Report - $report_date</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; background: #f5f5f5; }
        .container { max-width: 800px; margin: 0 auto; background: white; padding: 30px; border-radius: 10px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
        h1 { color: #333; border-bottom: 2px solid #667eea; padding-bottom: 10px; }
        .metric { display: inline-block; background: #667eea; color: white; padding: 20px 30px; margin: 10px; border-radius: 8px; text-align: center; }
        .metric .value { font-size: 2em; font-weight: bold; }
        .metric .label { font-size: 0.9em; opacity: 0.9; }
    </style>
</head>
<body>
    <div class="container">
        <h1>📊 Log Analyzer Report</h1>
        <p>Report Date: $report_date | Generated: $(date)</p>
        <p>Hostname: $(hostname)</p>
        
        <h2>Summary</h2>
        <div class="metric"><div class="value">$total_events</div><div class="label">Total Events</div></div>
        <div class="metric"><div class="value">$total_anomalies</div><div class="label">Anomalies</div></div>
        <div class="metric"><div class="value">$total_alerts</div><div class="label">Alerts</div></div>
        
        <h2>Recent Alerts</h2>
        <pre>$(tail -20 "${BASE_DIR}/alert_history/alerts.log" 2>/dev/null || echo "No alerts")</pre>
        
        <h2>Recent Anomalies</h2>
        <pre>$(tail -20 "$ANOMALY_LOG" 2>/dev/null || echo "No anomalies")</pre>
    </div>
</body>
</html>
EOF
    
    log_info "Report generated: $report_file"
    echo "$report_file"
}

#-------------------------------------------------------------------------------
# DAEMON MANAGEMENT
#-------------------------------------------------------------------------------

start_daemon() {
    if [[ -f "$PID_FILE" ]]; then
        local existing_pid=$(cat "$PID_FILE")
        if kill -0 "$existing_pid" 2>/dev/null; then
            log_error "Log analyzer already running (PID: $existing_pid)"
            exit 1
        fi
    fi
    
    echo $$ > "$PID_FILE"
    
    log_info "Starting Log Analyzer v${VERSION}..."
    
    load_configuration
    start_log_followers
    
    # Start background tasks
    (while true; do sleep 60; flush_all_aggregated; done) &
    (while true; do sleep 300; generate_report daily >/dev/null; done) &
    
    log_info "Log Analyzer started successfully"
    
    # Main loop
    while true; do
        # Check follower health
        for name in "${!LOG_FOLLOWERS[@]}"; do
            if ! kill -0 "${LOG_FOLLOWERS[$name]}" 2>/dev/null; then
                log_warn "Follower $name died, restarting..."
                local source_path
                source_path=$(grep -E "^${name//_/.}" <<< "${CONFIG[LOG_SOURCES]}" || echo "")
                if [[ -n "$source_path" ]]; then
                    follow_log_source "$name" "$source_path" &
                    LOG_FOLLOWERS["$name"]=$!
                fi
            fi
        done
        
        sleep 30
    done
}

stop_daemon() {
    if [[ -f "$PID_FILE" ]]; then
        local pid=$(cat "$PID_FILE")
        if kill -0 "$pid" 2>/dev/null; then
            log_info "Stopping Log Analyzer (PID: $pid)..."
            kill "$pid"
            sleep 2
            kill -9 "$pid" 2>/dev/null || true
            rm -f "$PID_FILE"
            log_info "Log Analyzer stopped"
        else
            log_warn "Process not running, cleaning up PID file"
            rm -f "$PID_FILE"
        fi
    else
        log_warn "PID file not found"
    fi
}

status_daemon() {
    if [[ -f "$PID_FILE" ]]; then
        local pid=$(cat "$PID_FILE")
        if kill -0 "$pid" 2>/dev/null; then
            echo "Log Analyzer is running (PID: $pid)"
            echo ""
            echo "Metrics:"
            echo "  Events processed: $(cat "${BASE_DIR}"/metrics/events_* 2>/dev/null | awk '{sum+=$1} END {print sum+0}')"
            echo "  Anomalies detected: $(cat "${BASE_DIR}/metrics/anomalies_total" 2>/dev/null || echo 0)"
            echo "  Alerts sent: $(wc -l < "${BASE_DIR}/alert_history/alerts.log" 2>/dev/null || echo 0)"
            return 0
        fi
    fi
    
    echo "Log Analyzer is not running"
    return 1
}

#-------------------------------------------------------------------------------
# CLI INTERFACE
#-------------------------------------------------------------------------------

show_help() {
    cat << EOF
Log Analyzer and Alerting System v${VERSION}

Usage: $SCRIPT_NAME <command> [options]

Commands:
    start           Start the log analyzer daemon
    stop            Stop the log analyzer daemon
    restart         Restart the log analyzer daemon
    status          Show daemon status and metrics
    report [type]   Generate report (daily/weekly/monthly)
    test            Run self-tests
    help            Show this help message

Options:
    -c, --config    Path to configuration file
    -d, --debug     Enable debug logging
    -v, --version   Show version

Examples:
    $SCRIPT_NAME start
    $SCRIPT_NAME report daily
    $SCRIPT_NAME status
EOF
}

#-------------------------------------------------------------------------------
# MAIN
#-------------------------------------------------------------------------------

main() {
    local command="${1:-help}"
    shift || true
    
    case "$command" in
        start)
            start_daemon
            ;;
        stop)
            stop_daemon
            ;;
        restart)
            stop_daemon
            sleep 2
            start_daemon
            ;;
        status)
            status_daemon
            ;;
        report)
            load_configuration
            generate_report "${1:-daily}"
            ;;
        test)
            run_tests
            ;;
        -v|--version)
            echo "Log Analyzer v${VERSION}"
            ;;
        help|-h|--help)
            show_help
            ;;
        *)
            echo "Unknown command: $command"
            show_help
            exit 1
            ;;
    esac
}

# Run if executed directly
[[ "${BASH_SOURCE[0]}" == "${0}" ]] && main "$@"
```

***

<a name="section-9"></a>

# 9. MINI CHALLENGES & FINAL TEST (15 minutes)

## 9.1 Mini Challenges

### Challenge 1: Parse Custom Log Format

```bash
#!/bin/bash
# CHALLENGE: Parse this custom log format and extract fields
# Format: [TIMESTAMP] {LEVEL} <SERVICE> (USER@IP): MESSAGE

log_line='[2023-10-15 14:30:45] {ERROR} <api-gateway> (admin@192.168.1.100): Database connection timeout'

# YOUR SOLUTION:
parse_custom_format() {
    local line="$1"
    local regex='^\[([^\]]+)\] \{([A-Z]+)\} <([^>]+)> \(([^@]+)@([^)]+)\): (.*)$'
    
    if [[ "$line" =~ $regex ]]; then
        echo "Timestamp: ${BASH_REMATCH[1]}"
        echo "Level: ${BASH_REMATCH[2]}"
        echo "Service: ${BASH_REMATCH[3]}"
        echo "User: ${BASH_REMATCH[4]}"
        echo "IP: ${BASH_REMATCH[5]}"
        echo "Message: ${BASH_REMATCH[6]}"
        return 0
    else
        echo "Parse failed"
        return 1
    fi
}

# Test
echo "=== Challenge 1: Custom Log Parser ==="
parse_custom_format "$log_line"
```

**Expected Output:**

```
=== Challenge 1: Custom Log Parser ===
Timestamp: 2023-10-15 14:30:45
Level: ERROR
Service: api-gateway
User: admin
IP: 192.168.1.100
Message: Database connection timeout
```

### Challenge 2: Detect Brute Force Attack

```bash
#!/bin/bash
# CHALLENGE: Detect brute force attack from these logs
# Alert if same IP has 5+ failed logins in 60 seconds

test_logs=(
    "Oct 15 14:30:01 server sshd[1234]: Failed password for root from 192.168.1.50"
    "Oct 15 14:30:05 server sshd[1235]: Failed password for admin from 192.168.1.50"
    "Oct 15 14:30:10 server sshd[1236]: Failed password for root from 192.168.1.50"
    "Oct 15 14:30:15 server sshd[1237]: Failed password for user from 192.168.1.50"
    "Oct 15 14:30:20 server sshd[1238]: Failed password for guest from 192.168.1.50"
    "Oct 15 14:30:25 server sshd[1239]: Accepted password for admin from 10.0.0.1"
)

# YOUR SOLUTION:
detect_brute_force() {
    declare -A ip_failures
    local threshold=5
    
    for log in "${test_logs[@]}"; do
        if [[ "$log" =~ [Ff]ailed.*from\ ([0-9.]+) ]]; then
            local ip="${BASH_REMATCH[1]}"
            ((ip_failures[$ip]++))
            
            if ((ip_failures[$ip] >= threshold)); then
                echo "🚨 BRUTE FORCE DETECTED: $ip (${ip_failures[$ip]} failures)"
                return 0
            fi
        fi
    done
    
    echo "No brute force detected"
    return 1
}

# Test
echo "=== Challenge 2: Brute Force Detection ==="
detect_brute_force
```

**Expected Output:**

```
=== Challenge 2: Brute Force Detection ===
🚨 BRUTE FORCE DETECTED: 192.168.1.50 (5 failures)
```

### Challenge 3: Calculate Error Rate Anomaly

```bash
#!/bin/bash
# CHALLENGE: Given hourly error counts, detect anomalies (>2 std devs from mean)

hourly_errors=(12 15 10 14 11 13 12 14 10 11 85 13 12 14)  # Hour 10 is anomaly

# YOUR SOLUTION:
detect_error_anomaly() {
    local -a values=("$@")
    local count=${#values[@]}
    
    # Calculate mean
    local sum=0
    for val in "${values[@]}"; do
        ((sum += val))
    done
    local mean=$((sum / count))
    
    # Calculate std dev
    local sq_sum=0
    for val in "${values[@]}"; do
        local diff=$((val - mean))
        ((sq_sum += diff * diff))
    done
    local variance=$((sq_sum / count))
    local std_dev=$(echo "sqrt($variance)" | bc)
    
    # Threshold
    local upper=$((mean + 2 * std_dev))
    local lower=$((mean - 2 * std_dev))
    
    echo "Mean: $mean, Std Dev: $std_dev, Threshold: $lower - $upper"
    echo ""
    
    # Check each value
    for i in "${!values[@]}"; do
        if ((values[i] > upper || values[i] < lower)); then
            echo "⚠️ ANOMALY at hour $i: ${values[i]} (outside $lower-$upper)"
        fi
    done
}

# Test
echo "=== Challenge 3: Error Rate Anomaly ==="
detect_error_anomaly "${hourly_errors[@]}"
```

**Expected Output:**

```
=== Challenge 3: Error Rate Anomaly ===
Mean: 16, Std Dev: 19, Threshold: -22 - 54

⚠️ ANOMALY at hour 10: 85 (outside -22-54)
```

## 9.2 Final Test: Security Incident Auto-Response

````bash
#!/bin/bash
#===============================================================================
# FINAL TEST: Security Incident Detection and Auto-Response System
# Requirements:
# 1. Monitor multiple log files simultaneously
# 2. Detect: SQL injection, XSS, brute force, unusual hours access
# 3. Auto-respond: Block IP, send alerts, generate report
# 4. Target: Detect and respond within 10 seconds
#===============================================================================

# TEST CONFIGURATION
readonly TEST_DIR="/tmp/log_analyzer_test_$$"
readonly TEST_LOG_1="${TEST_DIR}/app.log"
readonly TEST_LOG_2="${TEST_DIR}/auth.log"
readonly BLOCKED_IPS="${TEST_DIR}/blocked_ips.txt"
readonly INCIDENT_REPORT="${TEST_DIR}/incidents.json"
readonly ALERT_LOG="${TEST_DIR}/alerts.log"

# Initialize test environment
setup_test() {
    mkdir -p "$TEST_DIR"
    touch "$TEST_LOG_1" "$TEST_LOG_2" "$BLOCKED_IPS" "$INCIDENT_REPORT" "$ALERT_LOG"
    echo "Test environment ready: $TEST_DIR"
}

cleanup_test() {
    rm -rf "$TEST_DIR"
}

#-------------------------------------------------------------------------------
# SECURITY INCIDENT DETECTOR (Your implementation)
#-------------------------------------------------------------------------------

declare -A INCIDENT_TRACKING
declare -i INCIDENT_COUNT=0

# Main monitoring function
monitor_logs() {
    echo "Starting security monitor..."
    
    # Start monitoring both logs in background
    tail -F "$TEST_LOG_1" 2>/dev/null | process_log "app" &
    local pid1=$!
    
    tail -F "$TEST_LOG_2" 2>/dev/null | process_log "auth" &
    local pid2=$!
    
    # Return PIDs for cleanup
    echo "$pid1 $pid2"
}

process_log() {
    local source="$1"
    
    while IFS= read -r line; do
        local timestamp=$(date +%s.%N)
        local incident=""
        local severity=""
        local ip=""
        
        # Extract IP
        [[ "$line" =~ ([0-9]+\.[0-9]+\.[0-9]+\.[0-9]+) ]] && ip="${BASH_REMATCH[1]}"
        
        # DETECTION RULES
        
        # SQL Injection
        if [[ "$line" =~ (UNION|SELECT.*FROM|INSERT.*INTO|DELETE.*FROM|DROP|--|\') ]]; then
            incident="SQL_INJECTION"
            severity="CRITICAL"
        fi
        
        # XSS Attempt
        if [[ "$line" =~ (\<script|javascript:|onerror=|onclick=) ]]; then
            incident="XSS_ATTEMPT"
            severity="CRITICAL"
        fi
        
        # Brute Force (track failures per IP)
        if [[ "$line" =~ [Ff]ailed.*([Pp]assword|[Ll]ogin|[Aa]uth) ]]; then
            if [[ -n "$ip" ]]; then
                local key="brute_${ip}"
                ((INCIDENT_TRACKING[$key]++))
                
                if ((INCIDENT_TRACKING[$key] >= 5)); then
                    incident="BRUTE_FORCE"
                    severity="HIGH"
                fi
            fi
        fi
        
        # Unusual Hours Access (outside 6AM-10PM)
        local hour=$(date +%H)
        if ((hour < 6 || hour >= 22)); then
            if [[ "$line" =~ [Aa]ccepted|[Ss]uccess|[Ll]ogin ]]; then
                incident="UNUSUAL_HOURS_ACCESS"
                severity="MEDIUM"
            fi
        fi
        
        # Process incident if detected
        if [[ -n "$incident" ]]; then
            ((INCIDENT_COUNT++))
            handle_incident "$incident" "$severity" "$ip" "$line" "$timestamp"
        fi
    done
}

handle_incident() {
    local incident_type="$1"
    local severity="$2"
    local ip="$3"
    local raw_log="$4"
    local timestamp="$5"
    
    local incident_id="INC-$(date +%Y%m%d)-${INCIDENT_COUNT}"
    local response_start=$(date +%s.%N)
    
    echo "[$(date -Iseconds)] INCIDENT: $incident_type from $ip (Severity: $severity)" | tee -a "$ALERT_LOG"
    
    # AUTO-RESPONSE ACTIONS
    
    # 1. Block IP for critical/high severity
    if [[ "$severity" == "CRITICAL" || "$severity" == "HIGH" ]]; then
        block_ip "$ip" "$incident_type"
    fi
    
    # 2. Send alert
    send_test_alert "$incident_id" "$incident_type" "$severity" "$ip"
    
    # 3. Log incident
    log_incident "$incident_id" "$incident_type" "$severity" "$ip" "$raw_log" "$timestamp"
    
    # Calculate response time
    local response_end=$(date +%s.%N)
    local response_time=$(echo "$response_end - $response_start" | bc)
    
    echo "  Response completed in ${response_time}s"
}

block_ip() {
    local ip="$1"
    local reason="$2"
    
    # Check if already blocked
    if grep -q "^$ip$" "$BLOCKED_IPS" 2>/dev/null; then
        return
    fi
    
    echo "$ip" >> "$BLOCKED_IPS"
    echo "  🚫 BLOCKED IP: $ip (Reason: $reason)"
    
    # In production, you would run:
    # iptables -A INPUT -s "$ip" -j DROP
    # or update firewall rules
}

send_test_alert() {
    local incident_id="$1"
    local type="$2"
    local severity="$3"
    local ip="$4"
    
    echo "  📧 ALERT SENT: [$severity] $type from $ip (ID: $incident_id)"
}

log_incident() {
    local incident_id="$1"
    local type="$2"
    local severity="$3"
    local ip="$4"
    local raw_log="$5"
    local timestamp="$6"
    
    local record=$(cat << EOF
{"id":"$incident_id","type":"$type","severity":"$severity","ip":"$ip","timestamp":"$timestamp","raw":"${raw_log:0:200}"}
EOF
)
    echo "$record" >> "$INCIDENT_REPORT"
}

#-------------------------------------------------------------------------------
# TEST SCENARIOS
#-------------------------------------------------------------------------------

run_attack_simulation() {
    echo ""
    echo "=== ATTACK SIMULATION STARTING ==="
    echo ""
    
    # Simulate SQL Injection
    echo "Simulating SQL Injection attack..."
    sleep 0.5
    echo '2023-10-15 14:30:00 192.168.1.100 GET /api/users?id=1 UNION SELECT * FROM passwords--' >> "$TEST_LOG_1"
    sleep 1
    
    # Simulate XSS
    echo "Simulating XSS attack..."
    sleep 0.5
    echo '2023-10-15 14:30:05 192.168.1.101 GET /page?name=<script>alert("XSS")</script>' >> "$TEST_LOG_1"
    sleep 1
    
    # Simulate Brute Force
    echo "Simulating brute force attack..."
    for i in {1..6}; do
        echo "Oct 15 14:30:$((10+i)) server sshd[1234]: Failed password for root from 192.168.1.102" >> "$TEST_LOG_2"
        sleep 0.2
    done
    sleep 1
    
    # Final summary
    echo ""
    echo "=== SIMULATION COMPLETE ==="
}

verify_results() {
    echo ""
    echo "=== VERIFICATION ==="
    echo ""
    
    local pass=true
    
    # Check blocked IPs
    echo "Blocked IPs:"
    cat "$BLOCKED_IPS" 2>/dev/null || echo "  (none)"
    
    local blocked_count=$(wc -l < "$BLOCKED_IPS" 2>/dev/null || echo 0)
    if ((blocked_count >= 2)); then
        echo "✅ IP blocking: PASS ($blocked_count IPs blocked)"
    else
        echo "❌ IP blocking: FAIL (expected >=2, got $blocked_count)"
        pass=false
    fi
    
    # Check incidents logged
    echo ""
    echo "Incidents logged:"
    cat "$INCIDENT_REPORT" 2>/dev/null | head -5 || echo "  (none)"
    
    local incident_count=$(wc -l < "$INCIDENT_REPORT" 2>/dev/null || echo 0)
    if ((incident_count >= 3)); then
        echo "✅ Incident logging: PASS ($incident_count incidents)"
    else
        echo "❌ Incident logging: FAIL (expected >=3, got $incident_count)"
        pass=false
    fi
    
    # Check alerts
    echo ""
    echo "Alerts sent:"
    grep -c "ALERT SENT" "$ALERT_LOG" 2>/dev/null || echo "0"
    
    echo ""
    if [[ "$pass" == true ]]; then
        echo "🎉 FINAL TEST: PASSED"
    else
        echo "❌ FINAL TEST: FAILED"
    fi
}

#-------------------------------------------------------------------------------
# RUN TEST
#-------------------------------------------------------------------------------

run_final_test() {
    trap cleanup_test EXIT
    
    echo "============================================"
    echo "  FINAL

```bash
#-------------------------------------------------------------------------------
# RUN TEST (Continued)
#-------------------------------------------------------------------------------

run_final_test() {
    trap cleanup_test EXIT
    
    echo "============================================"
    echo "  FINAL TEST: SECURITY INCIDENT AUTO-RESPONSE"
    echo "============================================"
    echo ""
    echo "Requirements:"
    echo "  1. Monitor multiple log files simultaneously"
    echo "  2. Detect: SQL injection, XSS, brute force, unusual hours"
    echo "  3. Auto-respond: Block IP, send alerts, generate report"
    echo "  4. Target: Detect and respond within 10 seconds"
    echo ""
    
    # Setup
    setup_test
    
    # Start monitoring
    local pids
    pids=$(monitor_logs)
    
    # Give monitors time to start
    sleep 2
    
    # Record start time
    local test_start=$(date +%s.%N)
    
    # Run attack simulation
    run_attack_simulation
    
    # Wait for processing
    sleep 3
    
    # Record end time
    local test_end=$(date +%s.%N)
    local total_time=$(echo "$test_end - $test_start" | bc)
    
    # Stop monitors
    for pid in $pids; do
        kill "$pid" 2>/dev/null || true
    done
    
    # Verify results
    verify_results
    
    echo ""
    echo "Total test time: ${total_time}s"
    
    if (( $(echo "$total_time < 10" | bc -l) )); then
        echo "✅ Response time: PASS (under 10 seconds)"
    else
        echo "❌ Response time: FAIL (exceeded 10 seconds)"
    fi
    
    echo ""
    echo "============================================"
}

# Execute final test
run_final_test
````

## 9.3 Complete Self-Test Suite

```bash
#!/bin/bash
#===============================================================================
# COMPREHENSIVE SELF-TEST SUITE FOR LOG ANALYZER
#===============================================================================

readonly TEST_RESULTS_DIR="/tmp/log_analyzer_tests_$$"
declare -i TESTS_PASSED=0
declare -i TESTS_FAILED=0

# Test utilities
setup_tests() {
    mkdir -p "$TEST_RESULTS_DIR"
    echo "Test suite initialized: $TEST_RESULTS_DIR"
}

cleanup_tests() {
    rm -rf "$TEST_RESULTS_DIR"
}

assert_equals() {
    local expected="$1"
    local actual="$2"
    local test_name="$3"
    
    if [[ "$expected" == "$actual" ]]; then
        echo "✅ PASS: $test_name"
        ((TESTS_PASSED++))
        return 0
    else
        echo "❌ FAIL: $test_name"
        echo "   Expected: $expected"
        echo "   Actual:   $actual"
        ((TESTS_FAILED++))
        return 1
    fi
}

assert_contains() {
    local haystack="$1"
    local needle="$2"
    local test_name="$3"
    
    if [[ "$haystack" == *"$needle"* ]]; then
        echo "✅ PASS: $test_name"
        ((TESTS_PASSED++))
        return 0
    else
        echo "❌ FAIL: $test_name"
        echo "   Expected to contain: $needle"
        echo "   Actual: $haystack"
        ((TESTS_FAILED++))
        return 1
    fi
}

assert_true() {
    local condition="$1"
    local test_name="$2"
    
    if eval "$condition"; then
        echo "✅ PASS: $test_name"
        ((TESTS_PASSED++))
        return 0
    else
        echo "❌ FAIL: $test_name"
        ((TESTS_FAILED++))
        return 1
    fi
}

#-------------------------------------------------------------------------------
# TEST: Log Format Detection
#-------------------------------------------------------------------------------

test_log_format_detection() {
    echo ""
    echo "=== TEST SUITE: Log Format Detection ==="
    echo ""
    
    # Test JSON detection
    local json_log='{"timestamp":"2023-10-15T14:30:00Z","level":"ERROR","message":"test"}'
    local result=$(detect_format "$json_log")
    assert_equals "json" "$result" "JSON format detection"
    
    # Test Syslog detection
    local syslog_log="Oct 15 14:30:00 server sshd[1234]: Connection accepted"
    result=$(detect_format "$syslog_log")
    assert_equals "syslog" "$result" "Syslog format detection"
    
    # Test Apache detection
    local apache_log='192.168.1.100 - - [15/Oct/2023:14:30:00 +0000] "GET /api HTTP/1.1" 200 1234'
    result=$(detect_format "$apache_log")
    assert_equals "apache" "$result" "Apache format detection"
    
    # Test unknown format
    local unknown_log="This is just plain text"
    result=$(detect_format "$unknown_log")
    assert_equals "unknown" "$result" "Unknown format detection"
}

detect_format() {
    local line="$1"
    
    [[ "$line" =~ ^\{.*\}$ ]] && echo "json" && return
    [[ "$line" =~ ^[A-Z][a-z]{2}\ [0-9 ]{2}\ [0-9:]+\ .+\[.+\]: ]] && echo "syslog" && return
    [[ "$line" =~ ^[0-9.]+\ -\ .+\ \[.+\]\ \".+\" ]] && echo "apache" && return
    echo "unknown"
}

#-------------------------------------------------------------------------------
# TEST: Pattern Detection
#-------------------------------------------------------------------------------

test_pattern_detection() {
    echo ""
    echo "=== TEST SUITE: Pattern Detection ==="
    echo ""
    
    # SQL Injection patterns
    local sqli_tests=(
        "SELECT * FROM users WHERE id=1 UNION SELECT password FROM admins--"
        "1' OR '1'='1"
        "admin'--"
        "INSERT INTO users VALUES(1,2,3)"
        "DROP TABLE users"
    )
    
    for test in "${sqli_tests[@]}"; do
        local result=$(detect_sql_injection "$test")
        assert_equals "true" "$result" "SQL Injection: ${test:0:40}..."
    done
    
    # XSS patterns
    local xss_tests=(
        "<script>alert('XSS')</script>"
        "javascript:alert(document.cookie)"
        "<img onerror=alert(1)>"
        "<body onload=alert('test')>"
    )
    
    for test in "${xss_tests[@]}"; do
        local result=$(detect_xss "$test")
        assert_equals "true" "$result" "XSS: ${test:0:40}..."
    done
    
    # Path traversal
    local traversal_tests=(
        "../../../etc/passwd"
        "..\\..\\..\\windows\\system32"
        "%2e%2e%2f%2e%2e%2fetc/passwd"
    )
    
    for test in "${traversal_tests[@]}"; do
        local result=$(detect_path_traversal "$test")
        assert_equals "true" "$result" "Path Traversal: ${test:0:40}..."
    done
    
    # False positives (should NOT detect)
    local safe_inputs=(
        "SELECT your menu choice"
        "The script was written by John"
        "Navigate to parent directory"
    )
    
    for test in "${safe_inputs[@]}"; do
        local sqli=$(detect_sql_injection "$test")
        local xss=$(detect_xss "$test")
        assert_equals "false" "$sqli" "Safe input (not SQLi): ${test:0:30}..."
        assert_equals "false" "$xss" "Safe input (not XSS): ${test:0:30}..."
    done
}

detect_sql_injection() {
    local input="$1"
    if [[ "$input" =~ (UNION|SELECT.*FROM|INSERT.*INTO|DELETE.*FROM|DROP.*TABLE|\'.*OR.*\'|\'.*--) ]]; then
        echo "true"
    else
        echo "false"
    fi
}

detect_xss() {
    local input="$1"
    if [[ "$input" =~ (\<script|javascript:|onerror=|onload=|\<img.*=) ]]; then
        echo "true"
    else
        echo "false"
    fi
}

detect_path_traversal() {
    local input="$1"
    if [[ "$input" =~ (\.\./\.\.|\.\.\\|%2e%2e) ]]; then
        echo "true"
    else
        echo "false"
    fi
}

#-------------------------------------------------------------------------------
# TEST: Anomaly Detection
#-------------------------------------------------------------------------------

test_anomaly_detection() {
    echo ""
    echo "=== TEST SUITE: Anomaly Detection ==="
    echo ""
    
    # Test statistical anomaly (z-score based)
    local normal_values=(10 12 11 10 13 11 12 10 11 12)
    local anomaly_value=50
    
    local result=$(is_statistical_anomaly "${normal_values[*]}" "$anomaly_value" 2)
    assert_equals "true" "$result" "Statistical anomaly detection (value: 50, baseline: ~11)"
    
    local normal_test_value=11
    result=$(is_statistical_anomaly "${normal_values[*]}" "$normal_test_value" 2)
    assert_equals "false" "$result" "Normal value not flagged (value: 11)"
    
    # Test rate limiting detection
    local request_count=150
    local time_window=60
    local threshold=100
    
    result=$(is_rate_exceeded "$request_count" "$time_window" "$threshold")
    assert_equals "true" "$result" "Rate limit exceeded (150 req/60s, limit 100)"
    
    request_count=50
    result=$(is_rate_exceeded "$request_count" "$time_window" "$threshold")
    assert_equals "false" "$result" "Rate limit not exceeded (50 req/60s)"
    
    # Test brute force detection
    local failure_count=10
    local threshold=5
    result=$(is_brute_force "$failure_count" "$threshold")
    assert_equals "true" "$result" "Brute force detected (10 failures, threshold 5)"
    
    failure_count=3
    result=$(is_brute_force "$failure_count" "$threshold")
    assert_equals "false" "$result" "Brute force not detected (3 failures)"
}

is_statistical_anomaly() {
    local values_str="$1"
    local test_value="$2"
    local threshold="$3"
    
    read -ra values <<< "$values_str"
    local count=${#values[@]}
    
    # Calculate mean
    local sum=0
    for val in "${values[@]}"; do
        ((sum += val))
    done
    local mean=$((sum / count))
    
    # Calculate std dev
    local sq_sum=0
    for val in "${values[@]}"; do
        local diff=$((val - mean))
        ((sq_sum += diff * diff))
    done
    local std_dev=$(echo "sqrt($sq_sum / $count)" | bc)
    ((std_dev == 0)) && std_dev=1
    
    # Calculate z-score
    local z_score=$(echo "scale=2; ($test_value - $mean) / $std_dev" | bc)
    local abs_z=$(echo "$z_score" | tr -d '-')
    
    if (( $(echo "$abs_z > $threshold" | bc -l) )); then
        echo "true"
    else
        echo "false"
    fi
}

is_rate_exceeded() {
    local count="$1"
    local window="$2"
    local threshold="$3"
    
    local rate=$((count * 60 / window))
    if ((rate > threshold)); then
        echo "true"
    else
        echo "false"
    fi
}

is_brute_force() {
    local failures="$1"
    local threshold="$2"
    
    if ((failures >= threshold)); then
        echo "true"
    else
        echo "false"
    fi
}

#-------------------------------------------------------------------------------
# TEST: Alert System
#-------------------------------------------------------------------------------

test_alert_system() {
    echo ""
    echo "=== TEST SUITE: Alert System ==="
    echo ""
    
    local alert_file="${TEST_RESULTS_DIR}/test_alerts.log"
    
    # Test alert severity routing
    local result=$(get_alert_priority "CRITICAL")
    assert_equals "1" "$result" "CRITICAL priority is 1"
    
    result=$(get_alert_priority "HIGH")
    assert_equals "2" "$result" "HIGH priority is 2"
    
    result=$(get_alert_priority "MEDIUM")
    assert_equals "3" "$result" "MEDIUM priority is 3"
    
    result=$(get_alert_priority "LOW")
    assert_equals "4" "$result" "LOW priority is 4"
    
    # Test cooldown logic
    local cooldown_file="${TEST_RESULTS_DIR}/cooldown_test"
    echo "$(date +%s)" > "$cooldown_file"
    
    result=$(is_in_cooldown_period "$cooldown_file" 300)
    assert_equals "true" "$result" "Cooldown active (just set)"
    
    # Simulate old cooldown
    echo "$(($(date +%s) - 400))" > "$cooldown_file"
    result=$(is_in_cooldown_period "$cooldown_file" 300)
    assert_equals "false" "$result" "Cooldown expired (400s > 300s)"
    
    # Test alert aggregation
    local agg_result=$(test_alert_aggregation)
    assert_contains "$agg_result" "aggregated" "Alert aggregation works"
}

get_alert_priority() {
    local severity="$1"
    case "$severity" in
        CRITICAL) echo "1" ;;
        HIGH)     echo "2" ;;
        MEDIUM)   echo "3" ;;
        LOW)      echo "4" ;;
        *)        echo "5" ;;
    esac
}

is_in_cooldown_period() {
    local cooldown_file="$1"
    local cooldown_seconds="$2"
    
    if [[ -f "$cooldown_file" ]]; then
        local last_alert=$(cat "$cooldown_file")
        local now=$(date +%s)
        local elapsed=$((now - last_alert))
        
        if ((elapsed < cooldown_seconds)); then
            echo "true"
            return
        fi
    fi
    echo "false"
}

test_alert_aggregation() {
    declare -A agg_buffer
    local key="TEST_ALERT"
    
    # Simulate multiple alerts
    for i in {1..5}; do
        if [[ -z "${agg_buffer[$key]:-}" ]]; then
            agg_buffer[$key]="1"
        else
            ((agg_buffer[$key]++))
        fi
    done
    
    if ((agg_buffer[$key] > 1)); then
        echo "aggregated: ${agg_buffer[$key]} alerts"
    else
        echo "single alert"
    fi
}

#-------------------------------------------------------------------------------
# TEST: Report Generation
#-------------------------------------------------------------------------------

test_report_generation() {
    echo ""
    echo "=== TEST SUITE: Report Generation ==="
    echo ""
    
    local report_file="${TEST_RESULTS_DIR}/test_report.html"
    
    # Generate test report
    generate_test_report "$report_file" 100 5 2
    
    # Verify report exists
    assert_true "[[ -f '$report_file' ]]" "Report file created"
    
    # Verify report content
    local content=$(cat "$report_file")
    assert_contains "$content" "<!DOCTYPE html>" "Report has HTML structure"
    assert_contains "$content" "100" "Report contains event count"
    assert_contains "$content" "Security" "Report contains security section"
}

generate_test_report() {
    local output="$1"
    local events="$2"
    local errors="$3"
    local incidents="$4"
    
    cat > "$output" << EOF
<!DOCTYPE html>
<html>
<head><title>Test Report</title></head>
<body>
<h1>Log Analyzer Test Report</h1>
<p>Events: $events</p>
<p>Errors: $errors</p>
<p>Security Incidents: $incidents</p>
</body>
</html>
EOF
}

#-------------------------------------------------------------------------------
# TEST: Integration Test
#-------------------------------------------------------------------------------

test_integration() {
    echo ""
    echo "=== TEST SUITE: Integration Test ==="
    echo ""
    
    local test_log="${TEST_RESULTS_DIR}/integration.log"
    local result_file="${TEST_RESULTS_DIR}/integration_results.txt"
    
    # Create test log with various patterns
    cat > "$test_log" << 'EOF'
Oct 15 14:30:00 server app[1234]: Normal operation
Oct 15 14:30:01 server app[1234]: User logged in successfully
Oct 15 14:30:02 server app[1234]: ERROR: Database connection failed
Oct 15 14:30:03 server sshd[5678]: Failed password for root from 192.168.1.100
Oct 15 14:30:04 server sshd[5678]: Failed password for root from 192.168.1.100
Oct 15 14:30:05 server sshd[5678]: Failed password for root from 192.168.1.100
Oct 15 14:30:06 server sshd[5678]: Failed password for root from 192.168.1.100
Oct 15 14:30:07 server sshd[5678]: Failed password for root from 192.168.1.100
Oct 15 14:30:08 server app[1234]: GET /api?id=1 UNION SELECT * FROM users--
Oct 15 14:30:09 server app[1234]: CRITICAL: System overload detected
EOF
    
    # Process log
    local errors=0
    local warnings=0
    local security=0
    local brute_force=0
    
    declare -A ip_failures
    
    while IFS= read -r line; do
        [[ "$line" =~ ERROR|CRITICAL ]] && ((errors++))
        [[ "$line" =~ WARNING ]] && ((warnings++))
        [[ "$line" =~ (UNION|SELECT.*FROM|INSERT|DELETE|DROP) ]] && ((security++))
        
        if [[ "$line" =~ [Ff]ailed.*password.*from\ ([0-9.]+) ]]; then
            local ip="${BASH_REMATCH[1]}"
            ((ip_failures[$ip]++))
            ((ip_failures[$ip] >= 5)) && brute_force=1
        fi
    done < "$test_log"
    
    # Verify results
    assert_equals "2" "$errors" "Error count detection (ERROR + CRITICAL)"
    assert_equals "1" "$security" "SQL injection detection"
    assert_equals "1" "$brute_force" "Brute force detection"
    assert_equals "5" "${ip_failures[192.168.1.100]}" "IP failure tracking"
}

#-------------------------------------------------------------------------------
# TEST: Performance Test
#-------------------------------------------------------------------------------

test_performance() {
    echo ""
    echo "=== TEST SUITE: Performance Test ==="
    echo ""
    
    local test_log="${TEST_RESULTS_DIR}/perf_test.log"
    local line_count=10000
    
    # Generate test log
    echo "Generating $line_count log lines..."
    for ((i=1; i<=line_count; i++)); do
        echo "Oct 15 14:30:00 server app[$i]: Log line number $i with some data payload for testing performance" >> "$test_log"
    done
    
    # Time processing
    local start_time=$(date +%s.%N)
    
    local count=0
    while IFS= read -r line; do
        # Simulate minimal processing
        [[ "$line" =~ ERROR ]] && ((count++)) || true
    done < "$test_log"
    
    local end_time=$(date +%s.%N)
    local duration=$(echo "$end_time - $start_time" | bc)
    local lines_per_sec=$(echo "scale=0; $line_count / $duration" | bc)
    
    echo "Processed $line_count lines in ${duration}s"
    echo "Throughput: $lines_per_sec lines/second"
    
    # Should process at least 1000 lines/second
    assert_true "(( lines_per_sec > 1000 ))" "Performance: >1000 lines/sec (actual: $lines_per_sec)"
}

#-------------------------------------------------------------------------------
# RUN ALL TESTS
#-------------------------------------------------------------------------------

run_all_tests() {
    trap cleanup_tests EXIT
    
    echo "============================================"
    echo "  LOG ANALYZER COMPREHENSIVE TEST SUITE"
    echo "============================================"
    echo ""
    echo "Starting tests at $(date)"
    echo ""
    
    setup_tests
    
    # Run test suites
    test_log_format_detection
    test_pattern_detection
    test_anomaly_detection
    test_alert_system
    test_report_generation
    test_integration
    test_performance
    
    # Summary
    echo ""
    echo "============================================"
    echo "  TEST SUMMARY"
    echo "============================================"
    echo ""
    echo "Tests Passed: $TESTS_PASSED"
    echo "Tests Failed: $TESTS_FAILED"
    echo "Total Tests:  $((TESTS_PASSED + TESTS_FAILED))"
    echo ""
    
    if ((TESTS_FAILED == 0)); then
        echo "🎉 ALL TESTS PASSED!"
        return 0
    else
        echo "❌ SOME TESTS FAILED"
        return 1
    fi
}

# Execute tests
run_all_tests
```

***

<a name="section-10"></a>

# 10. TIME-BASED LEARNING PLAN (2.5 Hours)

## 10.1 Structured Learning Schedule

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                    LOG ANALYSIS & ALERTING SYSTEM                            ║
║                         2.5 HOUR MASTERY PLAN                                ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║  HOUR 1: FOUNDATIONS (60 minutes)                                            ║
║  ────────────────────────────────────────────────────────────────────────    ║
║                                                                              ║
║  [0:00 - 0:15] Concept Foundation                                            ║
║    □ Understand log analysis pipeline (collect→parse→analyze→alert)          ║
║    □ Learn RFC 5424 severity levels (0-7)                                    ║
║    □ Understand why real-time processing matters                             ║
║    ✓ Checkpoint: Explain severity levels from memory                         ║
║                                                                              ║
║  [0:15 - 0:35] Log Format Deep Dive                                          ║
║    □ Master Syslog format (RFC 3164/5424)                                    ║
║    □ Parse Apache/Nginx combined log format                                  ║
║    □ Handle JSON logs with jq and bash                                       ║
║    □ Build universal format detector                                         ║
║    ✓ Checkpoint: Parse 3 different log formats correctly                     ║
║                                                                              ║
║  [0:35 - 1:00] Real-Time Parsing Engine                                      ║
║    □ Implement tail-based log following                                      ║
║    □ Create AWK high-performance parser                                      ║
║    □ Set up parallel log processing                                          ║
║    □ Handle log rotation gracefully                                          ║
║    ✓ Checkpoint: Monitor 2+ log files simultaneously                         ║
║                                                                              ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║  HOUR 2: DETECTION & ALERTING (60 minutes)                                    ║
║  ────────────────────────────────────────────────────────────────────────    ║
║                                                                              ║
║  [1:00 - 1:25] Anomaly Detection                                             ║
║    □ Implement statistical baseline calculation                              ║
║    □ Build z-score based spike detection                                     ║
║    □ Create sliding window analysis                                          ║
║    □ Detect attack patterns (SQLi, XSS, brute force)                         ║
║    ✓ Checkpoint: Detect 3 types of security incidents                        ║
║                                                                              ║
║  [1:25 - 1:50] Alert Escalation Framework                                    ║
║    □ Configure multi-channel alerting (email, Slack, PagerDuty)              ║
║    □ Implement cooldown and rate limiting                                    ║
║    □ Build alert aggregation system                                          ║
║    □ Create escalation logic (Level 1 → 2 → 3)                               ║
║    ✓ Checkpoint: Send test alert to 2+ channels                              ║
║                                                                              ║
║  [1:50 - 2:00] Break & Review                                                ║
║    □ Review key concepts                                                     ║
║    □ Test alerting manually                                                  ║
║                                                                              ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║  HOUR 3: PRODUCTION & TESTING (30 minutes)                                    ║
║  ────────────────────────────────────────────────────────────────────────    ║
║                                                                              ║
║  [2:00 - 2:15] Reporting & Integration                                       ║
║    □ Generate HTML executive reports                                         ║
║    □ Set up scheduled report generation                                      ║
║    □ Export Prometheus metrics                                               ║
║    □ Integrate with monitoring tools                                         ║
║    ✓ Checkpoint: Generate and view HTML report                               ║
║                                                                              ║
║  [2:15 - 2:30] Final Test & Validation                                       ║
║    □ Run complete self-test suite                                            ║
║    □ Execute attack simulation                                               ║
║    □ Verify all components work together                                     ║
║    □ Measure response time (<10 seconds target)                              ║
║    ✓ FINAL: System detects and responds to security incidents automatically  ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

## 10.2 Quick Reference Cheat Sheet

```bash
#!/bin/bash
#===============================================================================
# LOG ANALYZER CHEAT SHEET
# Quick reference for common operations
#===============================================================================

# ═══════════════════════════════════════════════════════════════════════════════
# LOG FORMATS
# ═══════════════════════════════════════════════════════════════════════════════

# SYSLOG (RFC 3164):
# <PRI>TIMESTAMP HOSTNAME TAG[PID]: MESSAGE
# Example: <34>Oct 15 14:30:00 server sshd[1234]: Connection accepted

# APACHE COMBINED:
# IP - USER [TIMESTAMP] "REQUEST" STATUS BYTES "REFERER" "USER-AGENT"
# Example: 192.168.1.1 - - [15/Oct/2023:14:30:00] "GET / HTTP/1.1" 200 1234 "-" "curl"

# JSON:
# {"timestamp":"ISO8601","level":"LEVEL","message":"MSG"}

# ═══════════════════════════════════════════════════════════════════════════════
# ESSENTIAL PATTERNS
# ═══════════════════════════════════════════════════════════════════════════════

# Security Detection Patterns:
SQLI_PATTERN='(UNION|SELECT.*FROM|INSERT|DELETE|DROP|--)'
XSS_PATTERN='(<script|javascript:|onerror=)'
TRAVERSAL_PATTERN='(\.\./|%2e%2e)'
BRUTE_FORCE_PATTERN='[Ff]ailed.*(password|login|auth)'

# System Health Patterns:
DISK_FULL='([Nn]o space left|[Dd]isk.*full)'
OOM_PATTERN='([Oo]ut of memory|OOM|[Kk]illed process)'
SERVICE_DOWN='([Cc]onnection refused|[Ss]ervice.*down)'

# ═══════════════════════════════════════════════════════════════════════════════
# COMMON COMMANDS
# ═══════════════════════════════════════════════════════════════════════════════

# Follow multiple logs:
tail -F /var/log/syslog /var/log/auth.log | grep -E "ERROR|FAIL"

# Parse with AWK:
awk '/ERROR/ {count++} END {print "Errors:", count}' /var/log/syslog

# Extract IPs from logs:
grep -oP '\d+\.\d+\.\d+\.\d+' /var/log/auth.log | sort | uniq -c | sort -rn

# Find failed logins:
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c

# JSON parsing with jq:
cat log.json | jq -r 'select(.level=="ERROR") | .message'

# ═══════════════════════════════════════════════════════════════════════════════
# QUICK FUNCTIONS
# ═══════════════════════════════════════════════════════════════════════════════

# Quick log analysis
analyze_log() {
    local log="$1"
    echo "=== Log Analysis: $log ==="
    echo "Total lines: $(wc -l < "$log")"
    echo "Errors: $(grep -c -i error "$log" 2>/dev/null || echo 0)"
    echo "Warnings: $(grep -c -i warning "$log" 2>/dev/null || echo 0)"
    echo "Failed auth: $(grep -c -i "failed.*password\|auth.*fail" "$log" 2>/dev/null || echo 0)"
}

# Send Slack alert
slack_alert() {
    local webhook="$1" severity="$2" message="$3"
    curl -s -X POST -H "Content-Type: application/json" \
        -d "{\"text\":\"[$severity] $message\"}" "$webhook"
}

# Block IP with iptables
block_ip() {
    local ip="$1"
    iptables -A INPUT -s "$ip" -j DROP
    echo "Blocked: $ip"
}

# Calculate error rate
error_rate() {
    local log="$1" window="${2:-60}"  # seconds
    local total=$(wc -l < "$log")
    local errors=$(grep -c "ERROR\|CRITICAL" "$log")
    echo "scale=2; $errors * 100 / $total" | bc
}

# ═══════════════════════════════════════════════════════════════════════════════
# SEVERITY LEVELS (RFC 5424)
# ═══════════════════════════════════════════════════════════════════════════════

# 0 - EMERGENCY   : System unusable
# 1 - ALERT       : Immediate action needed
# 2 - CRITICAL    : Critical conditions
# 3 - ERROR       : Error conditions
# 4 - WARNING     : Warning conditions
# 5 - NOTICE      : Normal but significant
# 6 - INFO        : Informational
# 7 - DEBUG       : Debug messages

# ═══════════════════════════════════════════════════════════════════════════════
# COMMON MISTAKES TO AVOID
# ═══════════════════════════════════════════════════════════════════════════════

# ❌ WRONG: Not handling log rotation
tail -f /var/log/syslog  # Stops working after rotation

# ✅ RIGHT: Use -F for rotation handling
tail -F /var/log/syslog  # Continues after rotation

# ❌ WRONG: Grep in tight loop (slow)
while read line; do echo "$line" | grep "ERROR"; done < log

# ✅ RIGHT: Let grep do the work
grep "ERROR" log

# ❌ WRONG: Not escaping regex special chars
grep "user[1]" log  # Matches user1, userA, etc.

# ✅ RIGHT: Escape or use fixed strings
grep -F "user[1]" log  # Matches literal "user[1]"

# ❌ WRONG: Alert on every error (alert fatigue)
if grep -q ERROR; then send_alert; fi

# ✅ RIGHT: Use cooldown and aggregation
if ! in_cooldown && error_rate > threshold; then send_alert; fi

# ═══════════════════════════════════════════════════════════════════════════════
# PERFORMANCE TIPS
# ═══════════════════════════════════════════════════════════════════════════════

# 1. Use AWK instead of bash loops for large files
# 2. Process logs in batches, not line-by-line
# 3. Use grep -F for fixed strings (faster)
# 4. Implement sampling for high-volume logs
# 5. Use named pipes for IPC instead of temp files
# 6. Parallelize processing for multiple log sources
```

## 10.3 Knowledge Verification Checklist

```bash
#!/bin/bash
# Self-assessment checklist

echo "╔════════════════════════════════════════════════════════════╗"
echo "║         LOG ANALYZER KNOWLEDGE VERIFICATION                ║"
echo "╚════════════════════════════════════════════════════════════╝"
echo ""

questions=(
    "Can you explain the 8 syslog severity levels?"
    "Can you parse Apache combined log format with regex?"
    "Can you detect SQL injection patterns?"
    "Can you implement z-score based anomaly detection?"
    "Can you set up multi-channel alerting (email + Slack)?"
    "Can you implement alert cooldown to prevent fatigue?"
    "Can you create escalation logic for unacknowledged alerts?"
    "Can you generate HTML executive reports?"
    "Can you export metrics to Prometheus format?"
    "Can you process 10,000 log lines in under 10 seconds?"
)

passed=0
for i in "${!questions[@]}"; do
    echo -n "$((i+1)). ${questions[$i]} [y/n]: "
    read -r answer
    [[ "$answer" == "y" ]] && ((passed++))
done

echo ""
echo "Score: $passed/${#questions[@]} ($((passed * 100 / ${#questions[@]}))%)"

if ((passed >= 8)); then
    echo "🎉 EXCELLENT! You've mastered log analysis!"
elif ((passed >= 6)); then
    echo "👍 GOOD! Review the areas you're unsure about."
else
    echo "📚 Keep studying! Review the weak areas."
fi
```

***

# 📚 FINAL SUMMARY

## Key Takeaways

| Component              | Purpose                 | Key Implementation                   |
| ---------------------- | ----------------------- | ------------------------------------ |
| **Log Parsing**        | Extract structured data | Regex patterns, AWK, jq              |
| **Format Detection**   | Handle multiple formats | Pattern matching cascade             |
| **Anomaly Detection**  | Find unusual patterns   | Z-score, baselines, thresholds       |
| **Security Detection** | Identify attacks        | SQLi, XSS, brute force patterns      |
| **Alerting**           | Notify on issues        | Multi-channel, cooldown, aggregation |
| **Escalation**         | Ensure response         | Timed escalation levels              |
| **Reporting**          | Executive visibility    | HTML reports, metrics export         |

## Production Checklist

```
□ Log rotation handled (tail -F)
□ Multiple formats supported
□ Security patterns comprehensive
□ False positive rate acceptable
□ Alert fatigue prevented (cooldown + aggregation)
□ Escalation configured
□ Reports scheduled
□ Monitoring integration working
□ Performance tested (>1000 lines/sec)
□ Recovery procedures documented
```

## Final Test Criteria

✅ **PASS if the system can:**

1. Monitor multiple log files simultaneously
2. Detect SQL injection, XSS, brute force attacks
3. Send alerts within 5 seconds of detection
4. Block malicious IPs automatically
5. Generate incident reports
6. Complete full detection-to-response in under 10 seconds

***

**Congratulations!** You've completed the Log Analysis & Alerting System module. This production-ready system provides comprehensive security monitoring and automated incident response capabilities essential for modern DevOps environments.
