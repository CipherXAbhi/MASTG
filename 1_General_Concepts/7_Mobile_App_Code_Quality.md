# Mobile App Code Quality
Mobile app developers use various programming languages and frameworks, which can lead to common security issues like SQL injection, buffer overflows, and XSS if secure coding practices are ignored.

Both Android and iOS apps can have similar flaws, so this guide first covers common vulnerabilities. Later sections will focus on OS-specific issues and mitigation techniques.

---

## Injection Flaws
Injection flaws occur when user input is inserted into backend queries or commands without proper validation. Attackers can exploit this to execute malicious code, such as manipulating SQL queries to access or modify database records.

While these vulnerabilities are common in server-side web services, they are less frequent in mobile apps due to a smaller attack surface. For instance, mobile apps using local SQLite databases typically avoid storing sensitive data, making SQL injection less effective. However, input validation remains essential to prevent potential exploitation.

---

## SQL Injection
SQL injection occurs when an attacker embeds SQL commands into input fields to manipulate database queries. This can allow unauthorized access, data modification, or even administrative actions, depending on server permissions.

Example: SQL Injection in an Android App

Consider an Android app that stores user credentials in a local SQLite database (a poor practice). The login query might look like this:
```
SQLiteDatabase db;
String sql = "SELECT * FROM users WHERE username = '" + username + "' AND password = '" + password + "'";
Cursor c = db.rawQuery(sql, null);
return c.getCount() != 0;

```
If an attacker enters the following input:
```
username = 1' or '1' = '1
password = 1' or '1' = '1

```
The query becomes:
```
SELECT * FROM users WHERE username='1' OR '1' = '1' AND password='1' OR '1' = '1'
```
Since '1' = '1' is always true, the query returns all records, bypassing authentication.

**Real-World Examples**
Ostorlab exploited SQL injection in Yahoo’s weather app via the sort parameter.
Mark Woods found SQL injection vulnerabilities in the Qnotes and Qget apps on QNAP NAS devices, exposing stored credentials.

## XML Injection
XML injection involves injecting XML meta-characters to manipulate an XML structure. Attackers can alter app logic, exploit XML parsers, or trigger malicious behaviors.

**XML External Entity (XXE) Attack**
A dangerous variant, XXE injection, allows attackers to access local files, send HTTP requests, launch CSRF attacks, or cause denial-of-service (DoS).

**Example XXE Payload:**
```
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [  
  <!ELEMENT foo ANY >
  <!ENTITY xxe SYSTEM "file:///dev/random" >]>
<foo>&xxe;</foo>
```
This payload forces the XML parser to access /dev/random, leading to an infinite data stream and potential DoS.

Prevention
Though XML is becoming less common with the rise of REST/JSON services, developers should always validate input and escape meta-characters when using XML-based queries, especially in parsers like NSXMLParser on iOS.

## Injection Attack Vectors
Mobile apps have a different attack surface than web and network applications. They rarely expose network services, and user interface-based attacks are uncommon. Instead, injection attacks typically occur through inter-process communication (IPC), where a malicious app targets another app on the same device.

**Identifying Injection Vulnerabilities**
Two main approaches help locate potential flaws:

1. Tracing Untrusted Input – Identify entry points and track data flow to vulnerable functions.
2. Analyzing Dangerous API Calls – Locate risky functions (e.g., SQL queries) and check if they process unchecked input.

**Common Entry Points for Untrusted Data**
+ IPC calls
+ Custom URL schemes
+ QR codes
+ Files received via Bluetooth, NFC, etc.
+ Pasteboards (clipboard)
+ User interface input

**Best Practices for Mitigation**
✅ Validate all untrusted input (check data type and enforce allowlists).<br>
✅ Use parameterized queries to prevent SQL injection.<br>
✅ Disable external entity resolution when parsing XML to prevent XXE attacks.<br>
✅ Use secure x509 parsers, as older Bouncy Castle versions (<1.6) allow remote code execution.

---
## Cross-Site Scripting (XSS) Vulnerabilities
**Overview**
XSS vulnerabilities allow attackers to inject malicious client-side scripts into web pages viewed by users. This enables bypassing the same-origin policy, leading to exploits like:<br>
✔ Stealing session cookies<br>
✔ Logging key presses<br>
✔ Performing unauthorized actions

**🛑 Risk in Mobile Apps**
While native apps are less prone to XSS, WebView components (Android's WebView, iOS's WKWebView) can still be exploited.

🔹 Example: Skype iOS XSS (Phil Purviance)
+ Skype failed to encode the sender's name properly
+ Allowed JavaScript injection
+ Exploit: Stealing user’s address book

## Static Analysis - Identifying XSS in Mobile Apps
🕵️ Key areas to check:<br>
1️⃣ WebView loading user-controlled input

+ Example (Vulnerable Code - Java & Kotlin):
```
webView.loadUrl("javascript:initialize(" + myNumber + ");");
```
```
webView.loadUrl("javascript:initialize($myNumber);")
```

2️⃣ Overridden methods handling URLs

