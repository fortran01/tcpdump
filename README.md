# tcpdump

All the `tcpdump` you need in your life.

Got help from friendly AI bots! 🤖✨

- [tcpdump](#tcpdump)
  - [Monitor SYN packets from a specific host to internal networks](#monitor-syn-packets-from-a-specific-host-to-internal-networks)
    - [TCP Flags Explained](#tcp-flags-explained)
    - [IP Range Details](#ip-range-details)
  - [Monitor HTTP (Port 80) Traffic and Content](#monitor-http-port-80-traffic-and-content)

## Monitor SYN packets from a specific host to internal networks

```bash
tcpdump -i any -n -v -l 'tcp[tcpflags] & (tcp-syn) != 0 and tcp[tcpflags] & (tcp-ack) = 0 and src host 10.199.39.45 and dst net (10.0.0.0/8 or 172.16.0.0/12 or 192.168.0.0/16)' | awk '{ print strftime("%Y-%m-%d %H:%M:%S"), $0; fflush(); }'
```

Additional improvements:

- `-l`: Line-buffered output (better for piping)
- `awk` formatting to add timestamps
- `fflush()`: Ensures immediate output

### TCP Flags Explained

The filter `tcp[tcpflags] & (tcp-syn) != 0 and tcp[tcpflags] & (tcp-ack) = 0` breaks down as:

- Matches packets with SYN flag set but ACK flag not set
- This specifically catches the first packet of TCP's three-way handshake
- Useful for monitoring new connection attempts before they're established

### IP Range Details

The destination filter `dst net (10.0.0.0/8 or 172.16.0.0/12 or 192.168.0.0/16)` covers:

- `10.0.0.0/8`: Class A private network (10.0.0.0 to 10.255.255.255)
- `172.16.0.0/12`: Class B private network (172.16.0.0 to 172.31.255.255)
- `192.168.0.0/16`: Class C private network (192.168.0.0 to 192.168.255.255)

These ranges are reserved for private networks as per RFC 1918 and won't route over the internet.

## Monitor HTTP (Port 80) Traffic and Content

```bash
tcpdump -i any -n -A -s0 'tcp port 80 and (src net (10.0.0.0/8 or 172.16.0.0/12 or 192.168.0.0/16) and dst net (10.0.0.0/8 or 172.16.0.0/12 or 192.168.0.0/16)) and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
```

This command breaks down as follows:

- `-i any`: Listen on any network interface
- `-n`: Don't convert addresses to hostnames (faster)
- `-A`: Print packet payload in ASCII
- `-s0`: Capture entire packet (no length limit)

The complex filter means:

1. `tcp port 80`: Match HTTP traffic
2. `src net (10.0.0.0/8 or 172.16.0.0/12 or 192.168.0.0/16)`: Filter source to internal networks
3. `dst net (10.0.0.0/8 or 172.16.0.0/12 or 192.168.0.0/16)`: Filter destination to internal networks
4. The payload calculation:
   - `ip[2:2]`: Total IP packet length
   - `((ip[0]&0xf)<<2)`: IP header length
   - `((tcp[12]&0xf0)>>2)`: TCP header length
   - When subtracting headers from total length, non-zero means there's data