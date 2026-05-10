# TCP Handshake
Filters for TCP setup, reset, or close.

## TCP Flags
Following show side by side comparison of filters available for displaying or capturing frames with various TCP flags.

| Meaning (TCP Session Phase)                                                                                  | Display Filter                                                                                          | Capture Filter                                |
|--------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------|-----------------------------------------------|
| Only the SYN frame (first step of 3 way handshake)                                                           | `"(tcp.flags.syn == 1 and tcp.flags.ack == 0 and tcp.len == 0 and !tcp.flags.fin and !tcp.flags.rst)"`  | `"tcp[tcpflags] == tcp-syn"`                  |
| Only the SYN-ACK frame (second step of 3 way handshake)                                                      | `"(tcp.flags.syn == 1 and tcp.flags.ack == 1)"`                                                         | `"tcp[tcpflags] == (tcp-syn\|tcp-ack)"`       |
| Only the ACK frame (third step of 3 way handshake or ACK corresponding to FIN or RST or keepalive empty ACK) | `"(tcp.flags.syn==0 and tcp.flags.ack==1 and tcp.len==0)"`                                              | `"tcp[tcpflags] == tcp-ack and tcp[14] == 0"` |
| Only the DATA frames (after 3 way handshake but before RST or FIN)                                           | `"tcp.len > 0 and not tcp.flags.syn and not tcp.flags.ack and not tcp.flags.fin and not tcp.flags.rst"` | `"tcp[tcpflags] & 0x1F == 0 and tcp[14] > 0"` |
| Only the FIN frame                                                                                           | `"tcp.flags.fin == 1 and tcp.flags.ack == 0 and tcp.flags.rst == 0"`                                    | `"tcp[tcpflags] == tcp-fin != 0"`             |
| Only the RST frame                                                                                           | `"tcp.flags.rst == 1 and tcp.flags.ack == 0 and tcp.flags.fin == 0"`                                    | `"tcp[tcpflags] == tcp-rst != 0"`             |

Note:
`tcp.flags.syn == 1` is the same as `tcp.flags.syn == True` so `(tcp.flags.syn == 1 and tcp.flags.ack == 1)` is the same as `(tcp.flags.syn == True and tcp.flags.ack == True)`.
Correspondingly `tcp[tcpflags] & 0x1F == 0` is the same as `tcp[tcpflags] & 0x1F == False` which means the check against frames with TCP flags FIN, SYN, RST, PUSH, ACK return False.

If the aim is to filter for frames related to 3 way handshake or tear down use the following filters.
```shell
## Display Filter
PCAP=<pcap file name>
tshark -tad -r ${PCAP} -Y "(tcp.flags.syn == 1 and tcp.flags.ack == 0 and tcp.len == 0 and !tcp.flags.fin and !tcp.flags.rst) or (tcp.flags.syn == 1 && tcp.flags.ack == 1) or (tcp.flags.syn == 0 && tcp.flags.ack == 1 && tcp.len == 0) or (tcp.flags.fin == 1 and tcp.flags.ack == 0 and tcp.flags.rst == 0) or (tcp.flags.rst == 1 and tcp.flags.ack == 0 and tcp.flags.fin == 0)"

## Capture Filter
tcpdump -i any '(tcp[tcpflags] == tcp-syn) or (tcp[tcpflags] == (tcp-syn|tcp-ack) or (tcp[tcpflags] == tcp-ack and tcp[14] == 0) or (tcp[tcpflags] == tcp-fin != 0) or (tcp[tcpflags] == tcp-rst != 0)'
```

## TCP Completeness
Applicable only for Wireshark / tshark version >= 3.6.0 as display filter, `tcp.completeness` is not in the header but available only from the Wireshark Expert Analysis process.

Use following to validate if `tcp.completeness` is supported or not.
```shell
tshark --version | awk 'NR==1 {version=$2; split(version,v,"."); print "Major:", v[1], " Minor:", v[2]; exit !(v[1]>3 || (v[1]=3 and v[2]>=6) ) }' and echo "✅ SUPPORTED" || echo "❌ NOT SUPPORTED"
tshark -G fields | grep -i "tcp.completeness" and echo "✅ SUPPORTED" || echo "❌ NOT SUPPORTED"
```

