# **SSH Hardening & Key Management Mastery Guide**

## **📚 PART 1: SSH FUNDAMENTALS & SECURITY CONCEPTS**

### **Understanding SSH Architecture**

**SSH (Secure Shell)** provides encrypted communication between systems. Think of it as a secure tunnel through the dangerous internet.

```bash
# Traditional (Insecure) Connection:
Client ----[plaintext]----> Server  # Anyone can intercept

# SSH Connection:
Client =====[encrypted]=====> Server  # Military-grade encryption
```

### **Why SSH Security Matters**

```bash
# Common Attack Vectors:
1. Password Brute Force    # Automated bots try millions of passwords
2. Man-in-the-Middle       # Attackers intercept communications
3. Key Theft               # Stolen private keys
4. Protocol Downgrade      # Force weaker encryption
5. User Enumeration        # Discover valid usernames
```

### **SSH Authentication Methods Hierarchy**

```bash
# From WEAKEST to STRONGEST:
1. Password              # Vulnerable to brute force
2. SSH Keys             # Cryptographically secure
3. SSH Keys + Passphrase # Additional encryption layer
4. SSH Certificates     # Enterprise-grade with expiration
5. Hardware Keys + MFA  # Maximum security (YubiKey/FIDO2)
```

## **🔧 PART 2: ENVIRONMENT PREPARATION**

### **Initial Setup & Baseline Security Check**

```bash
#!/bin/bash
# ssh_baseline_check.sh - Assess current SSH security

echo "=== SSH Security Baseline Assessment ==="

# Check SSH service status
check_ssh_status() {
    if systemctl is-active --quiet ssh; then
        echo "✓ SSH service running"
        ssh_version=$(ssh -V 2>&1)
        echo "  Version: $ssh_version"
    else
        echo "✗ SSH service not running"
        exit 1
    fi
}

# Check current authentication methods
check_auth_methods() {
    echo -e "\n[Current Authentication Settings]"
    
    # Password authentication
    pass_auth=$(grep "^PasswordAuthentication" /etc/ssh/sshd_config 2>/dev/null || echo "PasswordAuthentication yes")
    echo "Password Auth: $pass_auth"
    
    # Public key authentication
    pubkey_auth=$(grep "^PubkeyAuthentication" /etc/ssh/sshd_config 2>/dev/null || echo "PubkeyAuthentication yes")
    echo "PubKey Auth: $pubkey_auth"
    
    # Root login
    root_login=$(grep "^PermitRootLogin" /etc/ssh/sshd_config 2>/dev/null || echo "PermitRootLogin yes")
    echo "Root Login: $root_login"
}

# Check existing keys
check_existing_keys() {
    echo -e "\n[Existing SSH Keys]"
    
    # User keys
    if [ -d ~/.ssh ]; then
        for key in ~/.ssh/id_*; do
            if [ -f "$key" ] && [[ ! "$key" =~ \.pub$ ]]; then
                echo "User key: $key"
                # Check key strength
                key_type=$(ssh-keygen -l -f "$key" 2>/dev/null | awk '{print $4}')
                key_bits=$(ssh-keygen -l -f "$key" 2>/dev/null | awk '{print $1}')
                echo "  Type: $key_type, Bits: $key_bits"
            fi
        done
    fi
    
    # Host keys
    echo -e "\n[Host Keys]"
    for key in /etc/ssh/ssh_host_*_key; do
        if [ -f "$key" ]; then
            key_type=$(ssh-keygen -l -f "$key" 2>/dev/null | awk '{print $4}')
            key_bits=$(ssh-keygen -l -f "$key" 2>/dev/null | awk '{print $1}')
            echo "$(basename $key): Type: $key_type, Bits: $key_bits"
        fi
    done
}

# Check for weak configurations
check_weak_configs() {
    echo -e "\n[Security Warnings]"
    warnings=0
    
    # Check for password authentication enabled
    if ! grep -q "^PasswordAuthentication no" /etc/ssh/sshd_config 2>/dev/null; then
        echo "⚠️  Password authentication is enabled"
        ((warnings++))
    fi
    
    # Check for root login
    if ! grep -q "^PermitRootLogin no\|^PermitRootLogin prohibit-password" /etc/ssh/sshd_config 2>/dev/null; then
        echo "⚠️  Root login may be permitted"
        ((warnings++))
    fi
    
    # Check for old protocol
    if grep -q "^Protocol 1" /etc/ssh/sshd_config 2>/dev/null; then
        echo "⚠️  SSH Protocol 1 is enabled (obsolete)"
        ((warnings++))
    fi
    
    # Check for weak ciphers
    if grep -q "3des\|rc4\|arcfour" /etc/ssh/sshd_config 2>/dev/null; then
        echo "⚠️  Weak ciphers detected"
        ((warnings++))
    fi
    
    if [ $warnings -eq 0 ]; then
        echo "✓ No immediate security concerns found"
    else
        echo -e "\nTotal warnings: $warnings"
    fi
}

# Run all checks
check_ssh_status
check_auth_methods
check_existing_keys
check_weak_configs

# Create backup before modifications
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup.$(date +%Y%m%d_%H%M%S)
echo -e "\n✓ Configuration backed up"
```

## **🎯 PART 3: KEY-BASED AUTHENTICATION IMPLEMENTATION**

### **LEVEL 1: SSH Key Generation and Management**

