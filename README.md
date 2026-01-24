# CTF CyberSecurity Challenge Kosova & Albania Final Round

This repository contains challenges from the CTF Cyber Security Challenge Kosovo-Albania Final Round.

** Complete writeups are now available for all challenges!** Each challenge directory contains a comprehensive `README.md` file with detailed solutions, methodologies, and educational content.

## Categories

- [Android](#android) - 2 challenges
- [Cryptography](#cryptography) - 5 challenges
- [Forensics](#forensics) - 9 challenges
- [Reverse Engineering](#reverse-engineering) - 2 challenges
- [Web](#web) - 1 challenge

**Total: 19 challenges**

---

## Android

Mobile application security and reverse engineering challenges.

| Challenge | Points | Description | Writeup |
|-----------|--------|-------------|---------|
| [Posta-beta](./Android/Posta-beta) | 400 | Android APK reverse engineering with XOR encryption | [📖 Writeup](./Android/Posta-beta/README.md) |
| [Posta-dev](./Android/Posta-dev) | 200 | Finding hardcoded credentials in Android source | [📖 Writeup](./Android/Posta-dev/README.md) |

---

## Cryptography

Custom encryption schemes and cipher challenges.

| Challenge | Points | Description | Writeup |
|-----------|--------|-------------|---------|
| [Caesar_Again](./Cryptography/Caesar_Again) | 150 | Caesar cipher with obfuscated code | [📖 Writeup](./Cryptography/Caesar_Again/README.md) |
| [CryptoDome](./Cryptography/CryptoDome) | 300 | Custom "lasagna" encryption with XOR and bit rotation | [📖 Writeup](./Cryptography/CryptoDome/README.md) |
| [Illusion](./Cryptography/Illusion) | 300 | Multi-layer encryption (mist, fog, haze) | [📖 Writeup](./Cryptography/Illusion/README.md) |
| [XORShifter](./Cryptography/XORShifter) | 150 | XOR encryption with predictable random seed | [📖 Writeup](./Cryptography/XORShifter/README.md) |
| [maze](./Cryptography/maze) | 150 | JavaScript obfuscation and hidden files | [📖 Writeup](./Cryptography/maze/README.md) |

---

## Forensics

Digital forensics, log analysis, and data recovery challenges.

| Challenge | Points | Description | Writeup |
|-----------|--------|-------------|---------|
| [Dangerous_Events](./Forensic/Dangerous_Events) | 300 | Windows Event Log analysis - malicious PowerShell | [📖 Writeup](./Forensic/Dangerous_Events/README.md) |
| [Discord Bot](./Forensic/Discord%20Bot) | 150 | Git history forensics and secret recovery | [📖 Writeup](./Forensic/Discord%20Bot/README.md) |
| [Ghostrider](./Forensic/Ghostrider) | 300 | Apache log analysis - finding malicious beacons | [📖 Writeup](./Forensic/Ghostrider/README.md) |
| [Unclear](./Forensic/Unclear) | 100 | Binary to ASCII conversion steganography | [📖 Writeup](./Forensic/Unclear/README.md) |
| [Wired](./Forensic/Wired) | 300 | Network packet capture (PCAP) analysis | [📖 Writeup](./Forensic/Wired/README.md) |
| [Zipception](./Forensic/Zipception) | 500 | Nested ZIP files with password cracking | [📖 Writeup](./Forensic/Zipception/README.md) |
| [usb_keystrokes](./Forensic/usb_keystrokes) | 400 | USB HID keylogger data extraction | [📖 Writeup](./Forensic/usb_keystrokes/README.md) |
| [ransomware](./Forensic/ransomware) | N/A | Ransomware binary analysis and decryption | [📖 Writeup](./Forensic/ransomware/README.md) |
| [RedTape](<./Forensic/RedTape%20(1)>) | N/A | ZIP password cracking challenge | [📖 Writeup](<./Forensic/RedTape%20(1)/RedTape/README.md>) |

---

## Reverse Engineering

Binary analysis and reverse engineering challenges.

| Challenge | Points | Description | Writeup |
|-----------|--------|-------------|---------|
| [Secret](./Reverse%20Enginner/Secret) | 300 | Binary analysis with Ghidra - XOR decryption | [📖 Writeup](./Reverse%20Enginner/Secret/README.md) |
| [Stack Attack](./Reverse%20Enginner/Stack%20Attack) | 450 | Stack buffer analysis with complex XOR chains | [📖 Writeup](./Reverse%20Enginner/Stack%20Attack/README.md) |

---

## Web

Web application security and exploitation challenges.

| Challenge | Points | Description | Writeup |
|-----------|--------|-------------|---------|
| [graphql](./Web/graphql) | 300 | GraphQL introspection and hidden endpoints | [📖 Writeup](./Web/graphql/README.md) |

---

## 📖 About the Writeups

Each `README.md` file contains:

- **Challenge Information** - Category, points, and difficulty level
- **Challenge Description** - What the challenge is about
- **Tools Required** - Software and tools needed to solve
- **Detailed Methodology** - Step-by-step solution process with commands
- **Solution** - Complete working solution with the flag
- **Security Analysis** - Why the vulnerability exists and how it works
- **Key Takeaways** - Important lessons learned
- **Defense Recommendations** - How to prevent similar issues
- **Educational Value** - Skills and concepts taught by the challenge

##  Difficulty Levels

- **Easy** (100-150 points) - Basic concepts, straightforward solutions
- **Medium** (200-300 points) - Moderate complexity, multiple steps
- **Hard** (400-500 points) - Advanced techniques, complex analysis

##  Common Tools Used

### Analysis Tools
- **Wireshark/tshark** - Network analysis
- **Ghidra** - Binary reverse engineering
- **JADX** - Android APK decompilation
- **binwalk** - Firmware and file analysis

### Forensics Tools
- **strings** - Extract text from binaries
- **file** - Identify file types
- **xxd/hexdump** - Hexadecimal viewers
- **fcrackzip** - ZIP password cracking

### Cryptography Tools
- **Python 3** - Scripting and decryption
- **CyberChef** - Swiss Army knife for encoding
- **base64** - Encoding/decoding utilities

### Web Tools
- **curl** - HTTP client
- **Burp Suite** - Web proxy
- **Browser DevTools** - Inspect and debug

## 📚 Learning Resources

### For Beginners
- Start with Easy challenges (100-150 points)
- Read the writeups even after solving
- Practice with similar CTF platforms

### For Intermediate
- Focus on understanding the "why" not just the "how"
- Try solving before reading writeups
- Experiment with different tools

### For Advanced
- Compare your solution with the writeup
- Look for alternative approaches
- Contribute improvements or optimizations

##  Security Skills Covered

- **Mobile Security** - Android reverse engineering, APK analysis
- **Cryptanalysis** - Breaking custom crypto, understanding ciphers
- **Digital Forensics** - Log analysis, data recovery, PCAP analysis
- **Reverse Engineering** - Binary analysis, decompilation, debugging
- **Web Security** - API exploitation, GraphQL attacks
- **Network Forensics** - Packet analysis, protocol understanding

##  Educational Use

These writeups are intended for:
- **CTF Players** - Learning new techniques and approaches
- **Security Students** - Understanding real-world security concepts
- **Educators** - Teaching materials for cybersecurity courses
- **Researchers** - Reference implementations and methodologies

##  Contributing

If you find errors or have improvements:
1. Open an issue describing the problem
2. Submit a pull request with fixes
3. Suggest alternative solutions or approaches

##  Ethical Use

These challenges and writeups are for educational purposes only. Always:
- Practice ethical hacking principles
- Only test on systems you own or have permission to test
- Respect privacy and legal boundaries
- Use knowledge for defense, not attack

##  License

Educational use only. Respect the original CTF organization's terms.

---

**🏆 Happy Hacking and Learning!**

*"The best way to learn is by doing, and the second best way is by reading how others did it."*
