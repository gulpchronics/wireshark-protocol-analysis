# Wireshark Protocol Analysis

A hands-on network protocol analysis project built on my [Linux Network Security Lab](https://github.com/gulphchronics/linux-network-security-lab) using Ubuntu VMs, `tcpdump`, and Wireshark.

The goal is to understand how network communication works at the packet level, identify important protocol fields, and develop practical troubleshooting skills through real traffic captures.

## Project Background

This project extends my existing Linux Network Security Lab, where I configured a Linux gateway router and two client VMs in VirtualBox. Instead of relying only on command output to verify connectivity, I capture and examine packets exchanged between devices.

The focus is on understanding what happens behind common networking operations such as gateway resolution, IP configuration, name resolution, TCP connections, and SSH sessions.

## Lab Environment

- **Virtualization:** Oracle VirtualBox
- **Operating system:** Ubuntu Linux VMs
- **Packet capture:** `tcpdump`
- **Packet analysis:** Wireshark
- **Network testing:** `ping` and Linux networking commands
- **Capture format:** PCAP

### Network Topology

```text
                 Router-VM
              Linux Gateway
               /          \
         192.168.10.1   192.168.20.1
              /              \
          Client-A          Client-B
       192.168.10.10     192.168.20.10
           LAN-A             LAN-B
```

The VMs use separate VirtualBox internal networks connected through the Linux gateway. Client-A is used to generate traffic and capture packets for analysis.

## Project Objectives

- Capture real network traffic using `tcpdump`.
- Analyze protocol headers and packet fields in Wireshark.
- Understand ARP resolution and how devices discover MAC addresses.
- Examine DHCP address assignment and DNS name resolution.
- Analyze TCP connection establishment through the three-way handshake.
- Observe SSH connection traffic and understand the role of encryption.
- Document observations with PCAP files, screenshots, and technical notes.
- Build practical packet-analysis skills for networking and network-security troubleshooting.

## Protocol Analysis Progress

| Protocol | Objective | Status |
|---|---|---|
| ARP | Examine IP-to-MAC address resolution | Completed |
| DHCP | Investigate the DHCP address assignment process (DORA) | Investigated; no complete DORA exchange captured |
| DNS | Inspect DNS queries and responses | Queries captured; responses not observed |
| TCP | Analyze SYN, SYN-ACK, and ACK | Completed |
| SSH | Inspect version identification, key exchange, and encrypted traffic | Captured and analyzed |

## Completed Lab: ARP Analysis

### Objective

Capture ARP traffic between Client-A and the Linux gateway and examine how the devices resolve each other's MAC addresses.

### Commands Used

Check the network interface:

```bash
ip -br addr
```

Clear the cached gateway entry:

```bash
sudo ip neigh flush to 192.168.10.1
```

Capture ARP packets:

```bash
sudo tcpdump -ni enp0s3 -w /tmp/arp-analysis.pcap arp
```

Generate traffic:

```bash
ping -c 4 192.168.10.1
```

The capture was saved locally and copied to the VirtualBox shared folder for analysis in Windows Wireshark.

### Observations

The capture contained four ARP packets forming two request–reply exchanges.

| Packet | Observation |
|---|---|
| 1 | Client-A requested the MAC address for `192.168.10.1`. |
| 2 | The gateway replied with MAC `08:00:27:6d:9c:9a`. |
| 3 | The gateway requested the MAC address for `192.168.10.10`. |
| 4 | Client-A replied with MAC `08:00:27:2a:5a:c8`. |

### What I Learned

- ARP resolves IPv4 addresses to MAC addresses on a local network.
- ARP requests are broadcast when the sender needs to discover a target's MAC address.
- ARP replies provide the requested IP-to-MAC mapping.
- Linux neighbor tables can be inspected using `ip neigh show`.
- Wireshark makes it possible to inspect protocol fields rather than relying only on connectivity tests.

The additional ARP exchange from the gateway was observed in the capture. The available evidence does not establish the exact neighbor-cache state that triggered it.

### ARP Evidence

- **Capture:** [`arp-analysis.pcap`](captures/arp-analysis.pcap)
- **Screenshot:** [`arp-analysis.png`](screenshots/arp-analysis.png)
- **Detailed report:** [`arp.md`](analysis/arp.md)

## DHCP Investigation

A DHCP capture on Client-A showed repeated DHCP Discover messages sent from `0.0.0.0:68` to the broadcast address `255.255.255.255:67`.

No DHCP Offer was observed in the captured packets, so a complete DORA sequence was not captured. Client-A already had the manually configured address `192.168.10.10/24`. The investigation did not establish a successful DHCP lease exchange.

This result is documented as an incomplete investigation rather than a successful DORA demonstration.

- **Capture:** [`dhcp-analysis.pcap`](captures/dhcp-analysis.pcap)
- **Screenshot:** [`dhcp-analysis.png`](screenshots/dhcp-analysis.png)
- **Detailed report:** [`dhcp.md`](analysis/dhcp.md)

## DNS Investigation

The DNS capture contains repeated queries for the A record of `example.com` from Client-A (`192.168.10.10`) to DNS server `10.226.81.32` over UDP port 53.

No DNS responses appear in this capture. This records what was observed during the test; it does not, by itself, establish the underlying cause.

- **Capture:** [`dns-analysis.pcap`](captures/dns-analysis.pcap)
- **Screenshot:** [`dns-analysis.png`](screenshots/dns-analysis.png)

## TCP Three-Way Handshake

The TCP capture shows a connection sequence between Client-A (`192.168.10.10`) and Client-B (`192.168.20.10`) using the SSH service port.

1. **SYN:** Client-A initiates a TCP connection from an ephemeral source port to destination port `22`.
2. **SYN-ACK:** Client-B acknowledges the request and responds.
3. **ACK:** Client-A acknowledges the response, completing the TCP three-way handshake.

The capture also contains the SSH exchange that follows the TCP handshake.

- **Capture:** [`tcp-handshake.pcap`](captures/tcp-handshake.pcap)

## SSH Protocol Analysis

The existing TCP capture was reused to examine SSH traffic. With the Wireshark display filter `ssh`, the packet list shows SSH version identification, key-exchange messages, and encrypted packets.

### Observations

- **Client identification:** `SSH-2.0-OpenSSH_10.2p1 Ubuntu-2ubuntu3.5`
- **Server identification:** `SSH-2.0-OpenSSH_10.2p1 Ubuntu-2ubuntu3.7`
- Client and server `Key Exchange Init` messages are visible.
- The capture includes a hybrid key-exchange initialization and server reply.
- Later packets are identified as encrypted SSH traffic in both directions.

TCP establishes the transport connection first. SSH then performs its own protocol negotiation and key exchange over that connection. Packet metadata remains visible, while encrypted SSH application contents are not normally readable without appropriate session decryption material.

### SSH Evidence

![SSH protocol analysis](screenshots/ssh-analysis.png)

- **Capture:** [`tcp-handshake.pcap`](captures/tcp-handshake.pcap)
- **Detailed report:** [`ssh-analysis.md`](analysis/ssh-analysis.md)

## Repository Structure

```text
wireshark-protocol-analysis/
├── README.md
├── captures/
│   ├── arp-analysis.pcap
│   ├── dhcp-analysis.pcap
│   ├── dns-analysis.pcap
│   └── tcp-handshake.pcap
├── screenshots/
│   ├── arp-analysis.png
│   ├── dhcp-analysis.png
│   ├── dns-analysis.png
│   └── ssh-analysis.png
└── analysis/
    ├── arp.md
    ├── dhcp.md
    └── ssh-analysis.md
```

## Tools and Skills Practiced

- Linux networking commands
- Packet capture with `tcpdump`
- Display filters and packet inspection in Wireshark
- ARP and neighbor-cache troubleshooting
- Investigation of incomplete DHCP and DNS exchanges
- TCP connection analysis
- SSH key-exchange and encrypted-traffic identification
- VirtualBox networking
- Technical documentation and evidence-based analysis

## Purpose

This project is part of my practical networking and network-security learning journey. It builds on my Linux gateway lab and provides packet-level evidence of how network protocols behave in a controlled virtual environment.
