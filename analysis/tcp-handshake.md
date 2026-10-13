# TCP Three-Way Handshake Analysis

## 1. Overview

This project captures and analyzes the TCP three-way handshake using `tcpdump` and Wireshark in a virtualized Linux network lab.

An SSH connection was initiated from Client-A to Client-B to generate real TCP traffic. The captured packets were then examined to understand how TCP establishes a reliable connection before application-layer communication begins.

**Objective:** Identify the SYN, SYN-ACK, and ACK packets and analyze their TCP flags, port numbers, sequence numbers, and acknowledgment numbers.

## 2. Lab Environment

| Component | Details |
|---|---|
| Virtualization | Oracle VirtualBox |
| Client-A | `192.168.10.10` |
| Client-B | `192.168.20.10` |
| Transport protocol | TCP |
| Application protocol | SSH |
| Destination port | TCP 22 |
| Capture tool | `tcpdump` |
| Analysis tool | Wireshark |

Client-A and Client-B are connected to separate subnets through a Linux router. IPv4 forwarding is enabled on the router to allow communication between the two networks.

## 3. Traffic Capture

The following command was executed on Client-A to capture TCP traffic associated with SSH:

```bash
sudo tcpdump -ni enp0s3 -w /tmp/tcp-handshake.pcap 'tcp port 22'
```

An SSH connection was then initiated from Client-A to Client-B:

```bash
ssh <username>@192.168.20.10
```

After the connection was established and the session was closed, the capture was stopped and opened in Wireshark.

The capture was saved as:

`captures/tcp-handshake.pcap`

## 4. TCP Three-Way Handshake

TCP uses a three-step exchange to establish a connection between a client and a server.

### Packet 1 — SYN

- **Source:** `192.168.10.10`
- **Destination:** `192.168.20.10`
- **Source port:** `34322`
- **Destination port:** `22`
- **TCP flag:** SYN

Client-A initiates the connection by sending a SYN packet to Client-B. The client uses an ephemeral source port (`34322`) while connecting to the SSH service on TCP port `22`.

The SYN flag indicates a request to establish a TCP connection.

### Packet 2 — SYN-ACK

- **Source:** `192.168.20.10`
- **Destination:** `192.168.10.10`
- **Source port:** `22`
- **Destination port:** `34322`
- **TCP flags:** SYN, ACK

Client-B responds with SYN-ACK. The SYN flag indicates that the server is participating in connection establishment, while the ACK flag acknowledges the client's initial SYN.

This response confirms that the server received the connection request and is willing to proceed.

### Packet 3 — ACK

- **Source:** `192.168.10.10`
- **Destination:** `192.168.20.10`
- **Source port:** `34322`
- **Destination port:** `22`
- **TCP flag:** ACK

Client-A acknowledges the server's response. This completes the TCP three-way handshake, establishing the TCP connection.

The capture shows the following sequence:

| Packet | Direction | TCP flags | Purpose |
|---|---|---|---|
| 1 | Client-A → Client-B | SYN | Initiates connection |
| 2 | Client-B → Client-A | SYN, ACK | Acknowledges request and responds |
| 3 | Client-A → Client-B | ACK | Completes connection establishment |

## 5. TCP Header Analysis

The capture also exposes important fields in the TCP header.

- **Source and destination ports:** Identify the endpoints of the TCP connection. In this capture, the client uses port `34322` and the server uses port `22`.
- **Sequence number:** Tracks the position of data in the TCP byte stream. Wireshark displays relative sequence numbers by default.
- **Acknowledgment number:** Indicates the next sequence number expected from the other endpoint when the ACK flag is set.
- **Flags:** Control connection establishment and other TCP operations. SYN and ACK are the key flags in this handshake.
- **Window size:** Advertises how much data the receiver is currently prepared to accept.
- **TCP options:** The handshake packets negotiate capabilities such as Maximum Segment Size (MSS), Window Scaling, and SACK Permitted.

These fields help explain how TCP establishes and manages a reliable connection.

## 6. TCP vs. SSH

Although SSH was used to generate the traffic, the three-way handshake belongs to TCP.

The order of events is:

1. TCP establishes the connection using SYN, SYN-ACK, and ACK.
2. SSH exchanges protocol version information and negotiates cryptographic algorithms.
3. SSH establishes encryption and proceeds with authentication.
4. Encrypted SSH traffic is exchanged over the established TCP connection.

The capture shows SSH protocol packets after the initial TCP handshake. This demonstrates how an application-layer protocol uses TCP as its transport mechanism.

## 7. Results

The Wireshark capture confirms the following observations:

- A TCP connection was initiated from Client-A to Client-B.
- The initial SYN was followed by a SYN-ACK from Client-B.
- Client-A sent the final ACK, completing the three-way handshake.
- The connection used client source port `34322` and server destination port `22`.
- SSH negotiation and encrypted traffic followed TCP connection establishment.
- The connection crossed two separate subnets through the Linux router.

## 8. Conclusion

This experiment demonstrates TCP connection establishment through packet-level analysis. By inspecting the SYN, SYN-ACK, and ACK packets, the sequence of events required to establish a TCP connection was identified and explained.

Using an SSH session provided real network traffic for analysis and demonstrated the relationship between TCP transport services and application-layer protocols.

## 9. Evidence

- **Packet capture:** [`tcp-handshake.pcap`](../captures/tcp-handshake.pcap)
- **Wireshark screenshot:** [`tcp-handshake.png`](../screenshots/tcp-handshake.png)

## 10. Skills Demonstrated

- TCP connection establishment and flag analysis
- Packet capture using `tcpdump`
- Packet inspection and filtering in Wireshark
- TCP ports, sequence numbers, and acknowledgment numbers
- Understanding the relationship between TCP and SSH
- Network troubleshooting across routed subnets
