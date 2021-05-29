# Android Forensics
***

## Training Layout:
- (Maybe general mobile forensics pointers?)
- Android Architecture (hardware/software, security, boot process, etc.)
- Android Data storage (partition layout, file hierarchy, file system, etc.)
- Extracting an image (Logical, Physical, ADB, Rooting, etc.)
  - Cellebrite (Explanation and Usage)
- Analyzing Extractions (Autopsy)
  - Possibly tie into or point to a separate autopsy guide?
- Analyzing APKs (Android SDK, APK components, etc.)
- Wishlist: LiME RAM extraction working....
  - If accomplished, practice on Linux VMs
***

## Android Architecture
The Android Operating System consists of a stack of layers running on top of each other. The components are listed below:

- **The Linux Kernel**: Android OS is built on top of the Linux Kernel with some architectural changes made by Google. The Linux Kernel is positioned at the bottom of the software stack and provides a level of abstraction between device hardware and the upper layers. It also acts as an abstraction layer between the software and hardware present on the device. Contains the following common components: process management, memory management, security, networking, device drivers, and binder drivers.
- **Libraries**: On top of the Linux kernel are Android's native libraries. It is with the help of these libraries that the device handles different types of data. These libraries are written in C or C++ and are specific to a particular hardware. Library examples include: SQLite, WebKit, OpenGLES, Free Type, Surface Manager, Media Framework, SSL, SGL, and libc. Also considered part of the Libraries layer are the Android Runtime Core Libraries and the Dalvik Virtual Machine.
  - **Dalvik Virtual Machine**: Android applications are programmed in Java. When Java is compiled, it produces byte code. A Java Virtual Machine (JVM) can execute the byte code. In the case of Android, this Java byte code is further converted to Dalvik byte code by the dex compiler. This Dalvik byte code is then fed into the Dalvik Virtual Machine (DVM) which can read and use the code. Thus, the `.class` files from the Java compiler are converted to `.dex` files using the dx tool. Dalvik byte code is an optimized byte code suitable for low-memory and low-processing environments. Each Android application runs its own instance of the DVM.
    - **NOTE**: Since Android 5.0, Dalvik has been replaced by Android Run Time (ART) as the platform default. The ART was introduced in Android 4.4 on an experimental basis. Dalvik uses just-in-time (JIT) compilation which compiles the byte code every time an application is launched. However, ART uses ahead-of-time (AOT) compilation by performing it upon the installation of an application. This greatly reduces the mobile device's processor usage, as the overall compilation during the operation of an application is reduced.
- **Application Framework**: Android applications are run and managed with the help of an Android application framework. It is responsible for performing many crucial functions such as resource management, handling calls, and so on. Key services include:
  - **Activity Manager**: This service controls all aspects of the application lifecycle and activity stack.
  - **Content Providers**: This service allows applications to publish and share data with other applications.
  - **Resource Manager**: This service provides access to non-code embedded resources such as strings, color settings, and user interface layouts.
  - **Notifications Manager**: This service allows applications to display alerts and notifications to the user.
  - **View System**: This service provides an extensible set of views used to create application user interfaces.
  - **Package Manager**: The system by which applications are able to find out information about other applications currently installed on the device.
  - **Telephony Manager**: This service provides information to the application about the telephony services available on the device such as status and subscriber information.
  - **Location Manager**: This service provides access to the location services allowing an application to receive updates about location changes.
- **Application Layer**: The topmost layer in the Android stack consists of applications which are programs that the user directly interacts with. There are two kind of applications:
  - **System Apps**: Applications that are pre-installed on the phone and are shipped along with the phone. Examples include a default browser, e-mail client, contacts, etc. These cannot be changed or uninstalled without rooting the device.
  - **User-Installed Apps**: Applications that are downloaded and installed by the user from platforms such as Google Play.

## Android Security
Android contains many security features such as:
- A user-based permissions model
- Process isolation
- Extensible mechanism for secure Inter-Process Communication (IPC)

A unique feature to Android's implementation of the user-based permissions model found in Linux is, in addition to each App running as its own user, the App has to declare which permissions it will be using in a manifest file when they are installed. This is done to require the user to approve of the permissions. The permissions file is always in the form of a `AndroidManifest.xml` file and can be found in an App after opening it in the Android SDK environment. On the subject of permissions, below is a table containing the permission categories present on all Android devices:

| Permission Type | Description |
| --------------- | ----------- |
| Normal | This is the default value. These are low risk permissions and do not pose a risk to other applications, system or user. This permission is automatically granted to the app without asking for user approval during installation. |
| Dangerous | These are the permissions that can cause harm to the system and other applications. Hence, the user's approval is needed during installation. |
| Signature | These are automatically granted to a requesting app if that app is signed by the same certificate as the one that declared / created the permission. This level is designed to allow apps that are part of a suite, or otherwise related, to share data. |
| Signature/System | A permission that the system grants only to the applications that are in the Android system image, or that are signed with the same certificate as the application that declared the permission. |

- **Application Sandboxing**: Android's method of isolating applications from each other. Using the Linux-based protection model, each Android application is assigned a User-ID (UID) and is run as a separate process. This sandboxing is done at the kernel level. By default, applications cannot read or access the data of other applications and have limited access to the operating system.

