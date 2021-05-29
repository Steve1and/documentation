# Linux System Administration Concepts
***

## System Administration Tool and Concepts
The root user has complete control of the operation of a Linux System.
- By default, the root account has no password set in Ubuntu. This means that even though the account exists, you cannot log in using it or use `su` to become the root user. This adds an additional level of security to Ubuntu and requires you to use `sudo` before each command that you want to execute as the root user.

**Becoming root from the shell (su command)**
```
$ su
Password: ******
#
```
When you are prompted, type the root user's password. The prompt for the regular user ($) changes to the superuser prompt (#). At this point, you have full permission to run any command and use any file on the system. However, one thing that the `su` command doesn't do when used this way is read in the root user's environment. To fix this problem, use the `su` command with the dash (-) option instead.
```
$ su -
Password: ******
#
```
You can also use the `su` command to become a user other than root.
- `su - jsmith`
As the root user, however, after you type the `su` command to become another user, you don't need a password to continue.
- To return to the previous shell by exiting the current shell by typing `exit`.

**Gaining administrative access with sudo**
Using `sudoers` for any user or groups on the system, you can do the following:
- Assign root privilege for any command they run with `sudo`.
- Assign root privilege for a select set of commands.
- Give users root privilege without telling them the root password because they only have to provide their own user password to gain root privilege.
- Allow users, if you choose, to run `sudo` without entering a password at all.
- Track which users have run administrative commands on your system. (Using `su`, all you know is that someone with the root password logged in, whereas the `sudo` command logs which user runs an administrative command.)

Steps to add a user (joe) to the `sudoers` file:
1. As the root user, edit the `/etc/sudoers` file by running the `visudo` command:
  - `/usr/sbin/visudo`
  - The reason for using `visudo` is that the command locks the `/etc/sudoers` file and does some basic sanity checking of the file to ensure that it has been edited correctly.
2. Add the following line to allow joe to have full root privileges on the computer:
  - `joe      ALL=(ALL)     ALL`
  - This line causes joe to provide a password (his own password, not the root password) in order to use administrative commands. To allow joe to have that privilege without using a password, type the following line instead:
    - `joe      ALL=(ALL)     NOPASSWD: ALL`
3. Save the changes to the `/etc/sudoers` file.

After entering his password successfully, joe can enter as many `sudo` commands as he wants for the next five minutes, on RHEL and Fedora systems, without having to enter it again. For Ubunutu, this is set to zero, for no time-out. (You can change the time-out value from five minutes to any length of time you want by setting the `passwd_timeout` value in the `/etc/sudoers` file.)

**Administrative Commands**
- `/sbin`: Originally contained commands needed to boot your system, including commands for checking filesystems (`fsck`) and turning on swap devices (`swapon`).
- `/usr/sbin`: Originally contained commands for such things as managing user accounts (such as `useradd`) and checking processes that are holding files open (such as `lsof`). Commands that run as daemon processes are also contained in this directory.

For the latest Ubuntu, RHEL, and Fedora releases, all administrative commands from the two directories are stored in the `/usr/sbin` directory (which is symbolically linked from `/sbin`).

**Administrative configuration files**
- `$HOME`: All users store in their home directories information that directs how their login accounts behave.
- `/etc`: This directory contains most of the basic Linux system configuration files.
- `/etc/cron*`: Directories in this set contain files that define how the `crond` utility runs applications on a daily (`cron.daily`), hourly (`cron.hourly`), monthly (`cron.monthly`), or weekly (`cron.weekly`) schedule.
- `/etc/cups`: Contains files used to configure the CUPS printing service.
- `/etc/default`: Contains files that set default values for various utilities. For example, the file for the `useradd` command defines the default group number, home directory, password expiration date, shell, and skeleton directory (`/etc/skel`) used when creating a new user account.
- `/etc/httpd`: Contains a variety of files used to configure the behavior of your Apache web server. (On Ubuntu and other Linux systems, `/etc/apache` or `/etc/apache2` is used instead.)
- `/etc/mail`: Contains files used to configure your `sendmail` mail transport agent.
- `/etc/postfix`: Contains configuration files for the `postfix` mail transport agent.
- `/etc/ppp`: Contains several configuration files used to set up Point-to-Point Protocol (PPP) so that you can have your computer dial out to the Internet.
- `/etc/rc?.d`: There is a separate `rc?.d` directory for each valid system state: `rc0.d` (shutdown state), `rc1.d` (single-user state), `rc2.d` (multi-user state), `rc3.d` (multi-user plus networking state), `rc4.d` (user-defined state), `rc5.d` (multi-user, networking, plus GUI login state), and `rc6.d` (reboot state).
- `/etc/security`: Contains files that set a variety of default security conditions for your computer, basically defining how authentication is done. These files are part of the `pam` (pluggable authentication modules) package.
- `/etc/skel`: Any files contained in this directory are automatically copied to a user's home directory when that user is added to the system.
- `/etc/sysconfig`: Contains important system configuration files that are created and maintained by various services (including `firewalld`, `samba`, and most networking services).
- `/etc/systemd`: Contains files associated with the `systemd` facility, for managing the boot process and system services.
- `/etc/xinetd.d`: Contains a set of files, each of which defines an on-demand network service that the `xinetd` daemon listens for on a particular port.
- `/etc/aliases`: Can contain distribution lists used by the Linux mail services. (This file is located in `/etc/mail` in Ubuntu when you install the `sendmail` package.)
- `/etc/bashrc`: Sets system-wide defaults for bash shell users. (This may be called `bash.bashrc` on some Linux distributions.)
- `/etc/crontab`: Sets times for running automated tasks and variables associated with the `cron` facility (such as the `SHELL` and `PATH` associated with `cron`).
- `/etc/csh.cshrc` (or `cshrc`): Sets system-wide defaults for `csh` (C shell) users.
- `/etc/exports`: Contains a list of local directories that are available to be shared by remote computers using the Network File System (NFS).
- `/etc/fstab`: Identifies the devices for common storage media (hard disk, DVD, CD-ROM, and so on) and locations where they are mounted in the Linux system.
- `/etc/group`: Identifies group names and group IDs (GIDs) that are defined on the system.
- `/etc/gshadow`: Contains shadow passwords for groups.
- `/etc/host.conf`: Used by older applications to set the locations in which domain names (for example, `redhat.com`) are searched for on TCP/IP networks (such as the Internet). By default, the local hosts file is searched and then any name server entries in `resolve.conf`.
- `/etc/hostname`: Contains the hostname for the local system (beginning in RHEL 7 and recent Fedora and Ubuntu systems).
- `/etc/hosts`: Contains IP addresses and hostnames that you can reach from your computer. (Usually this file is used just to store names of computers on your LAN or small private network).
- `/etc/inittab`: On earlier Linux systems, contained information that defined which programs start and stop when Linux boots, shuts down, or goes into different states in between. This file is no longer used on systems that support `systemd`.
- `/etc/mtab`: Contains a list of filesystems that are currently mounted.
- `/etc/mtools.conf`: Contains settings used by DOS tools in Linux.
- `/etc/named.conf`: Contains DNS settings if you are running your own DNS server (`bind` or `bind9` package).
- `/etc/nsswitch.conf`: Contains name service switch settings, for identifying where critical system information comes from.
- `/etc/ntp.conf`: Includes information for all valid users on the local system. Also includes other information such as the home directory and default shell.
- `/etc/passwd`: Stores account information for all valid users on the local system. Also includes other information, such as the home directory and default shell.
- `/etc/printcap`: Contains definitions for the printers configured for your computer.
- `/etc/profile`: Sets system-wide environment and startup programs for all users.
- `/etc/protocols`: Sets protocol numbers and names for a variety of Internet services.
- `/etc/rpc`: Defines remote procedure call names and numbers.
- `/etc/services`: Defines TCP/IP and UDP service names and their port assignments.
- `/etc/shadow`: Contains encrypted passwords for users who are defined in the `passwd` file.
- `/etc/shells`: Lists the shell command-line interpreters (`bash`, `sh`, `csh`, and so on) that are available on the system as well as their locations.
- `/etc/sudoers`: Sets commands that con be run by users, who may not otherwise have permission to run the command, using the `sudo` command.
- `/etc/rsyslog.conf`: Defines what logging messages are gathered by the `rsyslogd` daemon and in which files they are stored.
- `/etc/xinetd.conf`: Contains simple configuration information used by the `xinetd` daemon process.

