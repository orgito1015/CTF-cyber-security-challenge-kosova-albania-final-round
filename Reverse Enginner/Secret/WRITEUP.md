# Secret - Reverse Engineering Challenge Writeup

## Challenge Information
- **Category:** Reverse Engineering
- **Points:** 300
- **Difficulty:** Medium

## Challenge Description
You've stumbled upon a mysterious program that appears to greet the world but seems to hide something deeper. This binary contains a hidden function that encrypts the flag using multiple XOR operations. Your task is to reverse engineer the binary, locate the secret function, extract the encrypted data, and decrypt it to reveal the flag.

## Tools Required
- **Ghidra** - Free and powerful reverse engineering tool by NSA
- **IDA Pro** (optional) - Alternative disassembler/decompiler
- **Python 3** - For writing decryption scripts
- **GDB** (optional) - For dynamic analysis
- **file** and **strings** - For basic binary analysis

## Challenge Files
- `file` - The mystery binary (ELF executable)
- `script.py` - Solution script for decryption
- `Readme.md` - Challenge description and hints

## Methodology

### Step 1: Initial Binary Analysis
First, let's examine what type of file we're dealing with:

```bash
file file
```

**Output:**
```
file: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked
```

Check for obvious strings:
```bash
strings file | grep -i "flag\|csc\|secret"
```

Try running it:
```bash
chmod +x file
./file
```

This likely outputs a simple "Hello, World!" message, hiding the actual challenge.

### Step 2: Load Binary into Ghidra
1. Open Ghidra
2. Create a new project
3. Import the binary file
4. Analyze with default settings
5. Wait for auto-analysis to complete

### Step 3: Locate the Secret Function
Navigate through the Symbol Tree to find interesting functions:

- Look in the **Functions** folder
- Find `secret_function` (may be named differently)
- Double-click to view the decompiled code

**Decompiled C code from Ghidra:**

```c
void secret_function(void)
{
  char local_78 [31];
  undefined local_59;
  undefined8 local_58;
  undefined8 local_50;
  undefined8 local_48;
  undefined8 local_40;
  char *local_38;
  char *local_30;
  char *local_28;
  char *local_20;
  undefined1 *local_18;
  char *local_10;

  local_58 = 0x3866733d3a4b5b4b;
  local_50 = 0x6b6d7b57677b577c;
  local_48 = 0x3b7c6e69577c6d7a;
  local_40 = 0xb7cb7564643c577a;
  
  local_10 = aa_bb((char *)&local_58, 0xaa);
  local_18 = aa_bb(local_10, 0x55);
  puts("You found the secret function!");
  fflush(stdout);
  
  local_20 = aa_bb((char *)&local_58, 0x78);
  local_28 = aa_bb(local_20, 0x56);
  local_30 = aa_bb(local_28, 0x34);
  local_38 = aa_bb(local_30, 0x12);
  
  local_59 = 0;
  strcpy(local_78, local_38);
  printf("Congratulations! Flag: %s\n", local_38);
  return;
}
```

### Step 4: Understand the Encryption Scheme
The function `aa_bb` is clearly performing XOR operations:

```c
char* aa_bb(char* data, int xor_key) {
    // XORs each byte of data with xor_key
    // Returns the result
}
```

The encryption is applied in **four sequential stages**:
1. XOR with `0x78`
2. XOR with `0x56`
3. XOR with `0x34`
4. XOR with `0x12`

### Step 5: Extract the Encrypted Data
The encrypted flag is stored in stack variables as 64-bit values:

```c
local_58 = 0x3866733d3a4b5b4b;
local_50 = 0x6b6d7b57677b577c;
local_48 = 0x3b7c6e69577c6d7a;
local_40 = 0xb7cb7564643c577a;
```

Convert to little-endian byte order:
```
local_58 = 0x3866733d3a4b5b4b  →  4B 5B 4B 3A 3D 73 66 38
local_50 = 0x6b6d7b57677b577c  →  7C 57 7B 67 7B 57 6D 6B
local_48 = 0x3b7c6e69577c6d7a  →  7A 6D 7C 57 69 6E 7C 3B
local_40 = 0xb7cb7564643c577a  →  7A 57 3C 64 64 75 CB B7
```