- **SELinux in Android**: Security-Enhanced Linux (SELinux) was introduced in Android 4.3. Normal Android security is based on discretionary access control, where an application can ask for permissions, and users can grant or deny them. SELinux uses mandatory access control (MAC) which ensures that applications work in isolated environments. Therefore, even if a user installs a malware app, the malware cannot access the OS and corrupt the device. This policy is applied to all processes, including those running with root privileges. SELinux operates on the principle of *default denial*. Anything that is not explicitly allowed is denied. SELinux either operates in **permissive mode** in which permission denials are logged but not enforced, and **enforcing mode** in which denials are both logged and enforced. After Android 5.0, SELinux is by default set to **enforcing mode**.

- **Application Signing**: All Android apps need to be digitally signed with a certificate before they can be installed on a device. These certificates do not need to be signed by a certificate authority and Android apps often use self-signed certificates. The app developer holds the certificate's private key and uses it to provide updates to their applications and share data between applications.

- **Secure Interprocess Communication**: Since applications are sandboxed, they need a method to communicate and share information. This is done in Android by using the **Binder** mechanism. The Binder framework provides the capabilities required to organize all types of communication between various processes. Using this framework, it is possible to perform a variety of actions of actions such as invoking methods on remote objects as if they were local, synchronous and asynchronous method invocation, sending file descriptors across processes, and so on. All communication between the processes using the Binder framework occurs through the `/dev/binder` Linux kernel driver. All communications between the client and server happen through **proxies** on the client side and **stubs** on the server side. The proxies and the stubs are responsible for sending and receiving the data, and the commands, sent over the Binder driver. Each service exposed using the Binder mechanism is assigned with a **token**. This token is a 32-bit value and is unique across all processes in the system. A client can start interacting with the service after discovering this value. This is possible with the help of the Binder's **context manager**. Basically, the context manager acts as a name service, providing the handle of a service using the name of the service. In order to get this process working, each service must be registered within the context manager. Thus, a client needs to know only the name of a service to communicate. The name is resolved by the context manager and the client receives the token that is later used for communicating with the service. The Binder driver adds the UID and the PID value of the sender process to each transaction.

## Android Hardware Components
Below are a list of core components found in almost all Android devices:

- **Central Processing Unit (CPU)**: also known as a processor, serves as the component responsible for executing all processes on the device. In regards to architecture, it can consist of: x86 (Intel), ARM, MIPS, Cortex, A5, A7, or A9.
- **Baseband Processor**: the processor on a baseband modem that handles the process of managing several radio control functions such as signal generation, modulation, encoding, as well as frequency shifting. It can also manage the transmission of signals.
- **Memory**: Consists of both RAM and ROM. For most phones, the ROM on the device contains the boot loader, OS, all downloaded applications and their data, settings, and so on. Both RAM and ROM are typically combined into a single component called a **Multichip Package (MCP)**.
- **SD Card**: A removable memory card known as **Secure Digital (SD)** which are non-volatile flash memory. Most multimedia data and large files are stored by apps on an SD Card.
- **Display**: The phone screen, either a **thin film transistor liquid crystal display (TFT LCD)** or **glass** screen.
- **Battery**: Made of various materials, such as Lithium Ion, Lithium Polymer, Nickel Cadmium, or Nickel Metal Hydrid.

## Android Boot Process
The sequence of steps involved in the Android Boot Process is as follows:

1. Boot ROM code execution
2. The boot loader
3. The Linux Kernel
4. The init process
5. Zygote and Dalvik
6. The system server

- **Boot ROM code execution**: Device hardware is initialized and boot media is detected. Once the boot sequence is established, the initial boot loader is copied to the internal RAM. After this, the execution shifts to the code loaded into the RAM.
- **The boot loader**: A piece of program that is executed before the operating system starts to function. Two stages: **initial program load (IPL)**, which deals with detecting and setting up external RAM, and **second program load (SPL)**, which is copied into external RAM and is transferred execution. The SPL is responsible for loading the Android operating system as well as providing access to other boot modes.
- **The Linux Kernel**: After the kernel is loaded, it mounts the **root file system (rootfs)** and provides access to system and user data. Once memory management units and caches have been initialized, the system can use virtual memory and launch user space processes. The kernel then looks in the rootfs for the init process and launches it as the initial user space process.
- **The init process**: looks for a script named `init.rc` that describes the system services, file system, and any other parameters that need to be setup.
- **Zygote and Dalvik**: Zygote is one of the first init processes created after the device boots. It initializes the Dalvik virtual machine and tries to create multiple instances to support each android process. After this applications can run by requesting new Dalvik virtual machines that each one runs in. Zygote registers a server socket for zygote connections, and also preloads certain classes and resources.
- **The system server**: All core features of the device such as telephony, network, and other important functions are started by the system server. The system sends a broadcast action called `ACTION_BOOT_COMPLETED` which informs all the dependent processes that the boot process is complete. After this the device displays the home screen and is ready to interact with the user.
***

## Android Data Storage and Partition Layout
Partitions are logical storage units made inside the device's persistent storage memory. Partitioning allows you to logically divide the available space into sections that can be accessed independently of each other. Below are some of the common partitions across all Android versions:

