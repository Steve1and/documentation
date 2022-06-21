# Windows Fundamentals
***

Mount a windows share on a Linux host:
- `mkdir /mnt/windows_share`
- `sudo apt-get install cifs-utils`
- `sudo mount -t cifs -o username=<username>,password=<password> //<ipoftarget>/"share_name" /mnt/windows_share`

## Windows Versions

| Operating System Names | Version Number (kernel) |
| ---------------------- | -------------- |
| Windows NT 4 | 4.0 |
| Windows 2000 | 5.0 |
| Windows XP | 5.1 |
| Windows Server 2003, 2003 R2 | 5.2 |
| Windows Vista, Server 2008 | 6.0 |
| Windows 7, Server 2008 R2 | 6.1 |
| Windows 8, Server 2012 | 6.2 |
| Windows 8.1, Server 2012 R2 | 6.3 |
| Windows 10, Server 2016, Server 2019 | 10.0 |

You can use the PowerShell cmdlet `Get-WmiObject` to obtain information about the operating system.
- `Get-WmiObject -Class Win32_OperatingSytem | select Version,BuildNumber`
- Other useful classes that can be used with `Get-WmiObject`:
  - `Win32_Process`: Get a process listing
  - `Win32_Service`: Get a listing of services
  - `Win32_Bios`: Get BIOS information
  - `ComputerName`: Get information about remote computers.
- `Get-WmiObject` can be used to start and stop services on local and remote computers, and more.
***

## Operating System Structure

