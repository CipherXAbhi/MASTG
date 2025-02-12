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
