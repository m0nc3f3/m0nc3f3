


## title: "Nmap — Live Host Discovery & Port Scanning Cheat Sheet"  
aliases: ["nmap-live-host-discovery", "nmap-cheatsheet"]  
tags: [nmap, reconnaissance, network-scanning, security]  
created: 2025-10-28  
summary: "Practical Nmap host discovery and port scanning techniques, with examples and best practices for lab use." 

# Nmap — Live Host Discovery & Port Scanning Cheat Sheet

![Nmap host discovery overview](https://chatgpt.com/c/assets/diagram-host-discovery.png "Overview of host discovery methods")

## Introduction

This note summarizes practical techniques for **live host discovery** and **port scanning** using **Nmap**. It explains common discovery probes, a variety of TCP/UDP scan types, useful flags, and examples you can run during labs (TryHackMe / HTB) or controlled assessments.

> **Important:** Only scan networks when you have explicit authorization. Some scans require elevated privileges (`sudo`/root) to send raw packets.

---

## Table of contents

- [Live Host Discovery](#live-host-discovery)
    
- [Discovery Options & Examples]
    
- [Port Scanning Overview](##Port scanning overview)
    
- [Common TCP/UDP Scan Types]
    
- [Custom Flags & Evading]
    
- [Output, Verbosity & Useful Options]
    
- [Best Practices]
    
- [Example Workflows]
    

---

## Live Host Discovery

Use `-sn` to perform host discovery only (no port scan).

- `-sn` — Ping scan (host discovery only).
    
- `-n` — No DNS resolution (faster for large ranges).
    

### ARP discovery (local LAN)

```bash
nmap -PR -sn 192.168.1.0/24
# -PR : ARP ping (local Ethernet)
# -sn : host discovery only
```

**Behavior:** Sends ARP requests; replies show host is up and reveal MAC address.

### ICMP Echo

```bash
nmap -PE -sn 10.0.0.0/24
# -PE : ICMP Echo Request
```

Reply (Echo Reply) => host is up.

### ICMP Timestamp

```bash
nmap -PP -sn 10.0.0.0/24
# -PP : ICMP Timestamp Request
```

Useful when Echo is filtered; some hosts still respond.

### TCP / UDP probes

Use transport probes when ICMP is blocked:

```bash
nmap -PS -sn 10.0.0.0/24
# -PS : TCP SYN ping (defaults to a port if not specified)
```

- `-PA` (TCP ACK): often elicits RST responses from hosts, indicating liveness:
    

```bash
sudo nmap -PA -sn 10.0.0.0/24
```

- `-PU` (UDP probe):
    

```bash
sudo nmap -PU -sn 10.10.68.220/24
```

If you receive ICMP port unreachable (type 3) from a closed UDP port, host is up. No response may mean open|filtered.

---

## Discovery quick reference

- `-sn` : host discovery only
    
- `-n` : no DNS resolution
    
- `-PR` : ARP ping
    
- `-PE` : ICMP echo
    
- `-PP` : ICMP timestamp
    
- `-PS` : TCP SYN ping
    
- `-PA` : TCP ACK ping
    
- `-PU` : UDP ping
    

---

## Port scanning overview

- **Connect scan** `-sT` — completes full TCP handshake (non-privileged).
    
- **SYN (stealth) scan** `-sS` — sends SYN, analyzes replies (needs root).
    
- **UDP scan** `-sU` — sends UDP datagrams; lack of reply often treated as open|filtered.
    

Examples:

```bash
nmap -sT TARGET          # TCP connect scan (no root)
sudo nmap -sS TARGET     # SYN stealth scan (requires root)
sudo nmap -sU TARGET     # UDP scan (requires root)
```

---

## Common TCP/UDP scan types (what they send & interpretation)

> Behavior depends on target OS and networking devices.

### Null scan (`-sN`)

```bash
sudo nmap -sN TARGET
```

- Sends TCP segments with **no flags**.
    
- Closed ports typically reply with RST. No reply → _open|filtered_.
    

### FIN scan (`-sF`)

```bash
sudo nmap -sF TARGET
```

- Sends FIN flag. No RST reply → _open|filtered_.
    

### Xmas scan (`-sX`)

```bash
sudo nmap -sX TARGET
```

- FIN+PSH+URG. Similar interpretation as `-sF`/`-sN`.
    

### ACK scan (`-sA`)

```bash
sudo nmap -sA TARGET
```

- Useful to detect stateful firewall behavior. Responses often RST; use to infer filtering.
    

### Window scan (`-sW`)

```bash
sudo nmap -sW TARGET
```

- Based on TCP window sizes; supplementary to ACK scans.
    

---

## Custom TCP flags

Craft arbitrary flags for advanced testing:

```bash
nmap --scanflags RSTSYNFIN TARGET
```

---

## Spoofing & evasion (use responsibly)

- **IP spoofing**: `nmap -S 1.2.3.4 target` — you will not receive replies unless you control the spoofed address or capture traffic on-path.
    
- **Decoys**: `nmap -D decoy1,decoy2,ME,target` — hides origin among decoys.
    
- **Fragmentation**: `nmap -f target` or `-ff` — fragments packets to attempt to evade simple filters.
    

---

## Output & verbosity

- `-v`, `-vv` : verbosity
    
- `-d`, `-dd` : debug
    
- `--reason` : show reason for each state
    
- `-oN`, `-oG`, `-oX`, `-oA` : output formats
    

Example:

```bash
sudo nmap -sS -A --reason -vv -oA scan-output 10.10.10.0/24
```

---

## Best practices

- Always get permission.
    
- Prefer ARP (`-PR`) on LANs.
    
- If ICMP blocked, probe with TCP/UDP (`-PS`, `-PA`, `-PU`).
    
- Use `-n` to speed scans by skipping DNS lookups.
    
- For speed & stealth, use `-sS` with privileges.
    
- UDP scans are slow; scan specific ports when possible.
    
- Use `--reason` when results are ambiguous, and follow up with service probes.
    

---

## Example workflows

**LAN host discovery (fast & reliable)**

```bash
nmap -PR -sn 192.168.1.0/24
```

**When ICMP blocked**

```bash
nmap -PS80,443 -sn 10.0.0.0/24
```

**Full port scan (SYN + reason + verbose)**

```bash
sudo nmap -sS -p 1-1024 --reason -vv -oN quick-syn-scan.txt 10.10.10.5
```

**Targeted UDP scan**

```bash
sudo nmap -sU -p 53,67,123,161 --reason 10.10.10.5
```

---

