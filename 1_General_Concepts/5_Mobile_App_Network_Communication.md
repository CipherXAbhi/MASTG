# Mobile App Network Communication
Most mobile apps use HTTP or HTTPS to send and receive data over the internet. This makes them vulnerable to network attacks like packet sniffing and man-in-the-middle (MITM) attacks. This section covers common risks, testing methods, and best practices for securing app communications.

## Why Secure Connections Matter
Using plain HTTP is risky because data is sent without encryption. Instead, apps should use HTTPS, which is HTTP with an added security layer called Transport Layer Security (TLS). TLS ensures:

1. Confidentiality – Data is encrypted, so attackers can’t read it.
2. Integrity – Data can’t be altered without detection.
3. Authentication – The app verifies the server’s identity to prevent fake connections.

--- 

## Server Trust Evaluation
Certificate Authorities (CAs) help secure communication between apps and servers. Every operating system has a default trust store containing pre-approved CAs. For example, iOS includes over 200 root certificates (see Apple’s documentation).

However, new CAs can be added:

+ Manually by users
+ Through enterprise device management (MDM)
+ By malware

This raises a key question: Can you trust all CAs by default?
Some CAs have been hacked or tricked into issuing fake certificates, leading to security risks.

## Custom CA Trust
Both Android and iOS allow apps to override the default trust store and use specific CAs. Reasons for this include:

1. Connecting to a server with a self-signed or internal CA.
2. Restricting connections to a trusted CA list.
3. Allowing extra CAs not included by default.

---

## Trust Stores & Extending Trust
When an app connects to a server with a self-signed or unknown certificate, the connection fails because the system doesn’t trust it. Organizations like governments, corporations, or schools often use private CAs for internal services.

Both Android and iOS allow adding extra trusted CAs, but users can also install CAs manually. If security is a concern, apps may need to:

+ Avoid trusting user-added CAs
+ Trust only a specific certificate or CA

For most apps, the default trust settings are secure. However, for high-risk apps (like banking or healthcare), the possibility of a compromised CA must be considered.

## Restricting Trust: Identity Pinning
Some apps increase security by restricting which CAs they trust. Instead of trusting all system-approved CAs, they only allow specific ones. This is called Identity Pinning, often done through:

+ Certificate Pinning – Trusting a specific SSL certificate
+ Public Key Pinning – Trusting a server’s cryptographic key

Once a mobile app pins a server identity, it will only connect if the expected identity matches. This reduces the attack surface by preventing unauthorized CAs from being trusted.

## Best Practices for Pinning
+ Pin only trusted endpoints controlled by the developer.
+ Use a backup key to prevent connection issues if the CA changes.
+ Pin at development time using recommended methods for Android (Network Security Config) and iOS (App Transport Security).
+ Follow OWASP’s guidelines for proper implementation.

## Platform-Specific Recommendations
### Android:
Google cautions against pinning due to the risk of breaking connections if server certificates change. However, if implemented, developers must include a backup key to avoid issues.

### iOS:
Apple advises creating a long-term authentication strategy for managing pinned certificates.

## OWASP’s Recommendation
Pinning is recommended for high-security apps (MASVS-L2), but it should only be used for trusted endpoints under the developer’s control, with a backup key and update strategy.

---

## Cipher Suites Simplified
A cipher suite defines how data is securely exchanged over TLS. Its structure is:
Protocol_KeyExchangeAlgorithm_WITH_BlockCipher_IntegrityCheckAlgorithm

### Example: TLS_RSA_WITH_3DES_EDE_CBC_SHA
+ TLS → Protocol
+ RSA → Key Exchange Algorithm (for authentication)
+ 3DES_EDE_CBC → Block Cipher (for encryption)
+ SHA → Integrity Check Algorithm (for message authentication)

### Key Points About Cipher Suites in TLS 1.3
+ TLS 1.3 does not include the Key Exchange Algorithm in the cipher suite. Instead, it's negotiated separately during the handshake.
+ It enforces stronger security by using predefined secure cipher suites.

### Categories of Cipher Suite Components