Another directory, `/etc/X11`, includes subdirectories that each contain system-wide configuration files used by X and different X window managers available for Linux. The `xorg.conf` file (configures your computer and monitor to make it usable with X) and configuration directories containing files used by `xdm` and `xinit` to start X are in here.

**Administrative log files and systemd journal**
For Linux systems that don't use the `systemd` facility, the main utility for logging error and debugging messages is the `rsyslogd` daemon. (Some older Linux systems use `syslogd` and `syslogd` daemons). `systemd` has its own method of gathering and displaying messages called the `systemd` journal (`journalctl` command).

The primary command for viewing messages from the `systemd` journal is the `journalctl` command. The boot process, the kernel, and all `systemd`-managed services direct their status and error messages to the `systemd` journal.
- `journalctl`
- `journalctl --list-boots`
The `journalctl` command with no options lets you page through all messages in the `systemd` journal. To list the boot IDs for each time the system was booted, use the `-list-boots` option. To view messages associated with a particular boot instance, use the `-b` option with one of the boot instances. To see only kernel messages, use the `-k` option.
- `journalctl _SYSTEMD_UNIT=sshd.service`
- `journalctl PRIORITY=0`
- `journalctl -a -f`
Use the `_SYSTEMD_UNIT=` options to show messages for specific services. To see messages associated with a particular syslog log level, set `PRIORITY=` to a value from 0 to 7. To follow messages as they come in, use the `-f` option; to show all fields, use the `-a` option.

The `rsyslogd` facility, and its predecessor `syslogd`, gather log messages and direct them to log files or remote log hosts. Logging is done according to information in the `/etc/rsyslog.conf` file. Messages are typically directed to log files that are usually in the `/var/log` directory, but they can also be directed to log hosts for additional security. Here are a few common log files:
- `boot.log`: Contains boot messages about services as they start up.
- `messages`: Contains many general informational messages about the system.
- `secure`: Contains security-related messages, such as login activity or any other act that authenticates users.

### Using Other Administrative Accounts
Administrative logins are available with Linux; however, logging in directly as these users is disabled by default. The accounts are maintained primarily to provide ownership for files and processes associated with particular services.
- `lp`: User owns such things as the `/var/log/cups` printing log file and various printing cache and spool files. The home directory for `lp` is `/var/spool/lpd`.
- `apache`: User can set up content files and directories on an Apache web server.
- `avahi`: User runs the `avahi` daemon process to provide `zeroconf` services on your network.
- `chrony`: User runs the `chronyd` daemon, which is used to maintain accurate computer clocks.
- `postfix`: User owns various mail server spool directories and files. The user runs the daemon processes used to provide the postfix service (master).
- `bin`: User owns many commands in `/bin` in traditional UNIX systems. This is not the case in some Linux systems (such as Ubuntu, Fedora, and Gentoo) because root owns most executable files.
- `news`: User could do administration of Internet news services, depending on how you set permission for `/var/spool/news` and other news-related resources. The home directory for news is `/etc/news`.
- `/etc/rpc`: User runs the remote procedure calls daemon (`rpcbind`), which is used to receive calls for services on the host system.
By default, the administrative logins in the preceding list are disabled. You would need to change the default shell from its current setting (usually `/sbin/nologin` or `/bin/false`) to a real shell (typically `/bin/bash`) to be able to log in as these users.

### Checking and Configuring Hardware
The Udev subsystem dynamically names and creates devices as hardware comes and goes. It runs as the `udevd` daemon and creates and removes devices in the `/dev` directory.

When your system boots, the kernel detects your hardware and loads drivers that allow Linux to work with that hardware.
- Any user can run the `dmesg` command to see what hardware was detected and which drivers were loaded by the kernel at boot time.
- A second way to see boot messages is the `journalctl` command to show the messages associated with a particular boot instance.
  - You can use the `journalctl -f` command to follow messages as they come into the `systemd` journal.
- The `lspci` command lists PCI busses on your computer and devices connected to them.
- The host bridge connects the local bus to the other components on the PCI bridge.
- `lsusb` lists information about the computer's USB hubs along with any USB devices connected to the computer's USB ports
- To see details about your processor, run the `lscpu` command.

**Working with loadable modules**

Kernel modules are installed in `/lib/modules/` subdirectories. The name of each subdirectory is based on the release number of the kernel.
- To see which modules are currently loaded into the running kernel on your computer, use the `lsmod` command.
- To find information about any of the loaded modules, use the `modinfo` command.
  - `/sbin/modinfo -d e1000`
