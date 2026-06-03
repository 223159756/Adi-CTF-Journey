# Vault Door 4

| Field      | Detail              |
|------------|---------------------|
| Platform   | PicoCTF             |
| Category   | Reverse Engineering |
| Difficulty | Medium              |
| Points     | 200                 |
| Date       | 2026-06-04          |
| Status     | Solved              |

---

## Description

This vault uses ASCII encoding for the password. The source code for this vault is here: VaultDoor4.java

---

## Initial Observations

Given a Java source file. Same wrapper stripping as before. The `checkPassword` method this time converts the input string to a byte array with `getBytes()`, then compares it byte-by-byte against a hardcoded `myBytes` array. The myBytes array mixes three number bases and char literals across its 32 entries:

- Positions 0-7: decimal integers
- Positions 8-15: hexadecimal (`0x` prefix)
- Positions 16-23: octal (`0` prefix in Java)
- Positions 24-31: char literals (`'2'`, `'1'`, etc.)

The goal is to convert every entry to its ASCII character equivalent.

---

## Approach

**Step 1 - Recognise the mixed base encoding**

Java stores all of these as byte values. Decimal, hex, and octal are just different ways of writing the same integers. Char literals like `'2'` are stored as their ASCII value (50). The comparison is purely numeric — `passBytes[i] != myBytes[i]`.

**Step 2 - Write a Python solver**

First attempt used `chr(int(num))` for the char entries, which converts `'2'` to `chr(2)` — a non-printable control character. The output looked correct in the terminal but the last 8 characters were invisible, making the flag wrong.

Second issue: Python 3 does not accept Java-style octal literals like `0142`. Python 3 requires the `0o` prefix. Fixed by rewriting all octal values with `0o`.

Corrected script:

```python
myBytes = [
    106, 85, 53, 116, 95, 52, 95, 98,
    0x55, 0x6e, 0x43, 0x68, 0x5f, 0x30, 0x66, 0x5f,
    0o142, 0o131, 0o164, 0o63, 0o163, 0o137, 0o145, 0o60,
    '2', '1', '3', '8', '7', '2', '1', '3',
]

flag = []
for num in myBytes:
    if isinstance(num, str):
        flag.append(num)
    else:
        flag.append(chr(num))

print('picoCTF{' + ''.join(flag) + '}')
```

**Step 3 - Run it**

```
python3 vaultunlock.py
picoCTF{jU5t_4_bUnCh_0f_bYt3s_e021387213}
```

---

## Tools Used

- **Python** - converted mixed-base byte array to ASCII characters

---

## Key Finding

The password was stored in plain sight as ASCII values written in three different number bases. Converting each value to its character equivalent with `chr()` recovers the flag directly. The only traps were Python's different octal syntax and the char literals needing to be kept as-is rather than re-converted.

---

## Solution

```python
myBytes = [
    106, 85, 53, 116, 95, 52, 95, 98,
    0x55, 0x6e, 0x43, 0x68, 0x5f, 0x30, 0x66, 0x5f,
    0o142, 0o131, 0o164, 0o63, 0o163, 0o137, 0o145, 0o60,
    '2', '1', '3', '8', '7', '2', '1', '3',
]

flag = []
for num in myBytes:
    if isinstance(num, str):
        flag.append(num)
    else:
        flag.append(chr(num))

print('picoCTF{' + ''.join(flag) + '}')
```

**Flag:** `picoCTF{jU5t_4_bUnCh_0f_bYt3s_e021387213}`

---

## Lessons Learned

**Java and Python handle octal differently**
Java uses a leading `0` for octal literals. Python 3 requires `0o`. Copying Java code directly into Python will either silently give wrong results or throw a SyntaxError.

**Non-printable characters are invisible in terminal output**
`chr(2)` prints nothing. If a flag looks truncated in the terminal, suspect a bad character conversion before assuming the logic is wrong.