**Protocols (Security levels vary)**
+ Deprecated: SSLv1, SSLv2, SSLv3, TLS 1.0, TLS 1.1
+ Recommended: TLS 1.2, TLS 1.3

**Key Exchange Algorithms (Used for authentication)**
+ RSA (Common in older versions)
+ ECDHE (Recommended for strong security)
+ DHE, PSK, ECDSA (Other options based on use case)

**Block Ciphers (Used for encryption)**
+ Recommended: AES_128_GCM, AES_256_GCM, CHACHA20_POLY1305
+ Avoid: DES, 3DES, RC4 (Weak and insecure)

**Integrity Check Algorithms (Used for verifying message integrity)**
+ SHA-256, SHA-384 (Secure options)
+ Avoid: MD5, SHA-1 (Vulnerable)

**Recommended Cipher Suites**
+ Check IANA and OWASP recommendations for the latest secure cipher suites.
+ Ensure compatibility by checking supported cipher suites for Android and iOS.

**How to Verify Server Cipher Suites**
+ nscurl (for iOS)
+ testssl.sh (command-line tool to check TLS settings and vulnerabilities)
+ Qualys SSL Labs Test (web-based tool for best practices verification)

**Final Best Practices**
+ Use TLS 1.2 or 1.3 only
+ Choose strong cipher suites (AES-GCM, CHACHA20)
+ Avoid deprecated algorithms (SSL, 3DES, RC4, MD5)
+ Test your server configuration regularly

---

## Intercepting HTTP(S) Traffic
One of the easiest ways to analyze a mobile app’s network traffic is by using a system proxy. This redirects HTTP(S) requests from the app through an interception proxy on your computer. By monitoring these requests, you can:

+ Map server-side APIs
+ Understand the app’s communication protocols
+ Modify and replay requests to test for vulnerabilities

### Popular Proxy Tools
+ Burp Suite (widely used for penetration testing)
+ OWASP ZAP (free and open-source alternative)

### How to Intercept Traffic
1. Run a proxy tool on your computer.
2. Set a system-wide proxy on the mobile device.
3. Apps using standard HTTP libraries (like okhttp) will automatically route traffic through the proxy.
4. Bypassing SSL/TLS verification – Installing the proxy’s CA certificate allows intercepting encrypted traffic.

## Intercepting Non-HTTP Traffic
Interception proxies like Burp Suite and OWASP ZAP primarily handle HTTP(S) traffic and won’t capture non-HTTP protocols by default. However, you can use Burp plugins like:

+ Burp-non-HTTP-Extension – Helps visualize non-HTTP traffic.
+ Mitm-relay – Allows intercepting and modifying non-HTTP data.
⚠️ Note: Setting up non-HTTP interception can be complex and requires additional configuration.

---

## Intercepting Traffic Within the App
Instead of performing a full MITM attack, you can monitor network data before it leaves the app or when responses are received. This is useful for detecting if sensitive data is being transmitted, without needing to bypass SSL pinning.

## How?
You can hook network functions such as:

+ SSL_write and SSL_read (OpenSSL) – To capture encrypted data before transmission.

This works well with standard API libraries but can be challenging if:

+ The app uses a custom network stack, requiring reverse engineering.
+ Reconstructing HTTP response pairs is difficult due to multithreading.

## Useful Tools & Techniques
+ Frida – Hooks network functions to inspect data.
+ Signature Analysis – Helps find OpenSSL traces in the app.
+ Ready-made scripts (though they may require maintenance).

---

## Intercepting Traffic on the Network Layer
Intercepting network traffic is easy when an app uses standard HTTP libraries. However, some cases make interception more difficult, such as:

+ Apps using Xamarin (ignores system proxy settings).
+ Apps detecting system proxies and refusing to send requests.
+ Intercepting push notifications (e.g., GCM/FCM on Android).
+ Non-HTTP protocols (like XMPP).

## Alternative Methods to Intercept Traffic
### 1️⃣ Route Traffic Through the Host Computer
+ Set up your computer as a network gateway (e.g., using Internet Sharing).
+ Use Wireshark to capture and analyze network packets.