- You can load any module (as root user) that has been complied and installed (to a `/lib/modules` subdirectory) into your running kernel using the `modprobe` command.
  - The `modprobe` command loads modules temporarily; they disappear at the next reboot. To add the module to your subsystem permanently, add the `modprobe` command line to one of the startup scripts that are run at boot time.
- Use the `rmmod` command to remove a module from a running kernel.
  - With `modprobe -r`, instead of just removing the module you request, you can also remove dependent modules that are not being used by other modules.
***

## Getting and Managing Software
A *tarball* is a single file in which multiple files are gathered together for convenient storage or distribution.

**DEB (.deb) packaging**: The Debian GNU/Linux project created `.deb` packaging which is used by Debian and other distributions based on Debian.
- Debian software packages hold multiple files and metadata related to some set of software in the format of an `ar` archive file. The files can be executables (commands), configuration files, documentation, and other software items. The metadata includes such things as dependencies, licensing, package sizes, descriptions, and other information.
- **aptitude**: The `aptitude` command is a package installation tool that provides a screen-oriented menu that runs in the shell.
- **apt**: There is a set of `apt*` commands (`apt-get`, `apt`, `apt-config`, `apt-cache`, and so on) that can be used to manage package installation.
- Example of installing the `vsftpd` package:
  - `apt-get update`: Get the latest package versions
  - `apt-cache search vsftpd`: Find package by key word
  - `apt-cache show vsftpd`: Display information about a package
  - `apt-get install vsftpd`: Install the vsftpd package
  - `apt-get upgrade`: Update installed packages if upgrade ready
  - `apt-cache pkgnames`: List all packages that are installed

**RPM (.rpm) packaging**: Originally named Red Hat Package Manager, but later recursively renamed RPM package manager, RPM is the preferred package format for SUSE, Red Had distributions, and those based on Red Hat distributions.
- An *RPM package* is a consolidation of files needed to provide a feature. The commands, configuration files, and documentation that make up the software feature can be inside an RPM. However, an RPM file also contains metadata that stores information about the contents of that package, where the package came from, what it needs to run, and other information.
- To find out the name of an RPM package currently installed on your system, use the `rpm -q` command:
  - `rpm -q firefox`
- Query the local RPM database about a command:
  - `rpm -qi firefox`
- The software included with Linux distributions, or built to work with those distributions, comes from thousands of open-source projects all over the world. These projects are referred to as *upstream software providers*.
- A Linux distribution takes the source code and builds it into binaries. Then it gathers those binaries together with documentation, configuration files, scripts, and other components available from the upstream provider.
- After all of those components are gathered into the RPM, the RPM package is signed (so that users can test the package for validity) and placed in a repository of RPMs for the specific distribution and architecture (32-bit x86, 64-bit x86, and so on). The repository is placed on an installation CD or DVD or in a directory that is made available as an FTP, web, or NFS server.

### Managing RPM Packages with YUM
**YUM**: YellowDog Updater Modified

With repositories, the problem of dealing with dependencies fell not to the person who installed the software but to the Linux distribution or third-party software distributor that makes the software available.
- The locations of these repositories would then be stored on the user's system in the `/etc/yum.conf` file or, more typically, in separate configuration files in the `/etc/yum.repos.d` directory.

**DNF**: Dandified YUM
- While `dnf` maintains a basic command-line compatibility with yum, one of its main differences is that it adheres to a strict API. That API encourages the development of extensions and plug-ins to `dnf`.

Basic YUM command syntax: `yum [options] command`

**Phases of the YUM Install Process**
1. Checking `/etc/yum.conf`
  - When any `yum` command starts, it checks the file `/etc/yum.conf` for default settings. The `/etc/yum.conf` file is the basic YUM configuration file. You can also identify the location of repositories here, although the `/etc/yum.repos.d` directory is the more typical location for identifying repositories.
  - The `gpgcheck` is used to validate each package against a key that you receive from those who built the RPM. It is on by default (`gpgcheck=1`). For packages in Fedora or RHEL, the key comes with the distribution to check all packages. To install packages that are not from your distribution, you need either to import the key to verify those packages or to turn off that feature (`gpgcheck=0`).
2. Checking `/etc/yum.repos.d/*.repo` files:
  - Software repositories can be enabled by dropping files ending in `.repo` into the `/etc/yum.repos.d/` directory that point to the location of one or more repositories.
  - The `name` line contains a human-readable description of the repository. The `baseurl` line identifies the directory containing the RPM files, which can be an `httpd://`, `ftp://`, or `file://` entry.
  - The `enabled` line indicates whether the entry is active. A `1` is active; `0` is inactive. If there is no `enabled` line, the entry is active.
  - The `gpgkey` line shows the location of the key that is used to check the packages in this repository.
3. Downloading RPM packages and metadata from a YUM repository:
  - After `yum` knows the locations of the repositories, metadata from the `repodata` directory of each repository is downloaded to the local subsystem.
  - Metadata information is stored on the local system in the `/var/cache/yum` directory.
4. Installing RPM packages to Linux file system:
  - After all of the necessary packages are downloaded to the cache directories, yum runs `rpm` commands to install each package.
5. Storing YUM repository metadata to local RPM database.
  - The metadata contained in each RPM package that is installed is ultimately copied into the local RPM database. The RPM database is contained in files that are stored in the `/var/lib/rpm` directory.

**Searching for packages**
Using different searching subcommands, you can find packages based on key words, package contents, or other attributes.
- `yum search editor`: searches for packages with the keyword editor.
- To get information about a package, use the `info` command:
  - `yum info emacs`
- If you know the command, configuration file, or library name you want but don't know what package it is in, use the `provides` subcommand to search for the package.
  - `yum provides dvdrecord`
- The `list` subcommand can be used to list package names in different ways. Use it with a package base name to find the version and repository for a package. You can list just packages that are available or installed, or you can list all packages.
  - `yum list emacs`
  - `yum list available`
  - `yum list installed`
  - `yum list all`
- With `deplist`, you can see the components (dependency) but also the package that component comes in (provider).
  - `yum deplist emacs | less`

**Installing and removing packages**
- The `install` subcommand lets you install one or more packages, along with any dependent packages needed.
  - `yum install emacs`
- You can reinstall a package if you mistakenly delete components of an installed package. If you attempt a regular install, the system responds with "nothing to do". You must instead use the `reinstall` subcommand.
  - `yum reinstall zsh`
- You can remove a single package, along with its dependencies that aren't required by other packages, with the `remove` subcommand.
  - `yum remove emacs`
