# Zipception - Forensic Challenge Writeup

## Challenge Information
- **Category:** Forensic
- **Points:** 500
- **Difficulty:** Hard

## Challenge Description
The flag is hidden inside a password-protected ZIP file. However, this is not a simple extraction challenge - it involves multiple layers of obfuscation. You'll need to crack the ZIP password using a wordlist, extract an image file, and then discover that the image contains a hidden nested ZIP file using steganography techniques. This multi-stage challenge requires combining password cracking, steganography, and persistence.

## Tools Required
- **fcrackzip** - ZIP password cracker
- **John the Ripper** with zip2john - Alternative password cracker
- **unzip** - For extracting ZIP files
- **binwalk** - Firmware analysis tool for finding embedded files
- **steghide** (optional) - For steganography analysis
- **rockyou.txt** - Common password wordlist (usually in `/usr/share/wordlists/`)
- **Image viewer** - To examine extracted images

## Methodology

### Step 1: Examine the Challenge Files
List the files in the challenge directory:
```bash
ls -la
file zipception.zip
```

Attempt to extract without password:
```bash
unzip zipception.zip
```

This will fail with "incorrect password" error, confirming we need to crack it.

### Step 2: Prepare the Password Wordlist
The challenge hints that we should use `rockyou.txt`, a popular wordlist containing common passwords:

```bash
# Locate rockyou.txt (common locations)
locate rockyou.txt

# Common locations:
# /usr/share/wordlists/rockyou.txt (Kali Linux)
# /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt

# If compressed, decompress it:
gunzip /usr/share/wordlists/rockyou.txt.gz
```

### Step 3: Crack the ZIP Password Using fcrackzip
Use `fcrackzip` to brute force the password:

```bash
fcrackzip -u -v -D -p /usr/share/wordlists/rockyou.txt zipception.zip
```

**Parameter Explanation:**
- `-u`: Use unzip to verify password (more reliable)
- `-v`: Verbose mode (show progress)
- `-D`: Dictionary attack mode
- `-p`: Path to the password wordlist

**Expected Output:**
```
found file 'flag.jpg', (size cp/uc 164023/246426, flags 1, chk 7ec4)

PASSWORD FOUND!!!!: pw == mitch123
```

The password is: `mitch123`

#### Alternative Method: John the Ripper
```bash
# Extract hash from ZIP
zip2john zipception.zip > hash.txt

# Crack using John
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Show the cracked password
john --show hash.txt
```

### Step 4: Extract the ZIP File
Use the discovered password to extract the contents:
```bash
unzip zipception.zip
# When prompted, enter password: mitch123
```

Or directly with password:
```bash
unzip -P mitch123 zipception.zip
```

This extracts `flag.jpg`.

### Step 5: Examine the Image File
View the extracted image:
```bash
# View file information
file flag.jpg
ls -lh flag.jpg

# Open the image
xdg-open flag.jpg    # Linux
open flag.jpg        # macOS
```

The image displays text that appears to be a password: `cyb3rsecCh4llenge25!`

**This looks like the flag, but it's not in the correct format (CSC25{...})!** This is a red herring - there's more to discover.

### Step 6: Analyze the Image for Hidden Data
Since the displayed password isn't the flag, there must be hidden data. Use `binwalk` to search for embedded files:

```bash
binwalk flag.jpg
```

**Expected Output:**
```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.01
246426        0x3C33A         Zip archive data, at least v2.0 to extract
```

**Analysis:** There's a ZIP archive embedded at byte offset 246426 within the JPEG file!

### Step 7: Extract the Nested ZIP File
Use `binwalk` to automatically extract embedded files:

```bash
binwalk -e flag.jpg
```

Or extract manually using `dd`:
```bash
dd if=flag.jpg bs=1 skip=246426 of=hidden.zip
```

**Parameter Explanation:**
- `-e`: Extract embedded files automatically
- Binwalk creates a directory: `_flag.jpg.extracted/`

List the extracted contents:
```bash
ls -la _flag.jpg.extracted/
```

You'll find a ZIP file (e.g., `3C33A.zip` or similar).

### Step 8: Extract the Final ZIP File
This nested ZIP is also password protected. Use the password we found in the image:

```bash
cd _flag.jpg.extracted/
unzip -P "cyb3rsecCh4llenge25!" 3C33A.zip
```