In Windows operating systems, the root directory is `<drive_letter>:\` (commonly the C drive). The root directory, (also known as the boot partition), is where the operating system is installed. Other physical and virtual drives are assigned other letters, for example, Data (E:). The directory structure of the boot partition is as follows:

| Directory | Function |
| --------- | -------- |
| Perflogs | Can hold Windows performance logs but is empty by default. |
| Program Files | On 32-bit systems, all 16-bit and 32-bit programs are installed here. On 64-bit systems, only 64-bit programs are installed here. |
| Program Files (x86) | 32-bit and 16-bit programs are installed here on 64-bit editions of Windows. |
| ProgramData | This is a hidden folder that contains data that is essential for certain installed programs to run. This data is accessible by the program no matter what user is running it. |
| Users | This folder contains user profiles for each user that logs onto the system and contains the two folders Public and Default. |
| Default | This is the default user profile template for all created users. Whenever a new user is added to the system, their profile is based on the Default profile. |
| Public | This folder is intended for computer users to share files and is accessible to all users by default. This folder is shared over the network by default but requires a valid network account to access. |
| AppData | Per user application data and settings are stored in a hidden user subfolder (i.e., `user1\AppData`). Each of these folders contains three subfolders. The Roaming folder contains machine-independent data taht should follow the user's profile, such as custom dictionairies. The Local folder is specific to the computer itself and is never synchronized across the network. LocalLow is similar to the Local folder, but it has a lower data integrity level. Therefore it can be used, for example, by a web browser set to protected or safe mode. |
| Windows | The majority of the files required for the Windows operating system are contained here. |
| System, System32, SysWOW64 | Contains all DLLs required for the core features of Windows and the Windows API. The operating system searches these folders any time a program asks to load a DLL without specifying an absolute path. |
| WinSxS | The Windows Component Store contains a copy of all Windows components, updates, and service packs. |

***

## Exploring Directories Using Command Line

- `dir`: Display directory contents:
  - `dir c:\ /a`
- `tree`: Graphical display of a directory structure of a path or disk.
  - `tree "c:\Program Files (x86)\VMware"`
  - `tree c:\ /f | more`
***

## File System

There are 5 types of Windows file systems: FAT12, FAT16, FAT32, NTFS, and exFAT. FAT12 and FAT16 are no longer used on modern Windows operating systems.

FAT32 (File Allocation Table) is widely used across many types of storage devices such as USB memory sticks and SD cards but can also be used to format hard drives. The "32" in the name refers to the fact that FAT32 uses 32 bits of data for identifying data clusters on a storage device.
- **Pros of FAT32**:
  - Device compatibility: it can be used on computers, digital cameras, gaming consoles, smartphones, tablets, and more.
  - Operating system cross-compatibility: It works on all Windows operating systems starting from Windows 95 and is also supported by MacOS and Linux.
- **Cons of FAT32**:
  - Can only be used with files that are less than 4GB.
  - No built-in data protection or file compression features.
  - Must use third-party tools for file encryption.

NTFS (New Technology File System) is the default Windows file system since Windows NT 3.1. In additon to making up for the shortcomings of FAT32, NTFS also has better support for metadata and better performance due to improved data structuring.
- **Pros of NTFS**:
  - NTFS is reliable and can restore the consistency of the file system in the event of a system failure or power loss.
  - Provides security by allowing us to set granular permissions on both files and folders.
  - Supports very large-sized partitions.
  - Has journaling built-in, meaning that file modifications (addition, modification, deletion) are logged.
- **Cons of NTFS**:
  - Most mobile devices do not support NTFS natively.
  - Older media devices such as TVs and digital cameras do not offer support for NTFS storage devices.

### Permissions

The NTFS file system has many basic and advanced permissions. Some of the key permission types are:

| Permission Type | Description |
| --------------- | ----------- |
| Full Control | Allows reading, writing, changing, deleting of files/folders. |
| Modify | Allows reading, writing, and deleting of files/folders. |
| List Folder Contents | Allows for viewing and listing folders and subfolders as well as executing files. Folders only inherit this permission. |
| Read and Execute | Allows for viewing and listing files and subfolders as well as executing files. Files and folders inherit this permission. |
| Write | Allows for adding files to folders and subfolders and writing to a file. |
| Read | Allows for viewing and listing of folders and subfolders and viewing a file's contents. |
| Traverse Folder | This allows or denies the ability to move through folders to reach other files or folders. For example, a user may not have permission to list the directory contents or view files in the documents or web apps directory in this example `c:\users\user1\documents\webapps\backups\backup_01042022.zip` but with Traverse Folder permissions applied, they can access teh backup archive. |

Files and folders inherit the NTFS permissions of their parent folder for ease of administration, so administrators do not need to explicitly set permissions for each file and folder, as this would be extremely time-consuming. If permissions do need to be set explicitly, an administrator can disable permissions inheritance for the necessary files and folders and then set the permissions directly on each.

### Integrity Control Access Control List (icacls)

NTFS permissions on files and folders in Windows can be managed using the File Explorer GUI under the security tab. Apart from the GUI, we can also achieve a fine level of granularity over NTFS file permissions in Windows from the command line using the `icacls` utility.

We can list out the NTFS permissions on a specific directory by running either `icacls` from withing the working directory or `icacls C:\Windows` against a directory you are not currently in.

The resource access level is listed after each user in the output. The possible inheritance settings are:
- `(CI)`: container inherit
- `(OI)`: object inherit
- `(IO)`: inherit only
- `(NP)`: do not propagate inherit
- `(I)`: permission inherited from parent container

Basic access permissions for User accounts are:
- `F`: full access
- `D`: delete access
- `N`: no access
- `M`: modify access
- `RX`: read and execute access
- `R`: read-only access
- `W`: write-only access

You can add and remove permissions using `icacls`.
- `icacls c:\users /grant user1:f`: grant full access to `user1`

These permissions can be revoked using the command `icacls c:\users /remove user1`
***

## NTFS vs. Share Permissions

The **Server Message Block protocol (SMB)** is used in Windows to connect shared resources like files and printers.

**Share Permissions**

| Permission | Description |
| ---------- | ----------- |
| Full Control | Users are permitted to perform all actions given by Change and Read permissions as well as change permissions for NTFS files and subfolders. |
| Change | Users are permitted to read, edit, delete, and add files and subfolders. |
| Read | Users are allowed to view file & subfolder contents. |

**NTFS Basic Permissions**

| Permission | Description |
| ---------- | ----------- |
| Full Control | Users are permitted to add, edit, move, delete files & folders as well as change NTFS permissions that apply to all allowed folders. |
| Modify | Users are permitted or denied permissions to view and modify files and folders. This includes adding or deleting files. |
| Read & Execute | Users are permitted or denied permissions to read the contents of files and execute programs. |
| List Folder Contents | Users are permitted or denied permissions to view a listing of files and subfolders. |
| Read | Users are permitted or denied permissions to read the contents of files. |
| Write | Users are permitted or denied permissions to write changes to a file and add new files to a folder. |
| Special Permissions | A variety of advanced permissions options. |

**NTFS Special Permissions**

| Permission | Description |
| ---------- | ----------- |
| Full Control | Users are permitted or denied permissions to add, edit, move, delete files & folders as well as change NTFS permissions that apply to all permited folders. |
| Traverse Folder / Execute File | Users are permitted or denied permissions to access a subfolder within a directory structure even if the user is denied access to contents at the parent folder level. Users may also permitted or denied permissions to execute programs. |
| List Folder/Read Data | Users are permitted or denied permissions to view files and folders contained in the parent folder. Users can also be permitted to open and view files. |
| Read Attributes | Users are permitted or denied permissions to view basic attributes of a file or folder. Examples of basic attributes: system, archive, read-only, and hidden. |
| Read Extended Attributes | Users are permitted or denied permissions to view extended attributes of a file or folder. Attributes differ depending on the program. |
| Create Files/Write Data | Users are permitted or denied permissions to create files within a folder and make changes to a file. |
| Create Folders/Append Data | Users are permitted or denied permissions to create subfolders within a folder. Data can be added to files but pre-existing content cannot be overwritten. |
| Write Attributes | Users are permitted or denied to change file attributes. This permission does not grant access to creating files or folders. |
| Write Extended Attributes | Users are permitted or denied permissions to change extended attributes on a file or folder. Attributes differ depending on the program. |
| Delete Subfolders and Files | Users are permitted or denied permissions to delete subfolders and files. Parent folders will not be deleted. |
| Delete | Users are permitted or denied permissions to delete parent folders, subfolders and files. |
| Read Permissions | Users are permitted or denied permissions to read permissions of a folder. |
| Change Permissions | Users are permitted or denied permissions to change permissions of a file or folder. |
| Take Ownership | Users are permitted or denied permission to take ownership of a file or folder. The owner of a file has full permissions to change any permissions. |

Folders created in NTFS inherit permissions from parent folders by default. The share permissions apply when the folder is being accessed through SMB.

### Creating a Network Share

1. Create a New Folder, or select an existing folder.
2. Right click, select Properties.
3. Select Advanced Sharing.
4. Select the Checkbox for Share this folder.
5. Select Permissions.
6. Select Add, choose users to access the share, click OK.
7. Select the Permissions desired via check-boxes.
8. Click OK.
***

## Windows Defender Firewall Considerations

When a Windows system is part of a workgroup, all netlogon requests are authenticated against that particular Windows system's SAM database. When a Windows system is joined to a Windows Domain environment, all netlogon requests are authenticated against Active Directory. The primary difference between a workgroup and a Windows Domain in terms of authentication, is with a workgroup the local SAM database is used and in a Windows Domain a centralized network-based databased (Active Directory) is used.

Windows Defender Firewall Profiles:
- Public
- Private
- Domain

Use `net share` to display all the shared folders on a windows system.
***

## Windows Services

Services allow for the creation and management of long-running processes. They can be started automatically at system boot and continue to run in the background. Applications can also be created to install as a service.

Windows services are managed via the Service Control Manager (SCM) system, accessible via the `services.msc` MMC add-in. This add-in provides a GUI interface for interacting with and managing services and displays information about each installed service.
- You can also query and manage services via PowerShell:
  - `Get-Service | ? {$_.Status -eq "Running"} | select -First 2 | fl`
- Service statuses can appear as Running, Stopped, or Paused, and they can be set to start manually, automatically, or on a delay at system boot. Services can also be shown in the state of Starting or Stopping if some action has triggered them to either start or stop.
- Windows has three categories of services: Local Services, Network Services, and System Services.
- Services can usually only be created, modified, and deleted by users with administrative privileges.

**Critical Windows System Services** (Cannot be stopped and restared without a system restart)

| Service | Description |
| ------- | ----------- |
| smss.exe | Session Manager SubSystem. Responsible for handling sessions on the system. |
| csrss.exe | Client Server Runtime Process. The user-mode portion of the Windows subsystem. |
| wininit.exe | Starts the Wininit file .ini file that lists all of the changes to be made to Windows when the computer is restarted after installing a program. |
| logonui.exe | Used for facilitating user login into a PC |
| lsass.exe | The Local Security Authentication Server verifies the validity of user logons to a PC or server. It generates the process responsible for authenticating users for the Winlogon service. |
| services.exe | Manages the operation of starting and stopping services. |
| winlogon.exe | Responsible for handling the secure attention sequence, loading a user profile on logon, and locking the computer when a screensaver is running. |
| System | A background system process that runs the Windows kernel. |
| svchost.exe with RPCSS | Manages system services that run from dynamic-link libraries (files with the extension .dll) such as "Automatic Updates," "Windows Firewall," and "Plug and Play." Uses the Remote Procedure Call (RPC) Service (RPCSS). |
| svchost.exe with Dcom/PnP | Manages system services that run from dynamic-link libraries (files with the extension .dll) such as "Automatic Updates," "Windows Firewall," and "Plug and Play." Uses the Distributed Component Object Model (DCOM) and Plug and Play (PnP) services. |

### Processes

Processes run in the background on Windows systems. They either run automatically as part of the Windows operating system or are started by other installed applications.

### Local Security Authority Subsystem Service (LSASS)

`lsass.exe` is the process that is responsible for enforcing the security policy on Windows systems. When a user attempts to log on to the system, this process verifies their log on attempt and creates access tokens based on the user's permission levels. LSASS is also responsible for user account password changes. All events associated with this process (logon/logoff attempts, etc.) are logged within the Windows Security Log.

### Task Manager

Windows Task Manager provides information about running processes, system performance, running services, startup programs, logged-in users/logged in user processes, and services. Task Manager can be opened by right-clicking on the taskbar and selecting `Task Manager`, pressing ctrl + shift + Esc, pressing ctrl + alt + del and selecting `Task Manager`, opening the start menu and typing `Task Manager`, or typing `taskmgr` from a CMD or PowerShell console.

| Tab | Description |
| --- | ----------- |
| Processes tab | Shows a list of running applications and background processes along with the CPU, memory, disk, network, and power usage for each. |
| Performance tab | Shows graphs and data such as CPU utilization, system uptime, memory usage, disk and, networking, and GPU usage. We can also open the Resource Monitor, which gives us a much more in-depth view of the current CPU, Memory, Disk, and Network resource usage. |

**Resource Monitor**

| Tab | Description |
| --- | ----------- |
| App history tab | Shows resource usage for the current user account for each application for a period of time. |
| Startup tab | Shows which applications are configured to start at boot as well as the impact on the startup process. |
| Users tab | Shows logged in users and the processes/resource usage associated with their session. |
| Details tab | Shows the name, process ID (PID), status, associated username, CPU, and memory usage for each running application. |
| Services tab | Shows the name, PID, description, and status of each installed service. The Services add-in can be accessed from this tab as well. |

### Service Permissions

Part of the install process for Windows programs includes assigning a specific service to run using the credentials and privileges of a designated user, which by default is set within the currently logged-on user context.
- It is highly recommended to create an individual user account to run critical network services. These are referred to as service accounts.

Use `services.msc` to view and manage just about every detail regarding all services.

Most services run with LocalSystem privileges by default which is the highest level of access allowed on an individual Windows OS.

**Notable built-in service accounts in Windows**
- LocalService
- NetworkService
- LocalSystem

On the command line, the `sc` command can be used to configure and manage services.
- `sc qc wuauserv`
- The `sc qc` command is used to query a service.
- `sc //hostname query ServiceName` (or IP)
- `sc stop wuauserv`
- `sc sdshow wuauserv`: shows a service's security descriptor

