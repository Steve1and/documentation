# Linux Security Concepts
***

## Basic Linux Security
At its most basic level, securing a Linux system starts with physical security, data security, user accounts protection, and software security.

### Implementing Physical Security
Basic server room physical security includes:
- A lock or security alarm on the server room door.
- Access controls that allow only authorized access and that identify who accessed the room and when the access occurred, such as a card key entry system.
- A sign stating "no unauthorized access allowed" on the door.
- Policies on who can access the room and when that access may occur for groups such as the cleaning crew, server administrators, and others.

**Implementing disaster recovery**

Disaster recovery plans should include these things:
- What data is to be included in backups
- Where backups are to be stored
- How long backups are maintained
- How backup media is rotated through storage

Backup utilities on a Linux system include the following:
- `amanda`
- `cpio`
- `dump/restore`
- `tar`
- `rsync`

With `rsync`, you can set up a `cron` job to keep copies of all data in selected directories or mirror exact copies of directories on remote machines.

### Securing User Accounts
Some rules are necessary to increase security through user account management:
- One user per user account.
- Limit access to the root user account.
- Set expiration dates on temporary accounts.
- Remove unused user accounts.

All `sudo` use (who, what when) is recorded in `/var/log/secure`, including any failed `sudo` access attempts. Recent Linux systems store all `sudo` access in the systemd journal (type `journalctl -f` to watch live `sudo` access attempts, along with other system messages).

To set a user account with an expiration date, use the `usermod` command. The format is `usermod -e yyyy-mm-dd user_name`.

The `chage` command is primarily used to view and change a user account's password aging information.
- The `-l` option allows you to list various information to which `chage` has access.
- Pipe the output from the `chage` command into `grep` and search for the word *Account*. This produces only the user account's expiration date.
  - `chage -l tim | grep Account`
- If you do not use the `/etc/shadow` file for storing your account passwords, the `chage` utility doesn't work.

**Removing unused user accounts**

After a user has left an organization, it is best to perform a series of steps to remove their account along with data:
1. Find files on the system owned by the account, using the `find / -user username` command.
2. Expire or disable the account.
3. Back up the files.
4. Remove the files or reassign them to a new owner.
5. Delete the account from the system.

You can use a two-step process to find expired accounts in the `/etc/shadow` file automatically. First, set up a shell variable with today's date in "days since January 1, 1970" format. Then, using the `gawk` command, you can obtain and format the information needed from the `/etc/shadow` file.
- `TODAY=$(echo $(($(date --utc --date "$1" +%s)/86400)))`
- `echo $TODAY`
- `gawk -F: '{print $1,$8}' /etc/shadow`
- `gawk -F: '{if (($8 > 0) && ($TODAY > $8)) print $1}' /etc/shadow`

**Setting and changing passwords**

You set your own password using the `passwd` command.
- First, it prompts you to enter your old password.
- Assuming that you type your old password correctly, the `passwd` command prompts you for the new password. When you type your new password, it is checked using a utility called `cracklib` to determine whether it is a good or bad password. Non-root users are required to try a different password if the one they have chosen is not a good password.
- The root user is the only user who is permitted to assign bad passwords. After the password has been accepted by `cracklib`, the `passwd` command asks you to enter the new password a second time to make sure that there are no typos.

When running as root, changing a user's password is possible by supplying that user's login name as a parameter of the `passwd` command.

If users are having a difficult time creating secure and unique passwords, consider installing the `pwgen` utility on your Linux system. This open source password generating utility creates passwords that are made to be pronounceable and memorable.

For accounts that have already been created, you need to control password aging via the `chage` command. The command has the following options:

| Option | Description |
| ------ | ----------- |
| `-M` | Sets the maximum number of days before a password needs to be changed. Equivalent to `PASS_MAX_DAYS` in `/etc/login.defs`. |
| `-m` | Sets the minimum number of days before a password can be changed again. Equivalent to `PASS_MIN_DAYS` in `/etc/login.defs`. |
| `-W` | Sets the number of days a user is warned before being forced to change the account password. Equivalent to `PASS_WARN_AGE` in `/etc/login.defs`. |

- `chage -M 30 -m 5 -W 7 tim`
- `chage -l tim | grep days`

**Understanding the password files and password hashes**

A *hashed password* is created using a one-way mathematical process. After you create the hash, you cannot re-create the original characters from the hash.
- When a user enters the account password, the Linux system rehashes the password and then compares the hash result to the original hash in `/etc/shadow`. If they match, the user is authenticated and allowed into the system.

Security experts will tell you that the passwords are not just hashed but also salted. *Salting a hash* means that a randomly generated value is added to the original password before it is hashed. This makes it even more difficult for the hashed password to be matched to its original password. However, in Linux, the hash salt is also stored with the hashed passwords.

You may inherit a Linux system that still uses the old method of keeping the hashed passwords in the `/etc/passwd` file. It is easy to fix. Just use the `pwconv` command, and the `/etc/shadow` file is created and hashed passwords moved to it.

The following are stored in the `/etc/shadow` file, in addition to the account name and hashed password:
- Number of days (since January 1, 1970) since the password was changed.
- Number of days before the password can be changed.
- Number of days to warn a user before a password must be changed.
- Number of days after a password expires that an account is disabled.
- Number of days (since January 1, 1970) that an account has been disabled.

### Securing the Filesystem
Files with the SUID permission in the `Owner` category and execute permission in the `Other` category allow anyone to become the file's owner temporarily while the file is being executed in memory.
- Similarly, files with the SGID permission in the `Group` category and execute permission in the `Other` category allow anyone temporarily to become a group member of the file's group while the file is being executed in memory. SGID can also be set on directories. This sets the group ID of any files created in the directory to the group ID of the directory.
- Commands such as `passwd` and `sudo` are designed to be used as SUID programs.
- Using the `find` command, you can search your system to see if there are any hidden or otherwise inappropriate SUID and SGID commands on your system.
  - `find / -perm /6000 -ls`

**Securing the password files**

The `/etc/passwd` file should have the following permission settings:
- Owner: root
- Group: root
- Permissions: (644) Owner: `rw-` Group: `r--` Other: `r--`

The `/etc/shadow` file should have the following permission settings:
- Owner: root
- Group: root
- Permissions: (000) Owner: `---` Group: `---` Other: `---`

The `passwd` utility, `/usr/bin/passwd`, uses the special permission SUID.
- Thus the user running the `passwd` command temporarily becomes root while the command is executing in memory and can then write to the `/etc/shadow` file, but only to change the user's own password-related information.

The `/etc/group` file contains all of the groups on the Linux system. Its file permissions should be set exactly as the `/etc/passwd` file:
- Owner: root
- Group: root
- Permissions: (644) Owner: `rw-` Group: `r--` Other: `r--`

Also, the group password file, `/etc/gshadow`, needs to be properly secured. As you would expect, the file permission should be set exactly as the `/etc/shadow` file:
- Owner: root
- Group: root
- Permissions: (000) Owner: `---` Group: `---` Other: `---`

**Locking down the filesystem**

The `/etc/fstab` file should have the following permission settings:
- Owner: root
- Group: root
- Permissions: (644) Owner: `rw-` Group: `r--` Other: `r--`

Typically, you put the `/home` subdirectory, where user directories are located, on its own partition.
- You can set the `nosuid` option to prevent SUID and SGID permission-enabled executable programs from running from there.
- You can set the `nodev` option so that no device file located there will be recognized. Device files should be stored in `/dev` and not in `/home`.
- You can set the `noexec` option so that no executable programs, which are stored in `/home`, can be run.

You can put the `/tmp` subdirectory, where temporary files are located, on its own partition and use the same options settings as for `/home`:
- `nosuid`
- `nodev`
- `noexec`

You can put the `/usr` subdirectory, where user programs and data are located, on its own partition and set the `nodev` option so that no device file located there is recognized.

If the system is configured as a server, you probably want to put the `/var` directory on its own partition. You can use the same `mount` options with the `/var` partition as you do for `/home`:
- `nosuid`
- `nodev`
- `noexec`

Putting the preceding `mount` options into your `/etc/fstab` would look similar to the following:
```
/dev/sdb1   /home   ext4    defaults,nodev,noexec,nosuid    1 2
/dev/sdc1   /tmp    ext4    defaults,nodev,noexec,nosuid    1 2
/dev/sdb2   /usr    ext4    defaults,nodev                  1 2
/dev/sdb3   /var    ext4    defaults,nodev,noexec,nosuid    1 2
```

**Keeping up with security advisories**

Software updates needs to be done on a regular basis.

As security flaws are found in Linux software, the Common Vulnerabilities and Exposures (CVE) project tracks them and helps to quickly get fixes for those flaws worked on by the Linux community.
- Companies such as Red Hat provide updated packages to fix the security flaws and deliver them in what is referred to as *errata*.

### Monitoring Your Systems

**Monitoring log files**

The log files for your Linux system are primarily located in the `/var/log` directory. Most of the files in the `/var/log` directory are directed there from the `systemd` journal through the `rsyslogd` service.

