# RSA Pop Quiz

| Field      | Detail     |
|------------|------------|
| Platform   | PicoCTF    |
| Category   | Crypto     |
| Difficulty | Medium     |
| Points     | 200        |
| Date       | 2026-06-03 |
| Status     | Solved     |

---

## Description

![[Pasted image 20260603215905.png]]

---

## Initial Observations

A netcat service that runs through 8 RSA problems back to back. Each problem gives a subset of RSA parameters and asks two things: whether computing the target is possible and feasible (Y/N), and if yes, the actual value. Getting any problem wrong boots you out and you restart from scratch. All inputs and outputs are in decimal.

The 8 problems cover the full RSA parameter chain: key generation, encryption, decryption, and the edge cases around small public exponents.

---

## Approach

**Problem 1 - Given p and q, find n**

n is just the product of the two primes. Always feasible.

```python
n = p * q
```

![[Pasted image 20260603215933.png]]

---

**Problem 2 - Given p and n, find q**

n = p * q, so q = n / p. Integer division works exactly since p is a factor of n.

```python
q = n // p
```

![[Pasted image 20260603215955.png]]

---

**Problem 3 - Given e and a large n, find p and q**

Answered N. Factoring a large RSA modulus is computationally infeasible with current methods. Without p and q there is no way to proceed.

![[Pasted image 20260603220020.png]]

---

**Problem 4 - Given p and q, find totient(n)**

Euler's totient for an RSA modulus: phi(n) = (p-1)(q-1).

```python
totient = (p - 1) * (q - 1)
```

![[Pasted image 20260603220031.png]]

---

**Problem 5 - Given plaintext, e, and n, find ciphertext**

Standard RSA encryption. Modular exponentiation.

```python
ciphertext = pow(plaintext, e, n)
```

![[Pasted image 20260603220053.png]]

---

**Problem 6 - Given ciphertext, e=3, and n only, find plaintext**

Answered N. With e=3 and a small message, the cube root attack applies: if m^3 < n, then ciphertext = m^3 with no modular reduction, and you can recover m by taking the integer cube root of ciphertext directly.

To check: `gmpy2.iroot(ciphertext, 3)` returns `(root, exact)`. If `exact` is True, the attack works and the answer is Y. If False, m^3 >= n and modular reduction happened, making decryption infeasible without the private key.

In this case `exact` was False, so the answer was N.

![[Pasted image 20260603220104.png]]

---

**Problem 7 - Given p, q, and e, find d**

d is the modular inverse of e modulo phi(n). Python 3.8+ handles this natively.

```python
phi = (p - 1) * (q - 1)
d = pow(e, -1, phi)
```

![[Pasted image 20260603220117.png]]

---

**Problem 8 - Given p, ciphertext, e, and n, find plaintext**

Feasible. Having p means q = n // p, which gives phi(n), which gives d, which decrypts the ciphertext.

```python
q   = n // p
phi = (p - 1) * (q - 1)
d   = pow(e, -1, phi)
m   = pow(ciphertext, d, n)
```

The output m is a large integer. Converting to hex then ASCII gives the flag:

```python
print(bytes.fromhex(hex(m)[2:]).decode())
```

---

## Tools Used

- **Python** - all RSA arithmetic: `pow()` for modular exponentiation and modular inverse, integer division for key derivation
- **gmpy2** - integer square root check for the cube root feasibility test in Problem 6

---

## Key Finding

Problem 6 is the trap. e=3 looks like it should always allow the cube root attack, but it only works when m^3 < n. The feasibility check is `gmpy2.iroot(c, 3)[1]` returning True. If False, the ciphertext has already been reduced mod n and the attack does not apply.

Problem 8 is the reverse trap: looks hard because there is no d given, but having p alone is enough to reconstruct the entire private key.

---

## Solution

**Flag:** `picoCTF{wA8_th4till3aGal..o5A36695F}`

---

## Lessons Learned

**The cube root attack on RSA has a prerequisite**
e=3 does not automatically mean the cube root attack works. It only applies when the raw plaintext cubed is smaller than n. Always verify with an exact integer root check before answering Y.

**Having one prime factor is as good as having both**
p alone lets you derive q, phi, d, and decrypt anything. In forensics and CTF contexts, a single leaked prime completely breaks the key pair.