Every named object in Windows is a securable object, and even some unamed objects are securable. If it's securable in a Windows OS, it will have a security descriptor. Security descriptors identify the object's owner and a primary group containing a Discretionary Access Control List (DACL) and a System Access Control List (SACL).
- Generally, a DACL is used for controlling access to an object, and a SACL is used to account for a log access attempts.

**Security Descriptor Definition Language (SDDL)**:

 `D:(A;;CCLCSWRPLORC;;;AU)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;SY)`

The above SDDL can be read as:
- `D: (A;;CCLCSWRPLORC;;;AU)`
1. D: - the proceeding characters are DACL permissions
2. AU: - defines the security principal Authenticated Users
3. A;; - access is allowed
4. CC - SERVICE_QUERY_CONFIG is the full name, and it is a query to the service control manager (SCM) for the service configuration.
5. LC - SERVICE_QUERY_STATUS is the full name, and it is a query to the service control manager (SCM) for the current status of the service.
6. SW - SERVICE_ENUMERATE_DEPENDENTS is the full name, and it will enumerate a list of dependent services.
7. RP - SERVICE_START is the full name, and it will start the service.
8. LO - SERVICE_INTERROGATE is the full name, and it will query the service for its current status.
9. RC - READ_CONTROL is the full name, and it will query the security descriptor of the service.
- Each set of 2 characters in between the semi-colons represents actions allowed to be performed by a specific user or group.

