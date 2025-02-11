# Mobile App Authentication Architectures

Authentication and authorization issues are common security risks, ranking high in the OWASP Top 10. Most mobile apps require user login, with part of the process handled by the backend. Since iOS and Android use similar authentication methods, this guide covers common approaches and mistakes. OS-specific topics like biometric login will be discussed separately.

## General Assumption
When testing authentication and authorization, always check if extra security steps (like two-factor authentication) are enforced properly.

A common mistake is relying on client-side data (like URLs or cookies) to confirm authentication. Hackers can easily change these values to gain access. For example, if a URL has authenticated=no, someone could change it to authenticated=yes and bypass security.

To prevent this, security experts recommend using server-side session storage or cryptographically signed tokens (like JWT). However, even these can have flaws, like JWT misconfigurations that let attackers disable signature verification.

---
## General Guidelines on Testing Authentication
There is no single best way to handle authentication. When reviewing an app’s authentication system, consider whether it fits the app’s needs. Authentication can be based on:
1. Something the user knows – like a password, PIN, or pattern.
2. Something the user has – like a SIM card, one-time password (OTP), or hardware token.
3. A biometric property – like a fingerprint, retina scan, or voice recognition.

The level of security needed depends on the app’s sensitivity.
1. For regular apps (like social media), a username and password with a good password policy is usually enough.
2. For sensitive apps (like banking or healthcare), adding a second security step (like OTP or biometrics) is recommended.

Some industries have strict security rules. For example:
1. Financial apps must follow standards like PCI DSS and SOX.
2. Healthcare apps in the U.S. must comply with HIPAA and the Patient Safety Rule.

---
# Stateful vs. Stateless Authentication
Most mobile apps use HTTP, which is a stateless protocol. This means the app needs a way to remember which user is making requests. There are two ways to do this:

## 1. Stateful Authentication (Session-Based)
+ When a user logs in, the server creates a session ID and stores user data on the server.
+ The app sends this session ID with every request.
+ When the user logs out, the server deletes the session.

## ✅ Best Practices for Stateful Authentication:
+ ✔ Use randomly generated session IDs.
+ ✔ Always send session IDs over HTTPS to prevent theft.
+ ✔ Store session IDs securely (not in permanent storage).
+ ✔ End the session properly when a user logs out.

## 2. Stateless Authentication (Token-Based, JWT)
+ Instead of storing session data on the server, the app stores it in a token (like a JSON Web Token or JWT).
+ The token is sent with every request and contains all necessary user data.
+ The server only needs to verify the token’s signature.

## ✅ Best Practices for Stateless Authentication:
+ ✔ Don't store sensitive data in the token.
+ ✔ Use strong encryption and signatures to prevent tampering.
+ ✔ Add an expiration time to the token.
+ ✔ Use secure storage (like Android Keystore or iOS Keychain).
+ ✔ Prevent attacks by using unique token IDs (jti) and verifying the audience (aud) claim.

> 📌 Example Attack: Some JWT libraries mistakenly accept tokens with no signature, which allows hackers to modify user roles (e.g., changing from user to admin).
---
## 🔍 Tools for Testing JWT Security:

> Burp Suite Plugins (JWT Attacker, JSON Web Tokens)

> OWASP JWT Cheat Sheet for security guidelines

---
## OAuth 2.0
OAuth 2.0 is a framework that allows third-party applications to access a user’s data on a remote service without needing to know their actual credentials. Instead of asking for a username and password, OAuth uses tokens to grant limited access to resources.

## Key Components of OAuth 2.0
1. Resource Owner: The user who owns the data.
2. Client: The application that wants to access the user’s data.
3. Resource Server: The server that holds the user's data.
4. Authorization Server: The server that authenticates the user and issues access tokens.

## OAuth 2.0 Workflow
1. User Authorization: The app asks the user for permission to access their account.
2. Authorization Grant: If the user agrees, the app gets a special code (authorization grant).
3. Token Request: The app sends this code to the authorization server to request an access token.
4. Token Issuance: If the request is valid, the server issues an access token.
5. Resource Access: The app uses the access token to request data from the resource server.
6. Data Response: If the token is valid, the resource server provides the requested data.

## Methods of OAuth Authentication
### 1. External User Agent (Browser-Based)
  + Uses an external browser (e.g., Chrome) to log in.
  + The app never sees the user’s credentials.
  + Used for logging into third-party apps (e.g., Google, Facebook login).

### 2. Embedded User Agent (App-Based)
  + Uses an in-app WebView or authentication library.
  + More secure for closed systems like banking apps.
  + Helps control the security measures within the app.

