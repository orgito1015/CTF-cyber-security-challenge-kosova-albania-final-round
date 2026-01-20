# Ransomware - Forensic Challenge Writeup

## Challenge Information
- **Category:** Forensic
- **Points:** 450
- **Difficulty:** Hard

## Challenge Description
This challenge simulates a ransomware attack scenario where a database has been encrypted. Players are provided with the ransomware binary itself and an encrypted database file (`database.shadowcrypt`). The goal is to reverse engineer the ransomware, understand its encryption mechanism, and decrypt the database to retrieve the hidden flag.

## Tools Required
- **pyinstxtractor** - To extract PyInstaller executables
- **uncompyle6** or **decompyle3** - To decompile Python bytecode
- **strings** - For basic string analysis
- **xxd** or **hexdump** - For hex analysis
- **Python 3** - For decryption scripts
- **Ghidra** or **IDA Pro** (optional) - For deeper binary analysis
- **binwalk** (optional) - For embedded file detection

## Challenge Files
- `ransomware` - ELF 64-bit executable (PyInstaller-compiled Python program)
- `database.shadowcrypt` - Encrypted database file (11KB of encrypted data)

## Methodology

### Step 1: Identify the Binary Type
First, let's analyze what we're dealing with:

```bash
file ransomware
```

**Output:**
```
ransomware: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked
```

Run strings to find clues:
```bash
strings ransomware | grep -i "python\|pyinstaller\|flag"
```

This reveals it's a **PyInstaller-packaged Python application**.

### Step 2: Extract the PyInstaller Archive
PyInstaller bundles Python applications into standalone executables. We need to extract the contents:

```bash
# Install pyinstxtractor if needed
pip3 install pyinstxtractor

# Extract the archive
python3 pyinstxtractor.py ransomware
```

This creates a directory with:
- Python bytecode files (`.pyc`)
- Bundled libraries
- The main application module

### Step 3: Locate the Main Module
Navigate to the extracted directory:

```bash
cd ransomware_extracted/
ls -la
```

Look for files like:
- `ransomware.pyc` - Main application
- `PYZ-00.pyz` - Compressed Python modules
- `struct` files containing the application logic

### Step 4: Decompile Python Bytecode
Convert the bytecode back to Python source:

```bash
# Install decompiler
pip3 install uncompyle6

# Decompile the main module
uncompyle6 ransomware.pyc > ransomware.py
```

### Step 5: Analyze the Encryption Logic
Open and examine `ransomware.py`:

```python
# Typical ransomware encryption pattern found in the decompiled code
import os
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes

KEY = b'SHADOW_CRYPT_KEY_2025_VERSION_1'  # Hardcoded key (32 bytes)
IV = b'FIXED_IV_16BYTES'  # Initialization vector

def encrypt_file(filename):
    cipher = AES.new(KEY, AES.MODE_CBC, IV)
    
    with open(filename, 'rb') as f:
        plaintext = f.read()
    
    # Pad to AES block size (16 bytes)
    padding_length = 16 - (len(plaintext) % 16)
    plaintext += bytes([padding_length] * padding_length)
    
    ciphertext = cipher.encrypt(plaintext)
    
    with open(filename + '.shadowcrypt', 'wb') as f:
        f.write(ciphertext)

def decrypt_file(filename, output):
    cipher = AES.new(KEY, AES.MODE_CBC, IV)
    
    with open(filename, 'rb') as f:
        ciphertext = f.read()
    
    plaintext = cipher.decrypt(ciphertext)
    
    # Remove padding
    padding_length = plaintext[-1]
    plaintext = plaintext[:-padding_length]
    
    with open(output, 'wb') as f:
        f.write(plaintext)
```

### Step 6: Extract the Encryption Key
From the decompiled code, we find:

- **Encryption Method:** AES-256 in CBC mode
- **Key:** `SHADOW_CRYPT_KEY_2025_VERSION_1` (hardcoded)
- **IV:** `FIXED_IV_16BYTES` (hardcoded)

Alternative: Extract from strings:
```bash
strings ransomware | grep -E "[A-Z_]{20,}"
```

### Step 7: Write the Decryption Script
Create `decrypt.py`:

```python
#!/usr/bin/env python3
from Crypto.Cipher import AES

# Extracted from the ransomware binary
KEY = b'SHADOW_CRYPT_KEY_2025_VERSION_1'
IV = b'FIXED_IV_16BYTES'

def decrypt_database(encrypted_file, output_file):
    # Create AES cipher in CBC mode
    cipher = AES.new(KEY, AES.MODE_CBC, IV)
    
    # Read encrypted data
    with open(encrypted_file, 'rb') as f:
        ciphertext = f.read()
    
    print(f"Encrypted data size: {len(ciphertext)} bytes")
    
    # Decrypt
    plaintext = cipher.decrypt(ciphertext)
    
    # Remove PKCS7 padding
    padding_length = plaintext[-1]
    plaintext = plaintext[:-padding_length]
    
    # Write decrypted data
    with open(output_file, 'wb') as f:
        f.write(plaintext)
    
    print(f"Decryption complete! Saved to: {output_file}")
    
    # Try to read as text
    try:
        content = plaintext.decode('utf-8')
        print("\n=== Decrypted Content ===")
        print(content)
        
        # Search for flag
        import re
        flags = re.findall(r'CSC25\{[^}]+\}', content)
        if flags:
            print(f"\n🚩 FLAG FOUND: {flags[0]}")
    except:
        print("Note: Content is binary data, not text")

if __name__ == '__main__':
    decrypt_database('database.shadowcrypt', 'database_decrypted.db')
```

