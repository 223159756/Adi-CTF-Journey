# Vault Door 7

| Field      | Detail              |
|------------|---------------------|
| Platform   | PicoCTF             |
| Category   | Reverse Engineering |
| Difficulty | Medium              |
| Date       | 2026-06-04          |
| Status     | Solved              |

---

## Description

This vault uses bit operations make it harder to understand, can you decrypt the password?

![](../Screenshots/Pasted%20image%2020260604140645.png)

---

## Initial Observations

The Java source stores the password as an array of 8 integers. Each integer packs 4 ASCII characters into a 32-bit value by shifting each character into place with bit operations. The `checkPassword` method compares the packed form of the input directly against the 8 hardcoded integers.

![](../Screenshots/Pasted%20image%2020260604140701.png)

The 8 integers are:

```
x[0] == 1096770097
x[1] == 1952395366
x[2] == 1600270708
x[3] == 1601398833
x[4] == 1716808014
x[5] == 1734293606
x[6] == 909455713
x[7] == 1664103218
```

Each integer encodes 4 bytes in big-endian order. Converting them back to bytes gives the 32-character password.

---

## Approach

**Step 1 - Understand the packing**

The Java code builds each integer by shifting characters left by 24, 16, 8, and 0 bits then ORing them together:

```java
return ((password.charAt(0) << 24)
      | (password.charAt(1) << 16)
      | (password.charAt(2) << 8)
      | (password.charAt(3) << 0)) == x[0];
```

This is just big-endian packing. Each integer's 4 bytes are the ASCII values of 4 consecutive characters.

**Step 2 - Write the solver**

`int.to_bytes(4, 'big')` unpacks each integer back into its 4 bytes. Decode as ASCII and join.

```python
x = [1096770097, 1952395366, 1600270708, 1601398833, 1716808014, 1734293606, 909455713, 1664103218]

flag = ''.join(n.to_bytes(4, 'big').decode('ascii') for n in x)
print('picoCTF{' + flag + '}')
```

**Step 3 - Run it**

![](../Screenshots/Pasted%20image%2020260604140728.png)

```
python3 vaultunlock.py
picoCTF{A_rithmetic_1s_4_b1t_harder_here}
```

---

## Tools Used

- **Python** - unpacked each 32-bit integer into 4 ASCII bytes using `to_bytes`

---

## Key Finding

The integers are not hashes or encrypted values. They are the password characters packed into 32-bit integers with bit shifts. Python's `int.to_bytes(4, 'big')` directly reverses the packing -- no loop, no XOR, no decoding step needed.

---

## Solution

```python
x = [1096770097, 1952395366, 1600270708, 1601398833, 1716808014, 1734293606, 909455713, 1664103218]

flag = ''.join(n.to_bytes(4, 'big').decode('ascii') for n in x)
print('picoCTF{' + flag + '}')
```

**Flag:** `picoCTF{A_rithmetic_1s_4_b1t_harder_here}`

---

## Lessons Learned

**Bit shifting is just packing**
`char << 24 | char << 16 | char << 8 | char` is big-endian byte packing. Recognising that pattern means the reversal is one line: `int.to_bytes(4, 'big')`.

**Try the obvious representation first**
When integers appear in a password check, converting them to bytes and reading as ASCII is the first thing to try. It works here without any further analysis.
