# Nmap Quick Reference Card

A one-page cheat sheet for the most common Nmap tasks. For detailed explanations, see the [full manual](README.md).

---

## The 10 Most Useful Commands

| # | Task | Command |
|---|------|---------|
| 1 | **Ping sweep** (find live hosts) | `nmap -sn 192.168.1.0/24` |
| 2 | **SYN stealth scan** (requires root) | `sudo nmap -sS -sV -T4 192.168.1.1` |
| 3 | **Full port scan** (all 65535 ports) | `nmap -p- -sV -T4 192.168.1.1` |
| 4 | **UDP scan** (DNS, SNMP, etc.) | `nmap -sU -p 53,161 192.168.1.1` |
| 5 | **Aggressive scan** (OS, version, scripts) | `nmap -A -T4 192.168.1.1` |
| 6 | **Vulnerability scan** | `nmap --script vuln -T4 192.168.1.1` |
| 7 | **Save all output formats** | `nmap -oA scan_name 192.168.1.0/24` |
| 8 | **Decoy scan** (hide your IP) | `nmap -D RND:10 -sS 192.168.1.1` |
| 9 | **IPv6 scan** | `nmap -6 -sV -T4 2001:db8::1` |
| 10 | **Detect SYN scans** (defense) | `tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0 and tcp[tcpflags] & (tcp-ack) == 0'` |

---

## Speed vs. Stealth (Timing Templates)

| Template | Flag | Best For |
|----------|------|----------|
| Paranoid | `-T0` | Avoiding IDS at all costs (very slow) |
| Sneaky | `-T1` | Production networks (stealthy) |
| Polite | `-T2` | Normal scanning (good balance) |
| Normal | `-T3` | Default |
| Aggressive | `-T4` | Your own lab (fast) |
| Insane | `-T5` | When you don't care about detection |

---

## Common Service Ports

| Service | Port | UDP? |
|---------|------|------|
| HTTP | 80 | No |
| HTTPS | 443 | No |
| SSH | 22 | No |
| DNS | 53 | **Yes** |
| SNMP | 161 | **Yes** |
| SMB | 445 | No |
| MySQL | 3306 | No |
| RDP | 3389 | No |

---

## One-Liner Workflows

| Goal | Command |
|------|---------|
| Find live hosts, then deep scan them | `nmap -sn 192.168.1.0/24 -oG - \| awk '/Up/{print $2}' > ips.txt && nmap -sV -iL ips.txt` |
| Scan only the most common ports | `nmap --top-ports 100 192.168.1.1` |
| Scan a specific port range | `nmap -p 1-1000 192.168.1.1` |
| Extract open ports from previous scan | `grep "open" scan.txt \| cut -d'/' -f1 \| tr '\n' ','` |

---

📖 [Back to Full Manual](README.md)