`;;CCLCSWRPLORC;;;`

After the last set of semi-colons, the characters specify the security principal (User and/or Group) that is permitted to perform those actions.

`;;;AU`

The character immediately after the opening parentheses and before the first set of semi-colons defines whether the actions are Allowed or Denied.

`A;;`

This entire security descriptor associated with a service has three sets of access control entries because there are three different security principals. Each security principal has specific permissions applied.

To view service permissions with PowerShell, use the following command:
- `Get-ACL -Path HKLM:\System\CurrentControlSet\Services\wuauserv | Format-List`
***

## Windows Sessions

**Interactive**: An interactive, or logon session, is initiated by a user authenticating to a local or domain system by entering their credentials. An interactive logon can be initiated by logging directly into the system, by requesting a secondary logon session using the `runas` command via the command line, or through a Remote Desktop connection.

**Non-Interactive**: Non-interactive accounts in Windows differ from standard user accounts as they do not require login credentials. There are 3 types of non-interactive accounts: the Local System Account, Local Service Account, and the Network Service Account. Non-interactive accounts are generally used by the Windows operating system to automatically start services and applications without requiring user interaction. These accounts have no password associated with them and are usually used to start services when the system boots or to run scheduled tasks.

There are differences between the three types of accounts:

| Account | Description |
| ------- | ----------- |
| Local System Account | Also known as the NT AUTHORITY\SYSTEM account, this is the most powerful account in Windows systems. It is used for a variety of OS-related tasks, such as starting Windows services. This account is more powerful than accounts in the local administrators group. |
| Local Service Account | Known as the NT AUTHORITY\LocalService account, this is a less privileged version of the SYSTEM account and has similar privileges to a local user account. It is granted limited functionality and can start some services. |
| Network Service Account | This is known as the NT AUTHORITY\NetworkService account and is similar to a standard domain user account. It has similar privileges to the Local Service Account on the local machine. It can establish authenticated sessions for certain network services. |

