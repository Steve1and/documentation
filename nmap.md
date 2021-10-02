# Nmap Notes
***

Network Mapper (`Nmap`) is an open-source network analysis and security auditing tool written in C, C++, Python, and Lua. It is designed to scan networks and identify which hosts are available on the network using raw packets, and services and applications, including the name and version, where possible. It can also identify the operating systems and versions of these hosts.
***

## Nmap Architecture

Nmap can be divided into the following scanning techniques:
- Host discovery
- Port scanning
- Service enumeration and detection
- OS detection
- Scriptable interaction with the target service (Nmap Scripting Engine)

For the TCP-SYN scan (`nmap -sS localhost`), a packet with the SYN flag set is sent. Nmap will populate its results based on the following conditions:
- If the target sends a SYN-ACK flagged packet back to the scanned port, Nmap detects that the port is open.
- If the packet receives an RST flag, it is an indicator that the port is closed.
- If Nmap does not receive a packet back, it will display it as filtered. Depending on the firewall configuration, certain packets may be dropped or ignored by the firewall.
***

## Syntax

`nmap <scan types> <options> <target>`
***

## Host Discovery

The most effective host discovery method is to use **ICMP echo requests**.
- `nmap 10.129.2.0/24 -sn -oA scan_results | grep for | cut -d" " -f5`

A list of hosts can be saved to a text file (for example, `hosts.lst`)
- `nmap -sn -oA scan_results -iL hosts.lst | grep for | cut -d" " -f5`

Scanning multiple IPs:
- `nmap -sn -oA scan_results 10.129.2.18 10.129.2.19 10.129.2.20 | grep for | cut -d" " -f5`

Scan a range of IPs:
- `nmap -sn -oA scan_results 10.129.2.18-20 | grep for | cut -d" " -f5`

Scan a single IP to see if it is alive or not:
- `nmap 10.129.2.18 -sn -oA host`

Ensure that ICMP echo requests are sent, (instead of ARP requests):
- `nmap 10.129.2.18 -sn -oA host -PE --packet-trace`

To see the reason behind a specific host enumeration result, use the `--reason` option:
- `nmap 10.129.2.18 -sn -oA host -PE --reason`

To disable ARP pings, use the `--disable-arp-ping` option:
- `nmap 10.129.2.18 -sn -oA host -PE --packet-trace --disable-arp-ping`
***

## Host and Port Scanning

Six different states for a scanned port:

| State | Description |
| ----- | ----------- |
| open | This indicates that the connection to the scanned port has been established. These connections can be **TCP connections**, **UDP datagrams** as well as **SCTP associations**. |
| closed | When the port is shown as closed, the TCP protocol indicates that the packet we received back contains an RST flag. This scanning method can also be used to determine if our target is alive or not. |
| filtered | Nmap cannot correctly identify whether the scanned port is open or closed because either no response is returned from the target for the port or we can an error code from the target. |
| unfiltered | This state of a port only occurs during the **TCP-ACK** scan and means that the port is accessible, but it cannot be determined whether it is open or closed. |
| open|filtered | If we do not get a response for a specific port, Nmap will set it to that state. This indicates that a firewall or packet filter may protect the port. |
| closed|filtered | This state only occurs in the **IP ID idle** scans and indicates that it was impossible to determine if the scanned port is closed or filtered by a firewall. |

By default, Nmap scans the top 1000 TCP ports with the SYN scan (`-sS`).
- This is only true when run as root, otherwise the TCP scan (`-sT`) is used.

Ports can be defined as:
- One-by-one: `-p 22,25,80,139,445`
- A Range: `-p 22-445`
- Top ports from the Nmap database: `--top-ports=10`
- All ports: `-p-`
- Fast port scan (top 100 ports): `-F`

To have a clear view of a TCP-SYN scan, disable ICMP echo requests (`-Pn`), DNS resolution (`-n`), and ARP ping scans (`--disable-arp-ping`):
- `nmap 10.129.2.28 -p 21 --packet-trace -Pn -n --disable-arp-ping`

UDP port scan:
- `nmap 10.129.2.28 -F -sU`

Nmap sends empty datagrams to the scanned UDP ports. Responses will only be given if an applicaiton is configured to do so while listening on the port.
- `nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 137 --reason`
- If you get the ICMP `error code 3` response (port unreachable), you know that the port is indeed closed.
***

## Saving the Results

Nmap output file formats:
- Normal output (`-oN`) with the `.nmap` file extension.
- Grepable output (`-oG`) with the `.gnmap` file extension.
- XML output (`-oX`) with the `.xml` file extension.
- All formats (`-oA`).

`nmap 10.129.2.28 -p- -oA target`
- Will produce the files: `target.gnmap`, `target.html`, `target.nmap`.

With the XML output, you can create an HTML page:
- `xsltproc target.xml -o target.html`
***

## Service Enumeration

Use the `-sV` option to check for versions of services running on each port.
- `nmap 10.129.2.28 -p- -sV`

Press the space bar during an Nmap scan to see the current status output.
- To show the status of a command at a set interval, use the `--state-every=` command.
  - `nmap 10.129.2.28 -p- -sV --stats-every=5s`

