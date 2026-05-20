# Operation Orchid

| Field      | Detail     |
| ---------- | ---------- |
| Platform   | PicoCTF    |
| Category   | Forensics  |
| Difficulty | Medium     |
| Points     | 40         |
| Date       | 2026-05-21 |
| Status     | Solved     |

---

## Description

![challenge description](../Screenshots/Pasted%20image%2020260521033127.png)

---

## Initial Observations

Given a compressed disk image (`disk.flag.img`). The goal is to find a flag somewhere on the disk. Running `file` on the image revealed a DOS/MBR partitioned disk with three partitions - two Linux ext (0x83) and one Linux swap (0x82). The flag is somewhere in one of the filesystems.

---

## Approach

**Step 1 - Identify the partition layout**

Ran `file` on the disk image to read the MBR partition table.

![file disk.flag.img output](../Screenshots/Pasted%20image%2020260521032838.png)

Three partitions:
- Partition 1: `0x83` Linux ext, active, start sector 2048, 204800 sectors 
- Partition 2: `0x82` Linux swap, start sector 206848, 204800 sectors
- Partition 3: `0x83` Linux ext, start sector 411648, 407552 sectors 

The swap partition cannot be mounted as a filesystem. Focus is on partitions 1 and 3.

---

**Step 2 - Mount the partitions**

Used the disk image mounter to attach the image. Two volumes appeared: a 105MB volume and a 209MB volume. The swap partition was skipped automatically.

![two mounted volumes](../Screenshots/Pasted%20image%2020260521032851.png)

---

**Step 3 - Explore the 105MB volume**

Listed the contents of the smaller volume. All bootloader files:  kernel, initramfs, extlinux config. This is the boot partition. Nothing useful here.

![ls -la 105MB volume](../Screenshots/Pasted%20image%2020260521032910.png)

---

**Step 4 - Explore the 209MB volume**

Listed the root of the larger volume. Standard Linux root filesystem structure: `bin`, `etc`, `home`, `root`, `usr`, `var` etc. This is the OS partition.

![ls -la 209MB volume root](../Screenshots/Pasted%20image%2020260521032926.png)

---

**Step 5 - Check root's home directory**

Tried `ls -la root/` — permission denied. Used `sudo !!` to repeat with elevated privileges. Found two files: `.ash_history` (shell history) and `flag.txt.enc` (encrypted file).

![permission denied then sudo, finding .ash_history and flag.txt.enc](../Screenshots/Pasted%20image%2020260521033011.png)

---

**Step 6 - Inspect the encrypted file**

`cat flag.txt.enc` returned binary output starting with `Salted__`: the OpenSSL magic header. The file was encrypted with `openssl enc`. Cracking it directly is not possible (very very time consuming) without the key.

![cat flag.txt.enc showing Salted__ header](../Screenshots/Pasted%20image%2020260521033030.png)

---

**Step 7 - Read the shell history**

`cat .ash_history` showed every command run as root on this machine. The encryption command was logged in full:

```
openssl aes256 -salt -in flag.txt -out flag.txt.enc -k unbreakablepassword1234567
shred -u flag.txt
```

The original `flag.txt` was shredded after encryption, but the password was left in plaintext in the shell history.

![cat .ash_history showing openssl command with password](../Screenshots/Pasted%20image%2020260521033047.png)

---

**Step 8 - Decrypt the flag**

Used the recovered password to decrypt:

```bash
openssl aes256 -d -in flag.txt.enc -k unbreakablepassword1234567
```


![openssl decrypt output showing flag](../Screenshots/Pasted%20image%2020260521033102.png)

**Flag: `picoCTF{h4un71ng_p457_5113beab}`**

---

## Tools Used

- **file** - read the MBR partition table to identify partition types and start sectors
- **Disk Image Mounter** - mount individual partitions from the raw image without manual offset calculation
- **openssl** - decrypt the AES-256 encrypted flag file using the recovered password

---

## Key Finding

The password was stored in plaintext in `.ash_history`. The operator encrypted the flag and shredded the original, but shell history logging was active the entire time. The encryption was sound; the operational security was not.

---

## Lessons Learned

**Shell history is very useful in disk forensics**
`.bash_history`, `.ash_history`, `.zsh_history` log commands and should be a first line of investigation especially in forensic cases. 
