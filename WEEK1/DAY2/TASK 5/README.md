# Performance Troubleshooting Project - Complete Mastery Guide

## Table of Contents
1. [Foundation Concepts](#foundation)
2. [CPU-Intensive Processes](#cpu-intensive)
3. [Memory-Consuming Processes](#memory-consuming)
4. [I/O-Heavy Processes](#io-heavy)
5. [Profiling Tools Deep Dive](#profiling-tools)
6. [Optimization Strategies](#optimization)
7. [Real-World Scenarios](#real-world)
8. [Cheat Sheets](#cheat-sheets)
9. [Hands-On Exercises](#exercises)

---

## 1. Foundation Concepts {#foundation}

### What is Performance Troubleshooting?
**Why**: Production systems fail due to resource exhaustion, bottlenecks, or inefficient code. Performance troubleshooting identifies and resolves these issues.

**Key Resource Types**:
- **CPU**: Processing power (calculations, logic)
- **Memory (RAM)**: Data storage for running processes
- **I/O**: Disk reads/writes, network traffic

### Prerequisites Installation

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install essential tools
sudo apt install -y \
  linux-tools-common \
  linux-tools-generic \
  linux-tools-$(uname -r) \
  strace \
  ltrace \
  gdb \
  sysstat \
  htop \
  iotop \
  stress \
  stress-ng \
  valgrind \
  time \
  bc
```

**Breakdown**:
- `linux-tools-*`: Contains `perf` utility
- `strace`: System call tracer
- `ltrace`: Library call tracer
- `gdb`: GNU debugger
- `sysstat`: System performance tools (sar, iostat, mpstat)
- `htop`: Interactive process viewer
- `iotop`: I/O monitoring
- `stress/stress-ng`: Load testing tools
- `valgrind`: Memory profiler
- `time`: Execution timing
- `bc`: Calculator for scripts

---

## 2. CPU-Intensive Processes {#cpu-intensive}

### Theory: What Makes a Process CPU-Intensive?
- Heavy computation (loops, math operations)
- No blocking I/O
- Keeps CPU cores busy

### Creating CPU-Intensive Scripts

#### Script 1: Infinite Loop Calculator

```bash
#!/bin/bash
# File: cpu_intensive_simple.sh
# Purpose: Max out CPU with calculations

while true; do
    echo "scale=5000; 4*a(1)" | bc -l > /dev/null
done
```

**Breakdown**:
- `while true`: Infinite loop
- `bc -l`: Calculator with math library
- `4*a(1)`: Calculate π (arctangent formula)
- `scale=5000`: 5000 decimal places
- `> /dev/null`: Discard output (focus on CPU)

**Run it**:
```bash
chmod +x cpu_intensive_simple.sh
./cpu_intensive_simple.sh &
```

**Expected Output**: None (runs in background)

---

#### Script 2: Multi-Core CPU Stress

```bash
#!/bin/bash
# File: cpu_multicore_stress.sh
# Purpose: Stress all CPU cores

NUM_CORES=$(nproc)
echo "Stressing $NUM_CORES cores..."

for i in $(seq 1 $NUM_CORES); do
    sha256sum /dev/zero &
done

echo "PIDs: $(jobs -p)"
wait
```

**Breakdown**:
- `nproc`: Get number of CPU cores
- `seq 1 $NUM_CORES`: Generate sequence (1 to core count)
- `sha256sum /dev/zero`: Hash infinite zeros (CPU-bound)
- `&`: Run each in background
- `jobs -p`: Show process IDs
- `wait`: Keep script alive

**Usage**:
```bash
chmod +x cpu_multicore_stress.sh
./cpu_multicore_stress.sh

# Stop all:
pkill -P $$  # Kills child processes
```

---

#### Script 3: Controllable CPU Load

```bash
#!/bin/bash
# File: cpu_variable_load.sh
# Purpose: Generate specific CPU load percentage

if [ -z "$1" ]; then
    echo "Usage: $0 <cpu_percentage> [duration_seconds]"
    exit 1
fi

TARGET_LOAD=$1
DURATION=${2:-60}  # Default 60 seconds

echo "Generating ${TARGET_LOAD}% CPU load for ${DURATION}s"

# Calculate work/sleep ratio
WORK_TIME=$(echo "scale=6; $TARGET_LOAD / 100" | bc)
SLEEP_TIME=$(echo "scale=6; 1 - $WORK_TIME" | bc)

END_TIME=$(($(date +%s) + DURATION))

while [ $(date +%s) -lt $END_TIME ]; do
    # CPU work period
    timeout ${WORK_TIME}s yes > /dev/null 2>&1
    # Sleep period
    sleep ${SLEEP_TIME}s
done

echo "Load generation complete"
```

**Usage**:
```bash
chmod +x cpu_variable_load.sh
./cpu_variable_load.sh 50 120  # 50% CPU for 120 seconds
```

**Monitoring**:
```bash
# Terminal 1: Run script
./cpu_variable_load.sh 75 300

# Terminal 2: Monitor
top -p $(pgrep -f cpu_variable_load)
# Press '1' to see individual cores
```

---

### Profiling CPU Usage

#### Using `perf` (Performance Counter Tool)

**Basic CPU Profiling**:
```bash
# Record CPU performance for 30 seconds
perf record -F 99 -a -g -- sleep 30

# -F 99: Sample at 99 Hz (99 times/second)
# -a: All CPUs
# -g: Call graph (stack traces)
# sleep 30: Duration

# Analyze results
perf report
```

**Expected Output**:
```
Samples: 2K of event 'cycles:ppp'
  Children      Self  Command   Shared Object      Symbol
+   45.23%     0.00%  sha256sum [kernel.kallsyms]  [k] entry_SYSCALL_64_after_hwframe
+   32.14%    31.87%  sha256sum sha256sum          [.] sha256_process_block
```

**Navigation in perf report**:
- `↑/↓`: Move cursor
- `Enter`: Expand call graph
- `q`: Quit
- `/`: Search

---

**Profiling Specific Process**:
```bash
# Start CPU-intensive process
./cpu_intensive_simple.sh &
PID=$!

# Profile for 10 seconds
perf record -F 999 -p $PID -g -- sleep 10

# View report
perf report --stdio  # Text output

# Or interactive
perf report
```

---

**Flame Graph Generation**:
```bash
# Install FlameGraph tools
git clone https://github.com/brendangregg/FlameGraph.git

# Record
perf record -F 99 -a -g -- sleep 30

# Convert to flame graph
perf script | ./FlameGraph/stackcollapse-perf.pl | ./FlameGraph/flamegraph.pl > flamegraph.svg

# View in browser
firefox flamegraph.svg
```

---

#### Using `top` and `htop`

**top shortcuts**:
```bash
top

# Keyboard shortcuts:
# P - Sort by CPU%
# M - Sort by Memory%
# 1 - Toggle individual CPUs
# f - Select fields
# k - Kill process
# r - Renice (change priority)
# q - Quit
```

**htop advantages**:
```bash
htop

# F2 - Setup (customize)
# F3 - Search
# F4 - Filter
# F5 - Tree view
# F6 - Sort by column
# F9 - Kill process
# Space - Tag process
# U - Show user processes
```

---

## 3. Memory-Consuming Processes {#memory-consuming}

### Theory: Memory Types
- **RSS (Resident Set Size)**: Physical RAM used
- **VSZ (Virtual Size)**: Total virtual memory
- **Shared Memory**: Shared between processes
- **Memory Leak**: Gradual memory allocation without release

---

### Creating Memory-Intensive Scripts

#### Script 1: Simple Memory Allocator

```bash
#!/bin/bash
# File: memory_simple.sh
# Purpose: Allocate fixed memory amount

if [ -z "$1" ]; then
    echo "Usage: $0 <megabytes>"
    exit 1
fi

MB=$1
echo "Allocating ${MB}MB of memory..."

# Create large string in memory
BYTES=$((MB * 1024 * 1024))
DATA=$(head -c $BYTES /dev/zero | tr '\0' 'A')

echo "Memory allocated. Press Ctrl+C to release."
while true; do
    sleep 1
done
```

**Usage**:
```bash
chmod +x memory_simple.sh
./memory_simple.sh 500  # Allocate 500MB

# Monitor in another terminal:
ps aux | grep memory_simple
# Or
pmap -x $(pgrep -f memory_simple.sh)
```

---

#### Script 2: Memory Leak Simulator

```bash
#!/bin/bash
# File: memory_leak.sh
# Purpose: Simulate gradual memory leak

echo "Simulating memory leak (10MB every 5 seconds)"
echo "PID: $$"

COUNTER=0
declare -a LEAK_ARRAY

while true; do
    # Allocate 10MB
    LEAK_ARRAY[$COUNTER]=$(head -c 10485760 /dev/zero | tr '\0' 'X')
    
    COUNTER=$((COUNTER + 1))
    TOTAL_MB=$((COUNTER * 10))
    
    echo "$(date '+%T'): Leaked ${TOTAL_MB}MB (PID: $$)"
    
    sleep 5
done
```

**Usage**:
```bash
./memory_leak.sh &
PID=$!

# Monitor memory growth
watch -n 1 "ps -o pid,vsz,rss,cmd -p $PID"

# Kill when done
kill $PID
```

---

#### Script 3: Controlled Memory Pressure

```bash
#!/bin/bash
# File: memory_pressure.sh
# Purpose: Create memory pressure with cleanup

TARGET_MB=${1:-1024}  # Default 1GB
CHUNK_MB=100
DURATION=${2:-300}    # Default 5 minutes

echo "Creating ${TARGET_MB}MB memory pressure for ${DURATION}s"

declare -a MEM_CHUNKS
CHUNKS=$((TARGET_MB / CHUNK_MB))

# Allocate memory
for i in $(seq 1 $CHUNKS); do
    MEM_CHUNKS[$i]=$(head -c $((CHUNK_MB * 1024 * 1024)) /dev/zero | tr '\0' 'M')
    echo "Allocated chunk $i/${CHUNKS} (${CHUNK_MB}MB)"
    sleep 0.1
done

echo "Total allocated: ${TARGET_MB}MB. Holding for ${DURATION}s..."
sleep $DURATION

# Cleanup
unset MEM_CHUNKS
echo "Memory released"
```

---

### Profiling Memory Usage

#### Using `valgrind` (Memory Leak Detection)

**Create test program**:
```c
// File: memory_test.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    printf("Starting memory leak test...\n");
    
    for (int i = 0; i < 10; i++) {
        // Allocate 1MB without freeing (leak!)
        char *leak = malloc(1024 * 1024);
        printf("Leaked 1MB (iteration %d)\n", i+1);
        sleep(1);
    }
    
    printf("Done\n");
    return 0;
}
```

**Compile and analyze**:
```bash
# Compile with debug symbols
gcc -g -o memory_test memory_test.c

# Run with valgrind
valgrind --leak-check=full \
         --show-leak-kinds=all \
         --track-origins=yes \
         --verbose \
         --log-file=valgrind-out.txt \
         ./memory_test
```

**Expected Output**:
```
LEAK SUMMARY:
   definitely lost: 10,485,760 bytes in 10 blocks
   indirectly lost: 0 bytes in 0 blocks
```

---

#### Using `pmap` (Process Memory Map)

```bash
# Start memory-consuming process
./memory_simple.sh 1000 &
PID=$!

# View memory map
pmap -x $PID

# Extended view
pmap -X $PID

# Summary only
pmap -d $PID
```

**Output explanation**:
```
Address           Kbytes     RSS   Dirty Mode  Mapping
0000000000400000       4       4       0 r-x-- memory_simple
00007f8a2c000000 1024000 1024000 1024000 rw---   [ anon ]
                 -------  ------  ------
                 1048576 1048576 1048576 KB
```

---

#### Using `/proc` filesystem

```bash
#!/bin/bash
# File: monitor_memory.sh
# Purpose: Real-time memory monitoring

if [ -z "$1" ]; then
    echo "Usage: $0 <PID>"
    exit 1
fi

PID=$1

while true; do
    if [ ! -d "/proc/$PID" ]; then
        echo "Process $PID no longer exists"
        exit 1
    fi
    
    # Read memory stats
    read RSS < /proc/$PID/statm
    RSS_MB=$((RSS * 4 / 1024))  # Convert pages to MB
    
    # Get detailed memory info
    VmRSS=$(grep VmRSS /proc/$PID/status | awk '{print $2}')
    VmSize=$(grep VmSize /proc/$PID/status | awk '{print $2}')
    
    clear
    echo "=== Memory Monitor for PID $PID ==="
    echo "Time: $(date '+%T')"
    echo "VmSize: $((VmSize / 1024)) MB"
    echo "VmRSS:  $((VmRSS / 1024)) MB"
    echo "================================="
    
    sleep 2
done
```

**Usage**:
```bash
./memory_leak.sh &
PID=$!

./monitor_memory.sh $PID
```

---

## 4. I/O-Heavy Processes {#io-heavy}

### Theory: I/O Types
- **Sequential I/O**: Reading/writing large continuous blocks (efficient)
- **Random I/O**: Scattered reads/writes (slow)
- **Buffered vs Direct I/O**: Cache vs bypass cache
- **Sync vs Async**: Blocking vs non-blocking

---

### Creating I/O-Intensive Scripts

#### Script 1: Sequential Write Stress

```bash
#!/bin/bash
# File: io_sequential_write.sh
# Purpose: Generate heavy sequential write load

FILE_SIZE_GB=${1:-5}
OUTPUT_FILE=${2:-/tmp/io_test_$$}

echo "Writing ${FILE_SIZE_GB}GB sequentially to ${OUTPUT_FILE}"

# Method 1: Using dd
time dd if=/dev/zero of=$OUTPUT_FILE bs=1M count=$((FILE_SIZE_GB * 1024)) conv=fdatasync

# conv=fdatasync: Force physical write (no cache)

echo "Write complete. Stats:"
ls -lh $OUTPUT_FILE

# Cleanup
rm -f $OUTPUT_FILE
```

**Breakdown**:
- `if=/dev/zero`: Input source (infinite zeros)
- `of=`: Output file
- `bs=1M`: Block size (1 megabyte)
- `count=`: Number of blocks
- `conv=fdatasync`: Sync data to disk (real I/O)

**Expected Output**:
```
5242880000 bytes (5.2 GB, 4.9 GiB) copied, 15.3421 s, 342 MB/s

real    0m15.342s
user    0m0.043s
sys     0m3.212s
```

---

#### Script 2: Random I/O Generator

```bash
#!/bin/bash
# File: io_random.sh
# Purpose: Generate random I/O patterns

FILE_SIZE_MB=${1:-1024}
TEST_FILE="/tmp/random_io_test_$$"
DURATION=${2:-60}

echo "Creating ${FILE_SIZE_MB}MB test file..."
dd if=/dev/urandom of=$TEST_FILE bs=1M count=$FILE_SIZE_MB 2>/dev/null

echo "Starting random I/O for ${DURATION}s..."

END_TIME=$(($(date +%s) + DURATION))
READS=0
WRITES=0

while [ $(date +%s) -lt $END_TIME ]; do
    # Random read
    OFFSET=$((RANDOM % FILE_SIZE_MB))
    dd if=$TEST_FILE of=/dev/null bs=1M skip=$OFFSET count=1 2>/dev/null
    READS=$((READS + 1))
    
    # Random write
    OFFSET=$((RANDOM % FILE_SIZE_MB))
    dd if=/dev/zero of=$TEST_FILE bs=1M seek=$OFFSET count=1 conv=notrunc 2>/dev/null
    WRITES=$((WRITES + 1))
    
    # Progress
    if [ $((READS % 10)) -eq 0 ]; then
        echo "Reads: $READS, Writes: $WRITES"
    fi
done

echo "Test complete. Total - Reads: $READS, Writes: $WRITES"
rm -f $TEST_FILE
```

**Usage**:
```bash
# Terminal 1: Run I/O test
./io_random.sh 2048 120

# Terminal 2: Monitor I/O
sudo iotop -o  # Only show active I/O
# Or
iostat -x 2    # Extended stats every 2 seconds
```

---

#### Script 3: Mixed I/O Workload

```bash
#!/bin/bash
# File: io_mixed_workload.sh
# Purpose: Simulate realistic mixed I/O patterns

TEST_DIR="/tmp/io_workload_$$"
NUM_FILES=100
FILE_SIZE_KB=1024
DURATION=${1:-300}

mkdir -p $TEST_DIR
echo "Created test directory: $TEST_DIR"

# Create test files
echo "Creating $NUM_FILES test files..."
for i in $(seq 1 $NUM_FILES); do
    dd if=/dev/urandom of=$TEST_DIR/file_$i bs=1K count=$FILE_SIZE_KB 2>/dev/null
done

echo "Starting mixed I/O workload for ${DURATION}s..."

START_TIME=$(date +%s)
OPS=0

while [ $(($(date +%s) - START_TIME)) -lt $DURATION ]; do
    OPERATION=$((RANDOM % 4))
    FILE_ID=$((RANDOM % NUM_FILES + 1))
    
    case $OPERATION in
        0)  # Sequential read
            cat $TEST_DIR/file_$FILE_ID > /dev/null
            ;;
        1)  # Random read
            dd if=$TEST_DIR/file_$FILE_ID of=/dev/null bs=4K skip=$((RANDOM % 256)) count=1 2>/dev/null
            ;;
        2)  # Sequential write
            dd if=/dev/zero of=$TEST_DIR/file_$FILE_ID bs=1K count=$FILE_SIZE_KB conv=notrunc 2>/dev/null
            ;;
        3)  # Metadata operation
            touch $TEST_DIR/file_$FILE_ID
            ;;
    esac
    
    OPS=$((OPS + 1))
    
    if [ $((OPS % 100)) -eq 0 ]; then
        echo "Operations completed: $OPS"
    fi
done

TOTAL_TIME=$(($(date +%s) - START_TIME))
IOPS=$((OPS / TOTAL_TIME))

echo "Workload complete:"
echo "  Total operations: $OPS"
echo "  Duration: ${TOTAL_TIME}s"
echo "  IOPS: $IOPS"

# Cleanup
rm -rf $TEST_DIR
```

---

### Profiling I/O Performance

#### Using `iostat` (I/O Statistics)

```bash
# Install if needed
sudo apt install sysstat

# Basic usage
iostat -x 2 10
# -x: Extended statistics
# 2: Update every 2 seconds
# 10: 10 iterations

# Monitor specific device
iostat -x /dev/sda 1
```

**Output interpretation**:
```
Device   r/s   w/s   rkB/s   wkB/s  %util
sda     15.3  42.1   512.4  2048.7   75.2
```
- `r/s`, `w/s`: Reads/writes per second
- `rkB/s`, `wkB/s`: KB read/written per second
- `%util`: Device saturation (>80% = bottleneck)

---

#### Using `iotop` (I/O Top)

```bash
# Real-time I/O monitoring
sudo iotop

# Keyboard shortcuts:
# o - Only show processes with I/O
# a - Accumulated I/O (instead of bandwidth)
# p - Processes (instead of threads)
# Left/Right - Change sort column
# r - Reverse sort
# q - Quit

# Non-interactive mode (for logging)
sudo iotop -b -n 5 > iotop_log.txt
# -b: Batch mode
# -n 5: 5 iterations
```

---

#### Using `strace` (System Call Tracing for I/O)

```bash
# Start I/O process
./io_sequential_write.sh 1 &
PID=$!

# Trace I/O system calls
strace -p $PID -e trace=read,write,open,close,fsync -f -tt -T

# -p: Attach to PID
# -e trace=: Filter specific calls
# -f: Follow forks
# -tt: Timestamp (microseconds)
# -T: Show time spent in each call
```

**Expected Output**:
```
15:32:14.123456 open("/tmp/io_test", O_WRONLY|O_CREAT, 0666) = 3 <0.000234>
15:32:14.125678 write(3, "\0\0\0\0...", 1048576) = 1048576 <0.015432>
15:32:14.141234 fdatasync(3) = 0 <0.234567>
```

---

#### Using `blktrace` (Block Layer Tracing)

```bash
# Install
sudo apt install blktrace

# Start tracing (requires root)
sudo blktrace -d /dev/sda -o trace_output &

# Run I/O workload
./io_random.sh 500 60

# Stop tracing
sudo pkill blktrace

# Parse results
blkparse trace_output.blktrace.0 > parsed_trace.txt

# Generate report
btt -i trace_output.blktrace.0
```

---

## 5. Profiling Tools Deep Dive {#profiling-tools}

### `perf` - Performance Events

#### Installation and Setup

```bash
# Install
sudo apt install linux-tools-common linux-tools-$(uname -r)

# Enable kernel symbols
echo 0 | sudo tee /proc/sys/kernel/kptr_restrict

# Allow non-root users (optional, security implication)
echo -1 | sudo tee /proc/sys/kernel/perf_event_paranoid
```

---

#### Common perf Commands

**1. List available events**:
```bash
perf list
# Shows: Hardware events, software events, cache events, etc.

perf list | grep -i cache  # Cache-related events
```

**2. Count events**:
```bash
# Run program and count events
perf stat ./your_program

# Output:
#  Performance counter stats for './your_program':
#       1,245.67 msec task-clock         # 0.998 CPUs utilized
#          12,456 context-switches       # 0.010 M/sec
#             234 cpu-migrations         # 0.188 K/sec
#         123,456 page-faults            # 0.099 M/sec
#   4,567,890,123 cycles                 # 3.667 GHz
#   2,345,678,901 instructions           # 0.51  insn per cycle
```

**3. Record and report**:
```bash
# Record with call graphs
perf record -g ./your_program

# View report
perf report

# Annotate source code (requires debug symbols)
perf annotate
```

**4. Real-time monitoring**:
```bash
# Live CPU monitoring
perf top

# Filter by function
perf top -e cycles --call-graph dwarf
```

---

#### Advanced perf Techniques

**CPU cache analysis**:
```bash
perf stat -e cache-references,cache-misses ./your_program

# Output shows cache hit ratio
```

**Branch prediction analysis**:
```bash
perf stat -e branches,branch-misses ./your_program
```

**Off-CPU analysis** (waiting time):
```bash
perf record -e sched:sched_switch -g -- sleep 10
perf script > out.stacks
```

---

### `strace` - System Call Tracer

#### Basic Usage

```bash
# Trace all system calls
strace ./your_program

# Save to file
strace -o trace.log ./your_program

# Trace running process
strace -p 1234

# Trace with timestamps
strace -tt -T ./your_program
# -tt: Microsecond timestamps
# -T: Time spent in each call
```

---

#### Filtering and Analysis

**Filter specific calls**:
```bash
# Only file operations
strace -e trace=file ./your_program

# Only network calls
strace -e trace=network ./your_program

# Multiple categories
strace -e trace=open,read,write,close ./your_program
```

**Count system calls**:
```bash
# Summary statistics
strace -c ./your_program

# Output:
# % time     seconds  usecs/call     calls    errors syscall
# ------ ----------- ----------- --------- --------- --------
#  45.23    0.123456          12     10234           read
#  32.14    0.087654          45      1945           write
```

**Trace child processes**:
```bash
strace -f ./parent_program
# -f: Follow forks
```

---

#### Debugging Scenarios

**1. Find missing files**:
```bash
strace -e trace=open,openat ./your_program 2>&1 | grep ENOENT
# ENOENT = "No such file or directory"
```

**2. Debug slow startup**:
```bash
strace -tt -T -o startup.log ./slow_program
grep -E "<[0-9]+\.[0-9]{6}>" startup.log | sort -t'<' -k2 -rn | head -20
# Shows slowest 20 calls
```

**3. Network debugging**:
```bash
strace -e trace=network -s 1024 curl http://example.com
# -s 1024: Show 1024 bytes of string arguments
```

---

### `ltrace` - Library Call Tracer

#### Basic Usage

```bash
# Trace library calls
ltrace ./your_program

# Trace with timestamps
ltrace -tt -T ./your_program

# Save output
ltrace -o library.log ./your_program
```

---

#### Filtering Library Calls

```bash
# Only show specific functions
ltrace -e malloc,free ./your_program

# Exclude functions
ltrace -e '*'-e '!malloc' ./your_program

# Show all calls from specific library
ltrace -l /lib/x86_64-linux-gnu/libc.so.6 ./your_program
```

---

#### Memory Debugging

**Track malloc/free**:
```bash
#!/bin/bash
# File: trace_memory_leaks.sh

PROGRAM=$1

ltrace -e 'malloc+free+realloc+calloc' -o mem_trace.log $PROGRAM

# Parse log
echo "Memory allocation summary:"
grep -E 'malloc|calloc|realloc' mem_trace.log | wc -l
echo "Free calls:"
grep 'free' mem_trace.log | wc -l
```

---

### `gdb` - GNU Debugger

#### Basic Debugging Workflow

```bash
# Compile with debug symbols
gcc -g -o program program.c

# Start debugger
gdb ./program

# GDB commands:
(gdb) run                    # Run program
(gdb) run arg1 arg2          # Run with arguments
(gdb) break main             # Breakpoint at main
(gdb) break file.c:123       # Breakpoint at line 123
(gdb) continue               # Continue execution
(gdb) next                   # Step over (next line)
(gdb) step                   # Step into (enter function)
(gdb) print variable_name    # Print variable value
(gdb) backtrace              # Show call stack
(gdb) info locals            # Show local variables
(gdb) quit                   # Exit
```

---

#### Advanced GDB Techniques

**Attach to running process**:
```bash
# Find PID
ps aux | grep your_program

# Attach
sudo gdb -p 1234

# In GDB:
(gdb) backtrace          # See what it's doing
(gdb) thread apply all bt # All threads' stacks
(gdb) detach             # Detach without killing
```

**Conditional breakpoints**:
```bash
(gdb) break function_name if variable > 100
(gdb) condition 1 x == 5  # Modify existing breakpoint #1
```

**Watchpoints** (break when variable changes):
```bash
(gdb) watch variable_name
(gdb) rwatch variable_name  # Read watchpoint
(gdb) awatch variable_name  # Read/write watchpoint
```

---

#### Performance Debugging with GDB

**Profiling with sampling**:
```bash
#!/bin/bash
# File: gdb_profiling.sh
# Purpose: Poor man's profiler

PID=$1
SAMPLES=100
INTERVAL=0.01

for i in $(seq 1 $SAMPLES); do
    gdb -batch -p $PID -ex "thread apply all bt" 2>/dev/null
    sleep $INTERVAL
done | grep "^#" | sort | uniq -c | sort -rn | head -20
```

**Memory inspection**:
```bash
(gdb) info proc mappings    # Memory map
(gdb) info proc status       # Process status
(gdb) dump memory file.bin start_addr end_addr  # Dump memory
```

---

## 6. Optimization Strategies {#optimization}

### CPU Optimization

#### Strategy 1: Reduce Algorithmic Complexity

**Before** (O(n²)):
```c
// File: inefficient.c
#include <stdio.h>

int main() {
    int n = 10000;
    long long sum = 0;
    
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            sum += i * j;
        }
    }
    
    printf("Sum: %lld\n", sum);
    return 0;
}
```

**After** (O(n)):
```c
// File: efficient.c
#include <stdio.h>

int main() {
    int n = 10000;
    long long sum = 0;
    long long sum_i = 0, sum_j = 0;
    
    for (int i = 0; i < n; i++) sum_i += i;
    for (int j = 0; j < n; j++) sum_j += j;
    
    sum = sum_i * sum_j;
    
    printf("Sum: %lld\n", sum);
    return 0;
}
```

**Benchmark**:
```bash
gcc -O2 -o inefficient inefficient.c
gcc -O2 -o efficient efficient.c

time ./inefficient
time ./efficient

# Profile both
perf stat ./inefficient
perf stat ./efficient
```

---

#### Strategy 2: Compiler Optimization Flags

```bash
# No optimization
gcc -O0 -o prog_O0 program.c

# Basic optimization
gcc -O1 -o prog_O1 program.c

# Recommended optimization
gcc -O2 -o prog_O2 program.c

# Aggressive optimization
gcc -O3 -o prog_O3 program.c

# Size optimization
gcc -Os -o prog_Os program.c

# Architecture-specific
gcc -O3 -march=native -o prog_native program.c

# Compare performance
for opt in O0 O1 O2 O3 Os native; do
    echo "=== $opt ==="
    time ./prog_$opt
done
```

---

#### Strategy 3: Process Priority and CPU Affinity

**Nice values** (priority):
```bash
# Lower priority (nice value 19 = lowest)
nice -n 19 ./background_job.sh

# Higher priority (requires root)
sudo nice -n -20 ./critical_job.sh

# Change priority of running process
renice -n 10 -p 1234
```

**CPU affinity** (bind to specific cores):
```bash
# Run on CPU 0 only
taskset -c 0 ./program

# Run on CPUs 0,1,2,3
taskset -c 0-3 ./program

# Check current affinity
taskset -p 1234

# Set affinity for running process
taskset -cp 0,1 1234
```

---

### Memory Optimization

#### Strategy 1: Memory Pool Pattern

**Before** (fragmentation):
```c
// Repeated malloc/free causes fragmentation
for (int i = 0; i < 1000000; i++) {
    char *data = malloc(128);
    // Use data
    free(data);
}
```

**After** (pool):
```c
#include <stdlib.h>
#include <string.h>

#define POOL_SIZE 1000
#define OBJECT_SIZE 128

typedef struct {
    char data[OBJECT_SIZE];
    int in_use;
} PoolObject;

PoolObject pool[POOL_SIZE];

void* pool_alloc() {
    for (int i = 0; i < POOL_SIZE; i++) {
        if (!pool[i].in_use) {
            pool[i].in_use = 1;
            return pool[i].data;
        }
    }
    return NULL;
}

void pool_free(void* ptr) {
    PoolObject* obj = (PoolObject*)((char*)ptr - offsetof(PoolObject, data));
    obj->in_use = 0;
}
```

---

#### Strategy 2: Memory-Mapped Files

**Before** (read entire file):
```bash
#!/bin/bash
# Loads entire file into memory
CONTENT=$(cat huge_file.txt)
echo "$CONTENT" | grep "pattern"
```

**After** (streaming):
```bash
#!/bin/bash
# Process line by line
grep "pattern" huge_file.txt
```

**C example with mmap**:
```c
#include <sys/mman.h>
#include <fcntl.h>
#include <unistd.h>

int fd = open("largefile.dat", O_RDONLY);
struct stat sb;
fstat(fd, &sb);

char *data = mmap(NULL, sb.st_size, PROT_READ, MAP_PRIVATE, fd, 0);
// Access data directly without loading into heap
munmap(data, sb.st_size);
close(fd);
```

---

#### Strategy 3: Reduce Memory Footprint

**Script to monitor memory**:
```bash
#!/bin/bash
# File: optimize_memory.sh

echo "Memory usage before optimization:"
free -h

# Clear page cache (requires root)
sync && sudo sh -c 'echo 3 > /proc/sys/vm/drop_caches'

echo "Memory usage after cache clear:"
free -h

# Identify memory hogs
ps aux --sort=-%mem | head -10
```

---

### I/O Optimization

#### Strategy 1: Buffering

**Before** (unbuffered):
```bash
#!/bin/bash
for i in {1..1000}; do
    echo "Line $i" >> output.txt
done
```

**After** (buffered):
```bash
#!/bin/bash
{
    for i in {1..1000}; do
        echo "Line $i"
    done
} >> output.txt
```

**Benchmark**:
```bash
time bash unbuffered.sh
time bash buffered.sh
```

---

#### Strategy 2: Parallel I/O

```bash
#!/bin/bash
# File: parallel_io.sh
# Purpose: Process files in parallel

FILES=(file1.txt file2.txt file3.txt file4.txt)
MAX_PARALLEL=4

process_file() {
    local file=$1
    echo "Processing $file..."
    # Simulate processing
    grep -E "pattern" "$file" > "${file}.result"
}

# Export function for parallel use
export -f process_file

# Use GNU parallel
parallel -j $MAX_PARALLEL process_file ::: "${FILES[@]}"

# Or without parallel tool:
for file in "${FILES[@]}"; do
    process_file "$file" &
    # Limit concurrent jobs
    while [ $(jobs -r | wc -l) -ge $MAX_PARALLEL ]; do
        wait -n
    done
done
wait
```

---

#### Strategy 3: I/O Scheduler Tuning

```bash
# Check current scheduler
cat /sys/block/sda/queue/scheduler
# Output: [mq-deadline] kyber none

# Change scheduler (requires root)
echo kyber | sudo tee /sys/block/sda/queue/scheduler

# Scheduler options:
# - mq-deadline: Good for general use, reduces latency
# - kyber: Better for fast SSDs
# - none: No scheduler (good for NVMe)
# - bfq: Best for interactive/desktop

# For databases/heavy random I/O:
echo kyber | sudo tee /sys/block/sda/queue/scheduler

# For sequential workloads:
echo none | sudo tee /sys/block/nvme0n1/queue/scheduler
```

---

## 7. Real-World Production Scenarios {#real-world}

### Scenario 1: High CPU Usage Investigation

**Problem**: Application server CPU at 100%

**Investigation Script**:
```bash
#!/bin/bash
# File: investigate_cpu.sh

OUTPUT_DIR="cpu_investigation_$(date +%Y%m%d_%H%M%S)"
mkdir -p $OUTPUT_DIR

echo "=== CPU Investigation Started ==="

# 1. Identify top CPU consumers
echo "Top CPU processes:" | tee $OUTPUT_DIR/top_processes.txt
ps aux --sort=-%cpu | head -20 >> $OUTPUT_DIR/top_processes.txt

# 2. Get detailed process info
TOP_PID=$(ps aux --sort=-%cpu | awk 'NR==2 {print $2}')
echo "Investigating PID: $TOP_PID"

# 3. Process details
ps -p $TOP_PID -o pid,ppid,cmd,%cpu,%mem,etime > $OUTPUT_DIR/process_info.txt

# 4. Thread analysis
ps -T -p $TOP_PID > $OUTPUT_DIR/threads.txt

# 5. System call tracing
timeout 10s strace -c -p $TOP_PID 2> $OUTPUT_DIR/strace_summary.txt

# 6. CPU profiling
sudo perf record -F 99 -p $TOP_PID -g -- sleep 30
sudo perf report --stdio > $OUTPUT_DIR/perf_report.txt

# 7. Stack traces (if GDB available)
sudo gdb -batch -p $TOP_PID -ex "thread apply all bt" > $OUTPUT_DIR/stacktrace.txt 2>&1

# 8. File descriptors
ls -la /proc/$TOP_PID/fd > $OUTPUT_DIR/file_descriptors.txt

# 9. System load
uptime > $OUTPUT_DIR/system_load.txt
mpstat -P ALL 5 5 > $OUTPUT_DIR/cpu_stats.txt

echo "=== Investigation Complete ==="
echo "Results saved to: $OUTPUT_DIR"
```

**Analysis**:
```bash
./investigate_cpu.sh

# Review results
cd cpu_investigation_*

# Check what's consuming CPU
cat strace_summary.txt  # Look for hot system calls

# Check perf report
cat perf_report.txt     # Find hot functions

# If it's looping:
cat stacktrace.txt      # See where it's stuck
```

---

### Scenario 2: Memory Leak Detection

**Problem**: Application memory grows over time

**Monitoring Script**:
```bash
#!/bin/bash
# File: detect_memory_leak.sh

if [ -z "$1" ]; then
    echo "Usage: $0 <process_name>"
    exit 1
fi

PROCESS_NAME=$1
LOG_FILE="memory_leak_${PROCESS_NAME}_$(date +%Y%m%d_%H%M%S).log"
INTERVAL=60  # Check every 60 seconds
THRESHOLD_MB=100  # Alert if growth > 100MB

echo "Monitoring $PROCESS_NAME for memory leaks..."
echo "Log file: $LOG_FILE"

PREV_RSS=0

while true; do
    PID=$(pgrep -o $PROCESS_NAME)
    
    if [ -z "$PID" ]; then
        echo "$(date): Process not found" | tee -a $LOG_FILE
        sleep $INTERVAL
        continue
    fi
    
    # Get memory usage
    RSS=$(ps -p $PID -o rss= | awk '{print $1}')
    RSS_MB=$((RSS / 1024))
    
    TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
    
    if [ $PREV_RSS -gt 0 ]; then
        GROWTH_MB=$(( (RSS - PREV_RSS) / 1024 ))
        
        echo "$TIMESTAMP | PID: $PID | RSS: ${RSS_MB}MB | Growth: ${GROWTH_MB}MB" | tee -a $LOG_FILE
        
        if [ $GROWTH_MB -gt $THRESHOLD_MB ]; then
            echo "!!! ALERT: Memory grew by ${GROWTH_MB}MB !!!" | tee -a $LOG_FILE
            
            # Capture detailed snapshot
            pmap -x $PID > "pmap_${PID}_${TIMESTAMP//[: -]/}.txt"
            cat /proc/$PID/smaps > "smaps_${PID}_${TIMESTAMP//[: -]/}.txt"
        fi
    else
        echo "$TIMESTAMP | PID: $PID | RSS: ${RSS_MB}MB (baseline)" | tee -a $LOG_FILE
    fi
    
    PREV_RSS=$RSS
    sleep $INTERVAL
done
```

**Usage**:
```bash
# Start monitoring
./detect_memory_leak.sh nginx &

# Run Valgrind for detailed analysis
valgrind --leak-check=full \
         --show-leak-kinds=all \
         --track-origins=yes \
         --log-file=valgrind.log \
         ./your_application
```

---

### Scenario 3: Disk I/O Bottleneck

**Problem**: Slow database queries, high I/O wait

**Diagnostic Script**:
```bash
#!/bin/bash
# File: diagnose_io.sh

DURATION=60
OUTPUT_DIR="io_diagnostics_$(date +%Y%m%d_%H%M%S)"
mkdir -p $OUTPUT_DIR

echo "=== I/O Diagnostics Started (${DURATION}s) ==="

# 1. Current I/O wait
echo "I/O Wait %:" | tee $OUTPUT_DIR/io_wait.txt
iostat -x 1 10 | grep -E "sda|nvme" >> $OUTPUT_DIR/io_wait.txt

# 2. Top I/O consumers
echo "Top I/O processes:" | tee $OUTPUT_DIR/top_io.txt
sudo iotop -b -n 5 >> $OUTPUT_DIR/top_io.txt

# 3. Detailed per-process I/O
echo "Per-process I/O:" | tee $OUTPUT_DIR/per_process_io.txt
for pid in $(pgrep -x mysql,postgres,mongod); do
    echo "PID: $pid" >> $OUTPUT_DIR/per_process_io.txt
    cat /proc/$pid/io >> $OUTPUT_DIR/per_process_io.txt
    echo "---" >> $OUTPUT_DIR/per_process_io.txt
done

# 4. File system stats
df -h > $OUTPUT_DIR/filesystem_usage.txt
mount | grep -v tmpfs > $OUTPUT_DIR/mounted_filesystems.txt

# 5. Block device stats
iostat -dx 5 12 > $OUTPUT_DIR/iostat_extended.txt

# 6. Identify slow queries (MySQL example)
if command -v mysql &>/dev/null; then
    mysql -e "SELECT * FROM information_schema.processlist WHERE TIME > 5;" > $OUTPUT_DIR/slow_queries.txt 2>/dev/null || true
fi

# 7. Check for zombie/stuck processes
ps aux | awk '$8 ~ /D/ {print}' > $OUTPUT_DIR/uninterruptible_sleep.txt

echo "=== Diagnostics Complete ==="
echo "Results in: $OUTPUT_DIR"
```

**Analysis and Optimization**:
```bash
./diagnose_io.sh

# Check results
cd io_diagnostics_*

# High %util in iostat? -> Disk bottleneck
# High await? -> Slow disk responses
# Many D state processes? -> I/O blocking

# Solutions:
# 1. Add caching
echo "vm.dirty_ratio = 15" | sudo tee -a /etc/sysctl.conf
echo "vm.dirty_background_ratio = 5" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# 2. Optimize database
# Add indexes, optimize queries

# 3. Use faster storage
# Migrate to SSD/NVMe
```

---

### Scenario 4: System-Wide Performance Baseline

**Complete Performance Snapshot Script**:
```bash
#!/bin/bash
# File: performance_baseline.sh
# Purpose: Capture complete system performance baseline

OUTPUT_DIR="baseline_$(hostname)_$(date +%Y%m%d_%H%M%S)"
mkdir -p $OUTPUT_DIR

echo "=== Creating Performance Baseline ==="

# System info
uname -a > $OUTPUT_DIR/system_info.txt
cat /etc/os-release >> $OUTPUT_DIR/system_info.txt

# CPU info
lscpu > $OUTPUT_DIR/cpu_info.txt
cat /proc/cpuinfo > $OUTPUT_DIR/cpu_detailed.txt

# Memory info
free -h > $OUTPUT_DIR/memory_info.txt
cat /proc/meminfo > $OUTPUT_DIR/memory_detailed.txt

# Disk info
lsblk > $OUTPUT_DIR/block_devices.txt
df -h > $OUTPUT_DIR/disk_usage.txt

# Network info
ip addr > $OUTPUT_DIR/network_interfaces.txt
ss -s > $OUTPUT_DIR/socket_stats.txt

# Running processes
ps aux --sort=-%cpu | head -50 > $OUTPUT_DIR/top_cpu_processes.txt
ps aux --sort=-%mem | head -50 > $OUTPUT_DIR/top_mem_processes.txt

# Performance samples
echo "Collecting performance samples (60s)..."

# CPU
mpstat 5 12 > $OUTPUT_DIR/cpu_stats.txt &

# Memory
vmstat 5 12 > $OUTPUT_DIR/vmstat.txt &

# I/O
iostat -dx 5 12 > $OUTPUT_DIR/iostat.txt &

# Network
sar -n DEV 5 12 > $OUTPUT_DIR/network_stats.txt 2>/dev/null &

wait

# System load over time
uptime > $OUTPUT_DIR/load_average.txt

# Create summary
cat << EOF > $OUTPUT_DIR/summary.txt
=== Performance Baseline Summary ===
Date: $(date)
Hostname: $(hostname)
Uptime: $(uptime)

CPU Cores: $(nproc)
Total Memory: $(free -h | awk '/^Mem:/ {print $2}')
Disk Space: $(df -h / | awk 'NR==2 {print $2}')

Load Average: $(uptime | awk -F'load average:' '{print $2}')
EOF

tar czf ${OUTPUT_DIR}.tar.gz $OUTPUT_DIR
echo "=== Baseline Complete ==="
echo "Archive: ${OUTPUT_DIR}.tar.gz"
```

---

## 8. Cheat Sheets {#cheat-sheets}

### Quick Reference: Performance Tools

```
┌─────────────────────────────────────────────────────────────┐
│                  PERFORMANCE TOOL SELECTOR                  │
├─────────────────┬───────────────────────────────────────────┤
│ Problem         │ Tools to Use                              │
├─────────────────┼───────────────────────────────────────────┤
│ High CPU        │ top, htop, perf top, perf record         │
│ Memory Leak     │ valgrind, pmap, /proc/PID/smaps          │
│ Slow I/O        │ iostat, iotop, blktrace, iowait          │
│ Network Issues  │ ss, netstat, iftop, tcpdump              │
│ Slow Syscalls   │ strace -T, strace -c                     │
│ Library Issues  │ ltrace, ldd                               │
│ Deadlock        │ gdb, pstack, perf lock                   │
│ Cache Misses    │ perf stat -e cache-misses                │
└─────────────────┴───────────────────────────────────────────┘
```

---

### Essential Commands Reference

#### CPU Commands
```bash
# Real-time CPU monitoring
top -d 2                    # 2-second refresh
htop                        # Interactive, better UI
mpstat -P ALL 2             # Per-CPU stats every 2s

# CPU profiling
perf record -F 99 -ag -- sleep 30
perf top -e cycles          # Live function profiling
ps aux --sort=-%cpu | head -10

# Process CPU affinity
taskset -cp <pid>           # Check affinity
taskset -cp 0,1 <pid>       # Set to cores 0,1
```

#### Memory Commands
```bash
# Memory overview
free -h                     # Human-readable
vmstat 2 5                  # Virtual memory stats
cat /proc/meminfo           # Detailed memory info

# Process memory
ps aux --sort=-%mem | head -10
pmap -x <pid>              # Memory map
cat /proc/<pid>/smaps       # Detailed memory segments

# Memory leak detection
valgrind --leak-check=full ./program
```

#### I/O Commands
```bash
# I/O monitoring
iostat -xdm 2               # Extended disk stats
iotop -o                    # Only processes doing I/O
lsof -p <pid>               # Open files

# I/O profiling
strace -e trace=file <cmd>  # File operations
blktrace -d /dev/sda        # Block layer tracing
```

---

### Keyboard Shortcuts

#### `top` Shortcuts
```
P    - Sort by CPU%
M    - Sort by Memory%
T    - Sort by Time
N    - Sort by PID
1    - Toggle individual CPUs
k    - Kill process (enter PID)
r    - Renice (change priority)
f    - Field management
W    - Save configuration
q    - Quit
```

#### `htop` Shortcuts
```
F1   - Help
F2   - Setup
F3   - Search
F4   - Filter
F5   - Tree view
F6   - Sort by
F9   - Kill
F10  - Quit

Space - Tag process
U     - Show user's processes
t     - Tree view toggle
H     - Hide/show threads
```

#### `perf` Report Navigation
```
↑/↓    - Move cursor
Enter  - Expand/collapse
+      - Expand all
-      - Collapse all
a      - Annotate (view assembly)
d      - Show DSO/library
/      - Search
q      - Quit
```

---

### Performance Metrics Reference

```
CPU Metrics:
- %user:    Time in user mode (your code)
- %system:  Time in kernel mode (syscalls)
- %iowait:  Waiting for I/O (disk/network)
- %idle:    CPU doing nothing
- %steal:   Virtual CPU waiting (VMs only)

Good:     %user high, %iowait low, %idle > 20%
Warning:  %iowait > 20%, %system > 30%
Critical: %iowait > 50%, %idle < 5%

Memory Metrics:
- RSS:      Resident Set Size (physical memory)
- VSZ:      Virtual Size (allocated virtual memory)
- %MEM:     Percentage of total RAM
- Swap:     Disk-backed memory

Good:     Swap usage < 10%, free > 20%
Warning:  Swap usage 10-50%
Critical: Swap usage > 50%, thrashing

I/O Metrics:
- r/s, w/s:  Reads/writes per second
- rkB/s:     KB read per second
- wkB/s:     KB written per second
- await:     Average wait time (ms)
- %util:     Device saturation

Good:     %util < 60%, await < 10ms
Warning:  %util 60-80%, await 10-50ms
Critical: %util > 80%, await > 100ms
```

---

## 9. Hands-On Exercises {#exercises}

### Exercise 1: CPU Profiling Challenge

**Objective**: Find and fix CPU bottleneck

**Setup**:
```bash
# Create inefficient program
cat > cpu_exercise.c << 'EOF'
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Inefficient: O(n²) string search
int count_substring(const char *str, const char *substr) {
    int count = 0;
    int len = strlen(str);
    int sublen = strlen(substr);
    
    for (int i = 0; i < len; i++) {
        int match = 1;
        for (int j = 0; j < sublen; j++) {
            if (str[i+j] != substr[j]) {
                match = 0;
                break;
            }
        }
        if (match) count++;
    }
    return count;
}

int main() {
    char *text = malloc(1000000);
    memset(text, 'A', 999999);
    text[999999] = '\0';
    
    for (int i = 0; i < 100; i++) {
        int result = count_substring(text, "AAAA");
        printf("Found: %d\n", result);
    }
    
    free(text);
    return 0;
}
EOF

gcc -g -O0 -o cpu_exercise cpu_exercise.c
```

**Tasks**:
1. Profile with `perf`:
```bash
perf record -g ./cpu_exercise
perf report
# Question: Which function consumes most CPU?
```

2. Profile with `time`:
```bash
time ./cpu_exercise
# Note: real, user, sys times
```

3. Profile with `gprof`:
```bash
gcc -pg -o cpu_exercise_prof cpu_exercise.c
./cpu_exercise_prof
gprof cpu_exercise_prof gmon.out > analysis.txt
cat analysis.txt
```

**Challenge**: Optimize the `count_substring` function
- Hint: Use Boyer-Moore or KMP algorithm
- Target: 10x speedup

---

### Exercise 2: Memory Leak Hunt

**Setup**:
```bash
cat > memory_exercise.c << 'EOF'
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

typedef struct {
    int id;
    char data[1024];
} Record;

void process_records(int count) {
    for (int i = 0; i < count; i++) {
        Record *r = malloc(sizeof(Record));
        r->id = i;
        // BUG: Forgot to free!
        
        if (i % 100 == 0) {
            printf("Processed %d records\n", i);
        }
    }
}

int main() {
    printf("PID: %d\n", getpid());
    
    for (int i = 0; i < 10; i++) {
        process_records(1000);
        sleep(2);
    }
    
    return 0;
}
EOF

gcc -g -o memory_exercise memory_exercise.c
```

**Tasks**:
1. Monitor memory growth:
```bash
./memory_exercise &
PID=$!
watch -n 1 "ps -o pid,vsz,rss,cmd -p $PID"
```

2. Detect leak with Valgrind:
```bash
valgrind --leak-check=full --show-leak-kinds=all ./memory_exercise
# Question: How many bytes leaked?
```

3. Find leak source:
```bash
valgrind --leak-check=full --track-origins=yes ./memory_exercise 2>&1 | grep -A 10 "definitely lost"
```

**Challenge**: Fix all leaks, verify with Valgrind showing "no leaks possible"

---

### Exercise 3: I/O Optimization

**Setup**:
```bash
#!/bin/bash
# File: io_exercise.sh

# Create test files
mkdir -p /tmp/io_test
for i in {1..100}; do
    dd if=/dev/urandom of=/tmp/io_test/file_$i bs=1M count=10 2>/dev/null
done

# Inefficient: Read all files sequentially
echo "Method 1: Sequential reads"
time {
    for file in /tmp/io_test/*; do
        cat "$file" > /dev/null
    done
}
```

**Tasks**:
1. Measure baseline I/O:
```bash
./io_exercise.sh

# Monitor I/O during execution
sudo iotop -b -n 10 > iotop_baseline.txt
iostat -dx 1 10 > iostat_baseline.txt
```

2. Optimize with parallel reads:
```bash
#!/bin/bash
# File: io_exercise_optimized.sh

echo "Method 2: Parallel reads"
time {
    for file in /tmp/io_test/*; do
        cat "$file" > /dev/null &
        
        # Limit to 8 concurrent
        if [ $(jobs -r | wc -l) -ge 8 ]; then
            wait -n
        fi
    done
    wait
}
```

3. Use memory-mapped I/O:
```c
// File: mmap_read.c
#include <stdio.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

int main(int argc, char **argv) {
    int fd = open(argv[1], O_RDONLY);
    struct stat sb;
    fstat(fd, &sb);
    
    char *data = mmap(NULL, sb.st_size, PROT_READ, MAP_PRIVATE, fd, 0);
    
    // Process data (simulated)
    long sum = 0;
    for (size_t i = 0; i < sb.st_size; i++) {
        sum += data[i];
    }
    
    munmap(data, sb.st_size);
    close(fd);
    
    printf("Checksum: %ld\n", sum);
    return 0;
}
```

**Challenge**: Achieve 3x speedup from baseline

---

### Exercise 4: Production Simulation

**Scenario**: Web application under load

**Setup**:
```bash
#!/bin/bash
# File: production_sim.sh
# Simulates: Web server, database, background jobs

# Web server (CPU-bound)
web_server() {
    while true; do
        # Simulate request processing
        echo "scale=1000; 4*a(1)" | bc -l > /dev/null
        sleep 0.1
    done
}

# Database (I/O-bound)
database() {
    while true; do
        # Simulate queries
        dd if=/dev/urandom of=/tmp/db_$$ bs=1M count=10 2>/dev/null
        rm -f /tmp/db_$$
        sleep 0.5
    done
}

# Background job (Memory-intensive)
background_job() {
    while true; do
        # Allocate and release memory
        LEAK=$(head -c 50M /dev/zero | tr '\0' 'X')
        sleep 5
    done
}

# Start all services
web_server &
WEB_PID=$!

database &
DB_PID=$!

background_job &
BG_PID=$!

echo "Services started:"
echo "  Web: $WEB_PID"
echo "  DB:  $DB_PID"
echo "  BG:  $BG_PID"

# Cleanup function
cleanup() {
    kill $WEB_PID $DB_PID $BG_PID 2>/dev/null
    exit 0
}

trap cleanup SIGINT SIGTERM

wait
```

**Tasks**:
1. **Baseline Performance**:
```bash
./production_sim.sh &
MAIN_PID=$!

# Capture baseline
./performance_baseline.sh
```

2. **Stress Test**:
```bash
# Add load
stress-ng --cpu 4 --io 2 --vm 1 --vm-bytes 512M --timeout 60s &

# Monitor during stress
htop  # Terminal 1
sudo iotop  # Terminal 2
```

3. **Identify Bottleneck**:
```bash
# Complete investigation
./investigate_cpu.sh    # If CPU high
./diagnose_io.sh        # If I/O wait high
./detect_memory_leak.sh production_sim  # If memory grows
```

4. **Optimize**:
- Reduce web server CPU usage (use caching)
- Optimize database I/O (batch operations)
- Fix background job memory leak

**Success Criteria**:
- CPU usage < 70%
- I/O wait < 20%
- Memory stable over 10 minutes

---

### Exercise 5: Advanced Profiling Challenge

**Objective**: Multi-tool profiling workflow

**Setup**:
```bash
cat > complex_app.c << 'EOF'
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <pthread.h>

#define NUM_THREADS 4

// CPU-intensive function
void* cpu_worker(void *arg) {
    int id = *(int*)arg;
    for (int i = 0; i < 1000000; i++) {
        double result = 0;
        for (int j = 0; j < 100; j++) {
            result += (double)i / (j + 1);
        }
    }
    return NULL;
}

// Memory-intensive function
void* mem_worker(void *arg) {
    for (int i = 0; i < 100; i++) {
        char *data = malloc(1024 * 1024);  // 1MB
        memset(data, 'A', 1024 * 1024);
        // Intentional leak for exercise
        usleep(100000);
    }
    return NULL;
}

// I/O-intensive function
void* io_worker(void *arg) {
    int id = *(int*)arg;
    char filename[256];
    snprintf(filename, sizeof(filename), "/tmp/io_test_%d", id);
    
    for (int i = 0; i < 100; i++) {
        FILE *f = fopen(filename, "w");
        for (int j = 0; j < 1000; j++) {
            fprintf(f, "Line %d\n", j);
        }
        fclose(f);
        usleep(10000);
    }
    
    unlink(filename);
    return NULL;
}

int main() {
    pthread_t threads[NUM_THREADS * 3];
    int thread_ids[NUM_THREADS * 3];
    
    printf("Starting complex application (PID: %d)\n", getpid());
    
    // Start CPU workers
    for (int i = 0; i < NUM_THREADS; i++) {
        thread_ids[i] = i;
        pthread_create(&threads[i], NULL, cpu_worker, &thread_ids[i]);
    }
    
    // Start memory workers
    for (int i = 0; i < NUM_THREADS; i++) {
        thread_ids[NUM_THREADS + i] = i;
        pthread_create(&threads[NUM_THREADS + i], NULL, mem_worker, &thread_ids[NUM_THREADS + i]);
    }
    
    // Start I/O workers
    for (int i = 0; i < NUM_THREADS; i++) {
        thread_ids[NUM_THREADS * 2 + i] = i;
        pthread_create(&threads[NUM_THREADS * 2 + i], NULL, io_worker, &thread_ids[NUM_THREADS * 2 + i]);
    }
    
    // Join all threads
    for (int i = 0; i < NUM_THREADS * 3; i++) {
        pthread_join(threads[i], NULL);
    }
    
    printf("Application finished\n");
    return 0;
}
EOF

gcc -g -pthread -o complex_app complex_app.c
```

**Master Script**:
```bash
#!/bin/bash
# File: full_profiling_workflow.sh

APP="./complex_app"
OUTPUT="profiling_results_$(date +%Y%m%d_%H%M%S)"
mkdir -p $OUTPUT

echo "=== Starting Comprehensive Profiling ==="

# 1. Run with time
echo "[1/7] Basic timing..."
/usr/bin/time -v $APP > $OUTPUT/time_output.txt 2>&1

# 2. Run with perf
echo "[2/7] CPU profiling with perf..."
perf record -g -o $OUTPUT/perf.data $APP > /dev/null 2>&1
perf report -i $OUTPUT/perf.data --stdio > $OUTPUT/perf_report.txt

# 3. Run with strace
echo "[3/7] System call tracing..."
strace -c -o $OUTPUT/strace_summary.txt $APP > /dev/null 2>&1

# 4. Run with ltrace
echo "[4/7] Library call tracing..."
ltrace -c -o $OUTPUT/ltrace_summary.txt $APP 2>&1 | head -100 > $OUTPUT/ltrace_output.txt

# 5. Run with valgrind (memory)
echo "[5/7] Memory profiling (slow)..."
valgrind --leak-check=full --log-file=$OUTPUT/valgrind.txt $APP > /dev/null 2>&1

# 6. Run with valgrind (cache)
echo "[6/7] Cache profiling..."
valgrind --tool=cachegrind --cachegrind-out-file=$OUTPUT/cachegrind.out $APP > /dev/null 2>&1
cg_annotate $OUTPUT/cachegrind.out > $OUTPUT/cachegrind_report.txt

# 7. Run with gdb (thread analysis)
echo "[7/7] Thread analysis..."
gdb -batch -ex "set pagination off" -ex "run" -ex "info threads" -ex "thread apply all bt" -ex "quit" $APP > $OUTPUT/gdb_threads.txt 2>&1

echo "=== Profiling Complete ==="
echo "Results in: $OUTPUT/"
echo ""
echo "Summary:"
echo "  Execution time:  $(grep "Elapsed" $OUTPUT/time_output.txt)"
echo "  Memory leaked:   $(grep "definitely lost" $OUTPUT/valgrind.txt)"
echo "  Top syscall:     $(head -2 $OUTPUT/strace_summary.txt | tail -1)"
```

**Tasks**:
1. Run complete profiling:
```bash
chmod +x full_profiling_workflow.sh
./full_profiling_workflow.sh
```

2. Analyze each report:
```bash
cd profiling_results_*

# Time breakdown
cat time_output.txt

# CPU hotspots
head -50 perf_report.txt

# Syscall distribution
cat strace_summary.txt

# Memory leaks
grep -A 5 "LEAK SUMMARY" valgrind.txt

# Cache efficiency
grep "refs" cachegrind_report.txt
```

3. Create optimization plan based on findings

**Expected Findings**:
- CPU: 40% in cpu_worker
- Memory: 400MB leaked in mem_worker
- I/O: Excessive open/close calls
- Cache: Poor locality in cpu_worker

**Optimization Goals**:
- Fix memory leak (100% reduction)
- Reduce syscalls (50% reduction)
- Improve cache hits (20% improvement)

---

## 10. Muscle Memory Drills

### Drill 1: Fast Performance Check (30 seconds)
```bash
# Practice this sequence until it's automatic

# 1. CPU check (5s)
top -b -n 1 | head -20

# 2. Memory check (5s)
free -h && ps aux --sort=-%mem | head -5

# 3. I/O check (5s)
iostat -dx 1 2

# 4. Network check (5s)
ss -s

# 5. Process tree (5s)
pstree -p | head -20

# 6. Load average (5s)
uptime && w
```

**Practice**: Run this sequence 10 times, reducing time each iteration

---

### Drill 2: Quick Profiling (1 minute)
```bash
# Given a PID, profile in 60 seconds

PID=1234  # Replace with actual PID

# CPU (20s)
perf record -F 99 -p $PID -g -- sleep 20 &

# Memory snapshot (5s)
pmap -x $PID > pmap_snapshot.txt

# Syscalls (20s)
strace -p $PID -c -f 2> strace.txt &

# I/O (15s)
lsof -p $PID > open_files.txt

# Wait for perf to finish
wait

# Quick report
perf report --stdio | head -30
cat strace.txt
```

**Practice**: Profile different processes, aim for < 60s total time

---

### Drill 3: Investigation Workflow
```bash
# Memorize this investigation sequence:

# 1. IDENTIFY
ps aux --sort=-%cpu | head -3       # Top CPU
ps aux --sort=-%mem | head -3       # Top memory

# 2. ISOLATE
PID=$(ps aux --sort=-%cpu | awk 'NR==2 {print $2}')
ps -p $PID -o pid,ppid,cmd,%cpu,%mem,stat

# 3. INSPECT
lsof -p $PID | wc -l                # Open files count
ls -l /proc/$PID/fd | wc -l         # File descriptors
cat /proc/$PID/status | grep -E "Vm|Threads"

# 4. INVESTIGATE
strace -p $PID -c -f &              # Background trace
sleep 10
pkill strace

# 5. IMPLEMENT (fix based on findings)
```

**Practice**: Complete this workflow in < 2 minutes

---

## Common Mistakes & Solutions

### Mistake 1: Profiling in Production Without Safety
```bash
# WRONG: Unlimited profiling
perf record -a -g  # Runs forever!

# RIGHT: Time-limited
perf record -a -g -- sleep 30

# RIGHT: Specific process
perf record -p $PID -g -- sleep 10
```

---

### Mistake 2: Ignoring Compiler Optimization
```bash
# WRONG: Debug symbols without optimization
gcc -g program.c -o program

# RIGHT: Optimize AND keep symbols
gcc -g -O2 program.c -o program

# For profiling: Frame pointers
gcc -g -O2 -fno-omit-frame-pointer program.c -o program
```

---

### Mistake 3: Memory Leak vs Memory Growth
```bash
# Not all memory growth is a leak!

# Check if memory is released eventually:
watch -n 2 "ps -o pid,vsz,rss,cmd -p $PID"

# If VSZ grows but RSS doesn't: Virtual memory, OK
# If both grow indefinitely: Likely leak
```

---

### Mistake 4: Wrong I/O Benchmarking
```bash
# WRONG: Cached I/O (false fast results)
dd if=/dev/zero of=testfile bs=1M count=1000

# RIGHT: Direct I/O (real performance)
dd if=/dev/zero of=testfile bs=1M count=1000 conv=fdatasync

# BETTER: Drop caches first (requires root)
sync && echo 3 | sudo tee /proc/sys/vm/drop_caches
```

---

### Mistake 5: Profiling Short-Lived Processes
```bash
# WRONG: Process ends before profiling
perf record -p $PID  # Already finished!

# RIGHT: Profile the launcher
perf record -g ./script.sh

# Or use wrapper
perf record -g -- bash -c "your_command"
```

---

## Final Troubleshooting Decision Tree

```
Performance Issue?
│
├─ High CPU? (>80%)
│  ├─ User mode? → Profile with perf, optimize algorithm
│  ├─ System mode? → Check syscalls with strace
│  └─ IOWait? → See I/O branch
│
├─ High Memory? (>80%)
│  ├─ Growing? → Check for leaks (valgrind, pmap)
│  ├─ Swap usage? → Add RAM or reduce usage
│  └─ OOM kills? → Check dmesg, adjust limits
│
├─ High I/O?
│  ├─ Disk %util >80%? → Faster storage or caching
│  ├─ Many small files? → Batch operations
│  └─ Random access? → Sequential access or index
│
└─ Network Issues?
   ├─ Many connections? → Connection pooling
   ├─ High latency? → Check network with ping/mtr
   └─ Packet loss? → Check with netstat -s
```

---

## Summary Checklist

**Before Production Deployment**:
- [ ] Profile under realistic load
- [ ] Check for memory leaks (valgrind clean)
- [ ] Optimize hot paths (perf report)
- [ ] Reduce unnecessary syscalls
- [ ] Enable compiler optimizations (-O2/-O3)
- [ ] Set appropriate ulimits
- [ ] Configure monitoring/alerting
- [ ] Document baseline performance

**During Performance Issues**:
- [ ] Capture baseline metrics
- [ ] Identify bottleneck (CPU/Memory/I/O)
- [ ] Profile with appropriate tool
- [ ] Create reproducible test case
- [ ] Implement fix
- [ ] Verify improvement
- [ ] Document for future reference

**Key Takeaways**:
1. **Measure first, optimize second** - Don't guess
2. **Profile in realistic conditions** - Not synthetic benchmarks
3. **Focus on bottlenecks** - Amdahl's Law (optimize slowest part)
4. **Verify improvements** - Always benchmark before/after
5. **Document everything** - Future you will thank you

---

This guide provides everything needed to master performance troubleshooting in DevOps. Practice the exercises, memorize the workflows, and build muscle memory through repetition.