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
**Step 3 - ...** I went to a cipher identifier and put the encoded text in ![[Pasted image 20260517193903.png]]
and most likely cipher use was ROT13:
![[Pasted image 20260517193929.png]]
Next, Move to Cyberchef and use ROT13 and we get the flag: 
![[Pasted image 20260517194025.png|697]]

**picoCTF{p33kab00_1_s33_u_deadbeef}**


---

## Tools Used

- **Wireshark:** Easiest and most lightweight tool to analyse pcap files
- **dcode:** First step on triaging cipher type or presence when handling potentially encoded text
- **Cyberchef:** Best tool for decoding/encoding any cipher with tons of options and easy gui

---

## Key Finding

- Knowing what the PICO CTF flag format is helped trace it down faster
- Early Investigation on IP Sources helps narrow down suspicion


---

## Lessons Learned

**1. Recognise ROT13 from the flag prefix**  
If ciphertext starts with `cvpbPGS{`, it's ROT13 `picoCTF{` shifted by 13. Known plaintext breaks Caesar ciphers instantly without any tooling.

**2. CyberChef Magic misses simple ciphers, Use all tools**  
Cyberchef magic missed this letter based obfuscation initially so using dcode helped in giving logic before cyberchef handled it. 