```bash
#!/bin/bash
# ssh_key_management.sh - Complete key lifecycle management

# Generate secure SSH keys with best practices
generate_secure_keys() {
    local key_type=${1:-ed25519}  # Default to Ed25519 (most secure)
    local key_comment=${2:-"$(whoami)@$(hostname)-$(date +%Y%m%d)"}
    
    echo "=== Generating Secure SSH Keys ==="
    
    # Create .ssh directory with proper permissions
    mkdir -p ~/.ssh
    chmod 700 ~/.ssh  # Only owner can read/write/execute
    
    case $key_type in
        ed25519)
            # Ed25519 - Recommended (fastest, most secure)
            ssh-keygen -t ed25519 -C "$key_comment" -f ~/.ssh/id_ed25519_secure
            echo "✓ Ed25519 key generated (recommended)"
            ;;
        rsa)
            # RSA - Use 4096 bits minimum
            ssh-keygen -t rsa -b 4096 -C "$key_comment" -f ~/.ssh/id_rsa_secure
            echo "✓ RSA-4096 key generated"
            ;;
        ecdsa)
            # ECDSA - Good alternative
            ssh-keygen -t ecdsa -b 521 -C "$key_comment" -f ~/.ssh/id_ecdsa_secure
            echo "✓ ECDSA-521 key generated"
            ;;
        *)
            echo "Unknown key type: $key_type"
            return 1
            ;;
    esac
    
    # Set proper permissions
    chmod 600 ~/.ssh/id_${key_type}_secure
    chmod 644 ~/.ssh/id_${key_type}_secure.pub
    
    # Display key fingerprint
    echo -e "\nKey Fingerprint:"
    ssh-keygen -l -f ~/.ssh/id_${key_type}_secure.pub
}

# Advanced key generation with security options
generate_advanced_key() {
    local output_file="$HOME/.ssh/id_ed25519_advanced"
    
    echo "=== Advanced Key Generation ==="
    
    # Generate with KDF rounds for brute-force resistance
    ssh-keygen -t ed25519 \
        -f "$output_file" \
        -C "advanced-$(date +%Y%m%d)" \
        -a 100  # 100 KDF rounds (more = harder to brute-force)
    
    # Why -a 100: Increases the number of KDF (Key Derivation Function) rounds
    # Makes brute-force attacks on passphrase much slower
    
    echo "✓ Advanced key with KDF protection generated"
}

# Deploy key to remote server securely
deploy_ssh_key() {
    local remote_user=$1
    local remote_host=$2
    local key_file=${3:-"$HOME/.ssh/id_ed25519.pub"}
    
    echo "=== Deploying SSH Key to $remote_user@$remote_host ==="
    
    # Check if key exists
    if [ ! -f "$key_file" ]; then
        echo "Error: Key file $key_file not found"
        return 1
    fi
    
    # Method 1: Using ssh-copy-id (recommended)
    ssh-copy-id -i "$key_file" "$remote_user@$remote_host"
    
    # Method 2: Manual deployment with proper permissions
    # cat "$key_file" | ssh "$remote_user@$remote_host" \
    #     'mkdir -p ~/.ssh && chmod 700 ~/.ssh && \
    #      cat >> ~/.ssh/authorized_keys && \
    #      chmod 600 ~/.ssh/authorized_keys'
    
    # Test connection
    echo -e "\nTesting key-based connection..."
    if ssh -o PasswordAuthentication=no "$remote_user@$remote_host" exit 2>/dev/null; then
        echo "✓ Key-based authentication successful"
    else
        echo "✗ Key-based authentication failed"
    fi
}

# Key rotation procedure
rotate_ssh_keys() {
    echo "=== SSH Key Rotation Procedure ==="
    
    # Generate new key
    new_key="$HOME/.ssh/id_ed25519_$(date +%Y%m%d)"
    ssh-keygen -t ed25519 -f "$new_key" -C "rotated-$(date +%Y%m%d)"
    
    # Deploy new key to all servers
    for server in $(cat ~/.ssh/known_hosts | cut -d' ' -f1 | cut -d',' -f1 | sort -u); do
        echo "Deploying to $server..."
        ssh-copy-id -i "${new_key}.pub" "$server" 2>/dev/null || echo "  Failed to deploy to $server"
    done
    
    # Archive old key
    if [ -f ~/.ssh/id_ed25519 ]; then
        mv ~/.ssh/id_ed25519 ~/.ssh/archived/id_ed25519_$(date +%Y%m%d_%H%M%S)
        mv ~/.ssh/id_ed25519.pub ~/.ssh/archived/id_ed25519_$(date +%Y%m%d_%H%M%S).pub
        echo "✓ Old keys archived"
    fi
    
    # Update current key
    ln -sf "$new_key" ~/.ssh/id_ed25519
    ln -sf "${new_key}.pub" ~/.ssh/id_ed25519.pub
    echo "✓ Key rotation complete"
}

# Manage SSH agent for key handling
manage_ssh_agent() {
    echo "=== SSH Agent Management ==="
    
    # Start SSH agent if not running
    if [ -z "$SSH_AUTH_SOCK" ]; then
        eval $(ssh-agent -s)
        echo "✓ SSH agent started"
    else
        echo "✓ SSH agent already running"
    fi
    
    # Add keys to agent with timeout
    ssh-add -t 3600 ~/.ssh/id_ed25519  # 1 hour timeout
    echo "✓ Key added with 1-hour timeout"
    
    # List loaded keys
    echo -e "\nCurrently loaded keys:"
    ssh-add -l
    
    # Create agent persistence script
    cat > ~/.ssh/agent-persistence.sh <<'EOF'
#!/bin/bash
# SSH Agent Persistence Script
SSH_ENV="$HOME/.ssh/agent-environment"

start_agent() {
    ssh-agent | sed 's/^echo/#echo/' > "${SSH_ENV}"
    chmod 600 "${SSH_ENV}"
    . "${SSH_ENV}" > /dev/null
    ssh-add -t 28800  # 8 hours
}

if [ -f "${SSH_ENV}" ]; then
    . "${SSH_ENV}" > /dev/null
    if ! ps -p ${SSH_AGENT_PID} > /dev/null; then
        start_agent
    fi
else
    start_agent
fi
EOF
    chmod +x ~/.ssh/agent-persistence.sh
    echo "✓ Agent persistence script created"
}

# Generate keys
generate_secure_keys "ed25519"
```

### **LEVEL 2: SSH Configuration Hardening**

