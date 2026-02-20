# Package Management Warfare: Complete DevOps Mastery Guide
## Duration: 1.5 hours | Level: Beginner to Advanced

---

# 📋 TABLE OF CONTENTS

1. [Understanding Package Management](#part-1-understanding-package-management)
2. [APT Mastery (Debian/Ubuntu)](#part-2-apt-mastery)
3. [DPKG Deep Dive](#part-3-dpkg-deep-dive)
4. [YUM/DNF Mastery (RHEL/CentOS/Fedora)](#part-4-yumdnf-mastery)
5. [RPM Deep Dive](#part-5-rpm-deep-dive)
6. [Repository Management](#part-6-repository-management)
7. [Compiling from Source](#part-7-compiling-from-source)
8. [Cross-Distribution Skills](#part-8-cross-distribution-skills)
9. [Cheat Sheets & Quick Reference](#part-9-cheat-sheets)
10. [Hands-On Exercises](#part-10-exercises)

---

# PART 1: UNDERSTANDING PACKAGE MANAGEMENT

## 1.1 What is a Package Manager?

A package manager is software that automates installing, upgrading, configuring, and removing software packages. Think of it as an "app store" for your terminal.

### The Package Management Hierarchy

```
┌─────────────────────────────────────────────────────────┐
│                    HIGH-LEVEL TOOLS                      │
│         (User-friendly, handles dependencies)            │
│                                                          │
│    Debian/Ubuntu: apt, apt-get, aptitude                │
│    RHEL/CentOS:   yum, dnf                              │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                    LOW-LEVEL TOOLS                       │
│         (Direct package manipulation)                    │
│                                                          │
│    Debian/Ubuntu: dpkg                                  │
│    RHEL/CentOS:   rpm                                   │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                   PACKAGE FILES                          │
│                                                          │
│    Debian/Ubuntu: .deb files                            │
│    RHEL/CentOS:   .rpm files                            │
└─────────────────────────────────────────────────────────┘
```

### Why This Matters for DevOps

| Scenario | Why Package Management Matters |
|----------|-------------------------------|
| Server provisioning | Automate consistent software installation |
| CI/CD pipelines | Install build dependencies reproducibly |
| Security patches | Apply updates quickly across fleet |
| Troubleshooting | Understand what's installed, verify integrity |
| Container building | Minimize image size, understand layers |

---

# PART 2: APT MASTERY

## 2.1 Understanding APT vs APT-GET

### The Evolution

```
apt-get (1998) → Traditional, scriptable, verbose
apt (2014)     → Modern, user-friendly, colored output
```

### Key Differences

| Feature | apt | apt-get |
|---------|-----|---------|
| Progress bar | ✅ Yes | ❌ No |
| Colored output | ✅ Yes | ❌ No |
| Combined commands | ✅ Yes | ❌ No |
| Script-friendly | ⚠️ Output may change | ✅ Stable output |
| Upgrade behavior | `apt upgrade` safer | `apt-get upgrade` |

**DevOps Rule**: Use `apt-get` in scripts (stable output), use `apt` interactively.

---

## 2.2 APT Command Deep Dive

### 2.2.1 Updating Package Lists

```bash
sudo apt update
```

**What This Does:**
1. Contacts all configured repositories
2. Downloads package index files
3. Updates local cache at `/var/lib/apt/lists/`
4. Does NOT install anything

**Expected Output:**
```
Hit:1 http://archive.ubuntu.com/ubuntu jammy InRelease
Get:2 http://archive.ubuntu.com/ubuntu jammy-updates InRelease [119 kB]
Get:3 http://security.ubuntu.com/ubuntu jammy-security InRelease [110 kB]
Get:4 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 Packages [1,234 kB]
Fetched 1,463 kB in 2s (731 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
45 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

**Understanding the Output:**
- `Hit`: Package list unchanged (cached)
- `Get`: Downloading new package list
- `Ign`: Ignored (usually by configuration)
- `Err`: Error fetching (network/repo issue)

---

### 2.2.2 Upgrading Packages

#### Standard Upgrade (Safe)
```bash
sudo apt upgrade
```

**What This Does:**
- Upgrades installed packages
- Never removes packages
- Never installs new dependencies
- Safe for production systems

**Interactive Confirmation:**
```
The following packages will be upgraded:
  base-files curl libcurl4 openssl
4 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
Need to get 1,234 kB of archives.
After this operation, 0 B of additional disk space will be used.
Do you want to continue? [Y/n]
```

**Keyboard Shortcuts During Confirmation:**
| Key | Action |
|-----|--------|
| `Y` or `Enter` | Proceed with upgrade |
| `n` | Cancel operation |
| `d` | Show details about changes |

#### Full Upgrade (Complete)
```bash
sudo apt full-upgrade
```

**What This Does:**
- Upgrades all packages
- May remove packages if needed
- May install new dependencies
- Use for major version upgrades

**When to Use Each:**
```
Daily updates     → apt upgrade
Security patches  → apt upgrade
New Ubuntu release → apt full-upgrade
Kernel upgrades   → apt full-upgrade
```

---

### 2.2.3 Installing Packages

#### Basic Installation
```bash
sudo apt install nginx
```

**Breakdown:**
- `sudo`: Run as root (required for system changes)
- `apt`: The package manager
- `install`: Action to perform
- `nginx`: Package name

**Expected Output:**
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  nginx-common nginx-core
Suggested packages:
  fcgiwrap nginx-doc
The following NEW packages will be installed:
  nginx nginx-common nginx-core
0 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
Need to get 592 kB of archives.
After this operation, 1,588 kB of additional disk space will be used.
Do you want to continue? [Y/n]
```

**Understanding Dependencies:**
```
nginx (main package)
├── nginx-common (configuration files)
├── nginx-core (core binaries)
└── Depends on: libc6, libpcre3, libssl3, zlib1g
```

#### Advanced Installation Options

```bash
# Install multiple packages
sudo apt install nginx postgresql redis-server

# Install specific version
sudo apt install nginx=1.18.0-0ubuntu1

# Install without prompts (for scripts)
sudo apt install -y nginx

# Simulate installation (dry run)
sudo apt install --dry-run nginx
# OR
sudo apt install -s nginx

# Install recommended packages too
sudo apt install --install-recommends nginx

# Skip recommended packages
sudo apt install --no-install-recommends nginx

# Reinstall a package
sudo apt install --reinstall nginx

# Download only, don't install
sudo apt install --download-only nginx
```

**Flag Breakdown:**

| Flag | Long Form | Purpose |
|------|-----------|---------|
| `-y` | `--yes` | Automatic yes to prompts |
| `-s` | `--simulate` | Dry run, show what would happen |
| `-d` | `--download-only` | Download but don't install |
| `-q` | `--quiet` | Less output |
| `-V` | `--verbose-versions` | Show full versions |

---

### 2.2.4 Removing Packages

#### Standard Remove
```bash
sudo apt remove nginx
```

**What This Does:**
- Removes package binaries
- Keeps configuration files
- Keeps data files

#### Purge (Complete Removal)
```bash
sudo apt purge nginx
```

**What This Does:**
- Removes package binaries
- Removes configuration files
- Keeps data in `/var/lib/` and logs

#### Autoremove (Clean Dependencies)
```bash
sudo apt autoremove
```

**What This Does:**
- Removes packages that were auto-installed as dependencies
- Removes packages no longer needed

#### Complete Cleanup Command
```bash
sudo apt purge nginx && sudo apt autoremove -y
```

**Comparison Table:**

| Command | Binaries | Config Files | Dependencies |
|---------|----------|--------------|--------------|
| `remove` | ❌ Removed | ✅ Kept | ✅ Kept |
| `purge` | ❌ Removed | ❌ Removed | ✅ Kept |
| `autoremove` | N/A | N/A | ❌ Removed |

---

### 2.2.5 Searching and Getting Information

#### Search for Packages
```bash
# Search by name and description
apt search nginx

# Search only package names
apt search --names-only nginx

# Case-insensitive full-text search
apt search "web server"
```

**Expected Output:**
```
Sorting... Done
Full Text Search... Done
nginx/jammy-updates 1.18.0-6ubuntu14.3 amd64
  small, powerful, scalable web/proxy server

nginx-common/jammy-updates 1.18.0-6ubuntu14.3 all
  small, powerful, scalable web/proxy server - common files

nginx-core/jammy-updates 1.18.0-6ubuntu14.3 amd64
  nginx web/proxy server (standard version)
```

#### Show Package Information
```bash
apt show nginx
```

**Expected Output:**
```
Package: nginx
Version: 1.18.0-6ubuntu14.3
Priority: optional
Section: web
Origin: Ubuntu
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
Bugs: https://bugs.launchpad.net/ubuntu/+filebug
Installed-Size: 44.0 kB
Depends: nginx-core (<< 1.18.0-6ubuntu14.3.1~) | nginx-full (<< ...) | ...
Pre-Depends: dpkg (>= 1.15.6~)
Recommends: nginx-core (>= 1.18.0-6ubuntu14.3) | nginx-full (>= ...) | ...
Homepage: https://nginx.org
Download-Size: 3,596 B
APT-Sources: http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 Packages
Description: small, powerful, scalable web/proxy server
```

**Key Fields Explained:**

| Field | Meaning |
|-------|---------|
| `Version` | Package version |
| `Installed-Size` | Disk space after installation |
| `Download-Size` | Download size (compressed) |
| `Depends` | Required packages |
| `Recommends` | Suggested packages |
| `APT-Sources` | Which repository provides this |

#### List Packages
```bash
# List all installed packages
apt list --installed

# List upgradable packages
apt list --upgradable

# List all versions of a package
apt list -a nginx

# Count installed packages
apt list --installed 2>/dev/null | wc -l
```

---

## 2.3 APT-CACHE: The Query Tool

`apt-cache` is specifically for querying the package cache without modifying anything.

### 2.3.1 Search Operations

```bash
# Basic search
apt-cache search nginx

# Search with regex
apt-cache search '^nginx'

# Search in package names only
apt-cache search --names-only nginx
```

### 2.3.2 Package Information

```bash
# Show package details
apt-cache show nginx

# Show just the dependencies
apt-cache depends nginx
```

**Expected Output:**
```
nginx
  PreDepends: dpkg
  Depends: nginx-core
  Depends: nginx-full
  Depends: nginx-light
  Depends: nginx-extras
```

```bash
# Show reverse dependencies (what depends on this)
apt-cache rdepends nginx

# Show package statistics
apt-cache stats
```

**Expected Output:**
```
Total package names: 85,247 (1,705 k)
Total package structures: 85,247 (3,750 k)
  Normal packages: 65,342
  Pure virtual packages: 1,432
  Single virtual packages: 5,234
  Mixed virtual packages: 1,234
  Missing: 12,005
Total distinct versions: 68,234 (4,914 k)
Total distinct descriptions: 45,234 (1,085 k)
Total dependencies: 567,234 (15.9 M)
```

### 2.3.3 Policy and Priority

```bash
# Show installation candidate and priorities
apt-cache policy nginx
```

**Expected Output:**
```
nginx:
  Installed: 1.18.0-6ubuntu14.3
  Candidate: 1.18.0-6ubuntu14.3
  Version table:
 *** 1.18.0-6ubuntu14.3 500
        500 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 Packages
        100 /var/lib/dpkg/status
     1.18.0-6ubuntu14 500
        500 http://archive.ubuntu.com/ubuntu jammy/main amd64 Packages
```

**Understanding Priority Numbers:**

| Priority | Meaning |
|----------|---------|
| 1 | Installed, but will be upgraded |
| 100 | Currently installed version |
| 500 | Standard repository priority |
| 990 | Target release |
| 1000 | Will always be installed |

---

# PART 3: DPKG DEEP DIVE

## 3.1 Understanding DPKG

`dpkg` is the low-level tool that actually installs `.deb` packages. APT is a frontend that calls dpkg.

### Why Use DPKG Directly?

1. Installing downloaded `.deb` files
2. Querying installed packages
3. Troubleshooting broken packages
4. Extracting package contents
5. Verifying package integrity

---

## 3.2 DPKG Commands

### 3.2.1 Installing .deb Files

```bash
# Download a .deb file
wget https://example.com/package.deb

# Install the .deb file
sudo dpkg -i package.deb
```

**Common Issue: Missing Dependencies**
```
dpkg: dependency problems prevent configuration of package:
 package depends on libfoo; however:
  Package libfoo is not installed.
```

**Solution: Fix dependencies with APT**
```bash
sudo apt install -f
# OR
sudo apt --fix-broken install
```

**Complete Workflow:**
```bash
# Download package
wget https://example.com/package.deb

# Attempt install
sudo dpkg -i package.deb

# Fix any dependency issues
sudo apt install -f

# Verify installation
dpkg -l | grep package
```

### 3.2.2 Removing Packages

```bash
# Remove package (keep config)
sudo dpkg -r package-name

# Purge package (remove config too)
sudo dpkg -P package-name

# Force removal (dangerous!)
sudo dpkg --force-remove-reinstreq -r package-name
```

### 3.2.3 Querying Packages

```bash
# List all installed packages
dpkg -l

# Filter installed packages
dpkg -l | grep nginx

# List files installed by a package
dpkg -L nginx
```

**Expected Output:**
```
/.
/etc
/etc/nginx
/etc/nginx/nginx.conf
/etc/nginx/sites-available
/etc/nginx/sites-available/default
/usr/share/nginx
/usr/share/nginx/html
/usr/share/nginx/html/index.html
/var/log/nginx
```

```bash
# Find which package owns a file
dpkg -S /usr/bin/curl
```

**Expected Output:**
```
curl: /usr/bin/curl
```

```bash
# Show package information
dpkg -s nginx
```

**Expected Output:**
```
Package: nginx
Status: install ok installed
Priority: optional
Section: web
Installed-Size: 44
Maintainer: Ubuntu Developers
Architecture: amd64
Version: 1.18.0-6ubuntu14.3
```

### 3.2.4 Package Contents Without Installing

```bash
# List contents of .deb file
dpkg -c package.deb

# Show package info from .deb file
dpkg -I package.deb

# Extract package contents to current directory
dpkg -x package.deb ./extracted/

# Extract control files
dpkg -e package.deb ./control/
```

### 3.2.5 DPKG Flag Reference

| Flag | Long Form | Purpose |
|------|-----------|---------|
| `-i` | `--install` | Install a .deb file |
| `-r` | `--remove` | Remove package |
| `-P` | `--purge` | Remove with config |
| `-l` | `--list` | List packages |
| `-L` | `--listfiles` | List files in package |
| `-S` | `--search` | Find package owning file |
| `-s` | `--status` | Show package status |
| `-c` | `--contents` | List .deb contents |
| `-I` | `--info` | Show .deb info |
| `-x` | `--extract` | Extract files |
| `-e` | `--control` | Extract control info |

---

## 3.3 Troubleshooting with DPKG

### 3.3.1 Common Package States

```bash
dpkg -l | grep nginx
```

**Output Format:**
```
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name           Version      Architecture Description
+++-==============-============-============-============================
ii  nginx          1.18.0-6     amd64        small, powerful, web server
```

**Understanding Status Codes:**

| First Letter | Desired State |
|--------------|---------------|
| u | Unknown |
| i | Install |
| h | Hold |
| r | Remove |
| p | Purge |

| Second Letter | Current State |
|---------------|---------------|
| n | Not installed |
| i | Installed |
| c | Config files only |
| U | Unpacked |
| F | Failed config |
| H | Half installed |
| W | Triggers awaiting |
| t | Triggers pending |

### 3.3.2 Fixing Broken Packages

```bash
# Check for broken packages
dpkg --audit

# Reconfigure a package
sudo dpkg --configure -a

# Force configuration
sudo dpkg --configure --force-confold package-name

# Clear half-installed package
sudo dpkg --remove --force-remove-reinstreq package-name
```

### 3.3.3 Package Hold (Prevent Updates)

```bash
# Put package on hold
sudo apt-mark hold nginx
# OR
echo "nginx hold" | sudo dpkg --set-selections

# Show held packages
apt-mark showhold
# OR
dpkg --get-selections | grep hold

# Remove hold
sudo apt-mark unhold nginx
```

---

# PART 4: YUM/DNF MASTERY

## 4.1 Understanding YUM vs DNF

### Evolution
```
YUM (2003) → Yellowdog Updater Modified
DNF (2015) → Dandified YUM (Fedora 22+, RHEL 8+)
```

### Distribution Usage

| Distribution | Default Package Manager |
|--------------|------------------------|
| RHEL 7, CentOS 7 | YUM |
| RHEL 8+, CentOS 8+ | DNF |
| Fedora 22+ | DNF |
| Amazon Linux 2 | YUM |
| Amazon Linux 2023 | DNF |

**Compatibility Note**: DNF is backward compatible with YUM commands.

---

## 4.2 DNF/YUM Command Deep Dive

### 4.2.1 Updating System

```bash
# Check for updates
dnf check-update

# Update all packages
sudo dnf update

# Update specific package
sudo dnf update nginx

# Security updates only
sudo dnf update --security
```

**Difference from APT:**
- APT: `update` = refresh cache, `upgrade` = update packages
- DNF: `check-update` = refresh + show available, `update` = update packages

### 4.2.2 Installing Packages

```bash
# Basic installation
sudo dnf install nginx

# Install multiple packages
sudo dnf install nginx postgresql redis

# Install without prompts
sudo dnf install -y nginx

# Install specific version
sudo dnf install nginx-1.20.0

# Install local RPM file
sudo dnf install ./package.rpm

# Install from URL
sudo dnf install https://example.com/package.rpm

# Reinstall package
sudo dnf reinstall nginx
```

**Expected Output:**
```
Last metadata expiration check: 0:45:32 ago on Mon 15 Jan 2024 10:00:00 AM UTC.
Dependencies resolved.
================================================================================
 Package          Arch       Version               Repository          Size
================================================================================
Installing:
 nginx            x86_64     1:1.20.1-13.el9       appstream          36 k
Installing dependencies:
 nginx-core       x86_64     1:1.20.1-13.el9       appstream         565 k
 nginx-filesystem noarch     1:1.20.1-13.el9       appstream         8.5 k

Transaction Summary
================================================================================
Install  3 Packages

Total download size: 610 k
Installed size: 1.8 M
Is this ok [y/N]:
```

### 4.2.3 Removing Packages

```bash
# Remove package
sudo dnf remove nginx

# Remove with dependencies
sudo dnf autoremove nginx

# Remove unused dependencies
sudo dnf autoremove

# Clean metadata cache
sudo dnf clean all
```

### 4.2.4 Searching and Information

```bash
# Search for packages
dnf search nginx

# Search in names only
dnf search --all nginx

# Show package info
dnf info nginx

# Show installed packages
dnf list installed

# Show available packages
dnf list available

# Show specific package versions
dnf list nginx --showduplicates
```

### 4.2.5 Groups (Package Collections)

```bash
# List available groups
dnf group list

# Show group info
dnf group info "Development Tools"

# Install group
sudo dnf group install "Development Tools"

# Remove group
sudo dnf group remove "Development Tools"
```

**Common Groups:**
- "Development Tools" - Compilers, make, etc.
- "System Tools" - System utilities
- "Web Server" - Apache, mod_ssl, etc.

---

## 4.3 DNF History and Rollback

### 4.3.1 Transaction History

```bash
# View transaction history
dnf history

# View specific transaction
dnf history info 5

# Undo a transaction
sudo dnf history undo 5

# Redo a transaction
sudo dnf history redo 5

# Rollback to before transaction
sudo dnf history rollback 5
```

**Expected History Output:**
```
ID     | Command line             | Date and time    | Action(s)      | Altered
-------------------------------------------------------------------------------
     5 | install nginx            | 2024-01-15 10:00 | Install        |    3
     4 | update                   | 2024-01-14 09:00 | Update         |   45
     3 | install postgresql       | 2024-01-13 14:00 | Install        |    8
     2 | install htop vim         | 2024-01-12 11:00 | Install        |    2
     1 |                          | 2024-01-10 08:00 | Install        |  150
```

### 4.3.2 This is Crucial for DevOps

```bash
# Before major changes, note the ID
dnf history | head -5

# Make changes
sudo dnf update

# If something breaks, rollback
sudo dnf history rollback PREVIOUS_ID
```

---

# PART 5: RPM DEEP DIVE

## 5.1 Understanding RPM

RPM (Red Hat Package Manager) is the low-level tool for RHEL-based systems, similar to dpkg for Debian.

### 5.1.1 RPM Package Naming

```
nginx-1.20.1-13.el9.x86_64.rpm
│     │      │  │   │
│     │      │  │   └── Architecture
│     │      │  └────── Distribution (el9 = RHEL 9)
│     │      └───────── Release number
│     └──────────────── Version
└────────────────────── Package name
```

## 5.2 RPM Commands

### 5.2.1 Installing

```bash
# Install RPM package
sudo rpm -ivh package.rpm

# Flags:
# -i = install
# -v = verbose
# -h = show progress with hash marks

# Upgrade (install or upgrade)
sudo rpm -Uvh package.rpm

# Freshen (upgrade only if installed)
sudo rpm -Fvh package.rpm

# Install without dependencies (dangerous!)
sudo rpm -ivh --nodeps package.rpm
```

### 5.2.2 Querying

```bash
# List all installed packages
rpm -qa

# Query specific package
rpm -q nginx

# Show package info
rpm -qi nginx

# List files in package
rpm -ql nginx

# Find package owning file
rpm -qf /usr/bin/curl

# Show dependencies
rpm -qR nginx

# List config files
rpm -qc nginx

# List documentation
rpm -qd nginx
```

### 5.2.3 Querying Uninstalled RPMs

```bash
# Query an RPM file (not installed)
rpm -qip package.rpm    # Info
rpm -qlp package.rpm    # List files
rpm -qRp package.rpm    # Dependencies
```

### 5.2.4 Removing

```bash
# Remove package
sudo rpm -e nginx

# Force removal (dangerous!)
sudo rpm -e --nodeps nginx
```

### 5.2.5 Verifying Packages

```bash
# Verify all packages
rpm -Va

# Verify specific package
rpm -V nginx
```

**Verification Output:**
```
S.5....T.  c /etc/nginx/nginx.conf
```

**Understanding Verification Codes:**

| Code | Meaning |
|------|---------|
| S | Size differs |
| M | Mode differs |
| 5 | MD5 sum differs |
| D | Device differs |
| L | readLink differs |
| U | User differs |
| G | Group differs |
| T | mTime differs |
| P | caPabilities differ |
| c | Config file |
| d | Documentation |
| g | Ghost file |
| l | License file |
| r | Readme file |

### 5.2.6 RPM Flag Summary

| Query Flags | Purpose |
|-------------|---------|
| `-q` | Query |
| `-a` | All packages |
| `-i` | Info |
| `-l` | List files |
| `-f` | Find owner of file |
| `-R` | Requirements/dependencies |
| `-c` | Config files |
| `-d` | Documentation files |
| `-p` | Query uninstalled package |

| Action Flags | Purpose |
|--------------|---------|
| `-i` | Install |
| `-U` | Upgrade |
| `-F` | Freshen |
| `-e` | Erase/remove |
| `-V` | Verify |
| `-v` | Verbose |
| `-h` | Hash progress |

---

# PART 6: REPOSITORY MANAGEMENT

## 6.1 APT Repository Management (Debian/Ubuntu)

### 6.1.1 Understanding sources.list

```bash
# Main sources file
cat /etc/apt/sources.list

# Additional sources
ls /etc/apt/sources.list.d/
```

**Source Line Format:**
```
deb http://archive.ubuntu.com/ubuntu jammy main restricted universe multiverse
│   │                                 │     │
│   │                                 │     └── Components
│   │                                 └── Distribution codename
│   └── Repository URL
└── Package type (deb = binary, deb-src = source)
```

**Components Explained:**

| Component | Contents |
|-----------|----------|
| main | Official supported software |
| restricted | Proprietary drivers |
| universe | Community maintained |
| multiverse | Non-free software |

### 6.1.2 Adding Repositories

#### Method 1: Manual (Traditional)

```bash
# Edit sources.list directly
sudo nano /etc/apt/sources.list

# Or add to sources.list.d
echo "deb http://example.com/repo jammy main" | sudo tee /etc/apt/sources.list.d/example.list
```

#### Method 2: add-apt-repository

```bash
# Add PPA repository
sudo add-apt-repository ppa:ondrej/php

# Add repository with key
sudo add-apt-repository "deb http://example.com/repo jammy main"

# Remove repository
sudo add-apt-repository --remove ppa:ondrej/php
```

### 6.1.3 Managing GPG Keys (Modern Method)

```bash
# Download and add key (Ubuntu 22.04+ method)
curl -fsSL https://example.com/key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/example.gpg

# Create sources list with signed-by
echo "deb [signed-by=/etc/apt/keyrings/example.gpg] https://example.com/repo jammy main" | sudo tee /etc/apt/sources.list.d/example.list

# Update package lists
sudo apt update
```

### 6.1.4 Real-World Example: Adding Docker Repository

```bash
# Step 1: Install prerequisites
sudo apt install -y ca-certificates curl gnupg

# Step 2: Add Docker's GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Step 3: Set up repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Step 4: Update and install
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
```

### 6.1.5 APT Pinning (Priority Control)

Create `/etc/apt/preferences.d/custom`:

```bash
# Pin nginx to a specific version
Package: nginx
Pin: version 1.18.*
Pin-Priority: 1001

# Prefer packages from security repo
Package: *
Pin: release a=jammy-security
Pin-Priority: 600
```

**Priority Reference:**
| Priority | Effect |
|----------|--------|
| < 0 | Never install |
| 0-100 | Only if not installed |
| 100-500 | Install unless other version |
| 500-990 | Default priority |
| 990 | Target release |
| > 1000 | Always install, even downgrade |

---

## 6.2 DNF Repository Management (RHEL/CentOS)

### 6.2.1 Repository Configuration

```bash
# List configured repositories
dnf repolist

# List all repositories (including disabled)
dnf repolist all

# Show detailed repo info
dnf repoinfo
```

### 6.2.2 Repository Files

```bash
# Repository config location
ls /etc/yum.repos.d/

# Example repo file structure
cat /etc/yum.repos.d/example.repo
```

```ini
[example-repo]
name=Example Repository
baseurl=https://example.com/repo/el9/$basearch/
enabled=1
gpgcheck=1
gpgkey=https://example.com/RPM-GPG-KEY-example
```

**Variables Explained:**
- `$basearch`: System architecture (x86_64)
- `$releasever`: OS version (9)

### 6.2.3 Adding Repositories

```bash
# Add repository using config-manager
sudo dnf config-manager --add-repo https://example.com/repo.repo

# Enable repository
sudo dnf config-manager --set-enabled repo-name

# Disable repository
sudo dnf config-manager --set-disabled repo-name

# Install from specific repo
sudo dnf install --enablerepo=repo-name package
```

### 6.2.4 EPEL Repository (Essential for RHEL)

```bash
# Install EPEL (Extra Packages for Enterprise Linux)
sudo dnf install epel-release

# For RHEL (not CentOS), might need:
sudo subscription-manager repos --enable codeready-builder-for-rhel-9-x86_64-rpms
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
```

### 6.2.5 Real-World Example: Adding Docker Repository (RHEL/CentOS)

```bash
# Add Docker repository
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# Install Docker
sudo dnf install docker-ce docker-ce-cli containerd.io

# Start Docker
sudo systemctl start docker
sudo systemctl enable docker
```

---

# PART 7: COMPILING FROM SOURCE

## 7.1 Why Compile from Source?

| Reason | Explanation |
|--------|-------------|
| Latest version | Newer than repository packages |
| Custom options | Enable/disable specific features |
| Optimization | Compile for specific CPU |
| Not in repos | Software unavailable as package |
| Learning | Understand how software works |

## 7.2 Prerequisites

### 7.2.1 Install Build Tools

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install build-essential
# Includes: gcc, g++, make, libc6-dev

# Additional common tools
sudo apt install autoconf automake libtool pkg-config
```

**RHEL/CentOS:**
```bash
sudo dnf groupinstall "Development Tools"
# Includes: gcc, gcc-c++, make, kernel-headers

# Additional common tools
sudo dnf install autoconf automake libtool pkgconfig
```

### 7.2.2 Finding Dependencies

```bash
# Check package build dependencies (Debian)
apt-cache showsrc nginx | grep Build-Depends

# Or get source package
apt-get source nginx
# This shows dependencies in debian/control file
```

---

## 7.3 Standard Compilation Process

### The Classic "./configure && make && make install" Pattern

```bash
# Step 1: Download source
wget https://example.com/software-1.0.tar.gz

# Step 2: Extract
tar -xzf software-1.0.tar.gz
cd software-1.0

# Step 3: Configure
./configure --prefix=/usr/local

# Step 4: Compile
make

# Step 5: Install
sudo make install
```

### 7.3.1 Understanding Each Step

#### Step 1-2: Download and Extract

```bash
# Common archive formats
tar -xzf file.tar.gz     # gzip compressed
tar -xjf file.tar.bz2    # bzip2 compressed
tar -xJf file.tar.xz     # xz compressed
unzip file.zip            # zip archive
```

#### Step 3: Configure

```bash
# Show all options
./configure --help

# Common options
./configure \
    --prefix=/usr/local \           # Installation directory
    --sysconfdir=/etc \             # Config file location
    --enable-feature \              # Enable optional feature
    --disable-feature \             # Disable feature
    --with-library=/path \          # Use specific library
    --without-library               # Don't use library
```

**What configure does:**
1. Checks system capabilities
2. Finds required libraries
3. Generates Makefile
4. Configures build options

**Example with nginx:**
```bash
./configure \
    --prefix=/etc/nginx \
    --sbin-path=/usr/sbin/nginx \
    --modules-path=/usr/lib64/nginx/modules \
    --conf-path=/etc/nginx/nginx.conf \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --pid-path=/var/run/nginx.pid \
    --with-http_ssl_module \
    --with-http_v2_module \
    --with-http_realip_module
```

#### Step 4: Make (Compile)

```bash
# Basic compile
make

# Parallel compile (faster, use number of CPU cores)
make -j$(nproc)

# Or specify number manually
make -j4

# Verbose output
make V=1

# Clean previous build
make clean
```

#### Step 5: Make Install

```bash
# Install to prefix location
sudo make install

# Check what would be installed
make -n install

# Some projects support DESTDIR for staging
sudo make DESTDIR=/tmp/staging install
```

---

## 7.4 Real-World Example: Compiling nginx from Source

### Complete Step-by-Step

```bash
# 1. Install dependencies (Ubuntu)
sudo apt update
sudo apt install -y build-essential libpcre3-dev zlib1g-dev \
    libssl-dev libgd-dev libgeoip-dev

# 2. Download nginx source
cd /usr/local/src
sudo wget https://nginx.org/download/nginx-1.24.0.tar.gz
sudo tar -xzf nginx-1.24.0.tar.gz
cd nginx-1.24.0

# 3. Configure with common options
sudo ./configure \
    --prefix=/etc/nginx \
    --sbin-path=/usr/sbin/nginx \
    --modules-path=/usr/lib64/nginx/modules \
    --conf-path=/etc/nginx/nginx.conf \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --pid-path=/var/run/nginx.pid \
    --lock-path=/var/run/nginx.lock \
    --with-http_ssl_module \
    --with-http_v2_module \
    --with-http_realip_module \
    --with-http_gzip_static_module

# 4. Compile
sudo make -j$(nproc)

# 5. Install
sudo make install

# 6. Verify
nginx -v
nginx -V    # Show compile options

# 7. Create systemd service (if needed)
sudo tee /etc/systemd/system/nginx.service << 'EOF'
[Unit]
Description=The nginx HTTP and reverse proxy server
After=syslog.target network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF

# 8. Enable and start
sudo systemctl daemon-reload
sudo systemctl enable nginx
sudo systemctl start nginx
```

---

## 7.5 Using checkinstall for Clean Removal

`checkinstall` creates a .deb/.rpm package from source installation, enabling clean removal.

```bash
# Install checkinstall
sudo apt install checkinstall   # Debian/Ubuntu
sudo dnf install checkinstall   # RHEL (from EPEL)

# Instead of 'make install', use:
sudo checkinstall --install=yes --pkgname=custom-nginx --pkgversion=1.24.0

# Later, remove cleanly:
sudo dpkg -r custom-nginx       # Debian/Ubuntu
sudo rpm -e custom-nginx        # RHEL
```

---

## 7.6 Handling Compilation Errors

### Common Errors and Solutions

#### Missing Header File
```
error: pcre.h: No such file or directory
```
**Solution:**
```bash
# Find the package that provides it
apt-file search pcre.h          # Debian
dnf provides '*/pcre.h'         # RHEL

# Install development package
sudo apt install libpcre3-dev   # Debian
sudo dnf install pcre-devel     # RHEL
```

#### Missing Library
```
error: cannot find -lssl
```
**Solution:**
```bash
sudo apt install libssl-dev     # Debian
sudo dnf install openssl-devel  # RHEL
```

#### Wrong GCC Version
```
error: unrecognized command line option '-std=c++17'
```
**Solution:**
```bash
# Install newer GCC
sudo apt install gcc-11 g++-11
export CC=gcc-11
export CXX=g++-11
./configure ...
```

---

# PART 8: CROSS-DISTRIBUTION SKILLS

## 8.1 Command Translation Table

| Task | APT (Debian/Ubuntu) | DNF (RHEL/Fedora) |
|------|---------------------|-------------------|
| Update cache | `apt update` | `dnf check-update` |
| Upgrade all | `apt upgrade` | `dnf update` |
| Install package | `apt install pkg` | `dnf install pkg` |
| Remove package | `apt remove pkg` | `dnf remove pkg` |
| Search package | `apt search term` | `dnf search term` |
| Show info | `apt show pkg` | `dnf info pkg` |
| List installed | `apt list --installed` | `dnf list installed` |
| List files | `dpkg -L pkg` | `rpm -ql pkg` |
| Find owner | `dpkg -S /path` | `rpm -qf /path` |
| Clean cache | `apt clean` | `dnf clean all` |
| Add repo | `add-apt-repository` | `dnf config-manager --add-repo` |
| Fix broken | `apt --fix-broken install` | `dnf distro-sync` |
| Hold package | `apt-mark hold pkg` | `dnf versionlock pkg` |
| Show dependencies | `apt depends pkg` | `dnf repoquery --requires pkg` |

## 8.2 File Location Differences

| Purpose | Debian/Ubuntu | RHEL/CentOS |
|---------|---------------|-------------|
| Repo config | `/etc/apt/sources.list.d/` | `/etc/yum.repos.d/` |
| GPG keys | `/etc/apt/keyrings/` | `/etc/pki/rpm-gpg/` |
| Package cache | `/var/cache/apt/` | `/var/cache/dnf/` |
| Package database | `/var/lib/dpkg/` | `/var/lib/rpm/` |

## 8.3 Detection Script

```bash
#!/bin/bash
# detect-package-manager.sh

detect_pkg_manager() {
    if command -v apt &> /dev/null; then
        echo "apt"
    elif command -v dnf &> /dev/null; then
        echo "dnf"
    elif command -v yum &> /dev/null; then
        echo "yum"
    elif command -v pacman &> /dev/null; then
        echo "pacman"
    elif command -v zypper &> /dev/null; then
        echo "zypper"
    else
        echo "unknown"
    fi
}

PKG_MANAGER=$(detect_pkg_manager)

case $PKG_MANAGER in
    apt)
        UPDATE_CMD="sudo apt update && sudo apt upgrade -y"
        INSTALL_CMD="sudo apt install -y"
        ;;
    dnf|yum)
        UPDATE_CMD="sudo $PKG_MANAGER update -y"
        INSTALL_CMD="sudo $PKG_MANAGER install -y"
        ;;
    *)
        echo "Unsupported package manager"
        exit 1
        ;;
esac

echo "Package Manager: $PKG_MANAGER"
echo "Update Command: $UPDATE_CMD"
echo "Install Command: $INSTALL_CMD"
```

---

# PART 9: CHEAT SHEETS

## 9.1 APT Quick Reference

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                           APT CHEAT SHEET                                    ║
╠══════════════════════════════════════════════════════════════════════════════╣
║ UPDATING                                                                     ║
║   apt update                 - Refresh package lists                        ║
║   apt upgrade                - Upgrade packages (safe)                      ║
║   apt full-upgrade           - Upgrade with removals allowed                ║
║                                                                              ║
║ INSTALLING                                                                   ║
║   apt install pkg            - Install package                              ║
║   apt install -y pkg         - Install without prompt                       ║
║   apt install pkg=version    - Install specific version                     ║
║   apt install --reinstall    - Reinstall package                            ║
║   apt install --dry-run      - Simulate installation                        ║
║                                                                              ║
║ REMOVING                                                                     ║
║   apt remove pkg             - Remove (keep config)                         ║
║   apt purge pkg              - Remove with config                           ║
║   apt autoremove             - Remove unused dependencies                   ║
║                                                                              ║
║ SEARCHING                                                                    ║
║   apt search term            - Search packages                              ║
║   apt show pkg               - Show package info                            ║
║   apt list --installed       - List installed                               ║
║   apt list --upgradable      - List upgradable                              ║
║                                                                              ║
║ CACHE                                                                        ║
║   apt-cache search term      - Search package cache                         ║
║   apt-cache show pkg         - Show package details                         ║
║   apt-cache depends pkg      - Show dependencies                            ║
║   apt-cache rdepends pkg     - Show reverse dependencies                    ║
║   apt-cache policy pkg       - Show version priorities                      ║
║                                                                              ║
║ MAINTENANCE                                                                  ║
║   apt clean                  - Clear downloaded archives                    ║
║   apt autoclean              - Clear old archives                           ║
║   apt --fix-broken install   - Fix broken dependencies                      ║
║   apt-mark hold pkg          - Prevent upgrades                             ║
║   apt-mark unhold pkg        - Allow upgrades                               ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

## 9.2 DPKG Quick Reference

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                           DPKG CHEAT SHEET                                   ║
╠══════════════════════════════════════════════════════════════════════════════╣
║ INSTALLING                                                                   ║
║   dpkg -i file.deb           - Install .deb file                            ║
║   dpkg -i --force-confnew    - Replace config files                         ║
║                                                                              ║
║ REMOVING                                                                     ║
║   dpkg -r pkg                - Remove package                               ║
║   dpkg -P pkg                - Purge (remove config too)                    ║
║                                                                              ║
║ QUERYING                                                                     ║
║   dpkg -l                    - List all packages                            ║
║   dpkg -l | grep pattern     - Filter packages                              ║
║   dpkg -L pkg                - List files in package                        ║
║   dpkg -S /path/to/file      - Find package owning file                     ║
║   dpkg -s pkg                - Show package status                          ║
║                                                                              ║
║ PACKAGE FILES                                                                ║
║   dpkg -c file.deb           - List contents of .deb                        ║
║   dpkg -I file.deb           - Show .deb info                               ║
║   dpkg -x file.deb dir       - Extract files to dir                         ║
║   dpkg -e file.deb dir       - Extract control info                         ║
║                                                                              ║
║ TROUBLESHOOTING                                                              ║
║   dpkg --configure -a        - Configure pending packages                   ║
║   dpkg --audit               - Check for problems                           ║
║   dpkg --get-selections      - Show package states                          ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

## 9.3 DNF Quick Reference

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                           DNF CHEAT SHEET                                    ║
╠══════════════════════════════════════════════════════════════════════════════╣
║ UPDATING                                                                     ║
║   dnf check-update           - Check for updates                            ║
║   dnf update                 - Update all packages                          ║
║   dnf update pkg             - Update specific package                      ║
║   dnf update --security      - Security updates only                        ║
║                                                                              ║
║ INSTALLING                                                                   ║
║   dnf install pkg            - Install package                              ║
║   dnf install -y pkg         - Install without prompt                       ║
║   dnf install ./file.rpm     - Install local RPM                            ║
║   dnf reinstall pkg          - Reinstall package                            ║
║                                                                              ║
║ REMOVING                                                                     ║
║   dnf remove pkg             - Remove package                               ║
║   dnf autoremove             - Remove unused deps                           ║
║                                                                              ║
║ SEARCHING                                                                    ║
║   dnf search term            - Search packages                              ║
║   dnf info pkg               - Show package info                            ║
║   dnf list installed         - List installed                               ║
║   dnf list available         - List available                               ║
║   dnf provides /path         - Find package providing file                  ║
║                                                                              ║
║ GROUPS                                                                       ║
║   dnf group list             - List groups                                  ║
║   dnf group install "Name"   - Install group                                ║
║   dnf group remove "Name"    - Remove group                                 ║
║                                                                              ║
║ HISTORY                                                                      ║
║   dnf history                - Show history                                 ║
║   dnf history undo ID        - Undo transaction                             ║
║   dnf history rollback ID    - Rollback to point                            ║
║                                                                              ║
║ REPOS                                                                        ║
║   dnf repolist               - List repos                                   ║
║   dnf config-manager --add-repo URL  - Add repo                             ║
║   dnf config-manager --set-enabled REPO  - Enable repo                      ║
║                                                                              ║
║ MAINTENANCE                                                                  ║
║   dnf clean all              - Clear cache                                  ║
║   dnf distro-sync            - Sync installed to repo                       ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

## 9.4 RPM Quick Reference

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                           RPM CHEAT SHEET                                    ║
╠══════════════════════════════════════════════════════════════════════════════╣
║ INSTALLING                                                                   ║
║   rpm -ivh file.rpm          - Install (verbose, hash)                      ║
║   rpm -Uvh file.rpm          - Upgrade or install                           ║
║   rpm -Fvh file.rpm          - Freshen (upgrade only)                       ║
║                                                                              ║
║ REMOVING                                                                     ║
║   rpm -e pkg                 - Erase/remove package                         ║
║   rpm -e --nodeps pkg        - Force remove (dangerous)                     ║
║                                                                              ║
║ QUERYING INSTALLED                                                           ║
║   rpm -qa                    - List all installed                           ║
║   rpm -q pkg                 - Query if installed                           ║
║   rpm -qi pkg                - Show info                                    ║
║   rpm -ql pkg                - List files                                   ║
║   rpm -qc pkg                - List config files                            ║
║   rpm -qd pkg                - List documentation                           ║
║   rpm -qR pkg                - Show dependencies                            ║
║   rpm -qf /path              - Find package owning file                     ║
║                                                                              ║
║ QUERYING RPM FILES                                                           ║
║   rpm -qip file.rpm          - Info from file                               ║
║   rpm -qlp file.rpm          - List files from file                         ║
║   rpm -qRp file.rpm          - Dependencies from file                       ║
║                                                                              ║
║ VERIFYING                                                                    ║
║   rpm -Va                    - Verify all packages                          ║
║   rpm -V pkg                 - Verify specific package                      ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

---

## 9.5 Compilation Quick Reference

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                     SOURCE COMPILATION CHEAT SHEET                           ║
╠══════════════════════════════════════════════════════════════════════════════╣
║ STANDARD PROCESS                                                             ║
║   ./configure --prefix=/usr/local                                            ║
║   make -j$(nproc)                                                            ║
║   sudo make install                                                          ║
║                                                                              ║
║ CONFIGURE OPTIONS                                                            ║
║   --help                     - Show all options                             ║
║   --prefix=/path             - Installation directory                       ║
║   --sysconfdir=/path         - Config file location                         ║
║   --enable-FEATURE           - Enable optional feature                      ║
║   --disable-FEATURE          - Disable feature                              ║
║   --with-PACKAGE             - Use external package                         ║
║   --without-PACKAGE          - Don't use package                            ║
║                                                                              ║
║ MAKE OPTIONS                                                                 ║
║   -j N                       - Parallel jobs (use nproc)                    ║
║   -n or --dry-run            - Show what would run                          ║
║   V=1                        - Verbose output                               ║
║   clean                      - Remove built files                           ║
║   distclean                  - Remove configure output too                  ║
║                                                                              ║
║ PREREQUISITES                                                                ║
║   Ubuntu: sudo apt install build-essential                                  ║
║   RHEL:   sudo dnf groupinstall "Development Tools"                         ║
║                                                                              ║
║ TRACKING INSTALLATION                                                        ║
║   sudo checkinstall          - Create package from install                  ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

---

# PART 10: HANDS-ON EXERCISES

## Exercise 1: APT Fundamentals (10 minutes)

### Objective
Master basic APT operations for package discovery and installation.

### Tasks

```bash
# Task 1.1: Update package lists
sudo apt update

# Task 1.2: Search for web servers
apt search "web server" | head -20

# Task 1.3: Show detailed info about nginx
apt show nginx

# Task 1.4: Install nginx without prompt
sudo apt install -y nginx

# Task 1.5: Verify installation
dpkg -l | grep nginx
systemctl status nginx

# Task 1.6: List files installed by nginx
dpkg -L nginx

# Task 1.7: Find what package owns /usr/sbin/nginx
dpkg -S /usr/sbin/nginx

# Task 1.8: Remove nginx but keep config
sudo apt remove nginx

# Task 1.9: Check what's left
dpkg -l | grep nginx
# Notice: Config files remain (status 'rc')

# Task 1.10: Purge to remove everything
sudo apt purge nginx
sudo apt autoremove -y

# Task 1.11: Verify complete removal
dpkg -l | grep nginx
# Should show nothing
```

### Expected Learning
- Difference between `apt` and `apt-cache`
- Difference between `remove` and `purge`
- How to find package information

---

## Exercise 2: DPKG Operations (10 minutes)

### Objective
Learn to work with .deb files directly.

### Tasks

```bash
# Task 2.1: Download a .deb file
wget https://github.com/sharkdp/bat/releases/download/v0.24.0/bat_0.24.0_amd64.deb

# Task 2.2: Inspect the package BEFORE installing
dpkg -I bat_0.24.0_amd64.deb

# Task 2.3: List package contents
dpkg -c bat_0.24.0_amd64.deb

# Task 2.4: Install the package
sudo dpkg -i bat_0.24.0_amd64.deb

# Task 2.5: If dependencies failed, fix them
sudo apt install -f

# Task 2.6: Verify installation
dpkg -s bat
which bat
bat --version

# Task 2.7: Test the tool
bat /etc/passwd

# Task 2.8: Show files installed by bat
dpkg -L bat

# Task 2.9: Remove the package
sudo dpkg -r bat

# Task 2.10: Verify removal
dpkg -l | grep bat
```

---

## Exercise 3: Repository Management (15 minutes)

### Objective
Add third-party repositories securely.

### Tasks

```bash
# Task 3.1: List current repositories
grep -r "^deb" /etc/apt/sources.list /etc/apt/sources.list.d/

# Task 3.2: Add Docker's official repository (proper method)

# Step A: Add prerequisites
sudo apt install -y ca-certificates curl gnupg

# Step B: Create keyring directory
sudo install -m 0755 -d /etc/apt/keyrings

# Step C: Download and store the GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Step D: Set permissions
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Step E: Add the repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Task 3.3: Update and verify
sudo apt update
apt-cache policy docker-ce

# Task 3.4: Install from new repository
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Task 3.5: Verify Docker works
sudo docker --version
sudo docker run hello-world

# Task 3.6: (Cleanup - optional) Remove Docker repo
sudo rm /etc/apt/sources.list.d/docker.list
sudo rm /etc/apt/keyrings/docker.gpg
sudo apt update
```

---

## Exercise 4: DNF/YUM Operations (10 minutes)

### Objective
(For RHEL/CentOS systems - use a VM or container if needed)

### Setup (if using Ubuntu)
```bash
# Run a CentOS container for practice
docker run -it --name centos-practice centos:stream9 bash
```

### Tasks

```bash
# Task 4.1: Check for updates
dnf check-update

# Task 4.2: Install a package
dnf install -y nginx

# Task 4.3: Show package info
dnf info nginx

# Task 4.4: List files in package
rpm -ql nginx

# Task 4.5: Find what provides a file
rpm -qf /usr/sbin/nginx

# Task 4.6: Show package dependencies
rpm -qR nginx

# Task 4.7: Check transaction history
dnf history

# Task 4.8: Enable EPEL repository
dnf install -y epel-release
dnf repolist

# Task 4.9: Install from EPEL
dnf install -y htop

# Task 4.10: Clean up
dnf remove nginx htop
dnf autoremove -y
dnf clean all
```

---

## Exercise 5: Compiling from Source (20 minutes)

### Objective
Compile htop from source with custom options.

### Tasks

```bash
# Task 5.1: Install build dependencies
sudo apt update
sudo apt install -y build-essential autoconf automake \
    libncurses5-dev libncursesw5-dev

# Task 5.2: Download htop source
cd /tmp
wget https://github.com/htop-dev/htop/releases/download/3.3.0/htop-3.3.0.tar.xz

# Task 5.3: Extract
tar -xJf htop-3.3.0.tar.xz
cd htop-3.3.0

# Task 5.4: Check configuration options
./configure --help | head -50

# Task 5.5: Configure with custom prefix
./configure --prefix=/opt/htop-custom

# Task 5.6: Compile (parallel jobs)
make -j$(nproc)

# Task 5.7: Install
sudo make install

# Task 5.8: Verify installation
/opt/htop-custom/bin/htop --version

# Task 5.9: Add to PATH temporarily
export PATH=/opt/htop-custom/bin:$PATH
htop

# Task 5.10: Create wrapper script for system-wide use
sudo tee /usr/local/bin/htop-custom << 'EOF'
#!/bin/bash
/opt/htop-custom/bin/htop "$@"
EOF
sudo chmod +x /usr/local/bin/htop-custom

# Task 5.11: Clean up source
cd /tmp
rm -rf htop-3.3.0 htop-3.3.0.tar.xz
```

---

## Exercise 6: Troubleshooting Challenge (15 minutes)

### Objective
Fix a broken package situation.

### Scenario Setup

```bash
# Create a broken state (simulate)
# Download a package with unmet dependencies

# Download VS Code (has dependencies)
wget -O /tmp/code.deb "https://code.visualstudio.com/sha/download?build=stable&os=linux-deb-x64"

# Attempt install (will likely fail with deps)
sudo dpkg -i /tmp/code.deb
```

### Troubleshooting Tasks

```bash
# Task 6.1: Check the broken state
dpkg -l | grep -E "^.i"
# Look for 'iU' or 'iF' states

# Task 6.2: Audit for problems
dpkg --audit

# Task 6.3: Fix broken dependencies
sudo apt install -f

# Task 6.4: Configure pending packages
sudo dpkg --configure -a

# Task 6.5: Verify fix
dpkg -l | grep code

# Task 6.6: Test the application
code --version

# Task 6.7: Clean up if needed
sudo apt purge code
sudo apt autoremove -y
```

---

## Exercise 7: Cross-Distribution Challenge (10 minutes)

### Objective
Write a script that works on both Debian and RHEL systems.

### Task: Create Universal Package Installer

```bash
#!/bin/bash
# save as: universal-installer.sh

set -e

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

# Detect OS
detect_os() {
    if [ -f /etc/os-release ]; then
        . /etc/os-release
        OS=$ID
        VERSION=$VERSION_ID
    elif [ -f /etc/redhat-release ]; then
        OS="rhel"
    elif [ -f /etc/debian_version ]; then
        OS="debian"
    else
        OS="unknown"
    fi
    echo $OS
}

# Detect package manager
detect_pkg_manager() {
    if command -v apt &> /dev/null; then
        echo "apt"
    elif command -v dnf &> /dev/null; then
        echo "dnf"
    elif command -v yum &> /dev/null; then
        echo "yum"
    else
        echo "unknown"
    fi
}

# Update system
update_system() {
    local pm=$1
    echo -e "${YELLOW}Updating system with $pm...${NC}"
    
    case $pm in
        apt)
            sudo apt update
            ;;
        dnf|yum)
            sudo $pm check-update || true
            ;;
    esac
}

# Install package
install_package() {
    local pm=$1
    local package=$2
    
    echo -e "${YELLOW}Installing $package with $pm...${NC}"
    
    case $pm in
        apt)
            sudo apt install -y "$package"
            ;;
        dnf|yum)
            sudo $pm install -y "$package"
            ;;
    esac
}

# Main
main() {
    local OS=$(detect_os)
    local PM=$(detect_pkg_manager)
    
    echo -e "${GREEN}Detected OS: $OS${NC}"
    echo -e "${GREEN}Package Manager: $PM${NC}"
    
    if [ "$PM" == "unknown" ]; then
        echo -e "${RED}Unknown package manager. Exiting.${NC}"
        exit 1
    fi
    
    # Update
    update_system "$PM"
    
    # Install requested packages
    for pkg in "$@"; do
        install_package "$PM" "$pkg"
    done
    
    echo -e "${GREEN}All packages installed successfully!${NC}"
}

# Run with provided packages
main "$@"
```

### Test the Script

```bash
chmod +x universal-installer.sh

# Install test packages
./universal-installer.sh htop vim curl wget
```

---

# 🧠 MUSCLE MEMORY DRILLS

## Drill 1: Speed Installation (Repeat 5 times)

```bash
# Time yourself installing and removing nginx

time (
    sudo apt update
    sudo apt install -y nginx
    systemctl status nginx
    sudo apt purge -y nginx
    sudo apt autoremove -y
)
```

## Drill 2: Package Investigation (Repeat 3 times)

```bash
# For any package name you think of, find:
# 1. Is it installed?
# 2. What version?
# 3. What files does it include?
# 4. What are its dependencies?

PACKAGE="openssh-server"

# Check if installed
dpkg -l | grep $PACKAGE

# Show version and details
apt show $PACKAGE

# List files
dpkg -L $PACKAGE 2>/dev/null || apt-file list $PACKAGE

# Show dependencies
apt-cache depends $PACKAGE
```

## Drill 3: Troubleshooting Workflow

```bash
# When something is wrong with packages:

# Step 1: Check status
dpkg -l | grep -E "^.[^i]"

# Step 2: Audit
sudo dpkg --audit

# Step 3: Configure pending
sudo dpkg --configure -a

# Step 4: Fix dependencies
sudo apt install -f

# Step 5: Verify
apt list --installed | wc -l
```

---

# ⚠️ COMMON MISTAKES AND FIXES

## Mistake 1: Using `apt update` thinking it updates packages

```bash
# WRONG - This only refreshes the package list
sudo apt update

# RIGHT - This actually updates packages
sudo apt update && sudo apt upgrade
```

## Mistake 2: Not using `-y` in scripts

```bash
# WRONG - Script will hang waiting for input
apt install nginx

# RIGHT - Automatic yes for scripts
apt install -y nginx

# BETTER - Also check return code
apt install -y nginx || exit 1
```

## Mistake 3: Using `apt` in scripts

```bash
# WRONG - apt output may change between versions
apt install nginx >> /var/log/install.log

# RIGHT - apt-get has stable output for scripts
apt-get install -y nginx >> /var/log/install.log
```

## Mistake 4: Forgetting to fix dependencies after dpkg

```bash
# WRONG - Leave system in broken state
sudo dpkg -i package.deb
# (errors about missing deps)

# RIGHT - Always fix after dpkg
sudo dpkg -i package.deb
sudo apt install -f
```

## Mistake 5: Not removing old kernels

```bash
# System fills with old kernels
# Check current kernel
uname -r

# List installed kernels
dpkg -l | grep linux-image

# Safe removal of old kernels
sudo apt autoremove --purge
```

## Mistake 6: Adding PPAs without verification

```bash
# WRONG - Blindly trust any PPA
sudo add-apt-repository ppa:random-user/software

# RIGHT - Verify PPA authenticity first
# Check launchpad.net page for the PPA
# Verify maintainer reputation
# Read reviews and bug reports
# Then add with caution
```

---

# 🏆 FINAL ASSESSMENT QUIZ

Test your knowledge:

### Question 1
What's the difference between `apt upgrade` and `apt full-upgrade`?

<details>
<summary>Answer</summary>

`apt upgrade`: Upgrades packages but never removes packages or installs new dependencies.

`apt full-upgrade`: May remove packages and install new dependencies if needed for upgrades.
</details>

### Question 2
How do you find which package provides the file `/usr/bin/vim`?

<details>
<summary>Answer</summary>

```bash
# Debian/Ubuntu
dpkg -S /usr/bin/vim

# RHEL/CentOS
rpm -qf /usr/bin/vim
```
</details>

### Question 3
How do you prevent a package from being upgraded?

<details>
<summary>Answer</summary>

```bash
# Debian/Ubuntu
sudo apt-mark hold package-name

# RHEL/CentOS
sudo dnf versionlock add package-name
```
</details>

### Question 4
What does `./configure --prefix=/opt/custom` do?

<details>
<summary>Answer</summary>

Sets the installation directory for compiled software. Files will be installed under `/opt/custom` instead of the default `/usr/local`.
</details>

### Question 5
How do you undo the last DNF transaction?

<details>
<summary>Answer</summary>

```bash
dnf history
sudo dnf history undo TRANSACTION_ID
```
</details>

---

# 📚 ADDITIONAL RESOURCES

## Man Pages to Study
```bash
man apt
man apt-get
man apt-cache
man dpkg
man dnf
man rpm
man sources.list
```

## Configuration Files to Explore
```bash
# APT
/etc/apt/sources.list
/etc/apt/sources.list.d/
/etc/apt/preferences.d/
/etc/apt/apt.conf.d/

# DNF/YUM
/etc/yum.repos.d/
/etc/dnf/dnf.conf
/etc/yum.conf
```

---

# ✅ COMPLETION CHECKLIST

After completing this guide, you should be able to:

- [ ] Update and upgrade packages on any Linux system
- [ ] Install, remove, and purge packages
- [ ] Search for packages and view their details
- [ ] Work with .deb files using dpkg
- [ ] Work with .rpm files using rpm
- [ ] Add and manage repositories securely
- [ ] Compile software from source
- [ ] Fix broken package dependencies
- [ ] Hold packages to prevent updates
- [ ] View and rollback transaction history (DNF)
- [ ] Write cross-distribution package management scripts
- [ ] Troubleshoot common package problems

---

**🎯 You have completed Package Management Warfare!**