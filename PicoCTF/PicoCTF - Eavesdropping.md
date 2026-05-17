# Eavesdropping

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

![challenge description](../Screenshots/Pasted%20image%2020260517214731.png)

---

## Initial Observations

Pcap provided, no context given. Name suggests someone's communication is being intercepted look for cleartext data in TCP streams.

---

## Approach

**Step 1 —** Opened pcap in Wireshark, followed TCP streams to look for readable content. Found a plaintext conversation between two parties:

![TCP stream conversation](../Screenshots/Pasted%20image%2020260517214833.png)

![TCP stream continued](../Screenshots/Pasted%20image%2020260517214817.png)

The conversation handed over a full decryption command and confirmed a file transfer was coming over port 9002:

```
openssl des3 -d -salt -in file.des3 -out file.txt -k supersecretpassword123
```

**Step 2 -** Filtered for `tcp.port == 9002` to find the file transfer. Followed that TCP stream and confirmed binary data was being sent:

![port 9002 stream](../Screenshots/Pasted%20image%2020260517215252.png)

![raw bytes extracted](../Screenshots/Pasted%20image%2020260517215309.png)

Changed stream view to **Raw** and saved as `file.des3`.

**Step 3 -** Ran the decryption command from the conversation:

```bash
openssl des3 -d -salt -in file.des3 -out file.txt -k supersecretpassword123
```

Initial attempt failed with `bad magic number` - had accidentally saved the hex string as a text file instead of the actual binary. Converted correctly using:

```bash
echo "53616c7465645f5f..." | xxd -r -p > file.des3
```

![openssl decryption](../Screenshots/Pasted%20image%2020260517225959.png)

![file.txt output](../Screenshots/Pasted%20image%2020260517230414.png)

![flag](../Screenshots/Pasted%20image%2020260517230459.png)

**Flag: `picoCTF{nc_73115_411_5786acc3}`**

---

## Tools Used

- **Wireshark** - pcap analysis and TCP stream inspection
- **xxd** - hex to binary conversion when raw save failed
- **openssl** - decryption using the command found in cleartext stream

---

## Key Finding

- The decryption command, password, and transfer port were all transmitted in plaintext - following the TCP stream gave everything needed without any guessing
- `53616c7465645f5f` at the start of the hex data decodes to `Salted__`, confirming it was the correct OpenSSL-encrypted file before even attempting decryption

---

## Lessons Learned

**1. Follow every TCP stream**
The flag came from reading a conversation Cleartext streams are the first thing to check in any network forensics challenge.

**2. Raw vs hex when extracting files from Wireshark**
Saving a stream as hex and feeding it to a tool expecting binary fails. Set the stream view to Raw before saving, or convert with `xxd` or `openssl`.
