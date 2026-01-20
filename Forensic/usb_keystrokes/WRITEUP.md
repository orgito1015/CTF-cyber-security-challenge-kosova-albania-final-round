# USB Keystrokes - Forensic Challenge Writeup

## Challenge Information
- **Category:** Forensic
- **Points:** 400
- **Difficulty:** Medium-Hard

## Challenge Description
"We managed to hack the victim and deploy a keylogger. Now we need to extract the data."

This challenge involves analyzing USB HID (Human Interface Device) data from a packet capture to extract keystrokes captured by a USB keylogger. The challenge requires understanding USB protocol and decoding HID keyboard data.

## Tools Required
- **Wireshark** / **tshark** - Network protocol analyzer
- **Python 3** - For data processing scripts
- **Text Editor** - For analyzing extracted data
- **USB HID Knowledge** - Understanding USB keyboard protocol

## Background: USB HID Protocol

### What is USB HID?
- **HID** = Human Interface Device
- Protocol for keyboards, mice, game controllers
- Sends data in 8-byte packets
- Each packet contains modifier keys and key codes

### USB HID Data Format:
```
Byte 0: Modifier keys (Shift, Ctrl, Alt, etc.)
Byte 1: Reserved (usually 0x00)
Byte 2-7: Key codes (up to 6 simultaneous key presses)
```

Example: `02:00:04:00:00:00:00:00`
- `02` = Left Shift pressed
- `00` = Reserved
- `04` = 'a' key (but shifted, so 'A')
- Rest are zeros (no other keys)

## Methodology

### Step 1: Analyze the PCAP File
First, examine the packet capture:

```bash
# View capture info
capinfos usb_keystrokes.pcap

# Open in Wireshark
wireshark usb_keystrokes.pcap

# Or use tshark
tshark -r usb_keystrokes.pcap
```

Look for USB traffic with HID data.

### Step 2: Extract USB HID Data
Use tshark to extract HID data from the capture:

```bash
# Extract USB capture data (HID Data)
tshark -r usb_keystrokes.pcap -Y 'usb.capdata' -T fields -e usb.capdata > raw_data.txt
```

Or use the provided `wireshark.py` script to extract data from Wireshark's output.

### Step 3: Format the Data
The extracted data needs proper formatting. Use the extraction script:

```python
import sys

# Open the flag.txt file (contains Wireshark output)
with open("flag.txt", "r") as file:
    lines = file.readlines()

# Extract HID Data values
hid_data_values = []
for line in lines:
    if "HID Data:" in line:
        # Extract the value after "HID Data:"
        value = line.split("HID Data:")[1].strip()
        # Add a colon after every 2 characters
        formatted_value = ":".join(value[i:i+2] for i in range(0, len(value), 2))
        hid_data_values.append(formatted_value)

# Save formatted values to keyboard.txt
with open("keyboard.txt", "w") as output_file:
    for value in hid_data_values:
        output_file.write(value + "\n")
```

This converts:
- `020004000000000000` → `02:00:04:00:00:00:00:00`

### Step 4: Decode HID Data to Keystrokes
Use the USB HID to keystrokes converter (`usbToKey.py`):