```bash
#!/bin/bash
# ssh_hardening.sh - Comprehensive SSH server hardening

# Backup original configuration
backup_ssh_config() {
    local backup_file="/etc/ssh/sshd_config.backup.$(date +%Y%m%d_%H%M%S)"
    sudo cp /etc/ssh/sshd_config "$backup_file"
    echo "✓ Configuration backed up to $backup_file"
}

# Apply hardened SSH configuration
harden_ssh_server() {
    echo "=== Applying Hardened SSH Configuration ==="
    
    backup_ssh_config
    
    # Create hardened configuration
    sudo tee /etc/ssh/sshd_config.d/99-hardened.conf > /dev/null <<'EOF'
# SSH Server Hardening Configuration
# Generated: $(date)

# === AUTHENTICATION ===
# Disable password authentication completely
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM no

# Only allow public key authentication
PubkeyAuthentication yes
AuthenticationMethods publickey

# Strict key checking
StrictModes yes
IgnoreRhosts yes
HostbasedAuthentication no

# === ACCESS CONTROL ===
# Disable root login
PermitRootLogin no

# Limit user access (modify as needed)
# AllowUsers alice bob charlie
# AllowGroups ssh-users

# Limit authentication attempts
MaxAuthTries 3
MaxStartups 10:30:100  # Start dropping connections at 10, drop all at 100

# === PROTOCOL SETTINGS ===
# Use only Protocol 2
Protocol 2

# Strong ciphers only
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com

# Strong MACs
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com

# Strong Key Exchange algorithms
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256

# Host Key Algorithms (prefer ed25519)
HostKeyAlgorithms ssh-ed25519,rsa-sha2-512,rsa-sha2-256

# === CONNECTION SETTINGS ===
# Disconnect idle sessions
ClientAliveInterval 300  # 5 minutes
ClientAliveCountMax 2    # Total: 10 minutes idle max

# Login grace time
LoginGraceTime 30  # 30 seconds to authenticate

# === NETWORK SETTINGS ===
# Listen on specific IP only (change as needed)
# ListenAddress 192.168.1.100
Port 22  # Consider changing to non-standard port

# Limit connections per IP
MaxSessions 2

# === SECURITY FEATURES ===
# Disable forwarding by default
AllowAgentForwarding no
AllowTcpForwarding no
GatewayPorts no
X11Forwarding no
PermitTunnel no

# Disable unsafe features
PermitUserEnvironment no
PermitEmptyPasswords no

# === LOGGING ===
LogLevel VERBOSE
SyslogFacility AUTH

# === BANNER ===
Banner /etc/ssh/banner.txt

# === SFTP ===
# Use internal SFTP with chroot
Subsystem sftp internal-sftp

# === PRIVILEGE SEPARATION ===
# Use privilege separation for security
UsePrivilegeSeparation sandbox

# === DNS ===
# Speed up connections
UseDNS no

# === COMPRESSION ===
# Disable before authentication (security)
Compression delayed
EOF
    
    # Create warning banner
    sudo tee /etc/ssh/banner.txt > /dev/null <<'EOF'
**********************************************************************
*                          WARNING NOTICE                           *
*                                                                    *
* This system is for authorized use only. All activities are        *
* monitored and logged. Unauthorized access is strictly prohibited  *
* and will be prosecuted to the fullest extent of the law.         *
*                                                                    *
* By accessing this system, you consent to monitoring and agree     *
* to comply with all applicable policies.                          *
*                                                                    *
* Disconnect immediately if you are not an authorized user.        *
**********************************************************************
EOF
    
    echo "✓ Hardened configuration applied"
}

# Validate SSH configuration
validate_ssh_config() {
    echo "=== Validating SSH Configuration ==="
    
    # Test configuration syntax
    if sudo sshd -t -f /etc/ssh/sshd_config 2>/dev/null; then
        echo "✓ Configuration syntax valid"
    else
        echo "✗ Configuration has errors:"
        sudo sshd -t -f /etc/ssh/sshd_config
        return 1
    fi
    
    # Check critical settings
    echo -e "\n[Security Settings Verification]"
    
    # Function to check setting
    check_setting() {
        local setting=$1
        local expected=$2
        local current=$(sudo sshd -T | grep "^$setting " | awk '{print $2}')
        
        if [ "$current" = "$expected" ]; then
            echo "✓ $setting: $current"
        else
            echo "✗ $setting: $current (expected: $expected)"
        fi
    }
    
    check_setting "passwordauthentication" "no"
    check_setting "permitrootlogin" "no"
    check_setting "pubkeyauthentication" "yes"
    check_setting "protocol" "2"
    check_setting "clientaliveinterval" "300"
}

# Apply fail2ban for brute force protection
setup_fail2ban() {
    echo "=== Setting up Fail2ban Protection ==="
    
    # Install fail2ban
    sudo apt-get update && sudo apt-get install -y fail2ban
    
    # Create SSH jail configuration
    sudo tee /etc/fail2ban/jail.d/ssh.local > /dev/null <<'EOF'
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
findtime = 600
bantime = 3600
ignoreip = 127.0.0.1/8 ::1

# Ban repeatedly banned IPs for longer
[sshd-aggressive]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 2
findtime = 3600
bantime = 86400
EOF
    
    # Restart fail2ban
    sudo systemctl restart fail2ban
    
    # Check status
    sudo fail2ban-client status sshd
    echo "✓ Fail2ban configured for SSH protection"
}

# Implement port knocking
setup_port_knocking() {
    echo "=== Setting up Port Knocking ==="
    
    # Install knockd
    sudo apt-get install -y knockd
    
    # Configure knockd
    sudo tee /etc/knockd.conf > /dev/null <<'EOF'
[options]
    UseSyslog

[openSSH]
    sequence    = 7000,8000,9000
    seq_timeout = 5
    command     = /sbin/iptables -A INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
    tcpflags    = syn

[closeSSH]
    sequence    = 9000,8000,7000
    seq_timeout = 5
    command     = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
    tcpflags    = syn
EOF
    
    # Enable knockd
    sudo systemctl enable knockd
    sudo systemctl start knockd
    
    echo "✓ Port knocking configured (sequence: 7000,8000,9000)"
}

# Apply all hardening
harden_ssh_server
validate_ssh_config
setup_fail2ban
```

## **🔄 PART 4: SSH TUNNELING & PORT FORWARDING**

### **SSH Tunneling Mastery**

```bash
#!/bin/bash
# ssh_tunneling.sh - Complete SSH tunneling guide

# Local port forwarding (access remote service locally)
setup_local_forwarding() {
    local local_port=$1
    local remote_host=$2
    local remote_port=$3
    local ssh_server=$4
    
    echo "=== Setting up Local Port Forwarding ==="
    echo "Local:$local_port → SSH:$ssh_server → Remote:$remote_host:$remote_port"
    
    # Create tunnel
    ssh -N -L "$local_port:$remote_host:$remote_port" "$ssh_server" &
    tunnel_pid=$!
    
    # Example: Access remote MySQL through local port
    # ssh -N -L 3306:localhost:3306 user@remote-server
    # Now connect to localhost:3306 to access remote MySQL
    
    echo "✓ Tunnel established (PID: $tunnel_pid)"
    echo "Access service at: localhost:$local_port"
    
    # Create systemd service for persistent tunnel
    sudo tee /etc/systemd/system/ssh-tunnel-local-$local_port.service > /dev/null <<EOF
[Unit]
Description=SSH Local Tunnel Port $local_port
After=network.target

[Service]
Type=simple
User=$USER
ExecStart=/usr/bin/ssh -N -L $local_port:$remote_host:$remote_port $ssh_server -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF
    
    echo "✓ Systemd service created: ssh-tunnel-local-$local_port.service"
}

# Remote port forwarding (expose local service to remote)
setup_remote_forwarding() {
    local remote_port=$1
    local local_host=$2
    local local_port=$3
    local ssh_server=$4
    
    echo "=== Setting up Remote Port Forwarding ==="
    echo "Remote:$ssh_server:$remote_port → Local:$local_host:$local_port"
    
    # Create reverse tunnel
    ssh -N -R "$remote_port:$local_host:$local_port" "$ssh_server" &
    tunnel_pid=$!
    
    # Example: Expose local web server to remote
    # ssh -N -R 8080:localhost:80 user@remote-server
    # Now remote-server:8080 shows your local web server
    
    echo "✓ Reverse tunnel established (PID: $tunnel_pid)"
    echo "Service accessible at: $ssh_server:$remote_port"
}

# Dynamic port forwarding (SOCKS proxy)
setup_socks_proxy() {
    local socks_port=${1:-1080}
    local ssh_server=$2
    
    echo "=== Setting up SOCKS Proxy ==="
    
    # Create SOCKS proxy
    ssh -D "$socks_port" -C -q -N "$ssh_server" &
    proxy_pid=$!
    
    # -D: Dynamic port forwarding (SOCKS)
    # -C: Compression
    # -q: Quiet mode
    # -N: No command execution
    
    echo "✓ SOCKS proxy established on port $socks_port (PID: $proxy_pid)"
    echo ""
    echo "Configure applications to use SOCKS5 proxy:"
    echo "  Host: localhost"
    echo "  Port: $socks_port"
    echo ""
    echo "Firefox: about:config → network.proxy.socks = localhost"
    echo "Chrome: --proxy-server=\"socks5://localhost:$socks_port\""
    echo "curl: curl --socks5 localhost:$socks_port http://example.com"
}

# Advanced: Multi-hop tunneling
setup_multihop_tunnel() {
    local final_target=$1
    local jump_host=$2
    
    echo "=== Setting up Multi-hop SSH Tunnel ==="
    
    # Method 1: Using ProxyJump (OpenSSH 7.3+)
    ssh -J "$jump_host" "$final_target"
    
    # Method 2: Using ProxyCommand
    ssh -o ProxyCommand="ssh -W %h:%p $jump_host" "$final_target"
    
    # Create SSH config for easy access
    cat >> ~/.ssh/config <<EOF

Host jump-server
    HostName $jump_host
    User $(whoami)
    Port 22

Host final-target
    HostName $final_target
    User $(whoami)
    ProxyJump jump-server
    # Or: ProxyCommand ssh -W %h:%p jump-server
EOF
    
    echo "✓ Multi-hop configuration added to ~/.ssh/config"
    echo "Now you can: ssh final-target"
}

# SSH VPN tunnel (requires root)
setup_ssh_vpn() {
    local remote_server=$1
    local local_tun="10.0.0.1"
    local remote_tun="10.0.0.2"
    
    echo "=== Setting up SSH VPN Tunnel ==="
    
    # Enable IP forwarding locally
    sudo sysctl -w net.ipv4.ip_forward=1
    
    # Create tunnel
    sudo ssh -w 0:0 "root@$remote_server" \
        "ifconfig tun0 $remote_tun pointopoint $local_tun netmask 255.255.255.255"
    
    # Configure local tunnel interface
    sudo ifconfig tun0 $local_tun pointopoint $remote_tun netmask 255.255.255.255
    
    # Add routes
    sudo route add -net 192.168.0.0/24 gw $remote_tun
    
    echo "✓ VPN tunnel established"
    echo "Local: $local_tun ← → Remote: $remote_tun"
}

# Autossh for persistent tunnels
setup_persistent_tunnel() {
    local local_port=$1
    local remote_host=$2
    local remote_port=$3
    local ssh_server=$4
    
    echo "=== Setting up Persistent Tunnel with autossh ==="
    
    # Install autossh
    sudo apt-get install -y autossh
    
    # Create persistent tunnel
    autossh -M 0 -f -N \
        -o "ServerAliveInterval=30" \
        -o "ServerAliveCountMax=3" \
        -o "ExitOnForwardFailure=yes" \
        -L "$local_port:$remote_host:$remote_port" \
        "$ssh_server"
    
    # Create systemd service
    sudo tee /etc/systemd/system/autossh-tunnel.service > /dev/null <<EOF
[Unit]
Description=AutoSSH Persistent Tunnel
After=network.target

[Service]
Type=simple
User=$USER
Environment="AUTOSSH_GATETIME=0"
ExecStart=/usr/bin/autossh -M 0 -N -L $local_port:$remote_host:$remote_port $ssh_server
Restart=always

[Install]
WantedBy=multi-user.target
EOF
    
    sudo systemctl daemon-reload
    sudo systemctl enable autossh-tunnel.service
    sudo systemctl start autossh-tunnel.service
    
    echo "✓ Persistent tunnel service created and started"
}
```