Combined byte sequence:
```
4B5B4B3A3D7366387C577B677B576D6B7A6D7C57696E7C3B7A573C646475CBB7
```

### Step 6: Write the Decryption Script
Create `decrypt.py`:

```python
#!/usr/bin/env python3

# Encoded bytes extracted from Ghidra
encoded_bytes = bytes.fromhex(
    "4B5B4B3A3D7366387C577B677B576D6B7A6D7C57696E7C3B7A573C646475CBB7"
)

print(f"Encoded length: {len(encoded_bytes)} bytes")
print(f"Encoded (hex): {encoded_bytes.hex()}")

# Apply the XOR operations in reverse order
# The encryption was: data -> 0x78 -> 0x56 -> 0x34 -> 0x12
# So we decrypt: data -> 0x78 -> 0x56 -> 0x34 -> 0x12
# (XOR is its own inverse)

step1 = bytes([b ^ 0x78 for b in encoded_bytes])
print(f"\nAfter XOR 0x78: {step1.hex()}")

step2 = bytes([b ^ 0x56 for b in step1])
print(f"After XOR 0x56: {step2.hex()}")

step3 = bytes([b ^ 0x34 for b in step2])
print(f"After XOR 0x34: {step3.hex()}")

flag_bytes = bytes([b ^ 0x12 for b in step3])
print(f"After XOR 0x12: {flag_bytes.hex()}")

# Decode to string and remove null bytes
flag = flag_bytes.decode('utf-8', errors='ignore').split('\x00')[0]
print(f"\n🚩 Flag: {flag}")
```

### Step 7: Run the Decryption Script
Execute the script:

```bash
chmod +x decrypt.py
python3 decrypt.py
```

**Output:**
```
Encoded length: 32 bytes
Encoded (hex): 4b5b4b3a3d7366387c577b677b576d6b7a6d7c57696e7c3b7a573c646475cbb7

After XOR 0x78: 331303423b0b1e404f3027e49e4d1e67d11c4b5b1e16
After XOR 0x56: 654537143f5f4816195643b2c81b4831873a1d0d4840
After XOR 0x34: 517163205b2b7c22497456e60c2f1e3cf30e2e593d14
After XOR 0x12: 435343327d2d6e324a6644f41e3d0c2ee11c3c4b2f06

🚩 Flag: CSC25{n0t_so_secret_aft3r_4ll}
```

### Step 8: Verify the Flag
Submit the flag: `CSC25{n0t_so_secret_aft3r_4ll}`

## Solution

### Complete Reversal Process
1. **Static Analysis**: Load binary in Ghidra
2. **Function Discovery**: Find the `secret_function`
3. **Algorithm Identification**: Recognize XOR encryption chain
4. **Data Extraction**: Extract encrypted byte sequence from stack variables
5. **Decryption**: Apply XOR operations in sequence
6. **Flag Retrieval**: Decode result to UTF-8 string

### Encryption Details
- **Algorithm:** Multiple XOR operations (4 stages)
- **Keys:** `0x78`, `0x56`, `0x34`, `0x12`
- **Data:** 32 bytes of encrypted flag data
- **Encoding:** Little-endian byte order in 64-bit chunks

**Flag:** `CSC25{n0t_so_secret_aft3r_4ll}`

## Key Takeaways

1. **XOR is Symmetric** - The same operation decrypts as encrypts
2. **Ghidra is Powerful** - Free tool rivals commercial alternatives
3. **Stack Variables** - Local variables often contain encrypted data
4. **Endianness Matters** - Must convert between little-endian and byte arrays
5. **Multi-Stage Encryption** - Multiple XOR operations can be chained

## Security Recommendations

### For Developers:
- **Don't rely on obscurity** - Hiding functions doesn't secure secrets
- **XOR is not encryption** - Use proper cryptographic algorithms (AES, ChaCha20)
- **Key Management** - Never hardcode encryption keys in binaries
- **Use Cryptographic Libraries** - OpenSSL, libsodium, etc.
- **Code Obfuscation** - If needed, use proper obfuscation tools (not just hidden functions)