- An alternative method to remove a set of packages that you have installed is to use the `history` subcommand. All of the packages that you have installed can be uninstalled using the `undo` option of the `history` subcommand.
  - `yum history`
  - `yum history info 12`
  - `yum history undo 12`

**Updating packages**
If a new version of a package shows up later, you can download and install the new version of the package by using the `update` subcommand.
- The `check-update` subcommand can check for updates.
  - `yum check-update`
  - `yum update`
  - `yum update cups`

**Updating groups of packages**
To make it easier to manage a whole set of packages at once, YUM supports package groups.
- `yum grouplist | less`
- `yum groupinfo LXDE`
- If you decide that you want to install a package group, use the `groupinstall` subcommand:
  - `yum groupinstall LXDE`
- If you decide that you don't like the group of packages, you can remove the entire group at once using the `groupremove` subcommand:
  - `yum groupremove LXDE`

**Maintaining your RPM package database and cache**
Metadata stored in cache directories can be cleared, causing fresh metadata to be downloaded from all enabled YUM repositories the next time `yum` is run.
- `yum clean packages`
- `yum clean metadata`
- `yum clean all`
- You can check the RPM database to look for errors (`yum check`) or just rebuild the RPM database files.
  - `yum check`
  - `rpm --rebuilddb`
  - `yum check`

**Downloading RPMs from a YUM repository**
To download the latest version of the Firefox web browser package with `yum-downloader` from the YUM repository to your current directory, type the following:
- `yumdownloader firefox`
To use the `dnf` command, type this:
- `dnf download firefox`

### Installing, Querying, and Verifying Software with the rpm Command
Because the database contains fingerprints (md5sums) of every file in every package, it can be queried with RPM to find out if files from any packages have been tampered with.

**Installing and removing packages with rpm**
- `rpm -i zsh-5.5.1-6.e18.x86_64.rpm`
If an earlier version of `zsh` were installed, you could upgrade the package using `-U`. Often, people use `-h` and `-v` options to getr hash signs printed and more verbose output during the upgrade:
- `rpm -Uhv zsh-5.5.1.-6.e18.x86_64.rpm`
A third type of install, called freshen (`-F`), installs a package only if an existing, earlier version of a package is installed on the computer, as in this example:
- `rpm -Fhv *.rpm`
The `--replacepkgs` option enables you to reinstall an existing version of a package (if, for example, you had mistakenly deleted some components), and the `--oldpackage` enables you to replace a newer package with an earlier version.
- `rpm -Uhv --replacepkgs emacs-26.1-5.e18.x86_64.rpm`
- `rpm -Uhv --oldpackage zsh-5.0.2-25.e17_3.1.x86_64.rpm`
You can remove a package with the `-e` option.
- `rpm -e emacs`

**Querying rpm information**
Using the `-q` option, you can see information about the package including a description (`-qi`), list of files (`-ql`), documentation (`-qd`), and configuration files (`-qc`).
- `rpm -qi zsh`
- `rpm -ql zsh`
- `rpm -qd zsh`
- `rpm -qc zsh`
- You can find what an RPM needs for it to be installed (`--requires`), what version of software a package provides (`--provides`), what scripts are run before and after an RPM is installed or removed (`--scripts`), and what changes have been made to an RPM (`--changelog`).
  - `rpm -q --requires emacs-common`
  - `rpm -q --provides emacs-common`
  - `rpm -q --scripts httpd`
  - `rpm -q --changelog httpd | less`
- Using a feature called `--queryformat`, you can query different tags of information and output them in any form you like.
  - `rpm --querytags | less`
- By adding a `-p` to those query options, you can query an RPM file sitting in your local directory instead.
  - `rpm -qip zsh-5.7.1-1.fc30.x86_64.rpm`: View info about the RPM file.
  - `rpm -qlp zsh-5.7.1-1.fc30.x86_64.rpm`: List all files in the RPM file.
  - `rpm -qdp zsh-5.7.1-1.fc30.x86_64.rpm`: Show docs in the RPM file.
  - `rpm -qcp zsh-5.7.1-1.fc30.x86_64.rpm`: List config files in the RPM file

**Verifying RPM Packages**
Using the `-V` option, you can check the packages installed on your system to see if the components have been changed since the packages were first installed.
- Each time that you see a letter or a number instead of a dot from the `rpm -V` output, it is an indication of what has changed. Letters that can replace the dots (in order) include the following:
  - `S  file Size differs`
  - `M  Mode differs (includes permissions and file type)`
  - `5  MD5 sum differs`
  - `D  Device major/minor numbers mismatch`
  - `L  readLink(2) path mismatch`
  - `U  User ownership differs`
  - `G  Group ownership differs`
  - `T  mTime differs`
  - `P  capabilities differ`
- To restore the package to its original state, use `rpm` with the `--replacepkgs` option, as shown next. Then check it with `-V` again. No output from `-V` means that every file is back to its original state.
  - `rpm -i --replacepkgs zsh-5.7.1-1.fc30.x86_64.rpm`
  - `rpm -V zsh`
***

## Managing User Accounts
*User accounts* keep boundaries between the people who use your systems and between the processes that run on your systems. *Groups* are a way of assigning rights to your system that can be assigned to multiple users at once.

**Adding users with useradd**
The most straightforward method for creating a new user from the shell is the `useradd` command. The only required parameter is the login name of the user.
- `-c "comment"`: Provide a description of the new user account. Use quotes to enter multiple words (for example, `-c "Jake Jackson"`).
- `-d home_dir`: Set the home directory to use for the account. The default is to name it the same as the login name and to place it in `/home`.
- `-D`: Rather than create a new account, save the supplied information as the new default settings for any new accounts that are created.
- `-e expire_date`: Assign the expiration date for the account in *YYYY-MM-DD* format.
- `-f -1`: Set the number of days after a password expires until the account is permanently disabled. The default, -1, disables the option. Setting this to 0 disables the account immediately after the password has expired.
- `-g group`: Set the primary group (it must already exist in the `/etc/group` file) the new user will be in. Without this option, a new group is created that is the same as the username and is used as that user's primary group.
- `-G grouplist`: Add the nwe user to the supplied comma-separated list of supplementary groups (for example, `-G wheel,sales,tech,lunch`).
- `-k skel_dir`: Set the skeleton directory containing initial configuration files and login scripts that should be copied to a new user's home directory. This parameter can be used only in conjunction with the `-m` option. (Without this option, the `/etc/skel` directory is used).
- `-M`: Do not create the new user's home directory, even if the default behavior is set to create it.
- `-n`: Turn off the default behavior of creating a new group that matches the name and user ID of the new user.
- `-o`: Use with `-u uid` to create a user account that has the same UID as another username.
- `-p passwd`: Enter a password for the account you are adding. This must be an encrypted password. (To generate an encrypted MD5 password, type `openssl passwd`).
- `-s shell`: Specify the command shell to use for this account.
- `-u user_id`: Specify the user ID number for the account (for example, `-u 1793`). Without the `-u` option, the default behavior is to assign the next available number automatically. User IDs that are automatically assigned to regular users begin at 1000.