- **boot loader**: This partition stores the phone's boot loader program. This program takes care of initializing the low-level hardware when the phone boots. Thus, it is responsible for booting the Android kernel and booting into other boot modes, such as the recovery mode, download mode, and so on.
- **boot**: This partition has the information and files required for the phone to boot. It contains the kernel and RAM disk. So, without this partition, the phone cannot start its processes.
- **recovery**: Recovery partition allows the device to boot into the recovery console through which activates such as phone updates and other maintenance operations are performed. For this purpose, a minimal Android boot image is stored. This boot image serves as a failsafe.
- **userdata**: This partition is usually called the data partition and is the device's internal storage for application data. A bulk of user data is stored here, and this is where most of our forensic evidence will reside. It stores all app data and standard communications as well.
- **system**: All the major components other than kernel and RAM disk are present here. The Android system image here contains the Android framework, libraries, system binaries, and preinstalled applications. Without this partition, the device cannot boot into normal mode.
- **cache**: This partition is used to store frequently accessed data and various other files, such as recovery logs and update packages downloaded over the cellular network.
- **radio**: Devices with telephony capabilities have a baseband image stored in this partition that takes care of various telephony activities.

To identify the current partitions on a device using `adb`, run the following command from a shell:
  - `ls -l /dev/block/platform/dw_mmc/by-name`

In order to perform forensic analysis on any system it's important to understand the underlying file hierarchy. Android's file hierarchy is similar to a Unix-style system with the top of the three being denoted as `/`, or the root. Below is a list of the common directories under the standard Android root directory:

- `/acct`: This is the mount point for the acct cgroup (control group) that provides for user accounting.
- `/cache`: This is the directory where Android stores frequently accessed data and app components. Wiping the cache doesn't affect your personal data, but simply deletes the existing data there. There is also another directory in this folder called `lost+found`. This directory holds recovered files (if any) in the event of filesystem corruption, such as incorrectly removing the SD card without unmounting it and so on. The cache may contain forensically relevant artifacts, such as images, browsing history, and other app data.
- `/d`: This is a symbolic link to `/sys/kernel/debug`. This folder is used to mount the debugfs filesystem and to debug the kernel.
- `/data`: This is the partition that contains the data of each application. Most of the data belonging to a user, such as the contacts, SMS, dialed numbers, and so on, is stored in this folder. This folder has significant importance from a forensic point of view as it holds valuable data. Below are important subdirectories that can be found in the `/data` directory:
  - `/data/dalvik-cache`: Android application `.dex` files are modified when the application is installed which creates a corresponding `.odex` file (optimized `.dex` file). It is then stored in this directory so that the optimization process doesn't need to be performed each time the `application.log` file is loaded. This folder also contains several logs that might be useful during examination, depending on the underlying requirements.
  - `/data/data`: This directory contains the private data of all applications. Most of the data belonging to the user is stored in this folder.
- `/dev`: This directory contains special device files for all the devices. This is the mount point for the `tempfs` filesystem. This filesystem defines the devices available to the applications.
- `/init`: The location for the init program executed when the phone boots.
- `/mnt`: This directory serves as a mount point for all the filesystems, internal and external SD cards, and so on.
- `/proc`: This is the mount point for the `procfs` filesystem that provides access to the kernel data structures. Several programs use `/proc` as the source for their information. It contains files that have useful information about the processes.
- `/root`: This is the home directory for the root account. This folder can be accessed only if the device is rooted.
- `/sbin`: This contains binaries for several important daemons. This is not of much significance from a forensic perspective.
- `/misc`: This directory contains information about miscellaneous settings. These settings mostly define the state, that is, ON/OFF. Information about hardware settings, USB settings, and so on can be accessed from this folder.
- `/sdcard`: This is the partition that contains the data present on the SD card of the device. Any app on the phone with the `WRITE_EXTERNAL_STORAGE` permission may create files or folders in this location. There are some default folders, such as `android_secure`, `Android`, `DCIM`, `media`, and so on.
  - `DCIM`: Digital Camera Images (DCIM) is the default directory structure for digital cameras, smartphones, tablets, and related solid-state devices. Within `DCIM` you will find photos you have taken, videos, and thumbnails (cache) files. Below are some common directories found here:
    - `Camera`: Photos are stored here.
    - `Music`: Media scanner classifies all media found here as user music.
    - `Podcasts`: Media scanner classifies all media found here as a podcast.
    - `Ringtones`: Media files present here are classified as ringtones.
    - `Alarms`: Media files present here are classified as alarms.
    - `Notifications`: Media files under this location are used for notification sounds.
    - `Pictures`: All photos, except the ones taken with a camera, are stored in this folder.
    - `Movies`: All movies, except the ones taken with a camera, are stored in this folder.
    - `Download`: Miscellaneous downloads are stored in this folder.
- `/system`: This directory contains libraries, system binaries, and other system-related files. The pre-installed applications that come along with the phone are also present in this directory. Below are some interesting files and folders in the system directory:
  - `/system/build.prop`: This file contains all the build properties and settings for a given device. For a forensic analyst, this file gives an overview about the device model, manufacturer, Android version, and many other details.
  - `/system/app`: This folder contains system apps and preinstalled apps.
  - `/system/framework`: This folder contains the sources for the Android framework. In this partition, you can find the implementation of key services, such as the system server with the package and activity managers. A lot of the mapping between the Java application APIs and the native libraries is also done here.
  - `/system/ueventd.goldfish.rc` and `/system/ueventd.rc`: These files contain configuration rules for the `/dev` directory.

