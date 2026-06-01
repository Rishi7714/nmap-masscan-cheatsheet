# 🗺️ Nmap Commands — Complete Reference

> All Nmap commands documented from real VAPT internship usage and TryHackMe practice.
> Every command includes an explanation of what it does and when to use it.

---

## 📑 Table of Contents

1. [Basic Scanning](#1-basic-scanning)
2. [Port Specification](#2-port-specification)
3. [Scan Types](#3-scan-types)
4. [Service & Version Detection](#4-service--version-detection)
5. [OS Detection](#5-os-detection)
6. [Timing & Performance](#6-timing--performance)
7. [Firewall & IDS Evasion](#7-firewall--ids-evasion)
8. [Output Formats](#8-output-formats)
9. [Ping & Host Discovery](#9-ping--host-discovery)
10. [Useful Combinations](#10-useful-combinations)

---

## 1. Basic Scanning

```bash
# ── SIMPLEST SCAN ──────────────────────────────────────────────
# Scans top 1000 most common ports using default settings
nmap target.com

# What happens:
# 1. Nmap pings the host to check if it's alive
# 2. Scans the 1000 most commonly used ports
# 3. Returns open/closed/filtered status for each
# When to use: First scan on any target — quick overview


# ── SCAN WITH VERSION DETECTION ────────────────────────────────
# -sV = detect service versions on open ports
nmap -sV target.com

# What it does:
# Sends additional probes to open ports to determine:
# - Service name (HTTP, SSH, FTP)
# - Software name (Apache, OpenSSH)
# - Version number (Apache 2.4.50)
# When to use: After finding open ports — to identify software versions for CVE lookup
# Output example: 80/tcp open http Apache httpd 2.4.50


# ── SCAN WITH DEFAULT SCRIPTS ──────────────────────────────────
# -sC = run default NSE scripts
nmap -sC target.com

# What it does:
# Runs a collection of "safe" scripts that gather extra info:
# - Banner grabbing
# - SSL certificate info
# - HTTP title of web pages
# - SSH host key fingerprint
# When to use: Alongside -sV for richer info gathering


# ── VERSION + DEFAULT SCRIPTS (MOST COMMON COMBINATION) ────────
nmap -sV -sC target.com

# This is the bread-and-butter scan used in 90% of engagements
# Gives you: open ports + service names + versions + extra info
# When to use: Standard first detailed scan on a target
```

---

## 2. Port Specification

```bash
# ── SPECIFIC PORTS ─────────────────────────────────────────────
# Scan only port 80
nmap -p 80 target.com

# Scan specific ports
nmap -p 80,443,22,21 target.com

# Scan a port range
nmap -p 1-1000 target.com

# Scan ports 1 through 65535 (all ports)
nmap -p 1-65535 target.com
# Or shorthand:
nmap -p- target.com

# Scan only port 443 with version detection
nmap -p 443 -sV target.com


# ── TOP PORTS ──────────────────────────────────────────────────
# Scan top 100 most common ports (faster than default 1000)
nmap --top-ports 100 target.com

# Scan top 500 ports
nmap --top-ports 500 target.com

# Scan top 10 ports (ultra quick)
nmap --top-ports 10 target.com


# ── FAST SCAN ──────────────────────────────────────────────────
# -F = Fast mode (top 100 ports only)
nmap -F target.com
# When to use: Quick check when time is limited


# ── EXCLUDE PORTS ──────────────────────────────────────────────
# Scan all except port 80
nmap -p- --exclude-ports 80 target.com

# Exclude a range
nmap -p- --exclude-ports 1-1024 target.com


# ── COMMON PORT GROUPS FOR WEB TARGETS ─────────────────────────
# Web-focused scan
nmap -p 80,443,8080,8443,8000,8888,3000,4000,5000 target.com

# Full infrastructure scan
nmap -p 21,22,23,25,53,80,110,143,443,445,3306,3389,5432,6379,8080,8443,27017 target.com
```

---

## 3. Scan Types

```bash
# ── TCP SYN SCAN (STEALTH SCAN) ────────────────────────────────
# -sS = SYN scan (half-open scan)
nmap -sS target.com

# How it works:
# 1. Send SYN packet
# 2. If port open → receive SYN-ACK → send RST (never complete handshake)
# 3. If port closed → receive RST
# Why "stealth": Never completes the TCP handshake, so many older IDS miss it
# Requires: Root/sudo privileges
# When to use: Stealthy scanning on authorised engagements
sudo nmap -sS target.com


# ── TCP CONNECT SCAN ───────────────────────────────────────────
# -sT = full TCP connect scan
nmap -sT target.com

# How it works:
# Completes the full TCP 3-way handshake
# Less stealthy than SYN scan but doesn't require root
# When to use: When you don't have root privileges
# Note: More likely to be logged by target systems


# ── UDP SCAN ───────────────────────────────────────────────────
# -sU = UDP scan
nmap -sU target.com

# Scans UDP ports instead of TCP
# Important services on UDP: DNS (53), DHCP (67/68), SNMP (161), NTP (123)
# Much slower than TCP scans
# When to use: Check for UDP services (DNS, SNMP often missed)
sudo nmap -sU -p 53,67,68,123,161,162 target.com

# Combine TCP + UDP scan
nmap -sS -sU target.com


# ── NULL, FIN, XMAS SCANS (STEALTH) ───────────────────────────
# -sN = NULL scan (no flags set)
nmap -sN target.com

# -sF = FIN scan (only FIN flag)
nmap -sF target.com

# -sX = Xmas scan (FIN, PSH, URG flags set)
nmap -sX target.com

# How they work:
# Send packets with unusual flag combinations
# Open ports: usually no response
# Closed ports: RST response
# Useful for: Bypassing some packet filters
# Note: Don't work on Windows targets (Windows always sends RST)


# ── PING SCAN (HOST DISCOVERY ONLY) ───────────────────────────
# -sn = No port scan (ping only — check if hosts are alive)
nmap -sn 192.168.1.0/24

# Use cases:
# Discover live hosts on a network without port scanning
# Quick network inventory
# Output: List of IP addresses that responded


# ── ACK SCAN (FIREWALL MAPPING) ───────────────────────────────
# -sA = ACK scan
nmap -sA target.com

# How it works:
# Sends ACK packets to determine if ports are filtered or unfiltered
# Cannot determine if port is open (useful for firewall rule analysis)
# When to use: Map firewall rules, not for finding open ports


# ── IDLE SCAN (ADVANCED STEALTH) ──────────────────────────────
# -sI = Idle scan (uses a zombie host)
# Very stealthy — your IP never appears in target's logs
# Requires finding a suitable zombie host
nmap -sI ZOMBIE_HOST target.com
```

---

## 4. Service & Version Detection

```bash
# ── VERSION DETECTION ──────────────────────────────────────────
# -sV = service/version detection
nmap -sV target.com

# Version intensity (0=light, 9=most comprehensive)
nmap -sV --version-intensity 5 target.com   # Default is 7
nmap -sV --version-intensity 9 target.com   # Most thorough
nmap -sV --version-intensity 0 target.com   # Fastest (light)
nmap -sV --version-light target.com         # Same as intensity 2
nmap -sV --version-all target.com           # Same as intensity 9

# Version detection on specific port
nmap -sV -p 443 target.com
# Example output:
# 443/tcp open ssl/http Apache httpd 2.4.50 ((Ubuntu))

# Version detection with banner grabbing
nmap -sV --version-intensity 7 -p 22 target.com
# Example output:
# 22/tcp open ssh OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)


# ── AGGRESSIVE DETECTION ───────────────────────────────────────
# -A = Aggressive (OS + version + scripts + traceroute)
nmap -A target.com

# Equivalent to running:
# nmap -O -sV -sC --traceroute target.com
# When to use: Most thorough single-command scan on authorised targets
# Takes longer but gives maximum information


# ── READING VERSION OUTPUT ─────────────────────────────────────
# PORT    STATE  SERVICE   VERSION
# 22/tcp  open   ssh       OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
# 80/tcp  open   http      Apache httpd 2.4.29 ((Ubuntu))
# 443/tcp open   ssl/http  Apache httpd 2.4.29
# 3306/tcp open  mysql     MySQL 5.7.32

# HOW TO USE VERSION INFO:
# 1. Copy version string
# 2. Search: "Apache 2.4.29 CVE" on NVD or ExploitDB
# 3. Check if version is in the vulnerable range
# 4. Document in report with CVE reference
```

---

## 5. OS Detection

```bash
# ── BASIC OS DETECTION ─────────────────────────────────────────
# -O = OS detection
nmap -O target.com

# Requires: Root/sudo
# How it works:
# Sends carefully crafted packets and analyses responses
# Looks at TCP/IP fingerprints (TTL values, window size, etc.)
# Compares against Nmap's OS database (thousands of OS fingerprints)
sudo nmap -O target.com

# Example output:
# Running: Linux 4.X|5.X
# OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
# OS details: Linux 4.15 - 5.8
# Network Distance: 1 hop


# ── OS + VERSION (COMBINED) ────────────────────────────────────
nmap -O -sV target.com

# Gives you: OS fingerprint + service versions
# Best combo for full host profiling


# ── AGGRESSIVE OS DETECTION ────────────────────────────────────
# --osscan-guess = Guess OS even if not 100% confident
nmap -O --osscan-guess target.com

# --osscan-limit = Only attempt OS detection on promising hosts
nmap -O --osscan-limit target.com


# ── WHY OS DETECTION MATTERS ───────────────────────────────────
# Windows Server 2008 → Check for EternalBlue (MS17-010)
# Ubuntu 18.04 → Check for specific kernel exploits
# CentOS 6 → Many outdated packages, check for Shellshock
# FreeBSD → Different exploit landscape
# Embedded Linux → Often has default credentials
```

---

## 6. Timing & Performance

```bash
# ── TIMING TEMPLATES ───────────────────────────────────────────
# Nmap has 6 timing templates (T0 through T5)

# T0 — Paranoid (slowest, max stealth)
nmap -T0 target.com
# Sends one packet every 5 minutes
# Use: High-security targets, IDS evasion
# Time for 1000 ports: Hours

# T1 — Sneaky (slow, stealthy)
nmap -T1 target.com
# Sends one packet every 15 seconds
# Use: Evading basic IDS
# Time for 1000 ports: ~15-30 minutes

# T2 — Polite (slower, less bandwidth)
nmap -T2 target.com
# Slower, uses less bandwidth
# Reduces chance of overwhelming target
# Time for 1000 ports: ~5 minutes

# T3 — Normal (DEFAULT)
nmap target.com   # T3 is default if no -T specified
# Balanced speed and reliability
# Time for 1000 ports: ~1-2 minutes

# T4 — Aggressive (fast, recommended for local networks)
nmap -T4 target.com
# Assumes fast, reliable network
# Good for internal network scans
# Time for 1000 ports: ~30 seconds
# MOST COMMONLY USED in CTFs and practice

# T5 — Insane (fastest, may miss results)
nmap -T5 target.com
# May miss open ports due to speed
# Use only on very fast networks
# Time for 1000 ports: ~10 seconds
# WARNING: May trigger IDS/IPS immediately


# ── MANUAL TIMING CONTROL ──────────────────────────────────────
# --min-rate = minimum packets per second
nmap --min-rate 1000 target.com

# --max-rate = maximum packets per second
nmap --max-rate 500 target.com

# --min-parallelism = minimum parallel probes
nmap --min-parallelism 100 target.com

# --max-parallelism = maximum parallel probes
nmap --max-parallelism 1 target.com   # Only 1 probe at a time (safe)

# --scan-delay = wait time between probes
nmap --scan-delay 500ms target.com

# --max-scan-delay = max wait time
nmap --max-scan-delay 1s target.com

# Combination for slow, careful scan
nmap -T2 --max-rate 100 --scan-delay 200ms target.com

# Combination for fast scan
nmap -T4 --min-rate 5000 target.com
```

---

## 7. Firewall & IDS Evasion

```bash
# ── FRAGMENTATION ──────────────────────────────────────────────
# -f = fragment packets (split into 8-byte fragments)
nmap -f target.com

# --mtu = set custom MTU (must be multiple of 8)
nmap --mtu 16 target.com
nmap --mtu 24 target.com

# Why: Some firewalls/IDS can't reassemble fragmented packets fast enough


# ── DECOY SCANNING ─────────────────────────────────────────────
# -D = use decoy IPs (scan appears to come from multiple sources)
nmap -D RND:10 target.com          # 10 random decoys
nmap -D 1.2.3.4,5.6.7.8 target.com # Specific decoy IPs
nmap -D ME,1.2.3.4,5.6.7.8 target.com # Include your real IP too

# Warning: Your real IP is still in the mix with RND
# Use with caution — can cause issues with innocent IPs


# ── SPOOF SOURCE ADDRESS ───────────────────────────────────────
# -S = spoof source IP
nmap -S SPOOF_IP -e eth0 target.com

# Note: You won't see results (responses go to spoofed IP)
# Use case: IDS evasion when combined with idle scan


# ── CUSTOM SOURCE PORT ─────────────────────────────────────────
# --source-port = use specific source port
nmap --source-port 53 target.com   # Appears to come from DNS
nmap --source-port 80 target.com   # Appears to be HTTP traffic

# Why: Some firewalls allow traffic from port 53 (DNS) through


# ── RANDOMISE HOST ORDER ───────────────────────────────────────
# --randomize-hosts = scan hosts in random order
nmap --randomize-hosts 192.168.1.0/24

# Why: Avoids sequential scanning patterns that IDS detect


# ── SKIP PING ──────────────────────────────────────────────────
# -Pn = skip host discovery (assume host is up)
nmap -Pn target.com

# When to use:
# Target blocks ICMP ping but ports may be open
# Firewall drops ping but allows port traffic
# Common in CTF environments


# ── BYPASS FIREWALLS WITH ACK SCAN ────────────────────────────
nmap -sA target.com

# Some stateless firewalls allow ACK packets through
# Can reveal which ports are "unfiltered"
```

---

## 8. Output Formats

```bash
# ── NORMAL OUTPUT ──────────────────────────────────────────────
# -oN = Normal output (human readable)
nmap -oN scan_results.txt target.com

# The default format you see in terminal — save it to a file
# Includes: open ports, services, timing info


# ── XML OUTPUT ─────────────────────────────────────────────────
# -oX = XML output (machine readable)
nmap -oX scan_results.xml target.com

# Best for: Importing into other tools (Metasploit, Nessus, etc.)
# Can be converted to HTML with:
xsltproc scan_results.xml -o scan_results.html


# ── GREPABLE OUTPUT ────────────────────────────────────────────
# -oG = Grepable output
nmap -oG scan_results.gnmap target.com

# Best for: Parsing with grep, awk, cut
# Extract all open ports:
grep "open" scan_results.gnmap
# Extract specific service:
grep "ssh" scan_results.gnmap | awk '{print $2}'


# ── SAVE ALL FORMATS AT ONCE ───────────────────────────────────
# -oA = Output in all 3 formats simultaneously
nmap -oA scan_results target.com

# Creates 3 files:
# scan_results.nmap  (normal)
# scan_results.xml   (XML)
# scan_results.gnmap (grepable)
# BEST PRACTICE: Always use -oA to preserve all formats


# ── VERBOSE OUTPUT ─────────────────────────────────────────────
# -v = verbose (show more info during scan)
nmap -v target.com

# -vv = very verbose
nmap -vv target.com

# Shows: Real-time port status, scan progress, timing info
# When to use: Long scans where you want live feedback


# ── REASON OUTPUT ──────────────────────────────────────────────
# --reason = show why port state was determined
nmap --reason target.com

# Example output:
# PORT   STATE SERVICE REASON
# 22/tcp open  ssh     syn-ack ttl 64
# 80/tcp open  http    syn-ack ttl 64
# When to use: Debugging scan results, understanding firewall behaviour


# ── PACKET TRACE ───────────────────────────────────────────────
# --packet-trace = show all packets sent and received
nmap --packet-trace target.com

# Very verbose — shows every single packet
# When to use: Deep debugging, learning how Nmap works


# ── OPEN PORTS ONLY ────────────────────────────────────────────
# --open = only show open ports (hide closed and filtered)
nmap --open target.com

# Much cleaner output when scanning many ports
# When to use: Large port ranges — you only care about what's open
```

---

## 9. Ping & Host Discovery

```bash
# ── PING SCAN ONLY (NO PORT SCAN) ─────────────────────────────
# -sn = Ping scan (no port scan)
nmap -sn 192.168.1.0/24

# Sends:
# ICMP echo request
# TCP SYN to port 443
# TCP ACK to port 80
# ICMP timestamp request
# Shows: Which hosts are alive


# ── SKIP PING CHECK ────────────────────────────────────────────
# -Pn = Treat all hosts as online (skip ping)
nmap -Pn target.com

# When to use:
# Target blocks ICMP (ping blocked by firewall)
# You know the host is up but ping doesn't work


# ── CUSTOM PING TYPES ──────────────────────────────────────────
# -PE = ICMP echo ping
nmap -PE -sn target.com

# -PP = ICMP timestamp ping
nmap -PP -sn target.com

# -PM = ICMP netmask ping
nmap -PM -sn target.com

# -PS = TCP SYN ping (specify ports)
nmap -PS80,443 -sn target.com
nmap -PS22,80,443,8080 -sn 192.168.1.0/24

# -PA = TCP ACK ping
nmap -PA80 -sn target.com

# -PU = UDP ping (specify ports)
nmap -PU53,161 -sn target.com


# ── SCAN MULTIPLE TARGETS ──────────────────────────────────────
# Multiple IPs
nmap 192.168.1.1 192.168.1.2 192.168.1.3

# IP range
nmap 192.168.1.1-254

# CIDR notation
nmap 192.168.1.0/24
nmap 10.0.0.0/8

# From a file
nmap -iL targets.txt

# Exclude specific IPs
nmap 192.168.1.0/24 --exclude 192.168.1.1
nmap 192.168.1.0/24 --excludefile excluded.txt

# Random targets
nmap -iR 10   # Scan 10 random internet hosts
```

---

## 10. Useful Combinations

```bash
# ══════════════════════════════════════════════════════════════
# COMBINATION 1 — Standard Web Server Scan
# Best for: HTTP/HTTPS service analysis
# ══════════════════════════════════════════════════════════════
nmap -sV -sC -p 80,443,8080,8443 target.com

# What you get:
# - Service versions on web ports
# - HTTP page titles
# - SSL certificate information
# - HTTP methods allowed
# - Redirect chains


# ══════════════════════════════════════════════════════════════
# COMBINATION 2 — Full Port + Version Scan
# Best for: Complete port inventory of a single host
# ══════════════════════════════════════════════════════════════
nmap -p- -sV -T4 target.com -oA full_scan

# What you get:
# - All 65,535 ports checked
# - Version info on every open port
# - Saved in all 3 output formats
# Time: 15–45 minutes depending on host


# ══════════════════════════════════════════════════════════════
# COMBINATION 3 — Aggressive Everything
# Best for: Authorised targets where you want maximum info
# ══════════════════════════════════════════════════════════════
nmap -A -T4 target.com -oA aggressive_scan

# Includes: OS detection + Version detection + Scripts + Traceroute
# Most comprehensive single command


# ══════════════════════════════════════════════════════════════
# COMBINATION 4 — Stealth Scan
# Best for: Evading IDS/IPS on authorised engagements
# ══════════════════════════════════════════════════════════════
sudo nmap -sS -T2 -f --scan-delay 500ms target.com

# SYN scan + slow timing + fragmentation + delay


# ══════════════════════════════════════════════════════════════
# COMBINATION 5 — After Masscan (detailed follow-up)
# Best for: Getting service details on ports Masscan found
# ══════════════════════════════════════════════════════════════
# Step 1 — Extract ports from Masscan output
PORTS=$(grep "open" masscan_results.txt | awk '{print $3}' | sort -u | tr '\n' ',')

# Step 2 — Run Nmap on just those ports
nmap -sV -sC -p $PORTS target.com -oA nmap_targeted


# ══════════════════════════════════════════════════════════════
# COMBINATION 6 — Internal Network Sweep
# Best for: Mapping an entire internal network
# ══════════════════════════════════════════════════════════════
# Phase 1: Find live hosts
nmap -sn 192.168.1.0/24 -oG live_hosts.gnmap

# Phase 2: Extract live IPs
grep "Up" live_hosts.gnmap | awk '{print $2}' > live_ips.txt

# Phase 3: Scan live hosts for common ports
nmap -iL live_ips.txt -p 21,22,23,25,80,443,445,3306,3389,8080 -sV -oA internal_scan


# ══════════════════════════════════════════════════════════════
# COMBINATION 7 — Vulnerability Scan
# Best for: Checking for known CVEs on open services
# ══════════════════════════════════════════════════════════════
nmap --script vuln -sV target.com -oA vuln_scan

# Runs ALL vulnerability scripts
# WARNING: Can be noisy and intrusive — only on authorised targets


# ══════════════════════════════════════════════════════════════
# COMBINATION 8 — CTF / TryHackMe Standard Scan
# Best for: Quickly getting all info on a CTF box
# ══════════════════════════════════════════════════════════════
nmap -sV -sC -A -T4 -p- TARGET_IP -oA ctf_scan

# Everything in one command for CTF environments


# ══════════════════════════════════════════════════════════════
# COMBINATION 9 — DNS Server Analysis
# Best for: Checking DNS server configuration
# ══════════════════════════════════════════════════════════════
nmap -sU -sV -p 53 --script dns-recursion,dns-zone-transfer target.com


# ══════════════════════════════════════════════════════════════
# COMBINATION 10 — SMB Analysis (Windows targets)
# Best for: Windows network share and vulnerability analysis
# ══════════════════════════════════════════════════════════════
nmap -p 445 --script smb-vuln-ms17-010,smb-security-mode,smb-enum-shares target.com
```

---

*Author: Rishabh Sankhla | CEH v13 | Last Updated: May 2026*
