# Ghostrider - Forensic Challenge Writeup

## Challenge Information
- **Category:** Forensic
- **Points:** 300
- **Difficulty:** Medium

## Challenge Description
A company has been compromised by an Advanced Persistent Threat (APT). The blue team has provided Apache web server logs for analysis. Your mission is to identify the malicious request where the attackers dropped a beacon. The hint suggests focusing on GET requests in the log file.

## Tools Required
- **Text Editor** or **grep** - For searching through log files
- **curl** or **wget** - For accessing URLs found in logs
- **Web Browser** - To view external resources

## Methodology

### Step 1: Examine the Apache Log File
Apache logs follow a standard format called the Common Log Format (CLF):
```
IP_ADDRESS - - [TIMESTAMP] "REQUEST_METHOD /PATH HTTP_VERSION" STATUS_CODE RESPONSE_SIZE
```

Begin by examining the log file structure:
```bash
head apache.log
wc -l apache.log
```

### Step 2: Filter for GET Requests
Since the hint mentions GET requests, we should focus on those specifically. GET requests are commonly used to retrieve resources and can be used to download malicious payloads:

```bash
grep "GET" apache.log | less
```

### Step 3: Look for Suspicious Patterns
When analyzing web logs for compromise indicators, look for:
- Unusual URLs or paths
- URL shortener services (tinyurl, bit.ly, etc.)
- Suspicious user agents
- Requests to external domains (which shouldn't normally appear in GET paths)
- Large response sizes for simple GET requests

### Step 4: Identify the Malicious Request
Navigate to line 820 of the log file:
```bash
sed -n '820p' apache.log
```

Or use line numbers in your text editor to jump directly to line 820.

You'll find the following suspicious entry:
```
4.53.215.180 - - [26/Mar/2025:15:17:10 +0100] "GET /tinyurl.com/3ant4vc2 HTTP/1.0" 200 24964
```

**Why is this suspicious?**
- The path contains a URL shortener domain (tinyurl.com) in the GET request
- This is unusual because GET paths typically reference local resources
- The attacker is using URL shortening to obfuscate the actual beacon location
- The response size (24964 bytes) suggests content was successfully delivered

### Step 5: Follow the Beacon URL
The suspicious GET request references: `tinyurl.com/3ant4vc2`

Visit the full URL:
```bash
curl -L https://tinyurl.com/3ant4vc2
# OR
curl https://pastebin.com/raw/Fx0y5fSu
```

The tinyurl redirects to a Pastebin page containing the flag.

### Step 6: Extract the Flag
The Pastebin content reveals the flag left by the APT group.

## Solution
By analyzing the Apache logs and identifying the suspicious GET request at line 820, we discover a URL shortener that leads to a Pastebin page containing the flag.

**Flag:** `CSC25{ApT_99_1s_uPOn_uS !!! }`

## Why This Vulnerability Exists

### Attack Technique: Web Shell / Beacon Deployment
1. **Initial Compromise**: The attacker gained access to the web server (method not shown in logs)
2. **Command & Control Setup**: The GET request retrieves a beacon/payload from an external source
3. **URL Obfuscation**: Using URL shorteners makes detection harder and hides the actual C2 server
4. **Persistence**: The beacon allows continued access even if initial entry point is closed

### Log Analysis Gaps
- The compromised web application didn't properly sanitize or validate request paths
- No alerting was configured for suspicious patterns (URL shorteners, external domains in paths)
- The large response size should have triggered anomaly detection

## Key Takeaways
1. **Web server logs are critical forensic artifacts** - They provide a timeline of attacker activity
2. **URL shorteners are red flags** - Legitimate applications rarely use URL shorteners in request paths
3. **Anomaly detection is essential** - Automated monitoring could catch these patterns in real-time
4. **Context matters in forensics** - Understanding normal vs. abnormal behavior is key to finding compromises
5. **APT groups leave traces** - Even sophisticated attackers leave forensic evidence

## Security Recommendations

### Detection & Monitoring
- Implement real-time log analysis with SIEM solutions (Splunk, ELK Stack, Wazuh)
- Configure alerts for suspicious patterns:
  - URL shortener domains in request paths
  - Unusual HTTP methods or paths
  - Large response sizes for unexpected resources
  - Requests from suspicious IP addresses

### Prevention
- Implement Web Application Firewalls (WAF) to filter malicious requests
- Use allowlisting for valid request patterns
- Regular security audits and penetration testing
- Keep web server and application software up to date

### Response
- Establish incident response procedures for compromises
- Maintain detailed logging for forensic analysis
- Implement network segmentation to limit attack spread
- Regular backup and recovery testing

### Log Management Best Practices
- Enable comprehensive logging (requests, responses, errors)
- Centralize log collection and storage
- Implement log retention policies
- Protect logs from tampering (write-once storage, log signing)

## Educational Value
This challenge teaches:
- **Web server log analysis** - Understanding Apache Common Log Format
- **Threat hunting techniques** - Identifying indicators of compromise in logs
- **URL analysis** - Recognizing obfuscation techniques used by attackers
- **Incident response** - Following the attack chain from initial detection to impact assessment
- **Real-world APT tactics** - Understanding how advanced threat groups operate

The challenge simulates a realistic scenario where blue teams must analyze logs to understand the scope of a compromise and identify attacker infrastructure.
