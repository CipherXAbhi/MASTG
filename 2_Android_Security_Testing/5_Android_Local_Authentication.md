# Local Authentication

## Overview

Local authentication allows an app to verify a user’s identity using credentials stored on the device. This means the user can "unlock" the app or certain features by entering a PIN, password, or using biometrics like a fingerprint or face scan.

The main purpose of local authentication is to make it easier for users to resume their session or to add extra security for sensitive actions.

However, it's important to remember that local authentication alone is not fully secure. It should always be backed by a remote server or a strong cryptographic method. Otherwise, attackers can easily bypass it if no secure data is returned during the authentication process.

## Local Authentication Methods on Android

On Android, there are two main ways to implement local authentication:

1. **Confirm Credential Flow** – Uses the device’s lock screen credentials (PIN, pattern, or password).
2. **Biometric Authentication Flow** – Uses fingerprints or facial recognition for verification.

### Confirm Credential Flow

Introduced in Android 6.0, the Confirm Credential Flow allows users to authenticate using their device's lock screen credentials (PIN, pattern, or password) instead of app-specific passwords.

If the user has recently unlocked their device, the system can use this authentication to access cryptographic keys stored in the Android Keystore. However, if too much time has passed (as defined by `setUserAuthenticationValidityDurationSeconds`), the user must unlock their device again.

🔴 **Security Note:** The strength of this method depends on the security of the device's lock screen. If a weak pattern or PIN is used, attackers may easily bypass it. Therefore, it’s not recommended for apps requiring high security (Level 2 or higher security controls).

### Biometric Authentication Flow

Biometric authentication allows users to log in using fingerprints, facial recognition, or other biometric methods. While it's convenient, it also introduces security risks.

Android provides three classes for biometric authentication:

- **Android 10+ (API 29) → BiometricManager:** Checks if the device has biometric hardware and if the user has set it up.
- **Android 9+ (API 28) → BiometricPrompt:** Displays a system-provided biometric authentication dialog.
- **Android 6.0+ (API 23) → FingerprintManager (deprecated in Android 9):** Supports fingerprint authentication but lacks a built-in UI.

✅ **Why is BiometricPrompt better?**

- It provides a consistent UI across Android devices.
- It supports multiple biometric methods, not just fingerprints.
- Unlike FingerprintManager, developers don’t have to create a custom UI.

---
# FingerprintManager (Deprecated in Android 9)

Android 6.0 (API 23) introduced the `FingerprintManager` class, which allowed apps to authenticate users via fingerprints. However, it was **deprecated** in Android 9 (API 28) in favor of `BiometricPrompt`, which provides a more secure and standardized experience.

## How it Works
1. An app creates a `FingerprintManager` object.
2. It calls the `authenticate` method.
3. The app listens for authentication results (success, failure, or error).

## 🔴 Security Warning
- This method **does not guarantee** that fingerprint authentication was actually performed.
- Attackers can bypass it using dynamic instrumentation or by modifying the authentication step.

## How to Improve Security

### 1️⃣ Using Fingerprint with KeyStore (Symmetric Cryptography)
A safer way to use fingerprints is by combining them with the Android KeyStore:

✔ A symmetric AES key is stored in the KeyStore and locked behind fingerprint authentication.
✔ The key encrypts an authentication token used to access a remote service.
✔ Calling `setUserAuthenticationRequired(true)` ensures the user must authenticate to retrieve the key.
✔ The encrypted token is stored securely on the device (e.g., in Shared Preferences).

### 2️⃣ Using Asymmetric Cryptography (Even More Secure) 🔐
For even better security, use asymmetric cryptography:

✔ The app generates a public-private key pair in the KeyStore.
✔ The **public key** is stored on the server.
✔ The **private key** is used to sign transactions, and the server verifies them using the public key.

---
Using `BiometricPrompt` instead of `FingerprintManager` ensures better security and a more standardized authentication experience across Android devices.

---
# Biometric Library

Android provides a **Biometric Library** that ensures compatibility for `BiometricPrompt` and `BiometricManager` APIs across different Android versions. This allows biometric authentication to work consistently from Android 6.0 (API 23) to Android 10+.

