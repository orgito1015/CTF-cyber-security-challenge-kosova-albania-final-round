# Posta-dev - Android Challenge Writeup

## Challenge Information
- **Category:** Android
- **Points:** 200
- **Difficulty:** Easy-Medium

## Challenge Description
This is a straightforward Android application reverse engineering challenge where hardcoded credentials are hidden directly in the application's source code. The goal is to find the admin password stored in the MainActivity class.

## Tools Required
- **JADX** or **APKTool** - For decompiling the APK
- **Text Editor** or **IDE** - For viewing the decompiled source code

## Methodology

### Step 1: Decompile the APK
First, decompile the `posta-dev.apk` file to access its source code:

```bash
# Using JADX (Recommended - provides cleaner Java output)
jadx posta-dev.apk -d output_directory/

# Alternative: Using APKTool
apktool d posta-dev.apk -o output_directory/
```

### Step 2: Navigate to the MainActivity
After decompilation, navigate to the main source directory:

```
output_directory/sources/com/example/posta/MainActivity.java
```

### Step 3: Locate the Hardcoded Credentials
Open `MainActivity.java` and search for any hardcoded strings, constants, or credentials. Look for:
- `ADMIN_PASSWORD` constant
- `PASSWORD` variables
- Any suspicious string values

### Step 4: Extract the Flag
In the MainActivity.java file, you'll find a constant named `ADMIN_PASSWORD` that contains the flag:

```java
public class MainActivity extends AppCompatActivity {
    private static final String ADMIN_PASSWORD = "CSC25{Posta_h4rd_c0ded_cr3d5}";
    
    // ... rest of the code
}
```

## Solution
The flag is directly visible in the source code as a hardcoded constant.

![Solution Screenshot](image1.png)

**Flag:** `CSC25{Posta_h4rd_c0ded_cr3d5}`

## Alternative Methods

### Method 1: Using grep to search
If you've decompiled to a directory, you can use grep to quickly find the flag:

```bash
grep -r "CSC25" output_directory/
```

### Method 2: Using APK Analyzer (Android Studio)
1. Open Android Studio
2. Go to Build → Analyze APK
3. Select the posta-dev.apk file
4. Navigate through the classes.dex to find MainActivity
5. View the constant values

### Method 3: Using strings command
For a quick search without full decompilation:

```bash
strings posta-dev.apk | grep -i "CSC25"
```

## Key Takeaways
1. **Never hardcode credentials** - This is a critical security vulnerability (CWE-798)
2. **Decompilation is trivial** - Android APKs can be easily reverse-engineered
3. **Obfuscation helps but isn't foolproof** - Even obfuscated code can be analyzed
4. **Source code in APKs is accessible** - All Java/Kotlin code in APKs can be decompiled

## Security Recommendations

### For Developers:
1. **Never store credentials in the app** - Use secure backend authentication
2. **Use Android Keystore** - For storing cryptographic keys securely
3. **Implement ProGuard/R8** - Obfuscate your code (but don't rely on it for secrets)
4. **Use environment variables** - For configuration, not hardcoded values
5. **Implement runtime checks** - Detect if the app is running on a rooted device or in an emulator
6. **Use certificate pinning** - For secure API communications

### Best Practices:
- Store sensitive data on the server side
- Use OAuth 2.0 or similar authentication frameworks
- Implement proper session management
- Use encrypted shared preferences for local data
- Regularly audit your code for hardcoded secrets

## Common Tools for Android Security Testing
- **JADX** - Java decompiler
- **APKTool** - APK reverse engineering tool
- **dex2jar** - Converts DEX to JAR files
- **JD-GUI** - Java decompiler GUI
- **Frida** - Dynamic instrumentation toolkit
- **MobSF** - Mobile Security Framework for automated analysis