***
## Interacting with the Windows Operating System

Remote Desktop Protocol (RDP) is a proprietary Microsoft protocol which allows a user to connect to a remote system over a network connection and obtain a graphical user interface. The user connects using RDP client software to a target system running RDP server software. RDP uses port 3389 to open a dedicated network channel for sending data back and forth.

For Windows Command Prompt (`cmd.exe`) typing `help` and pressing enter will display built-in commands.
- Command help menus can be displayed by typing `help` followed by the command and pressing enter.
- Some commands have their own help menus, displayed by typing the command, followed by `/?` and pressing enter.

PowerShell cmdlets are in the form of Verb-Noun. For example, `Get-ChildItem` can be used to list the current directory. Cmdlets also take arguments or flags, such as `Get-ChildItem -Recurse` which will show the current directory contents and all those of all subdirectories. To display the contents of a specific directory, use: `Get-ChildItem -Path C:\path\to\directory`.
- Some cmdlets have aliases, such as `Set-Location` which can be referenced as either `cd` or `sl`. `Get-ChildItem` can be called with `ls` or `gci`.
  - `Get-Alias` will display the currently set aliases.
  - `New-Alias` can set new aliases, for example: `New-Alias -Name "Show-Files" Get-ChildItem`
- To display help for PowerShell, type `help` and press enter. To get help for specific cmdlet, use `Get-Help` followed by the cmdlet.
  - The `-Online` flag for `Get-Help` will display the online help for the cmdlet.
  - `Update-Help` will download and install help files locally.

### Execution Policy

| Policy | Description |
| ------ | ----------- |
| AllSigned | All scripts can run, but a trusted publisher must sign scripts and configuration files. This includes both remote and local scripts. We receive a prompt before running scripts signed by publishers that we have not yet listed as either trusted or untrusted. |
| Bypass | No scripts or configuration files are blocked, and the user receives no warnings or prompts. |
| Default | This sets the default execution policy, Restricted for Windows desktop machines and RemoteSigned for Windows servers. |
| RemoteSigned | Scripts can run but requires a digital signature on scripts that are downloaded from the internet. Digital signatures are not required for scripts that are written locally. |
| Restricted | This allows individual commands but does not allow scripts to be run. All script file types, including configuration files (.ps1xml), module script files (.psm1), and PowerShell profiles (.ps1) are blocked. |
| Undefined | No execution policy is set for the current scope. If the execution policy for ALL scopes is set to undefined, then the default execution policy of Restricted will be used. |
| Unrestricted | This is the default execution policy for non-Windows computers, and it cannot be changed. This policy allows for unsigned scripts to be run but warns the user before running scripts that are not from the local intranet zone. |