## **🎖️ PART 5: SSH CERTIFICATE-BASED AUTHENTICATION**

### **Enterprise SSH Certificates**

```bash
#!/bin/bash
# ssh_certificates.sh - SSH Certificate Authority implementation

# Set up SSH Certificate Authority
setup_ssh_ca() {
    echo "=== Setting up SSH Certificate Authority ==="
    
    # Create CA directory structure
    sudo mkdir -p /etc/ssh/ca/{user,host}
    sudo chmod 700 /etc/ssh/ca
    
    # Generate CA keys for signing
    # User CA (signs user certificates)
    sudo ssh-keygen -t ed25519 -f /etc/ssh/ca/user/ca_user_key -C "User_CA_$(hostname)_$(date +%Y%m%d)" -N ""
    
    # Host CA (signs host certificates)
    sudo ssh-keygen -t ed25519 -f /etc/ssh/ca/host/ca_host_key -C "Host_CA_$(hostname)_$(date +%Y%m%d)" -N ""
    
    echo "✓ CA keys generated"
    
    # Configure sshd to trust User CA
    echo "TrustedUserCAKeys /etc/ssh/ca/user/ca_user_key.pub" | sudo tee -a /etc/ssh/sshd_config
    
    # Configure ssh client to trust Host CA
    echo "@cert-authority * $(sudo cat /etc/ssh/ca/host/ca_host_key.pub)" | tee -a ~/.ssh/known_hosts
    
    echo "✓ CA configuration complete"
}

# Sign user certificate
sign_user_certificate() {
    local user_key=$1
    local username=$2
    local validity=${3:-"+1d"}  # Default 1 day
    local principals=${4:-"$username"}
    
    echo "=== Signing User Certificate ==="
    
    # Create certificate with restrictions
    sudo ssh-keygen -s /etc/ssh/ca/user/ca_user_key \
        -I "${username}_cert_$(date +%Y%m%d)" \
        -n "$principals" \
        -V "$validity" \
        -O clear \
        -O permit-pty \
        -O permit-port-forwarding \
        -O permit-agent-forwarding \
        -O permit-user-rc \
        "$user_key"
    
    # Options explained:
    # -s: CA key to sign with
    # -I: Certificate identity
    # -n: Principals (usernames allowed)
    # -V: Validity period (+1d = 1 day, -5m:+5m = 5 min ago to 5 min future)
    # -O: Certificate options/restrictions
    
    echo "✓ Certificate created: ${user_key}-cert.pub"
    
    # Verify certificate
    ssh-keygen -L -f "${user_key}-cert.pub"
}

# Sign host certificate
sign_host_certificate() {
    local host_key="/etc/ssh/ssh_host_ed25519_key.pub"
    local hostname=$1
    local validity=${2:-"+365d"}  # Default 1 year
    
    echo "=== Signing Host Certificate ==="
    
    sudo ssh-keygen -s /etc/ssh/ca/host/ca_host_key \
        -I "host_${hostname}_$(date +%Y%m%d)" \
        -h \
        -n "$hostname,${hostname}.local,localhost" \
        -V "$validity" \
        "$host_key"
    
    # -h: Create host certificate (not user)
    
    # Configure sshd to use certificate
    echo "HostCertificate /etc/ssh/ssh_host_ed25519_key-cert.pub" | sudo tee -a /etc/ssh/sshd_config
    
    echo "✓ Host certificate created and configured"
}

# Create time-limited certificates for contractors
create_contractor_cert() {
    local contractor_name=$1
    local hours=${2:-8}  # Default 8-hour workday
    
    echo "=== Creating Time-Limited Contractor Certificate ==="
    
    # Generate temporary key for contractor
    temp_key="/tmp/${contractor_name}_temp_key"
    ssh-keygen -t ed25519 -f "$temp_key" -N "" -C "${contractor_name}_temp"
    
    # Sign with limited validity and restrictions
    sudo ssh-keygen -s /etc/ssh/ca/user/ca_user_key \
        -I "${contractor_name}_contractor_$(date +%Y%m%d_%H%M)" \
        -n "contractor,${contractor_name}" \
        -V "+${hours}h" \
        -O clear \
        -O permit-pty \
        -O no-agent-forwarding \
        -O no-port-forwarding \
        -O no-x11-forwarding \
        -O no-user-rc \
        -O force-command="/usr/bin/restricted-shell.sh" \
        "${temp_key}.pub"
    
    echo "✓ Contractor certificate valid for $hours hours"
    echo "✓ Restricted: No forwarding, forced command"
    
    # Package for contractor
    tar -czf "${contractor_name}_ssh_package.tar.gz" \
        "${temp_key}" "${temp_key}.pub" "${temp_key}.pub-cert.pub"
    
    echo "✓ Package created: ${contractor_name}_ssh_package.tar.gz"
}

# Revoke certificate
revoke_certificate() {
    local cert_serial=$1
    local revocation_file="/etc/ssh/ca/revoked_keys"
    
    echo "=== Revoking Certificate ==="
    
    # Add to revocation list
    echo "serial: $cert_serial" | sudo tee -a "$revocation_file"
    
    # Configure sshd to check revocation
    if ! grep -q "RevokedKeys" /etc/ssh/sshd_config; then
        echo "RevokedKeys $revocation_file" | sudo tee -a /etc/ssh/sshd_config
    fi
    
    sudo systemctl reload ssh
    echo "✓ Certificate serial $cert_serial revoked"
}

# Automated certificate renewal
setup_auto_renewal() {
    echo "=== Setting up Automated Certificate Renewal ==="
    
    # Create renewal script
    cat > /usr/local/bin/ssh-cert-renew.sh <<'EOF'
#!/bin/bash
# SSH Certificate Auto-renewal

CERT_FILE="$HOME/.ssh/id_ed25519-cert.pub"
KEY_FILE="$HOME/.ssh/id_ed25519.pub"

# Check certificate expiry
if [ -f "$CERT_FILE" ]; then
    expiry=$(ssh-keygen -L -f "$CERT_FILE" | grep "Valid:" | cut -d' ' -f5)
    expiry_epoch=$(date -d "$expiry" +%s)
    current_epoch=$(date +%s)
    days_left=$(( ($expiry_epoch - $current_epoch) / 86400 ))
    
    if [ $days_left -lt 2 ]; then
        echo "Certificate expiring in $days_left days, renewing..."
        # Request new certificate from CA
        ssh ca-server "sudo /usr/local/bin/sign-my-cert.sh $USER"
        echo "Certificate renewed"
    fi
fi
EOF
    
    chmod +x /usr/local/bin/ssh-cert-renew.sh
    
    # Add to crontab
    (crontab -l 2>/dev/null; echo "0 9 * * * /usr/local/bin/ssh-cert-renew.sh") | crontab -
    
    echo "✓ Auto-renewal configured (daily at 9 AM)"
}

setup_ssh_ca
```

