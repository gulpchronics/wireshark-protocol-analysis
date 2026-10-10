# DHCP Protocol Analysis

## 1. Objective

To capture and analyze DHCP traffic using `tcpdump` and Wireshark, understand the DHCP Discover message, and investigate DHCP server communication in a virtualized Linux network.

## 2. Lab Environment

- **Virtualization:** Oracle VirtualBox
- **Operating system:** Ubuntu Linux
- **Client interface:** `enp0s3`
- **Router interface:** `enp0s8`
- **Packet capture:** tcpdump
- **Packet analysis:** Wireshark
- **Network:** `192.168.10.0/24`

## 3. Capture Method

Started packet capture on Client-A:

```bash
sudo tcpdump -ni enp0s3 -w /tmp/dhcp-analysis.pcap 'udp port 67 or udp port 68'
```

Generated DHCP discovery traffic using:

```bash
sudo dhcpcd -T enp0s3
```

The `-T` option performs a test configuration and reports proposed settings. It normally does not apply the proposed configuration to the interface.

The resulting PCAP file was opened in Wireshark and inspected.

## 4. Packet Observations

Two DHCP Discover packets were captured approximately 3.54 seconds apart.

| Field | Observed value |
|---|---|
| Protocol | DHCP |
| Transport protocol | UDP |
| Source IP | `0.0.0.0` |
| Destination IP | `255.255.255.255` |
| Source port | `68` |
| Destination port | `67` |
| DHCP message type | Discover |
| Client MAC address | `08:00:27:2a:5a:c8` |
| Hostname | `client-1` |
| Transaction ID | `0x8f02cf5d` |

## 5. Packet Analysis

### Source and destination addresses

The client uses `0.0.0.0` as its source IPv4 address for the DHCP discovery exchange because it does not have a DHCP-assigned address for that exchange.

The destination `255.255.255.255` is the IPv4 limited-broadcast address. The client broadcasts its request to discover available DHCP servers on the local network.

### UDP ports

DHCP uses UDP port 68 for client messages and UDP port 67 for server messages.

- **UDP 68:** DHCP client.
- **UDP 67:** DHCP server.

The captured Discover messages travel from UDP port 68 to UDP port 67.

### DHCP message type

Wireshark identifies the messages as DHCP Discover. The DHCP Message Type option (53) has value 1.

The packets also include a parameter request list containing options such as subnet mask, default gateway, and DNS server.

## 6. DHCP Server Investigation

A DHCP server was configured on Router-VM using `dnsmasq`.

The intended DHCP range was:

`192.168.10.100–192.168.10.150`

The server was configured to use interface `enp0s8` and advertise `192.168.10.1` as the default gateway.

The configuration included:

```text
interface=enp0s8
bind-dynamic
dhcp-range=192.168.10.100,192.168.10.150,255.255.255.0,12h
dhcp-option=3,192.168.10.1
dhcp-authoritative
log-dhcp
```

The service status showed `dnsmasq` running and its DHCP sockets bound to `enp0s8`.

A router-side packet capture also confirmed that DHCP Discover packets from Client-A reached Router-VM.

However, no DHCP Offer was observed in the captures. The cause of the missing response was not conclusively identified.

## 7. DHCP DORA Status

A typical DHCP address assignment follows four stages:

1. **Discover:** Client searches for a DHCP server.
2. **Offer:** Server offers an IP address.
3. **Request:** Client requests the offered address.
4. **ACK:** Server confirms the lease.

| Stage | Result |
|---|---|
| Discover | Captured |
| Offer | Not observed |
| Request | Not observed |
| ACK | Not observed |

Therefore, this experiment demonstrates DHCP discovery and partial server-side troubleshooting, not a completed DORA exchange or successful DHCP lease assignment.

## 8. Conclusion

This experiment demonstrated how DHCP Discover packets use IPv4 broadcast addressing and UDP ports 68 to 67 to locate a DHCP server.

Wireshark was used to inspect DHCP message fields, client identification, and requested network parameters. A router-side capture verified that discovery packets reached the router interface.

The investigation also highlighted an important troubleshooting principle: receiving a client request does not prove that a server successfully processes or answers it. Further investigation is required to identify why the DHCP Offer was not observed.
