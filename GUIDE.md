# Nmap Operational Guide

This guide explains *why* and *when* to use specific Nmap commands. For a quick command reference, see [README.md](README.md). For a beginner tutorial, see [TUTORIAL.md](TUTORIAL.md).

---

## 1. Scanning Philosophy

### The Golden Rule of Network Scanning

**Step 1: Ping sweep first.**  
Find live hosts before scanning ports.  
`nmap -sn 192.168.1.0/24`

**Step 2: Scan only live hosts.**  
Feed the IPs from Step 1 into a targeted port scan.  
`nmap -sV -T4 192.168.1.10`

**Step 3: Deep dive on interesting services.**  
Once you find open ports, use scripts or version detection.  
`nmap -sC --script http-enum -p 80 192.168.1.10`

This three-step workflow is faster, quieter, and more professional than scanning everything blindly.

---

### When to Use a Ping Sweep vs. a Port Scan

| **Goal** | **Command** | **When to Use** |
|----------|-------------|-----------------|
| Find live hosts | `nmap -sn` | **First step** on any new network. Fast and low-noise. |
| Find open ports | `nmap -sS` or `-sT` | **Second step** after you know which hosts are alive. |
| Find service versions | `nmap -sV` | **Third step** after you find open ports. More intrusive. |

---

## 2. Scan Types (What Each One Does)

### TCP Scans

| Scan Type | Command | How It Works | Best For | Needs Root? |
|-----------|---------|--------------|----------|--------------|
| **SYN scan** | `-sS` | Sends SYN, gets SYN/ACK, sends RST | Stealth, speed | **Yes** |
| **Connect scan** | `-sT` | Completes full TCP handshake | Reliability | No |
| **ACK scan** | `-sA` | Sends ACK packet, looks for RST | Mapping firewall rules | **Yes** |
| **Window scan** | `-sW` | Same as ACK, but different logic | Detecting open/filtered ports | **Yes** |
| **FIN/NULL/XMAS** | `-sF` / `-sN` / `-sX` | Sends packets with unusual flags | Evading old firewalls | **Yes** |

### UDP Scans

| Scan Type | Command | How It Works | Best For |
|-----------|---------|--------------|----------|
| **UDP scan** | `-sU` | Sends UDP packets, waits for response | DNS (53), SNMP (161), DHCP (67/68) |

> **UDP scanning is slow and unreliable.** Most UDP services don't respond to empty probes. Use `-sU` only when you're specifically looking for UDP services.

---

## 3. Timing Templates (Speed vs. Stealth)

| Template | Command | Delay | Best For | Risk |
|----------|---------|-------|----------|------|
| **Paranoid** | `-T0` | 5 minutes per port | Avoiding IDS at all costs | Very low |
| **Sneaky** | `-T1` | 15 seconds per port | Production networks | Low |
| **Polite** | `-T2` | 0.4 seconds | Normal environments | Low-Medium |
| **Normal** | `-T3` | Default | Standard scans | Medium |
| **Aggressive** | `-T4` | Fast (assumes good network) | Your own lab | Medium-High |
| **Insane** | `-T5` | Very fast | When you don't care about detection | Very high |

### Real-world advice:
- **Use `-T1` or `-T2`** when scanning a client's production network.
- **Use `-T4`** when scanning your own lab or a trusted network.
- **Never use `-T5`** unless you're trying to trigger alerts (or don't care if you do).

---

## 4. Evasion Techniques (When to Use Them)

| Technique | Command | When to Use | How It Works |
|-----------|---------|-------------|--------------|
| **Decoy scan** | `-D RND:5` | Your IP is fixed and traceable | Hides your real IP among fake ones |
| **Fragment packets** | `-f` | Target has a simple firewall | Breaks packets into tiny pieces |
| **MAC spoofing** | `--spoof-mac 0` | Local network scanning | Changes your hardware address |
| **Source port manipulation** | `--source-port 53` | Firewall allows DNS traffic | Makes scan look like DNS queries |
| **Idle scan** | `-sI <zombie>` | Extreme stealth (advanced) | Uses a third-party "zombie" host |

> **Important:** Modern EDR (Endpoint Detection and Response) is not fooled by basic evasion. Use these techniques as one layer, not your only defense.

---

## 5. Detection (What Defenders See)

| Your Command | What the Defender's Logs Show | Risk Level |
|--------------|-------------------------------|------------|
| `nmap -sn` | ICMP echo requests to many IPs | Low |
| `nmap -sS` | SYN packets to many ports from one IP | Medium |
| `nmap -sS -D RND:10` | SYN packets from many IP addresses | Low-Medium |
| `nmap -sS -T5` | Flood of SYN packets. Triggers rate alerts. | High |
| `nmap -sV` | Full TCP connections + data probes | High |

**If you're defending:** Look for bursts of SYN packets, ICMP echo requests across a whole subnet, or connections to many ports on a single host in a short time.

---

## 6. Output and Exfiltration

| Command | What It Does | When to Use |
|---------|--------------|-------------|
| `-oA scan_name` | Saves all formats (normal, XML, grepable) | You want to keep results for later |
| `-oG scan.grep` | Grepable output | You plan to parse results with scripts |
| `grep "Nmap scan report" scan.txt \| awk '{print $5}'` | Extract IPs from scan | You need a list of live hosts |
| `grep "open" scan.txt \| awk '{print $1, $3}'` | Extract open ports | You need a list of services |