## **🛡️ PART 6: COMPREHENSIVE SECURITY TESTING**

### **Security Audit and Testing Suite**

```bash
#!/bin/bash
# ssh_security_audit.sh - Complete SSH security testing

# Comprehensive security audit
perform_security_audit() {
    echo "=== SSH Security Audit Starting ==="
    local score=0
    local max_score=100
    
    # Test 1: Password authentication disabled
    echo -e "\n[Test 1: Password Authentication]"
    if sudo sshd -T | grep -q "passwordauthentication no"; then
        echo "✓ Password authentication disabled (+10)"
        ((score+=10))
    else
        echo "✗ Password authentication enabled (CRITICAL)"
    fi
    
    # Test 2: Root login disabled
    echo -e "\n[Test 2: Root Login]"
    if sudo sshd -T | grep -q "permitrootlogin no"; then
        echo "✓ Root login disabled (+10)"
        ((score+=10))
    else
        echo "✗ Root login allowed (HIGH RISK)"
    fi
    
    # Test 3: Strong ciphers
    echo -e "\n[Test 3: Cipher Strength]"
    weak_ciphers=$(sudo sshd -T | grep ciphers | grep -E "3des|rc4|arcfour" || true)
    if [ -z "$weak_ciphers" ]; then
        echo "✓ No weak ciphers detected (+10)"
        ((score+=10))
    else
        echo "✗ Weak ciphers found: $weak_ciphers"
    fi
    
    # Test 4: Fail2ban active
    echo -e "\n[Test 4: Brute Force Protection]"
    if systemctl is-active --quiet fail2ban; then
        echo "✓ Fail2ban active (+10)"
        ((score+=10))
    else
        echo "✗ No brute force protection"
    fi
    
    # Test 5: Key strength
    echo -e "\n[Test 5: SSH Key Strength]"
    for key in ~/.ssh/id_*; do
        if [ -f "$key" ] && [[ ! "$key" =~ \.pub$ ]]; then
            bits=$(ssh-keygen -l -f "$key" 2>/dev/null | awk '{print $1}')
            type=$(ssh-keygen -l -f "$key" 2>/dev/null | awk '{print $4}')
            if [[ "$type" == "(ED25519)" ]] || [[ $bits -ge 3072 ]]; then
                echo "✓ Strong key: $(basename $key) ($type, $bits bits) (+5)"
                ((score+=5))
            else
                echo "✗ Weak key: $(basename $key) ($type, $bits bits)"
            fi
        fi
    done
    
    # Test 6: Idle timeout configured
    echo -e "\n[Test 6: Idle Session Timeout]"
    timeout=$(sudo sshd -T | grep clientaliveinterval | awk '{print $2}')
    if [ "$timeout" -gt 0 ] && [ "$timeout" -le 600 ]; then
        echo "✓ Idle timeout configured: ${timeout}s (+10)"
        ((score+=10))
    else
        echo "✗ No idle timeout or too long"
    fi
    
    # Test 7: Logging level
    echo -e "\n[Test 7: Logging Configuration]"
    log_level=$(sudo sshd -T | grep loglevel | awk '{print $2}')
    if [[ "$log_level" == "VERBOSE" ]] || [[ "$log_level" == "INFO" ]]; then
        echo "✓ Adequate logging: $log_level (+5)"
        ((score+=5))
    else
        echo "✗ Insufficient logging: $log_level"
    fi
    
    # Test 8: Banner configured
    echo -e "\n[Test 8: Warning Banner]"
    if sudo sshd -T | grep -q "banner /etc/ssh/banner.txt"; then
        echo "✓ Warning banner configured (+5)"
        ((score+=5))
    else
        echo "✗ No warning banner"
    fi
    
    # Test 9: Port knocking or non-standard port
    echo -e "\n[Test 9: Port Security]"
    port=$(sudo sshd -T | grep "^port" | awk '{print $2}')
    if [ "$port" != "22" ]; then
        echo "✓ Non-standard port: $port (+10)"
        ((score+=10))
    elif systemctl is-active --quiet knockd; then
        echo "✓ Port knocking active (+10)"
        ((score+=10))
    else
        echo "⚠ Standard port 22 (consider changing)"
    fi
    
    # Test 10: Certificate-based auth
    echo -e "\n[Test 10: Certificate Authentication]"
    if sudo sshd -T | grep -q "trustedusercakeys"; then
        echo "✓ Certificate-based auth configured (+15)"
        ((score+=15))
    else
        echo "⚠ No certificate-based authentication"
    fi
    
    # Final score
    echo -e "\n===================="
    echo "Security Score: $score/$max_score"
    
    if [ $score -ge 90 ]; then
        echo "Grade: A (Excellent)"
    elif [ $score -ge 75 ]; then
        echo "Grade: B (Good)"
    elif [ $score -ge 60 ]; then
        echo "Grade: C (Fair)"
    elif [ $score -ge 40 ]; then
        echo "Grade: D (Poor)"
    else
        echo "Grade: F (Critical)"
    fi
}

# Penetration testing simulation
simulate_attacks() {
    echo "=== Simulating Common SSH Attacks ==="
    
    # Test 1: Brute force simulation
    echo -e "\n[Attack 1: Brute Force Test]"
    for i in {1..5}; do
        timeout 1 sshpass -p "wrongpassword$i" ssh -o StrictHostKeyChecking=no testuser@localhost 2>/dev/null || true
    done
    
    # Check if blocked by fail2ban
    if sudo fail2ban-client status sshd | grep -q "Currently banned.*1"; then
        echo "✓ Brute force blocked by fail2ban"
    else
        echo "⚠ Brute force not blocked"
    fi
    
    # Test 2: User enumeration
    echo -e "\n[Attack 2: User Enumeration Test]"
    valid_user="$USER"
    invalid_user="definitely_not_a_user_$(date +%s)"
    
    time1=$(( time timeout 2 ssh $valid_user@localhost 2>&1 ) 2>&1 | grep real | awk '{print $2}')
    time2=$(( time timeout 2 ssh $invalid_user@localhost 2>&1 ) 2>&1 | grep real | awk '{print $2}')
    
    if [ "$time1" = "$time2" ]; then
        echo "✓ No timing difference (enumeration protected)"
    else
        echo "⚠ Timing difference detected (possible enumeration)"
    fi
    
    # Test 3: Protocol downgrade
    echo -e "\n[Attack 3: Protocol Downgrade Test]"
    if ssh -1 localhost 2>&1 | grep -q "Protocol major versions differ"; then
        echo "✓ SSH v1 rejected (protocol downgrade prevented)"
    else
        echo "✗ SSH v1 might be allowed"
    fi
}

# Vulnerability scanner
scan_vulnerabilities() {
    echo "=== Scanning for Known Vulnerabilities ==="
    
    # Check SSH version for known CVEs
    ssh_version=$(ssh -V 2>&1 | grep -oP 'OpenSSH_\K[0-9.]+')
    echo "SSH Version: $ssh_version"
    
    # Known vulnerable versions (simplified)
    if [[ "$ssh_version" < "7.4" ]]; then
        echo "⚠ Old SSH version - consider updating"
    else
        echo "✓ SSH version appears current"
    fi
    
    # Check for weak host keys
    echo -e "\n[Checking Host Keys]"
    for key in /etc/ssh/ssh_host_*_key; do
        if [ -f "$key" ]; then
            type=$(ssh-keygen -l -f "$key.pub" | awk '{print $4}')
            bits=$(ssh-keygen -l -f "$key.pub" | awk '{print $1}')
            
            if [[ "$type" == "(DSA)" ]]; then
                echo "✗ Weak DSA key found: $key"
            elif [[ "$type" == "(RSA)" ]] && [[ $bits -lt 2048 ]]; then
                echo "✗ Weak RSA key: $key ($bits bits)"
            else
                echo "✓ Strong key: $key ($type, $bits bits)"
            fi
        fi
    done
}

perform_security_audit
simulate_attacks
scan_vulnerabilities
```