The execution policy can be bypassed with the following methods:
- Typing a script's contents directly into the PowerShell window
- Downloading and invoking a script
- Specifying a script as an encoded command
- Adjusting the execution policy (with the proper permissions)
- Define a current process's scope:
  - `Set-ExecutionPolicy Bypass -Scope Process`
***

## Windows Management Instrumentation (WMI)

WMI is a subsystem of PowerShell that provides system administrators with powerful tools for system monitoring. The goal of WMI is to consolidate device and application management across corporate networks. WMI has the following components:

| Component Name | Description |
| -------------- | ----------- |
| WMI service | The Windows Management Instrumentation process, which runs automatically at boot and acts as an intermediary between WMI providers, the WMI repository, and managing applications. |
| Managed objects | Any logical or physical components that can be managed by WMI. |
| WMI providers | Objects that monitor events/data related to a specific object. |
| Classes | These are used by the WMI providers to pass data to the WMI service. |
| Methods | These are attached to classes and allow actions to be performed. For example, methods can be used to start/stop processes on remote machines. |
| WMI repository | A database that stores all static data related to WMI. |
| CMI Object Manager | The system that requests data from WMI providers and returns it to the application requesting it. |
| WMI API | Enables applications to access the WMI infrastructure. |
| WMI Consumer | Sends queries to objects via the CMI Object Manager. |

WMI can be run via the Windows command prompt by typing WMIC to open an interactive shell or by running a command directly such as `wmic computersystem get name` to get the hostname.
- To view a listing of WMIC commands and aliases by typing `wmic /?`
- List details of the operating system: `wmic os list brief`
- WMIC uses aliases and associated verbs, adverbs, and switches.

PowerShell has WMIC cmdlets, such as: `Get-WmiObject -Class Win32_OperatingSystem | select SystemDirectory,BuildNumber,SerialNumber,Version | ft`
- `Invoke_WmiMethod` is used to call the methods of WMI objects.
  - Rename a file: `Invoke-WmiMethod -Path "CIM_DataFile.Name='C:\users\public\spns.csv'" -Name Rename -ArgumentList "C:\Users\Public\kerberoasted_users.csv"`
***

## Windows Subsystem for Linux (WSL)

WSL is a feature that allows Linux binaries to be run natively on Windows 10 and Windows Server 2019. It was originally intended for developers who needed to run Bash, Ruby, and native Linux command-line tools such as `sed`, `awk`, `grep`, etc., directly on their Windows workstation. The second version of WSL, released in May 2019, introduced a real Linux kernel utilizing a subset of Hyper-V features.

WSL can be installed by running the PowerShell command `Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux` as an Administrator. Once this feature is enabled, we can either download a Linux distro from the Microsoft Store and install it or manually download the Linux distro and install it from the command line.

WSL installs an application called `Bash.exe`, which can be run by merely typing `bash` into a Windows console to spawn a Bash shell. You can access the `C$` volume and other volumes on the host operating system via the `mnt` directory.
***

## Windows Server Core

Windows Server Core was first released with Windows Server 2008 as a minimalistic Server environment only containing key Server functionality. As a result, Server Core has lower management requirements, a smaller attack surface, and uses less disk space and memory than its Desktop Experience (GUI) counterpart. In Server Core, all configuration and maintenance tasks are performed via the command-line, PowerShell, or remote management with Microsoft Management Console (MMC) or Remote Server Administration Tools (RSAT).

While Server Core aims to have a smaller footprint by lacking a GUI, some graphical programs are still supported, such as Registry Editor, Notepad, System Information, Windows Installer, Task Manager, and PowerShell. It also supports some Sysinternals suite tools such as Active Directory Explorer, Process Explorer, Process Monitor, and TCPView.

As of Windows Server 2019, Server Core or Desktop Experience must be selected at installation, and neither can be rolled back (i.e., converting Server Core to Desktop Experience). Once installed, the initial setup for Server Core can be done via Sconfig, which is a text-based interface (actually a VBScript executed by WScript). Sconfig is used for performing a variety of common commands such as configuring networking, checking for/installing Windows updates, account management, configuring remote management, activating Windows, and more.

Certain server applications cannot run on Server Core, including Microsoft Server Virtual Machine Manager 2019 (SCVMM), System Center Data Protection Manager 2019, SharePoint Server 2019, Project Server 2019.

