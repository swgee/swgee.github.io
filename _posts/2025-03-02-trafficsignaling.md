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

Another way to send signals is by filtering for crafted packets with unusual characteristics or "magic bytes" embedded in the data. The payload may carry different signals, allowing additional control.

___pick excerpt from filter___
___create scapy program to send to filter___

### Defensive Measures



## References

- [MITRE ATT&CK - Traffic Signaling](https://attack.mitre.org/techniques/T1205/)
