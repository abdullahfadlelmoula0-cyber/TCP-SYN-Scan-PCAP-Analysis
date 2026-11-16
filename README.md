## TCP SYN Scan Analysis Using Wireshark and TShark

### 1. Objective

The objective of this analysis was to examine a PCAP file, identify the observed network traffic, and determine the source and destination TCP ports involved in the captured SYN packets.

### 2. Initial Traffic Inspection Using Wireshark

I started by opening the PCAP file in **Wireshark** and navigating to:

`Statistics → Protocol Hierarchy`

The protocol hierarchy showed the following traffic:

| Protocol | Packet Count |
|---|---:|
| TCP | Approximately 2,000 |
| ARP | 4 |

I then inspected the TCP packets and observed that the traffic was primarily composed of TCP SYN requests.

The observed communication direction was:

- **Source IP:** `192.168.100.103`
- **Destination IP:** `192.168.100.102`
- **TCP Flag:** SYN

No corresponding SYN/ACK or RST responses were observed in the analyzed capture.

### 3. Identifying Source Ports Using TShark

To investigate which source ports were used to send the SYN requests, I used TShark to extract the TCP source-port field.

Command:

```bash
tshark -r name_file.pcap \
-Y "ip.src == 192.168.100.103 && ip.dst == 192.168.100.102 && tcp.flags.syn == 1 && tcp.flags.ack == 0" \
-T fields -e tcp.srcport | sort -nu
```

The output showed two unique source ports:

- `59660`
- `59661`

This indicates that the captured SYN requests used two distinct TCP source ports.

### 4. Identifying the Scanned Destination Ports

Next, I changed the extracted field from `tcp.srcport` to `tcp.dstport` to identify the destination ports targeted by the SYN requests.

Command:

```bash
tshark -r name_file.pcap \
-Y "ip.src == 192.168.100.103 && ip.dst == 192.168.100.102 && tcp.flags.syn == 1 && tcp.flags.ack == 0" \
-T fields -e tcp.dstport | sort -nu
```

This command displays the unique destination ports targeted by the captured SYN packets.

### 5. Counting Unique Destination Ports

To calculate the total number of unique destination ports, I added `wc -l` at the end of the command:

```bash
tshark -r name_file.pcap \
-Y "ip.src == 192.168.100.103 && ip.dst == 192.168.100.102 && tcp.flags.syn == 1 && tcp.flags.ack == 0" \
-T fields -e tcp.dstport | sort -nu | wc -l
```

**Command breakdown:**

- `tshark -r`: Reads the PCAP file.
- `-Y`: Applies a display filter to select the relevant packets.
- `tcp.flags.syn == 1`: Selects packets with the SYN flag set.
- `tcp.flags.ack == 0`: Excludes SYN/ACK packets.
- `-T fields`: Outputs selected packet fields.
- `-e tcp.dstport`: Extracts the TCP destination port.
- `sort -nu`: Sorts the port numbers numerically and removes duplicates.
- `wc -l`: Counts the resulting lines.

### 6. Findings and Conclusion

The analysis identified TCP SYN-based probing activity from `192.168.100.103` to `192.168.100.102`.

The following findings were obtained:

| Metric | Result |
|---|---:|
| Total TCP packets | Approximately 2,000 |
| ARP packets | 4 |
| Unique TCP source ports | 2 |
| Observed source ports | `59660`, `59661` |
| Unique TCP destination ports | 1,000 |
| Observed SYN/ACK responses | 0 |
| Observed RST responses | 0 |

The results indicate that SYN packets targeted **1,000 unique TCP destination ports** on the target device.

Two distinct source ports were observed in the captured traffic, while 1,000 unique destination ports were identified through TShark analysis.

No corresponding SYN/ACK or RST responses were observed in the analyzed PCAP file. Possible explanations include firewall filtering, packet loss, an unreachable target, or limitations in the packet-capture location.

**Conclusion:** The captured traffic demonstrates TCP SYN-based probing across 1,000 unique destination ports. However, the available capture does not provide sufficient evidence to determine whether those ports were open, closed, or filtered.

### 7. Tools Used

- **Wireshark:** Packet inspection and protocol hierarchy analysis.
- **TShark:** Command-line packet analysis and field extraction.
- **Linux Commands:** `sort`, `wc` for sorting, deduplication, and counting.