```python
# USB HID Key Code Mapping
KEY_CODES = {
    0x04:['a', 'A'], 0x05:['b', 'B'], 0x06:['c', 'C'], 0x07:['d', 'D'],
    0x08:['e', 'E'], 0x09:['f', 'F'], 0x0A:['g', 'G'], 0x0B:['h', 'H'],
    0x0C:['i', 'I'], 0x0D:['j', 'J'], 0x0E:['k', 'K'], 0x0F:['l', 'L'],
    0x10:['m', 'M'], 0x11:['n', 'N'], 0x12:['o', 'O'], 0x13:['p', 'P'],
    0x14:['q', 'Q'], 0x15:['r', 'R'], 0x16:['s', 'S'], 0x17:['t', 'T'],
    0x18:['u', 'U'], 0x19:['v', 'V'], 0x1A:['w', 'W'], 0x1B:['x', 'X'],
    0x1C:['y', 'Y'], 0x1D:['z', 'Z'], 0x1E:['1', '!'], 0x1F:['2', '@'],
    0x20:['3', '#'], 0x21:['4', '$'], 0x22:['5', '%'], 0x23:['6', '^'],
    0x24:['7', '&'], 0x25:['8', '*'], 0x26:['9', '('], 0x27:['0', ')'],
    0x28:['\n','\n'], 0x29:['[ESC]','[ESC]'], 0x2a:['[BACKSPACE]', '[BACKSPACE]'],
    0x2C:[' ', ' '], 0x2D:['-', '_'], 0x2E:['=', '+'], 0x2F:['[', '{'],
    0x30:[']', '}'], 0x33:[';', ':'], 0x34:['\'', '"'], 0x36:[',', '<'],
    0x37:['.', '>'], 0x38:['/', '?'], 0x39:['[CAPSLOCK]','[CAPSLOCK]'],
    0x4f:[u'→',u'→'], 0x50:[u'←',u'←'], 0x51:[u'↓',u'↓'], 0x52:[u'↑',u'↑']
}

def read_usb(file):
    with open(file, 'r') as f:
        datas = f.read().split('\n')
    
    datas = [d.strip() for d in datas if d]
    output = ''
    skip_next = False
    
    for data in datas:
        shift = int(data.split(':')[0], 16)  # Modifier byte
        key = int(data.split(':')[2], 16)     # Key code
        
        if skip_next:
            skip_next = False
            continue
        
        if key == 0:  # No key pressed
            continue
        
        # Check if shift is pressed (0x02 = left shift, 0x20 = right shift)
        if shift != 0:
            shift = 1
            skip_next = True
        
        # Handle special keys
        if key in KEY_CODES:
            if KEY_CODES[key][shift] == '[BACKSPACE]':
                output = output[:-1]
            elif KEY_CODES[key][shift] == '\n':
                output += '\n'
            else:
                output += KEY_CODES[key][shift]
    
    return output

if __name__ == '__main__':
    result = read_usb("keyboard.txt")
    print(result)
```

### Step 5: Run the Complete Solution

#### Complete Workflow:
```bash
# Step 1: Extract HID data from PCAP (if needed)
tshark -r usb_keystrokes.pcap -Y 'usb.capdata' -T fields -e usb.capdata > raw_hid.txt

# Step 2: Format the data (if starting from flag.txt)
python3 wireshark.py

# Step 3: Decode HID data to keystrokes
python3 usbToKey.py
```

### Step 6: Extract the Flag
The decoded output will contain the captured keystrokes, including the flag.

## Solution
By extracting and decoding USB HID data from the packet capture, we recover all keystrokes typed on the victim's keyboard, including the flag.

**Flag:** `CSC25{usB_f0r3nsic5s_re4lly_rock5!}`

## Understanding the Challenge

### Why USB Keyloggers Are Dangerous:
1. **Hardware-based** - Difficult to detect with software
2. **Passive** - Don't modify system behavior
3. **Pre-boot capture** - Can capture BIOS passwords
4. **No drivers needed** - Works immediately when plugged in
5. **Small & inconspicuous** - Can be hidden in cables

### Types of USB Keyloggers:
1. **Hardware dongles** - Between keyboard and computer
2. **Modified cables** - Keylogger built into cable
3. **Keyboard implants** - Embedded in the keyboard itself
4. **Firmware-based** - Modified USB controller firmware

## USB HID Data Structure Explained

### Modifier Byte (Byte 0):
```
Bit 0: Left Ctrl
Bit 1: Left Shift
Bit 2: Left Alt
Bit 3: Left GUI (Windows/Command key)
Bit 4: Right Ctrl
Bit 5: Right Shift
Bit 6: Right Alt
Bit 7: Right GUI
```

### Key Code Examples:
- `0x04` = A/a
- `0x1E` = 1/!
- `0x28` = Enter
- `0x2C` = Space
- `0x2A` = Backspace

### Example Decoding:
```
HID Data: 02:00:04:00:00:00:00:00
- Byte 0 (02): Left Shift is pressed
- Byte 2 (04): 'a' key is pressed
- Result: 'A' (uppercase due to shift)

HID Data: 00:00:04:00:00:00:00:00
- Byte 0 (00): No modifiers
- Byte 2 (04): 'a' key is pressed
- Result: 'a' (lowercase)
```

