**# 📜 Nmap NSE Scripts — Complete Reference

> The Nmap Scripting Engine (NSE) is one of Nmap's most powerful features.
> It allows users to write and share scripts to automate a wide variety
> of networking tasks including vulnerability detection, exploitation, and more.

---

## 📑 Table of Contents

1. [What is NSE?](#1-what-is-nse)
2. [Script Categories](#2-script-categories)
3. [How to Run Scripts](#3-how-to-run-scripts)
4. [HTTP & Web Scripts](#4-http--web-scripts)
5. [SSL/TLS Scripts](#5-ssltls-scripts)
6. [Authentication Scripts](#6-authentication-scripts)
7. [Vulnerability Scripts](#7-vulnerability-scripts)
8. [SMB/Windows Scripts](#8-smbwindows-scripts)
9. [Database Scripts](#9-database-scripts)
10. [DNS Scripts](#10-dns-scripts)
11. [Discovery Scripts](#11-discovery-scripts)
12. [Script Arguments](#12-script-arguments)
13. [Quick Reference by Use Case](#13-quick-reference-by-use-case)

---

## 1. What is NSE?

```
NSE (Nmap Scripting Engine) allows Nmap to run Lua scripts
that extend its functionality far beyond simple port scanning.

Scripts can:
  ✓ Detect specific vulnerabilities (Heartbleed, EternalBlue, etc.)
  ✓ Enumerate services (list SMB shares, DNS records)
  ✓ Brute-force authentication
  ✓ Detect malware/backdoors
  ✓ Gather information (HTTP headers, SSL certs, etc.)
  ✓ Run exploits (with exploit category scripts)

Script location on Kali:
  /usr/share/nmap/scripts/

Count all available scripts:
  ls /usr/share/nmap/scripts/*.nse | wc -l
  # ~600+ scripts available

Update script database:
  sudo nmap --script-updatedb
```

---

## 2. Script Categories

| Category | Description | Safety |
|----------|-------------|--------|
| `auth` | Authentication bypass/testing | Safe |
| `broadcast` | Network broadcast discovery | Safe |
| `brute` | Password brute-forcing | Intrusive |
| `default` | Safe, useful scripts (run with -sC) | Safe |
| `discovery` | Service enumeration | Safe |
| `dos` | Denial of service testing | Dangerous |
| `exploit` | Active exploitation | Dangerous |
| `external` | Use external resources | Safe |
| `fuzzer` | Fuzz testing | Intrusive |
| `intrusive` | Intrusive checks | Intrusive |
| `malware` | Malware detection | Safe |
| `safe` | Won't crash services | Safe |
| `version` | Version detection enhancement | Safe |
| `vuln` | Vulnerability detection | Intrusive |

```bash
# Run an entire category
nmap --script default TARGET        # Safe default scripts
nmap --script safe TARGET           # All safe scripts
nmap --script discovery TARGET      # All discovery scripts
nmap --script vuln TARGET           # All vulnerability scripts
nmap --script "not intrusive" TARGET # All except intrusive
```

---

## 3. How to Run Scripts

```bash
# ── RUN DEFAULT SCRIPTS ────────────────────────────────────────
# -sC is shorthand for --script=default
nmap -sC TARGET
nmap --script=default TARGET   # Same as above


# ── RUN A SPECIFIC SCRIPT ──────────────────────────────────────
nmap --script SCRIPT_NAME TARGET


# ── RUN MULTIPLE SPECIFIC SCRIPTS ─────────────────────────────
nmap --script "script1,script2,script3" TARGET


# ── RUN ALL SCRIPTS IN A CATEGORY ─────────────────────────────
nmap --script CATEGORY TARGET
nmap --script vuln TARGET
nmap --script "http-*" TARGET    # All scripts starting with http-


# ── RUN SCRIPTS MATCHING A PATTERN ────────────────────────────
nmap --script "ssl-*" TARGET     # All SSL scripts
nmap --script "smb-*" TARGET     # All SMB scripts
nmap --script "http-*" TARGET    # All HTTP scripts


# ── COMBINE CATEGORIES ─────────────────────────────────────────
nmap --script "default or safe" TARGET
nmap --script "default and safe" TARGET
nmap --script "vuln and not intrusive" TARGET


# ── EXCLUDE SPECIFIC SCRIPTS ───────────────────────────────────
nmap --script "default and not http-brute" TARGET


# ── SCRIPT WITH SPECIFIC PORT ──────────────────────────────────
nmap --script ssl-heartbleed -p 443 TARGET
nmap --script ssh-auth-methods -p 22 TARGET


# ── SCRIPT HELP ────────────────────────────────────────────────
nmap --script-help SCRIPT_NAME
nmap --script-help ssl-heartbleed
nmap --script-help "http-*"          # Help for all http scripts


# ── LIST ALL AVAILABLE SCRIPTS ─────────────────────────────────
ls /usr/share/nmap/scripts/ | grep ".nse"
# Or search for scripts by keyword:
ls /usr/share/nmap/scripts/ | grep "ssl"
ls /usr/share/nmap/scripts/ | grep "http"
ls /usr/share/nmap/scripts/ | grep "smb"
```

---

## 4. HTTP & Web Scripts

```bash
# ── HTTP TITLE ─────────────────────────────────────────────────
nmap --script http-title TARGET
# Gets the title of the web page
# Output: |_http-title: Welcome to Example.com
# Use: Quickly identify what's running on web servers


# ── HTTP HEADERS ───────────────────────────────────────────────
nmap --script http-headers TARGET
# Retrieves HTTP headers from the web server
# Output: Server, Content-Type, X-Powered-By, etc.
# Use: Find server version info, missing security headers
nmap --script http-headers -p 80,443 TARGET


# ── HTTP METHODS ───────────────────────────────────────────────
nmap --script http-methods TARGET
# Lists all HTTP methods the server accepts
# Output: GET, POST, HEAD, OPTIONS, PUT, DELETE, TRACE
# Use: Finding dangerous methods (PUT, DELETE, TRACE enabled = risk)
nmap --script http-methods -p 80,443 TARGET


# ── HTTP AUTH FINDER ───────────────────────────────────────────
nmap --script http-auth-finder TARGET
# Detects authentication mechanisms used
# Output: Basic, Digest, NTLM auth types
nmap --script http-auth-finder -p 80,443 TARGET


# ── HTTP ROBOTS.TXT ────────────────────────────────────────────
nmap --script http-robots.txt TARGET
# Retrieves and displays robots.txt
# Use: Discover hidden paths the site doesn't want indexed
nmap --script http-robots.txt -p 80,443 TARGET


# ── HTTP SITEMAP GENERATOR ─────────────────────────────────────
nmap --script http-sitemap-generator TARGET
# Crawls website and generates a sitemap
# Use: Discover all accessible pages
nmap --script http-sitemap-generator -p 80 TARGET


# ── HTTP BACKUP FINDER ─────────────────────────────────────────
nmap --script http-backup-finder TARGET
# Looks for backup files like .bak, .old, .orig, ~
# Checks for: index.php.bak, config.php.old, etc.
nmap --script http-backup-finder -p 80,443 TARGET


# ── HTTP CONFIG BACKUP ─────────────────────────────────────────
nmap --script http-config-backup TARGET
# Checks for common config backup files
nmap --script http-config-backup -p 80 TARGET


# ── WAF DETECTION ──────────────────────────────────────────────
nmap --script http-waf-detect TARGET
# Detects presence of a Web Application Firewall
# Sends requests with attack payloads and observes responses
nmap --script http-waf-detect -p 80,443 TARGET

nmap --script http-waf-fingerprint TARGET
# Identifies which WAF is in use (Cloudflare, ModSecurity, etc.)
nmap --script http-waf-fingerprint -p 80,443 TARGET


# ── HTTP SERVER HEADER ─────────────────────────────────────────
nmap --script http-server-header TARGET
# Returns the Server header from HTTP response
# Output: Apache/2.4.50 (Ubuntu)


# ── HTTP OPEN REDIRECT ─────────────────────────────────────────
nmap --script http-open-redirect TARGET
# Tests for open redirect vulnerabilities
nmap --script http-open-redirect -p 80,443 TARGET


# ── HTTP SQL INJECTION ─────────────────────────────────────────
nmap --script http-sql-injection TARGET
# Basic SQL injection detection
nmap --script http-sql-injection -p 80,443 TARGET


# ── HTTP XSS DETECTION ─────────────────────────────────────────
nmap --script http-stored-xss TARGET
# Detects stored XSS vulnerabilities
nmap --script http-stored-xss -p 80,443 TARGET


# ── HTTP DEFAULT CREDENTIALS ──────────────────────────────────
nmap --script http-default-accounts TARGET
# Tests for default username/password combinations
# Tests common defaults like admin/admin, admin/password
nmap --script http-default-accounts -p 80,443,8080 TARGET


# ── PHP INFO ──────────────────────────────────────────────────
nmap --script http-php-version TARGET
# Detects PHP version from headers or phpinfo() page
nmap --script http-php-version -p 80,443 TARGET


# ── WORDPRESS SCAN ────────────────────────────────────────────
nmap --script http-wordpress-enum TARGET
# Enumerates WordPress plugins and themes
nmap --script http-wordpress-enum -p 80,443 TARGET

nmap --script http-wordpress-users TARGET
# Enumerates WordPress usernames
nmap --script http-wordpress-users -p 80,443 TARGET


# ── DRUPAL DETECTION ──────────────────────────────────────────
nmap --script http-drupal-enum TARGET
# Enumerates Drupal modules
nmap --script http-drupal-enum -p 80 TARGET


# ── HTTP CROSS-DOMAIN ─────────────────────────────────────────
nmap --script http-cors TARGET
# Checks CORS (Cross-Origin Resource Sharing) configuration
nmap --script http-cors -p 80,443 TARGET


# ── FULL WEB SERVER SCAN ──────────────────────────────────────
# Combine multiple HTTP scripts
nmap --script "http-title,http-headers,http-methods,http-robots.txt,http-waf-detect,http-backup-finder" \
     -p 80,443,8080 TARGET
```

---

## 5. SSL/TLS Scripts

```bash
# ── SSL CERTIFICATE INFO ───────────────────────────────────────
nmap --script ssl-cert -p 443 TARGET
# Retrieves and displays SSL certificate details
# Output: Subject, Issuer, Validity dates, Subject Alt Names
# Use: Check cert validity, find subdomains in SANs
# Output example:
# | ssl-cert: Subject: commonName=example.com/organizationName=Example Corp
# | Not valid before: 2023-01-01T00:00:00
# |_Not valid after:  2024-01-01T00:00:00


# ── SSL CIPHERS ────────────────────────────────────────────────
nmap --script ssl-enum-ciphers -p 443 TARGET
# Lists ALL cipher suites supported and grades them A-F
# Output shows: Protocol versions + cipher names + key strength
# Use: Find weak ciphers, deprecated protocols
# Key things to look for:
#   A = Strong cipher (AES-256-GCM, CHACHA20)
#   F = Weak cipher (RC4, DES, NULL ciphers, EXPORT ciphers)
nmap --script ssl-enum-ciphers -p 443,8443 TARGET


# ── HEARTBLEED ─────────────────────────────────────────────────
nmap --script ssl-heartbleed -p 443 TARGET
# Tests for the Heartbleed vulnerability (CVE-2014-0160)
# Critical OpenSSL bug that allowed reading server memory
# Output: VULNERABLE or not
# Affects: OpenSSL 1.0.1 through 1.0.1f
nmap --script ssl-heartbleed -p 443,8443 TARGET


# ── POODLE ─────────────────────────────────────────────────────
nmap --script ssl-poodle -p 443 TARGET
# Tests for POODLE vulnerability (CVE-2014-3566)
# Affects SSLv3 — allows decryption of secure communications
# Output: VULNERABLE or not
nmap --script ssl-poodle -p 443 TARGET


# ── CCS INJECTION ──────────────────────────────────────────────
nmap --script ssl-ccs-injection -p 443 TARGET
# Tests for CCS Injection vulnerability (CVE-2014-0224)
# OpenSSL vulnerability allowing MITM attacks
nmap --script ssl-ccs-injection -p 443 TARGET


# ── DROWN ──────────────────────────────────────────────────────
nmap --script ssl-drown -p 443 TARGET
# Tests for DROWN vulnerability
# Affects servers that support SSLv2


# ── DH PARAMS ──────────────────────────────────────────────────
nmap --script ssl-dh-params -p 443 TARGET
# Checks Diffie-Hellman parameters
# Detects weak DH groups (Logjam vulnerability)
# Output: Key size and whether it's vulnerable
nmap --script ssl-dh-params -p 443 TARGET


# ── COMPLETE SSL AUDIT ─────────────────────────────────────────
# Run all SSL scripts at once
nmap --script "ssl-*" -p 443 TARGET

# Or specifically the most important ones
nmap --script "ssl-cert,ssl-enum-ciphers,ssl-heartbleed,ssl-poodle,ssl-dh-params" \
     -p 443 TARGET
```

---

## 6. Authentication Scripts

```bash
# ── SSH AUTH METHODS ──────────────────────────────────────────
nmap --script ssh-auth-methods -p 22 TARGET
# Lists authentication methods the SSH server accepts
# Output: publickey, password, keyboard-interactive
nmap --script ssh-auth-methods --script-args="ssh.user=root" -p 22 TARGET


# ── SSH HOSTKEY ────────────────────────────────────────────────
nmap --script ssh-hostkey -p 22 TARGET
# Retrieves SSH host key fingerprints
# Use: Track key changes (could indicate MITM attack)
nmap --script ssh-hostkey --script-args ssh_hostkey=full -p 22 TARGET


# ── SSH BRUTE FORCE ────────────────────────────────────────────
# WARNING: Only on authorised targets
nmap --script ssh-brute -p 22 TARGET
# Attempts common username/password combinations
nmap --script ssh-brute --script-args userdb=users.txt,passdb=passwords.txt -p 22 TARGET


# ── FTP ANON LOGIN ────────────────────────────────────────────
nmap --script ftp-anon -p 21 TARGET
# Checks if anonymous FTP login is allowed
# Critical finding — allows unauthenticated file access
# Output: VULNERABLE + list of files accessible


# ── FTP BOUNCE ─────────────────────────────────────────────────
nmap --script ftp-bounce -p 21 TARGET
# Tests if FTP server allows port scanning through it


# ── HTTP AUTH BRUTE ────────────────────────────────────────────
nmap --script http-brute -p 80,443 TARGET
# Brute forces HTTP Basic/Digest authentication
nmap --script http-brute --script-args userdb=users.txt,passdb=pass.txt -p 80 TARGET


# ── TELNET ─────────────────────────────────────────────────────
nmap --script telnet-ntlm-info -p 23 TARGET
# Gets NTLM authentication info from Telnet
nmap --script telnet-brute -p 23 TARGET
# Brute forces Telnet — use only on authorised targets
```

---

## 7. Vulnerability Scripts

```bash
# ── RUN ALL VULN SCRIPTS ──────────────────────────────────────
nmap --script vuln TARGET
# Runs ALL vulnerability detection scripts
# WARNING: This is intrusive — only on authorised targets
# Some scripts may crash vulnerable services


# ── ETERNALBLUE (MS17-010) ────────────────────────────────────
nmap --script smb-vuln-ms17-010 -p 445 TARGET
# Tests for EternalBlue vulnerability (used by WannaCry ransomware)
# Critical CVE — allows unauthenticated RCE on Windows
# Output: VULNERABLE or not
# Affects: Windows 7, Windows Server 2008 R2 and earlier (unpatched)


# ── MS08-067 ──────────────────────────────────────────────────
nmap --script smb-vuln-ms08-067 -p 445 TARGET
# Tests for MS08-067 (Conficker worm vulnerability)
# Remote code execution vulnerability


# ── LOG4SHELL ─────────────────────────────────────────────────
nmap --script http-log4shell -p 80,443,8080 TARGET
# Tests for Log4Shell vulnerability (CVE-2021-44228)
# Critical RCE in Apache Log4j


# ── SHELLSHOCK ─────────────────────────────────────────────────
nmap --script http-shellshock -p 80,443 TARGET
# Tests for Shellshock vulnerability (CVE-2014-6271)
# Bash code injection through CGI scripts
nmap --script http-shellshock --script-args uri=/cgi-bin/test.cgi -p 80 TARGET


# ── SLOWLORIS ──────────────────────────────────────────────────
nmap --script http-slowloris-check -p 80,443 TARGET
# Tests if server is vulnerable to Slowloris DoS attack
# (Check only — doesn't actually attack)


# ── FTP VSFTPD BACKDOOR ────────────────────────────────────────
nmap --script ftp-vsftpd-backdoor -p 21 TARGET
# Tests for backdoor in vsftpd 2.3.4 (famous CTF vulnerability)


# ── DISTCC EXECUTION ──────────────────────────────────────────
nmap --script distcc-cve2004-2687 -p 3632 TARGET
# Tests for remote code execution in distcc daemon


# ── SNMP COMMUNITY STRINGS ────────────────────────────────────
nmap --script snmp-brute -p 161 TARGET
# Brute forces SNMP community strings
nmap --script snmp-info -p 161 TARGET
# Gets SNMP device info using common community strings
```

---

## 8. SMB/Windows Scripts

```bash
# ── SMB SECURITY MODE ─────────────────────────────────────────
nmap --script smb-security-mode -p 445 TARGET
# Gets SMB security mode information
# Checks: Authentication level, message signing, access control
nmap --script smb-security-mode -p 445 TARGET
# Look for: message_signing: disabled (security risk)


# ── SMB ENUM SHARES ───────────────────────────────────────────
nmap --script smb-enum-shares -p 445 TARGET
# Enumerates SMB shares on the server
# Output: Share names, access rights, comment
nmap --script smb-enum-shares --script-args smbusername=admin,smbpassword=password -p 445 TARGET


# ── SMB ENUM USERS ────────────────────────────────────────────
nmap --script smb-enum-users -p 445 TARGET
# Enumerates Windows user accounts via SMB
nmap --script smb-enum-users -p 445 TARGET


# ── SMB OS DISCOVERY ──────────────────────────────────────────
nmap --script smb-os-discovery -p 445 TARGET
# Gets OS info via SMB (more reliable than -O for Windows)
# Output: Windows version, hostname, domain, workgroup


# ── SMB VULN COLLECTION ───────────────────────────────────────
# Run all SMB vulnerability scripts
nmap --script "smb-vuln-*" -p 445 TARGET

# The most important ones:
nmap --script "smb-vuln-ms17-010,smb-vuln-ms08-067,smb-security-mode,smb-enum-shares" \
     -p 445 TARGET


# ── NETBIOS ────────────────────────────────────────────────────
nmap --script nbstat -p 137 TARGET
# Gets NetBIOS name and MAC address
nmap -sU --script nbstat -p 137 TARGET


# ── RPC ENUM ──────────────────────────────────────────────────
nmap --script msrpc-enum -p 135 TARGET
# Enumerates Windows RPC endpoints
```

---

## 9. Database Scripts

```bash
# ── MYSQL ──────────────────────────────────────────────────────
nmap --script mysql-info -p 3306 TARGET
# Gets MySQL server info (version, protocol, etc.)

nmap --script mysql-empty-password -p 3306 TARGET
# Checks if root account has empty password (critical!)

nmap --script mysql-databases --script-args mysqluser=root -p 3306 TARGET
# Lists all databases (if accessible)

nmap --script mysql-brute -p 3306 TARGET
# Brute forces MySQL credentials

# All MySQL scripts:
nmap --script "mysql-*" -p 3306 TARGET


# ── MSSQL ──────────────────────────────────────────────────────
nmap --script ms-sql-info -p 1433 TARGET
# Gets MSSQL server info

nmap --script ms-sql-empty-password -p 1433 TARGET
# Checks for empty SA password

nmap --script ms-sql-config -p 1433 TARGET
# Gets MSSQL configuration

# All MSSQL scripts:
nmap --script "ms-sql-*" -p 1433 TARGET


# ── MONGODB ────────────────────────────────────────────────────
nmap --script mongodb-info -p 27017 TARGET
# Gets MongoDB server info and databases

nmap --script mongodb-databases -p 27017 TARGET
# Lists all MongoDB databases (if no auth)


# ── REDIS ──────────────────────────────────────────────────────
nmap --script redis-info -p 6379 TARGET
# Gets Redis server info

# If Redis has no auth (common misconfiguration):
nmap --script redis-info --script-args redis-info.db-index=0 -p 6379 TARGET


# ── POSTGRESQL ─────────────────────────────────────────────────
nmap --script pgsql-brute -p 5432 TARGET
# Brute forces PostgreSQL credentials


# ── ORACLE ─────────────────────────────────────────────────────
nmap --script oracle-brute -p 1521 TARGET
# Brute forces Oracle TNS credentials
```

---

## 10. DNS Scripts

```bash
# ── DNS BRUTE ──────────────────────────────────────────────────
nmap --script dns-brute TARGET
# Brute forces DNS subdomains
nmap --script dns-brute --script-args dns-brute.domain=example.com TARGET


# ── DNS ZONE TRANSFER ─────────────────────────────────────────
nmap --script dns-zone-transfer -p 53 TARGET
# Attempts DNS zone transfer (AXFR)
# If successful: reveals ALL DNS records (critical finding!)
nmap --script dns-zone-transfer --script-args "dns-zone-transfer.domain=example.com" \
     -p 53 TARGET


# ── DNS CACHE SNOOPING ────────────────────────────────────────
nmap --script dns-cache-snoop -p 53 TARGET
# Checks which domains the DNS server has recently resolved
# Reveals which websites org employees visit


# ── DNS RECURSION ──────────────────────────────────────────────
nmap --script dns-recursion -p 53 TARGET
# Checks if DNS server allows recursive queries (open resolver)
# Open resolvers can be used for DDoS amplification attacks


# ── DNS NSEC WALKING ──────────────────────────────────────────
nmap --script dns-nsec-enum -p 53 TARGET
# Enumerates DNS records using NSEC walking
nmap --script dns-nsec-enum --script-args "dns-nsec-enum.domains=example.com" -p 53 TARGET
```

---

## 11. Discovery Scripts

```bash
# ── BANNER GRABBING ────────────────────────────────────────────
nmap --script banner TARGET
# Grabs service banners from open ports
# Quick way to get service/version info without -sV


# ── WHOIS IP ───────────────────────────────────────────────────
nmap --script whois-ip TARGET
# Performs WHOIS lookup on the target IP


# ── WHOIS DOMAIN ──────────────────────────────────────────────
nmap --script whois-domain TARGET
# Performs WHOIS lookup on the domain name


# ── IP GEOLOCATION ─────────────────────────────────────────────
nmap --script ip-geolocation-geoplugin TARGET
nmap --script ip-geolocation-maxmind TARGET


# ── BROADCAST DISCOVERY ────────────────────────────────────────
# Find hosts on local network via broadcast
nmap --script broadcast-ping TARGET
nmap --script broadcast-dns-service-discovery TARGET
nmap --script broadcast-netbios-master-browser TARGET


# ── NETWORK TOPOLOGY ──────────────────────────────────────────
nmap --script traceroute-geolocation TARGET
# Maps network path with geolocation


# ── FINGER ─────────────────────────────────────────────────────
nmap --script finger -p 79 TARGET
# Queries finger service for user info


# ── IDENT ──────────────────────────────────────────────────────
nmap --script ident-owners TARGET
# Gets process/user info from ident service
```

---

## 12. Script Arguments

```bash
# ── PASSING ARGUMENTS TO SCRIPTS ──────────────────────────────
# Use --script-args to pass key=value pairs

# Single argument
nmap --script http-brute --script-args userdb=users.txt TARGET

# Multiple arguments
nmap --script smb-enum-shares \
     --script-args smbusername=admin,smbpassword=password123 \
     -p 445 TARGET

# From a file
nmap --script http-brute --script-args-file=brute_args.txt TARGET

# Common argument patterns:
# ─────────────────────────────────────────────────────────────

# SSH with credentials
nmap --script ssh-brute \
     --script-args userdb=usernames.txt,passdb=passwords.txt \
     -p 22 TARGET

# HTTP with specific URI
nmap --script http-shellshock \
     --script-args uri=/cgi-bin/vulnerable.cgi \
     -p 80 TARGET

# DNS with domain
nmap --script dns-brute \
     --script-args dns-brute.domain=example.com,dns-brute.threads=10 \
     TARGET

# SMB with credentials
nmap --script smb-enum-users \
     --script-args smbdomain=WORKGROUP,smbusername=user,smbpassword=pass \
     -p 445 TARGET

# Timeout adjustment
nmap --script http-brute \
     --script-args timeout=10,http-brute.firstonly=true \
     -p 80 TARGET
```

---

## 13. Quick Reference by Use Case

```bash
# ══════════════════════════════════════════════════════════════
# VAPT ENGAGEMENT — STANDARD SCRIPT SETS
# ══════════════════════════════════════════════════════════════

# 🌐 Web Application Assessment
nmap --script "http-title,http-headers,http-methods,http-robots.txt,
               http-waf-detect,http-backup-finder,http-sql-injection,
               http-stored-xss,http-default-accounts" \
     -p 80,443,8080,8443 TARGET

# 🔒 SSL/TLS Full Audit
nmap --script "ssl-cert,ssl-enum-ciphers,ssl-heartbleed,
               ssl-poodle,ssl-ccs-injection,ssl-dh-params" \
     -p 443,8443 TARGET

# 🪟 Windows/SMB Assessment
nmap --script "smb-security-mode,smb-vuln-ms17-010,smb-vuln-ms08-067,
               smb-enum-shares,smb-enum-users,smb-os-discovery" \
     -p 445 TARGET

# 🗄️ Database Security Check
nmap --script "mysql-info,mysql-empty-password,ms-sql-info,ms-sql-empty-password,
               mongodb-info,redis-info" \
     -p 1433,3306,5432,6379,27017 TARGET

# 🔑 Authentication Testing
nmap --script "ssh-auth-methods,ftp-anon,http-auth-finder,
               smtp-open-relay" \
     TARGET

# 📡 DNS Assessment
nmap --script "dns-zone-transfer,dns-recursion,dns-brute,dns-cache-snoop" \
     -p 53 TARGET

# 🛡️ Quick Vulnerability Scan
nmap --script "vuln" -sV TARGET

# 🔍 Full Recon (combine all safe scripts)
nmap --script "default,safe,discovery" -sV -p- TARGET

# ── CTF STANDARD SCAN ─────────────────────────────────────────
nmap -sV -sC -A -T4 TARGET

# ── MOST THOROUGH SCAN EVER ───────────────────────────────────
nmap -p- -sV -sC -O -A --script "default,safe,vuln" -T4 TARGET -oA thorough_scan
```

---

*Author: Rishabh Sankhla | CEH v13 | TryHackMe Top 2% | Last Updated: May 2026***