+ Example (Vulnerable Code - Java & Kotlin):
```
@Override
public boolean shouldOverrideUrlLoading(WebView view, String url) {
  if (url.substring(0,6).equalsIgnoreCase("yourscheme:")) {
    // Unsafe parsing of URL
  }
}
```
```
fun shouldOverrideUrlLoading(view: WebView, url: String): Boolean {
    if (url.substring(0, 6).equals("yourscheme:", ignoreCase = true)) {
        // Unsafe parsing of URL
    }
}
```
🔹 Example Exploitation (Quora Android - Sergey Bobrov)<br>

**💻 ADB Command Exploit:**
```
$ adb shell
$ am start -n com.quora.android/com.quora.android.ActionBarContentActivity \
-e url 'http://test/test' -e html 'XSS<script>alert(123)</script>'
```

**📋 Clipboard Data Injection:**
```
$ am start -n com.quora.android/com.quora.android.ModalContentActivity  \
-e url 'http://test/test' -e html \
'<script>alert(QuoraAndroid.getClipboardData());</script>'
```

**📲 3rd Party Intent Exploit:**
```
Intent i = new Intent();
i.setComponent(new ComponentName("com.quora.android",
"com.quora.android.ActionBarContentActivity"));
i.putExtra("url","http://test/test");
i.putExtra("html","XSS PoC <script>alert(123)</script>");
view.getContext().startActivity(i);
```

## Best Practices to Prevent XSS in Mobile Apps
**✅ Secure WebView Implementation**
+ Never render untrusted data in WebView
+ Sanitize & encode input before injecting into JavaScript

**✅ HTML & JavaScript Encoding**

| Character | Escaped Value |
|-----------|--------------|
| &         | `&amp;`       |
| <         | `&lt;`        |
| >         | `&gt;`        |
| "         | `&quot;`       |
| '         | `&#x27;`      |
| /         | `&#x2F;`      |

🔹 Refer to: OWASP XSS Prevention Cheat Sheet

Dynamic Analysis - Testing for XSS
## ✅ Input Fuzzing

+ Inject HTML tags & special characters into input fields
+ Verify output sanitization & escaping
## ✅ Reflected XSS Testing

+ Check if the app returns injected scripts in responses
+ Use tools like Burp Suite Scanner for automated XSS detection

---

## Memory Corruption Bugs
Memory corruption bugs occur when a program accesses unintended memory locations due to programming errors. Attackers exploit these flaws to hijack execution and execute malicious code. Common types include:

+ Buffer Overflow: Writing beyond allocated memory can overwrite critical data, potentially leading to code execution.
+ Out-of-Bounds Access: Incorrect pointer arithmetic causes access to unintended memory, leading to crashes or exploits.
+ Dangling Pointers & Use-After-Free: Using memory references after deallocation can cause unpredictable behavior or security breaches.
+ Integer Overflows: Arithmetic operations exceeding defined integer limits can result in unintended values, leading to buffer overflows.
+ Format String Vulnerabilities: Unchecked input in format functions (e.g., printf) allows attackers to read/write memory arbitrarily.

## Exploitation & Prevention
Attackers often use Return-Oriented Programming (ROP) to bypass security protections like Data Execution Prevention (DEP). While Java-based Android apps are generally safe, native apps using C/C++ (via JNI) remain vulnerable. iOS apps can also be at risk if they wrap C/C++ calls.

**Example of Buffer Overflow:**
```
void copyData(char *userId) {  
    char smallBuffer[10];  
    strcpy(smallBuffer, userId); // No bounds check!  
}
```
Unsafe string functions like strcpy, sprintf, and gets should be avoided.

## Secure Coding Best Practices
+ Use safe alternatives (strncpy, snprintf) and string classes in C++.
+ Perform bounds checking for array indexing and buffer sizes.
+ Prevent integer wrapping by using unsigned integers and precondition tests.
+ Avoid concatenating untrusted input into format strings.

## Security Testing Approaches
+ Static Analysis: Tools like RATS can detect basic flaws, but complex issues (e.g., race conditions) require deeper analysis.
+ Dynamic Analysis (Fuzzing): Malformed input is fed into an application to detect vulnerabilities through crashes and unexpected behavior.

---

## Binary Protection Mechanisms
### Position Independent Code (PIC) & Position Independent Executables (PIE)
+ PIC: Code that runs correctly regardless of its memory address, commonly used in shared libraries.
+ PIE: Executables built entirely from PIC, enabling ASLR (Address Space Layout Randomization) to randomize memory locations and prevent predictable exploits.

### Memory Management
+ Automatic Reference Counting (ARC) (Objective-C/Swift):
  + Frees memory when objects are no longer needed.
  + Does not automatically handle reference cycles, which can cause memory leaks.

+ Garbage Collection (GC) (Java/Kotlin/Dart):
  + Reclaims memory by removing objects no longer in use.
  + Used in Android runtime (ART) with improvements over traditional GC.

+ Manual Memory Management (C/C++):
  + Developers manually allocate and free memory.
  + Incorrect use can cause memory leaks and security vulnerabilities.

**Stack Smashing Protection**
+ Stack Canaries:
  + A hidden integer is placed before the return pointer in a function.
  + If overwritten (e.g., by a buffer overflow attack), the program detects the corruption and prevents exploitation.
