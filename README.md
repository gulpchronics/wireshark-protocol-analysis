# Wireshark Protocol Analysis

A hands-on network protocol analysis project built on my Linux Network Security Lab using Ubuntu VMs, `tcpdump`, and Wireshark.

The goal is to understand how network communication works at the packet level, identify important protocol fields, and develop practical troubleshooting skills through real traffic captures.

## Project Background

This project extends my existing Linux Network Security Lab, where I configured a Linux gateway router and two client VMs in VirtualBox. Instead of relying only on command output to verify connectivity, I am capturing and examining the packets exchanged between devices.

The focus is on understanding what happens behind common networking operations such as gateway resolution, IP configuration, name resolution, TCP connections, and SSH sessions.

## Lab Environment

- **Virtualization:** Oracle VirtualBox
- **Operating system:** Ubuntu Linux VMs
- **Packet capture:** `tcpdump`
- **Packet analysis:** Wireshark
- **Network testing:** `ping`, Linux networking commands
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
| DHCP | Analyze the address assignment process (DORA) | Planned |
| DNS | Inspect DNS queries and responses | Planned |
| TCP | Analyze SYN, SYN-ACK, and ACK | Planned |
| SSH | Observe connection establishment and encrypted traffic | Planned |

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
sudo tcpdump -ni enp0s3 \
-w /tmp/arp-analysis.pcap arp
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

## Repository Structure

```text
wireshark-protocol-analysis/
├── README.md
├── captures/
│   └── arp-analysis.pcap
├── screenshots/
│   └── arp-analysis.png
└── analysis/
    └── arp.md
```

Additional capture files, screenshots, and reports will be added as the remaining protocol labs are completed.

## Tools and Skills Practiced

- Linux networking commands
- Packet capture with `tcpdump`
- Display filters and packet inspection in Wireshark
- ARP and neighbor-cache troubleshooting
- VirtualBox networking
- Technical documentation and evidence-based analysis

## Next Steps

1. Capture and analyze DHCP traffic.
2. Inspect DNS queries and responses.
3. Examine the TCP three-way handshake.
4. Capture an SSH session and investigate its encrypted traffic.
5. Consolidate the observations into a complete protocol-analysis report.

## Purpose

This project is part of my practical networking and network-security learning journey. It builds on my Linux gateway lab and provides packet-level evidence of how network protocols behave in a controlled virtual environment.
