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
