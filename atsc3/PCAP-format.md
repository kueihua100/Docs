# PCAP File Format
The pcap file format is defined as a standard for capturing packets of network data, described [here](https://wiki.wireshark.org/Development/LibpcapFileFormat).

The data is organized in a binary file consisting of a global header, followed by packets of data each with individual headers.  
```
|Global Header|Packet Header|Packet Data|Packet Header|Packet Data|...|
```

#### Global Header (24 Bytes)
| bytes | type | Name | Description |
|-------|------|------| ----------- |
| 4 | uint32 | magic_number   | 'A1B2C3D4' means the endianness is correct |
| 2 | uint16 | version_major  | major number of the file format  |
| 2 | uint16 | version_minor  | minor number of the file format  |
| 4 | int32  | thiszone       | correction time in seconds from UTC to local time (0) |
| 4 | uint32 | sigfigs        | accuracy of time stamps in the capture (0) |
| 4 | uint32 | snaplen        | max length of captured packed (65535)  |
| 4 | uint32 | network        | type of data link (1 = ethernet)  |

# Packet Header
### PCAP Packet Header (16 Bytes)
| bytes | type | Name | Description |
|-------|------|------| ----------- |
| 4 | uint32 | ts_sec   | timestamp seconds |
| 4 | uint32 | ts_usec  | timestamp microseconds  |
| 4 | uint32 | incl_len | number of octets of packet saved in file  |
| 4 | uint32 | orig_len | actual length of packet  |

### Ethernet Header (14 Bytes)
| bytes | Name |
|-------|------|
| 6  | Destination MAC address| 
| 6  | Source MAC address |
| 2  | Ethernet Type |

### IPv4 Header (20 Bytes)
| bits | Name |
|-------|------|
| 4  | IP Version Number (4)| 
| 4  | IHL |
| 8  | Type of Service |
| 16 | Total Length |
| 16 | Identification |
| 4  | Flags |
| 12 | Fragment Offset |
| 8  | Time to Live |
| 8  | Protocol |
| 16 | Header Checksum |
| 32 | Source Address |
| 32 | Destination Address|

### UDP Header (8 Bytes)
| bytes | type | Name | Description |
|-------|------|------| ----------- |
| 2 | uint16 | source port   | ```2368 = data``` ```8308 = position``` |
| 2 | uint16 | destination port  |   |
| 2 | uint16 | length | number of bytes in packet  |
| 2 | uint16 | checksum |  |



