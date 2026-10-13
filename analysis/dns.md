# DNS Protocol Analysis

## 1. Objective

To capture and analyze DNS queries using `tcpdump` and Wireshark, understand DNS packet fields, and investigate why the client did not receive a DNS response.

## 2. Lab Environment

- Virtualization: Oracle VirtualBox
- Client: Ubuntu Linux (Client-A)
- Client IP: `192.168.10.10`
- DNS server queried: `10.226.81.32`
- Capture tool: `tcpdump`
- Analysis tool: Wireshark

## 3. Capture Method

Captured DNS traffic on Client-A's network interface:

```bash
sudo tcpdump -ni enp0s3 -w /tmp/dns-analysis.pcap 'port 53'
```

Generated DNS requests using:

```bash
nslookup example.com 10.226.81.32
```

The capture was opened in Wireshark with the display filter:

```text
dns
```

## 4. Observations

Three DNS query packets were captured.

| Field | Observed value |
|---|---|
| Source IP | `192.168.10.10` |
| Destination IP | `10.226.81.32` |
| Protocol | DNS over UDP |
| Destination port | `53` |
| Query name | `example.com` |
| Query type | A (IPv4 address record) |
| Queries captured | 3 |
| Responses captured | 0 |

## 5. Packet Analysis

### DNS query

The client sent a DNS query for `example.com`, requesting an A record. An A record maps a hostname to an IPv4 address.

### UDP port 53

The destination port is UDP 53, the standard port used for traditional DNS queries. DNS can also use TCP in other circumstances.

### Repeated queries

The three queries were separated by approximately five seconds. This pattern is consistent with retries after a response timeout.

### Missing response

No DNS response was visible in this capture. The `nslookup` command also timed out when querying `10.226.81.32` directly from Client-A.

The capture confirms that requests left Client-A's interface. It does not establish whether the router forwarded them, whether the DNS server received them, or exactly where a response was lost.

## 6. Troubleshooting Findings

Router-VM successfully resolved `example.com` using its configured resolver, while Client-A's direct query to `10.226.81.32` timed out.

This indicates a difference between the router's working DNS path and Client-A's path. Possible causes include routing, forwarding, firewall rules, or DNS reachability. The exact cause has not yet been confirmed.

## 7. Conclusion

This experiment demonstrates how to capture DNS queries and inspect their source and destination addresses, query name, record type, and transport port.

The capture also demonstrates a practical troubleshooting principle: observing a request leaving a client does not prove that the server received it or that a reply returned.

**Result:** DNS query capture completed; a successful query-response exchange was not observed.
