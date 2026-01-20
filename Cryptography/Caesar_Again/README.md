# Caesar_Again - Cryptography Challenge Writeup

## Challenge Information
- **Category:** Cryptography
- **Points:** 150
- **Difficulty:** Easy

## Challenge Description
This challenge involves a Caesar cipher implementation with obfuscated Python code. The goal is to either brute-force the cipher or find the key hidden in the source code.

## Background: Caesar Cipher
The Caesar cipher is one of the simplest and oldest encryption techniques. It's a substitution cipher where each letter is shifted by a fixed number of positions in the alphabet. For example, with a shift of 3:
- A → D
- B → E
- Hello → Khoor

## Tools Required
- **Python 3** - To run the decryption script
- **Text Editor** - For code analysis

## Methodology

### Step 1: Analyze the Challenge Code
Examine the provided `challenge.py` file:

```python
import string

class O:
    def __init__(self, a):
        self.__key = a
        self.__alpha = string.ascii_lowercase + string.digits

    def __sH(self, c, b):
        if c in self.__alpha:
            return self.__alpha[(self.__alpha.index(c) + b) % len(self.__alpha)]
        else:
            return c

    def Y(self, Z):
        return ''.join(self.__sH(c, self.__key) for c in Z)

    def X(self, W):
        return ''.join(self.__sH(c, -self.__key) for c in W)
```

Key observations:
- Class `O` implements a Caesar cipher
- `Y` method encrypts (shift forward)
- `X` method decrypts (shift backward)
- The alphabet includes lowercase letters and digits

### Step 2: Find the Hidden Flag
Looking at the encryption function:

```python
def h0RzE5G(dL8rY):
    P = O(7)  # Key is 7!
    uL5dQ9 = P.Y("CSC25{G1v3_c4eS4r_wHat_b3l0nGs}")  # This is the flag!
    return uL5dQ9
```

**Surprise!** The flag is actually hardcoded in the source code before encryption.

### Step 3: Verify with Decryption (Optional)
If you want to verify by decrypting the output, create a decryption script:

```python
import string

class O:
    def __init__(self, a):
        self.__key = a
        self.__alpha = string.ascii_lowercase + string.digits

    def __sH(self, c, b):
        if c in self.__alpha:
            return self.__alpha[(self.__alpha.index(c) + b) % len(self.__alpha)]
        else:
            return c

    def X(self, W):
        return ''.join(self.__sH(c, -self.__key) for c in W)

# Read the encrypted output
with open('output.txt', 'r') as f:
    encrypted = f.read().strip()

# Try the key found in the source code
cipher = O(7)
decrypted = cipher.X(encrypted)
print(f"Decrypted: {decrypted}")
```

## Alternative Solution: Brute Force

If you didn't notice the flag in the source code, you could brute-force all possible keys:

```python
import string

class O:
    def __init__(self, a):
        self.__key = a
        self.__alpha = string.ascii_lowercase + string.digits

    def __sH(self, c, b):
        if c in self.__alpha:
            return self.__alpha[(self.__alpha.index(c) + b) % len(self.__alpha)]
        else:
            return c

    def X(self, W):
        return ''.join(self.__sH(c, -self.__key) for c in W)

# Read encrypted text
with open('output.txt', 'r') as f:
    encrypted = f.read().strip()

# Brute force all possible keys (0-35 for lowercase + digits)
print("Trying all possible keys:")
for key in range(36):
    cipher = O(key)
    decrypted = cipher.X(encrypted)
    if "CSC25{" in decrypted or "csc25{" in decrypted.lower():
        print(f"Key {key}: {decrypted}")
```

## Solution
The flag can be found directly in the source code in the `h0RzE5G` function, or by decrypting the output with key 7.

**Flag:** `CSC25{G1v3_c4eS4r_wHat_b3l0nGs}`

## Key Takeaways
1. **Always examine source code first** - Sometimes the answer is hidden in plain sight
2. **Caesar cipher is weak** - With only 26 (or 36 with digits) possible keys, brute-forcing is trivial
3. **Obfuscation ≠ Security** - Variable name obfuscation doesn't protect the algorithm
4. **Modern cryptography needed** - Caesar cipher should never be used for real security

## Understanding the Flag
The flag message "Give caesar what belongs" is a reference to the famous quote "Render unto Caesar the things that are Caesar's" - a fitting message for a Caesar cipher challenge!

## Educational Value
This challenge teaches:
- How to recognize and analyze Caesar ciphers
- The importance of reading source code carefully
- Why ancient encryption methods are insecure
- Basic cryptanalysis techniques

## Security Recommendations
- Never use Caesar cipher for real encryption
- Don't hardcode secrets in source code
- Use modern, proven cryptographic algorithms (AES, RSA, etc.)
- Always use established cryptographic libraries
- Never implement your own crypto unless you're a cryptography expert
