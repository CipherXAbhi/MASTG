## General Concepts
### Mobile Application Taxonomy
When we say "mobile application" or "mobile app," we mean a computer program that runs on a mobile device. Right now, Android and iOS are the two biggest mobile operating systems, making up more than 99% of the market. More people now use the Internet on mobile devices than on desktops, making mobile apps the most common type of Internet-based software.


In this guide, we use the word "app" to describe any software that runs on a mobile OS. Apps can either run directly on a mobile platform, work through a web browser, or use a mix of both. In this chapter, we will organize mobile apps into different categories and explore the differences between them.

There are four categories of Mobile Application
1. Native Apps
2. Web Apps
3. Hybrid Apps
4. Progressive Web Apps

## 1. Native Apps
A native mobile app is an app built using a Software Development Kit (SDK) designed for a specific mobile operating system. If an app is called native, it means it was created using the standard programming languages for that OS—Swift or Objective-C for iOS and Java or Kotlin for Android.

### Why Native Apps?
Since native apps are built specifically for a platform, they provide fast performance, high reliability, and a smooth user experience. They follow platform-specific design rules, which make them look and feel more natural to users. Also, they have direct access to device features like the camera, sensors, and security hardware, which makes them more powerful than web or hybrid apps.

### Android’s Two Development Kits
Android has two types of development kits:
Android SDK – The standard kit for creating Android apps using Java or Kotlin.
Android NDK – A kit that allows developers to write parts of an app using C/C++, which can access low-level system features. Some native Android apps combine both SDK and NDK for better performance.

### The Downside of Native Apps
The biggest drawback of native apps is that they are limited to one platform. If a developer wants the app on both Android and iOS, they must maintain two separate codebases, which increases development time and effort.

### Cross-Platform Solutions
Some frameworks allow developers to write a single codebase and compile it for both Android and iOS, making development easier:

+ Xamarin
+ Google Flutter
+ React Native




## 2. Web Apps
### What Are Web Apps?
A web app is a website designed to look and work like a mobile app. Instead of installing it from an app store, you open it in a web browser. These apps are built using HTML5, just like modern websites. Some web apps even have launcher icons on the home screen, but these just open the app in a browser—similar to a bookmark.

### Limitations of Web Apps
Since web apps run inside a browser, they have limited access to a device’s features (like the camera or sensors) and usually don’t perform as well as native apps. Also, because they are made for multiple platforms, their design may not perfectly match any one operating system, like Android or iOS.

### Why Use Web Apps?
Web apps are cheaper and easier to develop because they use a single codebase that works on all devices. They also allow developers to update apps instantly without waiting for app store approvals. For example, updating an HTML file can immediately apply changes to all users, while updating a native app requires manual updates through an app store.




## 3. Hybrid Apps
### What Are Hybrid Apps?
Hybrid apps combine the features of both native and web apps. They run like native apps but rely on web technologies for most of their functions. A part of the app runs inside an embedded web browser (called WebView), allowing it to work across different platforms.

### Benefits of Hybrid Apps
+ They can access device features (like the camera and GPS), unlike regular web apps.
+ A single codebase can be used to create apps for both Android and iOS.
+ They can have a design that looks and feels like a native app.

### Limitations of Hybrid Apps
+ Since they rely on WebView, performance may not be as fast as fully native apps.
+ Some device features might not work as smoothly compared to native apps.

### Popular Hybrid App Frameworks
Developers use these frameworks to build hybrid apps:

Apache Cordova
+ Framework 7
+ Ionic
+ jQuery Mobile
+ Native Script
+ Onsen UI
+ Sencha Touch




## 4. Progressive Web Apps
What Are Progressive Web Apps (PWAs)?
Progressive Web Apps (PWAs) are web apps that look and work like mobile apps but run in a web browser. They use modern web technology to give a smooth and fast experience like native apps.

### Key Features of PWAs
+ Can be installed on a device like a regular app.
+ Work offline by storing data on the device.
+ Can access some device features, like the camera and GPS.

### Limitations of PWAs
+ Not all features work on iOS (e.g., Push Notifications, Face ID, and Augmented Reality).
+ Limited access to some advanced hardware features.