You can find reference implementations and instructions in the [Android Developer Documentation](https://developer.android.com/).

## Enhanced Security with CryptoObject 🔐
The `BiometricPrompt` class offers two authentication methods, one of which uses a `CryptoObject` for extra security.

### How the CryptoObject Authentication Flow Works:
1️⃣ A key is created in the Android KeyStore with the following security settings:
   - `setUserAuthenticationRequired(true)` → Ensures authentication is required.
   - `setInvalidatedByBiometricEnrollment(true)` → Prevents bypassing by enrolling a new fingerprint.
   - `setUserAuthenticationValidityDurationSeconds(-1)` → Requires authentication every time.
2️⃣ The key is used to encrypt sensitive data (e.g., session tokens).
3️⃣ A valid biometric scan is required to unlock the key and decrypt the data.

✅ **Why is this secure?**
- Even on a rooted device, this method cannot be bypassed because the key is securely stored in the KeyStore.
- If the `CryptoObject` is **NOT** used, biometric authentication can be bypassed using tools like Frida (see "Dynamic Instrumentation" for more details).

---


### FingerprintManager (Deprecated in Android 9)

Android 6.0 (API 23) introduced the FingerprintManager class, which allowed apps to authenticate users via fingerprints. However, it was deprecated in Android 9 (API 28) in favor of BiometricPrompt, which provides a more secure and standardized experience.

#### How Fingerprint Authentication Works

1️⃣ **Search for `FingerprintManager.authenticate()` calls.**
   - It should include a **CryptoObject** parameter (a wrapper for cryptographic objects).
   - If `null` is passed instead of **CryptoObject**, the authentication is event-based and may have security risks.

2️⃣ **Check how the key is created**
   - It should be generated using the **KeyGenerator** class.
   - `setUserAuthenticationRequired(true)` should be enabled in the **KeyGenParameterSpec** object.

3️⃣ **Verify authentication logic**
   - The remote server should validate the authentication result, ensuring that:
     - A valid secret from the **KeyStore** is used.
     - A derived value from the secret is presented.
     - A signature generated with the client’s private key is verified.

#### Preconditions for Fingerprint Authentication

Before allowing fingerprint authentication, the app should check:

✅ **Permission is requested in `AndroidManifest.xml`**:
```xml
<uses-permission android:name="android.permission.USE_FINGERPRINT" />
```

✅ **Fingerprint hardware is available**:
```java
FingerprintManager fingerprintManager = (FingerprintManager) context.getSystemService(Context.FINGERPRINT_SERVICE);
fingerprintManager.isHardwareDetected();
```

✅ **User has a protected lock screen**:
```java
KeyguardManager keyguardManager = (KeyguardManager) context.getSystemService(Context.KEYGUARD_SERVICE);
keyguardManager.isKeyguardSecure(); // If not, prompt the user to set up a lock screen
```

✅ **At least one fingerprint is registered**:
```java
fingerprintManager.hasEnrolledFingerprints();
```

✅ **App has fingerprint authentication permission**:
```java
context.checkSelfPermission(Manifest.permission.USE_FINGERPRINT) == PermissionResult.PERMISSION_GRANTED;
```

#### Security Considerations

🔹 **Not all devices have hardware-backed key storage**
```java
SecretKeyFactory factory = SecretKeyFactory.getInstance(getEncryptionKey().getAlgorithm(), "AndroidKeyStore");
KeyInfo keyInfo = (KeyInfo) factory.getKeySpec(yourEncryptionKey, KeyInfo.class);
keyInfo.isInsideSecureHardware();
```

🔹 **Check if the authentication policy is enforced by hardware**
```java
keyInfo.isUserAuthenticationRequirementEnforcedBySecureHardware();
```

### Using Symmetric Key for Fingerprint Authentication

🔹 **Create an AES key in KeyStore**:
```java
KeyGenerator generator = KeyGenerator.getInstance(KeyProperties.KEY_ALGORITHM_AES, "AndroidKeyStore");

generator.init(new KeyGenParameterSpec.Builder(KEY_ALIAS,
        KeyProperties.PURPOSE_ENCRYPT | KeyProperties.PURPOSE_DECRYPT)
        .setBlockModes(KeyProperties.BLOCK_MODE_CBC)
        .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_PKCS7)
        .setUserAuthenticationRequired(true)
        .build());

generator.generateKey();
```

🔹 **Authenticate before using the key**:
```java
FingerprintManager.CryptoObject cryptoObject = new FingerprintManager.CryptoObject(cipher);
fingerprintManager.authenticate(cryptoObject, new CancellationSignal(), 0, this, null);
```

### Using Asymmetric Key for Fingerprint Authentication

🔹 **Generate a signing key pair**:
```java
KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(KeyProperties.KEY_ALGORITHM_EC, "AndroidKeyStore");

keyPairGenerator.initialize(new KeyGenParameterSpec.Builder(MY_KEY,
        KeyProperties.PURPOSE_SIGN)
        .setDigests(KeyProperties.DIGEST_SHA256)
        .setAlgorithmParameterSpec(new ECGenParameterSpec("secp256r1"))
        .setUserAuthenticationRequired(true)
        .build());

keyPairGenerator.generateKeyPair();
```

🔹 **Authenticate and sign data**:
```java
Signature signature = Signature.getInstance("SHA256withECDSA");
KeyStore keyStore = KeyStore.getInstance("AndroidKeyStore");
keyStore.load(null);
PrivateKey key = (PrivateKey) keyStore.getKey(MY_KEY, null);
signature.initSign(key);
```

🔹 **Sign the data**:
```java
Signature signature = cryptoObject.getSignature();
signature.update(inputBytes);
byte[] signedData = signature.sign();
```

🔹 **Prevent replay attacks**:
   - Always include a random nonce in the signed data.

---

# Security Enhancements in Android Biometric Authentication

## Android 7.0 (API 24) Update
- Introduced `setInvalidatedByBiometricEnrollment(true)`, which ensures that previously stored fingerprint authentication keys are deleted when a new fingerprint is added.
- This prevents attackers from enrolling their own fingerprint and using it to access protected data.

## Android 8.0 (API 26) Update
- Added two new fingerprint error codes:
  - `FINGERPRINT_ERROR_LOCKOUT_PERMANENT`: Triggered when a user fails too many fingerprint attempts.
  - `FINGERPRINT_ERROR_VENDOR`: Indicates an error specific to the fingerprint hardware manufacturer.

## Implementing Secure Biometric Authentication

### Ensure Device Security
Check if the user has set up a secure lock screen before allowing biometric authentication.

```java
KeyguardManager mKeyguardManager = (KeyguardManager) getSystemService(Context.KEYGUARD_SERVICE);
if (!mKeyguardManager.isKeyguardSecure()) {
    // Prompt the user to set up a secure lock screen.
}
```

### Generate a Secure Key
The key should only be accessible when the user has recently unlocked the device.

```java
try {
    KeyStore keyStore = KeyStore.getInstance("AndroidKeyStore");
    keyStore.load(null);
    KeyGenerator keyGenerator = KeyGenerator.getInstance(
            KeyProperties.KEY_ALGORITHM_AES, "AndroidKeyStore");

    keyGenerator.init(new KeyGenParameterSpec.Builder(KEY_NAME,
            KeyProperties.PURPOSE_ENCRYPT | KeyProperties.PURPOSE_DECRYPT)
            .setBlockModes(KeyProperties.BLOCK_MODE_CBC)
            .setUserAuthenticationRequired(true)  // Requires recent unlock
            .setUserAuthenticationValidityDurationSeconds(30)  // Valid for 30 sec
            .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_PKCS7)
            .build());
    keyGenerator.generateKey();
} catch (Exception e) {
    throw new RuntimeException("Failed to create a secure key", e);
}
```

### Confirm Lock Screen Authentication
Prompt the user to confirm their identity using the lock screen before using the key.

```java
private static final int REQUEST_CODE_CONFIRM_DEVICE_CREDENTIALS = 1;
Intent intent = mKeyguardManager.createConfirmDeviceCredentialIntent(null, null);
if (intent != null) {
    startActivityForResult(intent, REQUEST_CODE_CONFIRM_DEVICE_CREDENTIALS);
}
```

### Handle Lock Screen Authentication Result
If the user successfully verifies their identity, proceed with the authentication process.

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == REQUEST_CODE_CONFIRM_DEVICE_CREDENTIALS) {
        if (resultCode == RESULT_OK) {
            // Proceed with authentication
        } else {
            // Authentication failed or was canceled
        }
    }
}
```

## Using Third-Party SDKs for Biometric Authentication
- Always use the official Android SDK and its APIs for fingerprint authentication.
- If using a third-party SDK, ensure:
  - It has been thoroughly tested for security vulnerabilities.
  - It relies on secure hardware (Trusted Execution Environment or Secure Element).
  - Biometric data is only used to unlock cryptographic secrets and cannot be bypassed.