**Pro tip:** Always save your scans with `-oA`. You'll thank yourself later when you need to reference or share results.

---

## 7. Automation Workflows

- Scanning multiple subnets from a list
```bash
while read subnet; do
    nmap -sn "$subnet" -oG "scan_$(echo "$subnet" | tr '/' '_').grep"
done < subnets.txt
```
* Use this when you have a file (subnets.txt) containing one subnet per line (e.g., 192.168.1.0/24).

- Deep scanning all hosts from a previous scan

```bash
while read ip; do
    nmap -sV -T4 "$ip" -oN "scan_${ip}.txt"
done < ip_list.txt
```

* Use this after a ping sweep (-sn) to scan only the hosts that replied.

- Parallel scanning (faster, noisier)

```bash
cat ip_list.txt | xargs -I {} -P 5 nmap -sV -T4 {} -oN "scan_{}.txt"
```
* The -P 5 runs up to 5 scans at the same time. Increase the number for more speed, but be aware it will generate more noise.

---


## 8. Target-Specific Strategies

|Target Type| Recommended Command| Why|
|-----|------|------|
|Web server| `nmap -sV --script http-enum -p 80,443,8080 <ip>` |Version detection + directory enumeration|
|Windows SMB |`nmap -p 445 --script smb-os-discovery,smb-security-mode <ip>`| OS version + security settings|
|SSH| `nmap -p 22 --script ssh-auth-methods,ssh-hostkey <ip>` |Authentication methods + host keys|
|MySQL| `nmap -p 3306 --script mysql-info <ip>` |Version and protocol info|
|SNMP| `nmap -sU -p 161 --script snmp-community <ip>`| UDP scan + community string detection|

---

## 9. Common Mistakes (and How to Fix Them)

|Mistake| Why It Happens| The Fix|
|----|-----|-----|
|Running -sS without sudo |Nmap silently falls back to -sT (TCP Connect)| Always use sudo nmap -sS for SYN scans|
|Scanning all ports (-p-) on a large subnet |Slow, noisy, generates massive logs| Scan top ports first (--top-ports 1000), then deep scan only interesting hosts|
|Target is reported "down" but you know it's up |The host blocks ICMP (ping)| Use -Pn to skip host discovery entirely|
|Your scan is taking forever |Timing template too slow, or scanning UDP| Use -T4 for speed; narrow UDP port range|
|Forgetting to save output |Didn't use -oA |Always use `-oA scan_name`|
|Scanning from a fixed IP without decoys| Your real IP is in the logs |Use `-D RND:5` to add decoys|

---

10. Real-World Workflows

Workflow 1: Cautious Discovery (Unknown Network)

```bash
# Step 1: Slow ping sweep with decoys
nmap -sn -T2 -D RND:5 192.168.1.0/24 -oA ping_sweep

# Step 2: Extract live IPs
grep "Up" ping_sweep.grep | cut -d' ' -f2 > live_hosts.txt

# Step 3: Slow SYN scan on live hosts
while read ip; do
    sudo nmap -sS -T2 -D RND:5 "$ip" -oN "scan_${ip}.txt"
done < live_hosts.txt
```

Workflow 2: Focused Web Server Enumeration

```bash
# Scan only web ports with version detection and scripts
sudo nmap -sS -sV -p 80,443,8080 --script http-enum,http-headers,http-title 192.168.1.10 -oA web_scan
```

Workflow 3: Full Lab Scan (Own Network)

```bash
# Fast, aggressive scan on your own lab
sudo nmap -A -T4 -p- 192.168.1.10 -oA full_lab_scan
```

---

11. Defense (Detecting and Blocking Scans)

Detecting SYN Scans

```bash
tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0 and tcp[tcpflags] & (tcp-ack) == 0'
```

Detecting FIN/NULL/XMAS Scans

```bash
tcpdump -i eth0 'tcp[tcpflags] & (tcp-fin) != 0'
tcpdump -i eth0 'tcp[tcpflags] == 0'
tcpdump -i eth0 'tcp[tcpflags] & (tcp-fin|tcp-urg|tcp-psh) == (tcp-fin|tcp-urg|tcp-psh)'
```

Rate-Limiting ICMP (Ping) Sweeps

```bash
iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/second --limit-burst 5 -j ACCEPT
iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
```

Rate-Limiting TCP Scans

```bash
iptables -A INPUT -p tcp --syn -m recent --name portscan --set -j DROP
iptables -A INPUT -p tcp --syn -m recent --name portscan --rcheck --seconds 60 --hitcount 10 -j DROP
```

---

12. Advanced Topics Summary

Topic Where to Learn More
NSE script development See README.md#13-nse-script-development
IPv6 scanning See README.md#11-ip-v6-scanning
Firewall evasion See README.md#12-firewall-evasion
Evading modern EDR See README.md#17-evading-modern-edr

---

✅ Guide Completion Note

This guide covers the operational decisions behind Nmap scanning. For a complete command reference, see README.md. For a one-page cheat sheet, see QUICKREF.md. For a beginner tutorial, see TUTORIAL.md.

