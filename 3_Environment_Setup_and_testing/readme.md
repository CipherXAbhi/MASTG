# Android-Pentesting-Environment-Setup
---

| No. | Topic                         | Description                           |
|----|--------------------------------|---------------------------------------|
| 1  | [Installing Linux OS](https://github.com/CipherXAbhi/Android-Pentesting-Environment-Setup/tree/main#installing-kali)            | I use Kali Linux for testing |
| 2  | [Installing Virtual Phone ](https://github.com/CipherXAbhi/Android-Pentesting-Environment-Setup/blob/main/README.md#installing-the-virtual-phone-for-testing)      | I use Genymotion for testing |
| 3  | [Installing Apktool](https://github.com/CipherXAbhi/Android-Pentesting-Environment-Setup#install-apktool-and-how-to-use-that-tool-in-the-apk-pentesting)             | For reverse engineering APK files|
| 4  | Install ADB              | communicate with an Android device for debugging, file transfers, and penetration testing|
| 5  | [Install Dex2jar tool](https://github.com/CipherXAbhi/Android-Pentesting-Environment-Setup#install-dex2jar-and-jd-gui) | onverting Dalvik Executable (DEX) files into Java Archive (JAR) files,          |
| 6  | [Install jd-gui tool](https://github.com/CipherXAbhi/Android-Pentesting-Environment-Setup#install-dex2jar-and-jd-gui)               | graphical decompiler for Java applications       |
| 7  | [Install JADX tool](https://github.com/CipherXAbhi/Android-Pentesting-Environment-Setup/blob/main/README.md#install-jadx)          | Additional security topics           |
| 8  | [Install DROZER](https://github.com/CipherXAbhi/Android-Pentesting-Environment-Setup/blob/main/README.md#drozer-setup)  | Android security assessment tool |

---
# Installing Kali
I installed Kali linux in the main system. you can also use the in the virtual machne.
### Step1: Download Kali linux ISO
Go to the official Kali Linux website:
👉 https://www.kali.org/get-kali/
Download the Kali Linux ISO for your system (64-bit recommended).

---

### 🔹 Step 2: Create a Bootable USB Drive
📌 Using Rufus (Windows)
+ Insert a USB drive (at least 8GB) into your computer.
+ Open Rufus (Download: https://rufus.ie).
+ Select your USB drive under "Device".
+ Click "SELECT" and choose the Kali ISO.
+ Set "Partition scheme" to MBR (for BIOS) or GPT (for UEFI).
+ Click "START" and wait for the process to complete.
---
### 🔹 Step 3: Boot from USB
+ Insert the Bootable USB into your system.
+ Restart your PC and enter BIOS/UEFI settings (Press F2, F12, DEL, or ESC).
+ Change boot priority to USB drive and save changes.
+ Restart, and Kali Linux installer should load.
---
### 🔹 Step 4: Install Kali Linux
+ Select "Graphical Install" (recommended).
+ Choose your language, location, and keyboard layout.
+ Set a hostname (e.g., kali).
+ Create a new user account (avoid using "root").
+ Set up a strong password.
---
### 🔹 Step 5: Disk Partitioning
You have two options:
1. Guided - Use Entire Disk (Recommended for beginners)
    + Select this if you want to erase everything and install Kali.
2. Manual Partitioning (For Dual Boot or Advanced Users)
    + Create a root (/) partition, a swap partition, and optionally a home (/home) partition.
  
---
### 🔹 Step 6: Install Kali Linux
+ Confirm partitions and select "Finish partitioning and write changes to disk".
+ The installation process will start.
+ When asked "Install GRUB bootloader?", select "Yes" and install it to your primary disk (e.g., /dev/sda).
--- 
### 🔹 Step 7: First Boot & Updates
+ Once installation is complete, remove the USB drive and reboot.
+ Login with your username and password.
+ Open a terminal and update Kali:
```
sudo apt update && sudo apt upgrade -y
```
---

# Installing the Virtual Phone for Testing
I use genymotion for Virtual Phone.
### Step1: Make your Account on the Genymotion.
### Step2: Download the Genymotion Desktop [software link.](https://dl.genymotion.com/releases/genymotion-3.8.0/genymotion-3.8.0-linux_x64.bin)
### Step3: Install the genymotion login and select personal use.
### How to Make virtual device in the genymotion.
![image](https://github.com/user-attachments/assets/df577cf3-9159-41ae-bbc2-dcd0ae5fd800)
![image](https://github.com/user-attachments/assets/ca46e139-7b39-47b7-bd54-dc219d4872a9)
![image](https://github.com/user-attachments/assets/bc207a86-6998-4266-9047-9f7bfe532614)
![image](https://github.com/user-attachments/assets/c8289426-d2a7-412e-bdc4-d88ab49d4b0d)
![image](https://github.com/user-attachments/assets/a2c6515e-42fa-49f2-b936-0deb01184fee)
![image](https://github.com/user-attachments/assets/2a8d69ea-8380-412c-943b-5ccd753f1a4a)
![image](https://github.com/user-attachments/assets/22852233-665c-4f5d-a5e3-135612300241)
### You Can see there has a new virtual Phone name Custome Phone
![image](https://github.com/user-attachments/assets/5ddca569-aa7b-4541-908d-6970c6e3f403)

### Click on the Start button
![image](https://github.com/user-attachments/assets/95df9872-c786-4fea-8d49-c3d245e1c5e0)

## Without root the device You can't test the applications in my case device root Ready.
### Step1: Install adb in the kali linux to check the device is connected or not.

![image](https://github.com/user-attachments/assets/74e1f77c-b2b3-4bc8-904a-4ca20120dff3)

### step2: check device is root or not 

![image](https://github.com/user-attachments/assets/d53190b1-aa77-4764-af08-b217979f8845)





---

# Install Apktool and how to use that tool in the apk pentesting
Type commmand on the terminal
```
sudo apt update
sudo apt install apktool
```

### Download any software.
### decompile that .apk with the use of apktool. Without this tool all content is in the encripted format.


![image](https://github.com/user-attachments/assets/14b04d81-8b87-4d0e-917c-bc0957262221)

### check the file is in the decripted format or not

![image](https://github.com/user-attachments/assets/c270825b-8488-4e5d-b464-9037806c5fab)

---
# Install dex2jar and jd-gui
run this command on the termainal
```
sudo apt update
sudo apt install dex2jar
```
run this command on the termainal
```
sudo apt update
sudo apt install jd-gui
```
---
# Install jadx
```
sudo apt install jadx
```
---
# Drozer Setup
run this command on the termainal
Step 1 : install Drozer Client
Step 2 : Install Drozer Agent

Install Drozer Client with this command
```
pipx install drozer
```

Download the Drozer Agent and Install in the virtual Phone.

```
https://github.com/WithSecureLabs/drozer-agent/releases/
```

## Step1: Open the Drozer Agent Application into the Android Virtual Phone and Click the Embedded Server ON.
![image](https://github.com/user-attachments/assets/0522fe97-4fcb-4384-ac66-cd9a51303606)
## Step2: Open the terminal then Forward the port.
## Step3: After that connect the console with drozer agent.
![image](https://github.com/user-attachments/assets/04943dfd-5093-402b-ab47-1346e2206864)

```
dz> ls
app.activity.forintent                   Find activities that can handle the given intent                                                                    
app.activity.info                        Gets information about exported activities.                                                                         
app.activity.start                       Start an Activity                                                                                                   
app.broadcast.info                       Get information about broadcast receivers                                                                           
app.broadcast.send                       Send broadcast using an intent                                                                                      
app.broadcast.sniff                      Register a broadcast receiver that can sniff particular intents                                                     
app.package.attacksurface                Get attack surface of package                                                                                       
app.package.backup                       Lists packages that use the backup API (returns true on FLAG_ALLOW_BACKUP)                                          
app.package.debuggable                   Find debuggable packages                                                                                            
app.package.info                         Get information about installed packages                                                                            
app.package.launchintent                 Get launch intent of package                                                                                        
app.package.list                         List Packages                                                                                                       
app.package.manifest                     Get AndroidManifest.xml of package                                                                                  
app.package.native                       Find Native libraries embedded in the application.                                                                  
app.package.shareduid                    Look for packages with shared UIDs                                                                                  
app.provider.columns                     List columns in content provider                                                                                    
app.provider.delete                      Delete from a content provider                                                                                      
app.provider.download                    Download a file from a content provider that supports files                                                         
app.provider.finduri                     Find referenced content URIs in a package                                                                           
app.provider.info                        Get information about exported content providers                                                                    
app.provider.insert                      Insert into a Content Provider                                                                                      
app.provider.query                       Query a content provider                                                                                            
app.provider.read                        Read from a content provider that supports files                                                                    
app.provider.update                      Update a record in a content provider                                                                               
app.service.info                         Get information about exported services                                                                             
app.service.send                         Send a Message to a service, and display the reply                                                                  
app.service.start                        Start Service                                                                                                       
app.service.stop                         Stop Service                                                                                                        
auxiliary.webcontentresolver             Start a web service interface to content providers.                                                                 
exploit.jdwp.check                       Open @jdwp-control and see which apps connect                                                                       
exploit.pilfer.general.apnprovider       Reads APN content provider                                                                                          
exploit.pilfer.general.settingsprovider  Reads Settings content provider                                                                                     
information.datetime                     Print Date/Time                                                                                                     
information.deviceinfo                   Get verbose device information                                                                                      
information.permissions                  Get a list of all permissions used by packages on the device                                                        
scanner.activity.browsable               Get all BROWSABLE activities that can be invoked from the web browser                                               
scanner.misc.native                      Find native components included in packages                                                                         
scanner.misc.readablefiles               Find world-readable files in the given folder                                                                       
scanner.misc.secretcodes                 Search for secret codes that can be used from the dialer                                                            
scanner.misc.sflagbinaries               Find suid/sgid binaries in the given folder (default is /system).                                                   
scanner.misc.writablefiles               Find world-writable files in the given folder                                                                       
scanner.provider.finduris                Search for content providers that can be queried from our context.                                                  
scanner.provider.injection               Test content providers for SQL injection vulnerabilities.                                                           
scanner.provider.sqltables               Find tables accessible through SQL injection vulnerabilities.                                                       
scanner.provider.traversal               Test content providers for basic directory traversal vulnerabilities.                                               
shell.exec                               Execute a single Linux command.                                                                                     
shell.send                               Send an ASH shell to a remote listener.                                                                             
shell.start                              Enter into an interactive Linux shell.                                                                              
tools.file.download                      Download a File                                                                                                     
tools.file.md5sum                        Get md5 Checksum of file                                                                                            
tools.file.size                          Get size of file                                                                                                    
tools.file.upload                        Upload a File                                                                                                       
tools.setup.busybox                      Install Busybox.                                                                                                    
tools.setup.minimalsu                    Prepare 'minimal-su' binary installation on the device.

```
### Get the package list.
![image](https://github.com/user-attachments/assets/533867d3-efa7-4bee-b4cd-f95b74ec7b39)

### If you find the particular package enter this command.
![image](https://github.com/user-attachments/assets/6fd4873f-e102-459b-a323-b4d179536573)

### Get the info of the package.
![image](https://github.com/user-attachments/assets/fbee3095-97c9-4fdb-8fa3-cab593f55b39)

### Get the menifest of the particular package
![image](https://github.com/user-attachments/assets/0dbdc724-7714-4d38-bf34-5c6284ddff64)

### get the fields where we attack with `**run app.package.attacksurface jakhar.aseem.diva**` this command
### get the info of the activity with `**run app.activity.info -a jakhar.aseem.diva**` this command
### start the activity where you attack for checking purpose `**run app.activity.start --component jakhar.aseem.diva jakhar.aseem.diva.APICredsActivity**`
![image](https://github.com/user-attachments/assets/369611d1-a8fb-4566-ad98-d86f3ae2556e)
![image](https://github.com/user-attachments/assets/1f5eb0a1-fb20-416b-bf17-d3286613fd26)

![image](https://github.com/user-attachments/assets/d9284a81-3868-49cb-93b7-cd516d9d9050)

![image](https://github.com/user-attachments/assets/1afd9fd5-9532-4870-b748-674fc5ba7d05)
![image](https://github.com/user-attachments/assets/f003c378-0927-44a8-b90f-cd03b1991975)