In summary, Server Core is lighter weight and less resource-intensive but has a steeper learning curve and can be more difficult to manage. It also has some limitations, such as performing management tasks using certain GUI programs.

Below are a list of applications available on Server Core:

| Application | Server Core | Desktop Experience |
| ----------- | ----------- | ------------------ |
| Command Prompt | Available | Available |
| Windows PowerShell/Microsoft .NET | Available | Available |
| Regedit | Available | Available |
| Diskmgmt.msc | Not Available | Available |
| Server Manager | Not Available | Available |
| Mmc.exe | Not Available | Available |
| Eventvwr | Not Available | Available |
| Services.msc | Not Available | Available |
| Control Panel | Not Available | Available |
| Windows Explorer | Not Available | Available |
| Taskmgr | Available | Available |
| Internet Explorer or Edge | Not Available | Available |
| Remote Desktop Services | Available | Available |

***

## Windows Security

### Security Identifier (SID)

Each of the security principles on the system has a unique security identifier (SID). The system automatically generates SIDs. Even with identical users, Windows can distinguish between them and their rights based on their SIDs. SIDs are string values with different lengths, which are stored in the security database. These SIDs are added to the user's access token to identify all actions that the user is authorized to take.

A SID consists of the Identifier Authority and the Relative ID (RID). In an Active Directory (AD) domain environment, the SID also includes the domain SID.
- Show your SID with: `whoami /user`
  - `S-1-5-21-674899381-4069889467-2080702030-1002`

| Number | Meaning | Description |
| ------ | ------- | ----------- |
| S | SID | Identifies the string as a SID. |
| 1 | Revision Level | To date, this has never changed and has always been 1. |
| 5 | Identifier-authority | A 48-bit string that identifies the authority (the computer or network) that created the SID. |
| 21 | Subauthority1 | This is a variable number that identifies the user's relation or group described by the SID to the authority that created it. It tells us in what order this authority created the user's account. |
| 674899381-4069889467-2080702030 | Subauthority2 | Tells us which computer (or domain) created the number |
| 1002 | Subauthority3 | The RID that distinguishes one account from another. Tells us whether this user is a normal user, a guest, an administrator, or part of some other group |

The SID is broken down into this pattern:
- `(SID)-(revision level)-(identifier-authority)-(subauthority1)-(subauthority2)-(etc)`

**SIDs for Specific User Accounts**:
- **Administrator**: S-1-5-domain-500
- **Guest**: S-1-5-32-546

### Security Accounts Manager (SAM) and Access Control Entries (ACE)

SAM grants rights to a network to execute specific processes.

The access rights themselves are managed by Access Control Entries (ACE) in Access Control Lists (ACL). The ACLs contain ACEs that define which users, groups, or processes have access to the file or to execute a process, for example.

The permissions to access a securable object are given by the security descriptor, classified into two types of ACLs: the **Discretionary Access Control List (DACL)** or **System Access Control List (SACL)**. Every thread and process started or initiated by a user goes through an authorization process. An integral part of this process is access tokens, validated by the Local Security Authority (LSA). In addition to the SID, these access tokens contain other security-relevant information.

### User Account Control (UAC)

UAC is a security feature in Windows to prevent malware from running or manipulating processes that could damage the computer or its contents. There is the Admin Approval Mode in UAC, which is designed to prevent unwanted software from being installed without the administrator's knowledge or to prevent system-wide changes from being made.

### Registry

The Registry is a hierarchical database in Windows that stores low-level settings for the Windows operating system and applications that choose to use it. It is divided into computer-specific and user-specific data. You can open the Registry Editor by typing `regedit` from the command line or Windows search bar.

The tree-structure consists of main folders (root keys) in which subfolders (subkeys) with their entries/files (values) are located. There are 11 different types of values that can be entered in a subkey.