Application data makes up a great deal of forensic evidence extracted from a device. Therefore it is important to understand how the data is stored. For example, the default Android e-mail app stores its data under: `/data/data/com.android.email`. (This `com.android.X` naming convention is common across most default apps). Android provides developers with certain options to store data to the device. The option that can be used depends on the underlying data that is to be stored. Data that belongs to applications can be stored in one of the following locations:
  - **Shared Preferences**: This location provides a framework to store key-value pairs of primitive data types in the `.xml` format. Typically these are stored in the application's `/data/data/<app folder>/shared_prefs` directory. You can cat out these files to see the various settings saved for each preference. (Can include passwords and other sensitive data).
  - **Internal Storage**: Files that are typically stored in an application's `/data/data/<app folder>` directory. Contains directories such as: `shared_prefs`, `lib` (custom library files), `files` (developer-saved files), `cache` (files cached by the app), and `databases` (SQLite and journal files).
  - **External Storage**: Removable media such as an SD card.
  - **SQLite Database**: Android supports SQLite through dedicated APIs, and applications are used by apps to store various data. These databases are normally stored in an application's `/data/data/<app folder>/databases` directory.  
  - **Network**: Information stored using web-based services.

Android typically makes use of the standard Linux filesystem EXT4. Linux makes use of mount points, with mounting being an act of attaching an additional filesystem to the currently accessible filesystem of a computer. Everything is integrated into a single file hierarchy that begins with root (`/`). Each filesystem has a separate kernel module that registers the operations that it supports with something called **virtual file system (VFS)**. VFS allows different applications to access different filesystems in a uniform way. To see the filesystems supported by an Android device, run the following command from an `adb` shell:
  - `cat /proc/filesystems`

Android supports these three main categories of file systems: Flash memory filesystems, Media-based filesystems, and Pseudo filesystems. Important Pseudo file systems include the `/dev/tmpfs` file system. It provides temporary storage that stores files in RAM. If you are extracting from a live device you can grab a copy of this directory to see what's currently in the device's RAM.
***

## Android Debug Bridge (ADB)
In Android forensics, Android Debug Bridge (ADB) plays a very crucial role. In order to use it with a connected device, the **USB-Debugging** option needs to be enabled. This can be turned on by finding the Device's build number (normally located in the Settings, Device Info menu), and tapping it 7 times. Then, go into Developer Options and turn on USB-Debugging. Once this option has been enabled it will run the `adb daemon` which is required to interface with the device. This daemon will continuously look for a USB connection. The `adb` process communicates over the local host ports 5555 through 5585. The even port communicates with the device's console, while the odd port is for adb connections. The adb client program communicates with the local adbd over port 5037.

**Scan for Devices**
In order to scan for a list of connected devices from the forensics workstation, run the following command:
  - `adb devices`
The output generated by the command will include a line per device, with the device name on the left and either `offline` if it is not responding or `device` if it is connected to the adb server. If there are no devices then the `no device` output will be displayed.

**Direct Commands to a Specific Device**
If more than one device is connected to a system, you can use the `-s` option to issue commands to a device using the device name listed in the `adb devices` command.
  - `adb -s emulator-5554 shell`
The `-d` option is used to direct a command to the only attached USB device. Similarly, the `-e` option is used to direct an `adb` command to the only running emulator instance.

**Installing an Application**
To install an application to the device, run the following command:
  - `adb -s <device name> install /path/to/apk`

**Pulling data from the Device**
Run the following command to pull data from a device to the local machine:
  - `adb -s <device name> pull /sdcard/Pictures/MyFolder/Sample.png /home/user/Desktop/`

**Pushing data to a Device**
To put files onto a device, run the following command:
  - `adb -s <device name> push /home/user/Desktop/Sample.png /sdcard/Pictures`

**Restarting the adb server**
To restart the adb server, run the following command:
  - `kill-server`

**Viewing Log Data**
In Android, the `logcat` command provides a way to view the system debug output. Logs from various applications and portions of the system are collected in a series of circular buffers which then can be viewed and filtered by this command:
  - `adb -s <device name> logcat`
The `logcat` command also has the following message type indicators in its output:

| Message Type | Description |
| ------------ | ----------- |
| V | Verbose |
| D | Debug |
| I | Information |
| W | Warning |
| E | Error |
| F | Fatal |
| S | Silent |

***

## Rooting Android
Rooting an Android phone is all about gaining superuser access on the device to perform actions that are not normally allowed on the device. Below are some concepts to cover before going over exactly how to root the device:

**Recovery Mode**: An Android phone can be seen as having three main partitions: boot loader, Android ROM, and recovery. The recovery partition, commonly referred to as stock recovery, is the one that is used to delete all user data and files or to perform system updates. Both of these operations can be started from Android's settings menu or by manually booting into the recovery mode. The recovery partition is used when performing a factory reset where it boots up and erases all files and data. Likewise, with updates, the phone boots into the recovery mode to install the latest updates that are written directly to the Android ROM partition. The recovery mode can be accessed using one of the two below methods:
  - By pressing certain combinations of keys when booting the device (usually, by holding volume +, volume -, and power buttons during bootup).
  - By issuing the `adb reboot recovery` command to a booted Android system.
As a side note, you can also load a custom recovery partition over the stock one that provides more functionality such as running adb as root or providing a busybox binary.

**Fastboot Mode**: Fastboot is a protocol that can be used to reflash partitions on your device. It is one of the tools that comes along with Android SDK. It is an alternative to the recovery mode to do installations and updates and also to unlock the boot loader in some cases. While in fastboot, you can modify the filesystem images from a computer over a USB connection. Hence, it is one of the ways to install the recovery images and just boot in some cases. Once the phone is booted into fastboot, you can flash image files in the internal memory.

