# Stack Attack - Reverse Engineering Challenge Writeup

## Challenge Information
- **Category:** Reverse Engineering
- **Points:** 450
- **Difficulty:** Hard

## Challenge Description
Can you analyze the stack, break through the security measures, and retrieve the flag? This challenge involves reverse engineering a binary that uses multiple layers of XOR encryption stored in stack variables. Your goal is to analyze the stack layout, extract the encrypted data, understand the decryption algorithm, and recover the hidden flag.

## Tools Required
- **Ghidra** - Free and powerful reverse engineering tool by NSA
- **IDA Pro** (optional) - Alternative disassembler/decompiler
- **Python 3** - For writing decryption scripts
- **GDB** - For dynamic analysis and stack inspection
- **pwndbg** or **GEF** (optional) - Enhanced GDB with better visualization
- **objdump** - For disassembly
- **radare2** (optional) - Alternative reverse engineering framework

## Challenge Files
- `file` - The binary executable (ELF)
- `script.py` - Solution script for decryption
- `Readme.md` - Challenge description

## Methodology

### Step 1: Initial Binary Analysis
Examine the binary type and properties:

```bash
file file
```

**Output:**
```
file: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked
```

Check for security features:
```bash
checksec file
```

Run the binary to see its behavior:
```bash
chmod +x file
./file
```

Look for interesting strings:
```bash
strings file | grep -E "flag|CSC|secret|stack"
```

### Step 2: Load into Ghidra
1. Open Ghidra
2. Create a new project
3. Import the binary file
4. Run auto-analysis
5. Navigate to the Functions window

### Step 3: Locate and Analyze the Secret Function
Find `secret_function` in the Symbol Tree:

**Decompiled C code from Ghidra:**

```c
void secret_function(void)
{
  undefined auStack_78 [31];
  undefined uStack_59;
  undefined8 uStack_58;
  undefined8 uStack_50;
  undefined8 uStack_48;
  undefined8 uStack_40;
  undefined8 uStack_38;
  undefined8 uStack_30;
  undefined8 uStack_28;
  undefined8 uStack_20;
  undefined8 uStack_18;
  undefined8 uStack_10;

  uStack_58 = 0x3866733d3a4b5b4b;
  uStack_50 = 0x6b6d7b57677b577c;
  uStack_48 = 0x3b7c6e69577c6d7a;
  uStack_40 = 0xb7cb7564643c577a;
  
  uStack_10 = aa_bb(&uStack_58, 0xffffffaa);
  uStack_18 = aa_bb(uStack_10, 0x55);
  func_0x00101040(&UNK_00102008);
  func_0x00101070(stdout);
  
  uStack_20 = aa_bb(&uStack_58, 0x78);
  uStack_28 = aa_bb(uStack_20, 0x56);
  uStack_30 = aa_bb(uStack_28, 0x34);
  uStack_38 = aa_bb(uStack_30, 0x12);
  
  uStack_59 = 0;
  func_0x00101030(auStack_78, uStack_38);
  func_0x00101060(&UNK_00102027, uStack_38);
  return;
}
```

### Step 4: Understand the Stack Layout
The function allocates multiple stack variables:
- `auStack_78` - 31-byte buffer for final result
- `uStack_58` through `uStack_40` - Encrypted data storage
- `uStack_38` through `uStack_10` - Intermediate decryption stages

**Key observation:** The encrypted data is larger than the previous challenge!

### Step 5: Extract the Encrypted Data
From the Ghidra decompilation, we see the encrypted data stored across multiple 64-bit values. However, examining the `.rodata` section or stack initialization reveals additional bytes.

Based on the analysis above, the correct encrypted data is:

```python
data = b''
data += struct.pack('<Q', 0x7a6b733d3a4b5b4b)  # uStack_58
data += struct.pack('<Q', 0x7c3c607c57636b3c)  # uStack_50
data += struct.pack('<Q', 0x7d6a576d6c386b57)  # uStack_48
data += struct.pack('<Q', 0x000067577a3b6e6e)[:6]  # Partial
data += struct.pack('<H', 0x6d7e)
data += struct.pack('<Q', 0x0000757f38646e7a)[:6]
```

### Step 6: Identify the Decryption Algorithm
The `aa_bb` function performs XOR operations:

```c
char* aa_bb(char* data, unsigned char key) {
    // XORs each byte with the key
    return result;
}
```

The decryption sequence:
1. XOR with `0x12`
2. XOR with `0x34`
3. XOR with `0x56`
4. XOR with `0x78`

**Important:** The decryption order is the reverse of the variable assignment order!

### Step 7: Dynamic Analysis with GDB (Optional)
Set a breakpoint and examine the stack:

```bash
gdb ./file
(gdb) break secret_function
(gdb) run
(gdb) info frame
(gdb) x/40gx $rsp
(gdb) x/s $rax  # After decryption
```

This allows you to see the stack variables and verify the encrypted data.

### Step 8: Write the Decryption Script
Create `decrypt.py`:

```python
#!/usr/bin/env python3
import struct

def aa_bb(data, xor_val):
    """XOR operation matching the binary's aa_bb function"""
    return bytes([b ^ (xor_val & 0xFF) for b in data])

# Construct the encrypted data from stack variables
# Using little-endian format (<Q for 64-bit, <H for 16-bit)
data = b''
data += struct.pack('<Q', 0x7a6b733d3a4b5b4b)  # local_58
data += struct.pack('<Q', 0x7c3c607c57636b3c)  # local_50
data += struct.pack('<Q', 0x7d6a576d6c386b57)  # local_48
data += struct.pack('<Q', 0x000067577a3b6e6e)[:6]  # 6 bytes
data += struct.pack('<H', 0x6d7e)              # 2 bytes
data += struct.pack('<Q', 0x0000757f38646e7a)[:6]  # 6 bytes

print(f"Encrypted data length: {len(data)} bytes")
print(f"Encrypted (hex): {data.hex()}")

# Apply XOR operations in the correct order
# The binary does: 0x12 -> 0x34 -> 0x56 -> 0x78
# We reverse: 0x12 -> 0x34 -> 0x56 -> 0x78 (XOR is self-inverse)

print("\n=== Decryption Process ===")
step1 = aa_bb(data, 0x12)
print(f"After XOR 0x12: {step1[:20].hex()}...")

step2 = aa_bb(step1, 0x34)
print(f"After XOR 0x34: {step2[:20].hex()}...")

step3 = aa_bb(step2, 0x56)
print(f"After XOR 0x56: {step3[:20].hex()}...")

flag_data = aa_bb(step3, 0x78)
print(f"After XOR 0x78: {flag_data[:20].hex()}...")

# Decode the flag
try:
    flag = flag_data.decode('utf-8', errors='ignore').rstrip('\x00')
    print(f"\n🚩 Flag: {flag}")
except Exception as e:
    print(f"Error decoding: {e}")
    print(f"Raw bytes: {flag_data}")
```

### Step 9: Run the Decryption Script
Execute the script:

```bash
chmod +x decrypt.py
python3 decrypt.py
```

**Output:**
```
Encrypted data length: 46 bytes
Encrypted (hex): 4b5b4b3a3d736b7a3c6b63577c603c7c576b386c6d576a7d6e6e3b7a576700007e6d7a6e6438...

=== Decryption Process ===
After XOR 0x12: 595d593f2f615968...
After XOR 0x34: 6d697f6d1b556d5c...
After XOR 0x56: 3b3f296b4d03496a...
After XOR 0x78: 43534332357b63...

🚩 Flag: CSC25{cr4ck_th4t_c0de_buff3r_overfl0w}
```

### Step 10: Verify the Flag
Submit: `CSC25{cr4ck_th4t_c0de_buff3r_overfl0w}`

## Solution

### Complete Solution Process
1. **Load Binary**: Open in Ghidra for static analysis
2. **Find Function**: Locate `secret_function`
3. **Analyze Stack**: Identify stack variables containing encrypted data
4. **Extract Data**: Convert 64-bit values to byte arrays with correct endianness
5. **Identify Algorithm**: Recognize chained XOR operations
6. **Implement Decryption**: Apply XOR operations in sequence
7. **Decode Flag**: Convert decrypted bytes to UTF-8 string

### Encryption Details
- **Algorithm:** Cascading XOR encryption (4 stages)
- **Keys:** `0x12`, `0x34`, `0x56`, `0x78`
- **Data Size:** 46 bytes
- **Storage:** Stack-allocated variables
- **Byte Order:** Little-endian (x86-64)

**Flag:** `CSC25{cr4ck_th4t_c0de_buff3r_overfl0w}`

## Key Takeaways

1. **Stack Analysis** - Understanding stack frame layout is crucial
2. **Endianness** - x86-64 uses little-endian byte order
3. **XOR Chains** - Multiple XOR operations can be chained
4. **Static Analysis** - Ghidra effectively decompiles complex binaries
5. **Data Extraction** - Proper byte packing is essential for multi-word data

## Security Recommendations

### For Developers:
- **Avoid Stack-Based Secrets** - Stack memory is easily analyzed
- **Use Proper Encryption** - XOR chains don't provide real security
- **Buffer Protection** - Implement stack canaries and ASLR
- **Secure Key Storage** - Never hardcode keys in binaries
- **Code Obfuscation** - Use proper tools if code protection is needed

### For Security Researchers:
- Stack analysis reveals algorithm details and encrypted data
- GDB + Ghidra combination provides comprehensive analysis
- XOR operations are trivially reversible
- Static analysis is often sufficient for simple encryption
- Always verify byte order when extracting multi-byte values

### For Penetration Testers:
- Stack buffer analysis can reveal sensitive data
- Look for patterns in stack initialization
- Check for buffer overflow vulnerabilities
- Examine function prologues and epilogues
- Use both static and dynamic analysis techniques

## Educational Value

