# Attacktive Directory

| Field      | Detail              |
|------------|---------------------|
| Platform   | TryHackMe           |
| Category   | Active Directory / Pentesting |
| Difficulty | Medium              |
| Points     | —                   |
| Date       | 2026-05-18          |
| Status     | In Progress         |

---

## Description

> 99% of Corporate Networks run off of AD. Can you exploit a vulnerable Domain Controller?

Room covers: Kerberos enumeration, AS-REP Roasting, Kerberoasting, Pass the Hash, and full domain compromise.

---

## Target Info

| Field       | Value          |
|-------------|----------------|
| Machine IP  |                |
| Domain      |                |
| DC Hostname |                |

---

## Initial Observations



---

## Approach

**Task 1 — Setup**

**Task 2 — Enumeration (kerbrute)**
![[Pasted image 20260518172342.png]]


**Task 3 — Abusing Kerberos (AS-REP Roasting)**

**Task 4 — Back to Basics (credential cracking)**
![[Pasted image 20260518174500.png]]



**Task 5 — Elevating Privileges (Kerberoasting)**
![[Pasted image 20260518175537.png|697]]
![[Pasted image 20260518175653.png]]
![[Pasted image 20260518180230.png|697]]


![[Pasted image 20260518180147.png]]

**Task 6 — Impacket / secretsdump**
![[Pasted image 20260518181123.png]]

![[Pasted image 20260518181602.png]]


![[Pasted image 20260518181738.png]]

![[Pasted image 20260518181828.png]]




**Task 7 — Flags**

---

## Tools Used

- kerbrute — enumerate valid AD usernames via Kerberos pre-auth
- impacket (GetNPUsers, GetUserSPNs, secretsdump) — Kerberos attacks and credential extraction
- hashcat / john — offline hash cracking
- evil-winrm — WinRM shell

---

## Key Finding



---

## Solution

```
# commands go here as you work through
```

**Flags:**
- `THM{...}` —
- `THM{...}` —

---

## Lessons Learned