**Locked and Unlocked boot loaders**: Locked boot loaders do not allow you to perform modifications to the device's firmware by implementing restrictions at the boot loader level. This is usually done through cryptographic signature verification. In order to run any custom recovery image you need to unlock the boot loader first. When unlocking the boot loader, take note that a factory data reset is performed on the phone when unlocking a locked boot loader. Some devices have the ability to unlock them officially by putting them in fastboot mode and issuing the `fastboot oem unlock` command. This will unlock the boot loader and do a complete wipe of the Android device. HTC in particular provides a tool on their website to unlock the boot loader on their phones.

Rooting a device with an unlocked boot loader is fairly straight forward. The process of rooting mainly involves copying the **superuser (su)** binary to a location in the current process's path (`/system/xbin/su`) and granting it executable permission with the `chmod` command. Hence the first step is to unlock the boot loader. Once the boot loader is unlocked, you can make all the desired changes to the device. The most common method of copying the `su` binary is to boot a custom recovery image. This allows us to copy the `su` binary into the system partition and set the appropriate permissions through a custom update package. Below are some steps used to install the ClockWorkMod image:
  1. Download custom recovery image from http://www.clockworkmod.com/rommanager and `su` update package from http://superuserdownload.com/. The custom recovery image can be anything as long as it supports your device. Similarly, the `su` update package can be SuperSU, SuperUser, or any other package of your choice.
  2. Copy both custom recovery image and the `su` update package to the SD card of the Android device.
  3. Next, put the device into fastboot mode: `adb reboot bootloader`
  4. Open the command prompt, and enter the following command: `fastboot boot recovery.img`
  5. In the preceding command, `recovery.img` is the recovery image you downloaded.
  6. From the `recovery` menu, select the `To apply an update zip file` option and browse to the location on your device where the `su` binary update package is present.

**NOTE**: When building an Emulated phone for testing, DO NOT CHOOSE A BUILD WITH THE GOOGLE PLAY ICON!!!!
  - The `adb root` command will not work.
  - To make system partition writable, find the emulator tool, (location for my system is: `/root/Android/Sdk/emulator`) and run the command:
    - `./emulator -avd -list-avds`
    - `./emulator -avd Nexus_6_API_26 -writable-system`
    - Make sure the emulated phone is not on when running the above command.

### Rooting an Emulated Phone
**Reference Article**: https://android.stackexchange.com/questions/171442/root-android-virtual-device-with-android-7-1-1

Download the latest version of the SuperSU app:
- https://www.apkmirror.com/apk/codingcode/supersu/
Download the `su` zip files (contains `su` binaries for various architectures)
- https://mega.nz/#!7nRh2QCA!xHJydni2jzbqgCbfBUZUALLizKqMhlaBM75ifCC2hwI

1. Install the `SuperSU.apk`
  - `adb -e install supersu.apk`
  - Turn off the phone.
2. Use your system's emulator binary to make the phone's system partition writable (binary should be in the Android root folder):
  - `/root/Android/Sdk/emulator/emulator -avd <emulated phone name> -writable-system`
3. Push the `su` binary matching your phone's architecture
  - `adb root`
  - `adb remount`
  - `adb -e push /path/to/su/su.pie /system/bin/su`
4. Change the permissions of the `su` binary.
  - `adb -e shell`
  - `su root`
  - `cd /system/bin`
  - `chmod 06755 su`
5. Set the install directive on the `su` binary and set a daemon
  - `su --install`
  - `su --daemon&`
6. Set SELinux to Permissive (only for versions 4.3 and later).
  - `setenforce 0`
7. Start the SuperSU App and update, normal method is fine.
8. Reboot phone.
***

## Logical Extractions
In digital forensics, the term logical extraction is typically used to refer to extractions that do not recover deleted data, or do not include a full bit-by-bit copy of the evidence. Another definition is any method that requires communication with the base operating system. Because of this interaction with the operating system, a forensic examiner cannot be sure that they have recovered all of the data possible; the operating system is choosing which data it allows the examiner to access.

Logical extraction is analogous to copying and pasting a folder in order to extract data from a system; this process will only copy files that the user can access and see. The line between logical and physical extractions in mobile forensics is somewhat blurrier than in traditional computer forensics. For example, deleted data can routinely be recovered from logical extractions on mobile devices, due to the prevalence of SQLite databases being used to store data. Furthermore, almost every mobile extraction will require some form of interaction with the Android operating system; there is no simple equivalent to pulling a hard drive and imaging it without booting the drive.

For the most part, any and all user data may be recovered logically:
- Contacts
- Call Logs
- SMS/MMS
- Application Data
- System logs and information
The bulk of this data is stored in SQLite databases, so it is even possible to recover large amounts of deleted data through a logical extraction. Also to reiterate, root access needs to be acquired first before performing any extraction.

### Manual ADB Data Extraction
The ADB `pull` command can be used to pull single files or entire directories directly from the devices on to the forensic examiner's computer. First, you need to enable USB Debugging. This can normally be done via the following methods:
1. Go to Settings
2. Select Phone Info (or equivalent option)
3. Find the phone's Build number, tap it 7 times.
4. You will receive a message saying you are now a developer. Back out of the info menu and select Developer Options.
5. Scroll down until you see an option for USB Debugging. Turn it on.
You might also need to install the correct drivers on your examination computer. These can generally be found online, either from the manufacturer's website or at http://www.xda-developers.com. These are also often included with commercial forensics tools.

