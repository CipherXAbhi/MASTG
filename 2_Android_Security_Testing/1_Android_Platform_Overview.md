# Android Platform Overview
This chapter introduces the Android platform from an architecture point of view. The following five key areas are discussed:

1. Android architecture
2. Android security: defense-in-depth approach
3. Android application structure
4. Android application publishing
5. Android application attack surface

---
## Android Architecture
![image](https://github.com/user-attachments/assets/e1676dc2-6102-4e68-ac95-a1af4736e909)

### 1. Linux Kernel (Lowest Layer)
+ Acts as the foundation of Android.
+ Manages hardware (CPU, memory, device drivers, power management).
+ Provides security, process management, and networking.
+ Used for features like Bluetooth, Wi-Fi, camera, etc.

### 2. Hardware Abstraction Layer (HAL)
+ Acts as a bridge between the kernel and higher-level APIs.
+ Provides interfaces for hardware components (e.g., sensors, audio, camera).
+ Ensures that applications can interact with different hardware without modification.

### 3. Android Runtime (ART) & Native Libraries
+ Android Runtime (ART): Executes apps using Ahead-of-Time (AOT) compilation.
+ Native Libraries: Includes C/C++ libraries such as OpenGL (graphics), SQLite (database), WebKit (browser), and Media Frameworks.

1. Dalvik and ART
  + Older Android versions (pre-Android 5.0) used Dalvik Virtual Machine (DVM) with Just-In-Time (JIT) compilation.
  + Android Runtime (ART) replaced DVM from Android 5.0 onwards, offering Ahead-Of-Time (AOT) compilation, JIT, and Profile-Guided Compilation (PGC).

2. Compilation Process
  + Java/Kotlin code → Java bytecode (.class files) → Dalvik bytecode (.dex files via d8 tool) → Execution on ART.
  + AOT Compilation: Converts .dex files to native ELF binaries (.oat files) at installation for faster startup.
  + JIT Compilation: Compiles frequently used code at runtime.
  + Profile-Guided Compilation (PGC): Tracks frequently used code via JIT, then precompiles it during idle time.

3. App Sandboxing
  + Each app runs in its own virtual machine (sandbox).
  + Limits direct hardware access and prevents cross-app memory access, enforcing Android’s defense-in-depth security model.
  + Prevents resource monopolization and isolates malicious apps.

### 4. Application Framework
+ Provides APIs for developers to build Android apps.
+ Manages UI, app lifecycle, resources, location services, notifications, etc.
+ Key components: Activity Manager, Content Providers, Resource Manager, and Notification Manager.

### 5. Applications (Top Layer)
+ The user-facing apps installed on the device (System apps & third-party apps).
+ Runs on the application framework and uses system services.

---

## Android Security: Defense-in-Depth Approach
This is an in-depth explanation of Android Security: Defense-in-Depth Approach, covering four key security domains:

## 1. System-wide Security
+ Device Encryption
  + Full-Disk Encryption (FDE) (Android 5.0+): Encrypts the entire disk with a single key. Deprecated due to limitations (e.g., no alarms/calls before unlocking).
  + File-Based Encryption (FBE) (Android 7.0+): Encrypts files individually, supporting Direct Boot for essential services before unlocking.
  + Adiantum (Android 9+): Encryption for devices without AES hardware acceleration.

+ Trusted Execution Environment (TEE)
  + Hardware-backed KeyStore: Protects cryptographic keys in a secure environment.
  + StrongBox (Android 9+): A separate hardware security module for cryptographic operations.
  + GateKeeper: Manages password and pattern authentication within the TEE.

+ Verified Boot
  + Ensures only trusted software executes by verifying boot stages with a chain of trust starting from the hardware.

## 2. Software Isolation
+ Android Users and Groups
  + Each app runs as a separate Linux user (UID) for isolation.
+ SELinux (Security-Enhanced Linux)
  + Enforces Mandatory Access Control (MAC) to restrict unauthorized process actions.
+ Permissions System
  + Install-time permissions (pre-Android 6.0): All permissions granted at installation.
  + Runtime permissions (Android 6.0+): Users must approve sensitive permissions at runtime.

## 3. Network Security
+ TLS by Default (Android 9+)
  + Blocks cleartext traffic unless explicitly allowed in network_security_config.xml.
+ DNS over TLS (Android 9+)
  + Encrypts DNS queries to prevent interception and tampering.

## 4. Anti-Exploitation
+ ASLR, KASLR, PIE, and DEP
  + ASLR (Android 4.1+): Randomizes memory addresses to prevent exploits.
  + KASLR (Android 8.0+): Extends ASLR to the kernel.
  + PIE (Android 5.0+): Ensures apps can load at random memory locations.
  + DEP: Blocks execution of injected malicious code.
+ SECCOMP Filter (Android 8+)

Restricts system calls to prevent unauthorized kernel access.

---

# Android Security: Defense-in-Depth Approach
The Android security architecture follows a defense-in-depth strategy, ensuring that sensitive user data, applications, and system resources are protected by multiple layers of security rather than relying on a single defense mechanism. This approach minimizes risks by making it harder for attackers to compromise the system.

+ System-wide security
+ Software isolation
+ Network security
+ Anti-exploitation

## System-wide security
This breakdown highlights key security mechanisms in Android's encryption and trusted computing architecture. Here’s a concise summary of each concept and its significance:

### 1. Device Encryption
Android provides encryption to protect user data against unauthorized access:

  + Full-Disk Encryption (FDE) (Android 5.0+): Encrypts the entire disk with a single key. Drawbacks: Device needs to be unlocked after reboot to function properly. Now deprecated.
  + File-Based Encryption (FBE) (Android 7.0+): Encrypts files individually using different keys. Advantage: Enables Direct Boot, allowing essential functions (like alarms) to work before unlocking.
  + Adiantum (Android 9.0+): Alternative encryption method for devices lacking AES hardware acceleration (used in low-end devices).

### 2. Trusted Execution Environment (TEE)
Android uses dedicated hardware to protect cryptographic keys and sensitive operations:

  + Hardware-backed KeyStore: Protects cryptographic keys and allows secure encryption for apps.
  + StrongBox (Android 9.0+): Dedicated hardware security module with its own CPU, storage, and TRNG, ensuring high security.
  + GateKeeper: Handles password/pattern authentication securely inside the TEE to prevent brute-force attacks.

### 3. Verified Boot
  + Ensures the device boots only with trusted software from the OEM.
  + Uses cryptographic verification at every boot stage, starting from the hardware Root-of-Trust (RoT).
  + Prevents tampering and malware infections that modify the system boot process.

---

## Software Isolation

## 1. Android Users and Groups (Linux-Based Isolation)
+ Unlike traditional Unix-like systems, Android assigns each app a unique Linux user ID (UID) to isolate it.
+ System processes have predefined UIDs and groups to restrict privileges:
  + AID_ROOT (0) → Root user (not typically accessible in standard Android use).
  + AID_SYSTEM (1000) → System server, responsible for key OS functions.
  + AID_SHELL (2000) → Used for ADB and debugging.
  + AID_APP_START (10000+) → Assigned to each installed app, isolating them from others.

📌 Key Benefit: Apps cannot access each other’s files unless explicitly allowed.

## 2. SELinux (Mandatory Access Control)
+ Uses labels and policies (user:role:type:mls_level) to restrict access to system resources.
+ Enforces least privilege—each process only gets the permissions it needs.
+ Prevents privilege escalation and lateral movement in case an app is compromised.

🔐 Example:

+ A media player app can read media files but cannot delete system files.
+ A malicious app exploiting a vulnerability will have limited impact due to strict SELinux policies.

## 3. Android Permissions (Access Control)
+ Controls how apps interact with sensitive data and system features.
+ Two major models:
  + Install-Time Permissions (Pre-Android 6.0): Users grant all requested permissions at installation.
  + Runtime Permissions (Android 6.0+): Users approve permissions only when needed (e.g., accessing the camera).

📌 Key Security Improvement:

Malware can no longer gain excessive privileges by tricking users during installation.

## Defense-in-Depth Approach

This table summarizes the different security layers in a Defense-in-Depth approach and their purposes.

| Security Layer          | Purpose                                                    | Example                                      |
|-------------------------|------------------------------------------------------------|----------------------------------------------|
| **User Isolation (UIDs)** | Sandboxes each app as a separate Linux user              | One app cannot access another app’s data     |
| **SELinux (MAC)**        | Restricts process capabilities beyond traditional Linux permissions | Even if an app is exploited, it cannot gain system privileges |
| **Permissions System**   | Ensures apps only access approved resources               | Camera access prompt in Android 6.0+        |


---

## 🌐 Network Security
### 1. TLS by Default (Android 9+)
+ Since Android 9 (API 28), all apps must use TLS (Transport Layer Security) for network communication.
+ Why? It prevents eavesdropping, data tampering, and MITM attacks.
+ Exceptions? Apps can opt-in for cleartext (HTTP) traffic using network_security_config.xml, but this is not recommended.

📌 Key Takeaway: Apps using HTTP instead of HTTPS are blocked by default.

🔗 More info: Android TLS by Default

### 2. DNS over TLS (Android 9+)
+ Ensures that DNS queries (which map domain names to IPs) are encrypted, preventing DNS spoofing and eavesdropping.
+ How? The system establishes a TLS-encrypted channel between the device and a DNS resolver.
+ Default behavior: Enabled on supported networks unless manually turned off.

📌 Key Takeaway: Prevents ISP snooping and MITM attacks on DNS queries.

🔗 More info: Android DNS over TLS

---

## 🛡️ Anti-Exploitation Techniques
### 1. ASLR, KASLR, PIE, and DEP
These techniques make memory-based attacks harder by randomizing memory addresses and preventing malicious code execution.

🔹 ASLR (Address Space Layout Randomization) – Since Android 4.1 (API 15)<br>
  + Randomizes memory addresses for apps and OS components, making buffer overflow exploits unreliable.

🔹 KASLR (Kernel ASLR) – Since Android 8.0 (API 26)<br>
  + Extends ASLR to randomize the kernel’s memory layout, making kernel exploits harder.

🔹 PIE (Position Independent Executable) – Since Android 5.0 (API 21)<br>
  + Ensures that executables can be loaded at random memory locations (mandatory for apps in Android 5+).

🔹 DEP (Data Execution Prevention)<br>
  + Prevents executing code in memory regions meant for data storage (e.g., stack, heap).
  + Protects against stack-based buffer overflow attacks.

📌 Key Takeaway: Makes exploiting memory vulnerabilities significantly harder.

🔗 More info: Android Memory Protections

## 2. SECCOMP Filters (Android 8+)
+ Limits system calls (syscalls) apps can make to reduce attack surface.
+ Why? Many exploits rely on abusing unused or restricted syscalls to gain privileges.
+ How? SECCOMP blocks risky system calls that apps should never need to use.

📌 Key Takeaway: Prevents apps from exploiting dangerous syscalls to gain unauthorized access to the system.

# 📌 Summary: Android’s Defense-in-Depth

This table highlights key security features in Android's Defense-in-Depth approach, their purposes, and when they were introduced.

| Security Feature      | Purpose                                       | Introduced in              |
|-----------------------|-----------------------------------------------|----------------------------|
| **TLS by Default**    | Enforces HTTPS for secure communication       | Android 9 (API 28)        |
| **DNS over TLS**      | Encrypts DNS queries to prevent spying        | Android 9 (API 28)        |
| **ASLR & KASLR**      | Randomizes memory locations to stop exploits  | ASLR: Android 4.1 / KASLR: Android 8 |
| **PIE**              | Ensures apps run in randomized memory          | Android 5.0 (API 21)      |
| **DEP**              | Blocks execution of injected code              | Built-in                  |
| **SECCOMP Filters**   | Restricts dangerous system calls              | Android 8.0 (API 26)      |


---

# 📱 Android Application Structure & OS Communication

Android apps interact with the OS using the **Android Framework**, which provides high-level Java APIs that communicate with system services. Let's break it down:

---

## 🔗 Communication with the OS  
- Apps **do not interact directly** with the kernel but use system services through Java APIs.  
- **Inter-Process Communication (IPC)** is used to communicate with system services running in the background.  

### Examples of System Services:
✅ Connectivity: Wi-Fi, Bluetooth, NFC  
✅ File Access  
✅ Camera & Microphone  
✅ Geolocation (GPS)  
✅ Cryptographic Functions  

📌 **Key Takeaway:** Apps use high-level APIs to communicate with system services rather than directly accessing hardware.

---

## 🗂️ Noteworthy API Versions & Security Updates  
Each Android version introduces **security improvements, API changes, and privacy features**.  

| Version    | API Level | Release Date | Notable Security Changes |
|-----------|-----------|-------------|--------------------------|
| Android 4.2  | 16  | Nov 2012  | Introduced SELinux |
| Android 4.3  | 18  | Jul 2013  | SELinux enabled by default |
| Android 4.4  | 19  | Oct 2013  | Android Runtime (ART) introduced |
| Android 5.0  | 21  | Nov 2014  | ART becomes default, improved memory protections |
| Android 6.0  | 23  | Oct 2015  | Runtime Permissions introduced |
| Android 7.0  | 24-25 | Aug 2016  | New JIT compiler for ART |
| Android 8.0  | 26-27 | Aug 2017  | Many security improvements, SECCOMP filter |
| Android 9    | 28  | Aug 2018  | TLS by default, background camera/mic restrictions |
| Android 10   | 29  | Sep 2019  | Scoped storage, better location privacy |
| Android 11   | 30  | Sep 2020  | Permissions auto-reset, APK Signature v4 |
| Android 12   | 31-32 | Aug 2021  | Privacy Dashboard, Web Intent Security |
| Android 13   | 33  | 2022  | Safer broadcast receivers, new photo picker |
| Android 14   | 34  | 2023  | Stronger security & privacy enhancements |

📌 **Key Takeaway:** Each Android version adds stronger security mechanisms to protect apps and user data.

🔗 **More info:** [Android Security Releases](https://source.android.com/security/bulletin)

---

## 🍩 Android Version Codenames  
Android versions were **named after desserts** in alphabetical order until Android 10.  

**Examples:**  
- **Android 1.5** → Cupcake  
- **Android 1.6** → Donut  
- **Android 4.4** → KitKat  
- **Android 9** → Pie  
- **Android 10+** stopped using dessert names publicly.  

🔗 **More info:** [Android Version History](https://en.wikipedia.org/wiki/Android_version_history)

---

## 📌 Summary: How Android Apps Work with the OS  
✔️ Apps interact with system services via the **Android Framework (Java APIs)**.  
✔️ **SELinux & security policies** restrict direct access to the kernel.  
✔️ **New Android versions** introduce stronger security mechanisms like runtime permissions, TLS enforcement, and stricter app isolation.  
✔️ **Android API levels** define which features and security patches are available.  


---

# 📦 Android App Sandbox, UIDs, and Process Management

Android isolates apps for security using **sandboxing, unique UIDs, and process management**. Here's how it works:

---

## 🔒 App Sandbox: Secure Isolation of Data  
- Each installed app runs in a **separate sandbox**, preventing unauthorized access.  
- Apps store their data in `/data/data/[package-name]`, accessible **only to their process**.  
- **Linux permissions** restrict access to other apps.  

📌 **Example: File system permissions**  

```bash
drwx------  4 u0_a97  u0_a97  4096  com.android.calendar
drwx------  6 u0_a120 u0_a120 4096  com.android.chrome
```

🔹 **Key Takeaway:** Apps **can’t read or write** each other’s data unless explicitly allowed.

---

## 🔑 Bypassing Sandboxing (Shared User ID)  
- Apps **signed with the same certificate** and configured to use the same `sharedUserId` can **share data**.  

📌 **Example: NFC App Sharing Data**  

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.android.nfc"
    android:sharedUserId="android.uid.nfc">
```

🔹 **Risk:** This can be **exploited** if apps are **compromised or misconfigured**.

---

## 👤 Linux User Management & App UIDs  
- Each app is assigned a **unique UID (User ID)** for isolation.  
- UIDs typically range between **10,000 and 99,999**.  
- Android **maps UIDs to usernames** (e.g., `u0_a188` for UID `10188`).  

📌 **Example: Checking App UID & Group Permissions**  

```bash
$ id
uid=10188(u0_a188) gid=10188(u0_a188) groups=10188(u0_a188),3003(inet),
9997(everybody),50188(all_a188)
```

🔹 **Key Takeaway:** **Group IDs (GIDs) determine app permissions**.

📌 **Example: GID Mapping (`platform.xml`)**  

```xml
<permission name="android.permission.INTERNET">
    <group gid="inet" />
</permission>
```

🔹 **Impact:** Apps granted **INTERNET permission** join the `inet` group.

---

## 🏗 Zygote: The App Launcher  
- The **Zygote process** starts at **boot** and listens for app launch requests.  
- Instead of **starting a new app from scratch**, **Zygote forks itself**, reducing load time.  
- Zygote ensures **faster app startup** and **shared memory for common resources**.  

📌 **Process Creation:**  
1️⃣ **Zygote listens** on `/dev/socket/zygote`.  
2️⃣ A **new app request arrives**.  
3️⃣ **Zygote forks** a child process.  
4️⃣ The **new process loads** and runs the app.  

🔹 **Key Takeaway:** **Zygote speeds up app launches** by preloading system libraries.

---

## 🔄 App Lifecycle & Process States  
The Android OS manages app processes based on importance.  

| **State**     | **Description**                     |
|--------------|---------------------------------|
| Foreground   | Active app (highest priority). |
| Visible      | Partially visible UI elements. |
| Service      | Background tasks (e.g., music, sync). |
| Cached       | Unused, can be killed if needed. |

📌 **Example: System Killing Processes**  
- **High-priority** processes are kept alive.  
- **Low-priority (cached)** processes may be terminated when memory is low.  

🔹 **Key Takeaway:** Android efficiently manages memory by dynamically **killing and restarting apps**.

---

## 🚀 Summary: Android Security Mechanisms  
✔ **Sandboxing** isolates apps from each other.  
✔ **Unique UIDs** prevent direct data access between apps.  
✔ **Zygote process** optimizes app startup.  
✔ **Linux groups & permissions** define access control.  
✔ **App lifecycle management** ensures smooth performance.  

---
