# Linux Networking, Diagnostics & Firewall (UFW) Guide

A comprehensive, production-grade guide to Linux network inspection, modern socket diagnostics (`ss`), connectivity troubleshooting, DNS resolution, firewall security with **UFW**, and brute-force mitigation with **Fail2ban**.

---

## 📑 Table of Contents

1. [Modern Network Inspection (`ip` Tool Suite)](#-1-modern-network-inspection-ip-tool-suite)
   - [Legacy vs. Modern Tool Replacements](#legacy-vs-modern-tool-replacements)
   - [Managing Interfaces & IP Addresses](#managing-interfaces--ip-addresses)
   - [Routing Table & Gateway](#routing-table--default-gateway)
2. [Socket & Port Inspection (`ss`)](#-2-socket--port-inspection-ss)
   - [Deep Dive into `ss -tulpn`](#deep-dive-into-ss--tulpn)
   - [Filtering Sockets by Port & State](#filtering-sockets-by-port--connection-state)
3. [Connectivity & Port Diagnostics](#-3-connectivity--port-diagnostics)
   - [Path Diagnostics: `ping` vs. `mtr`](#path-diagnostics-ping-vs-mtr)
   - [Testing TCP/UDP Ports with Netcat (`nc`)](#testing-ports-with-netcat-nc)
   - [HTTP & Latency Benchmarks with `curl`](#http--latency-benchmarking-with-curl)
4. [DNS Resolution & Troubleshooting](#-4-dns-resolution--troubleshooting)
   - [The Linux DNS Stack](#the-linux-dns-lookup-flow)
   - [Querying Nameservers with `dig`](#querying-nameservers-with-dig)
   - [Systemd-Resolved & Cache Flushing](#systemd-resolved--cache-flushing)
5. [Firewall Management with UFW](#️-5-firewall-management-with-ufw)
   - [Safe Initial Configuration](#safe-initial-configuration-dont-lock-yourself-out)
   - [Allowing, Denying, and Rate-Limiting Rules](#allowing-denying-and-rate-limiting-rules)
   - [IP Whitelisting & Subnet Rules](#ip-whitelisting--subnet-rules)
   - [Deleting & Managing Rules by Number](#deleting--managing-rules-by-number)
6. [Brute-Force Protection with Fail2ban](#-6-brute-force-protection-with-fail2ban)
   - [Setup & Jail Configuration](#fail2ban-setup--configuration)
   - [Fail2ban CLI Management](#fail2ban-cli-management)
7. [Static IP Configuration (Netplan Overview)](#-7-static-ip-configuration-netplan-overview)
8. [Quick Reference Cheat Sheet](#-8-quick-reference-cheat-sheet)

---

## 🌐 1. Modern Network Inspection (`ip` Tool Suite)

### Legacy vs. Modern Tool Replacements

The legacy `net-tools` package (`ifconfig`, `netstat`, `route`, `arp`) was deprecated in 2009. Modern Linux systems use the `iproute2` suite (`ip`, `ss`).

| Legacy Command (Deprecated) | Modern Replacement | Purpose |
| :--- | :--- | :--- |
| `ifconfig` | `ip address` / `ip a` | View and manage interface IP addresses |
| `ifconfig eth0 up/down` | `ip link set eth0 up/down` | Enable or disable network interfaces |
| `route -n` | `ip route` / `ip r` | View and edit kernel routing tables |
| `netstat -tulpn` | `ss -tulpn` | Inspect open sockets and listening ports |
| `arp -a` | `ip neighbor` / `ip neigh` | Inspect ARP cache (MAC address mappings) |

---

### Managing Interfaces & IP Addresses

```bash
# 1. View all network interfaces and assigned IPs:
ip address show
# (Shortcut: ip a)

# 2. Compact, single-line overview:
ip -br a

# 3. View only IPv4 addresses:
ip -4 a

# 4. View statistics (packets, errors, dropped packets) for an interface:
ip -s link show eth0

# 5. Bring an interface up or down:
sudo ip link set dev eth0 up
sudo ip link set dev eth0 down

# 6. Temporarily assign a secondary IP address:
sudo ip addr add 192.168.1.100/24 dev eth0

# 7. Remove an IP address:
sudo ip addr del 192.168.1.100/24 dev eth0
```

---

### Routing Table & Default Gateway

```bash
# View active routing table:
ip route show
# (Shortcut: ip r)

# Identify which network interface and gateway handle traffic to an IP:
ip route get 8.8.8.8
# Example Output: 8.8.8.8 via 192.168.1.1 dev eth0 src 192.168.1.50 uid 1000

# Add a temporary static route to a specific subnet:
sudo ip route add 10.200.0.0/16 via 192.168.1.254 dev eth0

# Delete a static route:
sudo ip route del 10.200.0.0/16
```

---

## 🔌 2. Socket & Port Inspection (`ss`)

The **`ss` (Socket Statistics)** utility reads directly from Linux kernel netlink sockets, providing instant results even on servers with tens of thousands of active connections.

### Deep Dive into `ss -tulpn`

This is the single most important command for discovering which services are listening on which ports:

```bash
sudo ss -tulpn
```

#### Flag Breakdown

* **`-t`**: Display **TCP** sockets.
* **`-u`**: Display **UDP** sockets.
* **`-l`**: Show only **listening** sockets (services waiting for connections).
* **`-p`**: Show the **process** name and PID holding the socket (requires `sudo`).
* **`-n`**: Do not resolve service names (displays `:80` instead of `:http`, `:22` instead of `:ssh`). **Crucial for speed!**

#### Example Output

```text
Netid  State   Recv-Q  Send-Q   Local Address:Port   Peer Address:Port  Process
tcp    LISTEN  0       128            0.0.0.0:22          0.0.0.0:*      users:(("sshd",pid=1204,fd=3))
tcp    LISTEN  0       511          127.0.0.1:8000        0.0.0.0:*      users:(("uvicorn",pid=8421,fd=4))
tcp    LISTEN  0       4096                 *:80                *:*      users:(("nginx",pid=2311,fd=6))
```

* `0.0.0.0:22`: Accessible from **any** network interface (public/LAN).
* `127.0.0.1:8000`: Bound to **localhost only** (cannot be accessed from external networks).

---

### Filtering Sockets by Port & Connection State

```bash
# Find specifically what process is using port 80 or 443:
sudo ss -tulpn '( sport = :80 or sport = :443 )'

# View all actively ESTABLISHED outbound and inbound TCP connections:
ss -t state established

# Summary of overall socket counts (TCP, UDP, RAW):
ss -s
```

---

## 📡 3. Connectivity & Port Diagnostics

### Path Diagnostics: `ping` vs. `mtr`

#### `ping`: Basic ICMP reachability

```bash
# Ping with count limit and 1-second timeout:
ping -c 4 -W 1 1.1.1.1
```

#### `mtr`: Real-Time Combined Ping & Traceroute

Standard `traceroute` only shows a single probe per hop. **`mtr` (My Traceroute)** runs continuous probes, showing packet loss percentage and latency jitter across every intermediate router:

```bash
sudo apt install -y mtr
mtr google.com
```

* Press `d`: Switch display modes.
* Press `q`: Quit.
* Generate a non-interactive report for pasting into tickets:

  ```bash
  mtr --report --report-cycles 10 api.github.com
  ```

---

### Testing Ports with Netcat (`nc`)

Check if a remote TCP or UDP port is open **without needing full client software**:

```bash
# Check if port 443 is open on example.com:
nc -zv -w 3 example.com 443
```

* `-z`: Zero-I/O mode (scan without sending data).
* `-v`: Verbose output (`Connection to example.com 443 port [tcp/https] succeeded!`).
* `-w 3`: 3-second connection timeout.

#### Test a range of ports

```bash
nc -zv 192.168.1.100 20-25
```

---

### HTTP & Latency Benchmarking with `curl`

Test endpoints, inspect headers, and measure latency bottlenecks:

```bash
# 1. Fetch only response headers (-I) and follow redirects (-L):
curl -IL https://api.example.com/health

# 2. Measure detailed HTTP connection timings:
curl -o /dev/null -s -w \
  "DNS Lookup:     %{time_namelookup}s\n\
Connect:        %{time_connect}s\n\
TLS Handshake:  %{time_appconnect}s\n\
TTFB (First byte): %{time_starttransfer}s\n\
Total Time:     %{time_total}s\n" \
  https://google.com
```

---

## 🔎 4. DNS Resolution & Troubleshooting

### The Linux DNS Lookup Flow

```text
1. Check local /etc/hosts
       │
       ▼
2. Query local DNS resolver (/etc/resolv.conf)
       │
       ▼
3. Forward to systemd-resolved or upstream recursive nameservers (e.g. 1.1.1.1)
```

---

### Querying Nameservers with `dig`

`dig` (Domain Information Groper) is the gold standard for querying DNS servers.

```bash
# 1. Standard DNS lookup:
dig example.com

# 2. Short answer only (clean IP output):
dig +short example.com

# 3. Query a specific nameserver directly (bypassing local DNS cache):
dig @1.1.1.1 example.com

# 4. Query specific record types:
dig example.com MX       # Mail Exchange records
dig example.com TXT      # SPF, DKIM verification records
dig example.com CNAME    # Alias records

# 5. Trace the full DNS hierarchy from root servers down:
dig +trace example.com

# 6. Reverse DNS lookup (IP to domain):
dig -x 8.8.8.8 +short
```

---

### Systemd-Resolved & Cache Flushing

Modern Ubuntu and Debian distros use `systemd-resolved` for local caching.

```bash
# View active DNS servers per interface:
resolvectl status

# Flush the local DNS cache:
sudo resolvectl flush-caches

# Verify cache size and hit statistics:
resolvectl statistics
```

---

## 🛡️ 5. Firewall Management with UFW

**UFW (Uncomplicated Firewall)** is the standard, user-friendly interface for managing Linux iptables and nftables packet filters.

---

### Safe Initial Configuration (Don't Lock Yourself Out!)

> [!CAUTION]
> Always allow SSH **before** enabling UFW on a remote server, otherwise you will sever your connection immediately!

```bash
# 1. Set secure default policies (deny all incoming, allow all outgoing):
sudo ufw default deny incoming
sudo ufw default allow outgoing

# 2. Explicitly allow your SSH port:
sudo ufw allow 22/tcp
# (If using custom SSH port 2222: sudo ufw allow 2222/tcp)

# 3. (Optional) Rate-limit SSH to ban IP brute-forcers (6 attempts / 30 seconds):
sudo ufw limit 22/tcp

# 4. Safely enable the firewall:
sudo ufw enable

# 5. Verify status:
sudo ufw status verbose
```

---

### Allowing, Denying, and Rate-Limiting Rules

```bash
# Allow web traffic (HTTP & HTTPS):
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Or allow by application profile name:
sudo ufw allow "Nginx Full"
sudo ufw allow "OpenSSH"

# Allow specific port ranges:
sudo ufw allow 6000:6010/tcp

# Deny a specific malicious IP address completely:
sudo ufw deny from 203.0.113.50
```

---

### IP Whitelisting & Subnet Rules

Restrict database or internal administrative access to trusted sources:

```bash
# Allow only a specific office IP to connect to PostgreSQL (port 5432):
sudo ufw allow from 198.51.100.25 to any port 5432 proto tcp

# Allow an entire private internal subnet:
sudo ufw allow from 10.0.0.0/24 to any port 6379 proto tcp

# Allow all traffic across a specific local interface:
sudo ufw allow in on eth1
```

---

### Deleting & Managing Rules by Number

Never guess rule syntax when deleting firewall rules—always use numbered indexes:

```bash
# 1. List rules with index numbers:
sudo ufw status numbered
```

Example Output:

```text
     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere
[ 2] 80/tcp                     ALLOW IN    Anywhere
[ 3] 5432/tcp                   ALLOW IN    198.51.100.25
```

```bash
# 2. Delete rule number 3:
sudo ufw delete 3
```

---

## 🚫 6. Brute-Force Protection with Fail2ban

Fail2ban monitors service log files (`/var/log/auth.log`, Nginx logs) and automatically updates firewall rules to block IPs exhibiting malicious behavior.

### Fail2ban Setup & Configuration

```bash
sudo apt install -y fail2ban
```

Never edit `/etc/fail2ban/jail.conf` directly. Create an override file:

```bash
sudo nano /etc/fail2ban/jail.local
```

```ini
[DEFAULT]
# Ban duration (1 hour)
bantime = 1h

# Window of time to inspect (10 minutes)
findtime = 10m

# Max retry attempts before ban
maxretry = 5

# Whitelist trusted IPs (never ban local subnet or workstation)
ignoreip = 127.0.0.1/8 ::1 192.168.1.50

# Default action: block using UFW
banaction = ufw

[sshd]
enabled = true
port = 22
logpath = %(sshd_log)s
backend = %(sshd_backend)s
maxretry = 3
```

Enable and start:

```bash
sudo systemctl enable --now fail2ban
```

---

### Fail2ban CLI Management

```bash
# 1. View overall Fail2ban status and active jails:
sudo fail2ban-client status

# 2. Check jailed SSH statistics and list currently banned IPs:
sudo fail2ban-client status sshd

# 3. Manually unban an IP (e.g. if you locked yourself out):
sudo fail2ban-client set sshd unbanip 203.0.113.10

# 4. Manually ban an abusive IP:
sudo fail2ban-client set sshd banip 198.51.100.99
```

---

## 📝 7. Static IP Configuration (Netplan Overview)

Modern Ubuntu systems manage network configurations declaratively using **Netplan** YAML files in `/etc/netplan/`.

Example: Configuring a static IP for `eth0` in `/etc/netplan/01-netcfg.yaml`:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.1.50/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 1.1.1.1
          - 8.8.8.8
```

Apply changes:

```bash
sudo netplan try    # Tests config with 120s auto-revert if connectivity drops
sudo netplan apply  # Permanently applies config
```

---

## 🎯 8. Quick Reference Cheat Sheet

| Task | Command |
| :--- | :--- |
| **List IPs on all interfaces** | `ip -br a` |
| **Inspect Listening Ports & PIDs** | `sudo ss -tulpn` |
| **Find what is on Port 80** | `sudo ss -tulpn '( sport = :80 )'` |
| **Interactive Latency & Packet Loss** | `mtr <host>` |
| **Test Port Reachability** | `nc -zv -w 3 <host> <port>` |
| **Short DNS Lookup** | `dig +short <domain>` |
| **Flush Linux DNS Cache** | `sudo resolvectl flush-caches` |
| **Check UFW Rules by Number** | `sudo ufw status numbered` |
| **Allow Inbound Web Traffic** | `sudo ufw allow 80,443/tcp` |
| **Delete UFW Rule #2** | `sudo ufw delete 2` |
| **List Banned IPs in Fail2ban** | `sudo fail2ban-client status sshd` |
| **Unban an IP in Fail2ban** | `sudo fail2ban-client set sshd unbanip <IP>` |
