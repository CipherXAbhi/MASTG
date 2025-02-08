# #Mobile Penetration Testing Overview
Mobile app security testing involves evaluating an app’s security through static and dynamic analysis. It is often part of a broader security assessment that includes server-side APIs and backend systems.

## There are two key approaches:

+ Traditional Penetration Testing – Conducted near the end of development to identify vulnerabilities before release.
+ Continuous Security Testing – Integrated into the development lifecycle to detect and mitigate risks early.






---
## Testing Principles
Security testing follows three main approaches:

### Black-box Testing
+ The tester has no prior knowledge of the application.
+ Simulates a real-world attacker with no insider access.
+ Focuses on discovering vulnerabilities from an external perspective.
> Example: Testing an app like a hacker who has only public information.


### White-box Testing
+ The tester has full access to the app’s source code, documentation, and architecture.
+ Allows for deeper security analysis and faster vulnerability detection.
+ Helps find logical flaws, insecure coding practices, and hidden vulnerabilities.
> Example: A developer reviewing code for security weaknesses.


### Gray-box Testing
+ A mix of black-box and white-box testing.
+ The tester has partial knowledge of the app, such as credentials or API documentation.
+ Balances real-world attack simulation with efficient vulnerability detection.
> Example: A tester with user access trying to escalate privileges.







---
# #Vulnerability Analysis
Vulnerability analysis is the process of identifying security flaws in a mobile application. This can be done manually or through automated scanners. It is a critical step in mobile penetration testing and includes Static and Dynamic Analysis.

## Static Analysis (SAST)
Static Application Security Testing (SAST) examines an app’s source code, binaries, and configurations without running it. This helps identify issues like hardcoded credentials, weak encryption, and insecure API calls.

### Types of Static Analysis
+ Manual Code Review – Security experts analyze source code for vulnerabilities using tools like grep or integrated development environments (IDEs).
+ Automated Code Analysis – Tools scan the codebase to detect security weaknesses and compliance issues.

Common Issues Found in Static Analysis:
+ ✔ Hardcoded secrets (API keys, credentials)
+ ✔ Insecure cryptographic implementations
+ ✔ Weak permissions on exported components
+ ✔ Misconfigurations in manifest files

### Tools for Static Analysis:
+ 🔹 jadx – Decompile APKs and analyze source code.
+ 🔹 apktool – Reverse-engineer Android apps.
+ 🔹 MobSF – Automates static analysis for Android and iOS apps.
+ 🔹 Burp Suite & grep – Identify security flaws in code manually.

## Dynamic Analysis (DAST)
Dynamic Application Security Testing (DAST) examines an app while it is running to identify vulnerabilities that only appear during execution. This approach detects issues such as insecure data storage, authentication bypass, and runtime manipulations.

### How Dynamic Analysis Works:
1. Run the app on a rooted device or emulator.
2. Monitor behavior – Check how it handles sensitive data, API calls, and permissions.
3. Intercept and modify network traffic – Identify data leaks and insecure transmission.
4. Manipulate runtime execution – Use tools like Frida or Xposed to bypass security mechanisms.

Common Issues Found in Dynamic Analysis:
+ ✔ Insecure data storage (e.g., plaintext credentials in SharedPreferences)
+ ✔ API endpoint vulnerabilities (e.g., missing authentication, broken access control)
+ ✔ Weak SSL/TLS implementation allowing man-in-the-middle (MITM) attacks
+ ✔ Runtime protections that can be bypassed (e.g., root detection, debugging protections)

Tools for Dynamic Analysis:
+ 🔹 Frida – Instrumentation framework for runtime analysis and manipulation.
+ 🔹 Xposed – Modify app behavior at runtime.
+ 🔹 adb (Android Debug Bridge) – Execute commands and debug applications.
+ 🔹 drozer – Identify and exploit Android security flaws.
+ 🔹 AppUse VM – Pre-configured environment for mobile security testing.

## Avoiding False Positives in Automated Testing
Automated tools may flag false positives, meaning reported vulnerabilities that are not exploitable. This commonly happens with:
+ ❌ Cross-Site Request Forgery (CSRF) – Mobile apps rarely auto-submit session cookies.
+ ❌ Reflected XSS – WebViews may be vulnerable, but reflected XSS is less common in native apps.

How to Reduce False Positives:
+ ✔ Manually verify findings from automated scanners.
+ ✔ Focus on real attack scenarios rather than just tool-generated reports.
+ ✔ Combine static and dynamic analysis for a more accurate security assessment.

---











# #Penetration Testing (Pentesting) – Simple Explanation
Pentesting is a security test performed on a final or near-final version of an app to identify vulnerabilities before release. It follows a structured approach:

1. Preparation – Define what will be tested, set goals, and get legal permission. Testing without approval is illegal!
2. Information Gathering – Learn about the app's structure, environment, and data flow.
3. Mapping the Application – Identify entry points, possible weaknesses, and rank vulnerabilities by risk.
4. Exploitation – Act like a hacker and try to break into the app using the discovered vulnerabilities.
5. Reporting – Document the findings, explain the risks, and suggest fixes to improve security.

## Preparation Phase
### 1. Defining the Security Level
+ Decide how deeply the app will be tested.
+ Security needs differ for each organization based on industry and regulations.
+ Use the Mobile App Security Verification Standard (MASVS) to set testing guidelines.

### 2. Coordinating with the Client
+ Ensure a proper test environment is available.
+ Some security features (e.g., root detection, certificate pinning) can slow down testing.
+ Ideally, get two app versions:
  + Release build (to test real security controls).
  + Debug build (with security controls disabled for deeper testing).