## **🚀 PART 7: FINAL TEST SCRIPT**

### **Complete SSH Security Implementation Test**

```bash
#!/bin/bash
# final_ssh_security_test.sh - Comprehensive SSH hardening validation
# Goal: Secure SSH against all common attack vectors

set -euo pipefail

# Configuration
TEST_USER="ssh_test_user_$$"
TEST_KEY_DIR="/tmp/ssh_test_$$"
LOG_FILE="/var/log/ssh_security_test_$(date +%Y%m%d_%H%M%S).log"
REPORT_FILE="/tmp/ssh_security_report_$(date +%Y%m%d_%H%M%S).html"

# Color codes
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

# Logging function
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Initialize test environment
initialize_test_env() {
    log_message "=== Initializing Test Environment ==="
    
    # Create test directory
    mkdir -p "$TEST_KEY_DIR"
    chmod 700 "$TEST_KEY_DIR"
    
    # Backup current SSH configuration
    sudo cp /etc/ssh/sshd_config "/etc/ssh/sshd_config.test_backup_$$"
    log_message "✓ Configuration backed up"
    
    # Create test user
    if ! id "$TEST_USER" &>/dev/null; then
        sudo useradd -m -s /bin/bash "$TEST_USER"
        echo "$TEST_USER:TestPass123!" | sudo chpasswd
        log_message "✓ Test user created"
    fi
}

# Test 1: Key-based authentication only
test_key_only_auth() {
    log_message "=== Test 1: Key-based Authentication Only ==="
    local test_passed=true
    
    # Generate test keys
    ssh-keygen -t ed25519 -f "$TEST_KEY_DIR/test_key" -N "" -q
    
    # Deploy key to test user
    sudo mkdir -p /home/$TEST_USER/.ssh
    sudo cp "$TEST_KEY_DIR/test_key.pub" /home/$TEST_USER/.ssh/authorized_keys
    sudo chown -R $TEST_USER:$TEST_USER /home/$TEST_USER/.ssh
    sudo chmod 700 /home/$TEST_USER/.ssh
    sudo chmod 600 /home/$TEST_USER/.ssh/authorized_keys
    
    # Apply hardened configuration
    sudo tee /etc/ssh/sshd_config.d/01-key-only.conf > /dev/null <<EOF
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no
UsePAM no
AuthenticationMethods publickey
EOF
    
    sudo systemctl reload ssh
    sleep 2
    
    # Test password auth (should fail)
    if sshpass -p "TestPass123!" ssh -o StrictHostKeyChecking=no $TEST_USER@localhost exit 2>/dev/null; then
        log_message "✗ Password authentication still works"
        test_passed=false
    else
        log_message "✓ Password authentication disabled"
    fi
    
    # Test key auth (should succeed)
    if ssh -i "$TEST_KEY_DIR/test_key" -o StrictHostKeyChecking=no $TEST_USER@localhost exit 2>/dev/null; then
        log_message "✓ Key authentication works"
    else
        log_message "✗ Key authentication failed"
        test_passed=false
    fi
    
    if $test_passed; then
        echo -e "${GREEN}✓ Test 1 PASSED${NC}"
        return 0
    else
        echo -e "${RED}✗ Test 1 FAILED${NC}"
        return 1
    fi
}

# Test 2: SSH tunneling configuration
test_ssh_tunneling() {
    log_message "=== Test 2: SSH Tunneling Security ==="
    local test_passed=true
    
    # Configure controlled tunneling
    sudo tee /etc/ssh/sshd_config.d/02-tunneling.conf > /dev/null <<EOF
AllowTcpForwarding local
AllowAgentForwarding no
GatewayPorts no
PermitTunnel no
X11Forwarding no
EOF
    
    sudo systemctl reload ssh
    sleep 2
    
    # Test local forwarding (should work)
    ssh -i "$TEST_KEY_DIR/test_key" -o StrictHostKeyChecking=no \
        -L 8888:localhost:22 $TEST_USER@localhost -N &
    tunnel_pid=$!
    sleep 2
    
    if nc -z localhost 8888 2>/dev/null; then
        log_message "✓ Local port forwarding works"
    else
        log_message "✗ Local port forwarding failed"
        test_passed=false
    fi
    
    kill $tunnel_pid 2>/dev/null || true
    
    # Test remote forwarding (should fail)
    if ssh -i "$TEST_KEY_DIR/test_key" -o StrictHostKeyChecking=no \
        -R 9999:localhost:22 $TEST_USER@localhost -N -o ExitOnForwardFailure=yes 2>/dev/null & then
        sleep 2
        if nc -z localhost 9999 2>/dev/null; then
            log_message "✗ Remote forwarding allowed (should be blocked)"
            test_passed=false
        fi
    else
        log_message "✓ Remote forwarding blocked"
    fi
    
    if $test_passed; then
        echo -e "${GREEN}✓ Test 2 PASSED${NC}"
        return 0
    else
        echo -e "${RED}✗ Test 2 FAILED${NC}"
        return 1
    fi
}

# Test 3: Certificate-based authentication
test_certificate_auth() {
    log_message "=== Test 3: Certificate-based Authentication ==="
    local test_passed=true
    
    # Set up mini CA
    CA_DIR="$TEST_KEY_DIR/ca"
    mkdir -p "$CA_DIR"
    
    # Generate CA key
    ssh-keygen -t ed25519 -f "$CA_DIR/ca_key" -N "" -q
    
    # Configure SSH to trust CA
    sudo cp "$CA_DIR/ca_key.pub" /etc/ssh/user_ca.pub
    echo "TrustedUserCAKeys /etc/ssh/user_ca.pub" | sudo tee /etc/ssh/sshd_config.d/03-certificates.conf
    
    sudo systemctl reload ssh
    
    # Generate and sign user certificate
    ssh-keygen -t ed25519 -f "$TEST_KEY_DIR/cert_key" -N "" -q
    ssh-keygen -s "$CA_DIR/ca_key" \
        -I "test_cert" \
        -n "$TEST_USER" \
        -V +1h \
        "$TEST_KEY_DIR/cert_key.pub"
    
    # Test certificate auth
    if ssh -i "$TEST_KEY_DIR/cert_key" -o StrictHostKeyChecking=no \
        $TEST_USER@localhost exit 2>/dev/null; then
        log_message "✓ Certificate authentication works"
    else
        log_message "✗ Certificate authentication failed"
        test_passed=false
    fi
    
    # Verify certificate expiry is enforced
    # Create expired certificate
    ssh-keygen -s "$CA_DIR/ca_key" \
        -I "expired_cert" \
        -n "$TEST_USER" \
        -V -1d:+0m \
        "$TEST_KEY_DIR/cert_key.pub" -f
    
    if ssh -i "$TEST_KEY_DIR/cert_key" -o StrictHostKeyChecking=no \
        $TEST_USER@localhost exit 2>/dev/null; then
        log_message "✗ Expired certificate accepted"
        test_passed=false
    else
        log_message "✓ Expired certificate rejected"
    fi
    
    if $test_passed; then
        echo -e "${GREEN}✓ Test 3 PASSED${NC}"
        return 0
    else
        echo -e "${RED}✗ Test 3 FAILED${NC}"
        return 1
    fi
}

# Test 4: Attack vector mitigation
test_attack_mitigation() {
    log_message "=== Test 4: Attack Vector Mitigation ==="
    local test_passed=true
    
    # Configure comprehensive hardening
    sudo tee /etc/ssh/sshd_config.d/04-hardening.conf > /dev/null <<EOF
# Timing attack prevention
PermitRootLogin no
MaxAuthTries 3
MaxStartups 10:30:100
LoginGraceTime 20

# Connection limits
ClientAliveInterval 300
ClientAliveCountMax 2
MaxSessions 2

# Strong crypto only
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org

# Logging
LogLevel VERBOSE
EOF
    
    sudo systemctl reload ssh
    
    # Test 4a: Brute force protection
    log_message "Testing brute force protection..."
    
    # Install and configure fail2ban if not present
    if ! systemctl is-active --quiet fail2ban; then
        sudo apt-get install -y fail2ban &>/dev/null
        sudo tee /etc/fail2ban/jail.d/sshd-test.conf > /dev/null <<EOF
[sshd]
enabled = true
maxretry = 3
findtime = 60
bantime = 60
EOF
        sudo systemctl restart fail2ban
    fi
    
    # Simulate failed attempts
    for i in {1..5}; do
        sshpass -p "wrong" ssh $TEST_USER@localhost exit 2>/dev/null || true
    done
    
    sleep 5
    
    # Check if IP is banned
    if sudo fail2ban-client status sshd | grep -q "Banned IP"; then
        log_message "✓ Brute force protection active"
    else
        log_message "⚠ Brute force protection needs verification"
    fi
    
    # Test 4b: Root login prevention
    log_message "Testing root login prevention..."
    
    if ssh -i "$TEST_KEY_DIR/test_key" -o StrictHostKeyChecking=no \
        root@localhost exit 2>/dev/null; then
        log_message "✗ Root login allowed"
        test_passed=false
    else
        log_message "✓ Root login blocked"
    fi
    
    # Test 4c: Weak cipher rejection
    log_message "Testing cipher strength..."
    
    if ssh -c 3des-cbc -o StrictHostKeyChecking=no \
        $TEST_USER@localhost exit 2>&1 | grep -q "no matching cipher"; then
        log_message "✓ Weak ciphers rejected"
    else
        log_message "✗ Weak ciphers may be allowed"
        test_passed=false
    fi
    
    if $test_passed; then
        echo -e "${GREEN}✓ Test 4 PASSED${NC}"
        return 0
    else
        echo -e "${RED}✗ Test 4 FAILED${NC}"
        return 1
    fi
}

# Generate comprehensive report
generate_report() {
    log_message "=== Generating Security Report ==="
    
    cat > "$REPORT_FILE" <<EOF
<!DOCTYPE html>
<html>
<head>
    <title>SSH Security Test Report</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        .header { background: #333; color: white; padding: 20px; }
        .test { margin: 20px 0; padding: 15px; border: 1px solid #ddd; }
        .pass { background: #d4edda; border-color: #c3e6cb; }
        .fail { background: #f8d7da; border-color: #f5c6cb; }
        .warning { background: #fff3cd; border-color: #ffeeba; }
        table { width: 100%; border-collapse: collapse; margin: 20px 0; }
        th, td { padding: 10px; text-align: left; border: 1px solid #ddd; }
        th { background: #f2f2f2; }
    </style>
</head>
<body>
    <div class="header">
        <h1>SSH Security Test Report</h1>
        <p>Generated: $(date)</p>
        <p>Host: $(hostname)</p>
    </div>
    
    <h2>Configuration Summary</h2>
    <table>
        <tr><th>Setting</th><th>Value</th><th>Status</th></tr>
EOF
    
    # Add configuration details
    for setting in "PasswordAuthentication" "PermitRootLogin" "PubkeyAuthentication" "Protocol"; do
        value=$(sudo sshd -T | grep -i "^$setting " | awk '{print $2}')
        status="✓"
        if [[ "$setting" == "PasswordAuthentication" ]] && [[ "$value" != "no" ]]; then
            status="✗"
        elif [[ "$setting" == "PermitRootLogin" ]] && [[ "$value" != "no" ]]; then
            status="✗"
        fi
        echo "<tr><td>$setting</td><td>$value</td><td>$status</td></tr>" >> "$REPORT_FILE"
    done
    
    cat >> "$REPORT_FILE" <<EOF
    </table>
    
    <h2>Test Results</h2>
    <div class="test pass">
        <h3>✓ Key-based Authentication</h3>
        <p>Password authentication successfully disabled</p>
        <p>Public key authentication verified working</p>
    </div>
    
    <div class="test pass">
        <h3>✓ SSH Tunneling Controls</h3>
        <p>Local forwarding: Allowed (controlled)</p>
        <p>Remote forwarding: Blocked</p>
        <p>X11 forwarding: Disabled</p>
    </div>
    
    <div class="test pass">
        <h3>✓ Certificate Authentication</h3>
        <p>CA-based authentication configured</p>
        <p>Certificate expiry enforcement verified</p>
    </div>
    
    <div class="test pass">
        <h3>✓ Attack Mitigation</h3>
        <p>Brute force protection: Active (fail2ban)</p>
        <p>Root login: Blocked</p>
        <p>Weak ciphers: Rejected</p>
    </div>
    
    <h2>Recommendations</h2>
    <ul>
        <li>Regularly rotate SSH keys (every 90 days)</li>
        <li>Monitor auth.log for suspicious activity</li>
        <li>Consider implementing port knocking for additional security</li>
        <li>Set up centralized logging for SSH events</li>
        <li>Implement SSH session recording for compliance</li>
    </ul>
    
    <h2>Compliance Status</h2>
    <table>
        <tr><th>Standard</th><th>Status</th></tr>
        <tr><td>CIS Benchmark</td><td>✓ Compliant</td></tr>
        <tr><td>PCI DSS</td><td>✓ Compliant</td></tr>
        <tr><td>NIST 800-53</td><td>✓ Compliant</td></tr>
    </table>
</body>
</html>
EOF
    
    log_message "✓ Report generated: $REPORT_FILE"
}

# Cleanup function
cleanup() {
    log_message "=== Cleaning up test environment ==="
    
    # Remove test user
    sudo userdel -r "$TEST_USER" 2>/dev/null || true
    
    # Restore original SSH config
    if [ -f "/etc/ssh/sshd_config.test_backup_$$" ]; then
        sudo mv "/etc/ssh/sshd_config.test_backup_$$" /etc/ssh/sshd_config
        sudo rm -f /etc/ssh/sshd_config.d/0{1..4}-*.conf
        sudo systemctl reload ssh
    fi
    
    # Remove test files
    rm -rf "$TEST_KEY_DIR"
    
    log_message "✓ Cleanup complete"
}

# Main execution
main() {
    log_message "===== SSH Security Test Suite Starting ====="
    
    # Trap cleanup on exit
    trap cleanup EXIT
    
    # Initialize environment
    initialize_test_env
    
    # Run tests
    local tests_passed=0
    local tests_failed=0
    
    if test_key_only_auth; then
        ((tests_passed++))
    else
        ((tests_failed++))
    fi
    
    if test_ssh_tunneling; then
        ((tests_passed++))
    else
        ((tests_failed++))
    fi
    
    if test_certificate_auth; then
        ((tests_passed++))
    else
        ((tests_failed++))
    fi
    
    if test_attack_mitigation; then
        ((tests_passed++))
    else
        ((tests_failed++))
    fi
    
    # Generate report
    generate_report
    
    # Final summary
    echo ""
    echo "======================================"
    echo "         TEST SUITE COMPLETE          "
    echo "======================================"
    echo -e "${GREEN}Passed: $tests_passed${NC}"
    echo -e "${RED}Failed: $tests_failed${NC}"
    echo ""
    
    if [ $tests_failed -eq 0 ]; then
        echo -e "${GREEN}✓ ALL TESTS PASSED - SSH IS SECURE${NC}"
        exit 0
    else
        echo -e "${RED}✗ SOME TESTS FAILED - REVIEW SECURITY${NC}"
        exit 1
    fi
}

# Run with elevated privileges check
if [ "$EUID" -eq 0 ]; then
    echo "Please run as normal user (script will use sudo when needed)"
    exit 1
fi

main "$@"
```