### 2️⃣ Perform a MITM Attack
+ Use tools like Bettercap to force communication through your host.
+ Set up a rogue access point to redirect traffic.

### 3️⃣ Hook Network API Calls (Rooted Devices)
+ Use code injection to intercept network-related functions.
+ This allows capturing/manipulating data before encryption.

### 4️⃣ Sniff iOS Traffic (macOS Only)
+ Use Remote Virtual Interface (RVI) to monitor all traffic on an iOS device.

---
### 1️⃣ MITM with ARP Spoofing (Bettercap)
Prerequisites:
+ Kali Linux or Parrot OS
+ Bettercap installed (apt install bettercap)
+ Mobile device and host on the same network
+ Target device IP

**Attack Execution:**
```
sudo bettercap -eval "set arp.spoof.targets X.X.X.X; arp.spoof on; set arp.spoof.internal true; set arp.spoof.fullduplex true;"
```
🔹 This poisons the ARP cache of the target, making your system the gateway.<br>
🔹 Use Wireshark or Bettercap's built-in sniffer to analyze traffic.

### 2️⃣ MITM via a Fake Access Point
Prerequisites:
+ Kali Linux
+ External WiFi adapter (if needed)
+ hostapd, dnsmasq, iptables, wpa_supplicant installed

**Configuration Files:**
+ hostapd.conf (Access Point setup)
+ dnsmasq.conf (DHCP & DNS setup)
+ wpa_supplicant.conf (Connect host to real network)

**Commands to Launch the AP:**
```
airmon-ng check kill
ifconfig wlan1 10.0.0.1 up
hostapd hostapd.conf
wpa_supplicant -B -i wlan0 -c wpa_supplicant.conf
dnsmasq -C dnsmasq.conf -d
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables --flush
iptables --table nat --append POSTROUTING --out-interface wlan0 -j MASQUERADE
iptables --append FORWARD --in-interface wlan1 -j ACCEPT
iptables -t nat -A POSTROUTING -j MASQUERADE

```

---
## Network Traffic Analysis & Proxy Interception
**1. Network Analyzer Tools**
To monitor and analyze network traffic, use:
+ Wireshark (GUI-based) or TShark (CLI)
+ tcpdump (CLI-based)

These tools are available on all major Linux and Unix systems.

**2. Setting a Proxy via Runtime Instrumentation**
On rooted/jailbroken devices, you can manipulate network traffic using:
+ Inspeckage (Android)
+ Frida or cycript (for runtime hooking)

**3. Intercepting Xamarin App Traffic**
Xamarin apps ignore system proxy settings. Use one of these methods to intercept their traffic:

**Method 1: Modify App Code (Requires Source Access)**
Add this line in OnCreate or Main method and recompile:
```
WebRequest.DefaultWebProxy = new WebProxy("192.168.11.1", 8080);
```
**Method 2: MITM with Bettercap**
Redirect traffic to your interception proxy:
```
sudo iptables -t nat -A PREROUTING -p tcp --dport 443 -j DNAT --to-destination 127.0.0.1:8080
```
For macOS, use:
```
echo "rdr pass inet proto tcp from any to any port 443 -> 127.0.0.1 port 8080" | sudo pfctl -ef -
```
Enable "Support Invisible Proxy" in Android Studio listener settings.

**Method 3: Modify /etc/hosts on Mobile**
+ Add an entry in /etc/hosts pointing the target domain to your proxy’s IP.
+ Redirect port 443 as in Method 2.

**4. Configuring Burp Suite for Traffic Redirection**
+ Go to Proxy > Options > Edit Listener
+ Under Request Handling, set:
  + Redirect to host: (Original traffic location)
  + Redirect to port: (Original port)
  + Enable "Force use of SSL" and "Support Invisible Proxy"

**5. Installing CA Certificates**
For HTTPS interception, install your proxy’s CA certificate on the mobile device:
+ Android 7+ requires modifying the app to trust user-installed certificates.
+ Install the CA certificate on iOS for HTTPS interception.

**6. Start Intercepting Traffic**
+ Open the target app and trigger network requests.
+ Analyze traffic using Wireshark, bettercap, or Burp Suite.
