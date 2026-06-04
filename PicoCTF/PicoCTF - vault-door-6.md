# Vault Door 6

| Field      | Detail              |
|------------|---------------------|
| Platform   | PicoCTF             |
| Category   | Reverse Engineering |
| Difficulty | Medium              |
| Date       | 2026-06-04          |
| Status     | Solved              |

---

## Description

This vault uses an XOR encryption scheme. The source code for this vault is here: VaultDoor6.java

![](../Screenshots/Pasted%20image%2020260604131420.png)

Hint: If X ^ Y = Z, then Z ^ Y = X. Write a program that decrypts the flag based on this fact.

---

## Initial Observations

The Java source shows a `checkPassword` method that XORs each byte of the input against `0x55` and compares the result to a hardcoded `myBytes` array. The password must be exactly 32 characters.

![](../Screenshots/Pasted%20image%2020260604131425.png)

```java
for (int i=0; i<32; i++) {
    if (((passBytes[i] ^ 0x55) - myBytes[i]) != 0) {
        return false;
    }
}
```

So: `passBytes[i] ^ 0x55 == myBytes[i]`

Rearranging using the hint's property: `passBytes[i] = myBytes[i] ^ 0x55`

The flag bytes are recovered by XORing each value in `myBytes` against `0x55`.

---

## Approach

**Step 1 - Extract `myBytes` from the source**

```
0x3b, 0x65, 0x21, 0xa,  0x38, 0x0,  0x36, 0x1d,
0xa,  0x3d, 0x61, 0x27, 0x11, 0x66, 0x27, 0xa,
0x21, 0x1d, 0x61, 0x3b, 0xa,  0x2d, 0x65, 0x27,
0xa,  0x33, 0x34, 0x34, 0x30, 0x6d, 0x37, 0x61,
```

**Step 2 - Write the solver**

XOR each byte against `0x55`, convert to a character, then wrap in `picoCTF{...}`.

![](../Screenshots/Pasted%20image%2020260604131434.png)

```python
myBytes = [
    0x3b, 0x65, 0x21, 0xa,  0x38, 0x0,  0x36, 0x1d,
    0xa,  0x3d, 0x61, 0x27, 0x11, 0x66, 0x27, 0xa,
    0x21, 0x1d, 0x61, 0x3b, 0xa,  0x2d, 0x65, 0x27,
    0xa,  0x33, 0x34, 0x34, 0x30, 0x6d, 0x37, 0x61,
]

flag = [''] * 32

for i in range(len(myBytes)):
    flag[i] = myBytes[i] ^ 0x55

flag = list(map(lambda x: chr(int(x)), flag))
flag = ('picoCTF{' + ''.join(flag) + '}')

print(flag)
```

**Step 3 - Run it**

![](../Screenshots/Pasted%20image%2020260604131441.png)

```
python3 vaultunlock.py
picoCTF{n0t_mUcH_h4rD3r_tH4n_x0r_faae8b4}
```

---

## Tools Used

- **Python** - XOR each `myBytes` value against `0x55` to recover the password

---

## Key Finding

XOR is its own inverse. If `passBytes[i] ^ 0x55 == myBytes[i]`, then `myBytes[i] ^ 0x55 == passBytes[i]`. The hardcoded array is not a hash or ciphertext in any strong sense -- a single XOR against a fixed byte is fully reversible with no key material beyond what is already in the source.

---

## Solution

```python
myBytes = [
    0x3b, 0x65, 0x21, 0xa,  0x38, 0x0,  0x36, 0x1d,
    0xa,  0x3d, 0x61, 0x27, 0x11, 0x66, 0x27, 0xa,
    0x21, 0x1d, 0x61, 0x3b, 0xa,  0x2d, 0x65, 0x27,
    0xa,  0x33, 0x34, 0x34, 0x30, 0x6d, 0x37, 0x61,
]

flag = [chr(b ^ 0x55) for b in myBytes]
print('picoCTF{' + ''.join(flag) + '}')
```

**Flag:** `picoCTF{n0t_mUcH_h4rD3r_tH4n_x0r_faae8b4}`

![](../Screenshots/Pasted%20image%2020260604131447.png)

---

## Lessons Learned

**XOR is symmetric**
The same operation that checks the password also decrypts it. Any scheme of the form `input ^ constant == stored` can be reversed by XORing the stored values against the same constant.

**The key is in the source**
The XOR constant `0x55` is hardcoded in the Java file alongside the encrypted bytes. Static analysis alone is enough. 
