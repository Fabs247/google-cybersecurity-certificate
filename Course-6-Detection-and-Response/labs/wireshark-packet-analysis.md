# Analyzing Network Traffic with Wireshark

## Project Description

In this lab, I acted as a security analyst investigating network traffic to a website. Using Wireshark  a network protocol analyzer (packet sniffer) with a graphical interface I opened a sample packet capture (`.pcap`) file, explored its structure layer by layer, and applied display filters to isolate specific traffic. The goal was to reconstruct a single web-browsing session from raw packet data: identify the source and destination IP addresses, examine the protocols used to make the connection, and inspect individual packets to understand what information was exchanged.

Packet analysis is a foundational SOC skill. When a security team needs to understand what actually happened on a network a suspicious download, a beaconing host, a data exfiltration attempt the packet capture is the ground truth. Everything else is interpretation.

---

## Tool

```
Wireshark — network protocol analyzer (packet sniffer)
Environment — Windows VM
Capture file — sample.pcap
```

---

## The Scenario in One Picture

Every packet in the capture is part of a single story: a user visiting `opensource.google.com`. The session breaks down into ordered steps, and each filter I applied isolates one of them.

```
1. User requests opensource.google.com
        ↓
2. DNS query    → "what IP maps to this name?"   UDP port 53
        ↓
3. DNS answer   → "142.250.1.139"                UDP port 53
        ↓
4. TCP connection opens to 142.250.1.139         TCP port 80
        ↓
5. HTTP page data flows back and forth           HTTP over TCP

   (ICMP echo requests also present — connectivity checks)
```

---

## Understanding the Wireshark Interface

Wireshark presents packets in three panes:

| Pane | What it shows |
|------|---------------|
| **Packet List** | Every captured packet, one per row No., Time, Source, Destination, Protocol, Length, Info |
| **Packet Details** | The layered breakdown of a single selected packet (the protocol subtrees) |
| **Packet Bytes** | The raw data in hexadecimal and ASCII |

**Coloring rules** provide quick visual classification for example, light blue for DNS packets, green for TCP/HTTP, light pink for ICMP. In a large capture, colour lets you spot relevant traffic at a glance.

### A packet is a set of nested layers

The Packet Details pane reveals that every packet is wrapped in layers, each added by a different part of the network stack:

```
Frame          → the entire packet (length, arrival time)
 └ Ethernet II → source/destination MAC addresses (hardware)
    └ IP v4    → source/destination IP, TTL, inner protocol
       └ TCP   → source/destination ports, sequence numbers, flags
          └ payload → the actual application data
```

Expanding each subtree peels back one layer. This nesting an Ethernet frame carrying an IP packet carrying a TCP segment carrying HTTP data  is the core mental model of packet analysis.

---

## Task 1 — Identifying ICMP Traffic

I scrolled the packet list to the first packet whose Info column began with `Echo (ping) request`.

**Q: What protocol is this packet?**
```
✅ ICMP
```

An echo request is the defining feature of ICMP it is literally what the `ping` command sends. ICMP (Internet Control Message Protocol) handles network diagnostics and error messaging, not application data. Seeing echo requests in a capture usually indicates a connectivity check.

---

## Task 2 — Filtering by IP Address and Inspecting a Packet

I applied a display filter to isolate all traffic to or from a single host:

```
ip.addr == 142.250.1.139
```

This reduced the list to just ICMP (light pink) and TCP/HTTP (light green) packets involving that address. I opened the first TCP packet and walked its subtrees Frame, Ethernet II, IP v4, TCP  observing how the source and destination values at each layer matched the summary row.

**Q: What is the TCP destination port?**
```
✅ 80
```

Port 80 is the default port for HTTP web traffic. A destination of port 80 confirms the user is connecting to a web server to load a page.

**Filter logic learned:**
```
ip.addr == x   → traffic where x is EITHER source or destination
ip.src == x    → traffic FROM x only
ip.dst == x    → traffic TO x only
```

---

## Task 3 — Filtering by MAC Address

I filtered on a hardware (Ethernet) address rather than an IP:

```
eth.addr == 42:01:ac:15:e0:02
```

Filtering by MAC captures all traffic involving that physical interface regardless of the higher-level protocols. Inside the first packet's IPv4 subtree, the **Protocol** field identifies what the IP packet is carrying.

**Q: What protocol is contained in the IPv4 subtree of the first packet?**
```
✅ TCP
```

**MAC vs IP — why both exist:**
```
MAC address → hardware identity, fixed to the network interface,
              used for delivery within the local network segment
IP address  → logical identity, can change, used for routing
              across networks
```

