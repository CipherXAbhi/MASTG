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
  This is a well-detailed explanation of the Android Runtime Environment and Sandboxing. It highlights key concepts like:

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
