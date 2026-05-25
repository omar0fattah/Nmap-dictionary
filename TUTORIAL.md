# Nmap Tutorial: From Zero to Network Scanner

This tutorial assumes you know nothing about Nmap. By the end, you'll be able to scan networks, interpret results, and understand the risks.

> **⚠️ Legal Disclaimer**  
> This tutorial is for educational purposes. Only scan networks you own or have written permission to test. Unauthorized scanning is illegal.

---

## 📖 Table of Contents

1. [What is Nmap?](#1-what-is-nmap)
2. [Installing Nmap](#2-installing-nmap)
3. [Your First Scan](#3-your-first-scan)
4. [Understanding Port States](#4-understanding-port-states)
5. [Scanning Your Own Network](#5-scanning-your-own-network)
6. [Service Version Detection](#6-service-version-detection)
7. [Ping Sweeps (Finding Live Hosts)](#7-ping-sweeps-finding-live-hosts)
8. [Scanning Specific Ports](#8-scanning-specific-ports)
9. [Output and Saving Results](#9-output-and-saving-results)
10. [Staying Quiet (or Not)](#10-staying-quiet-or-not)
11. [Real-World Workflow](#11-real-world-workflow)
12. [Next Steps](#12-next-steps)

---

## 1. What is Nmap?

- **Nmap (Network Mapper) is a tool that sends packets to computers and listens for replies. It answers:**

- Which computers are alive on a network?
- What doors (ports) are open on those computers?
- What software is running behind those doors?
- What operating system is the computer running?

**Think of it like this:** Your computer has 65,535 doors (ports). Some doors are open for business (web server, file sharing). Some doors are closed. Some doors have guards (firewalls). Nmap knocks on every door and reports back.

- For a complete list of commands, see the [README.md](README.md). For a one-page cheat sheet, see [QUICKREF.md](QUICKREF.md). For strategy and depth, see [GUIDE.md](GUIDE.md).

---

## 2. Installing Nmap

### On Linux (Ubuntu/Debian/Kali)
```bash
sudo apt update
sudo apt install nmap -y
```

- **On macOS**

```bash
brew install nmap
```

- **On Windows**

- Download the installer from https://nmap.org/ and run it.

- **Verify installation**

```bash
nmap --version
```

- You should see output like: Nmap version 7.xx

---

## 3. Your First Scan

- **Open a terminal. Type:**

```bash
nmap scanme.nmap.org
```

- This scans a test server that Nmap.org provides for practice. You'll see output like:

```
Starting Nmap 7.xx ( https://nmap.org )
Nmap scan report for scanme.nmap.org (45.33.32.156)
Host is up (0.090s latency).
Not shown: 996 closed ports
PORT     STATE    SERVICE
22/tcp   open     ssh
80/tcp   open     http
9929/tcp open     nping-echo
31337/tcp open     Elite

Nmap done: 1 IP address (1 host up) scanned in 3.42 seconds
```

- What you just learned: Nmap scanned the most common 1000 ports on that server and found 4 open ports (22, 80, 9929, 31337).

---

## 4. Understanding Port States

- **After a scan, each port has a state:**

|State|Meaning|
|----|----|
|open |A service is listening. The door is open.|
|closed| No service is listening, but the computer is reachable.|
|filtered| Something (firewall) blocked the probe. Nmap can't tell if it's open or closed.|
|unfiltered |The port is reachable but Nmap can't tell if it's open or closed (rare).|

- Why this matters: open ports are your entry points. filtered ports mean a firewall is protecting them.

---

5. Scanning Your Own Network

First, find your IP address:

```bash
ip a   # Linux
ifconfig   # macOS
ipconfig   # Windows
```

Look for an address like 192.168.1.5 or 10.0.0.2. The first three numbers (192.168.1) are your network. The last number (.5) is your device.

Now scan your local network:

```bash
nmap 192.168.1.0/24
```

The /24 means "scan all addresses from 192.168.1.1 to 192.168.1.254". You'll see every device on your Wi-Fi that responds.

What you'll see: A list of IP addresses, open ports, and services. You might see your router, your phone, other computers.

---

6. Service Version Detection

The basic scan tells you a port is open (e.g., port 80). But what's actually running there? Apache? Nginx? A custom web server?

Add -sV to ask for version information:

```bash
nmap -sV 192.168.1.1
```

Now the output includes service names and versions:

```
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.9p1
80/tcp   open  http       Apache httpd 2.4.52
```

Why this matters: Different versions have different vulnerabilities. Knowing the version tells you what exploits to try.

---

7. Ping Sweeps (Finding Live Hosts)

Scanning all 254 addresses on your network for open ports takes time. First, find which hosts are alive with a ping sweep:

```bash
nmap -sn 192.168.1.0/24
```

This pings every address and tells you which respond. It's fast and quiet.

Why this matters: Why scan 254 addresses for open ports when only 10 are alive? Ping sweep first, then scan only the live ones.

---

8. Scanning Specific Ports

By default, Nmap scans the top 1000 most common ports. To scan specific ports, use -p:

```bash
nmap -p 22,80,443 192.168.1.1
```

Scans only ports 22, 80, and 443.

```bash
nmap -p 1-1000 192.168.1.1
```

Scans ports 1 through 1000.

```bash
nmap -p- 192.168.1.1
```

Scans all 65,535 ports. This takes much longer. Use it when you know a service is on a non-standard port.

Why this matters: Attackers (and defenders) often check the top 1000 ports first. Important services sometimes hide on unusual ports.

---

9. Output and Saving Results

Don't lose your scan results. Use -oA to save everything:

```bash
nmap -sV -oA my_scan 192.168.1.1
```

This creates three files:

· my_scan.nmap (human-readable)
· my_scan.xml (for tools)
· my_scan.gnmap (grepable)

To see what's in the file:

```bash
cat my_scan.nmap
```

Why this matters: Later, you'll want to compare scans or share results. Saving output is a professional habit.

---

10. Staying Quiet (or Not)

Nmap has timing templates that control speed and stealth:

Template Command Speed Stealth Best For
Paranoid -T0 Very slow Very high Avoiding detection at all costs
Sneaky -T1 Slow High Production networks
Polite -T2 Moderate Medium Normal environments
Normal -T3 Default Medium Everyday scanning
Aggressive -T4 Fast Low Your own lab
Insane -T5 Very fast Very low When you don't care about detection

Example: Sneaky scan

```bash
nmap -T1 -sV 192.168.1.1
```

Example: Fast lab scan

```bash
nmap -T4 -sV 192.168.1.1
```

Rule of thumb:

· On a client's network or any system you don't control → use -T1 or -T2
· On your own lab or trusted network → use -T4
· Never use -T5 unless you want to trigger every alarm

---

11. Real-World Workflow

Here's how a professional scans a new network:

```bash
# Step 1: Find live hosts (fast, quiet)
nmap -sn -T2 192.168.1.0/24 -oA ping_sweep

# Step 2: Extract live IPs from the scan
grep "Up" ping_sweep.gnmap | cut -d' ' -f2 > live_hosts.txt

# Step 3: Scan only live hosts for open ports and versions
while read ip; do
    sudo nmap -sS -sV -T4 "$ip" -oA "scan_$ip"
done < live_hosts.txt
```

This workflow is faster, quieter, and more professional than scanning everything blindly.

---

12. Next Steps

You now know enough to scan networks and understand the results. To go further:

· Explore the README.md for every command
· Use the QUICKREF.md as a daily cheat sheet
· Read the GUIDE.md for operational depth and strategy
· Practice on your own lab (use VirtualBox + Metasploitable or VulnHub)
· Never scan systems you don't own or have permission to test

Remember: Nmap is a tool. Like any tool, it can be used for good or for harm. Use it to learn, to defend, and to understand. Not to break.

🔝 Back to Top

---

✅ Tutorial Completion Note

You've completed the tutorial. You can now scan networks, interpret results, and save your work. You're not an expert yet, but you have the foundation.
