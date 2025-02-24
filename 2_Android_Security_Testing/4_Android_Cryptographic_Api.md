# Mobile App Cryptography - Android

## Overview
In the Mobile App Cryptography chapter, we discussed best practices and common mistakes in cryptography. Now, we'll focus on Android's cryptography APIs, learning how to identify them in source code and check if they follow current best practices.

## Key Components of Android Cryptography
- **Security Provider** – Supplies cryptographic algorithms and security features.
- **KeyStore** – Manages cryptographic keys securely (covered in "Testing Data Storage").
- **KeyChain** – Works with KeyStore for key management (also in "Testing Data Storage").

## Android Cryptography APIs
Android cryptography is built on **Java Cryptography Architecture (JCA)**, which separates cryptographic functions from their implementations.
Most cryptography-related classes are found in:
- `java.security.*` and `javax.crypto.*` (Standard Java APIs)
- `android.security.*` and `android.security.keystore.*` (Android-specific APIs)

## Key Management Phases
Cryptographic keys go through these stages:
1. **Generating a key**
2. **Using a key**
3. **Storing a key** (Covered in "Testing Data Storage")
4. **Archiving a key**
5. **Deleting a key**

The **KeyStore** and **KeyChain** manage these phases, but their implementation depends on the app developer. During analysis, focus on:
- Key generation
- Random number generation
- Key rotation

## Changes in Android Cryptography
### Android 7.0+ (API 24+)
- No need to manually specify a security provider—use the default patched provider.
- The **Crypto provider** is deprecated, including its **SHA1PRNG** random number generator.

### Android 8.1+ (API 27+)
- **Conscrypt (AndroidOpenSSL)** is preferred over **Bouncy Castle**.
- New implementations for encryption methods like **AES, HMAC, and ECDSA**.
- Use **GCMParameterSpec** instead of **IvParameterSpec** for **GCM mode**.
- Changes in SSL handling (e.g., new socket types, stricter session management).

### Android 9+ (API 28+)
- Manually specifying a security provider triggers a **warning**.
- The **Crypto provider** is removed, causing a `NoSuchProviderException` if called.

### Android 10+ (API 29+)
- Introduced new **network security changes** (check the official documentation for details).

---
## General Recommendations
When reviewing an app’s cryptography, follow these best practices:

- **Follow Best Practices** – Ensure the app follows the recommendations in the "Cryptography for Mobile Apps" chapter.
- **Keep Security Providers Updated** – Always update the security provider for the latest fixes and improvements.
- **Use the Default Security Provider** – Do not manually specify a provider; instead, use **AndroidOpenSSL** or **Conscrypt**.
- **Avoid Deprecated Crypto Provider** – The **Crypto provider** and **SHA1PRNG** are outdated and should not be used.
- **Specify a Provider Only for Keystore** – If needed, specify a security provider only when using the **Android Keystore system**.
- **Use Proper Encryption Ciphers** – Do not use password-based encryption without an **Initialization Vector (IV)**.
- **Use KeyGenParameterSpec** – Instead of **KeyPairGeneratorSpec**, use **KeyGenParameterSpec** for generating cryptographic keys.

---

## Key Generation in Android
In Android, keys are generated to encrypt and decrypt data securely. Starting from **Android 6.0 (API 23)**, the **KeyGenParameterSpec** class provides better control over how keys are created and used.

### 1. Generating AES Keys in Android KeyStore
Android’s KeyStore allows secure storage and restricted key usage. Here’s how you can generate an AES key:

#### Step 1: Create a Key with KeyGenParameterSpec
```java
String keyAlias = "MySecretKey";

KeyGenParameterSpec keyGenParameterSpec = new KeyGenParameterSpec.Builder(keyAlias,
        KeyProperties.PURPOSE_ENCRYPT | KeyProperties.PURPOSE_DECRYPT)
        .setBlockModes(KeyProperties.BLOCK_MODE_CBC)
        .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_PKCS7)
        .setRandomizedEncryptionRequired(true)
        .build();

KeyGenerator keyGenerator = KeyGenerator.getInstance(KeyProperties.KEY_ALGORITHM_AES, "AndroidKeyStore");
keyGenerator.init(keyGenParameterSpec);
SecretKey secretKey = keyGenerator.generateKey();
```

