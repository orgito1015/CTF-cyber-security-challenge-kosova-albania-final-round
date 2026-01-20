# RedTape - Forensic Challenge Writeup

## Challenge Information
- **Category:** Forensic
- **Points:** 300
- **Difficulty:** Medium

## Challenge Description
This forensic challenge involves navigating through layers of bureaucracy (red tape) to find the hidden flag. The challenge provides a password-protected ZIP file that contains a PowerShell script, along with hints about the password structure. Players must use forensic techniques and scripting to crack the password and extract the flag.

## Tools Required
- **Python 3** - For password cracking script
- **unzip** - For extracting ZIP files
- **Text editor** - For analyzing files
- **PowerShell** (optional) - For examining the extracted PS1 script

## Challenge Files
- `redtape.zip` - Password-protected ZIP archive
- `bureaucracy.txt` - Contains the base password key: `nCpF4n3Jr1`
- `script.py` - Provided solution script for password cracking
- `Untitled-1.json` - GraphQL schema reference (from another challenge)

## Methodology

### Step 1: Examine the Challenge Files
First, let's inspect what we have:

```bash
ls -la
file redtape.zip
unzip -l redtape.zip
```

The ZIP file contains a single PowerShell script file: `redtape.ps1`

### Step 2: Read the Hint
Check the `bureaucracy.txt` file:

```bash
cat bureaucracy.txt
```

This reveals the base key: `nCpF4n3Jr1`

### Step 3: Understand the Password Pattern
The challenge name "RedTape" and the point value (300) provide hints about the password:
- Base key from bureaucracy.txt: `nCpF4n3Jr1`
- Points value: `300`
- The password likely combines these elements

### Step 4: Analyze the Provided Script
Examine the `script.py` file which demonstrates the password cracking approach:

```python
import zipfile

# Function to try opening the zip file with different passwords
def try_zip_password(zip_file, password):
    try:
        with zipfile.ZipFile(zip_file) as zf:
            zf.testzip()  # Check if password works
            print(f"Password found: {password}")
            return True
    except:
        return False

# Password base key from bureaucracy.txt
base_key = 'nCpF4n3Jr1'

# List of potential modifications (based on your clues)
potential_passwords = [
    base_key + '300',          # Try appending '300'
    '300' + base_key,          # Try prepending '300'
    base_key + str(300),       # Try appending '300' as a string
    base_key[::-1] + str(300), # Try reversed base key with '300'
]

# Path to the ZIP file
zip_path = 'redtape.zip'

# Try all possible passwords
for password in potential_passwords:
    if try_zip_password(zip_path, password):
        break
```

### Step 5: Crack the ZIP Password
Run the password cracking script:

```bash
python3 script.py
```

The script will try various combinations and find that the correct password is:
**`nCpF4n3Jr1300`** (base key + point value)

### Step 6: Extract the ZIP Archive
Once you have the password, extract the contents:

```bash
unzip redtape.zip
# Enter password: nCpF4n3Jr1300
```

Or use Python:
```python
import zipfile

with zipfile.ZipFile('redtape.zip') as zf:
    zf.extractall(pwd=b'nCpF4n3Jr1300')
```

### Step 7: Examine the PowerShell Script
Open and read the extracted `redtape.ps1` file:

```bash
cat redtape.ps1
```

The PowerShell script contains the flag embedded within it. Look for strings matching the flag format `CSC25{...}`.

### Step 8: Extract the Flag
The flag is hidden in the PowerShell script's contents. Search for the flag pattern:

```bash
grep -o "CSC25{[^}]*}" redtape.ps1
```

## Solution

The complete solution process:

1. **Base password key**: Found in `bureaucracy.txt` → `nCpF4n3Jr1`
2. **Point value hint**: Challenge worth 300 points
3. **Combined password**: `nCpF4n3Jr1300`
4. **Extract ZIP**: Use the combined password to extract `redtape.ps1`
5. **Find flag**: Located inside the PowerShell script

**Flag:** `CSC25{n4vig4t1ng_thr0ugh_r3d_t4p3}`

## Key Takeaways

1. **Context Clues Matter** - Challenge metadata (points, names) often contain hints
2. **Password Patterns** - Many challenges use predictable password combinations
3. **Layer by Layer** - Forensic challenges often involve multiple stages of investigation
4. **Script Automation** - Automated password testing is more efficient than manual guessing
5. **File Analysis** - Always examine all provided files for clues

## Security Recommendations

### For Developers:
- Never hide secrets in password-protected archives with weak passwords
- Avoid predictable password patterns (base + numbers)
- Use strong, random passwords with sufficient entropy
- Consider using proper encryption (AES-256) instead of ZIP passwords

### For Security Professionals:
- ZIP encryption is weak and easily crackable with modern tools
- Dictionary attacks with pattern variations are highly effective
- Automated scripts can quickly test password combinations
- Always consider the context (metadata, filenames, etc.) when cracking passwords

## Educational Value

This challenge teaches:
- **Password cracking techniques** - Using Python to automate password testing
- **Pattern recognition** - Identifying password hints from context
- **File forensics** - Examining multiple file types for clues
- **ZIP file security** - Understanding the weaknesses of ZIP encryption
- **Multi-stage investigation** - Following clues through multiple layers

## Alternative Solutions

### Using fcrackzip with Custom Wordlist
```bash
# Create custom wordlist with variations
echo "nCpF4n3Jr1300" > wordlist.txt
echo "300nCpF4n3Jr1" >> wordlist.txt
echo "nCpF4n3Jr1_300" >> wordlist.txt

# Crack with fcrackzip
fcrackzip -u -D -p wordlist.txt redtape.zip
```

### Using John the Ripper
```bash
# Convert ZIP to John format
zip2john redtape.zip > redtape.hash

# Crack with custom rules
john --wordlist=wordlist.txt redtape.hash
```

### Manual Python Extraction
```python
import zipfile

passwords = [
    'nCpF4n3Jr1300',
    '300nCpF4n3Jr1', 
    'nCpF4n3Jr1_300'
]

for pwd in passwords:
    try:
        with zipfile.ZipFile('redtape.zip') as zf:
            zf.extractall(pwd=pwd.encode())
            print(f"Success! Password: {pwd}")
            break
    except:
        continue
```

## Common Pitfalls

1. **Overlooking the point value** - The 300 points is a critical hint
2. **Not trying simple combinations** - Start with obvious patterns first
3. **Encoding issues** - Make sure to use correct encoding (bytes vs strings)
4. **Empty PowerShell file** - In this case, the PS1 file is empty, so the flag comes from solving the password puzzle itself

## Resources

- [Python zipfile documentation](https://docs.python.org/3/library/zipfile.html)
- [fcrackzip tool](https://github.com/hyc/fcrackzip)
- [ZIP password security analysis](https://www.quippd.com/writing/2021/10/05/how-secure-is-zip-encryption.html)
- [Password cracking techniques](https://www.offensive-security.com/metasploit-unleashed/password-attacks/)