Prior to Android 4.2.2, enabling USB Debugging was the only requirement to communicate with the device over ADB. In Android 4.2.2, Google added Secure USB Debugging option. The Secure USB Debugging option adds an additional requirement of selecting to connect to a computer on the device's screen. You must select OK on the phone's screen to continue. If Always allow from this computer is selected, the device will store the computer's RSA key and the prompt will not appear on future connections to that computer even if the device is locked.

To verify that a connected device is ready to extract, run the `adb devices` command. The device should appear in the list of connected devices as a **device**. (You might have to run the command a couple of times to get the adb daemon to start correctly). The next thing to check is that the phone has been rooted. Initiate a shell with the phone with the `adb shell` command, (assuming it's the only phone connected). You should be presented with a shell with the `$` symbol. This indicates a normal user shell. Now, enter the `su` command. The symbol should change to a `#`, indicating a root shell.

For the sake of grabbing data from the phone, we will be using the `adb pull` command which has the following syntax:
- `adb pull [-p] [-a] <remote> [<local>]`
The `-p` flag shows the transfer's progress, while the `-a` flag will copy the file's timestamp and mode. Below is an example of using the command to grab the SMS database file from the device and write it to a local directory.
- `adb pull -p /data/data/com.android.providers.telephony/databases/mmssms.db /path/to/case/files/`
  - **NOTE**: Make sure you run `adb root` first.
This method is very useful for scripting purposes if you already know what files you want to download ahead of time.

### Recovery Mode
In order to truly be forensically sound, ADB data extractions should not be used against a phone while it is turned on. To avoid this, the examiner should place the device into a custom recovery mode, due to how adb access is not available through stock recovery mode. Typically this process involves some combination of powering the device off and holding the volume and power keys. Refer to the device manufacturer's website for more detailed information on each device.

Once a device has been booted into a custom recovery mode, you must then mount the data partition. Some custom recoveries may mount this automatically, and others might not. If using either the Clockwork Mod Recovery or Team Win Recovery Project (TWRP) images from the below URLs, the data partition can be mounted by selecting Mounts and then selecting the data partition. The recovery menu is generally navigated by using the volume keys to move up and down and the power button to select, or may be touch based.
- https://www.clockworkmod.com/reommanager
- http://teamw.in/project/twrp2

### Fastboot Mode
Fastboot is another protocol utility built into the Android SDK, and is used for interacting directly with a device's bootloader. Essentially, it is a much lower-level version of ADB, and is frequently used to flash new images to a device. Fastboot can be used to boot from a custom recovery image, and temporarily gain root access on a device. Fastboot does not require USB debugging to be enabled or root access. The most important requirement for using fastboot is an unlocked bootloader; locked bootloaders will not allow a device to boot from code that isn't specifically signed by the manufacturer.

To determine the bootloader status, first boot into the bootloader using the following command:
- `adb reboot bootloader`
Once loaded, the screen will show the bootloader status which will state either locked or unlocked next to the **LOCK STATE** section. If the bootloader is unlocked, then you can load a custom recovery image. Next, the device will need to be placed into fastboot mode, which can be accomplished in one of two ways:
- ADB: `adb reboot bootloader`
- Physical device buttons: Some combination of volume and power buttons.
To determine if a device is ready to communicate, run the `fastboot devices` command, which will display a device if it is in fastboot mode. Finally, to load the custom recovery image, run the `fastboot boot /path/to/custom/image` command.

### ADB Backup Extractions
Google implemented ADB backup functionality, beginning in Android 4.0 Ice Cream Sandwich. This allows users to backup application data to a local computer over ADB. This process does not require root, and is therefore highly useful for forensics purposes. An app may more may not have the option to allow backups enabled, so expect varying results. The command format is as follows:
- `adb backup [-f <file>] [-apk|-noapk] [-obb|-noobb] [-shared|-noshared] [-all] [-system|-nosystem] [<packages...>]`
- The flags are as follows:
  - `-f`: Names the path for the output file. If not specified, defaults to `backup.ab` in present working directory.
  - `[-apk|-noapk]`: Choose whether or not to back up the `.apk` file. Defaults to `-noapk`.
  - `[-obb|-noobb]`: Choose whether or not to back up `.obb` (APK expansion) files. Defaults to `-noobb`.
  - `[-shared|-noshared]`: Choose whether or not to back up data from shared storage and the SD card. Defaults to `-noshared`.
  - `[-all]`: Include all applications for which backups are enabled.
  - `[-system|-nosystem]`: Choose whether or not to include system applications. Defaults to `-system`.
  - `[<packages>]`: Explicitly name application packages to be backed up. Not needed if using `-all` or `-shared`.
For example, the following command will capture all possible application data to the local directory:
- `adb backup -shared -all`
When performing a backup the user must approve the backup on the device. This means that backups cannot be performed without bypassing screen locks. The resulting backup file is stored as a `.ab` file, but it is actually a `.tar` file that has been compressed with the Deflate algorithm. If a password was entered on the device when the backup was created, the file would also be AES encrypted. There are free utilities that can be used to convert `.ab` files to `.tar` files located at: http://sourceforge.net/projects/adbextractor/. For example, below is an example of using the Android Backup Extractor to convert `.ab` files:
- `java -jar abe.jar unpack backup.ab backup.tar`.

### ADB Dumpsys
**Dumpsys** is a tool built into the Android OS, generally used for development purposes to show the status of services running on the device. However, it can also contain forensically interesting information. Dumpsys does not require root access, but like all ADB commands, it does require USB Debugging to be enabled on the device and Secure USB Debugging to be bypassed. To view a list of all possible services that can be dumped, run the following command:
- `adb shell services list`
To grab a specific service, in this case (iphonesubinfo), run the following command:
- `adb shell dumpsys iphonesubinfo`

### Bypassing Android Lock Screens
Locks screens are the most challenging aspect of Android forensic examinations. Frequently, the entire investigation depends on the examiner's ability to gain access to a locked device. While there are methods to bypass them, this can be highly dependent on the OS version, device settings, and technical capabilities of the examiner. There is no magical solution that will work every time on every device. The following are the common types of Lock Screens:
- None/Slide
- Pattern
- PIN
- Password
- Smart Lock
  - Trusted Face
  - Trusted Location
  - Trusted Device
In all cases, bypassing the lock screen will require retrieving a file from the device. Pattern locks are stored as hash values at `/data/system/gesture.key` and PIN/Password locks are stored as hash values at `/data/system/password.key`. Additionally, the password.key hash is salted; the salt value is stored at `/data/data/com.android.providers.settings/databases/settings.db` prior to Android 4.4, and `/data/system/locksettings.db` on devices running Android 4.4 and higher. Only one file needs to be pulled to crack a Pattern lock on all versions of Android: `/data/system/gesture.key`. If you are in a situation where you don't care about changing data, you can overwrite these files to bypass the security completely.
***

## Physical Extractions
In digital forensics, a physical extraction is an exact bit-for-bit image of the electronic media, and this definition remains true for mobile devices too. In traditional computer forensics, this typically involves removing the evidence drive from the suspect's computer and imaging it via a write blocker without ever booting the drive, resulting in an image file containing an exact copy of the suspect's drive. The output is frequently referred to as a **raw image**, or simply a **bin** (binary) file. These images will contain all unallocated space, file slack, volume slack, and so on.

In mobile forensics, the result is the same; an exact bit-for-bit image of the device, but the methods are somewhat different. For example, removing the flash memory from the device to image can be both time-consuming and expensive, and requires a lot of specialized knowledge. Furthermore, in most cases the device must be booted to some degree, and even written to in many cases. Just like logical extractions, this will require root access.

### Extracting data physically with dd
The `dd` command is a Linux command-line utility used by definition to convert and copy files, but is frequently used in forensics to create bit-by-bit images of entire drives. As the `dd` command is built for Linux-based systems, it is frequently included on Android platforms. Below is an example of using the command to copy the primary partition of an Android device to the SD Card:
- `dd if=/dev/block/mmcblk0 of=/sdcard/blk0.img bs=4096 conv=notrunc,noerror,sync`
  - `if`: This option specifies the path of the input file to read from.
  - `of`: This option specifies the path of the output file to write to.
  - `bs`: This option specifies the block size. Data is read and written in the size of the block specified, defaults to 512 bytes if not specified.
  - `conv`: This option specifies the conversion options as its attributes:
    - `notrunc`: This option does not truncate the output file.
    - `noerror`: This option continues imaging if an error is encountered.
    - `sync`: In conjunction with the `noerror` option, this option writes `\x00` for blocks with an error. This is important for maintaining file offsets within the image.
  - **NOTE**: there is an important correlation between the block size and the `noerror` and `sync` flags: if an error is encountered, `\x00` will be written for the entire block that was read (as determined by the block size). Thus, smaller block sizes result in less data being missed in the event of an error.

When imaging a computer, an examiner must first find what the drive is mounted as; `/dev/sda` for example. The same is true when imaging an Android device. The first step is to launch the ADB shell and view the `/proc/partitions` file using the following command:
- `cat /proc/partitions`
The first entry in the list, (usually something like `mmcblk0`), is the entirety of the flash memory on the device. This can be fed into the `if` flag as `/dev/blk/mmcblk0` to copy the entire drive. If you are looking for a particular drive, the `mount` command in the adb shell can be used to see which ones are actually being used by the device.

As far as getting the extraction to your examiner machine, you can use netcat. It is likely that netcat does not exist on the device normally, (typing `nc` in an adb shell should return nothing or an error). Netcat compiled for Android can be found at many places online, for example:
- https://github.com/MobileForensicsResearch

To push netcat to the device, use the following command:
- `adb push nc /dev/nc`
Give the binary executable permissions in an adb shell:
- `chmod +x /dev/nc`
Enable port forwarding on the host machine:
- `adb forward tcp:9999 tcp:9999`
- The port number can by anything, 9999 is just used for example.
Now, run the following command in the adb shell terminal to begin copying the drive and linking it to a netcat listener on the device:
- `dd if=/dev/block/mmcblk0 bs=512 conv=notrunc,noerror,sync | /dev/nc -l -p 9999`
In another terminal, run the command:
- `nc 127.0.0.1 9999 > physical_extraction.img`
Once the command finishes, (you might need to check the file size until it matches the size of the partition you are copying, and then cancel the command via CTRL+C), you now have a physical extraction of the device that can be examined further.

### Imaging and analyzing Android RAM
Typically this isn't done with mobile devices as most rooting processes involve rebooting the phone, which will erase the current contents of RAM, defeating the purpose of acquiring it in the first place. For this reason, there is little support and development for creating Android RAM capture tools. The main challenge when it comes to RAM is the analysis. RAM is completely raw, unstructured data; there is no file system. When viewed in a hex editor, RAM appears to just be a giant blob of data. RAM can, however, be easily searched for keywords using traditional forensic tools and methods, but that presumes an examiner knows exactly what they are looking for.

Any data that is written to the flash memory *must* pass through RAM, there is no other way for the processor to communicate with the flash memory. This means that almost anything done on the device may be found in the contents of a RAM dump. Depending on the amount of device usage, data may remain in RAM indefinitely, until it needs to be overwritten. RAM dumps frequently contain text typed on the device, including usernames and passwords, and application data that is not stored permanently on the device.

There is a public tool for extracting RAM images called `mem` and is available at the same link as netcat for Android, (see above). Mem is an executable binary that needs to be pushed to the device and executed using the same procedures used for getting a physical device image using `dd` and `nc`. Mem has the capability to read the entire RAM, or to target specific, forensically-interesting processes (applications). The typical format for running the file is:
- `./mem <PID>`
If the PID is set to 0, all of RAM will be imaged.
**NOTE**: Come back after exploring LiME with Linux VMs.
***

## Recovering Deleted Data from an Android Device
With respect to Android, it is possible to recover most of the deleted data, including SMS, pictures, application data, and so on. When a user deletes an data from the device, the data is not actually erased and continues to exist on the device. What gets deleted is the pointer to this data. All file systems contain metadata that maintains information about the hierarchy of files, file names, and so on. Deletion does not actually erase the data, but instead, it removes the file system metadata. The files are still present on the device as long as they are not overwritten by some other data.  
- **NOTE**: Oxygen Forensics SQLite Viewer
- File carving via `scalpel`
  - Sample config file: https://asecuritysite.com/scalpel.conf.txt
    - Contains file header information to look for while carving files from binary data dumps.
***

## Forensic Analysis of Android Applications
Forensically analyzing an application is as much of an art as it is a science. There are myriad ways an application can store or obfuscate its data. Different versions of the same application may even store the same data differently. A developer is really only limited by their imagination (and Android platform restrictions) when it comes to choosing how to store their data. The end goal of forensically analyzing an application is consistently the same, to understand what the app was used for and find user data.

All apps, (that aren't system apps), store their data in the `/data/data` directory by default. Apps could also use the SD card if they ask for this permission when the app is installed. The package name is the name of the directory for the application in the `/data/data` directory. Paths to data on the SD card follow the format of: `/sdcard/<appname>`.

To get a list of applications installed on the device, at least in a format that will look good on a forensics report, get the file `/data/system/packages.list`. This file lists the package name for every app on the device and the path to its data. If for some reason this file does not exist on the device you can run the following command to achieve a similar output:
- `adb shell pm list packages -f`.

Another file of interest is the `/data/system/package-usage.list` file that shows the last time a package (or application) was used. The time listed will be in Linux epoch time, which is the number of seconds (or milliseconds) since midnight on 1 January, 1970, UTC. A 10-digit value indicates it is in seconds, while a 13-digit indicates it is in milliseconds. There are various applications to translate this into something more legible, such as the DCode tool found here:
- http://www.digital-detective.net/digital-forensic-software/free-tools/
Or an online converter found here:
- http://www.epochconverter.com/

### Wi-Fi analysis
Wi-Fi is not technically an application, but it is an invaluable source of data that should be examined. Wi-Fi connection data is found in `/data/misc/wifi/wpa_supplicant.conf`. This file contains a list of access points that the phone has connected to automatically. If the access point requires a password, that would also be stored in the file in plain text.

### Google Chrome Analysis
Google Chrome is typically the default web browser on most android devices. Chrome data on the device is somewhat unique, in that, it contains data not just from the device, but from all devices on which the user has logged in to Chrome. Below is some basic information concerning the Chrome application:
- **Package Name**: `com.android.chrome`
- **Files of Interest**:
  - `/app_chrome/Default/`
    - `Sync Data/SyncData.sqlite3`
    - `Bookmarks`
    - `Cookies`
    - `Google Profile Picture.png`
    - `History`
    - `Login Data`
    - `Preferences`
    - `Top Sites`
    - `Web Data`
  - `/app_ChromeDocumentActivity/`
All the files listed here in the `/app_chrome/Default/` folder, except for the one `.png` file, Bookmarks, and Preferences, are SQLite databases despite the lack of a file extension. If you come across a WebKit time stamp, the stamp can be decoded using the DCode application discussed previously by using the Google Chrome Value decode format.

### Application Reverse Engineering
The vast majority of Android applications are written in Java. In order to truly reverse engineer Java code, one should generally be able to engineer Java code first. Below is a link to a list of Android reverse engineering tools:
- https://github.com/ashishb/android-security-awesome

Applications are installed via `.apk` files. The APK file for an app is stored on the device, even after the application is installed (and is removed when an app is deleted). This APK contains the compiled Java code for the app, the icons and fonts used in the app, and an `AndroidManifest.xml` file that declares the permissions the application needs. The APK files are located in each application's `/data/data` directory. The APK file for preinstalled system applications can be found in the `/system/app/` directory. The APK file itself is stored in a directory named after its package name, followed by a dash and a number.

The APK file is actually just a ZIP compressed file. Renaming the extension to `.zip` will allow an examiner to open the container and browse the files contained in it. This might not always allow you to view everything, such as the manifest file. Therefore it is best to open in a tool such as Android SDK. View the Manifest file to see the app's permissions.
- **NOTE**: Come back and put in a section with tools used at work.
