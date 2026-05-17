# [## Wireshark doo dooo do doo...]

| Field      | Detail            |
| ---------- | ----------------- |
| Platform   | PicoCTF           |
| Category   | Forensics         |
| Difficulty | Medium            |
| Points     | 000               |
| Date       | 2026-05-17        |
| Status     | Solved / Unsolved |

---

## Description

![[Screenshot 2026-05-17 183356.png]]
---

## Initial Observations

Wireshark in name so go figure.
Pcap Provided, without any information so certainly flag hidden inside. 


---

## Approach

Walk through my thinking step by step. Include dead ends 

**Step 1 -** Open Wireshark, load pcap, look at ipv4 statistics. 4 IP's total:
![[Pasted image 20260517192436.png]]
First 3 Ip's look standard (probably NAT with 192.168.38.0/24) but the last two look weird with very low packet transfer. 

**Step 2 -...** Look at the packet capture overview, noticed the 18.222.37.134 ip having a weird HTTP conversation with the main IP 192.168.38.104, filtered for the communication: ![[Pasted image 20260517192712.png|697]]
Expand TCP Packet stream using follow and noticed HTTP data transfer after TCP handshake: 
![[Pasted image 20260517192825.png]]
Last line looks like the format for a CTF flag. 
**Step 3 - ...** 

---

## Tools Used

- tool name, why I used it

---

## Key Finding

Key insights on cracking it

---

## Solution

```
commands or code that produced the flag
```

**Flag:** `picoCTF{...}`

---

## Lessons Learned

What to remember for next time