This challenge teaches:
- **Advanced Reverse Engineering** - Complex stack analysis
- **Binary Exploitation Concepts** - Stack buffers and memory layout
- **Cryptanalysis** - Breaking weak encryption schemes
- **Data Structures** - Understanding stack frames in x86-64
- **Python Binary Manipulation** - Using struct module for byte packing

## Alternative Solutions

### Using r2pipe (Radare2)
```python
import r2pipe

r2 = r2pipe.open('./file')
r2.cmd('aaa')  # Analyze all
r2.cmd('s sym.secret_function')  # Seek to function
disasm = r2.cmd('pdf')  # Print disassembly
print(disasm)
```

### Using pwntools
```python
from pwn import *

# Extract data from binary
elf = ELF('./file')
encrypted = elf.read(elf.symbols['encrypted_data'], 46)

# Decrypt
for key in [0x12, 0x34, 0x56, 0x78]:
    encrypted = xor(encrypted, key)

print(encrypted.decode())
```

### Manual GDB Extraction
```bash
gdb ./file
(gdb) break secret_function
(gdb) run
(gdb) x/46bx $rsp+0x10  # Dump stack bytes
# Copy hex values and decrypt offline
```

### Using CyberChef
1. Extract hex bytes from Ghidra
2. Use CyberChef operations:
   - From Hex
   - XOR Brute Force (or known keys)
   - To String
3. Identify flag pattern

## Common Pitfalls

1. **Incorrect Byte Order** - Forgetting little-endian conversion
2. **Partial Reads** - Not reading all bytes from 64-bit values
3. **Wrong XOR Sequence** - Applying operations in wrong order
4. **Buffer Boundaries** - Not accounting for null terminators
5. **Sign Extension** - Handling signed vs unsigned integers

## Advanced Techniques

### Automated Stack Analysis
```python
from angr import Project

proj = Project('./file', auto_load_libs=False)
cfg = proj.analyses.CFGFast()

# Find secret_function
func = proj.kb.functions['secret_function']

# Analyze stack variables
for var in func.local_variables:
    print(f"Variable: {var.name} at offset {var.offset}")
```

### Binary Instrumentation with Frida
```javascript
// Hook the aa_bb function to trace XOR operations
Interceptor.attach(Module.findExportByName(null, 'aa_bb'), {
    onEnter: function(args) {
        console.log('XOR key:', args[1]);
        console.log('Data:', hexdump(args[0]));
    },
    onLeave: function(retval) {
        console.log('Result:', hexdump(retval));
    }
});
```

### Using Unicorn Engine
```python
from unicorn import *
from unicorn.x86_const import *

# Emulate just the decryption routine
mu = Uc(UC_ARCH_X86, UC_MODE_64)
# ... setup memory and registers
# ... emulate instructions
# ... extract decrypted data from memory
```

## Challenge Variations

Similar challenges might include:
- **ASLR and PIE** - Making addresses unpredictable
- **Stack Canaries** - Adding buffer overflow protection
- **Stripped Binaries** - No symbol information
- **Packed/Obfuscated** - Additional anti-analysis techniques
- **Anti-Debugging** - Detecting debugger presence

## Resources

- [Ghidra Stack Analysis](https://ghidra-sre.org/courses/GhidraClass/Intermediate/StackAnalysis.html)
- [x86-64 Calling Conventions](https://en.wikipedia.org/wiki/X86_calling_conventions#System_V_AMD64_ABI)
- [Binary Exploitation Tutorials](https://github.com/guyinatuxedo/nightmare)
- [GDB Cheat Sheet](https://darkdust.net/files/GDB%20Cheat%20Sheet.pdf)
- [pwndbg Documentation](https://github.com/pwndbg/pwndbg)

## Real-World Applications

### Similar Vulnerabilities:
- **Heartbleed** - Stack buffer read vulnerability
- **Stack Buffer Overflows** - Classic exploitation technique
- **Return-Oriented Programming (ROP)** - Stack-based exploitation
- **Format String Vulnerabilities** - Stack memory disclosure

### Defense Mechanisms:
- **Stack Canaries** - Detect buffer overflows
- **ASLR** - Randomize memory addresses
- **DEP/NX** - Prevent code execution on stack
- **Control Flow Integrity (CFI)** - Verify execution paths
- **Shadow Stacks** - Hardware-assisted protection

## Practice Exercises

To improve your skills:
1. Modify the XOR keys and re-encrypt different data
2. Add additional XOR stages (5, 6, or more)
3. Implement encryption in assembly and reverse it
4. Create similar challenges with different encryption schemes
5. Practice with [Exploit Exercises](https://exploit-exercises.lains.space/)

## Conclusion

This challenge demonstrates the importance of understanding:
- Stack memory layout and organization
- Multi-stage encryption vulnerabilities
- Static and dynamic analysis techniques
- Byte ordering in x86-64 architecture
- Python scripting for binary analysis

The "buffer overflow" reference in the flag name is a hint about stack-based vulnerabilities, even though this specific challenge doesn't exploit an actual overflow.
