# Vault Door 3

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

This vault uses for-loops and byte arrays. The source code for this vault is here: VaultDoor3.java

![](../Screenshots/Pasted%20image%2020260604131118.png)

---

## Initial Observations

Given a Java source file. Same structure as vault-door-1: strip the `picoCTF{...}` wrapper, pass the 32-character inner string to `checkPassword()`. This time there is no direct character comparison. Instead, the method uses four for-loops to copy characters from the input into a `buffer` array in scrambled positions, then checks if the buffer equals the hardcoded string `"jU5t_a_sna_3lpm16g041_u_4_m2r547"`.  
![](../Screenshots/Pasted%20image%2020260604131138.png)


The scrambling happens to the input before comparison, so the hardcoded string is not the flag. The flag is whatever input, when scrambled by those loops, produces that string.

---

## Approach

**Step 1 - Understand what each loop does**

Converted the Java to Python to reason through it:

```python
to_crack = "jU5t_a_sna_3lpm16g041_u_4_m2r547"
```

Loop 1 (`i = 0..7`): copies input positions 0-7 straight into buffer positions 0-7. No change.

Loop 2 (`i = 8..15`): copies `input[23-i]` into `buffer[i]`. This reverses the block between indices 8 and 15.

Loop 3 (`i = 16..30, step 2`): copies `input[46-i]` into even buffer slots 16-30. Another reversal, even positions only.

Loop 4 (`i = 31..17, step -2`): copies `input[i]` straight into `buffer[i]`. Odd positions 17-31, no change.

**Step 2 - Recognise the transformation is self-inverse**

Each loop either copies straight (no change) or reverses a mapping using `23-i` or `46-i`. Applying the same formula twice returns to the original index. So running the same transformation on the target string gives back the original input.

**Step 3 - Write the solver**

Applied the exact same loop logic to `to_crack` instead of an unknown input:
![](../Screenshots/Pasted%20image%2020260604131148.png)

```python
to_crack = "jU5t_a_sna_3lpm16g041_u_4_m2r547"

flag = [''] * 32

for i in range(0, 8):
    flag[i] = to_crack[i]

for i in range(8, 16):
    flag[i] = to_crack[23 - i]

for i in range(16, 32, 2):
    flag[i] = to_crack[46 - i]

for i in range(31, 16, -2):
    flag[i] = to_crack[i]

print(''.join(flag))
```

**Step 4 - Run it**

Output is the flag inner string. Wrapped with `picoCTF{...}` and submitted.

---

## Tools Used

- **Python** - translated the Java loop logic and ran it to recover the original input

---

## Key Finding

The scrambling transformation is its own inverse. Each loop either does a straight copy or a symmetric reversal. Feeding the target string back through the same loops undoes the scramble and gives the original password directly.

---

## Solution

```python
to_crack = "jU5t_a_sna_3lpm16g041_u_4_m2r547"
flag = [''] * 32

for i in range(0, 8):
    flag[i] = to_crack[i]
for i in range(8, 16):
    flag[i] = to_crack[23 - i]
for i in range(16, 32, 2):
    flag[i] = to_crack[46 - i]
for i in range(31, 16, -2):
    flag[i] = to_crack[i]

print('picoCTF{' + ''.join(flag) + '}')
```

**Flag:** `picoCTF{jU5t_a_s1mpl3_an4gr4m_4_u_120567}`

---

## Lessons Learned

**Read what the code does to your input before trying to reverse it**
The loops looked complex but each one was doing something simple: either a straight copy or a symmetric index flip. Breaking it down loop by loop in Python made the pattern obvious.

**A self-inverse transformation needs no separate decryption step**
If scramble(scramble(x)) == x, then scramble(target) == original. Recognising this saves writing a separate unscramble function entirely.