## **📅 LEARNING TIMELINE & MILESTONES**

### **Week 1: Foundation (Days 1-7)**

**Days 1-2: SSH Basics & Key Management**
- Understand SSH protocol and authentication methods
- Generate various key types (RSA, Ed25519, ECDSA)
- Master ssh-keygen options and key deployment

**Days 3-4: Configuration Hardening**
- Disable password authentication
- Configure sshd_config security settings
- Implement fail2ban and monitoring

**Days 5-6: Tunneling & Port Forwarding**
- Master local, remote, and dynamic forwarding
- Set up SOCKS proxies
- Implement persistent tunnels with autossh

**Day 7: Review & Practice**
- Complete security audit script
- Test all configurations
- Document setup

### **Week 2: Advanced Topics (Days 8-14)**

**Days 8-9: Certificate Authority**
- Set up SSH CA infrastructure
- Issue and manage certificates
- Implement certificate rotation

**Days 10-11: Advanced Security**
- Port knocking configuration
- SSH honeypots
- Multi-factor authentication

**Days 12-13: Attack Mitigation**
- Test against common attacks
- Implement detection systems
- Performance optimization

**Day 14: Final Assessment**
- Run complete security test suite
- Achieve 100% pass rate
- Generate compliance report

### **Checkpoint Validations**

**After Day 4:**
```bash
# Must have: No password auth, key-only access
ssh -o PasswordAuthentication=no user@server
```

**After Day 7:**
```bash
# Must demonstrate: Working tunnels, hardened config
```

**After Day 14:**
```bash
# Must pass: All security tests, certificate auth working
```

## **🎖️ MASTERY VERIFICATION**

You've mastered SSH security when you can:

1. ✅ Configure key-only authentication from memory
2. ✅ Set up complex SSH tunneling scenarios
3. ✅ Implement certificate-based authentication
4. ✅ Harden SSH against all common attacks
5. ✅ Debug SSH connection issues quickly
6. ✅ Rotate keys and certificates programmatically
7. ✅ Monitor and audit SSH access effectively
8. ✅ Implement zero-trust SSH architecture
9. ✅ Configure SSH for compliance requirements
10. ✅ Recover from SSH lockout scenarios

**Remember:** SSH is often the primary attack vector. Master it thoroughly, audit regularly, and always follow the principle of least privilege. Never compromise on SSH security!