| System Log Name | Filename | Description |
| --------------- | -------- | ----------- |
| Apache Access Log | `/var/log/httpd/access_log` | Logs requests for information from your Apache web server. |
| Apache Error Log | `/var/log/httpd/error_log` | Logs errors encountered from clients trying to access data on your Apache web server. |
| Bad Logins Log | `btmp` | Logs bad login attempts. |
| Boot Log | `boot.log` | Contains messages indicating which system services have started and shut down successfully and which (if any) have failed to start or stop. The most recent bootup messages are listed near the end of the file. |
| Kernel Log | `dmesg` | Records messages printed by the kernel when the system boots. |
| Cron Log | `cron` | Contains status messages from the crond daemon. |
| dpkg Log | `dpkg.log` | Contains information concerning installed Debian packages. |
| FTP Log | `vsftpd.log` | Contains messages relating to transfers made using the vsftpd daemon (FTP server). |
| FTP Transfer Log | `xferlog` | Contains information about files transferred using the FTP service. |
| GNOME Display Manager Log | `/var/log/gdm/:0.log` | Holds messages related to the login screen (GNOME display manager). Yes, there really is a colon in the filename. |
| LastLog | `lastlog` | Records the last time an account logs in to the system. |
| Login/out Log | `wtmp` | Contains a history of logins and logouts on the system. |
| Mail Log | `maillog` | Contains information about addresses to which and from which email was sent. Useful for detecting spamming. |
| MySQL Server Log | `mysqld.log` | Includes information related to activities of the MySQL database server (mysqld`).
| News Log | `spooler` | Provides a directory containing logs of messages from the Usernet News server if you are running one. |
| Samba Log | `/var/log/samba/smbd.log` `/var/log/samba/nmbd.log` | Shows messages from the Samba SMB file service daemon. |
| Security Log | `secure` | Records the date, time, and duration of login attempts and sessions. |
| Sendmail Log | `sendmail` | Shows error messages recorded by the sendmail daemon. |
| Squid Log | `/var/log/squid/access.log` | Contains messages related to the squid proxy/caching server. |
| System Log | `messages` | Provides a general-purpose log file where many programs record messages. |
| UUCP Log | `uucp` | Shows status messages from the UNIX to UNIX Copy Protocol daemon. |
| YUM Log | `yum.log` | Shows messages related to RPM software packages. |
| X.ORG X11 Log | `Xorg.0.log` | Includes messages output by the X.Org X server. |

**Viewing Log Files that need special Commands**

| Filename | View Command |
| -------- | ------------ |
| `btmp` | `dump-utmp btmp` |
| `dmesg` | `dmesg` |
| `lastlog` | `lastlog` |
| `wtmp` | `dump-utmp wtmp` |

To page through kernel messages, type the following command:
- `journalctl -k`

To view messages associated with a particular service, use the `-u` option followed by the service name to see log messages for any service:
- `journalctl -u NetworkManager.service`
- `journalctl -u httpd.service`
- `journalctl -u avahi-daemon.service`

Display boot IDs and then dhow boot messages for a selected boot ID:
- `journalctl --list-boots`
- `journalctl -b c848e7442932488d91a3a467e8d92fcf`

### Monitoring User Accounts

**Detecting counterfeit new accounts and privileges**

To help you monitor the `/etc/passwd` and `/etc/group` files, you can use the audit daemon. The audit daemon is an extremely powerful auditing tool that allows you to select system events to track and record them, and it provides reporting capabilities. To begin auditing the `/etc/passwd` and `/etc/group` files, you need to use the `auditctl` command. Two options at a minimum are required to start this process:
- `w filename`: Place a watch on *filename*. The audit daemon tracks the file by its inode number. An *inode number* is a data structure that contains information concerning a file, including its location.
- `-p trigger(s-)`: If one of these access types occurs (r=read, w=write, x=execute, a=attribute change) to *filename*, then trigger an audit record.
  - `auditctl -w /etc/passwd -p rwa`
- To turn off an audit, use the command:
  - `auditctl -W filename -p trigger(s)`
- To see a list of current audited files and their watch settings, type `auditctl -l` at the command line.
- To review the audit logs, use the audit daemon's `ausearch` command. The only option needed here is the `-f` option, which specifies which records you want to view from the audit log.
  - `ausearch -f /etc/passwd`

A few items will help you see what audit event happened to trigger a record:
- **time**: The time stamp of the activity.
- **name**: The filename, `/etc/passwd`, being watched.
- **inode**: The `/etc/passwd's` inode number on this filesystem.
- **uid**: The user ID, 1000, of the user running the program.
- **exe**: The program, `/bin/vi`, used on the `/etc/passwd` file.

Audit daemon utilities and configuration files:
- **auditd**: The audit daemon
- **auditd.conf**: The audit daemon configuration file.
- **autditctl**: Controls the auditing system
- **audit.rule**: Configuration rules loaded at boot
- **ausearch**: Searches the audit logs for specified items.
- **aureport**: Report creator for the audit logs
- **audispd**: Sends audit information to other programs

### Monitoring the Filesystem

**Verifying software packages**

Double-check your installed software packages to see if they have been compromised. The command to accomplish this is `rpm -V package_name`.
- If no problems are found, the `rpm -V` command returns nothing. However, if there are discrepancies, you get a coded listing. Here are the codes that can be listed:

| Code | Discrepancy |
| ---- | ----------- |
| S | File size |
| M | File permissions and type |
| 5 | MD5 check sum |
| D | Device file's major and minor numbers |
| L | Symbolic links |
| U | User ownership |
| G | Group ownership |
| T | File modified times (mtime) |
| P | Other installed packages this package is dependent upon (aka capabilities) |

To verify packages on Ubuntu, you need the `debsums` utility. It is not installed by default. To install `debsums`, use the command `sudo apt-get install debsums`. To check all installed packages, use the `debsums -a` command. To check one package, type `debsums packagename`.

**Scanning the filesystem**

To check for binary file modification, `find` can use the file's modify time, or `mtime`. The file `mtime` is the time when the contents of a file were last modified. Also `find` can monitor the file's create/change time, or `ctime`.
- A scan is made of the `/sbin` directory. To see if any binary files were modified less than 24 hours ago, the command `find /sbin -mtime -1`.
- To review each individual file's times, using the `stat filename` command.
  - `find /sbin -mtime -1`
  - `stat /sbin/init`

Here are some additional filesystem scans:

| File or Setting | Scan Command | Problem with File or Setting |
| --------------- | ------------ | ---------------------------- |
| SUID permission | `find / -perm -4000` | Allows anyone to become the file's owner temporarily while the file is being executed in memory. |
| SGID permission | `find / -perm -2000` | Allows anyone to become a group member of the file's group temporarily while the file is being executed in memory. |
| rhost files | `find /home -name .rhosts` | Allows a system to trust another system completely. It should not be in /home directories. |
| Ownerless files | `find / -nouser` | Indicates files that are not associated with any username. |
| Groupless files | `find / -nogroup` | Indicates files that are not associated with any group name. |

**Monitoring for viruses**

A *computer virus* is malicious software that can attach itself to already installed system software, and it has the ability to spread through media or networks.

Antivirus software scans files using virus signatures. A *virus signature* is a hash created from a virus's binary code. The hash will positively identify that virus.
- A good antivirus software choice for your Linux system, which is open source and free, is ClamAV. To install ClamAV on a Fedora or RHEL system, type the command `dnf install clamav`.
- You can review the packages available for Ubuntu installation by entering the command `apt-cache search clamav`.

**Monitoring for rootkits**

A rootkit is a malicious program that does the following:
- Hides itself, often by replacing system commands or programs.
- Maintains high-level access to a system.
- Is able to circumvent software created to locate it.

The purpose of a rootkit is to get and maintain root-level access to a system. The term was created by putting together *root*, which means that it has to have administrator access, and *kit*, which means it is usually several programs that operate in concert.

A rootkit detector that can be used on a Linux system is `chkrootkit`. To install `chkrootkit` on a Fedora or RHEL system, issue the command `yum install chkrootkit`. To install `chkrootkit` on an Ubuntu system, use the command `sudo apt-get install chkrootkit`.
- `chkrootkit | grep INFECTED`

A *false positive* is an indication of a virus, rootkit, or other malicious activity that does not really exist.

Another rootkit detector that might interest you is called Rootkit Hunter (`rkhunter`). Run the `rkhunter` script to check your system for malware and known rootkits.

**Detecting an Intrusion**

*Intrusion Detection System (IDS)* software: a software package that monitors a system's activities (or its network) for potential malicious activities and reports these activities.

Popular Linux Intrusion Detection Systems:

| IDS Name | Installation | Website |
| -------- | ------------ | ------- |
| aide | `yum install aide` `apt-get install aide` | http://aide.sourceforge.net |
| Snort | rpm or tarball packages from website | http://snort.org |
| tripwire | `yum install tripwire` `apt-get install tripwire` | http://tripwire.org |

The Advanced Intrusion Detection Environment (aide) IDS uses a method of comparison to detect intrusions.
- The command to create the initial database is `aide -i` and it takes a long time to run.
- The next step is to move the initial "first picture" database to a new location. This protects the original database from being overwritten. Plus, the comparison does not work unless the database is moved.
  - `cp /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz`
- The check option on the `aide` command, `-c`, creates a new database and runs a comparison against the old database.
  - `aide -C`

### Auditing and Reviewing Linux
A *compliance review* is an audit of the overall computer system environment to ensure that the policies and procedures you have set for the system are being carried out correctly. A *security review* is an audit of current policies and procedures to ensure that they follow accepted best security practices.

*Penetration testing* is an evaluation method used to test a computer system's security by simulating malicious attacks.
***

## Advanced Linux Security

### Implementing Linux Security with Cryptography
*Cryptography* is the science of concealing information. Cryptography terms:
- **Plain text**: Text that a human or machine can read and comprehend.
- **Ciphertext**: Text that a human or machine cannot read and comprehend.
- **Encryption**: The process of converting plain text into ciphertext using an algorithm.
- **Decryption**: The process of converting cipher text into plain text using an algorithm.
- **Cipher**: The algorithm used to encrypt plain text into ciphertext and decrypt ciphertext into plain text.
- **Block cipher**: A cipher that breaks data into blocks before encrypting.
- **Stream cipher**: A cipher that encrypts the data without breaking it up.
- **Key**: A piece of data required by the cipher to encrypt or decrypt data successfully.

**Understanding Hashing**

Hashing is not encryption, but it is a form of cryptography. *Hashing* is a one-way mathematical process used to create ciphertext. However, unlike encryption, after you create a hash, you cannot de-hash it back to its original plain text.

In order for a hashing algorithm to be used in computer security, it needs to be *collision-free*, which means that the hashing algorithm does not output the same hash for two totally different inputs. Each input must have a unique hashed output. Thus, *cryptographic hashing* is a one-way mathematical process that is collision-free.

Hashing is used on Linux systems for the following:
- Passwords
- Verifying files
- Digital signatures
- Virus signatures
A hash is also called a *message digest*, *checksum*, *fingerprint*, or *signature*.

**Understanding Encryption/Decryption**

Encryption/decryption processes use special math algorithms to accomplish their task. The algorithms are called *cryptographic ciphers*.

| Method | Description |
| ------ | ----------- |
| AES (Advanced Encryption Standard), also called Rijndael | *Symmetric cryptography*. Block cipher, encrypting data in 128-, 192-, 256-, 512-bit blocks using a 128-, 192-, 256-, or 512-bit key for encrypting/decrypting. |
| Blowfish | *Symmetric cryptography*. Block cipher, encrypting data in 64-bit blocks using the same 32-bit to 448-bit keys for encrypting/decrypting. |
| CASTS | *Symmetric cryptography*. Block cipher, encrypting data in 64-bit blocks using the same up to 128-bit key for encrypting/decrypting. |
| DES (Data Encryption Standard) | No longer considered secure. *Symmetric cryptography*. Block cipher, encrypting data in 64-bit blocks using the same 56-bit key for encrypting/decrypting. |
| 3DES | Improved DES cipher. *Symmetric cryptography*. Data is encrypted up to 48 times with three different 56-bit keys before the encryption process is completed. |
| El Gamal | *Asymmetric cryptography*. Uses two keys derived from a logarithm algorithm. |
| Elliptic Curve Cryptosystems | *Asymmetric cryptography*. Uses two keys derived from an algorithm containing two randomly chosen points on an elliptic curve. |
| IDEA | *Symmetric cryptography*. Block cipher, encrypting data in 64-bit blocks using the same 128-bit key for encrypting/decrypting. |
| RC4 also called ArcFour or ARC4 | Stream cipher, encrypting data in 64-bit blocks using a variable key size for encrypting/decrypting. |
| RC5 | *Symmetric cryptography*. Block cipher, encrypting data in 32-, 64-, or 128-bit blocks using the same up to 2,048-bit keys for encrypting/decrypting. |
| RC6 | *Symmetric cryptography*. Same as RC5, but slightly faster. |
| RSA | Most popular *asymmetric cryptography*. Uses two keys derived from an algorithm containing a multiple of two randomly generated prime numbers. |

**Understanding Cryptographic Cipher Keys**

Cryptographic ciphers require a piece of data, called a *key*, to complete their mathematical process of encryption/decryption. The key can be either a single key or a pair of keys. The key size is directly related to how easily the cipher is cracked.
- **Symmetric Key Cryptography**: *Symmetric cryptography*, also called *secret key* or *private key* cryptography, encrypts plain text using a single keyed cipher. The same key is needed in order to decrypt the data. The advantage of symmetric key cryptography is speed. The disadvantage is the need to share the single key if the encrypted data is to be decrypted by another person.
  - An example of symmetric key cryptography on a Linux system is accomplished using the OpenPGP utility. GNU Prvacy Guard, `gpg2`.
  - With the `-c` option, `gpg2` encrypts the file with a symmetric key. The original file is kept and a new encrypted file, `backup.tar.gz.gpg`, is created.
    - `tar -cvzf /tmp/backup.tar.gz /etc`
    - `gpg2 -c --force-mdc -o /tmp/backup.tar.gz.gpg /tmp/backup.tar.gz`
    - The single key used to encrypt the file is protected by a passphrase.
    - To decrypt the file, use the `gpg2` utility again.
    - `gpg2` with the `-d` option and provide the passphrase for the secret key.
    - `gpg2 -d --force-mdc /tmp/backup.tar.gz.gpg > /tmp/backup.tar.gz`
    - If the `gpg-agent` daemon is running on the system, that passphrase is cached so that file could be decrypted again without entering the passphrase again.
- **Asymmetric Key Cryptography**: also called private/public key cryptography, uses two keys, called a *key pair*. A key pair consists of a public key and a private key.
  - A plain-text file is encrypted using a public key of a key pair. The encrypted file then can be securely transmitted to another person. To decrypt the file, the private key is used. This private key must be from the public/private key pair. Thus, data that has been encrypted with the public key can only be decrypted with its private key. The advantage of asymmetric cryptography is heightened security. The disadvantage is speed and key management.
  - You can perform asymmetric encryption on your Linux system using `gpg2`.
  - Before you can encrypt a file, you must first create your key pair and a "key ring".
  - `gpg2 --gen-key`
    - The `gpg2` utility asks for several specifications to generate the desired public/private keys:
      - **User ID**: This identifies the public key portion of the public/private key pair.
      - **Email Address**: This is the email address associated with the key.
      - **Passphrase**: This is used to identify and protect the private key portion of the public/private key pair.
    - Check the key ring by using the `gpg2 --list-keys` command.
  - After the key pair and key ring are generated, files can be encrypted and decrypted. First, the public key must be extracted from the key ring so that it can be shared.
    - The extracted key is put into a file to be shared.
    - `gpg2 --export John Doe > JohnDoe.pub`
    - `ls *.pub`
    - `file JohnDoe.pub`
  - Another user can add the public key to their key ring using the `gpg2 --import` command.
    - `gpg2 --import JohnDoe.pub`
    - `gpg2 --list-keys`
  - After the key is added to the key ring, that public key can be used to encrypt data for the public key's original owner.
    - The user that obtained the public key encrypts it using the public key.
    - The encrypted file is created by the `--out` option.
    - The option `--recipient` identifies the user who created the public key using only the real name portion of their public key's UID in quotation marks.
    - `gpg2 --out MessageForJohn --recipient "John Doe" --encrypt MessageForJohn.txt`
    - The encrypted message file created from the plain-text file can be securely sent to the user who owns the private key. This private key is used to decrypt it.
    - `gpg2 --out JillsMessage --decrypt MessageForJohn`
  - To review, the steps needed for encryption/decryption of files using asymmetric keys include the following:
    1. Generate the key pair and the key ring.
    2. Export a copy of your public key to a file.
    3. Share the public key file.
    4. Individuals who want to send you encrypted files add your public key to their key ring.
    5. A file is encrypted using your public key.
    6. The encrypted file is sent to you.
    7. You decrypt the file using your private key.

**Understanding Digital Signatures**

A *digital signature* is an electronic originator used for authentication and data verification. It is a cryptographic token sent with a file, so the file's receiver can be assured that the file came from you and has not been modified in any way. When you create a digital signature, the following steps occur:
1. You create a file or message.
2. Using the `gpg2` utility, you create a hash or message-digest of the file.
3. The `gpg2` utility then encrypts the hash and the file, using an asymmetric key cipher. For the encryption, the private key of the public/private key pair is used. This is now a digitally signed encrypted file.
4. You send the encrypted hash (aka digital signature) and file to the receiver.
5. The receiver re-creates the hash or message digest of the received encrypted file.
6. Using the `gpg2` utility, the receiver decrypts the received digital signature using the public key, to obtain the original hash or message digest.
7. The `gpg2` utility compares the original hash to the re-created hash to see if they match. If they match, the receiver is told the digital signature is good.
8. The receiver can now read the decrypted file.

Original signatures have their own special ciphers. While several ciphers can handle both encryption and creating signatures, there are a few whose only job is to create digital signatures. Previously, the most popular cryptographic ciphers to use in creating signatures were RSA and Digital Signature Algorithm (DSA). The RSA algorithm can be used for both encryption and creating signatures, while DSA can be used only for creating digital signatures. Today, Ed25519 is considered to be more secure and faster than RSA, and ECDSA provides better protection than DSA.

The `--sign` option tells the `gpg2` utility that a file should be encrypted and used to create a digital signature. In response, the `gpg2` utility does the following:
- Creates a message digest (aka hash) of the message file.
- Encrypts the message digest, which creates the digital signature.
- Encrypts the message file.
- Places the encrypted contents into the file specified by the `--output` option, `JohnDoe.DS`.
- `gpg2 --output JohnDoe.DS --sign MessageForJill.txt`
  - After the user jill receives the signed and encrypted file, she can use the `gpg2` utility to check the digital signature and decrypt the file in one step. The `--decrypt` option is used along with the name of the digitally signed file.
    - `gpg2 --decrypt JohnDoe.DS`

The previous example of digitally signing a document allows anyone with the public key the ability to decrypt the document. In order to keep it truly private, use the public key of the recipient to encrypt with the `gpg2` options: `--sign` and `--encrypt`. The recipient can decrypt with their private key.

### Implementing Linux Cryptography
Message Digest Utilities:
- `sha224sum`
- `sha256sum`
- `sha384sum`
- `sha512sum`
These tools work just like the `sha1sum` command, except, they use the SHA-2 cryptographic hash standard. The only difference between the various SHA-2 tools is the key length they use.
- The SHA-2 cryptographic hash standard was created by the National Security Agency (NSA). SHA-3 is another cryptographic hash standard, which was released by NIST in August 2015.

**Encrypting a Linux Filesystem at Installation**

Linux Unified Key Setup (LUKS): https://gitlab.com/cryptsetup/cryptsetup

If you inherit a system with an encrypted disk, using root privileges, you can use the `lvs` and `cryptsetup` commands and the `/etc/cryptab` file to help.
- `lvs -o devices`: This command shows all of the logical volumes currently on the system and their underlying device names. If the device name starts with `luks`, it indicates that the LUKS standard for hard disk encryption has been used.
- Ubuntu does not have the `lvs` command installed by default. To install it, type `sudo apt-get install lvm2` at the command line.

The encrypted logical volumes are mounted at boot time using the information from the `/etc/fstab` file. However, contents of the `/etc/crypttab` file, which are used to trigger the capture of the password at boot time, will decrypt the `/etc/fstab` entries as they are mounted.
```
cat /etc/crypttab
luks-b099fbbe-0e56-425f-91a6-44f129db9f4b
cryptsetup status luks-b099fbbe-0e56-425f-91a6-44f129db9f4b
```

**Encrypting a Linux Directory**

You can also use the `encryptfs` utility to encrypt on a Linux system. The `encryptfs` utility is not a filesystem type, as the name would imply. Instead, it is a POSIX-compliant utility that allows you to create an encryption layer on top of any filesystem.
- First, there should be no files currently residing in the directory before it is encrypted. If you do not move them, you cannot access them while the directory is encrypted.
- To encrypt the directory `/home/johndoe/Secret`, use the `mount` command. You must have root privileges to mount and umount the encrypted directory in this method.
  - The item to mount and its mount point are the same directory. You are literally encrypting the directory and mounting it upon itself.
  - `mount -t encryptfs /home/johndoe/Secret /home/johndoe/Secret`

The `encryptfs` utility allows you to choose the following:
- Key type
- Passphrase
- Cipher
- Key size (in bytes)
- To enable or disable plain text to pass through
- To enable or disable filename encryption

The utility allows you to apply a digital signature to the mounted directory so that if you `mount` it again, it just mounts the directory and does not require a passphrase.
- Write down the selections you make when you mount an `encryptfs` folder for the first time. You need to exact selections you chose the next time you remount the folder.

Next, the file `my_secret_file` is copied to the encrypted directory. User `johndoe` can still use the `cat` command to display the file in plain text. The file is automatically decrypted by the `encryptfs` layer.
- `cp my_secret_file Secret`
- The `root` user can also read the file.

However, after the encrypted directory is unmounted using the `umount` command, the files are no longer automatically decrypted.
- `umount /home/johndoe/Secret`

As a non-root user, you could use the `encryptfs-setup-private` and `encryptfs-mount-private` commands to configure a private cryptographic mountpoint as a non-root user.

**Encrypting a Linux File**

Popular Linux Cryptography Tools:
- `aescrypt`: It uses the symmetric key cipher Rijndeal, also called AES. This third-party FOSS tool is available for download from www.aescriypt.com
- `bcrypt`: This tool uses the symmetric key cipher *blowfish*. It is not installed by default.
- `ccrypt`: This tool uses the symmetric key cipher Rijndael, also called AES. It was created to replace the standard Unix `crypt` utility and is not installed by default.
- `gpg`: This utility can use either asymmetric key pairs or a symmetric key. The default cipher to use is set in the `gpg.conf` file.

| Tool | Description |
| ---- | ----------- |
| Duplicity | Encrypts backups. |
| gpg-zip | Uses GNU Privacy Guard to encrypt or sign files into an archive. Installed by default. |
| Openssl | A toolkit that implements Secure Socket Layer (SSL) and Transport Layer Security (TLS) protocols. These protocols require encryption. Installed by default. |
| Seahorse | A GNU Privacy Guard encryption key manager. Installed by default on Ubuntu. |
| ssh | Encrypts remote access across a network. Installed by default. |
| Zipcloak | Encrypts entries in a Zip file. Installed by default. |

If you want to see a full list of installed cryptography tools on your current Linux distribution, type `man -k crypt` at the command line.

### Implementing Linux Security with PAM
*Pluggable Authentication Modules (PAM)* was invented by Sun Microsystems and originally implemented in the Solaris operating system.
- PAM is a centralized method of providing authentication for the Linux system and applications.

Applications can be written to use PAM; such applications are called "PAM-aware". A PAM-aware application does not have to be rewritten and recompiled to have its authentication settings changed. Any required changes are made within a PAM configuration file for the PAM-aware applications.
- You can see whether a particular Linux application or utility is PAM-aware. Check whether it is compiled with the PAM library, `libpam.so`.
- The `ldd` command checks a file's shared library dependencies. To keep it simple, `grep` is used to search for the PAM library. As you can see, `crontab` on this particular Linux system is PAM-aware.
  - `ldd /usr/bin/crontab | grep pam`
- The benefits of using PAM on your Linux system include the following:
  - Simplified and centralized authentication management from the administrator viewpoint.
  - Simplified application development, because developers can write applications using the documented PAM library instead of writing their own authentication routines.
  - Flexibility in authentication:
    - Allow or deny access to resources based on traditional criteria, such as identification.
    - Allow or deny access based on additional criteria, such as time-of-day restrictions.
    - Set subject limitations, such as resource usage.

**Understanding the PAM authentication process**

When a subject (user or process) requests to a PAM-aware application or utility, two primary components are used to complete the subject authentication process:
- The PAM-aware application's configuration file.
- The PAM modules the configuration file uses.

Each PAM-aware application configuration file is at the center of the process. The PAM configuration files call upon particular PAM modules to perform the needed authentication. PAM modules authenticate subjects from system authorization data, such as a centralized user account using LDAP.

A series of steps is taken by PAM using the modules and configuration files in order to ensure that proper application authentication occurs:
1. A subject (user or process) requests access to an application.
2. The application's PAM configuration file, which contains an access policy, is open and read.
  - The access policy is set via a list of all the PAM modules to be used in the authentication process. This PAM module(s) list is called a *stack*.
3. The PAM modules in the stack are invoked in the order in which they are listed.
4. Each PAM module returns either a success or failure status.
5. The stack continues to be read in order, and it is not necessarily stopped by a single returned failure status.
6. The status results of all of the PAM modules are combined into a single overall result of authentication success or failure.

Most PAM configuration files are located in `/etc/pam.d`. The general format of a PAM configuration file is:
- `context    control flag    PAM module [module options]`

**Understanding PAM contexts**

PAM modules have standard functions that provide different authentication services. These standard functions within a PAM module can be divided into function types called *contexts*. Contexts can also be called *module interfaces* or *types*.

| Context | Service Description |
| ------- | ------------------- |
| `auth` | Provides authentication management services, such as verifying account passwords. |
| `account` | Provides account validation services, such as time-of-day access restrictions. |
| `password` | Manages account passwords, such as password length restrictions. |

**Understanding PAM control flags**

In a PAM configuration file, control flags are used to determine the overall status, which are returned to the application. A control flag is either of the following:
- **Simple Keyword**: The only concern here is if the corresponding PAM module returns a response of either "failed" or "success".
- **Series of actions**: The returned module status is handled through the series of actions listed in the file.

**PAM Configuration Control Flags and Response Handling**

| Flag | Response Handling |
| ---- | ----------------- |
| `required` | If failed, returns a failure status to the application, after the rest of the contexts have been run in the stack. |
| `requisite` | If failed, returns a failure status to the application immediately without running the rest of the stack. (Be careful where you place this control in the stack.) |
| `sufficient` | If failed, the module status is ignored. If successful, then a success status is immediately returned to the application without running the rest of the stack. |
| `optional` | This control flag is important only for the final overall return status of success or failure. Think of it as a tiebreaker. When the other modules in the configuration file stack return statuses that are neither clear-cut failure nor success statuses, this optional module's status is used to determine the final status or break the tie. In cases where the other modules in the stack are returning a clear-cut path of failure or success, this status is ignored. |
| `include` | Get all the return statuses from this particular PAM configuration file's stack to include this stack's overall return status. It's as if the entire stack from the named configuration file is now in this configuration file. |
| `substack` | Similar to the include control flag, except for how certain errors and evaluations affect the main stack. This forces the included configuration file stack to act as a substack to the main stack. Thus, certain errors and evaluations affect only the substack and not the main stack. |

**Understanding PAM modules**

A PAM module is actually a suite of shared library modules (DLL files) stored in `/usr/lib64/security` (64-bit). You can see a list of the various installed PAM modules on your system by entering `ls /usr/lib64/security/pam*.so` on the command line.
- On Ubuntu, to find your PAM modules, type the command `sudo find / -name pam*.so` at the command line. (`/usr/lib/x86_64-linux-gnu/security/`)

PAM resources:
- http://www.openwall.com/pam/
- http://puszcza.gnu.org.ua/software/pam-modules/download.html
- Understanding PAM system event configuration files.

Other system events, such as logging into the Linux system, also use PAM. Thus, these events also have configuration files.
- `ls -l /etc/pam.d`
- You can modify these system event configuration files to implement your organization's specific security needs.

Each PAM-aware application should have its very own PAM configuration file. Each configuration file defines what particular PAM modules are used for that application.
- PAM comes with the "other" configuration file. If a PAM-aware application does not have a PAM configuration file, it defaults to using the "other" PAM configuration file.

The PAM `/etc/pam.d/other` configuration file should deny all access, which in terms of security is referred to as Implicit Deny. In computer security access control, *Implicit Deny* means that if certain criteria are not clearly met, access must be denied.

The problem with making changes to some of these PAM system event configuration files is that the utility `authselect` can rewrite these files and remove any locally made changes. Fortunately, each PAM-configuration file that runs this risk has it documented in a comment line within. Using `grep`, you can quickly find which PAM configuration files have this potential problem.
  - `grep "authselect" /etc/pam.d/*`

Problems such as fork bombs can be averted by limiting the number of processes a single user can create. A *fork bomb* occurs when a process spawns one process after another in a recursive manner until system resources are consumed.
- The PAM module `pam-limits` uses a special configuration file to set these resource limits: `/etc/security/limits.conf`. By default, this file has no resource limits set within it.
- PAM configuration files are in the `/etc/pam.d` and `/etc/security` directories.
- `cat /etc/security/limits.conf`. In this file there are items called `domain` and `type`:
  - **domain**: The limit applies to the listed user or group. If the domain is *, it iapplies to all users.
  - **type**: A `hard` limit cannot be exceeded. A `soft` limit can be exceeded, but only temporarily.
  - The `nproc` limit sets the maximum number of processes a user can start.

To prevent others from using an account beyond the primary user, set their account's `maxlogins` to 1 in a *.conf file in the `/etc/security/limits.d` directory.
- `johndoe    hard    maxlogins     1`
- To override any settings in the `limits.conf` file, add files named *.conf to the `/etc/security/limits.d` directory.
- The final step in limiting this resource is to ensure that the PAM module using `limits.conf` is included in one of the PAM system event configuration files. The PAM module using `limits.conf` is `pam_limits`.
  - `grep "pam_limits" /etc/pam.d/*`

**Implementing time restrictions with PAM**

The PAM configuration file that handles these restrictions is located in the `/etc/security/` directory, `/etc/security/time.conf`.
- To allow a regular user to only log in on terminals on weekdays from 0700 to 1900, create the following settings:
  - **services**: Login
  - **ttys-**: *, indicates that all terminals are to be included.
  - **users**: Everyone but root (`!root`).
  - **times**: Allowed on weekdays (`Wd`) from 0700 (`0700`) to 1900 (`1900`).
- The entry in `time.conf` would look like:
  - `login; * ; !root ; Wd0700-1900`
- The final step in implementing this example time restriction is to ensure that the PAM module using `time.conf` is included in one of the PAM system event configuration files. The PAM module using `time.conf` is `pam_time`.
  - `grep "pam_time" /etc/pam.d/*`
- If `pam_time` is not listed, you must modify the `/etc/pam.d/system-auth` file in order for PAM to enforce the time restrictions. The PAM configuration file `system-auth` is used by PAM at system login and during password modifications. This configuration file checks many items, such as time restrictions.
  - Add the following near the top of the "account" section of the configuration file.
    - `account    required    pam_time.so`
  - On Ubuntu, you need to modify the `/etc/pam.d/common-auth` file instead of the `system-auth` configuration file.

**Enforcing good passwords with PAM**

When a password is modified, the PAM module `pam_cracklib` is involved in the process. The module prompts the user for a password and checks its strength against a system dictionary and a set of rules for identifying poor choices.
- The `pam_cracklib` module is installed by default on Fedora and RHEL. For Ubuntu Linux systems, it is not installed by default. Run the `sudo apt-get install libpam-cracklib`.
- Using `pam_cracklib`, you can check a newly chosen password for the following:
  - Is it a dictionary word?
  - Is it a palindrome?
  - Is it the old password with the case changed?
  - Is it too much like the old password?
  - Is it too short?
  - Is it a rotated version of the old password?
  - Does it use the same consecutive characters?
  - Does it contain the username in some form?
- You can change the rules `pam_cracklib` uses for checking new passwords by making modifications to the `/etc/pam.d/system-auth` file, (or the `/etc/pam.d/common-password` file on Ubuntu systems).
  - `cat /etc/pam.d/passwd`

The following are keywords available for `pam_cracklib`:
- **debug**: causes module to write information to syslog.
  - `authtok_type=XXX`
  - Defaults to using `New UNIX password: ` and `Retype UNIX password: ` to request passwords.
  - Replace XXX with a word to use instead of UNIX.
- **retry=N**: Default = 1
  - Prompt the user at most N times before returning with an error.
- **difork=N**: Default = 5
  - The number of characters in the new password that must not be present in the old password.
  - *Exception 1:* If half of the characters in the new password are different, then the new password is accepted.
  - *Exception 2*: See `difignore`
- **difignore=N**: Default = 23
  - The number of characters that the password has before the `difork` setting is ignored.
- **minlen=N**: Default = 9
  - The minimum acceptable size for the new password.
  - See `dcredit`, `ucredit`, `lcredit`, and `ocredit` for how their settings affect `minlen`.
- **dcredit=N**: Default = 1
  - If (N >= 0): The maximum credit for having digits in the new password. If you have fewer than N digits, each digit counts +1 toward meeting the current `minlen` value.
  - If (N < 0): The minimum number of digits that must be met for anew password.
- **ucredit=N**: Default = 1
  - If (N >= 0): The maximum credit for having uppercase letters in the new password. If you have fewer than or N uppercase letters, each letter counts +1 toward meeting the current `minlen` value.
  - If (N < 0): The minimum number of uppercase letters that must be met for a new password.
- **lcredit=N**: Default = 1
  - If (N >= 0): The maximum credit for having lowercase letters in the new password. If you have fewer than or N lowercase letters, each letter counts +1 toward meeting the current `minlen` value.
  - If (N < 0): The minimum number of lowercase letters that must be met for a new password.
- **ocredit=N**: Default = 1
  - If (N >= 0): The maximum credit for having other characters in the new password. If you have fewer than or N other characters, each character counts +1 toward meeting the current `minlen` value.
  - If (N < 0): The minimum number of other characters that must be met for a new password.
- **minclass=N**: Default = 0
  - N out of four character classes is required for the new password. The four classes are digits, uppercase letters, lowercase letters, and other characters.
- **maxrepeat=N**: Default = 0
  - Reject passwords that contain more than N same consecutive characters.
- **reject_username**: Check whether the name of the user in straight or reversed form is contained in the new password. If it is found, the new password is rejected.
- **try_first_pass**: Try to get the password from a previous PAM module. If that does not work, prompt the user for the password.
- **use_authtok**: This argument is used to *force* the module not to prompt the user for a new password. Instead, the new password is provided by the previously stacked *password* module.
- **dictpath=/path**: Path to the `cracklib` dictionaries.
- **maxsequence=N**: Default = 0 (meaning this check is disabled).
  - N set to any number other than 0 causes passwords with monotonic characters longer than that number to be rejected.
- **maxclassrepeat=N**: Default = 0 (meaning this check is disabled).
  - N set to any number other than 0 causes passwords with consecutive characters in the same class that are longer than that number to be rejected.
- **gecoscheck=N**: Causes passwords with more than three straight characters from the user's GECOS field, typically containing the user's real name, to be rejected.
- **enforce_for_root=N**: Enforce failed password checks for the root user. This option is off by default.

`password required pam_cracklib.so minlen=10 dcredit=-2`
- If added to the `/etc/pam.d/system-auth` file (or `/etc/pam.d/common-password` file for Ubuntu), this command will require passwords to be 10 characters long and they must contain 2 digits.

**Encouraging sudo use with PAM**

The `su` command is PAM-aware, which greatly simplifies things. It uses the PAM module `pam_wheel` to check for users in the `wheel` group.
- `cat /etc/pam.d/su`

First, to restrict the use of `su`, if you are using the `wheel` group as your administrative group, you need to reassign your administrative group to a new group. If you are not using the `wheel` group, just be sure not to assign anyone in the future to this group.
- Next, you need to edit the `/etc/pam.d/su` configuration file. Remove the comment mark (`#`) from the following line:
  - `#auth    required    pam_wheel.so use_uid`
  - With these modifications, PAM disables the use of the `su` command.

**Obtaining more information on PAM**

To get more information on PAM configuration files, use the command: `man pam.conf`.

To get more information about a particular PAM module, enter `man pam_module_name`. Be sure to leave off the file extension `.so` from the module name.

- **The Official Linux-PAM website**: http://linux-pam.org
- **The Linux-PAM System Administrator's Guide**: http://linux-pam.org/Linux-PAM-html/Linux-PAM_SAG.html
- **PAM Module reference**: http://linux-pam.org/Linux-PAM-html/sag-module-reference.html
***

## SELinux

*Security Enhanced Linux (SELinux)* was developed by the National Security Agency (NSA) along with other security research organizations, such as the Secure Computing Corporation (SCC).

SELinux is a security enhancement module deployed on top of Linux. SELinux provides improved security on the Linux system via role based access controls (RBACs) on subjects and objects (aka processes and resources). "Traditional" Linux security uses Discretionary Access Controls (DACs). With DAC, a process can access any file, directory, device, or other resource that leaves itself open to access. With RBAC, a process only has access to resources that it is explicitly allowed to access, based on the assigned role. The way that SELinux implements RBAC is to assign an SELinux policy to a process. That policy restricts access as follows:
- Only letting the process access resources that carry explicit labels.
- Making potentially insecure features, such as write access to a directory, available as Booleans, which can be turned on or off.
- DAC rules are still used when using SELinux.
- DAC rules are checked first, and if access is allowed, then SELinux policies are checked.
- If DAC rules deny access, SELinux policies are not reviewed.

SELinux is the default security enhancement of Red Hat distributions, whereas AppArmor is the default security enhancement for Ubuntu. You can still install SELinux on Ubuntu by using the command: `sudo apt-get install selinux` and then reboot.

### Understanding How SELinux Works

SELinux provides a combination of role based access control (RBAC) and either *type enforcement (TE)* or *Multi-Level Security (MLS)*. In role based access control, access to an object is based on a subject's assigned role in the organization. Therefore, it is not based on the subject's username or process ID. Each role is granted access rights.

**Understanding Type Enforcement**

Type enforcement (TE) is necessary to implement the RBAC model. Type enforcement secures a system through these methods:
- Labeling objects as certain security types.
- Assigning subjects to particular domains and roles.
- Providing rules allowing certain domains and roles to access certain object types.

`ls` with the `-Z` option displays the SELinux security RBAC controls:
- `ls -lZ my_stuff`
- Four items associated with the file that are specific to SELinux are displayed:
  - **user**: (`unconfined_u`)
  - **role**: (`object_r`)
  - **type**: (`user_home_t`)
  - **level**: (`s0`)
- These four RBAC items (user, role, type, and level) are used in the SELinux access control to determine appropriate access levels. Together, the items are called the SELinux *security context*. A security context (ID badge) is sometimes called a security label. These security context assignments are given to subjects (processes and users). Each security context has a specific name. The name given depends upon what object or subject it has been assigned: Files have a file context, users have a user context, and processes have a process context, also called a domain. The rules allowing access are called allow rules or policy rules. A *policy rule* is the process SELinux follows to grant or deny access to a particular system security type.

**Understanding Multi-Level Security**

With SELinux, the default policy type is called a *targeted policy*, which primarily controls how network services (such as web servers and file servers) can be accessed on a Linux system. For a more restrictive policy, you can choose *Multi-Level Security (MLS)*. MLS uses type enforcement along with the additional feature of security clearances. It also offers Multi-Category Security, which gives classification levels to objects.
- Multi-Level Security enforces the Bell-LaPadula Mandatory Access security model. Enforcing this model is accomplished by granting object access based on a role's security clearance and an object's classification level.
- *Security clearance* is an attribute granted to roles allowing access to classified objects. *Classification level* is an attribute granted to an object, providing protection from subjects who have a security clearance attribute that is too low.

**Implementing SELinux security models**

SELinux implements these models through a combination of four primary SELinux pieces:
- Operational modes
- Security contexts
- Policy types
- Policy rule packages

SELinux comes with three operational modes:
1. **Disabled**: SELinux is turned off.
  - Just edit the configuration file `/etc/selinux/config` and change the text `SELINUX=` to `SELINUX=disabled`. SELinux will be disabled after a system reboot.
2. **Permissive**: In *permissive* mode, SELinux is turned on, but the security policy rules are not enforced. A message is sent to a log file denoting that access should have been denied.
3. **Enforcing**: In *enforcing* mode, SELinux is turned on and all of the security policy rules are enforced.

**Understanding SELinux security contexts**

A security context consists of four attributes: `user`, `role`, `type`, and `level`.
- **User**: The `user` attribute is a mapping of a Linux username to an SELinux name. The SELinux username ends with a `u`. Regular unconfined users have an `unconfied_u` user attribute in the default targeted policy.
- **Role**: A designated role in the company is mapped to an SELinux role name. The `role` attribute is then assigned to various subjects and objects. Each role is granted access to other subjects and objects based on the role's security clearance and the object's classification level. The SELinux role name has an `r` at the end. On a targeted SELinux system, processes run by the root user have a `system_r` role, while regular users run under the `unconfined_r` role.
- **Type**: This type attribute defines a domain type for processes, a user type for users, and a file type for files. Most policy rules are concerned with the security type of a process and what files, ports, devices, and other elements of the system that process has access to. The SELinux type name ends with a `t`.
- **Level**: The `level` is an attribute of Multi-Level Security (MLS), and it enforces the Bell-LaPadula model. It is optional in TE but is required if you are using MLS. The MLS level is a combination of the sensitivity and category values that together form the security level. A level is written as `sensitivity : category`.
  - `sensitivity`:
    - Represents the security or sensitivity level of an object, such as confidential or top secret.
    - Is hierarchal, with `s0` (unclassified) typically being the lowest.
    - Is listed as a pair of sensitivity levels (`lowlevel-highlevel`) if the levels differ.
    - Is listed as a single sensitivity level (`s0`) if there are no low and high levels. In some cases, however, even if there are no low and high levels, the range is still shown (`s0-s0`).
  - `category`:
    - Represents the category of an object, such as No Clearance, Top Clearance, and so on.
    - Traditionally, the values are between `c0` and `c255`.
    - Is listed as a pair of category levels (`lowlevel.highlevel`) if the levels differ.
    - Is listed as a single category (level) if there are no low and high levels.

To see your SELinux user context, enter the `id` command at the shell prompt.

To see an individual file's context, use the `-Z` option on the `ls` command:
- `ls -Z my_stuff`

To see process information on a Linux system, you typically use a variant of the `ps` command.
- `ps -el | grep bash`

To see a process's security context, you need to use the `-Z` option on the `ps` command.
- `ps -eZ | grep bash`

**Understanding SELinux Policy types**

The *policy type* chosen directly determines what sets of policy rules are used to dictate what an object can access. The policy type also determines what specific security context attributes are needed.

SELinux has different policies among which you can choose:
- **Targeted**: The *Targeted* policy's primary purpose is to restrict "targeted" daemons. Targeted daemons are sandboxed. All subjects and objects not targeted are run in the `unconfined_t` domain.
    - SELinux comes with the Targeted policy set as the default.
- **MLS**: The *MLS* policy's primary purpose is to enforce the Bell-LaPadula model. It grants access to other subjects and objects based upon a role's *security clearance* and the object's *classification level*.
  - In the MLS policy, a security context's MLS attribute is critical. Otherwise, the policy rules will not know how to enforce access restrictions.
- **Minimum**:  The *Minimum* policy is essentially the same as the Targeted policy, but only the base policy rule package is used.

**Understanding SELinux policy rule packages**

*Policy rules*, also called *allow rules*, are the rules used by SELinux to determine if a subject has access to an object. Policy rules are installed with SELinux and are grouped into packages, also called *modules*.

On your Linux system, there is user documentation on these various policy modules in the form of HTML files. To view this documentation on Fedora or RHEL, open your system's browser and type in the following URL: `file:///usr/share/doc/selinux-poicy/html/index.html`. For Ubuntu, the URL is `file:///usr/share/doc/selinux-policy-doc/html/index.html`. If you do not have the policy documentation on your system, you can install it on Fedora or RHEL system by typing `yum install selinux-policy-doc` at the command line. On Ubuntu, type `sudo apt-get install selinux-policy-doc` at the command line.

### Configuring SELinux

SELinux configurations can only be set and modified by the root user. Configuration and policy files are located in the `/etc/selinux` directory. The primary configuration file is the `/etc/selinux/config` file.

**Setting the SELinux mode**

To see SELinux's current mode on your system, use the `getenforce` command. To see both the current mode and the mode set in the configuration file, use the `setstatus` command.

To change the mode setting, you can use the `setenforce <newsetting>`, where `<newsetting>` is either.
- `enforcing` or `1`.
- `permissive` or `0`.

To disable SELinux, you must edit the SELinux configuration file. When switching from disabled to either enforcing or permissive mode, SELinux automatically relabels the filesystem after a reboot.

To modify the mode in the `/etc/selinux/config` file, change the line `SELINUX=` to one of the following:
- `SELINUX=disabled`
- `SELINUX=enforcing`
- `SELINUX=permissive`

**Setting the SELinux policy type**

By default, the policy type is set to `targeted`. To change the default policy type, edit the `/etc/selinux/config` file. Change the line `SELINUXTYPE=` to one of the following:
- `SELINUXTYPE=targeted`
- `SELINUXTYPE=mls`
- `SELINUXTYPE=minimum`

If you set the SELinux type to mls or minimum, you need to make sure that you have their policy package installed first. Check by typing the following command:
- `yum list selinux-policy-mls` or `yum list selinux-policy-minimum`
- To check the SELinux policy packages on Ubunut, use the command: `sudo apt-cache policy package_name`.

**Managing SELinux security contexts**

To view current SELinux file and process security contexts, use the `secon` command. Below are some command options:

| Option | Description |
| ------ | ----------- |
| `-u` | Use this option to show the user of the security context. |
| `-r` | Use this option to show the role of the security context. |
| `-t` | Use this option to show the type of security context. |
| `-s` | Use this option to show the sensitivity level of the security context. |
| `-c` | Use this option to show the clearance level of the security context. |
| `-m` | Use this option to show the sensitivity and clearance level of the security context as an MLS range. |

If you use the `secon` command with no designation, it shows you the current process's security context. To see another process's security context, use the `-p` option.
- `secon -urt`
- `secon -urt -p 1`

To view a file's security context, you use the `-f` option.
- `secon -urt -f /etc/passwd`

**Managing the user security context**

To see a mapping list on your system, enter the `semanage login -l` command. If a user login ID is not listed, then it uses the "default" login mapping, which is the `Login` Name of `_default_`.
- `semanage login -l`

To see a current display of the SELinux users and their associated roles, use the command `semanage user -l`.

If you need to add a new SELinux username, the `semanage` utility is used again. This time, the command is `semanage user -a selinux_username`. To map a login ID to the newly-added SELinux username, the command is `semanage login -a -s selinux_username loginID`.

**Managing the file security context**

To see a file's current label (aka security context), use the `ls -Z` command.
- `ls -Z /etc/passwd`
- Below are some file security context label management commands:

| Utility | Description |
| ------- | ----------- |
| `chcat` | Use this to change a file's security context label's category. |
| `chcon` | Use this to change a file's security context label. |
| `fixfiles` | This calls the `restorecon`/`setfiles` utility. |
| `restorecon` | This does the exact same thing as `setfiles` utility, but it has a different interface than `setfiles`. |
| `setfiles` | Use this to verify and/or correct security context labels. It can be run for the file label verification and/or relabeling files when adding a new policy module to the system. Does exactly the same thing as the `restorecon` utility but has a different interface than `restorecon`. |

The command `restorecon <filename>` changes a file back to its default security context.

**Managing the process security context**

On a system with SELinux, a process is also given a security context. How a process gets its security context depends upon which process started it. Remember that `systemd` (previously `init`) is the "mother" of all processes. Thus, many daemons and processes are started by `systemd`. The processes `systemd` starts are given new security contexts. For instance, when the `apache` daemon is started by `systemd`, it is assigned the type (aka domain) `httpd_t`. The context assigned is handled by the SELinux policy written specifically for that daemon. If no poicy exists for a process, then it is assigned a default type, `unconfined_t`.
- For a program or application run by a user (parent process), the new process (child process) inherits the user's security context.
- You can use a couple of commands to change the security contexts under which a program is run:
  - `runcon`: Run the program using options to determine the user, role, and type (aka domain).
  - `sandbox`: Run the program within a tightly controlled domain (aka sandbox).

**Managing SELinux policy rule packages**

Policy rules are the rules used by SELinux to determine whether a subject has access to an object. They are grouped into packages, also called modules, and are installed with SELinux. An easy way to view the modules on your system is to use the `semodule -l` command.

**SELinux Poicy Package Tools**

| Policy Tool | Description |
| `audit2allow` | Generates policy allow/dontaudit rules from logs of denied operations. |
| `audit2why` | Generates a description of why the access was denied from logs of denied operations. |
| `checkmodule` | Compiles policy modules. |
| `checkpolicy` | Compiles SELinux policies. |
| `load_policy` | Loads new policies into the kernel. |
| `semodlue_expand` | Expands a policy module package. |
| `semodule_link` | Links policy module packages together. |
| `semodule_package` | Creates a policy module package. |

**Managing SELinux via Booleans**

To see a list of all of the current Booleans used in SELinux, use the `getsebool -a` command.

To see a specific policy that can be modified by a Boolean, the `getsebool` command. This command changes the policy rule temporarily. When the system is rebooted, the Boolean returns to its original setting. If you need this setting to be permanent, you can use `setsebool` with the `-P` option.
- The `setsebool` command has six settings: three for turning a poicy on (`on`, `1`, or `true`), and three for turning a policy off (`off`, `0`, or `false`).

### Monitoring and Troubleshooting SELinux

**Understanding SELinux logging**

SELinux uses a cache called the *Access Vecotr Cache (AVC)* when reviewing policy rules for particular security contexts. When access is denied, called an AVC denial, a denial message is put into a log file. Where these denial messages are logged depends upon the status of the `auditd` and `rsyslogd` daemonic:
- If the `auditd` daemon is running, the denial messages are logged to `/var/log/audit/audit.log`.
- If `auditd` is not running, but the `rsyslogd` daemon is running, the denial messages are logged to `/var/log/messages`.
- `aureport | grep AVC`
- After you discover that an AVC denial has been logged in `audit.log`, you can use `ausearch` to review the denial message(s).
  - `ausearch -m avc`
- The display provides information on who was attempting access, along with their security context when attempting it. Look for these key words in an AVC denial message:
  - `type=AVC`
  - `avc: denied`
  - `com="httpd"`
  - `path="/var/myserver/services"`

You can run the `journalctl` command to check for AVC denial log messages as well.
- `journalctl | grep AVC`
- Since you know that there are AVC denials, you can pass the entire `/var/log/audit/audit.log` file to `sealert` to step through the issues:
  - `sealert -a /var/log/audit/audit.log`

**Troubleshooting SELinux logging**

Sometimes AVC denials are not logged because of `dontaudit` policy rules. Although the `dontaudit` rules help reduce false positives in the logs, they can cause problems when troubleshooting. To fix this, temporarily disable all `dontaudit` policy rules using the command `semodule -DB`.

**Troubleshooting common SELinux problems**

Whenever access is denied, you should first check the "traditional" Linux DAC permissions. With SELinux, several regular items can cause problems:
- Using a nonstandard directory for a service.
- Using a nonstandard port for a service.
- Moving files that result in losing their security context labels.
- Having Booleans set incorrectly.

You must let SELinux know that you want the `http` service to be able to access the files within `/abc/www/html`. The commands to accomplish this are `semanage` and `restorecon`.
- `semanage fcontext -a -t httpd_sys_context_t "/abc/www/html(/.*)?"`
- `restorecon -R -v /abc/www/html`

You decide for security purposes to move `sshd` from port 22 to a nonstandard port, 47347. SELinux does not know about this port, and the service fails to start. To fix this problem, you must first find the security context type for `sshd`. This is accomplished using the code that follows by issuing the `semanage port -l` command and piping the results into `grep` to search for `ssh`.
- `semanage port -l | grep ssh`
- `semanage port -a -t ssh_port_t -p tcp 47347`
- `semanage port -l | grep ssh`
At this point, eidt the `/etc/ssh/sshd_config` file to add a `Port 47347` line to the file. Then restart the `sshd` service so that the service listens on the nonstandard port 47347.

To restore a file's permanent security context, use the `restorecon <file>` command.

### Obtaining More Information on SELinux

Issue the command `man -k selinux` to find all of the various man pages that you can review for the SELinux utilities currently installed on your system.

http://selinuxproject.org
***

## Securing Linux on a Network

A *network service* is any task that the computer performs requiring it to send and receive information over the network using some predefined set of rules.
- For Linux machines, the supported network services can be found in the `/etc/services` file.

### Evaluating access to network services with nmap

To install `nmap`, use the following commands:
- **RHEL**: `yum install nmap -y`
- **Ubuntu**: `sudo apt-get install nmap`

Ports, or more correctly *network ports*, are numeric values used by the TCP and UDP network protocols as access points to services on a system. Standard port numbers are assigned to services so that a service knows to listen on a particular port number and a client knows to request the service on that port number.

**Nmap Port Scans**:
- **TCP Connect port scan**: For this scan, `nmap` attempts to connect to ports using the Transmission Control Protocol (TCP) on the server. If a port is listening, the connection attempt succeeds.
  - If you select a TCP Connect port scan, the `nmap` utility uses the three-way TCP hand-shake to do a little investigative activity on a remote server.
  - `nmap -sT 127.0.0.1`
- **UDP port scan**: For this scan, `nmap` sends a UDP packet to every port on the system being scanned. If the port is listening and has a service that uses the UDP protocol, it responds to the scan.
  - `nmap -sU 127.0.0.1`
- **No-Ping Scan**: Disables the scan's `ping` probes.
  - `nmap -sT -PN 127.0.0.1`

Nmap reports six possible port states:
1. **open**: This is the most dangerous state an `nmap` scan can report for a port. An `open` port indicates that a server has a service handling requests on this port.
2. **closed**: A `closed` port is accessible, but there is no service waiting on the other side of this door.
3. **filtered**: This is the best state to secure a port that you don't want anyone to access. It is possible that a service could be listening on a particular port, but the firewall is blocking access to that port, effectively preventing any access to the service through the particular network interface.
4. **unfiltered**: The `nmap` scan sees the port but cannot determine if the port is `open` or `closed`.
5. **open|filtered**: The `nmap` scan sees the port but cannot determine if the port is `open` or `filtered`.
6. **closed|filtered**: The `nmap` scan sees the port but cannot determine if the port is `closed` or `filtered`.

### Working with Firewalls

Early versions of Linux use TCP wrappers to allow or deny access to Linux services. It did this by offering `/etc/hosts.allow` and `/etc/hosts.deny` files where you could specifically indicate which services are available and which are blocked to particular outside system names and/or IP addresses.
- The program that manages TCP Wrappers is `/usr/sbin/tcpd`.

A computer *firewall* blocks the transmission of malicious or unwanted data into and out of a computer system or network.
- In Linux, *iptables* is the kernel-level firewall feature. `iptables` works by allowing you to create rules that can be applied to every packet that tries to enter (`INPUT`), leave (`OUTPUT`), or cross through your system (`FORWARD`).
- A Linux firewall is really just a filter that checks each network packet or application request coming into or out of a computer system or network.

Firewalls can be placed into different categories, depending upon their function:
- **A firewall is either network-based or host-based**: A network-based firewall is one that is protecting the entire network or subnet. A host-based firewall is one that is running on and protecting an individual host or server.
- **A firewall is either a hardware or a software firewall**: Firewalls can be located on network devices, such as routers. Firewalls can be located on a computer system as an application. A software firewall is also called a rule-based firewall.
- **A firewall is either a network-layer filter or an application-layer filter**: A firewall that examines individual network packets is also called a *packet filter*. A *network-layer firewall* allows only certain packets into and out of the system. It operates on the lower layers of the OSI reference model. An *application-layer firewall* filters at the higher layers of the OSI reference model. This firewall allows only certain applications access to and from the system.

On a Linux system, the firewall is a host-based, network-layer, software firewall managed by the `iptables` utility and related kernel-level components. With `iptables`, you can create a series of rules for every network packet coming through your Linux server. Fedora, RHEL, and other Linux distributions have added the `firewalld` service to provide a more dynamic way of managing firewall rules than were offered previously. For recent RHEL and Fedora releases, the iptables firewall backend was relaced with nftables. The Firewall Configuration window (`firewall-config` command) provides an easy way to open ports on your firewall and do masquerading (routing private addresses to a public network) or port forwarding. The `firewalld` service can react to changes in conditions, which the static iptables service can't do as well on its own.
- The `iptables` utility manages the Linux firewall, called `netfilter`.

Below are some `firewalld` management commands:
- Check `firewalld` status: `systemctl status firewalld`
- Install `firewalld` and start:
  - `yum install firewalld firewall-config`
  - `systemctl start firewalld.service`
  - `systemctl enable firewalld.service`

To manage the `firewalld` service, you can start the Firewall Configuration window.
- `firewall-config &`

With `firewalld`, you can select from a set of firewall zones, depending on which services you want to share and the level of protection you want for your system. If you connect your computer to networks on which you have different levels of trust (such as a wireless network at an airport), you can adjust your firewall rules by selecting a different zone.
- Enabling the FTP service also causes connection tracking modules to be loaded that allow nonstandard ports to be accessed through the firewall when needed.

### Understanding the iptables utility

`netfilter`/`iptables` basics:
- Tables
- Chains
- Policies
- Rules

**netfilter/iptables tables**: There are four tables in the `iptables` utility, with an additional table added by SELinux.
- **filter**: The `filter` table is the packet filtering feature of the firewall. In this table, access control decisions are made for packets traveling to, from, and through your Linux system.
- **nat**: The `nat` table is used for Network Address Translation (NAT). NAT table rules let you redirect where a packet goes.
- **mangle**: As you would suspect, packets are mangled (modified) according to the rules in the `mangle` table.
- **raw**: The `raw` table is used to exempt certain network packets from something called "connection tracking". This feature is important when you are using Network Address Translation and virtualization on your Linux server.
- **security**: This table is available only on Linux distributions that have SELinux. Although typically not used directly, the security allows SELinux to allow or block a packet based on SELinux policies.

**netfilter/iptables chains**: The `netfilter`/`iptables` firewall categorizes network packets into categories, called *chains*.
- **INPUT**: Network packets coming *into* the Linux server.
- **FORWARD**: Network packets coming into the Linux server that are to be *routed* out through another network interface on the server.
- **OUTPUT**: Network packets coming *out* of the Linux server.
- **PREROUTING**: Used by NAT for modifying network packets when they come into the Linux server.
- **POSTROUTING**: Used by NAT for modifying network packets before they come out of the Linux server.

**Chains Available for Each netfilter/iptables Table**

| Table | Chains Available |
| ----- | ---------------- |
| `filter` | INPUT, FORWARD, OUTPUT |
| `nat` | PREROUTING, OUTPUT, POSTROUTING |
| `mangle` | INPUT, FORWARD, PREROUTING, OUTPUT, POSTROUTING |
| `raw` | PREROUTING, OUTPUT |
| `security` | INPUT, FORWARD, OUTPUT |

After a network packet is categorized into a specific chain, `iptables` can determine what policies or rules apply to that particular packet.

**netfilter/iptables rules, policies, and targets**: For each network packet, a rule can be set up defining what to do with that individual packet.
- Source IP address
- Destination IP address
- Network protocol
- Inbound port
- Outbound port
- Network state

If no rule exists for a particular packet, then the overall policy is used. Each packet category or chain has a default policy. After a network packet matches a particular rule or falls to the default policy, then action on the packet can occur. The action taken depends upon what `iptables` target is set.
- **ACCEPT**: Network packet is accepted into the server.
- **REJECT**: Network packet is dropped and not allowed into the server. A rejection message is sent.
- **DROP**: Network packet is dropped and not allowed into the server. No rejection message is sent.
You may consider using `REJECT` for internal employees who should be told that you are rejecting their outbound network traffic and why. Consider using `DROP` for inbound traffic so that any malicious personnel are unaware that their traffic is being blocked.
- The `iptables` utility implements a software firewall using the `filter` table via policies and rules.

**Ubuntu netfilter/iptables firewall**: The firewall interface service running on this distribution is `ufw`. To see if the firewall service is running, enter `sudo ufw status` at the command line. The `ufw` service is an interface to the `iptables` utility that does not run as a service on Ubuntu. You can use `ufw` commands to manipulate firewall rules. However, all of the `iptables` utility commands are still valid for Ubuntu:
- To enable the firewall, enter `sudo ufw enable` at the command line.
- To disable the firewall, enter `sudo ufw disable` at the command line.

To see what policies and rules are currently in place for the `filter` (default) table, enter `iptables -nvL` at the command line.

**Modifying iptables policies and rules**: Below are some options for `iptables`:
- `-t <table>`: The `iptables` command listed along with this switch is applied to the table. By default, the `filter` table is used.
  - `iptables -t filter -P OUTPUT DROP`
- `-P <chain> <target>`: Sets the overall policy for a particular *chain*.
  - `iptables -P INPUT ACCEPT`
- `-A <chain>`: Sets a rule called an "appended rule", which is an exception to the overall policy for the *chain* designated.
  - `iptables -A OUTPUT -d 10.140.67.25 -j REJECT`
- `-I <rule#> <chain>`: Inserts an appended rule into a specific location, designated by the *rule#*, in the appended rule list for the *chain* designated.
  - `iptables -I 5 INPUT -s 10.140.67.23 -j DROP`
- `-D <chain> <rule#>`: Deletes a particular rule, designated by the *rule#*, from the *chain* designated.
  - `iptables -D INPUT 5`
- `-j <target>`: If the criteria in the rule are met, the firewall should jump to this designated *target* for processing.
  - `iptables -A INPUT -s 10.140.67.25 -j DROP`
- `-d <IP address>`: Assigns the rule listed to apply to the designated destination *IP address*.
  - `iptables -A OUTPUT -d 10.140.67.25 -j REJECT`
- `-s <IP address>`: Assigns the rule listed to apply to the designated source *IP address*.
  - `iptables -A INPUT -s 10.140.67.24 -j ACCEPT`
- `-p <protocol>`: Assigns the rule listed to apply to the *protocol* designated.
  - `iptables -A INPUT -p icmp -j DROP`
- `--dport <port#>`: Assigns the rule listed to apply to certain protocol packets coming into the designated *port#*.
  - `iptables -A INPUT -p tcp --dport 22 -j DROP`
- `--sport <port#>`: Assigns the rule listed to apply to certain protocol packets going out of the designated *port#*.
  - `iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT`
- `-m <state> --state <network_state>`: Assigns the rule listed to apply to the designated *network stat(s)*.
  - `iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT`

For policies, you cannot set the target to `REJECT`. It fails, and you receive the message "iptables: Bad policy name". Use `DROP` as your policy instead.

You can save the current set of firewall filter rules using the `iptables-save` command.
- `iptables-save > /tmp/myiptables`

You can flush the current `iptables` settings with the `iptables -F` command.

To restore `iptables` to a saved set of rules, use the `iptables-restore` command.
- `iptables-restore < /tmp/myiptables`

For an Ubuntu system, you can still use the `iptables-save` command to create an `iptables` configuration file from the current `iptables` setting and use `iptables-restore` to restore it.