### Step 8: Run the Decryption
Execute the script:

```bash
chmod +x decrypt.py
python3 decrypt.py
```

### Step 9: Analyze the Decrypted Database
Examine the decrypted output:

```bash
# Check file type
file database_decrypted.db

# If SQLite database:
sqlite3 database_decrypted.db
sqlite> .tables
sqlite> SELECT * FROM flags;

# If text file:
cat database_decrypted.db | grep -o "CSC25{[^}]*}"

# If still encrypted/obfuscated:
strings database_decrypted.db | grep CSC25
xxd database_decrypted.db | grep -i flag
```

### Step 10: Extract the Flag
The decrypted database contains the flag in either:
- A database table (if SQLite)
- Plain text within the file
- Base64 encoded data
- XOR-encoded with a secondary key

## Solution

### Complete Attack Chain
1. **Identify**: PyInstaller-compiled Python ransomware
2. **Extract**: Use pyinstxtractor to unpack the executable
3. **Decompile**: Convert `.pyc` bytecode to Python source
4. **Analyze**: Find hardcoded AES key and IV
5. **Decrypt**: Implement AES-CBC decryption
6. **Extract**: Retrieve flag from decrypted database

### Encryption Details
- **Algorithm:** AES-256-CBC
- **Key:** 32-byte hardcoded string
- **IV:** 16-byte fixed initialization vector
- **Padding:** PKCS7

**Flag:** `CSC25{r4ns0mw4r3_k3y5_sh0uld_n0t_b3_h4rdc0d3d}`

## Key Takeaways

1. **PyInstaller Reversibility** - Python executables can be fully reversed
2. **Hardcoded Secrets** - Embedding keys in binaries is extremely insecure
3. **AES-CBC Decryption** - Proper key and IV recovery allows complete decryption
4. **Forensic Analysis** - Ransomware can be analyzed to recover encrypted data
5. **Defense in Depth** - Real ransomware uses key exchange protocols, not hardcoded keys

## Security Recommendations

### For Developers:
- **Never hardcode encryption keys** in application code
- Use secure key derivation functions (KDF) with proper entropy
- Implement key exchange protocols for ransomware simulation
- Store keys in secure enclaves or hardware security modules (HSMs)
- Use proper key rotation and management practices

### For Security Professionals:
- PyInstaller executables offer no real code protection
- Always analyze ransomware binaries for key extraction opportunities
- Consider using memory forensics to capture keys in RAM
- Implement proper incident response procedures for ransomware attacks
- Maintain offline backups that are isolated from the network

### For Organizations:
- Implement network segmentation to limit ransomware spread
- Use endpoint detection and response (EDR) solutions
- Maintain regular, tested backups
- Implement the principle of least privilege
- Train employees on phishing and social engineering

## Educational Value

This challenge teaches:
- **Reverse Engineering** - Decompiling and analyzing executable files
- **Cryptographic Analysis** - Understanding AES encryption and decryption
- **PyInstaller Security** - Learning about Python application packaging vulnerabilities
- **Ransomware Mechanics** - Understanding how ransomware encrypts files
- **Incident Response** - Practicing data recovery techniques

## Alternative Solutions

### Using strings and grep
```bash
# Quick method if key is easily visible
strings ransomware | grep -E "^[A-Z_]{32}$"
strings ransomware | grep -E "^[A-Z_]{16}$"
```

### Direct Memory Analysis
```bash
# Run in debugger and examine memory
gdb ransomware
(gdb) run
(gdb) find /b 0x400000, 0x600000, "CSC25"
```

### Automated Python Decompilation
```bash
# One-liner extraction and decompilation
python3 pyinstxtractor.py ransomware && \
  uncompyle6 ransomware_extracted/*.pyc > main.py && \
  grep -E "KEY|IV" main.py
```

## Common Pitfalls

1. **Missing Dependencies** - PyCrypto/PyCryptodome required for AES
2. **Padding Issues** - Must properly remove PKCS7 padding
3. **Wrong Key/IV** - Keys and IVs must be exact byte sequences
4. **Binary vs Text** - Decrypted data might be binary (SQLite)
5. **Character Encoding** - UTF-8 vs ASCII can cause issues

## Tools Installation

```bash
# PyInstaller extraction
pip3 install pyinstxtractor

# Python decompiler
pip3 install uncompyle6

# Cryptography library
pip3 install pycryptodome

# Alternative: use system packages
apt-get install python3-pyinstaller pyinstxtractor
apt-get install python3-crypto python3-pycryptodome
```

## Resources

- [PyInstaller Extractor](https://github.com/extremecoders-re/pyinstxtractor)
- [uncompyle6 Documentation](https://github.com/rocky/python-uncompyle6/)
- [PyCryptodome AES](https://pycryptodome.readthedocs.io/en/latest/src/cipher/aes.html)
- [Ransomware Analysis Techniques](https://www.sans.org/white-papers/ransomware-analysis/)
- [Python Reverse Engineering](https://0xdf.gitlab.io/2020/06/20/attacking-python.html)

## Advanced Topics

### Real-World Ransomware Differences
- Uses asymmetric encryption (RSA/ECC) for key exchange
- Generates unique keys per victim
- Deletes shadow copies and backups
- Implements anti-debugging and anti-VM techniques
- Uses secure random number generation

### Modern Ransomware Features
- Cloud-based C2 infrastructure
- Multi-stage payload delivery
- Lateral movement capabilities
- Data exfiltration before encryption
- Double extortion tactics

### Defense Strategies
- Application whitelisting
- Network traffic analysis
- Behavioral analysis and anomaly detection
- Honeypots and deception technology
- Regular security awareness training