Or try to crack it:
```bash
fcrackzip -u -v -D -p /usr/share/wordlists/rockyou.txt 3C33A.zip
```

The password displayed in the image (`cyb3rsecCh4llenge25!`) is the key to the nested ZIP.

### Step 9: Extract the Flag
Once extracted, you'll find a text file containing the actual flag:
```bash
cat flag.txt
# OR
ls -la
cat <flag-file-name>
```

## Solution
This challenge requires multiple steps:
1. Crack the initial ZIP password using rockyou.txt wordlist → Password: `mitch123`
2. Extract `flag.jpg` from the ZIP
3. Discover the image contains a nested ZIP using binwalk
4. Extract the embedded ZIP from the image using binwalk
5. Use the password shown in the image (`cyb3rsecCh4llenge25!`) to extract the nested ZIP
6. Retrieve the final flag from the extracted files

**Flag:** `CSC25{p4ssw0rd_1n_Th3_1m4ge}`

The flag name hints at the solution - the password to the nested archive is literally "in the image" (displayed visually in flag.jpg).

## Why This Vulnerability Exists

### Multi-Layered Attack Simulation
This challenge simulates real-world scenarios where attackers use multiple obfuscation techniques:

1. **Weak Passwords**: Using common passwords that appear in breach databases
2. **Steganography**: Hiding data within seemingly innocent files (images, documents)
3. **Defense in Depth (Attacker Perspective)**: Multiple layers of protection/obfuscation
4. **Social Engineering**: Displaying a fake flag to discourage further investigation

### Security Concepts Demonstrated

#### Password Security
- **Weak Password Choices**: "mitch123" is a common pattern (name + numbers)
- **Password Reuse**: The second password is visible but serves a different purpose
- **Brute Force Vulnerability**: Common passwords can be cracked in seconds

#### Steganography
- **Hidden in Plain Sight**: Data can be embedded in images without visible changes
- **File Structure Exploitation**: ZIP archives and other files can be appended to images
- **Detection Challenges**: Standard viewers don't reveal hidden content

#### Multi-Stage Attacks
- **Layered Defenses**: Attackers use multiple techniques to hide malicious payloads
- **Forensic Evasion**: Each layer makes detection and analysis more difficult
- **Persistence**: Even if one layer is discovered, others remain hidden

## Key Takeaways
1. **Password strength matters** - Weak passwords can be cracked in seconds with wordlists
2. **Files can contain hidden data** - Always check for steganography and embedded files
3. **Visual information can be misleading** - The displayed "flag" was a decoy
4. **Multiple analysis tools needed** - Different tools reveal different aspects of files
5. **Persistence is key** - Don't stop at the first apparent solution
6. **Real-world relevance** - Malware often uses similar multi-stage obfuscation

## Security Recommendations

### Password Security Best Practices

#### For Users
- **Use strong, unique passwords** - Minimum 12+ characters with complexity
- **Password Managers** - Generate and store unique passwords for each service
  - BitWarden, 1Password, LastPass, KeePass
- **Passphrases** - Use multiple random words: `correct-horse-battery-staple`
- **Multi-Factor Authentication (MFA)** - Add additional security layers
- **Avoid common patterns** - No dictionary words, names, or simple number patterns

#### For Developers/Administrators
- **Enforce password policies**:
  ```
  - Minimum length: 12-16 characters
  - Complexity requirements
  - Password history (prevent reuse)
  - Regular password rotation for sensitive accounts
  ```
- **Rate limiting** - Prevent brute force attacks
- **Account lockout policies** - Lock after N failed attempts
- **Password breach monitoring** - Check against known compromised passwords
- **Encryption at rest** - Use bcrypt, Argon2, or scrypt for password hashing

### Steganography Detection and Prevention

#### Detection Tools
- **binwalk**: Finds embedded files in images and other formats
- **steghide**: Detects and extracts hidden data
- **stegdetect**: Automated steganography detection
- **exiftool**: Examines file metadata for anomalies
- **foremost**: File carving tool for recovering hidden files