#### Step 2: Encrypting Data with the AES Key
```java
String AES_MODE = KeyProperties.KEY_ALGORITHM_AES
        + "/" + KeyProperties.BLOCK_MODE_CBC
        + "/" + KeyProperties.ENCRYPTION_PADDING_PKCS7;

KeyStore keyStore = KeyStore.getInstance("AndroidKeyStore");
Key key = keyStore.getKey(keyAlias, null);

Cipher cipher = Cipher.getInstance(AES_MODE);
cipher.init(Cipher.ENCRYPT_MODE, key);

byte[] encryptedBytes = cipher.doFinal(input);
byte[] iv = cipher.getIV(); // Save IV for decryption
```

#### Step 3: Decrypting the Data
```java
Key key = keyStore.getKey(keyAlias, null);

Cipher cipher = Cipher.getInstance(AES_MODE);
IvParameterSpec params = new IvParameterSpec(iv);
cipher.init(Cipher.DECRYPT_MODE, key, params);

byte[] decryptedBytes = cipher.doFinal(encryptedBytes);
```

### 2. Why Use GCM Mode Instead of CBC?
- **GCM (Galois/Counter Mode)** provides built-in authentication, preventing data tampering.
- It doesn’t require padding, making encryption simpler and reducing vulnerabilities.

### 3. Key Generation Before Android 6.0 (API 23)
- Before API 23, AES keys weren’t supported in the KeyStore.
- Many apps used **RSA key pairs** for encryption.

Example: Generating an RSA Key Pair:
```java
KeyPairGenerator keyGenerator = KeyPairGenerator.getInstance("RSA", "AndroidKeyStore");
KeyPair keyPair = keyGenerator.generateKeyPair();
```

### 4. Generating a Strong AES Key from a Password (PBKDF2)
```java
public static SecretKey generateAESKey(char[] password, int keyLength) {
    int iterationCount = 10000;
    int saltLength = keyLength / 8;
    SecureRandom random = new SecureRandom();
    byte[] salt = new byte[saltLength];
    random.nextBytes(salt);

    KeySpec keySpec = new PBEKeySpec(password, salt, iterationCount, keyLength);
    SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA1");
    byte[] keyBytes = keyFactory.generateSecret(keySpec).getEncoded();
    return new SecretKeySpec(keyBytes, "AES");
}
```

### 5. Security Considerations
- Do not store cryptographic keys in the app code (they can be extracted).
- Rooted devices can expose sensitive keys from memory.
- Using NDK won’t fully hide encryption keys (attackers can dump memory and extract them).

---

### Random Number Generation in Android
Cryptographic applications require secure random number generation to ensure unpredictable values. However, using `java.util.Random` is not secure because attackers can potentially predict future values, leading to security risks like impersonation or unauthorized access.

#### 1. Use SecureRandom for Better Security
Instead of `java.util.Random`, always use `SecureRandom`, which provides stronger randomness and is recommended for cryptographic purposes:

```java
SecureRandom secureRandom = new SecureRandom();
byte[] randomBytes = new byte[16]; // Generate 16 random bytes
secureRandom.nextBytes(randomBytes);
```

#### 2. Android Versions Below 4.4 (API 19) – Extra Caution Needed
- Android 4.1 to 4.3 (API 16–18) had a bug where `SecureRandom` was not properly initialized.
- This could lead to weak random numbers, making cryptographic operations vulnerable.

#### 3. How to Properly Use SecureRandom
For most cases, use `SecureRandom` without arguments:

```java
SecureRandom secureRandom = new SecureRandom();
```

#### 4. SecureRandom Uses Strong Algorithms
On Android, `SecureRandom` relies on `SHA1PRNG` from the `AndroidOpenSSL` (Conscrypt) provider, ensuring better security than `java.util.Random`.

For more details, refer to the Android Documentation.
