# 三、 选项概要

## 三、 选项概要

当 Nmap 不带选项运行时，该选项概要会被输出，最新的版本在这里 [`www.insecure.org/nmap/data/nmap.usage.txt`](http://www.insecure.org/nmap/data/nmap.usage.txt)。它帮助人们记住最常用的选项，但不 能替代本手册其余深入的文档，一些晦涩的选项甚至不在这里。

```
Usage: nmap [Scan Type(s)] [Options] {target specification}
TARGET SPECIFICATION:
    Can pass hostnames, IP addresses, networks, etc.
    Ex: scanme.nmap.org, microsoft.com/24, 192.168.0.1; 10.0-255.0-255.1-254
    -iL <inputfilename>: Input from list of hosts/networks
    -iR <num hosts>: Choose random targets
    --exclude <host1[,host2][,host3],...>: Exclude hosts/networks
    --excludefile <exclude_file>: Exclude list from file
HOST DISCOVERY:
    -sL: List Scan - simply list targets to scan
    -sP: Ping Scan - go no further than determining if host is online
    -P0: Treat all hosts as online -- skip host discovery
    -PS/PA/PU [portlist]: TCP SYN/ACK or UDP discovery probes to given ports
    -PE/PP/PM: ICMP echo, timestamp, and netmask request discovery probes
    -n/-R: Never do DNS resolution/Always resolve [default: sometimes resolve]
SCAN TECHNIQUES:
    -sS/sT/sA/sW/sM: TCP SYN/Connect()/ACK/Window/Maimon scans
    -sN/sF/sX: TCP Null, FIN, and Xmas scans
    --scanflags <flags>: Customize TCP scan flags
    -sI <zombie host[:probeport]>: Idlescan
    -sO: IP protocol scan
    -b <ftp relay host>: FTP bounce scan
PORT SPECIFICATION AND SCAN ORDER:
    -p <port ranges>: Only scan specified ports
    Ex: -p22; -p1-65535; -p U:53,111,137,T:21-25,80,139,8080
    -F: Fast - Scan only the ports listed in the nmap-services file)
    -r: Scan ports consecutively - don't randomize
SERVICE/VERSION DETECTION:
    -sV: Probe open ports to determine service/version info
    --version-light: Limit to most likely probes for faster identification
    --version-all: Try every single probe for version detection
    --version-trace: Show detailed version scan activity (for debugging)
OS DETECTION:
    -O: Enable OS detection
    --osscan-limit: Limit OS detection to promising targets
    --osscan-guess: Guess OS more aggressively
TIMING AND PERFORMANCE:
    -T[0-6]: Set timing template (higher is faster)
    --min-hostgroup/max-hostgroup <msec>: Parallel host scan group sizes
    --min-parallelism/max-parallelism <msec>: Probe parallelization
    --min_rtt_timeout/max-rtt-timeout/initial-rtt-timeout <msec>: Specifies probe round trip time.
    --host-timeout <msec>: Give up on target after this long
    --scan-delay/--max_scan-delay <msec>: Adjust delay between probes
FIREWALL/IDS EVASION AND SPOOFING:
    -f; --mtu <val>: fragment packets (optionally w/given MTU)
    -D <decoy1,decoy2[,ME],...>: Cloak a scan with decoys
    -S <IP_Address>: Spoof source address
    -e <iface>: Use specified interface
    -g/--source-port <portnum>: Use given port number
    --data-length <num>: Append random data to sent packets
    --ttl <val>: Set IP time-to-live field
    --spoof-mac <mac address, prefix, or vendor name>: Spoof your MAC address
OUTPUT:
    -oN/-oX/-oS/-oG <file>: Output scan results in normal, XML, s|<rIpt kIddi3, and Grepable format, respectively, to the given filename.
    -oA <basename>: Output in the three major formats at once
    -v: Increase verbosity level (use twice for more effect)
    -d[level]: Set or increase debugging level (Up to 9 is meaningful)
    --packet-trace: Show all packets sent and received
    --iflist: Print host interfaces and routes (for debugging)
    --append-output: Append to rather than clobber specified output files
    --resume <filename>: Resume an aborted scan
    --stylesheet <path/URL>: XSL stylesheet to transform XML output to HTML
    --no_stylesheet: Prevent Nmap from associating XSL stylesheet w/XML output
MISC:
    -6: Enable IPv6 scanning
    -A: Enables OS detection and Version detection
    --datadir <dirname>: Specify custom Nmap data file location
    --send-eth/--send-ip: Send packets using raw ethernet frames or IP packets
    --privileged: Assume that the user is fully privileged
    -V: Print version number
    -h: Print this help summary page.
EXAMPLES:
    nmap -v -A scanme.nmap.org
    nmap -v -sP 192.168.0.0/16 10.0.0.0/8
    nmap -v -iR 10000 -P0 -p 80 
```