#### Analysis Techniques
```bash
# Check file signatures
file suspicious.jpg

# Look for embedded files
binwalk suspicious.jpg

# Extract all embedded data
binwalk -e suspicious.jpg

# Check file entropy (high entropy may indicate encryption/compression)
binwalk -E suspicious.jpg

# Examine metadata
exiftool suspicious.jpg

# Check file size anomalies
ls -lh suspicious.jpg
# Compare to typical file sizes for that type
```

#### Prevention Measures
- **File validation**: Verify files match expected formats and sizes
- **Content filtering**: Scan for embedded files at network boundaries
- **Integrity checking**: Use hashes to detect file modifications
- **DLP solutions**: Monitor for data exfiltration via steganography
- **Network monitoring**: Detect unusual file transfers

### ZIP Security

#### Password Protection Alternatives
- **Don't rely on ZIP passwords** - They're weak and easily crackable
- **Use proper encryption**:
  - GPG/PGP for file encryption
  - 7-Zip with AES-256 encryption (stronger than ZIP)
  - VeraCrypt for container-based encryption
- **Secure file transfer**: Use SFTP, SCP, or encrypted cloud storage
- **End-to-end encryption**: For sensitive communications

#### Password Cracking Protection
- **Key derivation functions**: If you must use password protection, use strong KDFs
- **Detect brute force attempts**: Monitor for repeated failed extractions
- **Physical security**: For highly sensitive data, rely on physical security controls

### Forensic Analysis Best Practices

#### File Analysis Workflow
1. **Initial triage**: Identify file type and basic properties
2. **Static analysis**: Examine without executing/extracting
3. **Signature verification**: Check for known malicious signatures
4. **Metadata extraction**: Review EXIF, file headers, timestamps
5. **Content analysis**: Use binwalk, strings, entropy analysis
6. **Controlled extraction**: Extract in isolated environment
7. **Documentation**: Record all findings and analysis steps

#### Tools for Comprehensive Analysis
- **File identification**: `file`, `TrID`
- **Metadata**: `exiftool`, `pdfinfo`
- **Embedded files**: `binwalk`, `foremost`, `scalpel`
- **Strings extraction**: `strings`, `floss`
- **Hex editor**: `xxd`, `hexdump`, `HxD`
- **Entropy analysis**: `binwalk -E`, `ent`
- **Password cracking**: `john`, `hashcat`, `fcrackzip`

### Organizational Security Measures

#### Email and File Sharing Security
- **Attachment scanning**: Scan all attachments for embedded files
- **File type restrictions**: Block or quarantine suspicious file types
- **Sandboxing**: Execute/extract files in isolated environments
- **User training**: Educate about steganography and hidden data risks

#### Data Loss Prevention (DLP)
- **Monitor outbound files**: Scan for embedded or encrypted data
- **Content inspection**: Deep packet inspection for hidden data channels
- **Policy enforcement**: Prevent unauthorized file types or encryption

#### Incident Response
When steganography or hidden files are discovered:
1. **Isolate affected systems**
2. **Preserve evidence** (forensic imaging)
3. **Analyze the payload** (what was hidden and why)
4. **Identify the source** (who embedded the data)
5. **Assess impact** (was data exfiltrated?)
6. **Remediate and improve** defenses

## Educational Value
This challenge teaches:
- **Password cracking techniques** - Using wordlists and dictionary attacks
- **Steganography detection** - Finding hidden data in files
- **File analysis fundamentals** - Understanding file structures and signatures
- **Multi-stage problem solving** - Recognizing when the obvious answer is wrong
- **Tool proficiency** - Using binwalk, fcrackzip, and other forensic tools
- **Persistence and thoroughness** - Not accepting the first apparent solution
- **Real-world attack simulation** - Multiple obfuscation layers like actual malware

### Real-World Applications

#### Malware Analysis
Attackers commonly use similar techniques:
- Password-protected archives to evade antivirus
- Steganography to hide C2 server information
- Multiple stages to complicate analysis
- Decoy data to mislead security researchers

#### Data Exfiltration
Insiders and attackers hide stolen data using:
- Embedded files in images or documents
- Password-protected archives sent through email
- Steganography to bypass DLP solutions

#### Digital Forensics
Investigators must:
- Crack passwords on evidence files
- Search for hidden data in seized devices
- Analyze complex, multi-layered obfuscation
- Document the complete extraction process for court

This challenge provides hands-on experience with techniques and tools used daily by forensic analysts, malware researchers, and incident responders.
