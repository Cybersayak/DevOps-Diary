# Advanced Scripting Fundamentals - Complete Mastery Guide

## Table of Contents
1. [Parameter Expansion Mastery](#parameter-expansion)
2. [Array Manipulation Deep Dive](#arrays)
3. [Modular Functions](#functions)
4. [Mini Challenges](#challenges)
5. [Final Test Script](#final-test)
6. [Learning Plan](#learning-plan)

---

## 1. Parameter Expansion Mastery <a name="parameter-expansion"></a>

### 1.1 Conceptual Foundation

**What is Parameter Expansion?**
Parameter expansion is Bash's way of manipulating variable values directly within the shell, without calling external commands like `sed`, `awk`, or `cut`. This makes scripts faster and more portable.

**Why Use It?**
- **Performance**: No subprocess creation (10-100x faster than external commands)
- **Portability**: Pure Bash, works everywhere
- **Readability**: Once learned, cleaner than piping through multiple commands
- **Memory Efficient**: No string copying between processes

---

### 1.2 Basic Parameter Expansion

#### Simple Variable Reference
```bash
#!/bin/bash

# Basic expansion
name="DevOps"
echo $name          # Output: DevOps
echo ${name}        # Output: DevOps (preferred for clarity)

# When braces are REQUIRED
echo "$nameEngineer"     # Wrong: looks for variable 'nameEngineer'
echo "${name}Engineer"   # Correct: DevOpsEngineer
```

**Why use braces?**
- Disambiguate variable names from surrounding text
- Required for advanced operations
- Professional standard

---

### 1.3 Pattern Removal Operators

#### ${var#pattern} - Remove Shortest Match from Beginning

```bash
#!/bin/bash

# Concept: '#' removes from the LEFT (think: # is on left of keyboard)
filepath="/home/user/documents/report.txt"

# Remove shortest match from beginning
echo ${filepath#*/}
# Output: home/user/documents/report.txt
# Pattern matched: "/", stopped at first match

# Real-world example: Extract filename
url="https://example.com/files/document.pdf"
echo ${url#*/}      # Output: /example.com/files/document.pdf
```

**Deep Syntax Breakdown:**
```
${variable#pattern}
  │        │ └─ Glob pattern to match
  │        └─ Operator: # (remove from start, shortest)
  └─ Variable name
```

**Pattern Matching Rules:**
- Uses glob patterns (*, ?, [...])
- NOT regular expressions
- Greedy from left, stops at shortest match

---

#### ${var##pattern} - Remove Longest Match from Beginning

```bash
#!/bin/bash

# Concept: '##' removes LONGEST match from LEFT
filepath="/home/user/documents/report.txt"

# Remove longest match
echo ${filepath##*/}
# Output: report.txt
# Pattern matched: "/home/user/documents/", stopped at last '/'

# Real-world example: Extract filename from path
full_path="/var/log/nginx/access.log"
filename=${full_path##*/}
echo "Processing: $filename"  # Output: Processing: access.log

# Extract base domain
url="https://api.staging.example.com/v1/users"
domain=${url##*/}
echo $domain  # Output: users (last segment)
```

**Common Patterns:**
```bash
# Extract filename
${path##*/}

# Remove all directories
${fullpath##*/}

# Remove protocol from URL
url="https://example.com"
${url##*://}  # Output: example.com
```

---

#### ${var%pattern} - Remove Shortest Match from End

```bash
#!/bin/bash

# Concept: '%' removes from RIGHT (think: % is on right of keyboard)
filename="report.txt.bak"

# Remove shortest match from end
echo ${filename%.*}
# Output: report.txt
# Pattern matched: ".bak"

# Real-world example: Remove file extension
script_name="deploy_app.sh"
base_name=${script_name%.*}
echo "Log file: ${base_name}.log"  # Output: Log file: deploy_app.log

# Strip trailing slashes
path="/home/user/"
clean_path=${path%/}
echo "$clean_path"  # Output: /home/user
```

**Practical Use Cases:**
```bash
#!/bin/bash

# Batch rename files (remove .old extension)
for file in *.txt.old; do
    mv "$file" "${file%.old}"
done

# Create backup with different extension
original="config.yaml"
backup="${original%.*}.backup"
cp "$original" "$backup"  # config.backup
```

---

#### ${var%%pattern} - Remove Longest Match from End

```bash
#!/bin/bash

# Concept: '%%' removes LONGEST match from RIGHT
filename="archive.tar.gz"

# Remove longest match from end
echo ${filename%%.*}
# Output: archive
# Pattern matched: ".tar.gz" (everything after first dot)

# Real-world example: Complete extension removal
file="document.backup.2024.tar.gz"
echo ${file%%.*}  # Output: document

# Extract domain from email
email="admin@mail.example.com"
username=${email%%@*}
domain=${email##*@}
echo "User: $username, Domain: $domain"
# Output: User: admin, Domain: mail.example.com
```

---

### 1.4 Visual Comparison Table

| Operator | Direction | Match Length | Example | Result |
|----------|-----------|--------------|---------|--------|
| `#` | From start → | Shortest | `${path#*/}` | Remove first dir |
| `##` | From start → | Longest | `${path##*/}` | Get filename |
| `%` | ← From end | Shortest | `${file%.*}` | Remove last ext |
| `%%` | ← From end | Longest | `${file%%.*}` | Remove all ext |

---

### 1.5 Advanced Pattern Techniques

#### String Replacement

```bash
#!/bin/bash

# ${var/pattern/replacement} - Replace FIRST occurrence
text="The cloud is in the cloud"
echo ${text/cloud/server}
# Output: The server is in the cloud

# ${var//pattern/replacement} - Replace ALL occurrences
echo ${text//cloud/server}
# Output: The server is in the server

# Real-world: Sanitize filenames
filename="My Document (2024).txt"
safe_name=${filename// /_}        # Replace spaces
safe_name=${safe_name//[()]/}     # Remove parentheses
echo "$safe_name"  # Output: My_Document_2024.txt
```

#### Case Modification

```bash
#!/bin/bash

# Convert to uppercase
name="john doe"
echo ${name^^}        # Output: JOHN DOE
echo ${name^^[jd]}    # Output: John Doe (only j and d)

# Convert to lowercase
CITY="NEW YORK"
echo ${CITY,,}        # Output: new york
echo ${CITY,,[NY]}    # Output: new yORK

# Capitalize first letter
word="docker"
echo ${word^}         # Output: Docker

# Real-world: Normalize environment variables
env_input="PrOdUcTiOn"
env_normalized=${env_input,,}
if [[ $env_normalized == "production" ]]; then
    echo "Production deployment initiated"
fi
```

---

### 1.6 Substring Extraction

```bash
#!/bin/bash

# ${var:offset:length}
text="DevOpsEngineer"

echo ${text:0:6}      # Output: DevOps (start at 0, take 6)
echo ${text:6}        # Output: Engineer (from position 6 to end)
echo ${text:(-8)}     # Output: Engineer (last 8 characters)
echo ${text:(-8):3}   # Output: Eng

# Real-world: Parse log timestamps
log_entry="2024-01-15 10:30:45 ERROR Database connection failed"
date=${log_entry:0:10}
time=${log_entry:11:8}
level=${log_entry:20:5}
echo "Date: $date, Time: $time, Level: $level"
# Output: Date: 2024-01-15, Time: 10:30:45, Level: ERROR
```

---

### 1.7 Default Values and Error Handling

```bash
#!/bin/bash

# ${var:-default} - Use default if unset or empty
echo ${username:-"anonymous"}  # Output: anonymous

username="alice"
echo ${username:-"anonymous"}  # Output: alice

# ${var:=default} - Assign default if unset
echo ${database:="localhost"}  # Sets AND returns
echo $database                 # Output: localhost (variable is now set)

# ${var:?error_message} - Exit with error if unset
process_file() {
    local filepath=${1:?"Error: Filepath required"}
    echo "Processing: $filepath"
}

# ${var:+alternative} - Use alternative if SET
debug_mode="true"
echo "Running in ${debug_mode:+debug} mode"
# Output: Running in debug mode
```

**Real-World Example: Configuration Management**

```bash
#!/bin/bash

# config_loader.sh - Safe configuration with defaults

load_config() {
    # Required variables (will exit if not set)
    APP_NAME=${APP_NAME:?"ERROR: APP_NAME must be set"}
    
    # Optional with defaults
    APP_PORT=${APP_PORT:-8080}
    APP_HOST=${APP_HOST:-"0.0.0.0"}
    LOG_LEVEL=${LOG_LEVEL:-"INFO"}
    
    # Environment-specific overrides
    if [[ ${ENVIRONMENT:="development"} == "production" ]]; then
        LOG_LEVEL=${LOG_LEVEL_PROD:-"ERROR"}
        DEBUG_MODE=""
    else
        DEBUG_MODE=${DEBUG_MODE:+"--debug"}
    fi
    
    echo "Configuration loaded:"
    echo "  App: $APP_NAME"
    echo "  Host: $APP_HOST:$APP_PORT"
    echo "  Log Level: $LOG_LEVEL"
    echo "  Debug: ${DEBUG_MODE:-disabled}"
}

# Usage
APP_NAME="UserService"
load_config
```

**Output:**
```
Configuration loaded:
  App: UserService
  Host: 0.0.0.0:8080
  Log Level: INFO
  Debug: disabled
```

---

### 1.8 Length and Indirect Expansion

```bash
#!/bin/bash

# ${#var} - Length of variable
password="Secr3t!"
if [[ ${#password} -lt 8 ]]; then
    echo "Password too short (${#password} characters)"
fi

# ${!prefix*} - List variables starting with prefix
export AWS_REGION="us-east-1"
export AWS_BUCKET="my-bucket"
export AWS_KEY="mykey"

echo "AWS variables:"
for var in ${!AWS*}; do
    echo "  $var = ${!var}"
done

# Output:
# AWS variables:
#   AWS_BUCKET = my-bucket
#   AWS_KEY = mykey
#   AWS_REGION = us-east-1
```

---

### 1.9 Practical Performance Comparison

```bash
#!/bin/bash

# performance_test.sh

test_string="https://cdn.example.com/images/logo.png"

# Method 1: Parameter expansion (FAST)
time_start=$SECONDS
for i in {1..10000}; do
    filename=${test_string##*/}
done
time_param=$((SECONDS - time_start))

# Method 2: basename command (SLOW)
time_start=$SECONDS
for i in {1..10000}; do
    filename=$(basename "$test_string")
done
time_base=$((SECONDS - time_start))

echo "Parameter expansion: ${time_param}s"
echo "Basename command: ${time_base}s"
echo "Speed increase: $((time_base / time_param))x faster"
```

**Typical Output:**
```
Parameter expansion: 1s
Basename command: 45s
Speed increase: 45x faster
```

---

### 1.10 Common Mistakes and Solutions

#### Mistake 1: Forgetting Quotes

```bash
#!/bin/bash

filename="my document.txt"

# WRONG: Word splitting occurs
base=${filename%.*}
echo $base           # Might work, but dangerous
rm $base.bak         # ERROR: tries to remove "my" and "document.bak"

# CORRECT: Always quote
echo "$base"
rm "${base}.bak"
```

#### Mistake 2: Wrong Pattern Type

```bash
#!/bin/bash

# WRONG: Using regex instead of glob
email="user@example.com"
domain=${email##.*@}  # Doesn't work - .* is NOT regex

# CORRECT: Use glob patterns
domain=${email##*@}   # Works - * is glob
```

#### Mistake 3: Modifying Read-Only Variables

```bash
#!/bin/bash

# WRONG: Can't expand special variables
echo ${1#prefix}     # Works, but $1 is NOT modified

# CORRECT: Assign to new variable
arg1_clean=${1#prefix}
```

---

### 1.11 Parameter Expansion Cheat Sheet

```bash
# ============================
# PATTERN REMOVAL
# ============================
${var#pattern}    # Remove shortest from start
${var##pattern}   # Remove longest from start
${var%pattern}    # Remove shortest from end
${var%%pattern}   # Remove longest from end

# Common patterns:
${path##*/}       # Filename from path
${path%/*}        # Directory from path
${file%.*}        # Remove extension
${file%%.*}       # Remove all extensions

# ============================
# SUBSTITUTION
# ============================
${var/pat/rep}    # Replace first occurrence
${var//pat/rep}   # Replace all occurrences
${var/#pat/rep}   # Replace at beginning
${var/%pat/rep}   # Replace at end

# ============================
# CASE MODIFICATION
# ============================
${var^^}          # All uppercase
${var,,}          # All lowercase
${var^}           # Capitalize first
${var~}           # Toggle case of first

# ============================
# SUBSTRING
# ============================
${var:offset}         # From offset to end
${var:offset:length}  # Substring
${var:(-n)}          # Last n characters

# ============================
# DEFAULTS
# ============================
${var:-default}   # Use default if unset
${var:=default}   # Assign default if unset
${var:?message}   # Error if unset
${var:+value}     # Use value if set

# ============================
# LENGTH
# ============================
${#var}           # Length of string
${#array[@]}      # Array length
```

---

## 2. Array Manipulation Deep Dive <a name="arrays"></a>

### 2.1 Conceptual Foundation

**What are Arrays?**
Arrays are ordered collections of values, accessible by numeric (indexed) or string (associative) keys.

**Why Use Arrays?**
- Store multiple related values in one variable
- Iterate over collections efficiently
- Build complex data structures
- Process batch operations

**Two Types in Bash:**
1. **Indexed Arrays**: Numeric keys (0, 1, 2...)
2. **Associative Arrays**: String keys (like hash maps/dictionaries)

---

### 2.2 Indexed Arrays - Basics

#### Declaration Methods

```bash
#!/bin/bash

# Method 1: Direct assignment
servers[0]="web1.example.com"
servers[1]="web2.example.com"
servers[2]="web3.example.com"

# Method 2: Parentheses syntax (PREFERRED)
servers=("web1.example.com" "web2.example.com" "web3.example.com")

# Method 3: Explicit declaration
declare -a servers=("web1" "web2" "web3")

# Method 4: Empty array
declare -a logs=()

# Method 5: Sparse array (gaps allowed)
nodes[0]="primary"
nodes[5]="backup"  # indices 1-4 are empty
```

---

#### Accessing Elements

```bash
#!/bin/bash

fruits=("apple" "banana" "cherry" "date")

# Single element
echo ${fruits[0]}      # Output: apple
echo ${fruits[2]}      # Output: cherry

# Last element
echo ${fruits[-1]}     # Output: date
echo ${fruits[-2]}     # Output: cherry

# ALL elements
echo ${fruits[@]}      # Output: apple banana cherry date
echo ${fruits[*]}      # Output: apple banana cherry date

# Difference between @ and * (IMPORTANT)
fruits_with_spaces=("red apple" "yellow banana")

# WRONG: Word splitting
for fruit in ${fruits_with_spaces[*]}; do
    echo "- $fruit"
done
# Output:
# - red
# - apple
# - yellow
# - banana

# CORRECT: Preserves elements
for fruit in "${fruits_with_spaces[@]}"; do
    echo "- $fruit"
done
# Output:
# - red apple
# - yellow banana
```

**Rule: Always use `"${array[@]}"` with quotes and `@`**

---

#### Array Properties

```bash
#!/bin/bash

servers=("web1" "web2" "db1" "cache1")

# Length
echo ${#servers[@]}    # Output: 4

# Indices
echo ${!servers[@]}    # Output: 0 1 2 3

# Length of specific element
echo ${#servers[2]}    # Output: 3 (length of "db1")

# Check if index exists
if [[ -v servers[2] ]]; then
    echo "Index 2 exists: ${servers[2]}"
fi
```

---

### 2.3 Indexed Arrays - Advanced Operations

#### Adding Elements

```bash
#!/bin/bash

# Append to end
logs=("error.log" "access.log")
logs+=("debug.log")
echo "${logs[@]}"  # Output: error.log access.log debug.log

# Prepend (inefficient but possible)
new_log="startup.log"
logs=("$new_log" "${logs[@]}")

# Insert at specific position (requires recreation)
logs=("error.log" "access.log" "system.log")
# Insert "security.log" at index 1
logs=("${logs[0]}" "security.log" "${logs[@]:1}")
echo "${logs[@]}"
# Output: error.log security.log access.log system.log
```

---

#### Removing Elements

```bash
#!/bin/bash

servers=("web1" "web2" "db1" "cache1" "web3")

# Remove by index
unset servers[2]
echo "${servers[@]}"   # Output: web1 web2 cache1 web3
echo "${!servers[@]}"  # Output: 0 1 3 4 (index 2 is missing)

# Remove by value
remove_element() {
    local array_name=$1
    local value=$2
    local temp_array=()
    
    # Create reference to array
    declare -n arr=$array_name
    
    for item in "${arr[@]}"; do
        [[ $item != "$value" ]] && temp_array+=("$item")
    done
    
    arr=("${temp_array[@]}")
}

# Usage
servers=("web1" "web2" "db1" "web2")
remove_element servers "web2"
echo "${servers[@]}"  # Output: web1 db1
```

---

#### Slicing Arrays

```bash
#!/bin/bash

numbers=(10 20 30 40 50 60 70 80 90)

# ${array[@]:start:length}
echo "${numbers[@]:2:3}"    # Output: 30 40 50 (from index 2, take 3)
echo "${numbers[@]:5}"      # Output: 60 70 80 90 (from index 5 to end)
echo "${numbers[@]:(-3)}"   # Output: 70 80 90 (last 3 elements)

# Real-world: Process in batches
process_batch() {
    local batch=("$@")
    echo "Processing batch of ${#batch[@]} items:"
    printf '  - %s\n' "${batch[@]}"
}

# Process in batches of 3
batch_size=3
for ((i=0; i<${#numbers[@]}; i+=batch_size)); do
    batch=("${numbers[@]:i:batch_size}")
    process_batch "${batch[@]}"
done
```

**Output:**
```
Processing batch of 3 items:
  - 10
  - 20
  - 30
Processing batch of 3 items:
  - 40
  - 50
  - 60
Processing batch of 3 items:
  - 70
  - 80
  - 90
```

---

#### Sorting and Transforming

```bash
#!/bin/bash

# Sort array
servers=("web3" "db1" "web1" "cache2" "web2")

# Method 1: Using process substitution
IFS=$'\n' sorted=($(sort <<<"${servers[*]}"))
unset IFS
echo "${sorted[@]}"  # Output: cache2 db1 web1 web2 web3

# Method 2: Using mapfile (Bash 4+)
mapfile -t sorted < <(printf '%s\n' "${servers[@]}" | sort)
echo "${sorted[@]}"

# Reverse sort
mapfile -t sorted < <(printf '%s\n' "${servers[@]}" | sort -r)

# Sort numerically
numbers=(100 20 3 45 7)
mapfile -t sorted_nums < <(printf '%s\n' "${numbers[@]}" | sort -n)
echo "${sorted_nums[@]}"  # Output: 3 7 20 45 100
```

---

#### Array Iteration Patterns

```bash
#!/bin/bash

servers=("web1" "web2" "db1")

# Pattern 1: Iterate over values
for server in "${servers[@]}"; do
    echo "Connecting to $server"
done

# Pattern 2: Iterate with indices
for i in "${!servers[@]}"; do
    echo "Server $((i+1)): ${servers[i]}"
done

# Pattern 3: C-style loop
for ((i=0; i<${#servers[@]}; i++)); do
    echo "Position $i: ${servers[i]}"
done

# Pattern 4: While loop with array
i=0
while [[ $i -lt ${#servers[@]} ]]; do
    echo "${servers[i]}"
    ((i++))
done

# Pattern 5: Parallel processing simulation
process_server() {
    echo "Processing $1"
    sleep 1
}

for server in "${servers[@]}"; do
    process_server "$server" &  # Background
done
wait  # Wait for all background jobs
echo "All servers processed"
```

---

### 2.4 Associative Arrays (Hash Maps)

#### Declaration and Basic Usage

```bash
#!/bin/bash

# MUST declare before use
declare -A server_ips

# Assignment
server_ips["web1"]="192.168.1.10"
server_ips["web2"]="192.168.1.11"
server_ips["db1"]="192.168.1.20"

# OR: Initialize at declaration
declare -A ports=(
    ["http"]=80
    ["https"]=443
    ["ssh"]=22
    ["mysql"]=3306
)

# Access
echo ${server_ips["web1"]}      # Output: 192.168.1.10
echo ${ports["https"]}          # Output: 443

# Get all keys
echo "${!server_ips[@]}"        # Output: web1 web2 db1

# Get all values
echo "${server_ips[@]}"         # Output: 192.168.1.10 192.168.1.11 192.168.1.20

# Check if key exists
if [[ -v server_ips["web1"] ]]; then
    echo "web1 exists with IP: ${server_ips["web1"]}"
fi
```

---

#### Real-World Example: Configuration Management

```bash
#!/bin/bash

# config_manager.sh - Manage application configurations

declare -A app_config

load_config() {
    # Load from file
    while IFS='=' read -r key value; do
        # Skip comments and empty lines
        [[ $key =~ ^#.*$ ]] && continue
        [[ -z $key ]] && continue
        
        # Trim whitespace
        key=$(echo "$key" | xargs)
        value=$(echo "$value" | xargs)
        
        app_config["$key"]="$value"
    done < config.txt
}

get_config() {
    local key=$1
    local default=$2
    echo "${app_config[$key]:-$default}"
}

save_config() {
    local file=${1:-config.txt}
    {
        echo "# Application Configuration"
        echo "# Generated: $(date)"
        echo ""
        for key in "${!app_config[@]}"; do
            echo "$key=${app_config[$key]}"
        done
    } > "$file"
}

# Create sample config
cat > config.txt <<EOF
# Database settings
db_host=localhost
db_port=5432
db_name=myapp

# Application settings
app_name=MyApplication
app_port=8080
debug_mode=true
EOF

# Usage
load_config
echo "Database: $(get_config db_host):$(get_config db_port)/$(get_config db_name)"
echo "App: $(get_config app_name) on port $(get_config app_port)"

# Modify and save
app_config["app_port"]=9000
save_config config_updated.txt
```

---

#### Iteration Patterns

```bash
#!/bin/bash

declare -A environment=(
    ["AWS_REGION"]="us-east-1"
    ["AWS_BUCKET"]="my-bucket"
    ["DB_HOST"]="db.example.com"
    ["CACHE_TTL"]="3600"
)

# Pattern 1: Iterate over keys and values
for key in "${!environment[@]}"; do
    echo "export $key=${environment[$key]}"
done

# Pattern 2: Sorted iteration
for key in $(echo "${!environment[@]}" | tr ' ' '\n' | sort); do
    printf "%-15s = %s\n" "$key" "${environment[$key]}"
done

# Output:
# AWS_BUCKET      = my-bucket
# AWS_REGION      = us-east-1
# CACHE_TTL       = 3600
# DB_HOST         = db.example.com
```

---

#### Advanced: Nested Data Structures

```bash
#!/bin/bash

# Simulating nested structures with key patterns

declare -A infrastructure

# Store server data with pattern: "server:name:property"
infrastructure["server:web1:ip"]="192.168.1.10"
infrastructure["server:web1:port"]="80"
infrastructure["server:web1:status"]="active"

infrastructure["server:db1:ip"]="192.168.1.20"
infrastructure["server:db1:port"]="5432"
infrastructure["server:db1:status"]="active"

infrastructure["server:cache1:ip"]="192.168.1.30"
infrastructure["server:cache1:port"]="6379"
infrastructure["server:cache1:status"]="maintenance"

# Query function
get_server_info() {
    local server_name=$1
    echo "Server: $server_name"
    echo "  IP: ${infrastructure["server:$server_name:ip"]}"
    echo "  Port: ${infrastructure["server:$server_name:port"]}"
    echo "  Status: ${infrastructure["server:$server_name:status"]}"
}

# Get all servers
get_all_servers() {
    local servers=()
    for key in "${!infrastructure[@]}"; do
        if [[ $key =~ ^server:([^:]+): ]]; then
            local server="${BASH_REMATCH[1]}"
            # Add if not already in array
            [[ ! " ${servers[@]} " =~ " ${server} " ]] && servers+=("$server")
        fi
    done
    echo "${servers[@]}"
}

# Usage
echo "All servers: $(get_all_servers)"
echo ""
for server in $(get_all_servers); do
    get_server_info "$server"
    echo ""
done
```

**Output:**
```
All servers: web1 db1 cache1

Server: web1
  IP: 192.168.1.10
  Port: 80
  Status: active

Server: db1
  IP: 192.168.1.20
  Port: 5432
  Status: active

Server: cache1
  IP: 192.168.1.30
  Port: 6379
  Status: maintenance
```

---

### 2.5 Array Performance Tips

```bash
#!/bin/bash

# TIP 1: Pre-allocate arrays when size is known (not directly possible, but pattern helps)
# Instead of:
results=()
for i in {1..1000}; do
    results+=("item$i")  # Reallocates each time
done

# Better: Use mapfile when possible
mapfile -t results < <(seq 1 1000 | sed 's/^/item/')

# TIP 2: Avoid repeatedly accessing ${#array[@]} in loops
# Slow:
for ((i=0; i<${#items[@]}; i++)); do
    echo "${items[i]}"
done

# Faster:
count=${#items[@]}
for ((i=0; i<count; i++)); do
    echo "${items[i]}"
done

# TIP 3: Use += for appending, not array recreation
# Slow:
array=("${array[@]}" "new_item")  # Creates new array

# Fast:
array+=("new_item")  # Modifies in place
```

---

### 2.6 Common Array Mistakes

#### Mistake 1: Not Quoting Array Expansion

```bash
#!/bin/bash

files=("file 1.txt" "file 2.txt")

# WRONG: Word splitting
for file in ${files[@]}; do
    echo "$file"
done
# Output:
# file
# 1.txt
# file
# 2.txt

# CORRECT:
for file in "${files[@]}"; do
    echo "$file"
done
# Output:
# file 1.txt
# file 2.txt
```

#### Mistake 2: Forgetting to Declare Associative Arrays

```bash
#!/bin/bash

# WRONG: Will create indexed array
config["host"]="localhost"
echo "${config["host"]}"  # Won't work as expected

# CORRECT:
declare -A config
config["host"]="localhost"
echo "${config["host"]}"  # Works correctly
```

#### Mistake 3: Modifying Array While Iterating

```bash
#!/bin/bash

items=("a" "b" "c" "d")

# WRONG: Unpredictable results
for item in "${items[@]}"; do
    if [[ $item == "b" ]]; then
        unset items[1]  # Modifying during iteration
    fi
done

# CORRECT: Iterate over copy
for item in "${items[@]}"; do
    echo "$item"
done  # Read-only iteration

# Then modify
unset items[1]
```

---

### 2.7 Array Cheat Sheet

```bash
# ============================
# INDEXED ARRAYS
# ============================
declare -a arr                    # Declare indexed array
arr=("a" "b" "c")                # Initialize
arr[0]="value"                   # Assign to index
arr+=("d")                       # Append element
echo "${arr[0]}"                 # Access element
echo "${arr[@]}"                 # All elements
echo "${arr[*]}"                 # All elements (different quoting)
echo "${#arr[@]}"                # Array length
echo "${!arr[@]}"                # All indices
echo "${arr[@]:2:3}"             # Slice (from index 2, take 3)
unset arr[1]                     # Remove element
arr=()                           # Empty array

# ============================
# ASSOCIATIVE ARRAYS
# ============================
declare -A map                   # MUST declare
map["key"]="value"               # Assign
echo "${map["key"]}"             # Access
echo "${!map[@]}"                # All keys
echo "${map[@]}"                 # All values
[[ -v map["key"] ]]              # Check if key exists
unset map["key"]                 # Remove key

# ============================
# ITERATION
# ============================
for item in "${arr[@]}"; do      # Iterate values
    echo "$item"
done

for i in "${!arr[@]}"; do        # Iterate indices
    echo "$i: ${arr[i]}"
done

for key in "${!map[@]}"; do      # Iterate keys
    echo "$key: ${map[$key]}"
done

# ============================
# CONVERSION
# ============================
IFS=',' read -ra arr <<< "$csv"  # CSV to array
str="${arr[*]}"                  # Array to string (space-separated)
IFS=',' eval 'str="${arr[*]}"'   # Array to CSV

# ============================
# MAPFILE (Bash 4+)
# ============================
mapfile -t arr < file.txt        # File to array (line by line)
mapfile -t arr < <(command)      # Command output to array
```

---

## 3. Modular Functions <a name="functions"></a>

### 3.1 Conceptual Foundation

**What are Functions?**
Functions are reusable code blocks that perform specific tasks, accept parameters, and optionally return values.

**Why Use Functions?**
- **DRY Principle**: Don't Repeat Yourself
- **Maintainability**: Fix bugs in one place
- **Testability**: Test individual components
- **Readability**: Clear, descriptive names
- **Modularity**: Build complex scripts from simple parts

---

### 3.2 Function Basics

#### Declaration Syntax

```bash
#!/bin/bash

# Syntax 1: function keyword (bash-specific)
function greet {
    echo "Hello, $1"
}

# Syntax 2: POSIX-compatible (PREFERRED)
greet() {
    echo "Hello, $1"
}

# Syntax 3: With explicit braces
function greet() {
    echo "Hello, $1"
}

# Calling functions
greet "Alice"     # Output: Hello, Alice
greet "Bob"       # Output: Hello, Bob
```

**Best Practice: Use POSIX syntax `name() { ... }` for portability**

---

### 3.3 Parameter Handling

#### Positional Parameters

```bash
#!/bin/bash

# Basic parameter access
deploy() {
    local env=$1
    local version=$2
    local region=$3
    
    echo "Deploying version $version to $env in $region"
}

deploy "production" "v1.2.3" "us-east-1"
# Output: Deploying version v1.2.3 to production in us-east-1

# All parameters
process_all() {
    echo "Received $# arguments"
    echo "All: $*"
    echo "Each:"
    for arg in "$@"; do
        echo "  - $arg"
    done
}

process_all apple banana "red cherry"
# Output:
# Received 3 arguments
# All: apple banana red cherry
# Each:
#   - apple
#   - banana
#   - red cherry
```

**Parameter Variables:**
- `$1, $2, ... $9, ${10}, ${11}...` - Positional parameters
- `$#` - Number of parameters
- `$@` - All parameters as separate words
- `$*` - All parameters as single word
- `$0` - Script name (not function name)

---

#### Named Parameters (Simulated)

```bash
#!/bin/bash

# Pattern: Using key=value pairs
create_user() {
    local username=""
    local email=""
    local role="user"
    
    # Parse named parameters
    while [[ $# -gt 0 ]]; do
        case $1 in
            --username)
                username="$2"
                shift 2
                ;;
            --email)
                email="$2"
                shift 2
                ;;
            --role)
                role="$2"
                shift 2
                ;;
            *)
                echo "Unknown parameter: $1" >&2
                return 1
                ;;
        esac
    done
    
    # Validate required parameters
    if [[ -z $username ]]; then
        echo "Error: --username is required" >&2
        return 1
    fi
    
    echo "Creating user:"
    echo "  Username: $username"
    echo "  Email: ${email:-not provided}"
    echo "  Role: $role"
}

# Usage
create_user --username "alice" --email "alice@example.com" --role "admin"
create_user --username "bob"
```

---

#### Default Values and Validation

```bash
#!/bin/bash

# Comprehensive parameter handling
connect_database() {
    # Required parameters
    local host=${1:?"Error: Database host is required"}
    
    # Optional with defaults
    local port=${2:-5432}
    local database=${3:-"postgres"}
    local user=${4:-"$USER"}
    
    # Validate port number
    if ! [[ $port =~ ^[0-9]+$ ]]; then
        echo "Error: Port must be a number" >&2
        return 1
    fi
    
    if [[ $port -lt 1 || $port -gt 65535 ]]; then
        echo "Error: Port must be between 1-65535" >&2
        return 1
    fi
    
    echo "Connecting to $database@$host:$port as $user"
}

# Usage
connect_database "db.example.com"                    # Uses defaults
connect_database "db.example.com" 3306 "myapp"      # Custom values
connect_database                                     # Error: host required
```

---

### 3.4 Return Values and Exit Codes

#### Exit Status

```bash
#!/bin/bash

# Functions return exit status (0-255)
# 0 = success, non-zero = failure

check_file() {
    local file=$1
    
    if [[ -f $file ]]; then
        echo "File exists: $file"
        return 0  # Success
    else
        echo "File not found: $file" >&2
        return 1  # Failure
    fi
}

# Using return status
if check_file "/etc/passwd"; then
    echo "System file found"
else
    echo "System file missing"
fi

# Capture status
check_file "/nonexistent"
status=$?
echo "Exit status: $status"
```

---

#### Returning Data

```bash
#!/bin/bash

# Method 1: Echo output (MOST COMMON)
get_user_email() {
    local username=$1
    # Simulate database lookup
    echo "${username}@example.com"
}

email=$(get_user_email "alice")
echo "Email: $email"

# Method 2: Modify global variable
RESULT=""
calculate_discount() {
    local price=$1
    local discount_pct=$2
    RESULT=$(awk "BEGIN {print $price * (1 - $discount_pct/100)}")
}

calculate_discount 100 20
echo "Final price: \$${RESULT}"

# Method 3: Nameref (Bash 4.3+)
calculate_total() {
    local -n result_var=$1  # Reference to variable
    local price=$2
    local tax=$3
    result_var=$(awk "BEGIN {print $price * (1 + $tax/100)}")
}

total=0
calculate_total total 100 8.5
echo "Total with tax: \$${total}"

# Method 4: Associative array (for multiple values)
declare -A user_info
get_user_info() {
    local -n info=$1
    local username=$2
    
    info["username"]=$username
    info["email"]="${username}@example.com"
    info["created"]=$(date +%Y-%m-%d)
}

get_user_info user_info "alice"
echo "User: ${user_info[username]}"
echo "Email: ${user_info[email]}"
echo "Created: ${user_info[created]}"
```

---

### 3.5 Local vs Global Scope

```bash
#!/bin/bash

# Global variable
config_dir="/etc/myapp"

update_config() {
    # Local variable (only exists in function)
    local temp_file="/tmp/config.tmp"
    
    # Modify global variable
    config_dir="/var/lib/myapp"
    
    echo "Temp: $temp_file"
    echo "Config: $config_dir"
}

echo "Before: $config_dir"
update_config
echo "After: $config_dir"
echo "Temp accessible? $temp_file"  # Empty - local variable

# Output:
# Before: /etc/myapp
# Temp: /tmp/config.tmp
# Config: /var/lib/myapp
# After: /var/lib/myapp
# Temp accessible?
```

**Best Practice: Always use `local` for function variables**

---

### 3.6 Advanced Function Patterns

#### Error Handling with Traps

```bash
#!/bin/bash

# Cleanup on function exit
deploy_application() {
    local temp_dir=$(mktemp -d)
    
    # Ensure cleanup happens even on error
    trap "rm -rf '$temp_dir'" RETURN
    
    echo "Working in: $temp_dir"
    
    # Simulate deployment
    cd "$temp_dir" || return 1
    echo "Deploying..." > deploy.log
    
    # Simulate error
    if [[ $1 == "fail" ]]; then
        echo "Deployment failed!" >&2
        return 1
    fi
    
    echo "Deployment successful"
    return 0
}

# Temp directory cleaned up automatically
deploy_application "success"
deploy_application "fail"  # Also cleaned up on error
```

---

#### Recursive Functions

```bash
#!/bin/bash

# Calculate factorial
factorial() {
    local n=$1
    
    # Base case
    if [[ $n -le 1 ]]; then
        echo 1
        return
    fi
    
    # Recursive case
    local prev=$(factorial $((n - 1)))
    echo $((n * prev))
}

echo "5! = $(factorial 5)"  # Output: 5! = 120

# Real-world: Directory tree traversal
list_files_recursive() {
    local dir=$1
    local indent=${2:-0}
    local prefix=$(printf '%*s' $indent '')
    
    for item in "$dir"/*; do
        if [[ -d $item ]]; then
            echo "${prefix}[DIR] $(basename "$item")"
            list_files_recursive "$item" $((indent + 2))
        else
            echo "${prefix}[FILE] $(basename "$item")"
        fi
    done
}

list_files_recursive "/etc/ssh"
```

---

#### Function Libraries

```bash
#!/bin/bash

# lib/logger.sh - Logging library

declare -A LOG_LEVELS=(
    ["DEBUG"]=0
    ["INFO"]=1
    ["WARN"]=2
    ["ERROR"]=3
)

LOG_LEVEL=${LOG_LEVEL:-INFO}

log() {
    local level=$1
    shift
    local message="$*"
    
    # Check if level is high enough
    if [[ ${LOG_LEVELS[$level]} -ge ${LOG_LEVELS[$LOG_LEVEL]} ]]; then
        local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
        echo "[$timestamp] [$level] $message" >&2
    fi
}

log_debug() { log "DEBUG" "$@"; }
log_info() { log "INFO" "$@"; }
log_warn() { log "WARN" "$@"; }
log_error() { log "ERROR" "$@"; }

# Usage in main script
source lib/logger.sh

LOG_LEVEL="DEBUG"

log_debug "Application starting"
log_info "Processing user request"
log_warn "Cache miss, fetching from database"
log_error "Database connection failed"
```

---

#### Dependency Injection Pattern

```bash
#!/bin/bash

# Abstract database interface
db_query() {
    echo "ERROR: Database not configured" >&2
    return 1
}

# PostgreSQL implementation
setup_postgres() {
    db_query() {
        local query=$1
        psql -t -c "$query" 2>/dev/null
    }
}

# MySQL implementation
setup_mysql() {
    db_query() {
        local query=$1
        mysql -N -e "$query" 2>/dev/null
    }
}

# SQLite implementation
setup_sqlite() {
    db_query() {
        local query=$1
        sqlite3 mydb.db "$query" 2>/dev/null
    }
}

# Application code doesn't care which database
get_user_count() {
    db_query "SELECT COUNT(*) FROM users"
}

# Configure at runtime
DB_TYPE=${1:-postgres}

case $DB_TYPE in
    postgres) setup_postgres ;;
    mysql) setup_mysql ;;
    sqlite) setup_sqlite ;;
    *) echo "Unknown database: $DB_TYPE" >&2; exit 1 ;;
esac

echo "User count: $(get_user_count)"
```

---

### 3.7 Real-World Example: AWS Deployment Script

```bash
#!/bin/bash

set -euo pipefail

# Configuration
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly LOG_FILE="${LOG_FILE:-/var/log/deploy.log}"

# Logger
log() {
    local level=$1
    shift
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] [$level] $*" | tee -a "$LOG_FILE" >&2
}

log_info() { log "INFO" "$@"; }
log_error() { log "ERROR" "$@"; }

# Error handler
error_exit() {
    log_error "$1"
    exit "${2:-1}"
}

# Validate AWS credentials
validate_aws_credentials() {
    if ! aws sts get-caller-identity &>/dev/null; then
        error_exit "AWS credentials not configured"
    fi
    log_info "AWS credentials validated"
}

# Check if S3 bucket exists
bucket_exists() {
    local bucket=$1
    aws s3 ls "s3://$bucket" &>/dev/null
}

# Create S3 bucket if not exists
ensure_bucket() {
    local bucket=$1
    local region=${2:-us-east-1}
    
    if bucket_exists "$bucket"; then
        log_info "Bucket exists: $bucket"
        return 0
    fi
    
    log_info "Creating bucket: $bucket"
    if aws s3 mb "s3://$bucket" --region "$region" 2>>"$LOG_FILE"; then
        log_info "Bucket created successfully"
        return 0
    else
        error_exit "Failed to create bucket: $bucket"
    fi
}

# Upload directory to S3
upload_to_s3() {
    local source_dir=$1
    local bucket=$2
    local prefix=${3:-}
    
    if [[ ! -d $source_dir ]]; then
        error_exit "Source directory not found: $source_dir"
    fi
    
    log_info "Uploading $source_dir to s3://$bucket/$prefix"
    
    local file_count=$(find "$source_dir" -type f | wc -l)
    log_info "Found $file_count files to upload"
    
    if aws s3 sync "$source_dir" "s3://$bucket/$prefix" \
        --delete \
        --size-only \
        2>>"$LOG_FILE"; then
        log_info "Upload completed successfully"
        return 0
    else
        error_exit "Upload failed"
    fi
}

# Invalidate CloudFront cache
invalidate_cloudfront() {
    local distribution_id=$1
    local paths=${2:-"/*"}
    
    log_info "Creating CloudFront invalidation for $distribution_id"
    
    local invalidation_id=$(aws cloudfront create-invalidation \
        --distribution-id "$distribution_id" \
        --paths "$paths" \
        --query 'Invalidation.Id' \
        --output text 2>>"$LOG_FILE")
    
    if [[ -n $invalidation_id ]]; then
        log_info "Invalidation created: $invalidation_id"
        echo "$invalidation_id"
        return 0
    else
        error_exit "Failed to create invalidation"
    fi
}

# Wait for invalidation to complete
wait_for_invalidation() {
    local distribution_id=$1
    local invalidation_id=$2
    local max_wait=${3:-300}  # 5 minutes
    local elapsed=0
    
    log_info "Waiting for invalidation to complete..."
    
    while [[ $elapsed -lt $max_wait ]]; do
        local status=$(aws cloudfront get-invalidation \
            --distribution-id "$distribution_id" \
            --id "$invalidation_id" \
            --query 'Invalidation.Status' \
            --output text 2>>"$LOG_FILE")
        
        if [[ $status == "Completed" ]]; then
            log_info "Invalidation completed"
            return 0
        fi
        
        sleep 10
        ((elapsed += 10))
        echo -n "." >&2
    done
    
    echo "" >&2
    error_exit "Invalidation timeout after ${max_wait}s"
}

# Main deployment function
deploy() {
    local source_dir=$1
    local bucket=$2
    local distribution_id=${3:-}
    
    log_info "Starting deployment"
    log_info "Source: $source_dir"
    log_info "Bucket: $bucket"
    
    validate_aws_credentials
    ensure_bucket "$bucket"
    upload_to_s3 "$source_dir" "$bucket"
    
    if [[ -n $distribution_id ]]; then
        local inv_id=$(invalidate_cloudfront "$distribution_id")
        wait_for_invalidation "$distribution_id" "$inv_id"
    fi
    
    log_info "Deployment completed successfully"
}

# Usage validation
usage() {
    cat <<EOF
Usage: $0 <source_dir> <s3_bucket> [cloudfront_distribution_id]

Deploy static website to S3 and optionally invalidate CloudFront cache.

Arguments:
    source_dir                 Directory containing website files
    s3_bucket                  S3 bucket name
    cloudfront_distribution_id (Optional) CloudFront distribution ID

Environment Variables:
    LOG_FILE                   Log file path (default: /var/log/deploy.log)
    AWS_PROFILE                AWS profile to use

Examples:
    $0 ./dist my-website-bucket
    $0 ./dist my-website-bucket E1234567890ABC
EOF
    exit 1
}

# Main entry point
main() {
    if [[ $# -lt 2 ]]; then
        usage
    fi
    
    deploy "$@"
}

# Run if executed directly
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
fi
```

**Usage:**
```bash
chmod +x deploy.sh
./deploy.sh ./website-dist my-s3-bucket E1234567890ABC
```

---

### 3.8 Function Best Practices Checklist

```bash
#!/bin/bash

# ✅ GOOD PRACTICES

# 1. Use local variables
process_data() {
    local input=$1
    local result=""
    # ... processing
    echo "$result"
}

# 2. Validate parameters
create_file() {
    local filename=${1:?"Filename required"}
    local content=${2:-""}
    
    [[ $filename =~ ^[a-zA-Z0-9._-]+$ ]] || {
        echo "Invalid filename" >&2
        return 1
    }
    
    echo "$content" > "$filename"
}

# 3. Single Responsibility Principle
# Each function does ONE thing well

# Bad: Does too much
deploy_and_notify() {
    # Deploy code
    # Send email
    # Update database
    # Post to Slack
}

# Good: Separate concerns
deploy() { :; }
send_notification() { :; }
update_deployment_log() { :; }

# 4. Meaningful names
get_user_by_id() { :; }           # Good
gubi() { :; }                      # Bad

# 5. Document complex functions
##
# Process CSV file and generate summary report
#
# Arguments:
#   $1 - Input CSV file path
#   $2 - Output report file path
#
# Returns:
#   0 on success, 1 on error
#
# Example:
#   process_csv_report data.csv report.txt
##
process_csv_report() {
    local csv_file=$1
    local report_file=$2
    # ... implementation
}

# 6. Consistent return values
# 0 = success, non-zero = error
validate_email() {
    [[ $1 =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]
    return $?  # Returns 0 if valid, 1 if invalid
}

# 7. Use set -e carefully with functions
safe_function() {
    local result
    result=$(risky_command) || {
        echo "Command failed, but continuing..." >&2
        result="default"
    }
    echo "$result"
}
```

---

### 3.9 Function Cheat Sheet

```bash
# ============================
# DECLARATION
# ============================
name() { commands; }              # POSIX syntax (preferred)
function name { commands; }       # Bash syntax
function name() { commands; }     # Both keywords

# ============================
# PARAMETERS
# ============================
$1, $2, ... ${10}                # Positional parameters
$#                                # Number of parameters
$@                                # All parameters (separate)
$*                                # All parameters (single string)
local var=$1                      # Local variable
local -r var=$1                   # Read-only local

# ============================
# RETURN
# ============================
return 0                          # Success
return 1                          # Failure
echo "result"                     # Return data via stdout
local -n ref=$1; ref="value"      # Return via nameref

# ============================
# VALIDATION
# ============================
${1:?"Error message"}             # Require parameter
[[ $# -eq 2 ]] || return 1        # Check parameter count
[[ -n $1 ]] || return 1           # Check not empty

# ============================
# SCOPE
# ============================
local var="value"                 # Function scope
var="value"                       # Global scope
declare -g var="value"            # Explicitly global

# ============================
# SPECIAL
# ============================
trap "cleanup" RETURN             # Run on function exit
source lib/module.sh              # Import functions
declare -f function_name          # Show function definition
unset -f function_name            # Delete function
```

---

## 4. Mini Challenges <a name="challenges"></a>

### Challenge 1: URL Parser

**Task**: Create a function that parses a URL into its components using only parameter expansion.

```bash
#!/bin/bash

# parse_url "https://user:pass@api.example.com:8080/v1/users?active=true#section"
# Should output:
#   Protocol: https
#   Username: user
#   Password: pass
#   Host: api.example.com
#   Port: 8080
#   Path: /v1/users
#   Query: active=true
#   Fragment: section

parse_url() {
    local url=$1
    local protocol host port path query fragment auth username password
    
    # YOUR CODE HERE
    
    echo "Protocol: ${protocol:-none}"
    echo "Username: ${username:-none}"
    echo "Password: ${password:-none}"
    echo "Host: ${host:-none}"
    echo "Port: ${port:-none}"
    echo "Path: ${path:-none}"
    echo "Query: ${query:-none}"
    echo "Fragment: ${fragment:-none}"
}
```

**Solution:**
```bash
parse_url() {
    local url=$1
    local protocol host port path query fragment auth username password
    
    # Extract protocol
    protocol=${url%%://*}
    url=${url#*://}
    
    # Extract fragment
    if [[ $url == *"#"* ]]; then
        fragment=${url##*#}
        url=${url%#*}
    fi
    
    # Extract query string
    if [[ $url == *"?"* ]]; then
        query=${url##*\?}
        url=${url%\?*}
    fi
    
    # Extract auth (username:password)
    if [[ $url == *"@"* ]]; then
        auth=${url%%@*}
        url=${url#*@}
        
        if [[ $auth == *":"* ]]; then
            username=${auth%%:*}
            password=${auth#*:}
        else
            username=$auth
        fi
    fi
    
    # Extract path
    if [[ $url == *"/"* ]]; then
        path=/${url#*/}
        url=${url%%/*}
    fi
    
    # Extract port
    if [[ $url == *":"* ]]; then
        host=${url%%:*}
        port=${url#*:}
    else
        host=$url
    fi
    
    echo "Protocol: ${protocol:-none}"
    echo "Username: ${username:-none}"
    echo "Password: ${password:-none}"
    echo "Host: ${host:-none}"
    echo "Port: ${port:-none}"
    echo "Path: ${path:-none}"
    echo "Query: ${query:-none}"
    echo "Fragment: ${fragment:-none}"
}

# Test
parse_url "https://user:pass@api.example.com:8080/v1/users?active=true#section"
```

---

### Challenge 2: Array Statistics

**Task**: Calculate statistics (min, max, average, median) for an array of numbers.

```bash
#!/bin/bash

# calculate_stats 45 23 89 12 67 34 56 78
# Should output:
#   Count: 8
#   Min: 12
#   Max: 89
#   Sum: 404
#   Average: 50.50
#   Median: 50.50

calculate_stats() {
    # YOUR CODE HERE
}
```

**Solution:**
```bash
calculate_stats() {
    if [[ $# -eq 0 ]]; then
        echo "Error: No numbers provided" >&2
        return 1
    fi
    
    local numbers=("$@")
    local count=${#numbers[@]}
    local sum=0
    local min=${numbers[0]}
    local max=${numbers[0]}
    
    # Calculate sum, min, max
    for num in "${numbers[@]}"; do
        sum=$((sum + num))
        [[ $num -lt $min ]] && min=$num
        [[ $num -gt $max ]] && max=$num
    done
    
    # Calculate average
    local avg=$(awk "BEGIN {printf \"%.2f\", $sum/$count}")
    
    # Sort for median
    local sorted
    IFS=$'\n' sorted=($(sort -n <<<"${numbers[*]}"))
    unset IFS
    
    # Calculate median
    local median
    local mid=$((count / 2))
    if [[ $((count % 2)) -eq 0 ]]; then
        # Even count - average of two middle numbers
        median=$(awk "BEGIN {printf \"%.2f\", (${sorted[mid-1]} + ${sorted[mid]})/2}")
    else
        # Odd count - middle number
        median=${sorted[mid]}
    fi
    
    echo "Count: $count"
    echo "Min: $min"
    echo "Max: $max"
    echo "Sum: $sum"
    echo "Average: $avg"
    echo "Median: $median"
}

calculate_stats 45 23 89 12 67 34 56 78
```

---

### Challenge 3: Log File Analyzer

**Task**: Create a log analyzer using associative arrays to track error counts by type.

```bash
#!/bin/bash

# Sample log file:
# 2024-01-15 10:30:45 ERROR Database connection failed
# 2024-01-15 10:30:46 INFO User logged in
# 2024-01-15 10:30:47 ERROR File not found
# 2024-01-15 10:30:48 WARN Low disk space
# 2024-01-15 10:30:49 ERROR Database connection failed

# Expected output:
# Error Summary:
#   Database connection failed: 2
#   File not found: 1
# Total errors: 3

analyze_logs() {
    local log_file=$1
    # YOUR CODE HERE
}
```

**Solution:**
```bash
analyze_logs() {
    local log_file=${1:?"Log file required"}
    
    if [[ ! -f $log_file ]]; then
        echo "Error: File not found: $log_file" >&2
        return 1
    fi
    
    declare -A error_counts
    local total_errors=0
    
    while IFS= read -r line; do
        # Check if line contains ERROR
        if [[ $line =~ ERROR ]]; then
            # Extract error message (everything after ERROR)
            local error_msg=${line#*ERROR }
            
            # Increment counter for this error type
            ((error_counts["$error_msg"]++))
            ((total_errors++))
        fi
    done < "$log_file"
    
    if [[ $total_errors -eq 0 ]]; then
        echo "No errors found"
        return 0
    fi
    
    echo "Error Summary:"
    # Sort by count (descending)
    for error in "${!error_counts[@]}"; do
        printf "%s|%d\n" "$error" "${error_counts[$error]}"
    done | sort -t'|' -k2 -rn | while IFS='|' read -r msg count; do
        printf "  %s: %d\n" "$msg" "$count"
    done
    
    echo "Total errors: $total_errors"
}

# Create test log
cat > test.log <<EOF
2024-01-15 10:30:45 ERROR Database connection failed
2024-01-15 10:30:46 INFO User logged in
2024-01-15 10:30:47 ERROR File not found
2024-01-15 10:30:48 WARN Low disk space
2024-01-15 10:30:49 ERROR Database connection failed
2024-01-15 10:30:50 ERROR Timeout exceeded
EOF

analyze_logs test.log
```

---

### Challenge 4: Configuration Validator

**Task**: Validate and normalize a configuration file using functions.

```bash
#!/bin/bash

# Config file format:
# key = value
# another_key=another_value
#   spaced_key  =  value with spaces  

# Requirements:
# 1. Trim whitespace from keys and values
# 2. Validate key format (alphanumeric and underscores only)
# 3. Detect duplicate keys
# 4. Support comments (#)
# 5. Output normalized config

validate_config() {
    local config_file=$1
    # YOUR CODE HERE
}
```

**Solution:**
```bash
validate_config() {
    local config_file=${1:?"Config file required"}
    
    if [[ ! -f $config_file ]]; then
        echo "Error: File not found: $config_file" >&2
        return 1
    fi
    
    declare -A config
    local line_num=0
    local errors=0
    
    while IFS= read -r line; do
        ((line_num++))
        
        # Skip empty lines and comments
        [[ -z $line || $line =~ ^[[:space:]]*# ]] && continue
        
        # Check if line contains '='
        if [[ ! $line =~ = ]]; then
            echo "Line $line_num: Invalid format (missing '=')" >&2
            ((errors++))
            continue
        fi
        
        # Split on first '='
        local key=${line%%=*}
        local value=${line#*=}
        
        # Trim whitespace
        key=$(echo "$key" | xargs)
        value=$(echo "$value" | xargs)
        
        # Validate key format
        if [[ ! $key =~ ^[a-zA-Z_][a-zA-Z0-9_]*$ ]]; then
            echo "Line $line_num: Invalid key format: '$key'" >&2
            ((errors++))
            continue
        fi
        
        # Check for duplicates
        if [[ -v config["$key"] ]]; then
            echo "Line $line_num: Duplicate key: '$key'" >&2
            ((errors++))
            continue
        fi
        
        # Store normalized config
        config["$key"]=$value
    done < "$config_file"
    
    if [[ $errors -gt 0 ]]; then
        echo "Validation failed with $errors error(s)" >&2
        return 1
    fi
    
    echo "# Normalized Configuration"
    for key in $(printf '%s\n' "${!config[@]}" | sort); do
        printf "%s=%s\n" "$key" "${config[$key]}"
    done
    
    return 0
}

# Test
cat > test.conf <<EOF
# Database settings
db_host = localhost
  db_port  =  5432  
db_name=myapp

# Application settings
app_name = My Application
app-invalid = value
db_host = duplicate

# Comment
valid_key=value
EOF

validate_config test.conf
```

---

## 5. Final Test Script <a name="final-test"></a>

### Comprehensive Data Processing Pipeline

**Scenario**: Build a log processing system that:
1. Reads multiple log files
2. Parses structured log entries
3. Aggregates statistics by service and error type
4. Generates HTML and JSON reports
5. Sends alerts for critical errors

```bash
#!/bin/bash

set -euo pipefail

##
# Advanced Log Processing System
# Demonstrates: Parameter expansion, arrays, functions, modular design
##

# ============================================================================
# CONFIGURATION
# ============================================================================

readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly OUTPUT_DIR="${OUTPUT_DIR:-$SCRIPT_DIR/reports}"
readonly ALERT_THRESHOLD=${ALERT_THRESHOLD:-10}

# ============================================================================
# LOGGING FUNCTIONS
# ============================================================================

declare -A LOG_LEVELS=(["DEBUG"]=0 ["INFO"]=1 ["WARN"]=2 ["ERROR"]=3)
LOG_LEVEL=${LOG_LEVEL:-INFO}

log() {
    local level=$1
    shift
    local message="$*"
    
    if [[ ${LOG_LEVELS[$level]:-0} -ge ${LOG_LEVELS[$LOG_LEVEL]:-1} ]]; then
        printf "[%s] [%s] %s\n" "$(date '+%Y-%m-%d %H:%M:%S')" "$level" "$message" >&2
    fi
}

log_debug() { log "DEBUG" "$@"; }
log_info() { log "INFO" "$@"; }
log_warn() { log "WARN" "$@"; }
log_error() { log "ERROR" "$@"; }

# ============================================================================
# DATA STRUCTURES
# ============================================================================

declare -A error_counts      # error_counts[service:error_type]=count
declare -A service_totals    # service_totals[service]=total_errors
declare -a critical_errors   # Array of critical error messages

# ============================================================================
# UTILITY FUNCTIONS
# ============================================================================

##
# Ensure directory exists
##
ensure_directory() {
    local dir=$1
    
    if [[ ! -d $dir ]]; then
        log_debug "Creating directory: $dir"
        mkdir -p "$dir" || {
            log_error "Failed to create directory: $dir"
            return 1
        }
    fi
}

##
# Validate log file format
##
validate_log_file() {
    local file=$1
    
    if [[ ! -f $file ]]; then
        log_error "File not found: $file"
        return 1
    fi
    
    if [[ ! -r $file ]]; then
        log_error "File not readable: $file"
        return 1
    fi
    
    # Check if file has expected format (at least one valid line)
    if ! grep -qE '^[0-9]{4}-[0-9]{2}-[0-9]{2}' "$file"; then
        log_warn "File may not be in expected format: $file"
    fi
    
    return 0
}

# ============================================================================
# LOG PARSING FUNCTIONS
# ============================================================================

##
# Parse log entry into components
# Format: 2024-01-15 10:30:45 [SERVICE] LEVEL Message
##
parse_log_entry() {
    local line=$1
    local -n result=$2
    
    # Extract timestamp
    result[timestamp]=${line:0:19}
    line=${line:20}
    
    # Extract service name (between brackets)
    if [[ $line =~ ^\[([^]]+)\] ]]; then
        result[service]=${BASH_REMATCH[1]}
        line=${line#*] }
    else
        result[service]="unknown"
    fi
    
    # Extract level
    local level=${line%% *}
    result[level]=$level
    
    # Extract message
    result[message]=${line#* }
}

##
# Categorize error message
##
categorize_error() {
    local message=$1
    
    # Use parameter expansion for pattern matching
    case ${message,,} in
        *database*|*sql*|*connection*)
            echo "database"
            ;;
        *timeout*|*deadline*)
            echo "timeout"
            ;;
        *authentication*|*unauthorized*|*forbidden*)
            echo "auth"
            ;;
        *"not found"*|*404*)
            echo "not_found"
            ;;
        *memory*|*oom*)
            echo "memory"
            ;;
        *)
            echo "other"
            ;;
    esac
}

# ============================================================================
# PROCESSING FUNCTIONS
# ============================================================================

##
# Process single log file
##
process_log_file() {
    local file=$1
    local line_count=0
    local error_count=0
    
    log_info "Processing: ${file##*/}"
    
    validate_log_file "$file" || return 1
    
    while IFS= read -r line; do
        ((line_count++))
        
        # Skip empty lines
        [[ -z $line ]] && continue
        
        # Parse log entry
        declare -A entry
        parse_log_entry "$line" entry
        
        # Process errors
        if [[ ${entry[level]} == "ERROR" ]]; then
            ((error_count++))
            
            local service=${entry[service]}
            local category=$(categorize_error "${entry[message]}")
            local key="$service:$category"
            
            # Increment counters
            ((error_counts["$key"]++))
            ((service_totals["$service"]++))
            
            # Track critical errors
            if [[ ${error_counts["$key"]} -ge $ALERT_THRESHOLD ]]; then
                critical_errors+=("$service has ${error_counts[$key]} $category errors")
            fi
        fi
    done < "$file"
    
    log_info "Processed $line_count lines, found $error_count errors"
}

##
# Process multiple log files
##
process_all_logs() {
    local log_files=("$@")
    
    if [[ ${#log_files[@]} -eq 0 ]]; then
        log_error "No log files provided"
        return 1
    fi
    
    log_info "Processing ${#log_files[@]} log file(s)"
    
    for file in "${log_files[@]}"; do
        process_log_file "$file"
    done
    
    log_info "Processing complete"
}

# ============================================================================
# REPORTING FUNCTIONS
# ============================================================================

##
# Generate text summary
##
generate_summary() {
    local total_errors=0
    
    echo "==============================================="
    echo "           LOG ANALYSIS SUMMARY"
    echo "==============================================="
    echo ""
    echo "Generated: $(date '+%Y-%m-%d %H:%M:%S')"
    echo ""
    
    # Service totals
    echo "Errors by Service:"
    echo "-------------------"
    for service in $(printf '%s\n' "${!service_totals[@]}" | sort); do
        local count=${service_totals[$service]}
        printf "  %-20s %5d\n" "$service" "$count"
        ((total_errors += count))
    done
    echo ""
    
    # Error categories
    echo "Errors by Category:"
    echo "-------------------"
    declare -A category_totals
    for key in "${!error_counts[@]}"; do
        local category=${key#*:}
        ((category_totals["$category"] += error_counts["$key"]))
    done
    
    for category in $(printf '%s\n' "${!category_totals[@]}" | sort); do
        printf "  %-20s %5d\n" "$category" "${category_totals[$category]}"
    done
    echo ""
    
    # Detailed breakdown
    echo "Detailed Breakdown:"
    echo "-------------------"
    for key in "${!error_counts[@]}"; do
        printf "%s|%d\n" "${key//:/ - }" "${error_counts[$key]}"
    done | sort -t'|' -k2 -rn | head -20 | while IFS='|' read -r desc count; do
        printf "  %-40s %5d\n" "$desc" "$count"
    done
    echo ""
    
    echo "Total Errors: $total_errors"
    echo ""
    
    # Critical alerts
    if [[ ${#critical_errors[@]} -gt 0 ]]; then
        echo "CRITICAL ALERTS:"
        echo "-------------------"
        printf '  - %s\n' "${critical_errors[@]}"
        echo ""
    fi
}

##
# Generate HTML report
##
generate_html_report() {
    local output_file=$1
    
    cat > "$output_file" <<'HTML_HEADER'
<!DOCTYPE html>
<html>
<head>
    <title>Log Analysis Report</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        h1 { color: #333; }
        table { border-collapse: collapse; width: 100%; margin: 20px 0; }
        th, td { padding: 10px; text-align: left; border: 1px solid #ddd; }
        th { background-color: #4CAF50; color: white; }
        tr:nth-child(even) { background-color: #f2f2f2; }
        .alert { background-color: #f44336; color: white; padding: 10px; margin: 10px 0; }
    </style>
</head>
<body>
    <h1>Log Analysis Report</h1>
    <p>Generated: HTML_HEADER
    
    date '+%Y-%m-%d %H:%M:%S' >> "$output_file"
    
    cat >> "$output_file" <<'HTML_MIDDLE'
</p>
    
    <h2>Errors by Service</h2>
    <table>
        <tr><th>Service</th><th>Error Count</th></tr>
HTML_MIDDLE
    
    for service in $(printf '%s\n' "${!service_totals[@]}" | sort); do
        echo "        <tr><td>$service</td><td>${service_totals[$service]}</td></tr>" >> "$output_file"
    done
    
    cat >> "$output_file" <<'HTML_FOOTER'
    </table>
    
    <h2>Critical Alerts</h2>
HTML_FOOTER
    
    if [[ ${#critical_errors[@]} -gt 0 ]]; then
        for alert in "${critical_errors[@]}"; do
            echo "    <div class=\"alert\">⚠️ $alert</div>" >> "$output_file"
        done
    else
        echo "    <p>No critical alerts</p>" >> "$output_file"
    fi
    
    echo "</body></html>" >> "$output_file"
    
    log_info "HTML report generated: $output_file"
}

##
# Generate JSON report
##
generate_json_report() {
    local output_file=$1
    
    {
        echo "{"
        echo "  \"generated\": \"$(date -Iseconds)\","
        echo "  \"summary\": {"
        
        # Service totals
        echo "    \"services\": {"
        local first=true
        for service in "${!service_totals[@]}"; do
            [[ $first == false ]] && echo ","
            printf "      \"%s\": %d" "$service" "${service_totals[$service]}"
            first=false
        done
        echo ""
        echo "    },"
        
        # Error details
        echo "    \"errors\": ["
        first=true
        for key in "${!error_counts[@]}"; do
            [[ $first == false ]] && echo ","
            local service=${key%%:*}
            local category=${key#*:}
            printf "      {\"service\": \"%s\", \"category\": \"%s\", \"count\": %d}" \
                "$service" "$category" "${error_counts[$key]}"
            first=false
        done
        echo ""
        echo "    ],"
        
        # Critical alerts
        echo "    \"alerts\": ["
        first=true
        for alert in "${critical_errors[@]}"; do
            [[ $first == false ]] && echo ","
            printf "      \"%s\"" "$alert"
            first=false
        done
        echo ""
        echo "    ]"
        
        echo "  }"
        echo "}"
    } > "$output_file"
    
    log_info "JSON report generated: $output_file"
}

# ============================================================================
# MAIN FUNCTION
# ============================================================================

main() {
    local log_files=()
    local generate_html=false
    local generate_json=false
    
    # Parse arguments
    while [[ $# -gt 0 ]]; do
        case $1 in
            --html)
                generate_html=true
                shift
                ;;
            --json)
                generate_json=true
                shift
                ;;
            --output-dir)
                OUTPUT_DIR=$2
                shift 2
                ;;
            --threshold)
                ALERT_THRESHOLD=$2
                shift 2
                ;;
            -*)
                log_error "Unknown option: $1"
                return 1
                ;;
            *)
                log_files+=("$1")
                shift
                ;;
        esac
    done
    
    # Validate
    if [[ ${#log_files[@]} -eq 0 ]]; then
        echo "Usage: $0 [options] <log_file>..." >&2
        echo "Options:" >&2
        echo "  --html               Generate HTML report" >&2
        echo "  --json               Generate JSON report" >&2
        echo "  --output-dir DIR     Output directory (default: ./reports)" >&2
        echo "  --threshold N        Alert threshold (default: 10)" >&2
        return 1
    fi
    
    # Setup
    ensure_directory "$OUTPUT_DIR"
    
    # Process
    process_all_logs "${log_files[@]}"
    
    # Generate reports
    generate_summary
    
    if [[ $generate_html == true ]]; then
        generate_html_report "$OUTPUT_DIR/report.html"
    fi
    
    if [[ $generate_json == true ]]; then
        generate_json_report "$OUTPUT_DIR/report.json"
    fi
    
    # Return critical status
    if [[ ${#critical_errors[@]} -gt 0 ]]; then
        log_warn "${#critical_errors[@]} critical alert(s) detected"
        return 2
    fi
    
    return 0
}

# ============================================================================
# ENTRY POINT
# ============================================================================

if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
fi
```

### Test Data Generator

```bash
#!/bin/bash

# generate_test_logs.sh - Create sample log files

generate_log_file() {
    local filename=$1
    local service=$2
    local num_entries=${3:-100}
    
    local levels=("INFO" "WARN" "ERROR")
    local errors=(
        "Database connection timeout"
        "SQL query failed"
        "Authentication failed"
        "File not found: /var/data/user.db"
        "Out of memory error"
        "Request timeout after 30s"
        "Unauthorized access attempt"
        "Connection refused to 192.168.1.100"
    )
    
    {
        for ((i=1; i<=num_entries; i++)); do
            local timestamp=$(date -d "now - $((RANDOM % 3600)) seconds" '+%Y-%m-%d %H:%M:%S')
            local level=${levels[$((RANDOM % ${#levels[@]}))]}
            
            if [[ $level == "ERROR" ]]; then
                local message=${errors[$((RANDOM % ${#errors[@]}))]}
            else
                local message="Normal operation message $i"
            fi
            
            echo "$timestamp [$service] $level $message"
        done
    } > "$filename"
    
    echo "Generated: $filename with $num_entries entries"
}

# Generate test logs
generate_log_file "api.log" "api-service" 200
generate_log_file "web.log" "web-service" 150
generate_log_file "db.log" "database" 100
generate_log_file "auth.log" "auth-service" 120

echo "Test log files generated"
```

### Running the Test

```bash
# Generate test data
chmod +x generate_test_logs.sh
./generate_test_logs.sh

# Run log processor
chmod +x log_processor.sh
./log_processor.sh --html --json *.log

# View results
cat reports/report.json
firefox reports/report.html  # or your browser
```

---

## 6. Time-Based Learning Plan <a name="learning-plan"></a>

### Week 1: Parameter Expansion Mastery

#### Day 1-2: Pattern Removal (4 hours)
- **Study**: Section 1.3-1.4 (2 hours)
- **Practice**:
  ```bash
  # Exercise 1: File path manipulation
  for path in /home/user/docs/file.tar.gz /var/log/app.log /tmp/data.csv.bak; do
      echo "Full: $path"
      echo "Filename: ${path##*/}"
      echo "Directory: ${path%/*}"
      echo "Extension: ${path##*.}"
      echo "Base: ${path%%.*}"
      echo "---"
  done
  
  # Exercise 2: URL cleaning
  urls=(
      "https://cdn.example.com/images/logo.png"
      "http://api.site.com/v1/users/123"
      "ftp://files.server.org/data/archive.zip"
  )
  for url in "${urls[@]}"; do
      echo "Protocol: ${url%%://*}"
      echo "Domain: ${url#*://}"; echo "Domain: ${url%%/*}"
      echo "Path: /${url#*/}"
  done
  ```
- **Checkpoint**: Complete Challenge 1 (URL Parser)

#### Day 3-4: String Manipulation (4 hours)
- **Study**: Section 1.5-1.7 (2 hours)
- **Practice**:
  ```bash
  # Exercise 3: Filename sanitization
  sanitize_filename() {
      local name=$1
      name=${name// /_}           # Replace spaces
      name=${name//[^a-zA-Z0-9._-]/}  # Remove special chars
      name=${name,,}              # Lowercase
      echo "$name"
  }
  
  # Test
  sanitize_filename "My Document (2024).TXT"
  
  # Exercise 4: Environment normalization
  normalize_env() {
      local env=${1,,}
      case $env in
          prod|production) echo "production" ;;
          dev|development) echo "development" ;;
          stg|staging) echo "staging" ;;
          *) echo "unknown" ;;
      esac
  }
  
  for env in Prod DEVELOPMENT staging unknown; do
      echo "$env -> $(normalize_env "$env")"
  done
  ```
- **Checkpoint**: Create a script that processes user input with all expansion techniques

#### Day 5-7: Advanced Techniques (6 hours)
- **Study**: Section 1.8-1.11 (2 hours)
- **Practice**: Build a configuration file parser using only parameter expansion
- **Checkpoint**: Complete Performance comparison script, achieve correct optimization

### Week 2: Array Mastery

#### Day 1-3: Indexed Arrays (6 hours)
- **Study**: Section 2.2-2.3 (3 hours)
- **Practice**:
  ```bash
  # Exercise 5: Array manipulation
  servers=("web1" "web2" "db1" "cache1")
  
  # Add server
  servers+=("web3")
  
  # Remove by value
  # (implement remove_element function)
  
  # Sort array
  # (implement sort_array function)
  
  # Slice array
  # (get first 2 servers, last 2 servers)
  
  # Process in batches
  # (implement batch processor)
  ```
- **Checkpoint**: Solve Challenge 2 (Array Statistics)

#### Day 4-5: Associative Arrays (4 hours)
- **Study**: Section 2.4 (2 hours)
- **Practice**:
  ```bash
  # Exercise 6: Build a simple database
  declare -A users
  
  add_user() {
      local username=$1
      local email=$2
      users["$username"]=$email
  }
  
  get_user() {
      local username=$1
      echo "${users[$username]:-User not found}"
  }
  
  list_users() {
      for user in "${!users[@]}"; do
          echo "$user: ${users[$user]}"
      done
  }
  
  # Test CRUD operations
  ```
- **Checkpoint**: Create nested data structure simulator

#### Day 6-7: Advanced Patterns (4 hours)
- **Study**: Section 2.5-2.7 (1 hour)
- **Practice**: Build log analyzer (Challenge 3)
- **Checkpoint**: Optimize array operations, measure performance

### Week 3: Modular Functions

#### Day 1-2: Function Basics (4 hours)
- **Study**: Section 3.2-3.4 (2 hours)
- **Practice**:
  ```bash
  # Exercise 7: Build utility library
  # lib/string.sh
  
  string_reverse() { ... }
  string_contains() { ... }
  string_starts_with() { ... }
  string_ends_with() { ... }
  string_trim() { ... }
  
  # Test all functions
  ```
- **Checkpoint**: Create reusable function library

#### Day 3-4: Parameter Handling (4 hours)
- **Study**: Section 3.3, 3.5 (2 hours)
- **Practice**:
  ```bash
  # Exercise 8: Named parameter parser
  process_request() {
      # Implement --method, --url, --data, --headers
  }
  
  # Should support:
  # process_request --method POST --url /api/users --data '{"name":"Alice"}'
  ```
- **Checkpoint**: Build flexible CLI argument parser

#### Day 5-7: Advanced Patterns (6 hours)
- **Study**: Section 3.6-3.9 (2 hours)
- **Practice**: 
  - Implement dependency injection pattern
  - Create recursive file processor
  - Build function library with error handling
- **Checkpoint**: Complete AWS deployment script example, customize for your use case

### Week 4: Integration & Mastery

#### Day 1-3: Mini Challenges (6 hours)
- Complete all 4 mini challenges
- Optimize solutions
- Add comprehensive error handling

#### Day 4-7: Final Project (8 hours)
- Understand and run final test script
- Extend with additional features:
  - Add email alerting
  - Implement rate limiting detection
  - Add trend analysis
  - Create dashboard view
- Document your code
- Create test suite

### Daily Practice Routine (Throughout Month)

```bash
# morning_drill.sh - 15 minutes daily practice

# Monday: Parameter expansion
echo "Extract filename: /home/user/document.pdf"
# Your answer here

# Tuesday: Arrays
echo "Sort: 45 12 78 23 56"
# Your answer here

# Wednesday: Functions
echo "Write function to validate email"
# Your answer here

# Thursday: Integration
echo "Combine all techniques"
# Your answer here

# Friday: Code review
echo "Review and optimize this week's work"
```

### Checkpoints & Assessments

**Week 1 Checkpoint:**
- [ ] Can remove patterns from start/end of strings
- [ ] Can perform case conversions
- [ ] Can extract substrings efficiently
- [ ] Understands performance implications
- [ ] Score: 8/10 on URL parser challenge

**Week 2 Checkpoint:**
- [ ] Can manipulate indexed arrays
- [ ] Can use associative arrays for lookup tables
- [ ] Can iterate arrays correctly with proper quoting
- [ ] Can implement sorting and filtering
- [ ] Score: 8/10 on statistics challenge

**Week 3 Checkpoint:**
- [ ] Can write functions with parameter validation
- [ ] Understands scope (local vs global)
- [ ] Can implement error handling
- [ ] Can create reusable libraries
- [ ] Score: 8/10 on AWS script customization

**Final Assessment:**
- [ ] Complete final test script: 90%+ functionality
- [ ] Add 3+ custom features
- [ ] Code is readable and well-documented
- [ ] Handles errors gracefully
- [ ] Passes all edge cases

### Recommended Resources

**During Learning:**
- Keep this guide open as reference
- Use `help test`, `help [[`, `help case` in terminal
- Bookmark: https://mywiki.wooledge.org/BashGuide
- Practice in isolated test environment

**For Advanced Topics:**
- Advanced Bash-Scripting Guide (abs-guide)
- ShellCheck tool for code quality
- Bash Hackers Wiki

### Success Metrics

By end of 4 weeks, you should be able to:
- Write a 300+ line production-ready script
- Process complex data structures efficiently
- Debug issues in 5 minutes or less
- Optimize code for 10x+ performance improvements
- Teach concepts to others

---

## Summary Cheat Sheet

```bash
# PARAMETER EXPANSION QUICK REFERENCE
${var#pattern}    ${var##pattern}    # Remove from start (shortest/longest)
${var%pattern}    ${var%%pattern}    # Remove from end (shortest/longest)
${var/pat/rep}    ${var//pat/rep}    # Replace first/all
${var^}  ${var^^}  ${var,}  ${var,,} # Case modification
${var:offset:length}                 # Substring
${var:-default}   ${var:=default}    # Default values

# ARRAY ESSENTIALS
declare -a indexed=("a" "b" "c")     # Indexed array
declare -A assoc=(["key"]="value")   # Associative array
"${array[@]}"                        # All elements (ALWAYS QUOTE)
${#array[@]}                         # Length
${!array[@]}                         # Keys/indices
${array[@]:start:length}             # Slice

# FUNCTION PATTERNS
name() {                             # POSIX syntax
    local param=$1                   # Local variable
    ${param:?"Error message"}        # Required parameter
    echo "result"                    # Return via stdout
    return 0                         # Exit status
}

# COMMON PATTERNS
for item in "${array[@]}"; do        # Iterate array
for key in "${!assoc[@]}"; do        # Iterate hash keys
mapfile -t array < <(command)        # Command output to array
local -n ref=$1; ref="value"         # Return via reference
```

**Remember**: Practice daily, test thoroughly, and always quote your variables!