| Value | Type |
| ----- | ---- |
| REG_BINARY | Binary data in any form. |
| REG_DWORD | A 32-bit number. |
| REG_DWORD_LITTLE_ENDIAN | A 32-bit number in little-endian format. Windows is designed to run on little-endian computer architectures. Therefore, this value is defined as REG_DWORD in the Windows header files. |
| REG_DWORD_BIG_ENDIAN | A 32-bit number in big-endian format. Some UNIX systems support big-endian architectures. |
| REG_EXPAND_SZ | A null-terminated string that contains unexpanded references to environment variables (for example, "%PATH%"). It will be a Unicode or ANSI string depending on whether you use the Unicode or ANSI functions. To expand the environment variable references, use the ExpandEnvironmentStrings function. |
| REG_LINK | A null-terminated Unicode string containing the target path of a symbolic link created by calling the RegCreateKeyEx function with REG_OPTION_CREATE_LINK. |
| REG_MULTI_SZ | A sequence of null-terminated strings, terminated by an empty string (\0). The following is an example: String1\0String2\0String3\0LastString\0\0 The first \0 terminates the first string, the second to the last \0 terminates the last string, and the final \0 terminates the sequence. Note that the final terminator must be factored into the length of the string. |
| REG_NONE | No defined value type. |
| REG_QWORD | A 64-bit number. |
| REG_QWORD_LITTLE_ENDIAN | A 64-bit number in little-endian format. Windows is designed to run on little-endian computer architectures. Therefore, this value is defined as REG_QWORD in the Windows header files. |
| REG_SZ | A null-terminated string. This will be either a Unicode or an ANSI string, depending on whether you use the Unicode or ANSI functions. |

Each folder under `Computer` is a key. The root keys all start with `HKEY`. A key such as `HKEY-LOCAL-MACHINE` is abbreviated to `HKLM`. HKLM contains all settings that are relevant to the local system. This root key contains six subkeys like `SAM`, `SECURITY`, `SYSTEM`, `SOFTWARE`, `HARDWARE`, and `BCD`, loaded into memory at boot time (except `HARDWARE` which is dynamically loaded).

The entire system registry is stored in several files on the operating system. You can find these under `C:\Windows\System32\Config\`.

The user-specific registry hive (HKCU) is stored in the user folder (i.e., `C:\Windows\Users\<USERNAME>\Ntuser.dat`).
- `gci -Hidden`

### Run on RunOnce Registry Keys

There are also so-called registry hives, which contain a logical group of keys, subkeys, and values to support software and files loaded into memory when the operating system is started or a user logs in. These hives are useful for maintaining access to the system. These are called `Run` and `RunOnce` registry keys.

The Windows registry includes the following four keys:
- `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run`
- `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run`
- `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce`
- `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce`

### Application Whitelisting

An application whitelist is a list of approved software applications or executables allowed to be present and run on a system. The goal is to protect the environment from harmful malware and unapproved software that does not align with the specific business needs of an organization.

Blacklisting, in contrast, specifies a list of harmful or disallowed software/applications to block, and all others are allowed to run/be installed. Whitelisting is based on a "zero trust" principle in which all software/applications are deemed "bad" except for those specifically allowed.

### AppLocker

AppLocker is Microsoft's application whitelisting solution and was first introduced in Windows 7. AppLocker gives granular control over executables, scripts, Windows installer files, DLLs, packaged apps, and packed app installers.

It allows for creating rules based on file attributes such as the publisher's name (which can be derived from the digital signature), product name, file name, and version. Rules can also be set up based on file paths and hashes. Rules can be applied to either security groups or individual users, based on the business need. AppLocker can be deployed in audit mode first to test the impact before enforcing all of the rules.

### Local Group Policy

Group Policy allows administrators to set, configure, and adjust a variety of settings. In a domain environment, group policies are pushed down from a Domain Controller onto all domain-joined machines that Group Policy objects (GPOs) are linked to. These settings can also be defined on individual machines using Local Group Policy.

Group Policy can be configured locally, in both domain environments and non-domain environments. Local Group Policy can be used to tweak certain graphical and network settings that are otherwise not accessible via the Control Panel. It can also be used to lock down an individual computer policy with stringent security settings, such as only allowing certain programs to be installed/run or enforcing strict user account password requirements.

We can open the Local Group Policy Editor by opening the Start menu and typing `gpedit.msc`. The editor is split into two categories under Local Computer Policy: `Computer Configuration` and `User Configuration`.

### Windows Defender Antivirus

Windows Defender Antivirus (Defender), formerly known as Windows Defender, is built-in antivirus that ships for free with Windows operating systems. Defender comes with several features such as real-time protection, which protects the device from known threats in real-time and cloud-delivered protection, which works in conjunction with automatic sample submission to upload suspicious files for analysis. When files are submitted to the cloud protection service, they are "locked" to prevent any potentially malicious behavior until the analysis is complete. Another feature is Tamper Protection, which prevents security settings from being changed through the Registry, PowerShell cmdlets, or group policy.
- Use `Get-MpComputerStatus | findstr "True"` in PowerShell to check which protection settings are enabled.
