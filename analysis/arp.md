
# ARP Packet Analysis Using Wireshark

## 1. Objective

To capture and analyze Address Resolution Protocol (ARP) traffic in a Linux-based virtual network using `tcpdump` and Wireshark. The objective is to understand how devices resolve IPv4 addresses to MAC addresses on a local network.

## 2. Lab Environment

- **Client:** Client-A (Linux VM)
- **Client IP:** `192.168.10.10`
- **Default gateway:** `192.168.10.1`
- **Capture interface:** `enp0s3`
- **Tools:** Linux `tcpdump`, Wireshark, Oracle VirtualBox

## 3. Procedure

1. Verified Client-A's IP address and network interface.
2. Confirmed connectivity to the default gateway using `ping`.
3. Cleared the cached neighbor entry for `192.168.10.1` to encourage fresh ARP resolution.
4. Captured ARP packets using `tcpdump`.
5. Opened the resulting PCAP file in Wireshark and applied the display filter `arp`.
6. Examined the ARP request and reply packets and compared the observed addresses with the Linux neighbor table.

## 4. Commands Used

Check the network interface and IP address:

```bash
ip -br addr
```

Clear the gateway's neighbor entry:

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

The capture was subsequently copied to the Windows shared folder for analysis in Wireshark.

## 5. Observations

The capture contained four ARP packets, forming two request–reply exchanges.

| Packet | Observed message | Interpretation |
|---|---|---|
| 1 | Who has `192.168.10.1`? Tell `192.168.10.10` | Client-A requests the gateway's MAC address. |
| 2 | `192.168.10.1` is at `08:00:27:6d:9c:9a` | The gateway provides its MAC address. |
| 3 | Who has `192.168.10.10`? Tell `192.168.10.1` | The gateway requests Client-A's MAC address. |
| 4 | `192.168.10.10` is at `08:00:27:2a:5a:c8` | Client-A provides its MAC address. |

### Important ARP fields

- **Opcode:** Identifies whether the packet is an ARP request (`1`) or reply (`2`).
- **Sender IP address:** IPv4 address of the device sending the ARP message.
- **Sender MAC address:** MAC address associated with the sender.
- **Target IP address:** IPv4 address whose MAC address is being resolved.
- **Target MAC address:** Typically unknown and set to `00:00:00:00:00:00` in a standard ARP request.

## 6. Analysis

ARP enables devices on the same local IPv4 network to discover the MAC addresses required for Ethernet delivery. When Client-A needs to send traffic to its default gateway, it must determine the gateway's MAC address because the gateway is on the same subnet.

The ARP request is broadcast on the local network because the sender does not yet know the target's MAC address. The device that owns the requested IP address can then respond with an ARP reply.

The capture also showed a request from the gateway for Client-A's MAC address. The subsequent neighbor-table inspection confirmed the corresponding IP-to-MAC mappings on both devices. However, the capture alone does not establish the precise cache state that caused the gateway to issue its request.

## 7. Conclusion

This lab demonstrated how ARP resolves IPv4 addresses to MAC addresses on a local network. Using `tcpdump` and Wireshark, I captured and examined ARP request–reply exchanges and identified the relevant protocol fields.

This exercise improved my understanding of packet capture, protocol filtering, neighbor resolution, and evidence-based network troubleshooting.

## 8. Evidence

- `captures/arp-analysis.pcap` — packet capture for independent inspection.
- `screenshots/arp-analysis.png` — Wireshark screenshot showing the captured ARP exchanges and expanded protocol fields.
