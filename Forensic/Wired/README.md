# Wired - Forensic Challenge Writeup

## Challenge Information
- **Category:** Forensic
- **Points:** 300
- **Difficulty:** Medium

## Challenge Description
Network traffic has been captured during a potential security incident. Your task is to analyze the packet capture (PCAP) file to identify malicious activity. The challenge involves examining network protocols, extracting artifacts, and analyzing a PowerShell automation script that uses encoding techniques to hide a command and control (C2) server URL.

The hint suggests to "check the wire first and then automate it" - indicating we need to analyze the network capture and then examine an automation script.

## Tools Required
- **Wireshark** - Network protocol analyzer for examining PCAP files
- **tshark** - Command-line network protocol analyzer (Wireshark CLI)
- **PowerShell** or **pwsh** - To understand and execute PowerShell scripts
- **Python** or **CyberChef** - For Base64 decoding (optional)
- **Text Editor** - For examining the PowerShell script

## Methodology

### Step 1: Examine the Provided Files
List the files in the challenge directory:
```bash
ls -la
```

You should find:
- `wired_capture.pcapng` - Network capture file
- `automation.ps1` - PowerShell automation script

### Step 2: Analyze the Network Capture
Open the PCAP file in Wireshark:
```bash
wireshark wired_capture.pcapng
```

Or use tshark for command-line analysis:
```bash
tshark -r wired_capture.pcapng
```

#### Look for Suspicious Traffic Patterns
- HTTP/HTTPS traffic to unusual domains
- DNS queries for suspicious domains
- Large data transfers
- Encoded data in network requests
- PowerShell-related network activity

#### Extract HTTP Objects
In Wireshark:
1. Go to `File` → `Export Objects` → `HTTP`
2. Look for downloaded scripts, executables, or suspicious files

#### Check DNS Queries
Filter for DNS traffic:
```bash
tshark -r wired_capture.pcapng -Y "dns" -T fields -e dns.qry.name
```

#### Examine PowerShell Activity
Look for:
- PowerShell user agents
- Base64 encoded commands
- Script downloads
- Suspicious URLs

### Step 3: Analyze the PowerShell Script
Open and examine `automation.ps1`:
```bash
cat automation.ps1
```

You'll find a PowerShell script with Base64 encoding logic:
```powershell
$X1 = "aHR0cHM6Ly9wYXN0ZWJpbi5jb20v"; $X2 = "amIyVU5hNko="
$U = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($X1 + $X2))
```

**Analysis of the Script:**
- Two variables `$X1` and `$X2` contain Base64 encoded strings
- They are concatenated and then decoded
- This is a common obfuscation technique to hide C2 server URLs

### Step 4: Decode the Base64 Strings
The script concatenates two Base64 strings before decoding them. This technique splits the malicious URL to evade simple string-based detection.

#### Method 1: Using PowerShell
```powershell
$X1 = "aHR0cHM6Ly9wYXN0ZWJpbi5jb20v"
$X2 = "amIyVU5hNko="
$Combined = $X1 + $X2
$Decoded = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($Combined))
Write-Host $Decoded
```

#### Method 2: Using Command Line (Linux/Mac)
```bash
echo "aHR0cHM6Ly9wYXN0ZWJpbi5jb20vamIyVU5hNko=" | base64 -d
```

#### Method 3: Using Python
```python
import base64
encoded = "aHR0cHM6Ly9wYXN0ZWJpbi5jb20vamIyVU5hNko="
decoded = base64.b64decode(encoded).decode('utf-8')
print(decoded)
```

#### Method 4: Using CyberChef
1. Go to https://gchq.github.io/CyberChef/
2. Add the "From Base64" operation
3. Input: `aHR0cHM6Ly9wYXN0ZWJpbi5jb20vamIyVU5hNko=`
4. Get the decoded URL

### Step 5: Decode the Obfuscated URL
Concatenating the two Base64 strings:
```
aHR0cHM6Ly9wYXN0ZWJpbi5jb20v + amIyVU5hNko= = aHR0cHM6Ly9wYXN0ZWJpbi5jb20vamIyVU5hNko=
```

Decoding the combined Base64 string reveals:
```
https://pastebin.com/jb2UNa6J
```

### Step 6: Access the Decoded URL
Visit the decoded URL to retrieve the flag:
```bash
curl https://pastebin.com/jb2UNa6J
# OR
curl https://pastebin.com/raw/jb2UNa6J
```

Alternatively, open it in a web browser.

### Step 7: Extract the Flag
The Pastebin page contains the flag.

## Solution
By analyzing the network capture and examining the PowerShell automation script, we discover that the attacker used Base64 encoding to obfuscate a Pastebin URL. Decoding the concatenated Base64 strings reveals the C2 server location, which contains the flag.

**Flag:** `CSC25{H1dd3n_iN_tH3_W1r3}`

The flag name "Hidden in the Wire" refers to the malicious payload hidden in network traffic and obfuscated within the PowerShell script.

## Why This Vulnerability Exists

### Attack Technique: Living Off The Land (LOtL)
1. **PowerShell Abuse**: Attackers use legitimate Windows tools to avoid detection
2. **Obfuscation**: Base64 encoding hides malicious URLs from simple pattern matching
3. **String Splitting**: Breaking the URL into parts bypasses signature-based detection
4. **Command and Control**: Using public services like Pastebin for C2 communication

