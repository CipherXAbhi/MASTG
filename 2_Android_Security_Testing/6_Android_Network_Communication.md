# Android Network Communication

## Overview
Most Android apps connect to remote services over the internet. Since public networks (like Wi-Fi) are not secure, attackers can try to intercept data. Therefore, securing network communication is essential.

---

## Android Network Security Configuration
Since **Android 7.0 (API 24)**, apps can use the **Network Security Configuration** feature to control network security settings.

### Key Features
1. **Cleartext Traffic Protection**: Prevents unencrypted (HTTP) data transmission by default.
2. **Custom Trust Anchors**: Allows apps to trust specific Certificate Authorities (CAs).
3. **Certificate Pinning**: Ensures the app only connects to specific, predefined certificates.
4. **Debugging Overrides**: Enables safe debugging without exposing security risks.

---

## Checking an App’s Network Security Configuration
To see if an app uses a custom security configuration, look for this line in **AndroidManifest.xml**:
```xml
<application android:networkSecurityConfig="@xml/network_security_config" />
```
This means the settings are stored in `res/xml/network_security_config.xml`.

To confirm, check system logs for:
```
D/NetworkSecurityConfig: Using Network Security Config from resource network_security_config
```

---

## How Network Security Configuration Works
Network security rules are defined in an **XML file**, allowing different settings for different domains.

### Example Configuration
This prevents **cleartext traffic (HTTP)** for all domains except `localhost`:
```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="false" />
    <domain-config cleartextTrafficPermitted="true">
        <domain>localhost</domain>
    </domain-config>
</network-security-config>
```

---

## Default Security Settings Based on Android Version

| **Android Version** | **Cleartext Traffic** | **Trusted Certificates** |
|--------------------|--------------------|----------------------|
| Android 9+ (API 28+) | **Disabled** | System certificates only |
| Android 7 - 8.1 (API 24 - 27) | **Enabled** | System certificates only |
| Android 6 & below (API 23-) | **Enabled** | System & user-installed certificates |

---

## Certificate Pinning
Certificate pinning enhances security by ensuring an app only connects to **specific trusted certificates**, reducing risks from fake certificates.

### How It Works
1. The app extracts the public key from the server certificate.
2. It calculates a **hash (digest)** of this key.
3. The hash is compared with predefined pinned hashes in the configuration.
4. If there’s a match, the connection is **allowed**; otherwise, it’s blocked.

### Example Configuration for Pinning
This configuration **pins certificates for owasp.org** (including subdomains):
```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">owasp.org</domain>
        <pin-set expiration="2025/12/31">
            <pin digest="SHA-256">YLh1dUR9y6Kja30RrAn7JKnbQG/uEtLMkBgFF2Fuihg=</pin>
            <pin digest="SHA-256">Vjs8r4z+80wjNcr1YKepWQboSIRi63WsWXhIMN+eWys=</pin>
        </pin-set>
    </domain-config>
</network-security-config>
```
- The **pin-set expiration date** ensures security updates when certificates change.

---

## Security Provider Updates
Android uses security providers (e.g., OpenSSL) for **SSL/TLS encryption**.
- Older versions may have **security flaws**.
- Since **July 2016**, Google **blocks Play Store apps** using vulnerable OpenSSL versions.
- Developers should update security providers to prevent attacks.

---

## Summary
- Android **Network Security Configuration** helps protect app communication.
- Apps should **disable cleartext traffic** and use **certificate pinning**.
- Developers must **update security providers** to avoid vulnerabilities.

---

## References
- [Android Developers - Network Security Configuration](https://developer.android.com/training/articles/security-config)
- [A Security Analyst’s Guide to Network Security Configuration in Android P](https://security.googleblog.com/2018/05/a-security-analysts-guide-to-network.html)
- [Android Codelab - Network Security Configuration](https://codelabs.developers.google.com/codelabs/android-network-security-config/)

---
