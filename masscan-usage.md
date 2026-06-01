# ⚡ Masscan — Complete Usage Guide

> Masscan is the fastest Internet port scanner. It can scan the entire Internet
> in under 6 minutes transmitting 10 million packets per second.
> This guide covers everything from installation to advanced use cases.

---

## 📑 Table of Contents

1. [What Makes Masscan Special](#1-what-makes-masscan-special)
2. [Installation](#2-installation)
3. [Basic Syntax](#3-basic-syntax)
4. [Rate Control](#4-rate-control)
5. [Port Specification](#5-port-specification)
6. [Target Specification](#6-target-specification)
7. [Output Formats](#7-output-formats)
8. [Advanced Options](#8-advanced-options)
9. [Real-World Use Cases](#9-real-world-use-cases)
10. [Masscan + Nmap Workflow](#10-masscan--nmap-workflow)
11. [Masscan vs Nmap Comparison](#11-masscan-vs-nmap-comparison)
12. [Common Issues & Fixes](#12-common-issues--fixes)

---

## 1. What Makes Masscan Special

```
Traditional scanners (Nmap):
  → Send packet → Wait for response → Record result → Next port
  → Sequential or limited parallel
  → ~1,000-10,000 ports/second

Masscan:
  → Sends millions of SYN packets at once
  → Uses asynchronous transmission (doesn't wait for replies)
  → Separate thread receives and records responses
  → Up to 10,000,000 packets/second

Result:
  Scanning all 65,535 ports on 1 host:
    Nmap:    ~2-5 minutes
    Masscan: ~1-2 seconds

  Scanning a /16 network (65,536 hosts):
    Nmap:    Days
    Masscan: Minutes
```

**Limitations to remember:**
- Only does TCP SYN and ICMP echo — no service detection
- Cannot detect service versions
- No OS fingerprinting
- No NSE scripting
- May miss ports on unstable networks (increase retries)
- Requires raw socket access (root/sudo required)

---

## 2. Installation

```bash
# ── KALI LINUX (usually pre-installed) ─────────────────────────
masscan --version
# Expected: Masscan version 1.3.2 (https://github.com/robertdavidgraham/masscan)


# ── BUILD FROM SOURCE (recommended for latest version) ─────────
# Install dependencies
sudo apt update
sudo apt install git gcc make libpcap-dev -y

# Clone the repository
git clone https://github.com/robertdavidgraham/masscan
cd masscan

# Compile
make

# Install system-wide
sudo make install

# Verify
masscan --version


# ── UBUNTU/DEBIAN ──────────────────────────────────────────────
sudo apt update
sudo apt install masscan -y


# ── VERIFY INSTALLATION ────────────────────────────────────────
sudo masscan --selftest
# Should output: "selftest passed"
```

---

## 3. Basic Syntax

```bash
# ── GENERAL SYNTAX ─────────────────────────────────────────────
# masscan [TARGET] -p [PORTS] [OPTIONS]

# ── SIMPLEST SCAN ──────────────────────────────────────────────
# Scan port 80 on a single IP
sudo masscan 192.168.1.1 -p 80

# Scan multiple ports
sudo masscan 192.168.1.1 -p 80,443,22

# Scan a port range
sudo masscan 192.168.1.1 -p 1-1000

# Scan ALL 65,535 ports
sudo masscan 192.168.1.1 -p 0-65535

# ── RATE SETTING (IMPORTANT) ───────────────────────────────────
# Always include --rate to control scan speed
sudo masscan 192.168.1.1 -p 0-65535 --rate=10000

# --rate=1000    → 1,000 packets/second  (safe, slow)
# --rate=10000   → 10,000 packets/sec    (good for single host)
# --rate=100000  → 100,000 packets/sec   (fast, network may struggle)
# --rate=1000000 → 1,000,000 packets/sec (very fast, use carefully)
```

---

## 4. Rate Control

```bash
# ── UNDERSTANDING RATE ─────────────────────────────────────────
# Rate = packets per second sent
# Higher rate = faster scan BUT:
#   - May overwhelm target (crash services, trigger IDS)
#   - May overwhelm your own network interface
#   - May miss responses (drop packets)
#   - May get you banned/blocked

# ── RECOMMENDED RATES BY SCENARIO ─────────────────────────────

# Single host — internal network (safe, thorough)
sudo masscan TARGET_IP -p 0-65535 --rate=10000
# Scans all 65,535 ports in ~7 seconds

# Single host — internet (careful)
sudo masscan TARGET_IP -p 0-65535 --rate=1000
# Scans all 65,535 ports in ~65 seconds

# Small subnet /24 — 256 hosts (internal)
sudo masscan 192.168.1.0/24 -p 80,443,22 --rate=5000

# Large subnet /16 — 65,536 hosts
sudo masscan 10.0.0.0/16 -p 80,443 --rate=10000

# Enterprise network (authorised, professional engagement)
sudo masscan 10.0.0.0/8 -p 80,443,22,3389 --rate=100000

# Maximum safe rate for home broadband
sudo masscan TARGET -p 0-65535 --rate=500
# 500 pps is safe for most home connections

# NEVER use these rates on internet targets without permission:
# --rate=10000000  # 10 million pps — can cause DoS


# ── BANDWIDTH CALCULATION ──────────────────────────────────────
# Each packet = ~40 bytes
# At --rate=10000: 10,000 × 40 = 400,000 bytes/sec = ~400 KB/s
# At --rate=100000: 100,000 × 40 = 4,000,000 bytes/sec = ~4 MB/s
# At --rate=1000000: 1,000,000 × 40 = ~40 MB/s (requires good connection)
```

---

## 5. Port Specification

```bash
# ── SINGLE PORT ────────────────────────────────────────────────
sudo masscan TARGET -p 80 --rate=1000

# ── MULTIPLE PORTS ─────────────────────────────────────────────
sudo masscan TARGET -p 80,443,22,21,3389 --rate=1000

# ── PORT RANGE ─────────────────────────────────────────────────
sudo masscan TARGET -p 1-1024 --rate=1000     # Well-known ports
sudo masscan TARGET -p 1024-65535 --rate=1000  # Registered + dynamic
sudo masscan TARGET -p 0-65535 --rate=1000    # ALL ports

# ── COMMON PORT GROUPS ─────────────────────────────────────────

# Top 20 most important ports
sudo masscan TARGET -p 21,22,23,25,53,80,110,111,135,139,143,443,445,993,995,1723,3306,3389,5900,8080 --rate=5000

# Web focused
sudo masscan TARGET -p 80,443,8080,8443,8000,8888,9000,3000,4443,4000 --rate=5000

# Database ports
sudo masscan TARGET -p 1433,1521,3306,5432,6379,27017,27018,5984,9200,9300 --rate=1000

# Remote access
sudo masscan TARGET -p 22,23,3389,5900,5901,5985,5986 --rate=1000

# Email servers
sudo masscan TARGET -p 25,110,143,465,587,993,995 --rate=1000

# File transfer
sudo masscan TARGET -p 20,21,22,69,115,989,990 --rate=1000

# ── UDP PORTS ──────────────────────────────────────────────────
# Note: Masscan UDP support is limited
sudo masscan TARGET -pU:53,161,162,67,68 --rate=500
```

---

## 6. Target Specification

```bash
# ── SINGLE IP ──────────────────────────────────────────────────
sudo masscan 192.168.1.1 -p 80 --rate=1000

# ── CIDR NOTATION ──────────────────────────────────────────────
sudo masscan 192.168.1.0/24 -p 80 --rate=1000   # 256 hosts
sudo masscan 10.0.0.0/16 -p 80 --rate=5000      # 65,536 hosts
sudo masscan 10.0.0.0/8 -p 80 --rate=10000      # 16,777,216 hosts

# ── IP RANGES ──────────────────────────────────────────────────
sudo masscan 192.168.1.1-192.168.1.254 -p 80 --rate=1000

# ── MULTIPLE TARGETS ───────────────────────────────────────────
sudo masscan 192.168.1.1 192.168.1.2 192.168.1.3 -p 80 --rate=1000

# ── FROM FILE ──────────────────────────────────────────────────
# Create targets.txt with one IP/CIDR per line
sudo masscan -iL targets.txt -p 80,443 --rate=1000

# ── EXCLUDE TARGETS ────────────────────────────────────────────
sudo masscan 192.168.1.0/24 -p 80 --rate=1000 --excludefile exclude.txt
# exclude.txt contains IPs/ranges to skip

# Exclude single IP (inline)
sudo masscan 192.168.1.0/24 -p 80 --rate=1000 --exclude 192.168.1.1

# ── INTERNET SCANNING (with permission/authorisation only) ─────
# Example of scanning a specific ASN's range
sudo masscan 203.0.113.0/24 -p 80,443 --rate=500
```

---

## 7. Output Formats

```bash
# ── LIST FORMAT (-oL) — Most readable ──────────────────────────
sudo masscan TARGET -p 0-65535 --rate=1000 -oL results.txt

# Output format:
# open tcp 80 192.168.1.1 1609459200
# open tcp 443 192.168.1.1 1609459200
# Fields: state, protocol, port, IP, timestamp
# BEST FOR: Quick review, grep parsing


# ── XML FORMAT (-oX) — For tool integration ────────────────────
sudo masscan TARGET -p 0-65535 --rate=1000 -oX results.xml

# Compatible with Nmap XML parsers
# Use with: Metasploit import, custom scripts


# ── JSON FORMAT (-oJ) — For scripting ──────────────────────────
sudo masscan TARGET -p 0-65535 --rate=1000 -oJ results.json

# Output:
# {"ip": "192.168.1.1", "timestamp": "1609459200", "ports": [{"port": 80, "proto": "tcp", "status": "open"}]}
# BEST FOR: Python scripts, automated processing


# ── GREPABLE FORMAT (-oG) — For grep/awk ───────────────────────
sudo masscan TARGET -p 0-65535 --rate=1000 -oG results.gnmap

# ── BINARY FORMAT (-oB) — Fastest, resume support ─────────────
sudo masscan TARGET -p 0-65535 --rate=1000 -oB results.bin
# Convert binary to list:
masscan --readscan results.bin -oL results.txt


# ── WRITE TO STDOUT (default) ──────────────────────────────────
sudo masscan TARGET -p 80 --rate=1000
# Results print directly to terminal


# ── PRACTICAL EXTRACTION COMMANDS ──────────────────────────────
# Extract all open ports as comma-separated list (for Nmap)
grep "open" results.txt | awk '{print $3}' | sort -n | tr '\n' ','

# Extract unique IPs with open ports
grep "open" results.txt | awk '{print $4}' | sort -u

# Count open ports found
grep "open" results.txt | wc -l

# Find all hosts with port 80 open
grep "open tcp 80" results.txt | awk '{print $4}'

# Find all hosts with port 443 open
grep "open tcp 443" results.txt | awk '{print $4}'
```

---

## 8. Advanced Options

```bash
# ── RESUME INTERRUPTED SCAN ────────────────────────────────────
# Masscan saves state to paused.conf automatically
# To resume:
sudo masscan --resume paused.conf

# To disable auto-save:
sudo masscan TARGET -p 0-65535 --rate=1000 --resume-index 0


# ── RETRY / WAIT ───────────────────────────────────────────────
# --retries = number of times to resend packets
sudo masscan TARGET -p 0-65535 --rate=1000 --retries 2
# Default is 1 retry
# Increase for unreliable networks

# --wait = seconds to wait after scan ends for late responses
sudo masscan TARGET -p 0-65535 --rate=1000 --wait 5
# Default is 10 seconds


# ── NETWORK INTERFACE ──────────────────────────────────────────
# -e = specify network interface
sudo masscan TARGET -p 80 --rate=1000 -e eth0
sudo masscan TARGET -p 80 --rate=1000 -e wlan0

# List available interfaces:
ip link show


# ── ROUTER MAC ─────────────────────────────────────────────────
# --router-mac = specify gateway MAC address
sudo masscan TARGET -p 80 --rate=1000 --router-mac AA:BB:CC:DD:EE:FF

# When to use: When Masscan can't auto-detect the router MAC


# ── SOURCE IP/PORT ─────────────────────────────────────────────
# --source-ip = spoof source IP (advanced)
sudo masscan TARGET -p 80 --rate=1000 --source-ip 192.168.1.100

# --source-port = specify source port range
sudo masscan TARGET -p 80 --rate=1000 --source-port 40000-50000


# ── CONFIGURATION FILE ─────────────────────────────────────────
# Save scan config to file
sudo masscan TARGET -p 80 --rate=1000 --echo > scan.conf

# Run from config file
sudo masscan -c scan.conf

# Example scan.conf:
cat << 'EOF' > scan.conf
rate = 1000
output-filename = results.txt
output-format = list
ports = 80,443,22,21
range = 192.168.1.0/24
EOF

sudo masscan -c scan.conf


# ── BANNER GRABBING (Limited) ──────────────────────────────────
# Masscan has very basic banner grabbing
sudo masscan TARGET -p 80 --rate=100 --banners

# Very limited compared to Nmap
# For proper banner grabbing, use Nmap after Masscan
```

---

## 9. Real-World Use Cases

```bash
# ══════════════════════════════════════════════════════════════
# USE CASE 1: Discover all open ports on a single target
# (First step in VAPT engagement)
# ══════════════════════════════════════════════════════════════
sudo masscan TARGET_IP -p 0-65535 --rate=10000 -oL all_ports.txt

echo "Open ports found:"
grep "open" all_ports.txt | awk '{print $3}' | sort -n


# ══════════════════════════════════════════════════════════════
# USE CASE 2: Quick web server discovery on a /24 subnet
# ══════════════════════════════════════════════════════════════
sudo masscan 192.168.1.0/24 -p 80,443,8080,8443 --rate=5000 -oL web_servers.txt

echo "Web servers found:"
grep "open" web_servers.txt | awk '{print $4}' | sort -u


# ══════════════════════════════════════════════════════════════
# USE CASE 3: Find all RDP and SSH servers (network audit)
# ══════════════════════════════════════════════════════════════
sudo masscan 10.0.0.0/16 -p 22,3389 --rate=50000 -oL remote_access.txt

echo "SSH servers (port 22):"
grep "open tcp 22" remote_access.txt | awk '{print $4}'

echo "RDP servers (port 3389):"
grep "open tcp 3389" remote_access.txt | awk '{print $4}'


# ══════════════════════════════════════════════════════════════
# USE CASE 4: Database exposure check
# ══════════════════════════════════════════════════════════════
sudo masscan 10.0.0.0/16 -p 3306,5432,1433,27017,6379,9200 --rate=10000 -oL databases.txt

echo "Potentially exposed databases:"
grep "open" databases.txt


# ══════════════════════════════════════════════════════════════
# USE CASE 5: Pre-engagement baseline scan
# Use before a VAPT to understand full scope
# ══════════════════════════════════════════════════════════════
# Scan all ports on entire target range
sudo masscan TARGET_RANGE -p 0-65535 --rate=5000 -oL full_baseline.txt

# Count total open port instances
echo "Total open ports found: $(grep -c "open" full_baseline.txt)"

# List unique IPs with any open port
echo "Unique hosts with open ports:"
grep "open" full_baseline.txt | awk '{print $4}' | sort -u | wc -l


# ══════════════════════════════════════════════════════════════
# USE CASE 6: Check if specific service is running anywhere
# ══════════════════════════════════════════════════════════════
# Check if any hosts have Telnet (port 23) open — security risk!
sudo masscan 192.168.0.0/16 -p 23 --rate=10000 -oL telnet_hosts.txt

if grep -q "open" telnet_hosts.txt; then
    echo "WARNING: Telnet (port 23) found open on these hosts:"
    grep "open" telnet_hosts.txt | awk '{print $4}'
else
    echo "Good: No Telnet services found"
fi
```

---

## 10. Masscan + Nmap Workflow

```bash
# ══════════════════════════════════════════════════════════════
# THE PROFESSIONAL WORKFLOW
# Step 1: Masscan (fast discovery)
# Step 2: Nmap (detailed analysis)
# ══════════════════════════════════════════════════════════════

TARGET="192.168.1.100"

# ── STEP 1: Masscan — Find ALL open ports fast ─────────────────
echo "[*] Step 1: Running Masscan to discover all open ports..."
sudo masscan $TARGET -p 0-65535 --rate=10000 -oL masscan_results.txt

echo "[+] Open ports discovered:"
grep "open" masscan_results.txt


# ── STEP 2: Extract open ports for Nmap ────────────────────────
echo "[*] Step 2: Extracting port list for Nmap..."
PORTS=$(grep "open" masscan_results.txt | awk '{print $3}' | sort -n | tr '\n' ',' | sed 's/,$//')
echo "[+] Ports to scan with Nmap: $PORTS"


# ── STEP 3: Nmap — Detailed scan on open ports only ────────────
echo "[*] Step 3: Running Nmap for detailed analysis..."
nmap -sV -sC -O -p $PORTS $TARGET -oA nmap_detailed


# ── STEP 4: NSE Vulnerability Scripts ──────────────────────────
echo "[*] Step 4: Running vulnerability scripts..."
nmap --script vuln -p $PORTS $TARGET -oA nmap_vulns


echo "[+] Scan complete! Check nmap_detailed.* and nmap_vulns.* for results"
```

**Save this as `vapt_scan.sh` and run it:**
```bash
chmod +x vapt_scan.sh
sudo ./vapt_scan.sh
```

---

## 11. Masscan vs Nmap Comparison

| Feature | Masscan | Nmap |
|---------|---------|------|
| Speed | ⭐⭐⭐⭐⭐ (10M pps) | ⭐⭐ (~1000 pps) |
| Service detection | ❌ No | ✅ Yes (-sV) |
| Version detection | ❌ No | ✅ Yes (-sV) |
| OS detection | ❌ No | ✅ Yes (-O) |
| NSE scripts | ❌ No | ✅ Yes (--script) |
| UDP scanning | ⚠️ Limited | ✅ Yes (-sU) |
| Banner grabbing | ⚠️ Basic | ✅ Detailed |
| Output formats | JSON, XML, List, Binary | Normal, XML, Grepable |
| Firewall evasion | ❌ Limited | ✅ Many options |
| Stealth | ❌ Noisy | ✅ Multiple modes |
| Large networks | ✅ Excellent | ⚠️ Slow |
| Single host deep scan | ❌ Surface only | ✅ Excellent |
| Root required | ✅ Yes | ⚠️ For most scans |
| Resume support | ✅ Yes | ❌ No |

---

## 12. Common Issues & Fixes

```bash
# ── ISSUE: "FAILED: setuid" ────────────────────────────────────
# Problem: Not running as root
# Fix:
sudo masscan TARGET -p 80 --rate=1000

# ── ISSUE: "FAILED: Failed to detect router MAC address" ───────
# Problem: Masscan can't find the gateway
# Fix: Specify it manually
ip neigh | head   # Find gateway MAC
sudo masscan TARGET -p 80 --rate=1000 --router-mac AA:BB:CC:DD:EE:FF

# ── ISSUE: Missing results / Too many dropped packets ──────────
# Problem: Rate too high for network/target
# Fix: Reduce rate and increase retries
sudo masscan TARGET -p 0-65535 --rate=1000 --retries 3

# ── ISSUE: Scan is very slow ───────────────────────────────────
# Problem: Rate too low
# Fix: Increase rate
sudo masscan TARGET -p 0-65535 --rate=50000

# ── ISSUE: "ERROR: Failed to find default interface" ───────────
# Problem: Multiple network interfaces
# Fix: Specify interface
sudo masscan TARGET -p 80 --rate=1000 -e eth0

# ── ISSUE: Getting blocked/banned ─────────────────────────────
# Problem: Rate too aggressive, target has IDS
# Fix: Reduce rate significantly
sudo masscan TARGET -p 0-65535 --rate=100 --wait 10

# ── ISSUE: Can't scan UDP ports ────────────────────────────────
# Problem: Masscan has limited UDP support
# Fix: Use Nmap for UDP scanning
nmap -sU -p 53,161,162,67,68,123 TARGET
```

---

*Author: Rishabh Sankhla | CEH v13 | TryHackMe Top 2% | Last Updated: May 2026*
