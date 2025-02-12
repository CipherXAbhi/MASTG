# Mobile App Cryptography Simplified
Cryptography is essential for securing user data, especially in mobile apps where attackers may have physical access to devices. Below are key cryptographic concepts and best practices for mobile security.

---

## Key Concepts
+ Confidentiality: Protects data privacy using encryption.
+ Integrity: Ensures data consistency and detects modifications using hashing.
+ Authenticity: Confirms data comes from a trusted source.

### Encryption
+ Symmetric Encryption: Uses a single key for both encryption and decryption. Fast and efficient but requires careful key management.
+ Asymmetric Encryption: Uses a public-private key pair. Public key encrypts, private key decrypts. More secure but slower, mainly used for encrypting small data like symmetric keys.

### Hashing & Message Integrity
+ Hashing: Converts data into a fixed-length value. It is irreversible and used for integrity verification but does not provide authenticity.
+ Message Authentication Code (MAC): Uses hashing with a secret key to ensure integrity and authenticity. HMAC (e.g., HMAC-SHA256) is the most common type.Digital Signatures: Use asymmetric encryption and hashing to ensure integrity, authenticity, and non-repudiation.

### Key Derivation Functions (KDFs)
+ Convert passwords into cryptographic keys.
+ Enhance security by increasing key randomness and length.

---

## Identifying Insecure & Deprecated Cryptographic Algorithms
Mobile apps must avoid outdated cryptographic algorithms and follow modern security standards. Over time, once-secure algorithms can become weak, so regular updates are essential.

### Common Insecure Algorithms
Avoid using these weak or deprecated cryptographic algorithms:
+ Block Ciphers: DES, 3DES, RC2, BLOWFISH
+ Stream Ciphers: RC4
+ Hash Functions: MD4, MD5, SHA1
+ Random Number Generators: Dual_EC_DRBG, SHA1PRNG

### Best Practices for Secure Cryptography
**1. Use Up-to-Date Algorithms**
  + Replace weak algorithms with modern alternatives.
  + Ensure encryption methods are standardized and open for verification.

**2. Follow Industry Standards for Key Lengths**
  + RSA: 3072 bits or higher
  + Diffie-Hellman (DH): 3072 bits or higher
  + Elliptic Curve Diffie-Hellman (ECDH): NIST P-384

**3. Ensure Proper Cryptographic Usage**
  + Do not mix cryptographic functions (e.g., signing with a public key).
  + Use proper key management and avoid reusing keys for different purposes.
  + Ensure cryptographic parameters (e.g., salts, IVs) are securely implemented.

4. Recommended Algorithms

| Purpose                   | Recommended Algorithms                              |
|---------------------------|-----------------------------------------------------|
| AEncryption               | AES-GCM-256, ChaCha20-Poly1305                      |
| Integrity                 | SHA-256, SHA-384, SHA-512, BLAKE3, SHA-3 family     |
| Digital Signatures        | RSA (3072+ bits), ECDSA (NIST P-384)                |
| Key Establishment         | RSA (3072+ bits), DH (3072+ bits), ECDH (NIST P-384)|

5. Utilize Secure Hardware
  + Store encryption keys securely using hardware-backed solutions when available.



---

# Common Configuration Issues
## Insufficient Key Length
Encryption is only secure if the key is strong enough. Weak keys make even the best algorithms vulnerable to brute-force attacks. Always use industry-recommended key lengths.

## Hard-Coded Cryptographic Keys
Symmetric encryption relies on keeping keys secret. If a key is hardcoded in the app, attackers can easily extract it, compromising security. Avoid storing keys in:

+ Source code (Java, Kotlin, Swift, etc.)
+ Application resources
+ Predictable values
Even obfuscation doesn't fully protect hardcoded keys. Always store secrets in a secure location, like a device’s keychain or secure enclave.

### Two-Way TLS Security
If your app uses two-way TLS (validating both client and server certificates), ensure:

+ Client certificate passwords are not stored locally.
+ Certificates are unique per installation.

### Secure Encrypted Containers
For apps using encrypted data storage:

+ Ensure key-wrapping schemes re-encrypt data with new keys when needed.
+ Users should not be able to decrypt with old passwords.

### Secure Key Storage
Always store cryptographic keys in secure device storage (e.g., Android Keystore or iOS Keychain) to prevent unauthorized access.

--- 

## Inadequate AES Configuration
Weak Block Cipher Mode
AES encryption divides data into fixed-size blocks. Choosing the wrong mode can expose patterns:

+ ECB (Electronic Codebook) encrypts each block separately, making repeated data identifiable.
+ CBC (Cipher Block Chaining) is preferred since it XORs each block with the previous ciphertext block, making it more secure.
Best Practice: Use CBC with HMAC to prevent padding oracle attacks, or GCM mode for encryption with integrity protection.

## Predictable Initialization Vector (IV)
Certain AES modes (CBC, GCM, etc.) require an IV, which must be:<br>
✅ Random<br>
✅ Unique for each encryption operation

Avoid: Hardcoded or predictable IVs (often found in bad example code).

## IVs in Stateful Modes
+ CTR (Counter Mode): Uses a nonce + counter for each block.
+ GCM (Galois/Counter Mode): Uses one IV per operation, which must not be repeated with the same key.

## Padding Oracle Attacks
Poor padding mechanisms allow attackers to guess decrypted values.
🚫 PKCS1.5 Padding is vulnerable—use OAEP Padding instead.
🚫 AES-CBC with PKCS#7 can be attacked if error messages leak information.

## Mitigation:
✔️ Use OAEP padding for asymmetric encryption.
✔️ Add an HMAC after encryption to detect tampering before decryption.

---

## Protecting Cryptographic Keys
### Keys in Storage and Memory
🔹 Remote Storage: Use cloud-based key vaults like AWS KMS, Azure Key Vault, or Google Cloud Functions to store and manage keys securely.

🔹 Hardware-backed Storage: Store keys in a Trusted Execution Environment (TEE) like Android Keystore or Secure Enclave on iOS.

🔹 Envelope Encryption: If keys must be stored outside secure environments, use multi-layer encryption (e.g., HPKE, Google/AWS key management).

🔹 Memory Protection: Keep keys in memory only as long as needed, then clear them to prevent memory dumps.

## ⚠️ Never reuse keys across accounts or devices!

## Keys in Transport
🔹 Use secure transmission methods: Encrypt symmetric keys with an asymmetric public key before sending them.

🔹 Avoid obfuscation methods that can be easily reversed—use asymmetric cryptography instead.

## Platform-Specific Cryptographic APIs
🔹 Android & iOS offer their own APIs for secure storage & network encryption (covered in platform-specific sections).

## Cryptographic Policies & Regulations
🔹 Follow a cryptographic policy (e.g., NIST Key Management guidelines) to prevent common mistakes.

🔹 Comply with export laws when distributing apps internationally—both US and local regulations may apply.

📌 Resources:

Apple & Google encryption compliance guides
US & global encryption laws
