# Vault Door 5

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

In the last challenge, you mastered octal (base 8), decimal (base 10), and hexadecimal (base 16) numbers, but this vault door uses a different change of base as well as URL encoding! The source code for this vault is here: VaultDoor5.java

---

## Initial Observations

Given a Java source file with two encoding helper methods and a `checkPassword` function. The check applies two layers of encoding to the input before comparing against a hardcoded string:

1. URL encode the password bytes (each byte becomes `%XX` in hex)
2. Base64 encode the result of step 1

The expected string is a concatenation of three base64 strings. Reversing the two layers gives the flag.

---

## Approach

**Step 1 - Read the encoding chain**

`checkPassword` does:
```
urlEncoded  = urlEncode(password.getBytes())
b64Encoded  = base64Encode(urlEncoded.getBytes())
return b64Encoded.equals(expected)
```

The `urlEncode` method formats each byte as `%%%02x` — a percent sign followed by two hex digits.

So the full chain is: `password → URL encode → base64 encode → expected string`

Reversing it: `expected → base64 decode → URL decode → password`

**Step 2 - Identify the trailing semicolon**

The expected string in the Java source ends with a semicolon from the assignment statement. That semicolon is not part of the encoded value and needs to be stripped before decoding.

**Step 3 - Write the solver**

```python
import base64

expected = ("JTYzJTMwJTZlJTc2JTMzJTcyJTc0JTMxJTZlJTY3JTVm"
            + "JTY2JTcyJTZkJTZkJTVmJTYyJTYxJTM1JTY1JTVmJTM2"
            + "JTM0JTVmJTM0JTMyJTZjNjQwOWI=")

decoded_bytes = base64.b64decode(expected)
decoded_str   = decoded_bytes.decode('utf-8')

# URL decode: strip % signs and convert hex pairs to chars
stripped      = decoded_str.replace('%', '')
ascii_string  = bytes.fromhex(stripped).decode('utf-8')

print('picoCTF{' + ascii_string + '}')
```

**Step 4 - Run it**

```
python3 vaultunlock.py
picoCTF{c0nv3rt1ng_fr0m_ba5e_64_42c6409b}
```

---

## Tools Used

- **Python** - base64 decode and URL decode to reverse the two encoding layers

---

## Key Finding

The password is not encrypted, just encoded twice. Base64 and URL encoding are both fully reversible with no key. The only non-obvious step was recognising the encoding order and reversing it correctly: base64 decode first, then URL decode.

---

## Solution

```python
import base64

expected = ("JTYzJTMwJTZlJTc2JTMzJTcyJTc0JTMxJTZlJTY3JTVm"
            + "JTY2JTcyJTZkJTZkJTVmJTYyJTYxJTM1JTY1JTVmJTM2"
            + "JTM0JTVmJTM0JTMyJTZjNjQwOWI=")

decoded_bytes = base64.b64decode(expected)
decoded_str   = decoded_bytes.decode('utf-8')
stripped      = decoded_str.replace('%', '')
ascii_string  = bytes.fromhex(stripped).decode('utf-8')

print('picoCTF{' + ascii_string + '}')
```

**Flag:** `picoCTF{c0nv3rt1ng_fr0m_ba5e_64_42c6409b}`

---

## Lessons Learned

**Encoding is not encryption**
Base64 and URL encoding have no key. Any encoded value can be reversed without any secret. Seeing these in a password check means the password is readable from the source alone.

**Order of operations matters when reversing**
The encoding was applied as URL encode then base64 encode. Reversing it requires base64 decode first, then URL decode. Doing it the wrong way around produces garbage.
