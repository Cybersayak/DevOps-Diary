# 🎯 TASK 2: Error Handling & Debugging Mastery (1 hr)

---

## 📚 TABLE OF CONTENTS

1. [Concept Explanation: Basics to Advanced](#section-1)
2. [Syntax Deep Dive & Variations](#section-2)
3. [Practical Implementation Examples](#section-3)
4. [The `trap` Command Mastery](#section-4)
5. [Debugging Tools & Techniques](#section-5)
6. [Optimization Techniques](#section-6)
7. [Common Mistakes & Edge Cases](#section-7)
8. [Mini Challenges](#section-8)
9. [Final Test: Fix Broken Scripts](#section-9)
10. [Cheat Sheet](#section-10)
11. [Time-Based Learning Plan](#section-11)

---

<a name="section-1"></a>
## 1️⃣ CONCEPT EXPLANATION: BASICS TO ADVANCED

### 🔹 What is Error Handling?

**Error handling** is the practice of anticipating, detecting, and responding to errors that occur during script execution.

#### WHY is Error Handling Critical?

```
WITHOUT Error Handling:
┌─────────────────────────────────────────────────────────┐
│ Script runs → Error occurs → Script continues blindly   │
│ → Data corruption → System damage → Hours of debugging  │
└─────────────────────────────────────────────────────────┘

WITH Error Handling:
┌─────────────────────────────────────────────────────────┐
│ Script runs → Error occurs → Script detects it          │
│ → Cleanup happens → User notified → Graceful exit       │
└─────────────────────────────────────────────────────────┘
```

### 🔹 Exit Codes: The Foundation

Every command in Linux returns an **exit code** (also called return code or exit status).

```bash
# Exit code meanings
0       = Success (command worked perfectly)
1       = General errors (catchall for most failures)
2       = Misuse of shell command (wrong syntax)
126     = Command found but not executable
127     = Command not found
128     = Invalid exit argument
128+N   = Fatal signal N (e.g., 130 = Ctrl+C which is signal 2)
255     = Exit status out of range
```

#### Checking Exit Codes

```bash
#!/bin/bash
# check_exit_codes.sh

# The special variable $? holds the exit code of the LAST command
ls /existing/directory
echo "Exit code: $?"   # Output: Exit code: 0

ls /nonexistent/path 2>/dev/null
echo "Exit code: $?"   # Output: Exit code: 2 (or 1 depending on system)

# Real example with explanation
grep "pattern" /etc/passwd
exit_code=$?

if [ $exit_code -eq 0 ]; then
    echo "Pattern found"
elif [ $exit_code -eq 1 ]; then
    echo "Pattern not found (but command worked)"
elif [ $exit_code -eq 2 ]; then
    echo "Error occurred (file not found, etc.)"
fi
```

### 🔹 Levels of Error Handling Maturity

```
Level 1: NONE (Beginner mistakes)
─────────────────────────────────
rm -rf $directory    # If $directory is empty, this becomes rm -rf /

Level 2: BASIC (Check important commands)
─────────────────────────────────────────
if [ -n "$directory" ]; then
    rm -rf "$directory"
fi

Level 3: INTERMEDIATE (Use set options)
───────────────────────────────────────
set -e  # Exit on error
set -u  # Exit on undefined variable

Level 4: ADVANCED (Comprehensive handling)
──────────────────────────────────────────
set -euo pipefail
trap cleanup EXIT
function cleanup() { ... }

Level 5: EXPERT (Production-grade)
──────────────────────────────────
- Custom error functions with stack traces
- Logging integration
- Retry mechanisms
- Transaction-like rollback
```

---

<a name="section-2"></a>
## 2️⃣ SYNTAX DEEP DIVE & VARIATIONS

### 🔹 The `set` Builtin Options

```bash
#!/bin/bash
# set_options_explained.sh

# ═══════════════════════════════════════════════════════════════
# set -e (errexit) - Exit immediately if a command exits non-zero
# ═══════════════════════════════════════════════════════════════

set -e

echo "This will print"
false                          # This command always returns exit code 1
echo "This will NEVER print"   # Script exits before reaching here

# ═══════════════════════════════════════════════════════════════
# set -u (nounset) - Treat undefined variables as errors
# ═══════════════════════════════════════════════════════════════

set -u

echo "Hello, $undefined_variable"  # ERROR: undefined_variable: unbound variable
# Script exits immediately

# ═══════════════════════════════════════════════════════════════
# set -o pipefail - Pipeline fails if ANY command fails
# ═══════════════════════════════════════════════════════════════

# WITHOUT pipefail:
set +o pipefail
false | echo "hello" | true
echo "Exit code: $?"   # Output: 0 (only last command matters)

# WITH pipefail:
set -o pipefail
false | echo "hello" | true
echo "Exit code: $?"   # Output: 1 (first failure is captured)

# ═══════════════════════════════════════════════════════════════
# set -x (xtrace) - Print commands before execution (debugging)
# ═══════════════════════════════════════════════════════════════

set -x
name="DevOps"
echo "Hello, $name"
# Output:
# + name=DevOps
# + echo 'Hello, DevOps'
# Hello, DevOps

# ═══════════════════════════════════════════════════════════════
# set -E (errtrace) - ERR trap is inherited by functions
# ═══════════════════════════════════════════════════════════════

set -E
trap 'echo "Error on line $LINENO"' ERR

function my_function() {
    false   # This will trigger the trap
}

my_function   # Error is caught even inside function
```

### 🔹 The Production-Ready Header

```bash
#!/bin/bash
#
# Script: production_template.sh
# Description: Template with comprehensive error handling
# Author: Your Name
# Date: 2024-01-15
#

# ═══════════════════════════════════════════════════════════════
# STRICT MODE - The "Unofficial Bash Strict Mode"
# ═══════════════════════════════════════════════════════════════

set -euo pipefail   # Combines -e, -u, and -o pipefail
IFS=$'\n\t'         # Internal Field Separator: only newline and tab
                    # (prevents word splitting on spaces)

# ═══════════════════════════════════════════════════════════════
# EXPLANATION OF EACH COMPONENT:
# ═══════════════════════════════════════════════════════════════
#
# -e (errexit):
#     Script exits if any command returns non-zero
#     EXCEPTION: Commands in if/while/until conditions
#     EXCEPTION: Commands with || or && operators
#
# -u (nounset):
#     Script exits if you use undefined variable
#     Prevents: rm -rf $UNDEFINED/ becoming rm -rf /
#
# -o pipefail:
#     Pipeline returns rightmost non-zero exit code
#     Without: `curl ... | grep ...` might hide curl failures
#
# IFS=$'\n\t':
#     Only split on newlines and tabs, not spaces
#     Prevents: filename="my file.txt" being split into two words
#
```

### 🔹 Conditional Execution Operators

```bash
#!/bin/bash
# conditional_operators.sh

# ═══════════════════════════════════════════════════════════════
# && (AND) - Run second command ONLY if first succeeds
# ═══════════════════════════════════════════════════════════════

mkdir /tmp/newdir && echo "Directory created successfully"
# If mkdir fails, echo doesn't run

# Chaining multiple commands
cd /project && npm install && npm build && npm deploy
# Stops at first failure

# ═══════════════════════════════════════════════════════════════
# || (OR) - Run second command ONLY if first fails
# ═══════════════════════════════════════════════════════════════

mkdir /tmp/existingdir 2>/dev/null || echo "Directory already exists or can't create"

# Common pattern: command || exit
cd /critical/directory || { echo "Cannot cd to directory"; exit 1; }

# ═══════════════════════════════════════════════════════════════
# Combining && and || for if-then-else
# ═══════════════════════════════════════════════════════════════

# Pattern: command && success_action || failure_action
[ -f /etc/passwd ] && echo "File exists" || echo "File missing"

# WARNING: This pattern has a subtle bug!
# If success_action fails, failure_action also runs!

true && false || echo "This prints unexpectedly!"
# Output: This prints unexpectedly!

# BETTER: Use explicit if-then-else for important logic
if [ -f /etc/passwd ]; then
    echo "File exists"
else
    echo "File missing"
fi
```

### 🔹 The `||` and `&&` with `set -e`

```bash
#!/bin/bash
set -e

# These commands WON'T exit the script on failure (set -e exception)
false || true                    # Explicit error handling
false && echo "never runs"       # Part of compound command
if false; then echo "no"; fi     # Condition context

# This WILL exit the script
false   # No error handling, script exits here

echo "Never reached"
```

---

<a name="section-3"></a>
## 3️⃣ PRACTICAL IMPLEMENTATION EXAMPLES

### 🔹 Example 1: Basic Error Handling Pattern

```bash
#!/bin/bash
# basic_error_handling.sh
# Demonstrates fundamental error handling patterns

set -euo pipefail

# ═══════════════════════════════════════════════════════════════
# CONFIGURATION
# ═══════════════════════════════════════════════════════════════

readonly SCRIPT_NAME=$(basename "$0")
readonly LOG_FILE="/var/log/${SCRIPT_NAME%.sh}.log"
readonly REQUIRED_COMMANDS=("curl" "jq" "grep")

# ═══════════════════════════════════════════════════════════════
# LOGGING FUNCTIONS
# ═══════════════════════════════════════════════════════════════

log_info() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [INFO] $*" | tee -a "$LOG_FILE"
}

log_error() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [ERROR] $*" | tee -a "$LOG_FILE" >&2
}

log_warn() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [WARN] $*" | tee -a "$LOG_FILE"
}

# ═══════════════════════════════════════════════════════════════
# ERROR HANDLING FUNCTION
# ═══════════════════════════════════════════════════════════════

die() {
    local message="${1:-Unknown error}"
    local exit_code="${2:-1}"
    
    log_error "$message"
    exit "$exit_code"
}

# ═══════════════════════════════════════════════════════════════
# VALIDATION FUNCTIONS
# ═══════════════════════════════════════════════════════════════

check_root() {
    if [[ $EUID -ne 0 ]]; then
        die "This script must be run as root" 1
    fi
    log_info "Root check passed"
}

check_dependencies() {
    local missing=()
    
    for cmd in "${REQUIRED_COMMANDS[@]}"; do
        if ! command -v "$cmd" &>/dev/null; then
            missing+=("$cmd")
        fi
    done
    
    if [[ ${#missing[@]} -gt 0 ]]; then
        die "Missing required commands: ${missing[*]}" 2
    fi
    log_info "All dependencies present"
}

# ═══════════════════════════════════════════════════════════════
# MAIN LOGIC
# ═══════════════════════════════════════════════════════════════

main() {
    log_info "Starting ${SCRIPT_NAME}"
    
    # Validation
    check_dependencies
    
    # Your actual logic here
    log_info "Performing main tasks..."
    
    log_info "${SCRIPT_NAME} completed successfully"
}

# Run main function
main "$@"
```

**Expected Output (Success):**
```
[2024-01-15 10:30:45] [INFO] Starting basic_error_handling.sh
[2024-01-15 10:30:45] [INFO] All dependencies present
[2024-01-15 10:30:45] [INFO] Performing main tasks...
[2024-01-15 10:30:45] [INFO] basic_error_handling.sh completed successfully
```

**Expected Output (Missing dependency):**
```
[2024-01-15 10:30:45] [INFO] Starting basic_error_handling.sh
[2024-01-15 10:30:45] [ERROR] Missing required commands: jq
```

---

### 🔹 Example 2: Advanced Error Handling with Stack Traces

```bash
#!/bin/bash
# advanced_error_handling.sh
# Production-grade error handling with stack traces

set -Euo pipefail

# ═══════════════════════════════════════════════════════════════
# GLOBAL VARIABLES
# ═══════════════════════════════════════════════════════════════

declare -r SCRIPT_NAME=$(basename "${BASH_SOURCE[0]}")
declare -r SCRIPT_DIR=$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)
declare -r TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# Error tracking
declare -g LAST_ERROR=""
declare -g ERROR_LINENO=0
declare -g ERROR_COMMAND=""

# ═══════════════════════════════════════════════════════════════
# STACK TRACE FUNCTION
# ═══════════════════════════════════════════════════════════════

print_stack_trace() {
    local frame=0
    echo ""
    echo "═══════════════════════════════════════════════════════════"
    echo "                     STACK TRACE                           "
    echo "═══════════════════════════════════════════════════════════"
    
    while caller $frame; do
        ((frame++))
    done | while read line func file; do
        echo "  → $file:$line in $func()"
    done
    
    echo "═══════════════════════════════════════════════════════════"
}

# ═══════════════════════════════════════════════════════════════
# ERROR HANDLER (Called by trap)
# ═══════════════════════════════════════════════════════════════

error_handler() {
    local exit_code=$?
    local line_number=$1
    local command="$2"
    
    # Don't handle if exit code is 0
    [[ $exit_code -eq 0 ]] && return
    
    echo ""
    echo "┌─────────────────────────────────────────────────────────┐"
    echo "│                    ERROR DETECTED                       │"
    echo "├─────────────────────────────────────────────────────────┤"
    echo "│  Exit Code  : $exit_code"
    echo "│  Line Number: $line_number"
    echo "│  Command    : $command"
    echo "│  Script     : ${BASH_SOURCE[1]:-$SCRIPT_NAME}"
    echo "└─────────────────────────────────────────────────────────┘"
    
    print_stack_trace
}

# ═══════════════════════════════════════════════════════════════
# SETUP TRAPS
# ═══════════════════════════════════════════════════════════════

# ERR trap - catches command failures
trap 'error_handler $LINENO "$BASH_COMMAND"' ERR

# ═══════════════════════════════════════════════════════════════
# TEST FUNCTIONS (Demonstrating nested calls)
# ═══════════════════════════════════════════════════════════════

level_3_function() {
    echo "  → Entering level_3_function"
    
    # This will fail and trigger the error handler
    cat /nonexistent/file/path.txt
    
    echo "  → This line never executes"
}

level_2_function() {
    echo " → Entering level_2_function"
    level_3_function
    echo " → Leaving level_2_function"
}

level_1_function() {
    echo "→ Entering level_1_function"
    level_2_function
    echo "→ Leaving level_1_function"
}

# ═══════════════════════════════════════════════════════════════
# MAIN
# ═══════════════════════════════════════════════════════════════

main() {
    echo "Starting script with advanced error handling..."
    echo ""
    
    level_1_function
    
    echo "Script completed successfully"
}

main "$@"
```

**Expected Output:**
```
Starting script with advanced error handling...

→ Entering level_1_function
 → Entering level_2_function
  → Entering level_3_function

┌─────────────────────────────────────────────────────────┐
│                    ERROR DETECTED                       │
├─────────────────────────────────────────────────────────┤
│  Exit Code  : 1
│  Line Number: 58
│  Command    : cat /nonexistent/file/path.txt
│  Script     : advanced_error_handling.sh
└─────────────────────────────────────────────────────────┘

═══════════════════════════════════════════════════════════
                     STACK TRACE                           
═══════════════════════════════════════════════════════════
  → advanced_error_handling.sh:58 in level_3_function()
  → advanced_error_handling.sh:65 in level_2_function()
  → advanced_error_handling.sh:71 in level_1_function()
  → advanced_error_handling.sh:80 in main()
═══════════════════════════════════════════════════════════
```

---

<a name="section-4"></a>
## 4️⃣ THE `trap` COMMAND MASTERY

### 🔹 What is `trap`?

`trap` allows you to catch **signals** and execute code when they occur. Think of it as event handlers for your script.

```bash
# Basic syntax
trap 'commands' SIGNAL [SIGNAL ...]

# Common signals
trap 'commands' EXIT      # Script exits (success or failure)
trap 'commands' ERR       # Command returns non-zero
trap 'commands' INT       # Ctrl+C pressed (interrupt)
trap 'commands' TERM      # kill command sent (terminate)
trap 'commands' HUP       # Terminal closed (hangup)
trap 'commands' DEBUG     # Before EVERY command (debugging)
trap 'commands' RETURN    # Function/source returns
```

### 🔹 Signal Reference Table

```
┌──────────┬────────┬────────────────────────────────────────────┐
│  Signal  │ Number │  Description                               │
├──────────┼────────┼────────────────────────────────────────────┤
│ EXIT     │   0    │ Script exit (any reason)                   │
│ HUP      │   1    │ Hangup (terminal closed)                   │
│ INT      │   2    │ Interrupt (Ctrl+C)                         │
│ QUIT     │   3    │ Quit (Ctrl+\)                              │
│ TERM     │  15    │ Terminate (kill command default)           │
│ KILL     │   9    │ Kill (cannot be trapped!)                  │
│ ERR      │   -    │ Command error (non-zero exit)              │
│ DEBUG    │   -    │ Before each command                        │
│ RETURN   │   -    │ Function/sourced script returns            │
└──────────┴────────┴────────────────────────────────────────────┘
```

### 🔹 Complete trap Examples

```bash
#!/bin/bash
# trap_mastery.sh
# Comprehensive trap examples

set -euo pipefail

# ═══════════════════════════════════════════════════════════════
# TEMPORARY FILE MANAGEMENT
# ═══════════════════════════════════════════════════════════════

# Create temporary files safely
readonly TEMP_DIR=$(mktemp -d -t "script_$$_XXXXXX")
readonly TEMP_FILE=$(mktemp -t "data_$$_XXXXXX")
readonly LOCK_FILE="/var/lock/${0##*/}.lock"

echo "Temp directory: $TEMP_DIR"
echo "Temp file: $TEMP_FILE"

# ═══════════════════════════════════════════════════════════════
# CLEANUP FUNCTION
# ═══════════════════════════════════════════════════════════════

cleanup() {
    local exit_code=$?
    
    echo ""
    echo "┌─────────────────────────────────────────┐"
    echo "│           CLEANUP INITIATED             │"
    echo "└─────────────────────────────────────────┘"
    
    # Remove temporary files
    if [[ -f "$TEMP_FILE" ]]; then
        echo "→ Removing temp file: $TEMP_FILE"
        rm -f "$TEMP_FILE"
    fi
    
    # Remove temporary directory
    if [[ -d "$TEMP_DIR" ]]; then
        echo "→ Removing temp directory: $TEMP_DIR"
        rm -rf "$TEMP_DIR"
    fi
    
    # Release lock file
    if [[ -f "$LOCK_FILE" ]]; then
        echo "→ Releasing lock: $LOCK_FILE"
        rm -f "$LOCK_FILE"
    fi
    
    # Report final status
    if [[ $exit_code -eq 0 ]]; then
        echo "→ Script completed successfully"
    else
        echo "→ Script failed with exit code: $exit_code"
    fi
    
    echo "┌─────────────────────────────────────────┐"
    echo "│           CLEANUP COMPLETE              │"
    echo "└─────────────────────────────────────────┘"
    
    exit $exit_code
}

# ═══════════════════════════════════════════════════════════════
# INTERRUPT HANDLER (Ctrl+C)
# ═══════════════════════════════════════════════════════════════

handle_interrupt() {
    echo ""
    echo "⚠️  INTERRUPT RECEIVED (Ctrl+C)"
    echo "   Cleaning up before exit..."
    exit 130  # Standard exit code for Ctrl+C
}

# ═══════════════════════════════════════════════════════════════
# TERMINATION HANDLER (kill command)
# ═══════════════════════════════════════════════════════════════

handle_term() {
    echo ""
    echo "⚠️  TERMINATION SIGNAL RECEIVED"
    echo "   Performing graceful shutdown..."
    exit 143  # Standard exit code for SIGTERM
}

# ═══════════════════════════════════════════════════════════════
# REGISTER TRAPS
# ═══════════════════════════════════════════════════════════════

# EXIT trap: ALWAYS runs, regardless of how script exits
trap cleanup EXIT

# INT trap: Ctrl+C handling
trap handle_interrupt INT

# TERM trap: kill command handling
trap handle_term TERM

# ═══════════════════════════════════════════════════════════════
# MAIN SCRIPT LOGIC
# ═══════════════════════════════════════════════════════════════

main() {
    echo "Script PID: $$"
    echo "Press Ctrl+C to test interrupt handling"
    echo "Or run: kill $$ from another terminal"
    echo ""
    
    # Simulate work with temp files
    echo "Creating data in temp locations..."
    echo "Important data" > "$TEMP_FILE"
    touch "$TEMP_DIR/work_file_1.txt"
    touch "$TEMP_DIR/work_file_2.txt"
    
    # Simulate long-running process
    for i in {1..10}; do
        echo "Working... step $i/10"
        sleep 1
    done
    
    echo ""
    echo "Main work completed!"
}

main "$@"
```

**Test Scenarios:**

```bash
# Scenario 1: Normal completion
$ ./trap_mastery.sh
# Let it run to completion
# Output shows cleanup happening after success

# Scenario 2: Ctrl+C interrupt
$ ./trap_mastery.sh
# Press Ctrl+C while running
# Output: INTERRUPT RECEIVED message, then cleanup

# Scenario 3: Kill signal
# Terminal 1:
$ ./trap_mastery.sh

# Terminal 2:
$ kill $(pgrep -f trap_mastery.sh)
# Output in Terminal 1: TERMINATION SIGNAL message, then cleanup
```

### 🔹 Advanced trap Patterns

```bash
#!/bin/bash
# advanced_trap_patterns.sh

set -Euo pipefail

# ═══════════════════════════════════════════════════════════════
# PATTERN 1: Trap inheritance in subshells
# ═══════════════════════════════════════════════════════════════

# By default, traps are NOT inherited in subshells
trap 'echo "Parent trap"' EXIT

(
    # This subshell has NO trap
    echo "In subshell"
)   # No "Parent trap" message

# ═══════════════════════════════════════════════════════════════
# PATTERN 2: Temporary trap modification
# ═══════════════════════════════════════════════════════════════

original_cleanup() {
    echo "Original cleanup"
}

enhanced_cleanup() {
    echo "Enhanced cleanup with extra steps"
    original_cleanup
}

# Start with original
trap original_cleanup EXIT

do_critical_work() {
    # Temporarily enhance trap
    trap enhanced_cleanup EXIT
    
    # Do work that needs extra cleanup
    echo "Doing critical work..."
    
    # Restore original trap (or leave enhanced)
}

# ═══════════════════════════════════════════════════════════════
# PATTERN 3: Trap with function context
# ═══════════════════════════════════════════════════════════════

# ERR trap with rich context
trap_with_context() {
    local err=$?
    local line=$1
    local func=${FUNCNAME[1]:-main}
    local script=${BASH_SOURCE[1]:-$0}
    
    cat <<EOF
╔════════════════════════════════════════════════════════════╗
║  ERROR CONTEXT                                              ║
╠════════════════════════════════════════════════════════════╣
║  Exit Code : $err
║  Line      : $line
║  Function  : $func
║  Script    : $script
║  Command   : $BASH_COMMAND
╚════════════════════════════════════════════════════════════╝
EOF
}

trap 'trap_with_context $LINENO' ERR

# ═══════════════════════════════════════════════════════════════
# PATTERN 4: DEBUG trap for tracing
# ═══════════════════════════════════════════════════════════════

enable_tracing() {
    # Called before EVERY command
    trap '
        echo "DEBUG [$(date +%H:%M:%S.%N)] Line $LINENO: $BASH_COMMAND"
    ' DEBUG
}

disable_tracing() {
    trap - DEBUG
}

# Usage:
# enable_tracing
# ... commands to trace ...
# disable_tracing

# ═══════════════════════════════════════════════════════════════
# PATTERN 5: Accumulating cleanup tasks
# ═══════════════════════════════════════════════════════════════

declare -a CLEANUP_TASKS=()

add_cleanup() {
    CLEANUP_TASKS+=("$1")
}

run_cleanup() {
    echo "Running ${#CLEANUP_TASKS[@]} cleanup tasks..."
    
    # Run in reverse order (LIFO)
    for ((i=${#CLEANUP_TASKS[@]}-1; i>=0; i--)); do
        echo "→ ${CLEANUP_TASKS[i]}"
        eval "${CLEANUP_TASKS[i]}"
    done
}

trap run_cleanup EXIT

# Usage:
# add_cleanup "rm -f /tmp/file1"
# add_cleanup "rmdir /tmp/mydir"
# add_cleanup "echo 'Final message'"
```

---

<a name="section-5"></a>
## 5️⃣ DEBUGGING TOOLS & TECHNIQUES

### 🔹 Built-in Bash Debugging

```bash
#!/bin/bash
# debugging_toolkit.sh
# Comprehensive debugging techniques

# ═══════════════════════════════════════════════════════════════
# METHOD 1: Command Line Options
# ═══════════════════════════════════════════════════════════════

# Run entire script with debugging:
# bash -x script.sh        # Print commands before execution
# bash -v script.sh        # Print lines as they're read
# bash -xv script.sh       # Both

# ═══════════════════════════════════════════════════════════════
# METHOD 2: In-Script Debugging Toggles
# ═══════════════════════════════════════════════════════════════

# Enable debugging for specific sections
debug_section() {
    echo "═══ Starting debug section ═══"
    
    set -x  # Turn ON command tracing
    
    # Commands here will be traced
    variable="hello"
    echo "$variable world"
    result=$((5 + 3))
    
    set +x  # Turn OFF command tracing
    
    echo "═══ End debug section ═══"
}

# ═══════════════════════════════════════════════════════════════
# METHOD 3: Custom PS4 Prompt
# ═══════════════════════════════════════════════════════════════

# PS4 is the prompt shown for `set -x` output
# Default: "+ "

# Enhanced PS4 with useful information:
export PS4='+(${BASH_SOURCE}:${LINENO}): ${FUNCNAME[0]:+${FUNCNAME[0]}(): }'

# Example output:
# +(script.sh:15): main(): variable="hello"

# Even more detailed:
export PS4='
+─────────────────────────────────────────────────
| ${BASH_SOURCE##*/}:${LINENO} ${FUNCNAME[0]:-main}()
| → '

# ═══════════════════════════════════════════════════════════════
# METHOD 4: Debug Function with Levels
# ═══════════════════════════════════════════════════════════════

# Debug levels
readonly DEBUG_OFF=0
readonly DEBUG_ERROR=1
readonly DEBUG_WARN=2
readonly DEBUG_INFO=3
readonly DEBUG_VERBOSE=4
readonly DEBUG_TRACE=5

# Current level (change to control output)
DEBUG_LEVEL=${DEBUG_LEVEL:-$DEBUG_INFO}

debug() {
    local level=$1
    shift
    local message="$*"
    
    # Only print if current level is high enough
    if [[ $DEBUG_LEVEL -ge $level ]]; then
        local level_name
        case $level in
            $DEBUG_ERROR)   level_name="ERROR"   ;;
            $DEBUG_WARN)    level_name="WARN"    ;;
            $DEBUG_INFO)    level_name="INFO"    ;;
            $DEBUG_VERBOSE) level_name="VERBOSE" ;;
            $DEBUG_TRACE)   level_name="TRACE"   ;;
        esac
        
        echo "[$(date '+%H:%M:%S')] [$level_name] ${FUNCNAME[1]:-main}:${BASH_LINENO[0]} - $message" >&2
    fi
}

# Convenience functions
error()   { debug $DEBUG_ERROR "$@"; }
warn()    { debug $DEBUG_WARN "$@"; }
info()    { debug $DEBUG_INFO "$@"; }
verbose() { debug $DEBUG_VERBOSE "$@"; }
trace()   { debug $DEBUG_TRACE "$@"; }

# Usage:
# error "Database connection failed"
# info "Processing file: $filename"
# trace "Entering loop iteration $i"

# ═══════════════════════════════════════════════════════════════
# METHOD 5: Breakpoints with read
# ═══════════════════════════════════════════════════════════════

breakpoint() {
    local message="${1:-Breakpoint}"
    
    echo ""
    echo "════════════════════════════════════════════════"
    echo "BREAKPOINT: $message"
    echo "Line: ${BASH_LINENO[0]} in ${FUNCNAME[1]:-main}"
    echo "════════════════════════════════════════════════"
    echo ""
    echo "Current variables:"
    echo "  \$? = ${PIPESTATUS[*]:-N/A}"
    echo ""
    echo "Press Enter to continue, or Ctrl+C to abort..."
    read -r
}

# Usage:
# breakpoint "Before critical operation"
# critical_operation
# breakpoint "After critical operation"

# ═══════════════════════════════════════════════════════════════
# METHOD 6: Variable Inspection
# ═══════════════════════════════════════════════════════════════

dump_var() {
    local var_name=$1
    local var_value
    
    # Get the value using indirect reference
    eval "var_value=\${$var_name:-UNDEFINED}"
    
    echo "┌─────────────────────────────────────────┐"
    echo "│ Variable: $var_name"
    echo "├─────────────────────────────────────────┤"
    
    if [[ "$var_value" == "UNDEFINED" ]]; then
        echo "│ Status: UNDEFINED"
    else
        echo "│ Value: '$var_value'"
        echo "│ Length: ${#var_value}"
        echo "│ Type: $(declare -p "$var_name" 2>/dev/null | cut -d' ' -f2 || echo "string")"
    fi
    
    echo "└─────────────────────────────────────────┘"
}

dump_all_vars() {
    echo "═══════════ ALL VARIABLES ═══════════"
    
    # Script variables (excluding functions)
    compgen -v | while read -r var; do
        case "$var" in
            # Skip internal variables
            BASH*|EUID|FUNCNAME|GROUPS|HOSTNAME|IFS|LINENO|PPID|PS*|PWD|SHELLOPTS|UID)
                continue
                ;;
            *)
                echo "$var = ${!var}"
                ;;
        esac
    done
}

# ═══════════════════════════════════════════════════════════════
# METHOD 7: Execution Time Profiling
# ═══════════════════════════════════════════════════════════════

declare -A TIMER_START

timer_start() {
    local name=$1
    TIMER_START[$name]=$(date +%s.%N)
}

timer_stop() {
    local name=$1
    local end=$(date +%s.%N)
    local start=${TIMER_START[$name]}
    local elapsed=$(echo "$end - $start" | bc)
    
    printf "Timer [%s]: %.4f seconds\n" "$name" "$elapsed"
}

# Usage:
# timer_start "database_query"
# run_database_query
# timer_stop "database_query"
# Output: Timer [database_query]: 0.2345 seconds
```

### 🔹 Using bashdb (Bash Debugger)

```bash
# Installation
sudo apt-get install bashdb

# Usage
bashdb script.sh

# Debugger commands:
#   h (help)     - Show help
#   n (next)     - Execute next line
#   s (step)     - Step into function
#   c (continue) - Continue execution
#   l (list)     - List source code
#   p (print)    - Print variable value
#   b (break)    - Set breakpoint
#   q (quit)     - Quit debugger
```

### 🔹 Using ShellCheck

```bash
# Installation
sudo apt-get install shellcheck

# Usage
shellcheck script.sh

# Common warnings it catches:
# SC2086: Double quote to prevent globbing and word splitting
# SC2046: Quote this to prevent word splitting
# SC2006: Use $(...) instead of `...`
# SC2115: Use "${var:?}" to ensure this never expands to /*

# Integration with editors:
# - VSCode: Install ShellCheck extension
# - Vim: Use ale or syntastic plugins
```

---

<a name="section-6"></a>
## 6️⃣ OPTIMIZATION TECHNIQUES

### 🔹 Error Handling Performance

```bash
#!/bin/bash
# optimized_error_handling.sh

# ═══════════════════════════════════════════════════════════════
# BAD: Checking errors after every command
# ═══════════════════════════════════════════════════════════════

bad_approach() {
    command1
    if [[ $? -ne 0 ]]; then handle_error; fi
    
    command2
    if [[ $? -ne 0 ]]; then handle_error; fi
    
    command3
    if [[ $? -ne 0 ]]; then handle_error; fi
}

# ═══════════════════════════════════════════════════════════════
# GOOD: Use set -e with || for specific handling
# ═══════════════════════════════════════════════════════════════

good_approach() {
    set -e  # Automatic error checking
    
    command1   # Auto-exits on failure
    command2   # Auto-exits on failure
    command3 || { echo "command3 failed, but we continue"; true; }
}

# ═══════════════════════════════════════════════════════════════
# BAD: Complex trap function checking many conditions
# ═══════════════════════════════════════════════════════════════

# This runs on EVERY error - keep it fast!
bad_trap() {
    # DON'T do heavy operations in trap
    log_to_database "Error occurred"  # Network call - slow!
    send_email_notification          # Network call - slow!
    generate_full_report              # CPU intensive - slow!
}

# ═══════════════════════════════════════════════════════════════
# GOOD: Lightweight trap, defer heavy operations
# ═══════════════════════════════════════════════════════════════

declare -g ERROR_OCCURRED=0
declare -g ERROR_INFO=""

lightweight_trap() {
    ERROR_OCCURRED=1
    ERROR_INFO="Line $1: $BASH_COMMAND"
    # Just set flags, don't do heavy work
}

trap 'lightweight_trap $LINENO' ERR

# Do heavy error reporting in cleanup
cleanup() {
    if [[ $ERROR_OCCURRED -eq 1 ]]; then
        # Now do the heavy operations
        log_to_database "$ERROR_INFO"
        send_email_notification
    fi
}
trap cleanup EXIT
```

---

<a name="section-7"></a>
## 7️⃣ COMMON MISTAKES & EDGE CASES

### 🔹 Mistake 1: Ignoring subshell exit codes

```bash
# WRONG: Pipe creates subshell, exit doesn't work as expected
cat file.txt | while read line; do
    if [[ "$line" == "error" ]]; then
        exit 1  # Only exits the subshell!
    fi
done
echo "This still runs!"

# RIGHT: Use process substitution or different approach
while read line; do
    if [[ "$line" == "error" ]]; then
        exit 1  # Exits the main script
    fi
done < file.txt

# OR: Check pipe status
cat file.txt | while read line; do
    [[ "$line" == "error" ]] && exit 1
done
if [[ ${PIPESTATUS[1]} -ne 0 ]]; then
    exit 1
fi
```

### 🔹 Mistake 2: set -e not working as expected

```bash
set -e

# These DON'T trigger exit:
false || true        # Explicit error handling
false && true        # Part of && chain
if false; then       # Condition context
    echo "yes"
fi
while false; do      # Loop condition
    echo "loop"
done
! false              # Negation

# This DOES trigger exit:
false
echo "Never reached"
```

### 🔹 Mistake 3: Unquoted variables in cleanup

```bash
# DANGEROUS: If filename has spaces or is empty
cleanup() {
    rm -f $TEMP_FILE   # Word splitting can cause issues
}

# SAFE: Always quote
cleanup() {
    rm -f "$TEMP_FILE"
    
    # Even safer: Check first
    [[ -n "$TEMP_FILE" && -f "$TEMP_FILE" ]] && rm -f "$TEMP_FILE"
}
```

### 🔹 Mistake 4: Trap not handling all signals

```bash
# INCOMPLETE: Only handles EXIT
trap cleanup EXIT

# COMPLETE: Handle all common signals
trap cleanup EXIT
trap 'trap - EXIT; cleanup; exit 130' INT
trap 'trap - EXIT; cleanup; exit 143' TERM

# OR: Single handler
trap 'cleanup' EXIT INT TERM HUP
```

---

<a name="section-8"></a>
## 8️⃣ MINI CHALLENGES

### Challenge 1: Implement Retry Logic

```bash
# Create a function that:
# - Attempts a command up to N times
# - Waits between attempts (exponential backoff)
# - Returns success/failure appropriately

# Your solution here:
retry() {
    # TODO: Implement
    :
}

# Test with:
# retry 3 curl https://flaky-server.example.com/api
```

<details>
<summary>📝 Solution</summary>

```bash
#!/bin/bash
# retry_solution.sh

retry() {
    local max_attempts=$1
    local delay=1
    shift
    local command="$@"
    
    local attempt=1
    
    while [[ $attempt -le $max_attempts ]]; do
        echo "Attempt $attempt/$max_attempts: $command"
        
        if eval "$command"; then
            echo "✓ Success on attempt $attempt"
            return 0
        fi
        
        if [[ $attempt -lt $max_attempts ]]; then
            echo "✗ Failed, waiting ${delay}s before retry..."
            sleep $delay
            delay=$((delay * 2))  # Exponential backoff
        fi
        
        ((attempt++))
    done
    
    echo "✗ All $max_attempts attempts failed"
    return 1
}

# Test
retry 3 ls /nonexistent/path
echo "Exit code: $?"
```
</details>

### Challenge 2: Create Rollback Mechanism

```bash
# Create a system that:
# - Tracks operations performed
# - Can rollback all operations on failure
# - Maintains order (last operation rolled back first)

# Your solution here:
```

<details>
<summary>📝 Solution</summary>

```bash
#!/bin/bash
# rollback_solution.sh

set -euo pipefail

# Stack for rollback operations
declare -a ROLLBACK_STACK=()

# Add rollback operation
push_rollback() {
    ROLLBACK_STACK+=("$1")
    echo "Registered rollback: $1"
}

# Execute all rollbacks
do_rollback() {
    echo ""
    echo "════════════════════════════════════════"
    echo "        INITIATING ROLLBACK             "
    echo "════════════════════════════════════════"
    
    # Process in reverse order
    for ((i=${#ROLLBACK_STACK[@]}-1; i>=0; i--)); do
        echo "→ Rolling back: ${ROLLBACK_STACK[i]}"
        eval "${ROLLBACK_STACK[i]}" || echo "  Warning: Rollback failed"
    done
    
    echo "════════════════════════════════════════"
}

# Trap for automatic rollback on error
trap do_rollback ERR

# Example usage
main() {
    echo "Starting deployment..."
    
    # Step 1: Create directory
    mkdir -p /tmp/deploy_test
    push_rollback "rmdir /tmp/deploy_test"
    
    # Step 2: Create config file
    echo "config=value" > /tmp/deploy_test/config.txt
    push_rollback "rm -f /tmp/deploy_test/config.txt"
    
    # Step 3: This fails
    echo "Creating critical file..."
    touch /root/cannot_create_here 2>/dev/null || false
    push_rollback "rm -f /root/cannot_create_here"
    
    # Step 4: Never reached
    echo "This never runs"
}

main
```
</details>

### Challenge 3: Debug a Mystery Error

```bash
# This script fails intermittently. Add debugging to find why.

#!/bin/bash

process_files() {
    for file in /tmp/process_*; do
        content=$(cat "$file")
        result=$((content * 2))
        echo "$result" > "${file}.result"
    done
}

main() {
    process_files
    echo "All files processed"
}

main
```

<details>
<summary>📝 Solution & Explanation</summary>

```bash
#!/bin/bash
# debug_mystery_solution.sh

set -euo pipefail

process_files() {
    echo "[DEBUG] Looking for files matching /tmp/process_*"
    
    # Bug 1: Glob might not match any files
    local files=(/tmp/process_*)
    
    if [[ ! -e "${files[0]}" ]]; then
        echo "[DEBUG] No matching files found!"
        return 0  # Or return 1 if this is an error
    fi
    
    echo "[DEBUG] Found ${#files[@]} files"
    
    for file in "${files[@]}"; do
        echo "[DEBUG] Processing: $file"
        
        # Bug 2: File might not contain a number
        if ! content=$(cat "$file"); then
            echo "[DEBUG] Failed to read $file"
            continue
        fi
        
        echo "[DEBUG] Content: '$content'"
        
        # Bug 3: Content might not be numeric
        if ! [[ "$content" =~ ^[0-9]+$ ]]; then
            echo "[DEBUG] Content is not numeric: '$content'"
            continue
        fi
        
        result=$((content * 2))
        echo "[DEBUG] Result: $result"
        
        echo "$result" > "${file}.result"
    done
}

main() {
    process_files
    echo "All files processed"
}

main
```

**Bugs found:**
1. Glob `/tmp/process_*` expands literally if no matches (with `set -u`)
2. `cat` fails if file doesn't exist or isn't readable
3. `$((content * 2))` fails if content isn't a number
</details>

---

<a name="section-9"></a>
## 9️⃣ FINAL TEST: FIX BROKEN SCRIPTS

### Test Script 1: Fix the Broken Backup Script

```bash
#!/bin/bash
# broken_backup.sh - FIX THIS SCRIPT

# This script has multiple bugs. Find and fix them all.

backup_dir=/tmp/backups
source_dir=$1

mkdir $backup_dir
cp -r $source_dir/* $backup_dir/
tar -czf backup.tar.gz $backup_dir
rm -rf $backup_dir

echo "Backup complete: backup.tar.gz"
```

<details>
<summary>📝 Fixed Version & Explanation</summary>

```bash
#!/bin/bash
# fixed_backup.sh - FIXED VERSION

set -euo pipefail

# ═══════════════════════════════════════════════════════════════
# CONFIGURATION
# ═══════════════════════════════════════════════════════════════

readonly SCRIPT_NAME=$(basename "$0")
readonly BACKUP_DIR=$(mktemp -d -t "backup_XXXXXX")
readonly TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# ═══════════════════════════════════════════════════════════════
# CLEANUP TRAP
# ═══════════════════════════════════════════════════════════════

cleanup() {
    local exit_code=$?
    
    # Bug 4 fix: Always clean up temp directory
    if [[ -d "$BACKUP_DIR" ]]; then
        rm -rf "$BACKUP_DIR"
    fi
    
    exit $exit_code
}

trap cleanup EXIT

# ═══════════════════════════════════════════════════════════════
# VALIDATION
# ═══════════════════════════════════════════════════════════════

# Bug 1 fix: Check if argument provided
if [[ $# -eq 0 ]]; then
    echo "Usage: $SCRIPT_NAME <source_directory>" >&2
    exit 1
fi

source_dir="$1"  # Bug 2 fix: Quote variable

# Bug 3 fix: Validate source directory
if [[ ! -d "$source_dir" ]]; then
    echo "Error: Source directory does not exist: $source_dir" >&2
    exit 1
fi

# ═══════════════════════════════════════════════════════════════
# MAIN LOGIC
# ═══════════════════════════════════════════════════════════════

# Bug 5 fix: Use safe filename
backup_file="backup_${TIMESTAMP}.tar.gz"

echo "Creating backup of $source_dir..."

# Bug 6 fix: Quote all variables
cp -r "$source_dir"/* "$BACKUP_DIR/" 2>/dev/null || {
    # Handle empty directory
    echo "Warning: Source directory appears to be empty"
}

# Bug 7 fix: Specify output path
tar -czf "$backup_file" -C "$(dirname "$BACKUP_DIR")" "$(basename "$BACKUP_DIR")"

echo "✓ Backup complete: $backup_file"
echo "  Size: $(du -h "$backup_file" | cut -f1)"
```

**Bugs Fixed:**
1. No argument validation
2. Unquoted variables (word splitting risk)
3. No check if source directory exists
4. No cleanup on error
5. Hardcoded backup filename (overwrites)
6. No set -e (errors ignored)
7. rm -rf before verifying tar succeeded
</details>

### Test Script 2: Fix the Broken Service Checker

```bash
#!/bin/bash
# broken_service_checker.sh - FIX THIS SCRIPT

services="nginx mysql redis"

for service in $services
do
    status=`systemctl status $service`
    if [ $? = 0 ]
    then
        echo "$service is running"
    else
        systemctl start $service
        echo "Started $service"
    fi
done

echo Done checking services
```

<details>
<summary>📝 Fixed Version & Explanation</summary>

```bash
#!/bin/bash
# fixed_service_checker.sh - FIXED VERSION

set -uo pipefail
# Note: NOT using -e because we expect some commands to fail

# ═══════════════════════════════════════════════════════════════
# CONFIGURATION
# ═══════════════════════════════════════════════════════════════

readonly SCRIPT_NAME=$(basename "$0")
readonly LOG_FILE="/var/log/service_checker.log"

# Bug 1 fix: Use array instead of string
declare -a SERVICES=("nginx" "mysql" "redis")

# ═══════════════════════════════════════════════════════════════
# FUNCTIONS
# ═══════════════════════════════════════════════════════════════

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

check_root() {
    # Bug 2 fix: Check for root privileges (needed for systemctl start)
    if [[ $EUID -ne 0 ]]; then
        echo "Error: This script must be run as root" >&2
        exit 1
    fi
}

check_service() {
    local service="$1"
    
    # Bug 3 fix: Use is-active instead of status (cleaner)
    if systemctl is-active --quiet "$service"; then
        return 0
    else
        return 1
    fi
}

start_service() {
    local service="$1"
    
    # Bug 4 fix: Proper error handling for start
    if systemctl start "$service" 2>/dev/null; then
        log "✓ Started $service"
        return 0
    else
        log "✗ Failed to start $service"
        return 1
    fi
}

# ═══════════════════════════════════════════════════════════════
# MAIN LOGIC
# ═══════════════════════════════════════════════════════════════

main() {
    check_root
    
    local failed_services=()
    
    log "Starting service check..."
    
    # Bug 5 fix: Proper array iteration
    for service in "${SERVICES[@]}"; do
        log "Checking: $service"
        
        # Bug 6 fix: Check if service exists
        if ! systemctl list-unit-files | grep -q "^${service}.service"; then
            log "⚠ Service not found: $service"
            continue
        fi
        
        if check_service "$service"; then
            log "✓ $service is running"
        else
            log "✗ $service is not running, attempting start..."
            if ! start_service "$service"; then
                failed_services+=("$service")
            fi
        fi
    done
    
    # Bug 7 fix: Report summary
    log ""
    log "═══════════════════════════════════════"
    if [[ ${#failed_services[@]} -eq 0 ]]; then
        log "All services are running"
        exit 0
    else
        log "Failed services: ${failed_services[*]}"
        exit 1
    fi
}

main "$@"
```

**Bugs Fixed:**
1. Used string instead of array for services
2. No root check (systemctl start needs root)
3. Used backticks instead of `$(...)` 
4. Used `[ $? = 0 ]` instead of `[[ $? -eq 0 ]]`
5. No error handling for service start
6. No check if service exists
7. No summary or exit code indication
</details>

---

<a name="section-10"></a>
## 🔟 CHEAT SHEET

```
╔═══════════════════════════════════════════════════════════════════╗
║                 ERROR HANDLING CHEAT SHEET                        ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║  STRICT MODE HEADER:                                              ║
║  ──────────────────                                               ║
║  #!/bin/bash                                                      ║
║  set -euo pipefail                                                ║
║  IFS=$'\n\t'                                                      ║
║                                                                   ║
║  EXIT CODES:                                                      ║
║  ───────────                                                      ║
║  0   = Success         126 = Not executable                       ║
║  1   = General error   127 = Command not found                    ║
║  2   = Misuse          130 = Ctrl+C (128+2)                       ║
║                        143 = SIGTERM (128+15)                     ║
║                                                                   ║
║  SET OPTIONS:                                                     ║
║  ────────────                                                     ║
║  -e  Exit on error          +e  Disable                           ║
║  -u  Error on undefined     +u  Disable                           ║
║  -x  Debug/trace mode       +x  Disable                           ║
║  -o pipefail                +o pipefail                           ║
║  -E  ERR trap inheritance                                         ║
║                                                                   ║
║  TRAP SIGNALS:                                                    ║
║  ─────────────                                                    ║
║  trap 'cmd' EXIT      # Always runs on exit                       ║
║  trap 'cmd' ERR       # On command failure                        ║
║  trap 'cmd' INT       # On Ctrl+C                                 ║
║  trap 'cmd' TERM      # On kill                                   ║
║  trap 'cmd' DEBUG     # Before each command                       ║
║  trap - SIGNAL        # Remove trap                               ║
║                                                                   ║
║  DEBUGGING:                                                       ║
║  ──────────                                                       ║
║  bash -x script.sh    # Trace mode                                ║
║  bash -n script.sh    # Syntax check only                         ║
║  shellcheck script.sh # Static analysis                           ║
║  bashdb script.sh     # Interactive debugger                      ║
║                                                                   ║
║  SPECIAL VARIABLES:                                               ║
║  ──────────────────                                               ║
║  $?              Last exit code                                   ║
║  ${PIPESTATUS[@]} All pipe exit codes                             ║
║  $LINENO         Current line number                              ║
║  $FUNCNAME       Current function name                            ║
║  $BASH_SOURCE    Script filename                                  ║
║  $BASH_COMMAND   Current command                                  ║
║                                                                   ║
║  PATTERNS:                                                        ║
║  ─────────                                                        ║
║  cmd || die "msg"           # Exit on failure                     ║
║  cmd || { cleanup; exit; }  # Cleanup and exit                    ║
║  cmd || true                # Ignore failure                      ║
║  if ! cmd; then ... fi      # Check failure                       ║
║                                                                   ║
╚═══════════════════════════════════════════════════════════════════╝
```

---

<a name="section-11"></a>
## 📅 TIME-BASED LEARNING PLAN

```
┌─────────────────────────────────────────────────────────────────────┐
│              60-MINUTE ERROR HANDLING MASTERY PLAN                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  MINUTES 0-10: Foundations                                          │
│  ─────────────────────────────                                      │
│  □ Understand exit codes ($?)                                       │
│  □ Learn set -e, -u, -o pipefail                                    │
│  □ Write strict mode header from memory                             │
│  ✓ Checkpoint: Explain why each option matters                      │
│                                                                     │
│  MINUTES 10-25: trap Mastery                                        │
│  ────────────────────────────                                       │
│  □ Learn trap syntax for EXIT, ERR, INT, TERM                       │
│  □ Implement cleanup function                                       │
│  □ Test Ctrl+C handling                                             │
│  ✓ Checkpoint: Script that cleans temp files on any exit            │
│                                                                     │
│  MINUTES 25-40: Debugging Tools                                     │
│  ────────────────────────────                                       │
│  □ Practice set -x debugging                                        │
│  □ Learn PS4 customization                                          │
│  □ Use LINENO and FUNCNAME                                          │
│  □ Create debug logging function                                    │
│  ✓ Checkpoint: Add debugging to existing script                     │
│                                                                     │
│  MINUTES 40-50: Advanced Patterns                                   │
│  ─────────────────────────────                                      │
│  □ Stack traces with caller                                         │
│  □ Retry mechanism                                                  │
│  □ Rollback pattern                                                 │
│  ✓ Checkpoint: Implement retry function                             │
│                                                                     │
│  MINUTES 50-60: Final Test                                          │
│  ───────────────────────────                                        │
│  □ Fix broken script 1                                              │
│  □ Fix broken script 2                                              │
│  □ Review cheat sheet                                               │
│  ✓ Final: Both scripts working without errors                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---
