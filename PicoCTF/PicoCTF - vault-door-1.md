# Vault Door 1

| Field      | Detail              |
|------------|---------------------|
| Platform   | PicoCTF             |
| Category   | Reverse Engineering |
| Difficulty | Medium              |
| Points     | 200                 |
| Date       | 2026-06-03          |
| Status     | Solved              |

---

## Description

This vault uses some complicated arrays! I hope you can make sense of it, special agent. The source code for this vault is here: VaultDoor1.java

---

## Initial Observations

Given a Java source file. The program takes a password wrapped in `picoCTF{...}`, strips the wrapper, and passes the inner 32-character string to `checkPassword()`. That method checks each character individually by index using `charAt()`, but the checks are written in scrambled order rather than 0 through 31.

The comment from Minion #8728 claims this is "UNHACKABLE" because the password is not stored as a plain string. It is. It is just written sideways.

---

## Approach

**Step 1 - Read the checkPassword method**

The method returns true only if all 32 character checks pass. Each line is of the form:

```
password.charAt(INDEX) == 'CHAR'
```

The checks are not in order. Index 0 is checked first, then 29, then 4, then 2, and so on. The characters are all present in the source, just not laid out sequentially.

**Step 2 - Build a Python solver**

Created a dictionary mapping each index to its required character, then joined them in order:

```python
chars = {
    0:'d', 1:'3', 2:'5', 3:'c', 4:'r', 5:'4', 6:'m', 7:'b',
    8:'l', 9:'3', 10:'_', 11:'t', 12:'H', 13:'3', 14:'_', 15:'c',
    16:'H', 17:'4', 18:'r', 19:'4', 20:'c', 21:'T', 22:'3', 23:'r',
    24:'5', 25:'_', 26:'2', 27:'9', 28:'e', 29:'8', 30:'d', 31:'8'
}
print('picoCTF{' + ''.join(chars[i] for i in range(32)) + '}')
```

**Step 3 - Run it**

```
python vaultunlock.py
picoCTF{d35cr4mbl3_tH3_cH4r4cT3r5_29e8d8}
```

![](../Screenshots/Pasted%20image%2020260604131014.png)

---

![](../Screenshots/Pasted%20image%2020260604131017.png)

![](../Screenshots/Pasted%20image%2020260604131055.png)

![](../Screenshots/Pasted%20image%2020260604131058.png)

## Tools Used

- **Python** - reassembled the password by sorting the charAt checks by index

---

## Key Finding

The password is fully present in the source code. The only obfuscation is that the character checks are written out of order. Reading them in index order reconstructs the password directly. Static analysis of the source is sufficient.

---

## Solution

```python
chars = {
    0:'d', 1:'3', 2:'5', 3:'c', 4:'r', 5:'4', 6:'m', 7:'b',
    8:'l', 9:'3', 10:'_', 11:'t', 12:'H', 13:'3', 14:'_', 15:'c',
    16:'H', 17:'4', 18:'r', 19:'4', 20:'c', 21:'T', 22:'3', 23:'r',
    24:'5', 25:'_', 26:'2', 27:'9', 28:'e', 29:'8', 30:'d', 31:'8'
}
print('picoCTF{' + ''.join(chars[i] for i in range(32)) + '}')
```

**Flag:** `picoCTF{d35cr4mbl3_tH3_cH4r4cT3r5_29e8d8}`

---

## Lessons Learned

**Scrambled order is not obfuscation**
Checking characters out of sequence does not hide the password. As long as each index maps to a known character in the source, sorting by index recovers the full string instantly. Real obfuscation would require the password to be derived, not stored.
