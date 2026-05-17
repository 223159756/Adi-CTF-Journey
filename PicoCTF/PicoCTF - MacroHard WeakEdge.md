# MacroHard WeakEdge

| Field      | Detail     |
| ---------- | ---------- |
| Platform   | PicoCTF    |
| Category   | Forensics  |
| Difficulty | Medium     |
| Points     | 000        |
| Date       | 2026-05-18 |
| Status     | Solved     |

---

## Description

![challenge description](../Screenshots/Pasted%20image%2020260518001238.png)

---

## Initial Observations

A .pptm file - PowerPoint with macros. Name literally says "WeakEdge" so there's probably something hidden poorly. First instinct: check the macro, then dig into the file structure since Office files are just ZIPs underneath.

---

## Approach

**Step 1 -** Checked for extension mismatch with `file`. Came back as a legitimate PowerPoint file, no tricks there.
![olevba output](../Screenshots/Pasted%20image%2020260517234309.png)


**Step 2 -** Ran `olevba` to extract the VBA macro:


![extracted structure](../Screenshots/Pasted%20image%2020260517235244.png)

Found a macro called `not_flag()` containing `sorry_but_this_isn't_it`. Classic red herring, moving on.

**Step 3 -** Unzipped the .pptm to inspect the internals:

```bash
unzip Forensics_is_fun.pptm -d analyse
```

![file listing](../Screenshots/Pasted%20image%2020260518000226.png)

**Step 4 -** While listing all files, spotted something immediately: `ppt/slideMasters/hidden` - no extension, sitting alone next to the normal slideMaster files. That's not supposed to be there.

```bash
cat /home/kali/Desktop/analyse/ppt/slideMasters/hidden
```

![hidden file contents](../Screenshots/Pasted%20image%2020260518000704.png)

![base64 string](../Screenshots/Pasted%20image%2020260518000808.png)

Output was a space-separated base64 string:

```
Z m x h Z z o g c G l j b 0 N U R n t E M W R f d V 9 r b j B 3 X 3 B w d H N f c l 9 6 M X A 1 f Q
```

**Step 5 -** Threw it into CyberChef - removed spaces with the "Find/Replace" operation, then base64 decoded:

![CyberChef decode](../Screenshots/Pasted%20image%2020260518001124.png)

![flag](../Screenshots/Pasted%20image%2020260518001201.png)

**Flag: `picoCTF{D1d_u_kn0w_ppts_r_z1p5}`**

---

## Tools Used

- **file** - confirm no extension mismatch
- **olevba** - extract and inspect VBA macros
- **unzip** - crack open the pptm since Office files are ZIP archives
- **CyberChef** - strip spaces and base64 decode the hidden file contents

---

## Key Finding

The flag was in a file literally named `hidden` inside `ppt/slideMasters/`. There wasn't any stegonography; just base64 encoding. Running `find` across the extracted directory is always worth doing.

---

## Lessons Learned

**1. Office files are ZIP archives**
The macro was a deliberate distraction. The flag was hidden in the file structure, not as a VBA (hence the name lol). Unzipping and using `find` took  10 seconds and covered a lot of ground.

**2. Unusual filenames in expected locations stand out immediately**
`slideMasters/hidden` with no extension next to `slideMaster1.xml` is obviously wrong. When enumerating extracted files, a manual check helps weed out common hiding spots.