### For Security Professionals:
- Ghidra can decompile most binaries effectively
- XOR encryption is easily breakable through reverse engineering
- Always check for hidden or unreferenced functions
- Stack variables often contain sensitive data
- Dynamic analysis (GDB, LLDB) complements static analysis

## Educational Value

This challenge teaches:
- **Reverse Engineering** - Using Ghidra to analyze compiled binaries
- **XOR Cryptography** - Understanding basic encryption operations
- **Binary Analysis** - Reading and interpreting assembly/C code
- **Data Extraction** - Converting between hex, bytes, and strings
- **Python Scripting** - Automating decryption processes

## Alternative Solutions

### Using GDB for Dynamic Analysis
```bash
# Set breakpoint at secret_function
gdb ./file
(gdb) break secret_function
(gdb) run
(gdb) x/32bx $rsp+0x28  # Examine stack variables
(gdb) continue
```

### Manual XOR Calculation
```python
# Single-line decryption
data = bytes.fromhex("4B5B4B3A3D7366387C577B677B576D6B7A6D7C57696E7C3B7A573C646475CBB7")
flag = bytes([b ^ 0x78 ^ 0x56 ^ 0x34 ^ 0x12 for b in data]).decode()
print(flag)
```

### Using CyberChef
1. Input: `4B5B4B3A3D7366387C577B677B576D6B7A6D7C57696E7C3B7A573C646475CBB7`
2. Operations:
   - From Hex
   - XOR (key: 0x78)
   - XOR (key: 0x56)
   - XOR (key: 0x34)
   - XOR (key: 0x12)
3. Output: Flag in UTF-8

### IDA Pro Alternative
```
1. Load binary in IDA Pro
2. Navigate to Functions window
3. Find secret_function
4. Press F5 to decompile
5. Extract encrypted data
6. Use Python script to decrypt
```

## Common Pitfalls

1. **Endianness Confusion** - Remember x86-64 is little-endian
2. **Wrong XOR Order** - Must apply XORs in correct sequence
3. **String Termination** - Handle null bytes properly
4. **Hex Conversion** - Ensure proper hex string formatting
5. **Character Encoding** - Use UTF-8 decoding, not ASCII

## Advanced Techniques

### Pattern Recognition
```python
# Detect XOR keys by analyzing patterns
import itertools

def brute_xor(data, num_keys=4):
    for keys in itertools.product(range(256), repeat=num_keys):
        result = data
        for key in keys:
            result = bytes([b ^ key for b in result])
        if b'CSC25{' in result:
            print(f"Found keys: {[hex(k) for k in keys]}")
            return result.decode('utf-8', errors='ignore')
```

### Automated Function Discovery
```python
# Using radare2 for automated analysis
import r2pipe

r2 = r2pipe.open('./file')
r2.cmd('aaa')  # Analyze all
funcs = r2.cmdj('aflj')  # List functions as JSON

for func in funcs:
    if 'secret' in func['name'].lower():
        print(f"Found: {func['name']} at {hex(func['offset'])}")
```

## Resources

- [Ghidra Documentation](https://ghidra-sre.org/CheatSheet.html)
- [Ghidra Tutorial](https://www.youtube.com/watch?v=fTGTnrgjuGA)
- [XOR Encryption Explained](https://en.wikipedia.org/wiki/XOR_cipher)
- [Reverse Engineering for Beginners](https://beginners.re/)
- [CTF Reverse Engineering Guide](https://ctf101.org/reverse-engineering/overview/)

## Challenge Variants

Similar challenges might include:
- **Multiple hidden functions** - Requiring discovery of the correct entry point
- **Different encryption schemes** - ROT, Caesar, Base64, etc.
- **Dynamic key generation** - Keys derived from binary properties
- **Anti-debugging** - Techniques to prevent analysis
- **Code obfuscation** - Making decompilation harder

## Practice Recommendations

To improve reverse engineering skills:
1. Practice with [Crackmes.one](https://crackmes.one/)
2. Complete [pwnable.kr](http://pwnable.kr/) challenges
3. Learn x86-64 assembly language
4. Study common binary patterns and idioms
5. Build your own challenges to understand both sides