## Key Takeaways
1. **USB HID is not encrypted** - All keystrokes visible in plaintext
2. **Network captures reveal everything** - If USB over network, it's capturable
3. **Physical security matters** - Anyone with physical access can install keylogger
4. **PCAP analysis is powerful** - Can reconstruct user activity
5. **USB forensics is critical** - For incident response and investigations

## Defense Against USB Keyloggers

### For Users:
1. **Physical inspection** - Check for dongles between keyboard and computer
2. **Use wireless keyboards** - Reduces physical attack surface (but has other risks)
3. **USB port locks** - Physical locks to prevent unauthorized devices
4. **Visual inspection** - Regularly check equipment for tampering
5. **Secure areas** - Don't leave computers unattended in public

### For Organizations:
1. **USB port controls** - Disable unused USB ports in BIOS
2. **USB monitoring** - Use Device Guard / AppLocker policies
3. **Endpoint protection** - EDR solutions that monitor USB devices
4. **Security cameras** - Monitor areas with sensitive computers
5. **Physical security** - Restrict access to secure areas
6. **Device authentication** - Only allow approved USB devices

### Technical Controls:
```powershell
# Windows: Disable USB storage
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\USBSTOR" -Name "Start" -Value 4

# List USB devices
Get-PnpDevice | Where-Object {$_.Class -eq "USB"}
```

## Forensic Analysis Tips

### Useful Wireshark Filters:
```
# Show only USB HID data
usb.capdata

# Filter by device address
usb.device_address == 2

# Show only non-zero data
usb.capdata && !(usb.capdata == 00:00:00:00:00:00:00:00)

# Show keyboard events (interface class 3 = HID)
usb.bInterfaceClass == 3
```

### Advanced Analysis:
```python
# Analyze timing between keystrokes
import time

def analyze_timing(keyboard_file):
    """Detect typing patterns, pauses, etc."""
    with open(keyboard_file) as f:
        lines = f.readlines()
    
    for i, line in enumerate(lines):
        # Analyze patterns
        # Look for:
        # - Long pauses (potential copy-paste)
        # - Rapid typing (likely passwords)
        # - Backspace patterns (errors/corrections)
        pass
```

## Educational Value
This challenge teaches:
- Understanding USB HID protocol
- Network packet analysis with Wireshark/tshark
- Data extraction and transformation
- Keystroke reconstruction from raw data
- Real-world keylogger attack scenarios
- Importance of physical security

## Tools for USB Forensics

### Packet Capture:
- **Wireshark** - GUI packet analyzer
- **tshark** - Command-line packet analyzer
- **tcpdump** - Packet capture tool
- **USBPcap** - Windows USB packet capture

### USB Analysis:
- **USBDeview** - View installed USB devices (Windows)
- **lsusb** - List USB devices (Linux)
- **USB Forensic Tracker** - Track USB device usage
- **USB Detective** - Comprehensive USB forensics

### Python Libraries:
```python
import pyshark  # Wireshark wrapper for Python
import usb.core  # USB device access
import scapy.all  # Packet manipulation
```

## Real-World Applications
- **Incident Response** - Investigate potential keylogger infections
- **Penetration Testing** - Demonstrate physical security risks
- **Forensic Investigations** - Reconstruct user actions
- **Security Awareness** - Train users on hardware threats
- **Compliance** - Meet regulations requiring USB monitoring

## Common Pitfalls

### Issue 1: Missing Packets
**Problem:** Some keystrokes are missing
**Solution:** Ensure packet capture includes all USB traffic

### Issue 2: Wrong Key Mapping
**Problem:** Characters don't match
**Solution:** Verify keyboard layout (US vs International)

### Issue 3: Modifier Key Confusion
**Problem:** Shift/Ctrl keys not recognized
**Solution:** Check byte 0 processing logic

### Issue 4: Multiple Keyboards
**Problem:** Data from multiple devices
**Solution:** Filter by USB device address

## Additional Resources
- **USB HID Specification** - Official USB.org documentation
- **Wireshark USB Capture Guide** - Wireshark wiki
- **CTF USB Writeups** - Previous CTF solutions
- **SANS USB Forensics** - Professional training materials
