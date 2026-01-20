# Maze - Cryptography Challenge Writeup

## Challenge Information
- **Category:** Cryptography
- **Points:** 150
- **Difficulty:** Easy

## Challenge Description
"The truth lies not in victory, but in the echoes left behind."

This is a web-based JavaScript maze game that serves as a red herring. The real flag isn't obtained by solving the maze, but by examining the source code files.

## Tools Required
- **Text Editor** or **IDE** - For viewing source files
- **grep** - For searching through files
- **Web Browser** (optional) - To see the actual maze game

## Methodology

### Step 1: Understand the Challenge Structure
The challenge provides a Docker setup with a web application:

```
maze/
├── Dockerfile
├── docker-compose.yml
└── src/
    ├── index.html    # Main HTML file
    ├── maze.js       # Obfuscated JavaScript (potential decoy)
    └── mazee         # File without extension (suspicious!)
```

### Step 2: Analyze the Hint
"The truth lies not in victory, but in the echoes left behind."

This hint suggests:
- **Not in victory** - Don't need to solve the maze
- **Echoes left behind** - Look at residual files, source code, or artifacts

### Step 3: Investigate Unusual Files
Notice the file `mazee` (no extension). This is suspicious because:
- Most web files have extensions (.html, .js, .css)
- The name is similar to "maze" but different
- It might contain leftover development code

### Step 4: Search for the Flag
Use grep to search all files for the flag format:

```bash
cd maze/src/
grep -r "CSC25" .
```

Output:
```
./mazee:  const flg = CSC25{s1mple_js_ObfusCation};
```

### Step 5: Examine the mazee File
Open the `mazee` file and search for the flag:

```bash
cat mazee | grep -A 2 -B 2 "CSC25"
```

Or open in a text editor and search for "CSC25" or "flag":

```javascript
// Around line 89 in the mazee file
function secret() {
  const flg = CSC25{s1mple_js_ObfusCation};
}
```

The flag is stored in a secret function that's never called!

## Solution
The flag is found in the `mazee` file (without extension) as a hardcoded constant in an unused function.

**Flag:** `CSC25{s1mple_js_ObfusCation}`

## Understanding the Challenge

### The Decoy:
- `maze.js` - 95KB obfuscated JavaScript file (looks intimidating!)
- `index.html` - Functional maze game (works but irrelevant)
- The maze can be played, but winning doesn't reveal the flag

### The Real Target:
- `mazee` - Unextensioned file with plaintext JavaScript
- Contains development code with hardcoded flag
- Likely a leftover file forgotten during production deployment

### The Lesson:
The challenge name "maze" has double meaning:
1. Literal maze game (the distraction)
2. Maze of files to navigate (the actual challenge)

## Why This Happens in Real Life

### Common Scenarios:
1. **Backup Files** - Developers leave .bak, .old, .tmp files
2. **Development Files** - Debug versions with test credentials
3. **Git Repositories** - .git folder exposed on production servers
4. **Source Maps** - .map files revealing original source code
5. **IDE Files** - .swp, .swo, *~ files with sensitive data

### Real-World Examples:
- `.DS_Store` files revealing directory structure
- `config.php.bak` with database passwords
- `.env` files with API keys
- `test.php` with hardcoded admin credentials

## How to Find These Files

### Manual Techniques:
```bash
# Find files without extensions
find . -type f ! -name "*.*"

# Search for backup patterns
find . -name "*.bak" -o -name "*.old" -o -name "*.tmp"

# Search for common patterns
grep -r "password\|secret\|key\|token\|flag" .
```

### Automated Tools:
- **DirBuster/GoBuster** - Directory brute-forcing
- **Nikto** - Web server scanner
- **OWASP ZAP** - Web application security scanner
- **Burp Suite** - Comprehensive web testing platform

## Key Takeaways
1. **Always search all files** - Don't assume you need to interact with the application
2. **Unusual file names are suspicious** - Files without extensions, odd naming
3. **Read the hints carefully** - "Echoes left behind" = leftover files
4. **Obfuscation is a distraction** - Large obfuscated files may be red herrings
5. **Source code review is critical** - Many flags are in plain sight

## Security Recommendations

### For Developers:
1. **Clean up before deployment**
   ```bash
   # Remove development files
   rm -f *.bak *.old *.tmp *~
   find . -name "*.swp" -delete
   ```

2. **Use .gitignore properly**
   ```
   # .gitignore
   *.bak
   *.old
   *.tmp
   *~
   .env
   .env.local
   config.local.*
   ```

3. **Disable directory listings**
   ```apache
   # Apache
   Options -Indexes
   ```

4. **Implement proper file permissions**
   ```bash
   # Only serve files that should be public
   chmod 644 public_files
   chmod 600 config_files
   ```

5. **Use build processes**
   - Separate development and production code
   - Use CI/CD pipelines that clean builds
   - Only deploy necessary files

### Security Checklist:
- [ ] No backup files in web root
- [ ] No .git directory accessible
- [ ] No debug/development endpoints enabled
- [ ] No hardcoded credentials in any file
- [ ] Directory listing disabled
- [ ] Source maps removed for production
- [ ] All sensitive config files protected
- [ ] File extensions properly configured

## Educational Value
This challenge teaches:
- The importance of thorough reconnaissance
- How leftover files create vulnerabilities
- Why file management is crucial for security
- Difference between obfuscation and hiding
- How to search and analyze source code effectively

## Similar CTF Techniques
- **Robots.txt Analysis** - Finding hidden directories
- **.git Directory Exploration** - Extracting commit history
- **Source Map Analysis** - Recovering original JavaScript
- **Backup File Discovery** - Finding .bak, .old files
- **Comment Mining** - Extracting info from HTML/JS comments
