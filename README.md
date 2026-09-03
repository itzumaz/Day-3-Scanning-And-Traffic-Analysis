# Day 3: Targeted Network Scanning & Traffic Analysis

## 📌 Project Overview
This repository contains the official documentation, methodologies, and network traffic captures compiled during **Day 3 of my Vulnerability Assessment and Penetration Testing (VAPT) Internship with TriosCyber** (in partnership with Ernith). 

The primary objective of this project was to perform targeted TCP scanning, selective UDP scanning, and real-time packet analysis using **Nmap** and **Wireshark** within an authorized, isolated local lab environment to evaluate service exposure and analyze packet-level behavior.

---

## 🛠️ Lab Environment & Topology
* **Attacker System:** Kali Linux (`192.168.145.128`)
* **Target System:** Metasploitable 2 (`192.168.145.132`)
* **Network Interface:** `eth0`
* **Tools Utilized:** Nmap, Wireshark

---

## 🚀 Execution & Methodology

### 1. Targeted TCP Scanning
To map high-priority target services while minimizing our network footprint, a stealth **SYN scan (`-sS`)** was executed against specific TCP ports (FTP, SSH, HTTP, and SMB).

```bash
sudo nmap -sS -p 21,22,80,445 192.168.145.132
```

**Results:**

| Port | Protocol | State | Service |
| :--- | :--- | :--- | :--- |
| **21** | TCP | Open | FTP (vsftpd) |
| **22** | TCP | Open | SSH (OpenSSH) |
| **80** | TCP | Open | HTTP (Apache) |
| **445** | TCP | Open | SMB (Samba) |

---

### 2. Selective UDP Scanning
Because the UDP protocol is connectionless, full-range scanning can trigger severe network latency. To optimize efficiency, a targeted **UDP scan (`-sU`)** was directed at core common service ports.

```bash
sudo nmap -sU -p 53,67,137,161 192.168.145.132
```

**Results:**

| Port | Protocol | State | Service |
| :--- | :--- | :--- | :--- |
| **53** | UDP | Open | Domain (DNS) |
| **67** | UDP | Closed | DHCP Server |
| **137** | UDP | Open | NetBIOS Name Service |
| **161** | UDP | Closed | SNMP |

---

### 3. Packet Analysis with Wireshark
While Nmap was running, Wireshark actively monitored raw network traffic on interface `eth0`. 

**Applied Wireshark Filter:**
```plaintext
ip.addr == 192.168.145.132 && (tcp.port == 80 || udp.port == 53)
```

#### 🔍 Observed TCP Port 80 Handshake Sequence:
1. **SYN Frame (Frame 38):** The Kali Attacker (`192.168.145.128`) sends a `TCP SYN` packet to port 80 on the Metasploitable target (`192.168.145.132`) to check if the port is listening.
2. **SYN-ACK Frame (Frame 50):** The target responds with a `SYN-ACK`, confirming that port 80 is open and actively accepting connections.
3. **RST Frame (Frame 51):** Rather than completing the full 3-way handshake with an `ACK`, the attacker host immediately terminates the attempt by sending a `RST` (Reset) packet. This keeps the scan stealthy and prevents the connection from logging as a full session.

---

## 🛡️ Remediation & Defensive Recommendations
Exposing open ports running outdated or unneeded services significantly increases an organization's attack surface. To secure the target environment from your end, implement the following mitigations:

* **Disable Unused Protocols & Services:** 
  If services like FTP (Port 21) or NetBIOS (Port 137) are not mission-critical, permanently disable or stop them to limit entry points.
* **Implement Strict Host Firewalls (iptables / UFW):** 
  Restrict access to open administrative ports (such as SSH on Port 22). Configure the firewall to only allow connections from specific, authorized management IP addresses.
* **Mitigate UDP Information Leakage:** 
  For open services like DNS (Port 53), verify zone transfer settings are securely locked down to prevent external attackers from mapping internal network architecture.
* **Upgrade and Patch Legacy Applications:** 
  Ensure the underlying service banners (like Apache, Samba, and OpenSSH) are regularly patched to remediate known public vulnerabilities and exploits.

---

## 🧠 Key Takeaways
* **Targeted Efficiency:** Scanning explicitly targeted ports vastly reduces execution times and lowers the chance of triggering Network Intrusion Detection Systems (NIDS).
* **Handshake Verification:** Wireshark verification visually proved the underlying mechanism of a stealth SYN scan—interacting with target ports without finalizing full TCP links.
* **UDP Dynamics:** Observed how Nmap infers UDP status by waiting for a response or parsing incoming ICMP "Port Unreachable" packets.

---

## 📂 Repository Structure
```plaintext
.
├── README.md
├── screenshots/
│   ├── nmap_tcp_udp_scan.png
│   └── wireshark_packet_capture.png
└── captures/
    └── wireshark_eth0OXZZU3.pcapng
```

---

## 👤 Author
**Azeez Umar Opeyemi**
* 💼 **Role:** VAPT Intern at TriosCyber
* 📧 **Email:** umaropeyemiazeez@gmail.com
* 🐙 **GitHub:** [itzumaz](https://github.com/itzumaz)
*   **LinkedIn:** [Azeez Umar Opeyemi](https://www.linkedin.com/in/azeez-umar-opeyemi-201a433a4/)