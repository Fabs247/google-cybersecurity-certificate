# Capturing Network Traffic with tcpdump

## Project Description

In this lab, I acted as a network analyst using **tcpdump** - a command-line packet sniffer to capture and analyze live network traffic on a Linux virtual machine. Working entirely from the terminal, I identified available network interfaces, captured live traffic, saved packets to a `.pcap` capture file, generated my own HTTP traffic to capture, and then filtered the saved data to inspect it at both the header and raw byte level.

This lab is the command-line counterpart to graphical analysis in Wireshark. Where Wireshark excels at deep visual inspection, tcpdump is the tool a security analyst reaches for on a headless server, over an SSH session, or inside an automated script  situations where no graphical interface is available. Knowing both, and knowing *when* to use each, is a core SOC skill.

---

## Environment

```
System:      Linux virtual machine
User:        analyst (pre-logged-in terminal)
Tool:        tcpdump (command-line packet sniffer)
Interface:   eth0
```

---

## Task 1 — Identifying Network Interfaces

Before capturing traffic, I had to know which interface to listen on. I identified the available network interfaces two ways.

```bash
sudo ifconfig
```

This listed the interfaces on the system:

```
eth0 → inet 172.17.0.2   (the Ethernet interface — capture here)
lo   → inet 127.0.0.1    (loopback — internal traffic only)
```

The Ethernet interface is identified by the `eth` prefix, so **eth0** is the interface I captured from throughout the lab.

I then confirmed the same thing using tcpdump directly:

```bash
sudo tcpdump -D
```

The `-D` flag lists all interfaces available for capture. This matters because some systems don't include `ifconfig`, and an analyst needs a way to enumerate interfaces using tcpdump alone.

**Concept — interfaces:**
```
eth0 → the real network connection (traffic to/from the network)
lo   → loopback, the machine talking to itself (127.0.0.1)
```

---

## Task 2 — Inspecting Live Traffic

Next I captured live traffic directly to the screen:

```bash
sudo tcpdump -i eth0 -v -c5
```

```
-i eth0   → capture from the eth0 interface
-v        → verbose: show detailed packet fields
-c5       → capture 5 packets, then stop
```

tcpdump began listening and printed packet data in real time. A captured packet looked like this:

```
10:57:33.427749 IP (tos 0x0, ttl 64, id 35057, offset 0, flags [DF], proto TCP (6), length 134)
  ...5000 > ...59788: Flags [P.], cksum 0x5851, seq 1080713945:1080714027,
  ack 62760789, win 501, length 82
```

### Reading a tcpdump line

Breaking that output down field by field — this is the skill that transfers directly to real investigations:

```
10:57:33.427749   → timestamp (HH:MM:SS.microseconds)
IP                → this is an IP packet
tos 0x0           → Type of Service (handling priority)
ttl 64            → Time to Live = 64 → Linux/Unix source
id 35057          → packet identifier
flags [DF]        → IP flag: Don't Fragment
proto TCP (6)     → carrying TCP (protocol number 6)
length 134        → total packet length in bytes

...5000  >  ...59788    → source > destination
                          the ">" means "to"
                          the number after the dot is the PORT

Flags [P.]        → TCP flags: P = PSH (push data), . = ACK
cksum 0x5851      → checksum (error detection)
seq / ack         → sequence & acknowledgment numbers
win 501           → window size (flow control)
length 82         → TCP payload = 82 bytes of actual data
```

**Key reading skills locked in:**
```
>          means "to"  (source on left, destination on right)
.NNNNN     the number after the last dot is the port
ttl 64     Linux/Unix   (128 would indicate Windows)
proto (6)  TCP's protocol number is 6
[P.]       PSH + ACK — normal for an established connection
           pushing data through
```

---

## Task 3 — Capturing Traffic to a File

Live-to-screen is useful for a quick look, but real analysis needs the traffic *saved*. I captured only web traffic (TCP port 80) into a `.pcap` file:

```bash
sudo tcpdump -i eth0 -nn -c9 port 80 -w capture.pcap &
```

```
-i eth0        → capture from eth0
-nn            → do NOT resolve IPs or ports to names
-c9            → capture 9 packets, then exit
port 80        → filter: only HTTP (TCP port 80) traffic
-w capture.pcap → write captured packets to this file
&              → run in the background
```

Then I generated my own HTTP traffic for tcpdump to capture:

```bash
curl opensource.google.com
```

Using `curl` to fetch a website produces HTTP (TCP port 80) traffic  exactly what the filter was waiting for. I confirmed the capture was saved:

```bash
ls -l capture.pcap
```

### Why `-nn` matters  a real security consideration

The `-nn` flag disables name resolution, and the reason is genuinely important:

```
Without -nn, tcpdump performs DNS lookups to turn IPs into names.
Those lookups:
  → generate extra network traffic
  → may return invalid or attacker-controlled data
  → can ALERT a threat actor that they are being investigated

With -nn, the analyst stays silent and sees the raw IPs and ports.
```

During an active investigation, you do not want your monitoring tool announcing itself to the adversary. This is a small flag with a real operational-security purpose.