To increase the verbosity of an Nmap scan, use the `-v` option:
- `nmap 10.129.2.28 -p- -sV -v`
***

## Nmap Scripting Engine

Nmap script are written in Lua for interaction with certain services. There are 14 categories into which these scripts can be divided:

| Category | Description |
| -------- | ----------- |
| auth | Determination of authentication credentials. |
| broadcast | Scripts, which are used for host discovery by broadcasting and the discovered hosts, can be automatically added to the remaining scans. |
| brute | Executes scripts that try to log in to the respective service by brute-forcing with credentials. |
| default | Default scripts executed by using the -sC option. |
| discovery | Evaluation of accessible services. |
| dos | These scripts are used to check services for denial of service vulnerabilities and are used less as it harms the services. |
| exploit | This category of scripts tries to exploit known vulnerabilities for scanned port. |
| external | Scripts that use external services for further processing. |
| fuzzer | This uses scripts to identify vulnerabilities and unexpected packet handling by sending different fields, which can take much time. |
| intrusive | Intrusive scripts that could negatively affect the target system. |
| malware | Checks if some malware infects the target system. |
| safe | Defensive scripts that do not perform intrusive and destructive access. |
| version | Extension for service detection. |
| vuln | Identification of specific vulnerabilities. |

Run default scripts: `nmap <target> -sC`

Specify a category of scripts: `nmap <target> --script <category>`

Run a defined script: `nmap <target> --script <script-name>,<script-name>,...`

Scan the SMTP port with the banner and smtp-commands scripts:
- `nmap 10.129.2.28 -p 25 --script banner,smtp-commands`

Scan a target using the aggressive option (`-A`), traceroute, (`--traceroute`), and OS detection (`-O`):
- `nmap 10.129.2.28 -p- -A --traceroute -O`

Vulnerability scan: `nmap 10.129.2.28 -p 80 -sV --script vuln`
***

## Performance

Nmap performance modifying options:
- Speed: `-T <1-5>`
  - `-T 0`: Paranoid slow
  - `-T 1`: Sneaky
  - `-T 2`: Polite
  - `-T 3`: Normal
  - `-T 4`: Aggressive
  - `-T 5`: Insane
- Frequency: `--min-parallelism <number>`
- Timeouts: `--max-rtt-timeout <time>`
- Simultaneous packets sent: `--min-rate <number>`
- Number of retries: `--max-retries <number>`

Default round trip time for Nmap is 100ms.

Optimized RTT: `nmap 10.129.2.0/24 -F --initial-rtt-timeout 50ms --max-rtt-timeout 100ms`
- `-F`: Top 100 ports (fast scan)
***

## Firewall and IDS/IPS Evasion

Nmap's TCP ACK scan (`-sA`) method is much harder to filter for firewalls and IDS/IPS systems than the TCP SYN (`-sS`) or Connect scans (`-sT`) because they only send a TCP packet with the ACK flag. When a port is closed or open, the host must respond with an RST flag. Unlike outgoing connection attempts (with the SYN flag) from external networks are usually blocked by firewalls. However, the packets with the ACK flag are often passed by the firewall because the firewall cannot determine whether the connection was first established from the external network or the internal network.
- `sudo nmap 10.129.2.28 -p 21,22,25 -sS -Pn -n --disable-arp-ping --packet-trace`
- `sudo nmap 10.129.2.28 -p 21,22 -sA -Pn -n --disable-arp-ping --packet-trace`

Use multiple Virtual Private Servers (VPS) with different IP addresses while scanning in-case one is blocked due to an IPS being triggered.

The Decoy option (`-D`) will cause Nmap to generate various random IP addresses and insert them into the IP header to disguise the origin of the packet sent. Use it with `RND:#` to specify the number of random IP addresses to use.
- `sudo nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5`

Use the `-S` option to specify a different source IP for scanning and the `-e` option to specify a different network interface.
- `sudo nmap 10.129.2.28 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0`

By default, Nmap performs a reverse DNS resolution unless otherwise specified to find more important information about a target. Use the `--dns-server <ns>,<ns>` option to specify a DNS server. This is useful if you are sitting in a DMZ and have access to a company's DNS server.
- Use `--source-port 53` to specify TCP port 53 for DNS rather than the default UDP port 53.
- `sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53`

Target: 10.129.176.117
- IDS/IPS Status page: http://10.129.224.173/status.php
Now our client wants to know if it is possible to find out the version of the running services. Submit the version of the service our client was talking about as the answer.
sudo nmap 10.129. -p 50000 -sS -Pn -n --disable-arp-ping --packet-trace
sudo nmap 10.129. -p 50000 -sA -Pn -n --disable-arp-ping --packet-trace
sudo nmap 10.129. -p 50000 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5
sudo nmap 10.129. -n -Pn -p50000 -O
sudo nmap 10.129. -n -Pn -p 50000 -O -S 10.129.2.200 -e tun0
sudo nmap 10.129. -p50000 -sS -Pn -n --disable-arp-ping --packet-trace
sudo nmap 10.129. -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53
sudo ncat -nv --source-port 53 10.129.2.28 50000