## OAuth 2.0 Best Practices
To ensure a secure implementation of OAuth 2.0, consider the following best practices:

### 1. User Agent Security
+ Users should have a way to verify they are interacting with a trusted service (e.g., TLS indicators, site verification).
+ Clients must validate the server's fully qualified domain name (FQDN) against the public key to prevent man-in-the-middle (MITM) attacks.

### 2. Grant Type Selection
+ Use Authorization Code Grant instead of Implicit Grant in native apps.
+ Implement PKCE (Proof Key for Code Exchange) to protect the Authorization Code Grant from interception attacks.
+ Ensure the authorization code is short-lived and used immediately. It should be stored in transient memory only (not in logs or files).

### 3. Client Secrets Management
+ Do not use shared secrets for client authentication in mobile/native apps since they can be extracted.
+ Instead, rely on client IDs as proof of identity.

### 4. End-User Credentials Protection
+ Always use TLS to securely transmit user credentials and prevent eavesdropping.

### 5. Token Security
+ Access tokens should be stored in transient memory and never written to disk.
+ Always transmit access tokens over encrypted connections (e.g., HTTPS).
+ Minimize scope and duration of access tokens, especially for sensitive data.
+ If using bearer tokens, ensure additional client verification mechanisms to prevent misuse.
+ Refresh tokens (long-lived credentials) should be stored securely, such as in Android Keystore or iOS Keychain.

## User Logout Best Practices in OAuth 2.0
Failing to properly implement logout functionality can lead to security risks, such as session hijacking. Here’s how to securely implement user logout:

## Common Logout Issues
1. Session Not Destroyed on Server:
  + If the session remains active after logout, attackers can use stolen credentials to maintain access.
2. Tokens Not Invalidated:
  + Stateless authentication systems relying on access and refresh tokens should ensure proper revocation.
3. Locally Stored Data Not Cleared:
  + Sensitive information (tokens, session IDs) must be removed from local storage.

## Best Practices for Secure Logout
1. Server-Side Session Termination
  + If the application uses server-side session storage, ensure the session is destroyed when a user logs out.
  + Example for different frameworks:
    + Spring (Java): request.getSession().invalidate();
    + Ruby on Rails: reset_session
    + PHP: session_destroy();
2. Token Revocation (Stateless Authentication)
  + If OAuth 2.0 tokens are used, ensure both access and refresh tokens are revoked:
    + Invalidate refresh tokens on the server to prevent re-authentication.
    + Use OAuth 2.0 Token Revocation Endpoint (/revoke) for compliant APIs.
3. Secure Token Deletion on Client-Side
  +  Delete access and refresh tokens from:
    + Memory
    + Secure storage (e.g., Keystore, Keychain)
    + Local storage or shared preferences (for mobile apps)
4. Remove Other Sensitive Data
  + Clear cache, cookies, and temporary data related to the user session.
  + Prevent session restoration by ensuring logout wipes all stored user-related data.
5. Implement Single Logout (SLO) for SSO
  + If using Single Sign-On (SSO), ensure logging out from one service logs out from all linked services.


---

## Supplementary Authentication¶
Authentication schemes are sometimes supplemented by passive contextual authentication ↗, which can incorporate:

+ Geolocation
+ IP address
+ Time of day
+ The device being used

Ideally, in such a system the user's context is compared to previously recorded data to identify anomalies that might indicate account abuse or potential fraud. This process is transparent to the user, but can become a powerful deterrent to attackers.

---

## Two-factor Authentication¶
Two-factor authentication (2FA) is standard for apps that allow users to access sensitive functions and data. Common implementations use a password for the first factor and any of the following as the second factor:

+ One-time password via SMS (SMS-OTP)
+ One-time code via phone call
+ Hardware or software token
+ Push notifications in combination with PKI and local authentication

Whatever option is used, it always must be enforced and verified on the server-side and never on client-side. Otherwise the 2FA can be easily bypassed within the app.

The 2FA can be performed at login or later in the user's session.

> For example, after logging in to a banking app with a username and PIN, the user is authorized to perform non-sensitive tasks. Once the user attempts to execute a bank transfer, the second factor ("step-up authentication") must be presented.

### Best Practices:

+ Don't roll your own 2FA: There are various two-factor authentication mechanisms available which can range from third-party libraries, usage of external apps to self implemented checks by the developers.
+ Use short-lived OTPs: A OTP should be valid for only a certain amount of time (usually 30 seconds) and after keying in the OTP wrongly several times (usually 3 times) the provided OTP should be invalidated and the user should be redirected to the landing page or logged out.
+ Store tokens securely: To prevent these kind of attacks, the application should always verify some kind of user token or other dynamic information related to the user that was previously securely stored (e.g. in the Keychain/KeyStore).