### Why This Works
- **Legitimate Tool**: PowerShell is a trusted Windows component
- **Encoded Content**: Base64 encoding bypasses keyword-based security tools
- **Public Services**: Pastebin is generally allowed through firewalls
- **Script-Based Attacks**: May not trigger antivirus designed for executables

### Detection Challenges
- PowerShell is widely used for legitimate administration
- Base64 encoding is common in legitimate scripts
- Pastebin and similar services are used by developers
- Network traffic may be encrypted (HTTPS)

## Key Takeaways
1. **Network forensics is crucial** - PCAP analysis reveals attack patterns and C2 infrastructure
2. **Obfuscation is common** - Attackers use encoding to hide malicious intent
3. **PowerShell is a double-edged sword** - Powerful for admins but also for attackers
4. **Context matters** - Understanding script behavior requires deobfuscation
5. **Multiple analysis layers needed** - Network + script analysis provides complete picture

## Security Recommendations

### Detection & Monitoring

#### PowerShell Logging
Enable comprehensive PowerShell logging:
```powershell
# Module Logging
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging" -Name "EnableModuleLogging" -Value 1

# Script Block Logging
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -Name "EnableScriptBlockLogging" -Value 1

# Transcription (captures all PowerShell session activity)
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription" -Name "EnableTranscripting" -Value 1
```

#### Network Monitoring
- **SSL/TLS Inspection**: Decrypt and inspect HTTPS traffic (with proper legal authorization)
- **DNS Monitoring**: Track DNS queries to detect C2 communication
- **Anomaly Detection**: Identify unusual PowerShell network activity
- **Indicators of Compromise (IOCs)**: Monitor for known malicious domains/IPs

#### Endpoint Detection and Response (EDR)
- Deploy EDR solutions that detect:
  - Base64 encoded PowerShell commands
  - Outbound connections from PowerShell
  - Suspicious script execution patterns
  - Living Off The Land Binary (LOLBin) abuse

### Prevention

#### PowerShell Hardening
- **Constrained Language Mode**: Restrict PowerShell functionality
  ```powershell
  $ExecutionContext.SessionState.LanguageMode = "ConstrainedLanguage"
  ```
- **Application Allowlisting**: Use AppLocker or Windows Defender Application Control
- **Just Enough Administration (JEA)**: Limit PowerShell capabilities per user
- **Execution Policy**: Set restrictive execution policies
  ```powershell
  Set-ExecutionPolicy AllSigned -Scope LocalMachine
  ```

#### Network Security
- **Firewall Rules**: Block unnecessary outbound PowerShell connections
- **Proxy Configuration**: Route PowerShell traffic through proxy for inspection
- **DNS Filtering**: Block access to known malicious domains and pastebins (if not needed)
- **Network Segmentation**: Limit lateral movement opportunities

#### User Training
- Educate users about phishing and malicious scripts
- Train administrators on secure PowerShell practices
- Implement principle of least privilege

### Investigation and Response

#### When Suspicious PowerShell Activity is Detected
1. **Isolate the system** - Disconnect from network to prevent further C2 communication
2. **Capture memory** - Use tools like WinPMEM, FTK Imager for forensic analysis
3. **Collect PowerShell logs** - Event IDs 4103, 4104, 4105, 4106
4. **Extract network artifacts** - Review DNS cache, network connections
5. **Analyze script content** - Deobfuscate and understand attacker intent

#### Forensic Analysis Tools
- **PowerShell**: Built-in Get-WinEvent for log analysis
- **DeepBlueCLI**: PowerShell module for security log analysis
- **KAPE**: Collects forensic artifacts including PowerShell logs
- **Sysmon**: Enhanced logging for PowerShell and process activity

### Deobfuscation Techniques

#### Common PowerShell Obfuscation Methods
1. **Base64 Encoding**: `-EncodedCommand` or `[Convert]::FromBase64String()`
2. **String Concatenation**: Breaking strings into parts
3. **Character Substitution**: Using ASCII codes or escape sequences
4. **Compression**: Using GZIP or Deflate
5. **Invoke-Expression (IEX)**: Dynamic code execution

#### Deobfuscation Tools
- **PSDecode**: PowerShell deobfuscation tool
- **PowerDecode**: Automated PowerShell deobfuscator
- **Revoke-Obfuscation**: Framework for detecting obfuscated PowerShell
- **CyberChef**: Web-based tool for decoding various formats

## Educational Value
This challenge teaches:
- **Network forensics** - Analyzing PCAP files to identify malicious activity
- **PowerShell analysis** - Understanding script behavior and obfuscation techniques
- **Base64 decoding** - A fundamental skill for analyzing encoded data
- **Attack patterns** - Recognizing Living Off The Land techniques
- **Multi-stage analysis** - Combining network and host-based forensics
- **Real-world attacker TTPs** - Understanding how adversaries hide C2 infrastructure

### MITRE ATT&CK Techniques Demonstrated
- **T1059.001**: Command and Scripting Interpreter: PowerShell
- **T1027**: Obfuscated Files or Information
- **T1132.001**: Data Encoding: Standard Encoding (Base64)
- **T1071.001**: Application Layer Protocol: Web Protocols
- **T1102**: Web Service (Pastebin for C2)

This challenge simulates a realistic scenario where attackers use PowerShell and encoding to establish command and control communication, requiring defenders to analyze both network traffic and malicious scripts to understand the full attack chain.
