# Wireshark doo dooo do doo...

| Field      | Detail     |
| ---------- | ---------- |
| Platform   | PicoCTF    |
| Category   | Forensics  |
| Difficulty | Medium     |
| Points     | 000        |
| Date       | 2026-05-17 |
| Status     | Solved     |

---

## Description

![challenge description](../Screenshots/Screenshot%202026-05-17%20183356.png)

---

## Initial Observations

Wireshark in name so go figure.
Pcap Provided, without any information so certainly flag hidden inside.

---

## Approach

Walk through my thinking step by step. Include dead ends

**Step 1 -** Open Wireshark, load pcap, look at ipv4 statistics. 4 IP's total:

![IPv4 statistics](../Screenshots/Pasted%20image%2020260517192436.png)

First 3 IPs look standard (probably NAT with 192.168.38.0/24) but the last two look weird with very low packet transfer.

**Step 2 -** Look at the packet capture overview, noticed the 18.222.37.134 IP having a weird HTTP conversation with the main IP 192.168.38.104, filtered for the communication:

![filtered HTTP communication](../Screenshots/Pasted%20image%2020260517192712.png)

Expand TCP Packet stream using follow and noticed HTTP data transfer after TCP handshake:

![TCP stream follow](../Screenshots/Pasted%20image%2020260517192825.png)

Last line looks like the format for a CTF flag.

**Step 3 -** I went to a cipher identifier and put the encoded text in:

![dcode cipher identifier](../Screenshots/Pasted%20image%2020260517193903.png)

Most likely cipher was ROT13:

![ROT13 identified](../Screenshots/Pasted%20image%2020260517193929.png)

Next, move to CyberChef and use ROT13 and we get the flag:

![CyberChef ROT13 decode](../Screenshots/Pasted%20image%2020260517194025.png)

**Flag: `picoCTF{p33kab00_1_s33_u_deadbeef}`**

---

## Tools Used

- **Wireshark** — easiest and most lightweight tool to analyse pcap files
- **dcode** — first step on triaging cipher type when handling potentially encoded text
- **CyberChef** — best tool for decoding/encoding any cipher with tons of options and easy GUI

---

## Key Finding

- Knowing the picoCTF flag format helped trace it down faster
- Early investigation on IP sources helps narrow down suspicion quickly

---

## Lessons Learned

**1. Recognise ROT13 from the flag prefix**
If ciphertext starts with `cvpbPGS{`, it's ROT13 — `picoCTF{` shifted by 13. Known plaintext breaks Caesar ciphers instantly without any tooling.

**2. CyberChef Magic misses simple ciphers — use dcode first**
CyberChef Magic missed this letter-based obfuscation. Use dcode to identify the cipher type first, then CyberChef to execute the decode.
