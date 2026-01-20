# Posta-beta - Android Challenge Writeup

## Challenge Information
- **Category:** Android
- **Points:** 400
- **Difficulty:** Medium-Hard

## Challenge Description
This challenge involves reverse engineering an Android application (APK) to discover hardcoded credentials that have been obfuscated using XOR encryption. The application uses a custom encryption mechanism to protect sensitive data.

## Tools Required
- **APKTool** or **JADX** - For decompiling the APK
- **Java Development Kit (JDK)** - To run the decryption script
- **Text Editor** - For analyzing the decompiled code

## Methodology

### Step 1: Decompile the APK
First, we need to decompile the `posta-beta.apk` file to examine its source code:

```bash
# Using JADX (recommended)
jadx posta-beta.apk -d output/

# OR using APKTool
apktool d posta-beta.apk
```

### Step 2: Analyze the Source Code
After decompilation, explore the source code to find interesting strings or encryption mechanisms. Look for:
- Hardcoded byte arrays
- Encryption/decryption functions
- Package names and constants

In this challenge, we find an interesting byte array in the decompiled code:
```java
byte[] SEC_BYTES = {32, 60, 46, 28, 80, 3, 49, 2, 3, 24, 4, 113, 18, 92, 7, 64, 62, 85, 90, 89, 28, 4, 72, 2, 94, 72, 8, 86, 72, 21, 18};
```

### Step 3: Identify the Encryption Scheme
By examining the code, we discover that the application uses **XOR encryption** with the package name as the key. The encryption scheme is:
- **Key:** Package name (`com.example.posta`)
- **Encrypted Data:** SEC_BYTES array
- **Algorithm:** XOR cipher with repeating key

### Step 4: Write the Decryption Script
Create a Java program to decrypt the password:

```java
public class DecryptPassword {
    public static void main(String[] args) {
        byte[] SEC_BYTES = {32, 60, 46, 28, 80, 3, 49, 2, 3, 24, 4, 113, 18, 92, 7, 64, 62, 85, 90, 89, 28, 4, 72, 2, 94, 72, 8, 86, 72, 21, 18};
        String packageName = "com.example.posta";
        
        try {
            byte[] ccskey = packageName.getBytes("ASCII");
            byte[] decrypted = new byte[SEC_BYTES.length];
            
            for (int i = 0; i < SEC_BYTES.length; i++) {
                decrypted[i] = (byte) (SEC_BYTES[i] ^ ccskey[i % ccskey.length]);
            }
            
            System.out.println("Decrypted password: " + new String(decrypted, "ASCII").trim());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### Step 5: Run the Decryption Script
Compile and execute the Java program:

```bash
javac DecryptPassword.java
java DecryptPassword
```

### Step 6: Extract the Flag
The decrypted output reveals the flag hidden in the encrypted byte array.

## Solution
The XOR decryption reveals the flag embedded in the application's encrypted data.

**Flag:** `CSC25{Posta_b3t4_6542a0c38d3fe}`

## Key Takeaways
1. **APK files can be easily decompiled** - Never store sensitive data directly in APK files
2. **XOR encryption is weak** - It's easily reversible, especially with known or guessable keys
3. **Package names are public** - Using package names as encryption keys provides no real security
4. **Obfuscation ≠ Security** - Code obfuscation may slow down attackers but doesn't prevent reverse engineering

## Security Recommendations
- Never hardcode credentials in mobile applications
- Use proper encryption with secure key management (e.g., Android Keystore)
- Implement certificate pinning for API communications
- Use ProGuard/R8 for code obfuscation (but don't rely on it for security)
- Store sensitive data on secure backend servers, not in the app itself
