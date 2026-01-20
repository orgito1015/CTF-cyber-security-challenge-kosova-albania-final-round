# CryptoDome - Cryptography Challenge Writeup

## Challenge Information
- **Category:** Cryptography
- **Points:** 300
- **Difficulty:** Medium

## Challenge Description
"The bank is using a strong encryption mechanism. Try to break the logic of the new encryption."

This challenge implements a custom encryption scheme with food-themed function names ("lasagna", "sauce", "cheese") that obfuscate the actual cryptographic operations.

## Tools Required
- **Python 3** - For running encryption/decryption scripts
- **base64 module** - For handling Base64 encoding/decoding

## Methodology

### Step 1: Analyze the Encryption Scheme
Examine `challenge.py` to understand the encryption flow:

```python
import random
import base64

def sauce(s):
    r = []
    random.seed(2025)  # Fixed seed!
    for _ in range(len(s)):
        r.append(random.randint(0, 255))
    return r

def lasagna(text, flavor):
    cooked = []
    for i in range(len(text)):
        a = ord(text[i])
        b = flavor[i]
        cooked.append((a ^ ((b << (i % 4)) | (b >> (8 - (i % 4)))) & 0xff))
    return base64.b64encode(bytes(cooked)).decode()
```

**Encryption Flow:**
1. `sauce()` - Generates a "random" key using seed 2025
2. `lasagna()` - XORs plaintext with bit-rotated key
3. Result is Base64-encoded

### Step 2: Identify the Vulnerability
Key observations:
- **Fixed random seed (2025)** - Key is completely deterministic
- **Bit rotation** - Adds complexity but is reversible
- **XOR operation** - Self-inverse (applying XOR twice returns original)
- **Base64 encoding** - Just encoding, not encryption

### Step 3: Understand the Bit Rotation
The encryption rotates key bytes before XORing:

```python
rotated_key = (b << (i % 4)) | (b >> (8 - (i % 4)))
```

- Left rotate by `(i % 4)` positions
- Rotation amount cycles: 0, 1, 2, 3, 0, 1, 2, 3...

### Step 4: Implement Reverse Lasagna
To decrypt, we reverse the process:

```python
def reverse_lasagna(encrypted, flavor):
    # Decode Base64
    cooked = base64.b64decode(encrypted)
    original_text = []
    
    for i in range(len(cooked)):
        b = flavor[i]
        # Apply same bit rotation
        rotated = (b << (i % 4)) | (b >> (8 - (i % 4)))
        rotated = rotated & 0xff
        # XOR again (self-inverse)
        a = cooked[i] ^ rotated
        original_text.append(chr(a))
    
    return ''.join(original_text)
```

### Step 5: Complete Solution Script

```python
import random
import base64

def sauce(length):
    r = []
    random.seed(2025)  # Use same seed as encryption
    for _ in range(length):
        r.append(random.randint(0, 255))
    return r

def reverse_lasagna(encrypted, flavor):
    cooked = base64.b64decode(encrypted)
    original_text = []
    for i in range(len(cooked)):
        b = flavor[i]
        a = cooked[i] ^ (((b << (i % 4)) | (b >> (8 - (i % 4)))) & 0xff)
        original_text.append(chr(a))
    return ''.join(original_text)

# Encrypted output from challenge
encrypted_output = "aboiMov66WuF7rvzCKw39Y1QYU4XiAf8r+SAEXS3"

# Decode to get correct length for key generation
decoded_length = len(base64.b64decode(encrypted_output))

# Generate the same "flavor" (key) using the same seed
flavor = sauce(decoded_length)

# Decrypt
flag = reverse_lasagna(encrypted_output, flavor)
print("Decrypted Flag:", flag)
```

### Step 6: Execute and Retrieve Flag

```bash
python script.py
```

## Solution
**Flag:** `CSC25{0bFusC4t4d_bY_tH3_d3ViL}`

## Understanding the Flag
"Obfuscated by the devil" - A fitting message! The challenge used creative function names to obfuscate what was really happening, but the underlying crypto was still weak.

## Why This is Insecure

### Critical Vulnerabilities:
1. **Deterministic Key Generation** - Fixed seed = predictable key
2. **No Key Secrecy** - Anyone with the code can reproduce the key
3. **Simple XOR Cipher** - Once key is known, trivial to decrypt
4. **Obfuscation ≠ Security** - Creative naming doesn't hide weak crypto

### Attack Surface:
- Known algorithm + known seed = complete break
- No authentication or integrity checks
- Base64 is encoding, not encryption

## Key Takeaways
1. **Never use predictable random seeds** for cryptographic purposes
2. **Code obfuscation doesn't provide security** - Only makes reading harder
3. **Custom crypto is dangerous** - Use proven algorithms (AES, ChaCha20)
4. **XOR requires truly random keys** - Otherwise completely insecure

## How the Functions Work

### The "Lasagna" Encryption:
```
plaintext → XOR with rotated key → Base64 encode → ciphertext
```

### The "Sauce" Key Generator:
```
Fixed seed 2025 → Generate random bytes → Return as key
```

### Bit Rotation Visualized:
```
For i=0, rotate by 0: no change
For i=1, rotate by 1: 11010010 → 10100101
For i=2, rotate by 2: 11010010 → 01001011
For i=3, rotate by 3: 11010010 → 10010110
For i=4, rotate by 0: back to no change
```

## Educational Value
This challenge teaches:
- How to reverse-engineer custom encryption schemes
- The danger of using fixed random seeds in cryptography
- Why obfuscation is not a security measure
- How XOR encryption works and its limitations
- The importance of proper key management

## Cryptographic Best Practices
1. **Use standard libraries** - PyCryptodome, cryptography.io
2. **Never roll your own crypto** - Unless you're a cryptographer
3. **Use proven algorithms** - AES-GCM, ChaCha20-Poly1305
4. **Proper key derivation** - PBKDF2, Argon2, scrypt
5. **Secure random sources** - os.urandom(), secrets module
6. **Add authentication** - Use AEAD modes (GCM, CCM, Poly1305)

## Related Concepts
- **Stream Ciphers** - XOR with keystream (similar to this challenge)
- **Block Ciphers** - AES, DES (more secure alternatives)
- **Key Derivation Functions** - Proper way to generate keys from passwords
- **CSPRNG** - Cryptographically Secure Pseudo-Random Number Generators