---

## Task 4 — Exploring DNS Packets

DNS traffic runs over UDP port 53, so I filtered for it:

```
udp.port == 53
```

I expanded the **Domain Name System (query)** subtree of the first packet and found the queried name under **Queries**: `opensource.google.com`. In a later packet, the **Answers** section held the resolved address.

**Q: Which IP address appears in the Answers section for opensource.google.com?**
```
✅ 142.250.1.139
```

This is the entire purpose of DNS — resolving a human-readable name into a machine-usable IP address. Critically, this is the **same IP** I filtered on in Task 2. That is the moment the story connects: the DNS answer here is what told the browser where to send the TCP connection there.

**Why DNS uses UDP:**
```
A DNS lookup is one small question and one small answer.
UDP is fast and connectionless — no setup overhead.
If a query is lost, it's cheap to just ask again.
```

---

## Task 5 — Exploring TCP Packets and Payload Text

I filtered for web traffic on TCP port 80:

```
tcp.port == 80
```

Opening the first packet (destination `169.254.169.254`) and reading its Frame and IPv4 subtrees gave a full set of packet properties:

| Field | Value | Where it lives |
|-------|-------|----------------|
| Time to Live (TTL) | **64** | IPv4 subtree |
| Frame Length | **54 bytes** | Frame subtree |
| Header Length | **20 bytes** | IPv4 subtree |
| Destination Address | **169.254.169.254** | IPv4 subtree |

I then searched packet payloads for specific text:

```
tcp contains "curl"
```

This isolates packets whose payload contains the string `curl`, revealing web requests made using the `curl` command. Filtering on payload text is how an analyst locates packets tied to a specific tool, username, filename, or indicator of interest.

**Reading the values:**
```
TTL 64        → standard starting TTL for Linux/Unix systems
                (Windows starts at 128) — a quick OS fingerprint
Header 20 B   → the standard minimum size of an IPv4 header
```

---

## Key Concepts Reinforced

### TTL — Time to Live
```
A hop counter. Every router the packet crosses decrements it by 1.
When it reaches 0, the packet is discarded.
Purpose: prevents packets from looping through the network forever.
Bonus:   the starting value hints at the source OS
         (64 → Linux/Unix, 128 → Windows)
```

### UDP vs TCP — seen side by side in one capture
```
DNS  → UDP port 53 → connectionless, fast, fire-and-forget
                     ideal for tiny query/answer exchanges

Web  → TCP port 80 → connection-oriented, reliable, ordered
                     ideal for delivering a full web page intact
```

This lab shows both protocols in the same session  DNS resolves the name over UDP, then the page loads over TCP. The contrast between "fast but unreliable" and "slower but guaranteed" is a fundamental networking distinction.

### The filter toolkit
```
ip.addr == x            both directions
ip.src == x             source only
ip.dst == x             destination only
eth.addr == x           by MAC (hardware) address
udp.port == 53          DNS traffic
tcp.port == 80          web traffic
tcp contains "text"     packets with specific payload text
```

---

## Why This Matters for Security

Reconstructing this benign browsing session is practice for the real job: **the same technique reveals malicious activity.** A SOC analyst uses these exact filters to answer questions like:

```
→ What domain did the malware resolve?      (udp.port == 53, read Queries)
→ What IP is a host beaconing to?           (ip.addr == suspect)
→ Is data leaving over an odd port?         (tcp.port == unusual)
→ Does any packet contain this indicator?   (tcp contains "string")
```

Learning to read a normal session is what lets you recognise an abnormal one. You can't spot the anomaly until you know what ordinary looks like.

---

## Skills Demonstrated

```
✅ Opening and navigating packet capture (.pcap) files
✅ Understanding the three-pane Wireshark interface
✅ Reading the layered structure of a packet (Frame → Ethernet → IP → TCP)
✅ Applying display filters by IP, MAC, port, and payload text
✅ Distinguishing ICMP, DNS (UDP), and HTTP (TCP) traffic
✅ Interpreting DNS query and answer records
✅ Reading packet fields — TTL, header length, frame length, ports
✅ OS fingerprinting from TTL values
✅ Understanding UDP vs TCP trade-offs
✅ Connecting packet analysis to real SOC investigation workflows
```

---

## References

- Wireshark — [Official Documentation](https://www.wireshark.org/docs/)
- Wireshark — [Display Filter Reference](https://www.wireshark.org/docs/dfref/)
- IANA — [Service Name and Port Number Registry](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml)
- RFC 792 — Internet Control Message Protocol (ICMP)
- RFC 1035 — Domain Name System (DNS)