---

## Task 4 — Filtering the Captured Data

With traffic saved, I read it back and filtered it. First, the header view:

```bash
sudo tcpdump -nn -r capture.pcap -v
```

```
-nn   → no name resolution (again stay quiet)
-r    → read from the capture file
-v    → verbose detail
```

The output showed the start of a TCP connection the **three-way handshake** in action:

```
172.17.0.2:46498 > 146.75.38.132:80:  Flags [S]    ← SYN     (client: "let's connect")
146.75.38.132:80 > 172.17.0.2:46498:  Flags [S.]   ← SYN-ACK (server: "ok, let's connect")
```

Reading the flags:
```
[S]   → SYN      first step of the TCP handshake
[S.]  → SYN-ACK  second step (S = SYN, . = ACK)
        the third step [.] = ACK completes it
```

Seeing the handshake in a real capture is a great reinforcement of how TCP establishes a reliable connection before any data flows.

Then I viewed the raw packet contents in hex and ASCII:

```bash
sudo tcpdump -nn -r capture.pcap -X
```

```
-X → display packet data as hexadecimal AND ASCII
```

**Why hex/ASCII matters:**
```
Analysts inspect raw hex + ASCII output to spot patterns or
anomalies during malware analysis and forensic investigation —
readable strings, suspicious payloads, or protocol data that
doesn't match what the port claims to be.
```

---

## The Complete Workflow

What this lab demonstrates end to end:

```
1. IDENTIFY   → find the right interface (ifconfig / tcpdump -D)
        ↓
2. INSPECT    → watch live traffic (tcpdump -i eth0 -v -c5)
        ↓
3. CAPTURE    → save filtered traffic to .pcap (-w capture.pcap)
        ↓
4. GENERATE   → create traffic to capture (curl)
        ↓
5. FILTER     → read & analyze the saved file (-r, -v, -X)
```

---

## tcpdump Flag Reference

A quick-reference of every flag used in this lab:

| Flag | Meaning |
|------|---------|
| `-i eth0` | Capture from the eth0 interface |
| `-D` | List available capture interfaces |
| `-v` | Verbose - show detailed packet fields |
| `-c N` | Capture N packets then stop |
| `-nn` | Disable IP/port name resolution (opsec) |
| `-w file` | Write captured packets to a file |
| `-r file` | Read packets from a saved file |
| `-X` | Show packet data in hex and ASCII |
| `port 80` | Filter: only port 80 (HTTP) traffic |
| `&` | Run the command in the background |

---

## tcpdump and Wireshark — The Combined Workflow

This lab is the natural partner to graphical analysis in Wireshark. Because both tools share the `.pcap` format, they combine into one workflow:

```
tcpdump captures on the server   →  writes capture.pcap
   (CLI, lightweight, no GUI needed)      ↓
              transfer the .pcap file
                        ↓
Wireshark opens it on a workstation  →  deep visual analysis
   (GUI, layer-by-layer inspection)
```

```
Reach for tcpdump when:          Reach for Wireshark when:
→ on a remote/headless server    → doing detailed investigation
→ working over SSH               → reconstructing a full session
→ scripting or automating        → you want graphs & statistics
→ you need a fast, quiet capture → the visual view speeds you up
```

Knowing both means being effective whether you're on a graphical workstation or dropped into a bare terminal on a compromised server.

---

## Why This Matters for Security

tcpdump is often the *first* tool available during an incident, because it's pre-installed on virtually every Linux/Unix system and needs no GUI. A SOC analyst uses these exact techniques to:

```
→ Capture traffic on a compromised server that has no desktop
→ Save evidence to a .pcap for later forensic analysis
→ Filter to a specific port or host to cut through the noise
→ Read raw payloads in hex/ASCII to spot malware or exfiltration
→ Do all of it quietly (-nn) without tipping off the attacker
```

---

## Skills Demonstrated

```
✅ Identifying network interfaces (ifconfig, tcpdump -D)
✅ Capturing live network traffic from the command line
✅ Reading and interpreting tcpdump output field by field
✅ Understanding IP and TCP header fields (TTL, flags, ports, seq/ack)
✅ Recognising the TCP three-way handshake (SYN / SYN-ACK / ACK)
✅ Filtering traffic by port
✅ Saving captures to .pcap files (-w)
✅ Reading captures back from file (-r)
✅ Generating test traffic with curl
✅ Inspecting raw packet data in hex and ASCII (-X)
✅ Applying operational security (-nn to avoid alerting threat actors)
✅ Understanding the tcpdump + Wireshark combined workflow
```

---

## References

- tcpdump — [Manual Page](https://www.tcpdump.org/manpages/tcpdump.1.html)
- tcpdump — [Resources and Documentation](https://www.tcpdump.org/index.html#documentation)
- The Tcpdump Group — [pcap-filter (BPF) syntax](https://www.tcpdump.org/manpages/pcap-filter.7.html)
- RFC 793 — Transmission Control Protocol (TCP three-way handshake)
- IANA — [Service Name and Port Number Registry](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml)