Using `tcp.completeness == 3` means finding TCP streams that meet the following criteria:
* Finished 3 way handshake.
* Not having exchanged any data.
* Not having reached either TCP RST or TCP FIN states.

There is no equivalent display filter using `tcp.flags` or capture filter using `tcp[tcpflags]` because these would first display the 3 way handshake frames as they are found and continue to display any additional frames with payload, RST, or FIN frames hence TCP Completeness can not be made available as a capture filter.

| tcp.completeness Value | Meaning (TCP Session Phase)                                                                                                     | tcp.completeness based display filter |
|------------------------|---------------------------------------------------------------------------------------------------------------------------------|---------------------------------------|
| 1                      | Only the SYN frame (first step of 3 way handshake) have been observed                                                           | `"tcp.completeness == 1"`             |
| 2                      | Only the SYN-ACK frame (second step of 3 way handshake) have been observed                                                      | `"tcp.completeness == 2"`             |
| 4                      | Only the ACK frame (third step of 3 way handshake or ACK corresponding to FIN or RST or keepalive empty ACK) have been observed | `"tcp.completeness == 4"`             |
| 8                      | Only DATA frames (after 3 way handshake but before RST or FIN) have been observed                                               | `"tcp.completeness == 8"`             |
| 16                     | Only the FIN frame have been observed                                                                                           | `"tcp.completeness == 16"`            |
| 32                     | Only the RST frame have been observed                                                                                           | `"tcp.completeness == 32"`            |

## Variations
| tcp.completeness Value | Meaning (TCP Session Phase)                                                 | filter                     |
|------------------------|-----------------------------------------------------------------------------|----------------------------|
| 3                      | SYN + SYN-ACK packets (first 2 steps of three-way handshake)                | `"tcp.completeness == 3"`  |
| 7                      | Complete three-way handshake (SYN + SYN-ACK + ACK)                          | `"tcp.completeness == 7"`  |
| 15                     | Complete three-way handshake and data exchange (SYN + SYN-ACK + ACK + DATA) | `"tcp.completeness == 15"` |
| 31                     | Complete TCP conversation (SYN + SYN-ACK + ACK + DATA + FIN)                | `"tcp.completeness == 31"` |
| 33                     | Rejection by server (SYN + RST)                                             | `"tcp.completeness == 33"` |

## Reference

### Filter Gotchas
Note TCP flags are just their hex value equivalents (e.g. tcp-syn == 0x02):
* `tcp-syn` means only the tcp-syn flag is present (e.g. first step of a 3 way handshake), and not any frames with tcp-syn included (e.g. tcp-syn and tcp-ack on second step of a 3 way handshake).
* Display filter `(tcp.flags.syn or tcp.flags.ack)` and capture filter `(tcp-syn|tcp-ack)` are both bitwise OR of flags hence `tcp[tcpflags]==(tcp-syn\|tcp-ack)` means both tcp-syn and tcp-ack flags are present on the same frame (e.g. second step of a 3 way handshake) rather than frames with either tcp-syn or tcp-ack or both tcp-syn and tcp-ack flags present.

| Display Filter Flag | Capture Filter Flag | Hex Value  | Bit Position  | Flag Meaning                   |
|---------------------|---------------------|------------|---------------|--------------------------------|
| tcp.flags.fin       | tcp-fin	            | 0x01	      | Bit 0         | Finish (Connection Close)      |
| tcp.flags.syn       | tcp-syn	            | 0x02	      | Bit 1         | Synchronize (Handshake Start)  |
| tcp.flags.rst       | tcp-rst	            | 0x04	      | Bit 2         | Reset (Abort Connection)       |
| tcp.flags.push      | tcp-push	           | 0x08	      | Bit 3         | Push (Immediate Data Delivery) |
| tcp.flags.ack       | tcp-ack	            | 0x10	      | Bit 4         | Acknowledgment                 |
| tcp.flags.urg       | tcp-urg	            | 0x20	      | Bit 5         | Urgent Pointer                 |
| tcp.flags.ece       | tcp-ece	            | 0x40	      | Bit 6         | ECN-Echo                       |
| tcp.flags.cwr       | tcp-cwr	            | 0x80	      | Bit 7         | Congestion Window Reduced      |