In creating a new user account, the `useradd` command performs several actions:
1. Reads the `/etc/login.defs` and `/etc/default/useradd` files to get default values to use when creating accounts.
2. Checks command-line parameters to find out which default values to override.
3. Creates a new user entry in the `/etc/passwd` and `/etc/shadow` files based on the default values and command-line parameters.
4. Creates any new group entries in the `/etc/group` file. (Fedora creates a group using the new user's name.)
5. Creates a home directory based on the user's name in the `/home` directory.
6. Copies any files located within the `/etc/skel` directory to the new home directory to the new home directory. This usually includes login and application startup scripts.

**Setting user defaults**
The `useradd` command determines the default values for new accounts by reading the `/etc/login.defs` and `/etc/default/useradd` files. You can modify those defaults by editing the files manually with a standard text editor. You can also see default settings by typing the `useradd` command with the `-D` option. You can also use the `-D` option to change defaults. When run with this flag, `useradd` refrains from actually creating a new user account; instead, it saves any additionally supplied options as the new default values in `/etc/default/useradd`.
- `-b default_home`: Set the default directory in which user home directories are created.
- `-e default_expire_date`: Set the default expiration date on which the user account is disabled in the *YYYY-MM-DD* format.
- `-f default_inactive`: Set the number of days after a password has expired before the account is disabled.
- `-g default_group`: Set the default group in which new users will be placed.
- `-e default_shell`: Set the default shell for new users.

**Modifying users with usermod**
The `usermod` command provides a simple and straightforward method for changing account parameters.
- `-c username`: Change the description associated with the user account.
- `-d home_dir`: Change the home directory to use for the account.
- `-e expire_date`: Assign a new expiration date for the account in *YYYY-MM-DD* format.
- `-f -1`: Change the number of days after a password expires until the account is permanently disabled. The default, `-1`, disables the option. Setting this to 0 disables the account immediately after the password has expired.
- `-g group`: Change the primary group (as listed in the `/etc/group` file) the user will be in.
- `-G grouplist`: Set the user's secondary groups to the supplied comma-separated list of groups. If the user is already in at least one group besides the user's private group, you must add the `-a` option as well (`-Ga`). If not, the user belongs to only the new set of groups and loses membership to any previous groups.
- `-l login_name`: Change the login name of the account.
- `-L`: Lock the account by putting an exclamation point at the beginning of the encrypted password in the `/etc/shadow` file.
- `-m`: Available only when `-d` is used. This causes the contents of the user's home directory to be copied to the new directory.
- `-o`: Use only with `-u uid` to remove the restriction that UIDs must be unique.
- `-s shell`: Specify a different command shell to use for this account.
- `-u user_id`: Change the user ID number for the account.
- `-U`: Unlocks a user's account.

**Deleting users with userdel**
`userdel` is used to remove users. The `-r` option removes the user's home directory as well.
- Before you delete the user, you may want to run a `find` command to find all files that would be left behind by the user.
  - `find / -user chris -ls`
  - `find / -uid 504 -ls`
- Here's an example of a `find` command that finds all files in the filesystem that are not associated with any user (the files are listed by UID):
  - `find / -nouser -ls`

### Understanding Group Accounts
Here are a few facts about using groups:
- When a user creates a file or directory, by default, that file or directory is assigned to the user's primary group.
- The user can belong to zero or more supplementary groups.
- The user can't add themselves to a supplementary group.
- If a user wants to create a file with a group they don't belong to associated with it, they can use the `newgrp` command.
  - Someone with root permission can use `gpasswd` to set a group password (such as `gpasswd sales`). After that, any user can type `newgrp sales` into a shell to temporarily use `sales` as their primary group by simply entering the group password when prompted.

**Creating group accounts**
As the root user, you can create new groups from the command line with the `groupadd` command.
- `groupadd kings`
- `groupadd -g 1325 jokers`

To change a group later, use the `groupmod` command.
- `groupmod -g 330 jokers`
- `groupmod -n jacks jokers`
- In the first example, the group ID for `jokers` is changed to 330. In the second, the name `jokers` is changed to `jacks`.

### Setting permissions with Access Control Lists
The *Access Control List (ACL)* feature was created so that regular users could share their files and directories selectively with other users and groups.
- For ACLs to be used, they must be enabled on a filesystem when that filesystem is mounted.
- In Fedora and Red Hat Enterprise Linux, ACLs are automatically enabled on any filesystem created when the system is installed.
- If you create a filesystem after installation (such as when you add a hard disk), you need to make sure that the `acl` mount option is used when the filesystem is mounted.
- To add ACLs to a file, you use the `setfacl` command; to view ACLs set on a file, you use the `getfacl` command.
- To set ACLs on any file or directory, you must be the actual owner (user) assigned to it.
- Because multiple users and groups can be assigned to a file/directory, the actual permission a user has is based on a union of all user/group designations to which they belong.

**Setting ACLs with setfacl**
Using the `setfacl` command, you can modify permissions (`-m`) or remove ACL permissions (`-x`).
- `setfacl -m u:username:rwx filename`
- In the example just shown, the modify option (`-m`) is followed by the letter `u`, indicating that you are setting ACL permissions for a user. After a colon (:), you indicate the username, followed by another colon and the permissions that you want to assign. As with the `chmod` command, you can assign read (r), write (w), and/or execute (x) permissions to the user or group (in the example, full rwx permission is given). The last argument is replaced by the actual filename you are modifying.
```
touch /tmp/memo.txt
ls -l /tmp/memo.txt
setfacl -m u:bill:rw /tmp/memo.txt
setfacl -m g:sales:rw /tmp/memo.txt
getfacl /tmp/memo.txt
```
The plus sign indicates that ACLs are set on a file, so you know to run the `getfacl` command to see how ACLs are set.
- As soon as you set ACLs on a file, the regular group permissions on the file sets a mask of the maximum permission an ACL user or group can have on a file. So, even if you provide an individual with more ACL permissions than the group permissions allow, the individual's effective permissions do not exceed the group permissions.

**Setting default ACLs**
Setting default ACLs on a directory enables your ACLs to be inherited. To set a user or group ACL permission as the default, you add a `d:` to the user or group designation.
```
mkdir /tmp/test
setfacl -m d:g:market:rwx /tmp/test
getfacl /tmp/test
```

**Enabling ACLs**
You can add the `acl` mount option in several ways:
- Add the `acl` option to the fifth field in the line in the `/etc/fstab` file that automatically mounts the filesystem when the system boots up.
- Implant the `acl` line in the `Default mount options` field in the filesystem's super block, so that the `acl` option is used whether the filesystem is mounted automatically or manually.
- Add the `acl` option to the `mount` command line when you mount the filesystem manually with the `mount` command.

To check that the `acl` option has been added to an ext filesystem, determine the device name associated with the filesystem, and run the `tune2fs -l` command to view the implanted mount options.
- `tune2fs -l /dev/mapper/mybox-home | grep "mount options"`

If the `Default mount options` filed is blank (such as when you have just created a new filesystem), you can add the `acl` mount option using the `tune2fs -o` command.
- `tune2fs -o acl /dev/sdc1`
- `tune2fs -l /dev/sdc1 | grep "mount options"`

A second way to add `acl` support to a filesystem is to add the `acl` option to the line in the `/etc/fstab` file that automatically mounts the filesystem at boot time.
- `/dev/sdc1    /var/stuff    ext4    ac1   1 2`

If the filesystem were already mounted, you could type the following `mount` command as root to remount the filesystem using `acl` or any other values added to the `/etc/fstab` file:
- `mount -o remount /dev/sdc1`

A third way that you can add ACL support to a filesystem is to mount the filesystem by hand and specifically request the `acl` mount option. Type the following command to mount the filesystem and include ACL support:
- `mount -o acl /dev/sdc1 /var/stuff`

**Adding directories for users to collaborate**
A special set of three permission bits are typically ignored when you use the `chmod` command to change permissions on the filesystem. These bits can set special permissions on commands and directories. As with read, write, and execute bits for `user`, `group`, and `other`, these special file permission bits can be set with the `chmod` command. If, for example, you run `chmod 755 /mnt/xyz`, the implied permission is actually `0775`. To change permissions, you can replace the number 0 with any combination of those three bits (4, 2, and 1), or you can use letter values instead.

| Name | Numeric Value | Letter Value |
| ---- | ------------- | ------------ |
| Set user ID bit | 4 | u+s |
| Set group ID bit | 2 | g+s |
| Sticky bit | 1 | o+t |

- `chmod 2775 /tmp/test`

**Creating group collaboration directories (set GID bit)**
When you create a set GID directory, any files created in that directory are assigned to the group assigned to the directory itself. The idea is to have a directory where all members of a group can share files but still protect them from other users.
- A set UID command owned by root would run with root permissions, a set GID command owned by apache would have apache group permissions.

**Creating restricted deletion directories (sticky bit)**
A *restricted deletion directory* is created by turning on a directory's sticky bit. Normally, if write permission is open to a user on a file or directory, that user can delete that file or directory. However, in a restricted deletion directory, unless you are the root user or the owner of the directory, you can never delete another user's files.

### Centralizing User Accounts
Authentication domains that are supported via the `authconfig` command include LDAP, NIS, and Windows Active Directory.
- **LDAP**: The *Lightweight Directory Access Protocol (LDAP)* is a popular protocol for providing directory services (such as phone books, addresses, and user accounts).
- **NIS**: The *Network Information Service (NIS)* was originally created by Sun Microsystems to propagate information such as user accounts, host configuration, and other types of system information across many UNIX systems. (Sends in cleartext, do not use).
- **Winbind**: Selecting *Winbind* from the Authentication Configuration window enables you to authenticate your users against a Microsoft Active Directory (AD) server.
***

## Managing Disks and Filesystems

### Understanding Disk Storage
When you install the operating system, the disk is divided into one or more partitions. Each partition is formatted with a filesystem. In the case of Linux, some of the partitions may be specially formatted for elements such as swap area or LVM physical volumes.

A *swap space* is a hard disk swap partition or a swap file where your computer can "swap out" data from RAM that isn't being used at the moment and then "swap in" the data back to RAM when it is again needed.

Another special partition is a *Logical Volume Manager (LVM)* physical volume. LVM physical volumes enable you to create pools of storage space called *volume groups*.

For Linux, at least one disk partition is required, assigned to the root (`/`) of the entire Linux filesystem.
- The word *mount* refers to the action of connecting a filesystem from a hard disk, USB drive, or network storage device to a particular point in the filesystem. This action is done using the `mount` command, along with options to tell the command where the storage device is located and to which directory in the filesystem to connect it.

Each disk partition created when you install Linux is associated with a device name. An entry in the `/etc/fstab` file tells Linux each partition's device name and where to mount it (as well as other bits of information). The mounting is done when the system boots.

### Partitioning Hard Disks

**Understanding partition tables**

PC architecture computers have traditionally used *Master Boot Record (MBR) partition tables* to store information about the sizes and layouts of the hard disk partitions.
- A few years ago, however, a new standard called *Globally Unique Identifier (GUID) partition tables* began being used on systems as part of the UEFI computer architecture to replace the older BIOS method of booting the system.
- MBR partitions are limited to 2TB in size. GUID partitions can create partitions up to 9.4ZB (zettabytes).

**Viewing disk partitions**

To view disk partitions, use the `parted` command with the `-l` option.
- `parted -l /dev/sda`
- When a USB flash drive is inserted, it is assigned to the next available `sd` device.
  - `fdisk -l /dev/sdb`
  - A SCSI or USB storage device, represented by an `sd?` device (such as `sda`, `sdb`, `sdc`, and so on) can have up to 16 minor devices (for example, the main `/dev/sdc` device and `/dev/sdc1` through `/dev/sdc15`). So, there can be 15 partitions total. A NVMe SSD storage device, represented by a `nvme` device (such as `nvme0`, `nvme1`, `nvme2`, and so on) can be divided into one or more namespaces (most devices just use the first namespace) and partitions. For example, `/dev/nvme0n1p1` represents the first partition in the first namespace on the first NVMe SSD.
  - For x86 computers, disks can have up to four primary partitions. So, to have more than four total partitions, one must be an extended partition. Any partitions beyond the four primary partitions are logical partitions that use space from the extended partition.

**Creating a single-partition disk**
1. For a USB flash drive, just plug it into an available USB port.
2. Determine the device name for the USB drive. As root user from a shell, type the following `journalctl` command, and then insert the USB flash drive.
  - `journalctl -f`
3. If the USB flash drive mounts automatically, unmount it.
  - `mount | grep sdb`
  - `umount /dev/sdb1`
4. Use the `parted` command to create partitions on the USB drive.
  - `parted /dev/sdb`
5. Use `p` to view all partitions and `rm` to delete the partition.
  - `(parted) p`
  - `(parted) rm`
  - `Partition number? 1`
6. Relabel the disk as having a gpt partition table.
  - `(parted) mklabel gpt`
  - `Yes/No? Yes`
7. To create a new partition, type `mkpart`. You are prompted for the file system type, then the start and end of the partition.
  - `(parted) mkpart`
8. Double-check that the drive is partitioned the way you want by pressing `p`.
  - `(parted) p`
9. Although the partitioning is done, the new partition is not yet ready to use. For that, you have to create a filesystem on the new partition. To create a filesystem on the new disk partition, use the `mkfs` command.
  - `mkfs -t xfs /dev/sdb1`
10. To be able to use the new filesystem, you need to create a mount point and mount it to the partition.
  - `mkdir /mnt/test`
  - `mount /dev/sdb1 /mnt/test`
  - `df -h /mnt/sdb1`
  - Any files or directories that you create later in the `/mnt/test` directory, and any of its subdirectories, are stored on the `/dev/sdb1` device.
11. When you are finished using the drive, you can umount it with the `umount` command, after which you can safely remove the drive.
  - `umount /dev/sdb1`

Overwrite the USB drive with the `dd` command (`dd if/dev/zero of=/dev/sd<number> bs=1M count=100`).

### Using Logical Volume Manager Partitions
With LVM, physical disk partitions are added to pools of space called volume groups. Logical volumes are assigned space from volume groups as needed. This gives you these abilities:
- Add more space to a logical volume from the volume group while the volume is still in use.
- Add more physical volumes to a volume group if the volume group begins to run out of space. The physical volumes can be from disks.
- Move data from one physical volume to another so you can remove smaller disks and replace them with larger ones while the filesystems are still in use -- again, without downtime.

With LVM, it is also easier to shrink filesystems to reclaim disk space, although shrinking does require that you unmount the logical volume (but no reboot is needed).

**Checking an existing LVM**
- `fdisk -l /dev/sda | grep /dev/sda`
- Use the `pvdisplay` command to see if that partition is being used in an LVM group:
  - `pvdisplay /dev/sda2`
- The smallest unit of storage that can be used from this physical volume is referred to as a *Phyiscal Extent (PE)*.
- To see information about a volume group:
  - `vgdisplay <vg_name>`
- Using `lvdisplay` as follows, you can see where PEs have been allocated:
  - `lvdisplay <vg_name>`

If you run out of space on any of the logical volumes, you can assign more space from the volume group. If the volume group is out of space, you can add another hard drive or network storage drive and add space from that drive to the volume group so more is available.

**Creating LVM logical volumes**

LVM logical volumes are used from the top down, but they are created from the bottom up. First you create one or more physical volumes (pv), use the physical volumes to create volume groups (vg), and then create logical volumes from the volume groups (lv).

Commands for working with each LVM component begin with the letters `pv`, `vg`, and `lv`. For example, `pvdisplay` shows physical volumes, `vgdisplay` shows volume groups, and `lvdisplay` shows logical volumes.

1. Use the `pvcreate` command to identify a partition as an LVM physical volume.
2. To add that physical volume to a new volume group, use the `vgcreate` command.
  - `vgcreate myvg0 /dev/sdc6`
3. To see the new volume group, type the following:
  - `vgdisplay myvg0`
4. Here's how to create a logical volume from some of the space in that volume group and then check that the device for that logical volume exists:
  - `lvcreate -n music -L 1G myvg0`
  - `ls /dev/mapper/myvg0`
5. `/dev/mapper/myvg0-music`; that device can now be used to put a filesystem on and mount, just as you did with regular partitions:
  - `mkfs -t ext4 /dev/mapper/myvg0-music`
  - `mkdir /mnt/mymusic`
  - `mount /dev/mapper/myvg0-music /mnt/mymusic`
  - `df -h /mnt/mymusic`
6. As with regular partitions, logical volumes can be mounted permanently by adding an entry to the `/etc/fstab` file, such as:
  - `/dev/mapper/myvg0-music /mnt/mymusic ext4 defaults 1 2`

**Growing LVM logical volumes**

If you run out of space on a logical volume, you can add space to it without even unmounting it. To do that, you must have space available in the volume group, grow the logical volume, and grow the filesystem to fill it.
1. Note how much space is currently on the logical volume, and then check that space is available in the logical volume's volume group:
  - `vgdisplay myvg0`
  - `df -h /mnt/mymusic`
2. Expand the logical volume using the `lvextend` command:
  - `lvextend -L +1G /dev/mapper/myvg0-music`
3. Resize the filesystem to fit the new logical volume size:
  - `resize2fs -p /dev/mapper/myvg0-music`
4. Check to see that the filesystem is now resized to include the additional disk space:
  - `df -h /mnt/mymusic`

### Mounting Filesystems

**Supported Filesystems**

To see filesystem types that are currently loaded in your kernel, type `cat /proc/filesystems`.
- **befs**: Filesystem used by the BeOS operating system.
- **btrfs**: A copy-on-write filesystem that implements advanced filesystem features.
- **cifs**: Common Internet Filesystem (CIFS), the virtual filesystem used to access servers that comply with the SNIA CIFS specification.
- **ext4**: Successor to the popular ext3 filesystem. It includes many improvements over ext3, such as support for volumes up to 1 exabyte and file sizes up to 16 terabytes.
- **ext3**: Ext filesystems are the most common in Linux systems. Includes journaling features that, compared to ext2, improve a filesystem's capability to recover from crashes.
- **ext2**: The default filesystem type for earlier Linux systems.
- **ext**: This is the first version of the ext filesystem.
- **iso9660**: Evolved from the High Sierra filesystem (the original standard for CD-ROMs).
- **kafs**: AFS client filesystem.
- **minix**: Minix filesystem type, used originally with the Minix version of UNIX.
- **msdos**: An MS-DOS filesystem**
- **vfat**: Microsoft extended FAT (VFAT) filesystem.
- **exfat**: Extended FAT (exFAT) file system that has been optimized for SD cards, USB drives and other flash memory.
- **umsdos**: An MS-DOS filesystem with extensions to allow features that are similar to UNIX (including long filenames).
- **proc**: Not a real filesystem, but rather a filesystem interface to the Linux kernel.
- **reiserfs**: ReiserFS journaled filesystem. ReiserFS was once a common default filesystem type for several Linux distributions.
- **swap**: Used for swap partitions. Swap areas are used to hold data temporarily when RAM is used up. Data is swapped to the swap area and then returned to RAM when it is needed again.
- **squashfs**: Compressed, read-only filesystem type.
- **nfs**: Network Filesystem (NFS) type of filesystem.
- **hpfs**: Filesystem is used to do read-only mounts of an OS/2 HPFS filesystem.
- **ncpfs**: A filesystem used with Novell NetWare.
- **ntfs**: Windows NT Filesystem.
- **ufs**: Filesystem popular on Sun Microsystem's operating systems.
- **jfs**: A 64-bit journaling filesystem by IBM that is relatively lightweight for the many features it has.
- **xfs**: A high-performance filesystem originally developed by Silicon Graphics that works extremely well with large files.
- **gfs2**: A shared disk filesystem that allows multiple machines to all use the same shared disk without going through a network filesystem layer such as CIFS, NFS, and so on.

**Enabling swap areas**

A **swap area** is an area of the disk that is made available to Linux if the system runs out of memory (RAM). If your RAM is full and you try to start another application without a swap area, that application will fail.
- To create a swap area from a partition or a file, use the `mkswap` command. To enable that swap area temporarily, you can use the `swapon` command.
  - `dd if=/dev/zero of=/var/opt/myswap bs=1M count=1024`
  - `mkswap /var/opt/myswap`
  - `swapon /var/opt/myswap`
  - `free -m`
- The `free` command shows the amount of swap before and after creating, making, and enabling the swap area with the `swapon` command.

**Disabling swap area**

If at any point you want to disable a swap area, you can do so using the `swapoff` command.
- `swapoof /var/opt/myswap`

**Using the fstab file to define mountable file systems**

The `/etc/fstab` file contains definintions for each partition, along with options describing how the partition is mounted.
- The `tmpfs,`, `devpts,`, `sysfs`, and `proc` entries are special devices associated with shared memory, terminal windows, device information, and kernel parameters, respectively.
- In general, the first column of `/etc/fstab` shows the device ore share (what is mounted), while the second column shows the mount point (where it is mounted). That is followed by the type of filesystem, any mount options (or defaults), and two numbers (used to tell commands such as `dump` and `fsck` what to do with the filesystem).
- Sometimes device names for `/`, `/home`, and `swap` as well as other drives can start with `/dev/mapper`. That's because they are LVM logical volumes that are assigned space from a pool of space called an LVM volume group.
- Instead of the device name (`/dev/sda1` for example), a unique identifier (UUID) identifies each device.
- To see all of the UUIDs assigned to storage devices on a system, type the `blkid` command.

Here is a description of each field of the `/etc/fstab` file:
- **Field 1**: Name of the device representing the filesystem. This field can include the `LABEL` or `UUID` option with which you can indicate a volume label or universally unique identifier (UUID) instead of a device name.
- **Field 2**: Mount point in the filesystem. The filesystem contains all data from the mount point down the directory tree structure unless another filesystem is mounted at some point beneth it.
- **Field 3**: Filesystem type.
- **Field 4**: Use `defaults` or a comma-separated list of options (no spaces) that you want to use when the entry is mounted. See the `mount` command manual page (under the `-o` option), for information on other supported options.
- **Field 5**: The number in this field indicates whether the filesystem needs to be dumped (that is, have its data backed up). A `1` means that the filesystem needs to be dumped, and a `0` means that it doesn't.
- **Field 6**: The number in this field indicates whether the indicated filesystem should be check with `fsck` when the time comes for it to be checked: `1` means it needs to be checked first, `2` means to check after all those indicated by `1` have already been checked, and `0` means don't check it.

**Using the mount command to mount file systems**

Linux systems automatically run `mount -a` (mount all filesystems from `/etc/fstab` file) each time you boot.

Any user can type `mount` (with no options) to see what filesystems are currently mounted on the local Linux system.

To mount a read-only disk partition `sdb1` that has an older `ext3` filesystem, you could type this:
- `mkdir /mnt/temp`
- `mount -t ext3 -o ro /dev/sdb1 /mnt/tmp`

Another reason to use the `mount` command is to remount a partition to change its mount options. Suppose that you want to remount `/dev/sdb1` as read/write, but you do not want to unmount it (maybe someone is using it). You could use the remount option as follows:
- `mount -t ext3 -o remount,rw /dev/sdb1`

**Mounting a disk image in loopback**

With the image on your hard disk, create a mount point and use the `-o loop` option to mount it locally.
- `mkdir /mnt/mydvdimage`
- `mount -o loop whatever-i686-disc1.iso /mnt/mydvdimage`

**Using the umount command**

When you are finished using a temporary filesystem, or you want to unmount a permanent filesystem temporarily, use the `umount` command. This command detaches the filesystem from its mount point in your Linux filesystem. To use `umount`, you can give it either a directory name or a device name.
- `umount /mnt/test`
- `umount /dev/sdb1`

An alternative for unmounting a busy device is the `-l` option. With `umount -l` (a lazy unmount), the unmount happens as soon as the device is no longer busy. To unmount a remote NFS filesystem that's no longer available (for example, the server went down), you can use the `umount -f` option to forcibly unmount the NFS filesystem.
- A really useful tool for discovering what's holding open a device you want to unmount is the `lsof` command. Type `lsof` with the name of the partition that you want to unmount (such as `lsof /mnt/test`). The output shows what commands are holding files open on that partition. The `fuser -v /mnt/test` command can be used in the same way.

### Using the mkfs Command to Create a Filesystem
You can create a filesystem for any supported filesystem type on a disk or partition that you choose. You do so with the `mkfs` command.

Before you create a new filesystem, make sure of the following:
- You have partitioned the disk as you want (using the `fdisk` command).
- You get the device name correct, or you may end up overwriting your hard disk by mistake.
- To unmount the partition if it's mounted before creating the filesystem.

- `mkfs -t xfs /dev/sdc1`
- `mkfs -t ext4 /dev/sdc3`

You can now mount either of these filesystems.
- `mkdir /mnt/myusb ; mount /dev/sdc1 /mnt/myusb`