---
# Security Risks & Best Practices for SMS-Based OTP Authentication
SMS-based one-time passwords (OTP) are widely used for two-factor authentication (2FA) but come with security vulnerabilities. The National Institute of Standards and Technology (NIST) has warned against relying on SMS for authentication due to interception risks.

## ⚠ Threats to SMS-Based OTP

## 1️⃣ Wireless Interception
+ Attackers can intercept SMS messages using vulnerabilities in telecom networks, such as femtocells or SS7 protocol flaws.
+ This can expose OTPs to unauthorized individuals.

## 2️⃣ Malware & Trojans
+ Malicious apps with SMS read permissions can steal OTPs and forward them to attackers.
+ Android banking trojans often use this technique.

## 3️⃣ SIM Swap Attacks
+ An attacker convinces or bribes a telecom operator to transfer a victim’s number to a new SIM card they control.
+ Once successful, the attacker receives all OTPs and account recovery messages.

## 4️⃣ Verification Code Forwarding Attacks
+ A user receives an OTP and is tricked into forwarding it via social engineering.
+ Attackers impersonate customer support or other trusted entities to request the code.

## 5️⃣ Voicemail Exploitation
+ Some systems send OTPs via voice calls when SMS is unavailable.
+ If the call goes to voicemail, attackers can hack the voicemail account and retrieve the OTP.

## ✅ Best Practices for Secure OTP Authentication
## 1️⃣ Improve OTP Messaging
+ Include clear instructions in OTP messages:
  + Example: “If you didn’t request this code, ignore this message.”
+ Warn users that your company never asks them to share OTPs via call or SMS.

## 2️⃣ Use a Secure Delivery Channel
+ Prefer push notifications (APN on iOS, FCM on Android) over SMS.
+ A trusted mobile app can securely handle OTPs without exposing them to SMS-based threats.

## 3️⃣ Strengthen OTP Security
+ Use at least 6-digit OTPs with high entropy to prevent brute-force attacks.
+ Format OTPs in groups (e.g., 123-456) for easier recall and manual entry.

## 4️⃣ Disable Voicemail OTPs
+ Do not send OTPs via voicemail.
+ Require users to verify manually instead of relying on voice-based OTP delivery.

## 5️⃣ Detect & Prevent SIM Swap Attacks
+ Use SIM swap detection APIs from telecom providers.
+ Notify users if their SIM card or phone number is changed.

## 6️⃣ Implement Additional Authentication Layers
+ Combine OTP with biometric authentication (fingerprint, facial recognition).
+ Use FIDO2/WebAuthn for stronger, phishing-resistant authentication.

## 🔍 Alternative Secure Authentication Methods
1. Authenticator Apps (Google Authenticator, Microsoft Authenticator, Authy)
  + More secure than SMS, as codes are generated locally.

2. Hardware Security Keys (YubiKey, Titan Security Key)
  + Resistant to phishing and SIM swap attacks.

3. Passkeys (FIDO2, WebAuthn)
  + Passwordless authentication that provides stronger security.

--- 

## Transaction Signing with Push Notifications and PKI
Transaction signing with Push Notifications & PKI is a secure way to approve important actions. The app creates a public and private key when the user registers. When a transaction happens, the server sends a push notification asking for approval. The user confirms it by unlocking their phone (PIN/Fingerprint), and the app signs the request with the private key. The server checks the signature with the public key to verify it's genuine before processing the transaction. This method is safer than OTPs and protects against hacking attempts like SIM swaps and phishing. 

---
## Login Activity and Device Blocking (Simplified)
Apps should notify users about login activities and allow them to block unauthorized devices. Here are key points:

1. Login Alerts – Users should receive a push notification when their account is accessed on a new device. They can block unauthorized devices directly from the notification.

2. Session History – After login, users should see details of their last session, including location, device, and app version. If something looks suspicious, they should be able to report and block that device.

3. Audit Logs – A self-service portal should allow users to view login history and manage logged-in devices.

4. Tracking Information – The app should track and display key details like:
   + Device Name – Shows which devices have accessed the account.
  + Date & Time – Displays the last activity timestamp.
  + Location – Provides details of where the account was accessed.

5. Sensitive Activity Logs – Apps should track critical actions such as:
  + Login attempts
  + Password changes
  + Updates to personal details (email, phone number, etc.)
  + Important transactions (purchases, payments, etc.)

6. Security Measures – Users should be able to log out of specific sessions and, in some cases, permanently block devices.
