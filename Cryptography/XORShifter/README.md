# XORShifter - Cryptography Challenge Writeup

## Challenge Information
- **Category:** Cryptography
- **Points:** 150
- **Difficulty:** Easy-Medium

## Challenge Description
This challenge involves a custom XOR-based encryption with bit rotation. The twist is that the key is generated using a fixed random seed, making the encryption deterministic and reversible.

**Challenge hint:** "Another XOR encryption maybe? Looking for a key? Nice try!"

## Background: XOR Cipher with Bit Rotation
- **XOR (Exclusive OR)** is a bitwise operation commonly used in cryptography
- **Bit rotation** shifts bits left or right, wrapping around
- **Random seed** makes "random" numbers predictable and reproducible

## Tools Required
- **Python 3** - For running the decryption script
- **Text Editor** - For code analysis

## Methodology

### Step 1: Analyze the Challenge Code
Examine `challenge.py` to understand the encryption mechanism:

```python
import random

f = "CSC25{fake_flag_for_testing}"
s = 1337  # Fixed seed!

def k(x):
    random.seed(s)
    return [random.randint(0, 255) for _ in range(x)]

def x(d, j):
    return [(ord(d[i]) ^ (((j[i] << (i % 8)) | (j[i] >> (8 - (i % 8)))) & 0xff)) for i in range(len(d))]
```

**Key observations:**
1. The random seed is **hardcoded to 1337**
2. Key generation uses Python's `random` module with this fixed seed
3. Encryption uses XOR with bit-rotated key bytes
4. Rotation amount changes with position: `(i % 8)`

### Step 2: Understand the Bit Rotation
The encryption performs a bit rotation on each key byte before XORing:

```python
rotated_key = (key[i] << (i % 8)) | (key[i] >> (8 - (i % 8)))
```

This is a left circular shift by `(i % 8)` positions.

### Step 3: Reproduce the Key
Since the seed is known (1337), we can regenerate the exact same key:

```python
import random

def get_key(length):
    random.seed(1337)  # Use the same seed!
    return [random.randint(0, 255) for _ in range(length)]
```

### Step 4: Implement Decryption
The decryption is essentially the same as encryption (XOR is self-inverse):

```python
def decrypt(enc_bytes, key):
    result = []
    for i in range(len(enc_bytes)):
        # Apply same rotation to key
        rotated = (key[i] << (i % 8)) | (key[i] >> (8 - (i % 8)))
        rotated = rotated & 0xff  # Keep it as a byte
        # XOR to decrypt
        result.append(chr(enc_bytes[i] ^ rotated))
    return ''.join(result)
```

### Step 5: Complete Solution Script

```python
import random

def get_key(length):
    random.seed(1337)
    return [random.randint(0, 255) for _ in range(length)]

def decrypt(enc_bytes, key):
    return ''.join([chr(enc_bytes[i] ^ (((key[i] << (i % 8)) | (key[i] >> (8 - (i % 8)))) & 0xff)) for i in range(len(enc_bytes))])

# Read the encrypted hex string from output.txt
with open('output.txt', 'r') as f:
    hex_str = f.read().strip()

# Convert hex string to bytes
enc_bytes = [int(hex_str[i:i+2], 16) for i in range(0, len(hex_str), 2)]

# Generate the key using the same seed
key = get_key(len(enc_bytes))

# Decrypt the message
flag = decrypt(enc_bytes, key)
print(f"Flag: {flag}")
```

### Step 6: Run the Script

```bash
python script.py
```

## Solution
**Flag:** `CSC25{tHis_i5_tHE_3asY_p4rT}`

## Why This is Insecure

### Critical Flaws:
1. **Fixed Random Seed** - Using a hardcoded seed makes the "random" key completely predictable
2. **Visible in Source** - The seed value is exposed in the encryption code
3. **XOR is Symmetric** - Same operation encrypts and decrypts, no key verification
4. **No Key Secrecy** - If the seed is known, the entire keystream can be regenerated

### Real-World Implications:
- In production code, using a fixed seed destroys all randomness
- Random number generators for cryptography should use `secrets` module in Python, not `random`
- Cryptographic keys should be generated with proper entropy sources

## Key Takeaways
1. **Never use fixed seeds** for cryptographic key generation
2. **random module ≠ cryptographic randomness** - Use `secrets` or `os.urandom()` for crypto
3. **XOR requires a truly random key** - If the key is predictable, XOR provides no security
4. **Source code review is essential** - The vulnerability was in the implementation, not the algorithm

## Bit Rotation Explained
For those curious about the bit rotation:

```python
# Example: Rotate byte 0b11010010 left by 3 positions
# Original:     11010010
# Left shift:   10010000  (shifts 3 left, loses leftmost bits)
# Right shift:  00000110  (shifts 5 right, gets the lost bits)
# OR combine:   10010110  (rotated result)
```

This rotation adds complexity but doesn't improve security if the key is known.

## Educational Value
This challenge teaches:
- Understanding XOR encryption
- The importance of proper random number generation in cryptography
- How to reverse-engineer simple encryption schemes
- Why deterministic "randomness" is dangerous in security

## Recommended Tools for Crypto CTF Challenges
- **CyberChef** - Web-based crypto/encoding tool
- **Python cryptography library** - For proper crypto implementations
- **XORtool** - Automated XOR cipher breaking
- **HashCat** - Password/hash cracking
- **John the Ripper** - Password cracking