+ Define whether testing will be black-box (no internal access) or white-box (full access).
  
### 3. Identifying Sensitive Data
Sensitive data varies by industry, but generally includes:
+ ✔ User credentials (passwords, PINs)
+ ✔ Personal data (SSN, credit card, bank details)
+ ✔ Device identifiers (that can track users)
+ ✔ Confidential business data (that can cause harm if leaked)
+ ✔ Legally protected information

Sensitive data exists in three states:
+ 📌 At rest (stored on the device)
+ 📌 In use (being processed in memory)
+ 📌 In transit (sent over a network)

### 4. Understanding Security Risks in Code
Some coding mistakes weaken security:
+ Weak random number generation (unsafe for encryption)
+ Improper hashing (hashing non-sensitive data isn’t an issue)
+ Confusing encoding with encryption (Base64 isn’t secure encryption)
+ Storing API tokens insecurely (can expose sensitive data)



## Information Gathering
Before testing an app's security, we need to collect information about how the app works and its environment. This helps in identifying possible security risks.

### 1. Environmental Information (Business Context)
This covers how the app fits into the organization, including:
+ 📌 App’s purpose – What the app does and how users interact with it.
+ 📌 Industry risks – Different industries (banking, healthcare, etc.) have different security concerns.
+ 📌 Key people – Knowing who owns, manages, and invests in the app.
+ 📌 Internal processes – How the company uses the app internally, which could create security loopholes.

### 2. Architectural Information (Technical Details)
This focuses on how the app is built and secured, including:
+ 📱 The App – How it handles data, manages user sessions, and reacts to rooted/jailbroken devices.
+ 🖥️ The Operating System – What OS (Android/iOS) the app supports and whether security policies like Mobile Device Management (MDM) are in place.
+ 🌐 Network Security – Whether the app uses strong encryption (TLS, SHA-2) and certificate pinning to secure data in transit.
+ 🔗 Remote Services – What external services the app connects to and whether a breach in these services could compromise security.



## Mapping the Application
Once a tester understands the app and its environment, the next step is to map out its structure. This means identifying:
+ 📌 Entry points – Where users (or attackers) can interact with the app.
+ 📌 Features – What the app can do.
+ 📌 Data flow – How data moves within the app.

How Mapping is Done
🔍 Using Internal Documents (White-box/Grey-box testing)

+ If available, project documents (like architecture diagrams, code, or functional specs) help testers understand the app faster.
+ Static Application Security Testing (SAST) tools analyze the source code to find vulnerabilities like SQL Injection.

⚙️ Using Automated Scanners (Black-box testing)
+ Dynamic Application Security Testing (DAST) tools scan the app in real-time to find security flaws.
+ These tools are fast, but they can’t find all vulnerabilities, so manual testing is still needed.

🛡️ Threat Modeling
+ This is a process to identify risks early in development.
+ It helps testers understand potential attack points, important assets, and the severity of vulnerabilities.
+ OWASP provides guidelines for threat modeling, which are useful for mobile apps.


## Exploitation
After finding possible vulnerabilities in the app, the next step is to test if they can actually be exploited. Not all vulnerabilities are dangerous—some might look serious but cause little harm, while others seem minor but can be very dangerous.

### How to Evaluate a Vulnerability?
Testers check vulnerabilities based on these five key factors:

+ 1️⃣ Damage Potential – How much harm can this vulnerability cause?
+ 2️⃣ Reproducibility – Is the attack easy to repeat?
+ 3️⃣ Exploitability – How simple is it to execute the attack?
+ 4️⃣ Affected Users – How many people could be impacted?
+ 5️⃣ Discoverability – How easy is it for attackers to find this weakness?

## Reporting
After testing, the security tester must document everything clearly so the client understands the results. A well-structured pentest report should include:

+ 📌 Executive Summary – A short overview for non-technical stakeholders.
+ 📌 Scope & Context – What was tested (e.g., specific systems, apps, or networks).
+ 📌 Methods Used – The approach and tools used for testing.
+ 📌 Sources of Information – Any data provided by the client or discovered during testing.
+ 📌 Findings (Prioritized) – List of vulnerabilities ranked by risk level (e.g., using DREAD classification).
+ 📌 Detailed Findings – Explanation of each issue, how it was found, and its impact.
+ 📌 Fix Recommendations – Steps to fix or mitigate each vulnerability.

---

# #Security Testing during the Software Development Life Cycle (SDLC)
Security Testing in the Software Development Life Cycle (SDLC)
Software development has changed over time, moving from Waterfall (step-by-step approach) to Agile (faster and more flexible). Earlier, security was handled separately, but now it must be built into the development process (DevSecOps).

Key Steps in Secure SDLC
+ 1️⃣ Risk Assessment – Identify security risks based on the app's data, functions, and regulatory requirements.

+ 2️⃣ Security Requirements – Define security rules early in development, including secure coding practices and compliance needs.

+ 3️⃣ Threat Modeling – Identify, analyze, and prioritize potential security threats before coding begins.

+ 4️⃣ Secure Development – Implement secure coding techniques, conduct code reviews, and use security tools like Static Analysis (SAST).

+ 5️⃣ Security Testing – Perform penetration testing (Pentests) and Dynamic Application Security Testing (DAST) to find vulnerabilities.

+ 6️⃣ Deployment & Monitoring – Ensure security controls remain effective when the app is released and in use.

+ 7️⃣ Decommissioning – Securely remove outdated software to prevent security risks.

