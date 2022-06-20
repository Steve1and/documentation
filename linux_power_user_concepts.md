# Linux Power-User Concepts
***

## Linux Desktop Environment Concepts

### X Window System
The X window system provides the basis for almost all Linux Desktop Environments.
- Works in a sort of backward client/server model.
- It provides a framework on which different types of desktop environments or simple window managers can be built.
- X predates Windows; it was built to be a lightweight, networked desktop framework.
- The X server runs on the local system, providing an interface to your screen, mouse, and keyboard.
  - X clients (such as word processors, music players, or image viewers) can be launched from the local system or from any system on your network.
- X itself provides a plain gray background and a simple "X" mouse cursor. There are no menus, panels, or icons on a plain X screen. Other features are added by a window manager.
- A full-blown desktop environment includes a window manager, but also adds menus, panels, and usually an application programming interface that is used to create applications that play well together.
  - A *window manager* adds the capability to manage the windows on a desktop and often provides menus for launching applications and otherwise working with the desktop.

**Desktop Environment Examples**
- **GNOME**: Focuses more on stability (MAC-Based)
- **K Desktop Environment (KDE)**: More effects than GNOME (Windows-Based)
- **Xfce**: Lightweight desktop environment that's good for older systems
- **LXDE**: Lightweight X11 Desktop Environment; newer lightweight desktop environment

### GNOME 3
- **Windows Key (Super Key)**: Switch between the blank desktop and the overview screen
  - Same can be accomplished by selecting the **Activities** menu.
  - Can also be accomplished by pressing `Alt + F1`
- **Work-spaces**: Shown on the right in the overview screen. Can be switched between either in the overview screen or pressing `Ctrl + Alt + up/down`
  - A new workspace is used when an application is opened in the last one.
- **Select Different Views**: From the Windows or Applications view, hold `Ctrl + Alt + Tab` to see a menu of different views. Keep pressing tab to switch between views.
  - **Top Bar**: Keeps the current view
  - **Dash**: Highlights the first application in the application bar on the left. Use the arrow keys to move up and down that menu, and press Enter to open the highlighted application.
  - **Windows**: Selects the Windows view.
  - **Applications**: Selects the Applications view.
  - **Search**: Highlights the search box. Type a few letters to show only icons for applications that contain the letters you type. Press Enter to launch an application.
  - **Message Tray**: Reveals the bottom message tray. This tray lets you view notifications and open removable media.
- **Select an Active Window**: On an active workspace, press `Alt + Tab` to see a list of all active windows. Continue to hold the Alt key and press the Tab key (or right or left arrow keys) to highlight the application you want from the list of active desktop application windows.
  - If an application has multiple windows open, press `Alt + back-tick` to choose among sub-menus.
  - Release the Alt key to select it.
- **Command Box**: Pres `Alt + F2` to display the command box.
- **Nautilus File Manager**: Comes with the GNOME desktop and works like other file managers like those in Windows or MAC.
- **Lock Screen**: `Ctrl + Alt + L`
- **Print Screen**: `Alt + Print Screen`, take a picture of the current window

### GNOME 2
- **Metacity**: The default window manager for GNOME 2.
- **Compiz**: window manager that can provide 3D Desktop effects.
- **Nautilus**: file manager/graphical shell
- **GNOME Panels**: Used for applications/task launcher; line the top and bottom of the screen.
- **Desktop Area**: Windows and icons on the desktop.
***

## Shell Basics
**Shell**: the program used to interpret and manage commands.
- Special shell features enable the user to gather data input and direct data output between commands and Linux filesystems.
- **$**: The default prompt for a regular user.
- **#**: The default prompt for the root user.

**Virtual Consoles**: Most Linux systems that include a desktop interface start multiple virtual consoles running on the computer. Virtual consoles are a way to have multiple shell sessions open at once in addition to the graphical interface.
- You can switch between virtual consoles by holding the `Ctrl` and `Alt` keys and pressing a function key between `F1` and `F6`.
- You can return to the GUI (if one is running) by pressing `Ctrl+Alt+F1`. On some systems, the GUI may run on a different virtual console, such as virtual console 2 (`Ctrl+Alt+F2`). Newer systems now start the gdm (the login screen) persistently on tty1 to allow multiple simultaneous GUI sessions: the gdm is on tty1, the first desktop is started on tty2, the second desktop is started on tty3, and so on.

**Figuring out the Default Shell**
- Useful commands:
  - `whoami`
  - `grep <username> /etc/passwd`

The characters and words that you can type after a command are called *options* and *arguments*.

### Command Syntax
Most commands have one or more *options* that you can add to change the command's behavior. Options typically consist of a single letter preceded by a hyphen.
- `ls -l -a -t`
- `ls -lat`

Some commands include options that are represented by a whole word. To tell a command to use a whole word as an option, you typically precede it with a double hyphen (--).
- `ls --help`

Many commands also accept arguments after certain options are entered or at the end of the entire command line. An *argument* is an extra piece of information, such as a filename, directory, username, device, or other item, that tells the command what to act on.
- `tar -cvf backup.tar /home/spar`
- For full-word options, the argument often follows an equal sign (=).
  - `ls --hide=Desktop`

When you log in to a Linux system, Linux views you as having a particular identity, which includes your username, group name, user ID, and group ID. Linux also keeps track of your login session: It knows when you logged in, how long you have been idle, and where you logged in from.

### Locating Commands
To find commands you type, the shell looks in what is referred to as your *path*. For commands that are not in your path, you can type the complete identity of the location of the command.
- The path consists of a list of directories that are checked sequentially for the commands you enter. To see your current path, enter the following:
  - `echo $PATH`
- Most user commands that come with Linux are stored in the `/bin`, `/usr/bin`, or `/usr/local/bin` directories. The `/sbin` and `/usr/sbin` directories contain administrative commands.
  - To make commands available to all users, add them to the `/usr/local/bin` directory.

Linux does not, by default, check the current directory for an executable before searching the path. It immediately begins searching the path and executables in the current directory are run only if they are in the *PATH* variable or you give their absolute or relative location.
- **Absolute Location**: Full path typed out (e.g., `/bin/bash`)
- **Relative Location**: Referencing a file in the current directory (e.g., `./command`)

Here is the order in which the shell checks for the commands you type:
1. **Aliases**: These are the names set by the `alias` command that represent a particular command and a set of options.
2. **Shell reserved word**: These are words reserved by the shell for special use. For example: `do`, `while`, `case`, and `else`.
3. **Function**: This is a set of commands that is executed together within the current shell.
4. **Built-in command**: This is a command built into the shell. As a result, there is no representation of the command in the filesystem.
  - Examples: `cd`, `echo`, `exit`, `fg`, `history`, `pwd`, `set`, and `type`.
5. **Filesystem command**: This command is stored in and executed from the computer's filesystem.

To determine the location of a particular command, you can use the `type` command.

### Recalling Commands Using Command History
The shell history is a list of the commands that you have entered before. Using the `history` command in a bash shell, you can view your previous commands.

By default, the bash shell uses command-line editing that is based on the emacs text editor.
- To set `vi` as the default, use the `set -o vi` command. To make this change permanent, add the command to the bottom of the `~/.bashrc` file.

**Keystrokes for Navigating Command Lines**

| Keystroke | Full Name | Meaning |
| --------- | --------- | ------- |
| `Ctrl+F` | Character forward | Go forward one character. |
| `Ctrl+B` | Character backward | Go backward one character. |
| `Alt+F` | Word forward | Go forward one word. |
| `Alt+B` | Word backward | Go backward one word. |
| `Ctrl+A` | Beginning of line | Go to the beginning of the current line. |
| `Ctrl+E` | End of line | Go to the end of the line. |
| `Ctrl+L` | Clear screen | Clear screen and leave line at the top of the screen. |

**Keystrokes for Editing Command Lines**

| Keystroke | Full Name | Meaning |
| --------- | --------- | ------- |
| `Ctrl+D` | Delete current | Delete the current character. |
| `Backspace` | Delete previous | Delete the previous character. |
| `Ctrl+T` | Transpose character | Switch positions of the current and previous characters. |
| `Alt+T` | Transpose words | Switch positions of current and previous words. |
| `Alt+U` | Uppercase words | Change the current word to uppercase. |
| `Alt+L` | Lowercase words | Change the current word to lowercase. |
| `Alt+C` | Capitalize words | Change the current word to an initial capital. |
| `Ctrl+V` | Insert special character | Add a special character. For example, to add a Tab character, press `Ctrl+V+Tab`. |

**Keystrokes for Cutting and Pasting Text from within Command Lines**

| Keystroke | Full Name | Meaning |
| --------- | --------- | ------- |
| `Ctrl+K` | Cut end of line | Cut text to the end of the line. |
| `Ctrl+U` | Cut beginning of line | Cut text to the beginning of the line. |
| `Ctrl+W` | Cut previous word | Cut the word located behind the cursor. |
| `Alt+D` | Cut next word | Cut the word following the cursor. |
| `Ctrl+Y` | Paste recent text | Paste most recently cut text. |
| `Alt+Y` | Paste earlier text | Rotate back to previously cut text and past it. |
| `Ctrl+C` | Delete whole line | Delete the entire line. |

After you type a command, the entire command line is saved in your shell's history list. The list is stored in the current shell until you exit the shell. After that, it is written to a history file, from which any command can be recalled to be run again in your next session. To view the history, use the `history` command.
- A number precedes each command line in the list. You can recall one of the commands using an exclamation point (!).
  - `!#`: *Run command number*: Replace the # with the number of the command line and that line is run.
  - `!!`: *Run previous command*
  - `!?string?`: *Run command containing string*: Runs the most recent command that contains a particular string of characters.
- Type `fc` followed by a history line number, and that command line is opened in a text editor (`vi` by default). Make the changes that you want. When you exit the editor, the command runs.

**Keystrokes for Using Command History**

| Key(s) | Function Name | Description |
| ------ | ------------- | ----------- |
| Arrow keys | Step | Press the up and down arrow keys to step through each command line in your history list to arrive at the one you want. (`Ctrl+P` and `Ctrl+N` do the same functions, respectively). |
| `Ctrl+R` | Reverse incremental search | After you press these keys, you enter a search string to do a reverse search. As you type the string, a matching command line appears that you can run or edit. |
| `Ctrl+S` | Forward incremental search | This is the same as the preceding function but for forward search. (It may not work in all instances). |
| `Alt+P` | Reverse search | After you press these keys, you enter a string to do a reverse search. Type a string and press Enter to see the most recent command line that includes that string. |
| `Alt+N` | Forward search | This is the same as the preceding function but for forward search. (It may not work in all instances). |

After you close your shell, the history list is stored in the `.bash_history` file in your home directory. Up to 1,000 history commands are stored by default.
- This can be disabled by setting the `$HISTFILE` shell variable to `/dev/null`.

### Command-line Completion
To attempt to complete a value, type the first few characters and press Tab. Here are some values you can type partially from a bash shell:
- **Command, alias, or function**: If the text begins with regular characters, the shell tries to complete the text with a command, alias, or function name.
- **Variable**: text that begins with a ($) dollar sign.
- **Username**: text that begins with a (~) tilde.
- **Hostname**: text that begins with a (@) at symbol.

To check the possible ways that text can be expanded, press Tab twice.

### Connecting and Expanding Commands
**Metacharacter**: a typed character that has special meaning to the shell for connecting commands or requesting expansion.
- **Pipe (|)**: connects the output from one command to the input of another command.
  - `cat /etc/passwd | sort | less`
- **Semicolons (;)**: Creates a sequence of commands to run, with one command completing before the next command begins by separating them with semicolons.
  - `date ; troff -me verylargedocument | lpr ; date`
- **Ampersand (&)**: Runs commands in the background by putting an ampersand at the end.
  - `troff -me verylargedocument | lpr &`
  - The process will be killed if the shell is closed before it is complete.
- **Dollar Sign and Parentheses ($()) and Backticks (``)**: If commands are placed in these the command's standard output will be the argument for another command.
  - `vi $(find /home | grep xyzzy)`
- **Dollar Sign and Square Brackets ($[]) and Dollar Sign and Parentheses ($())**: These can be used to expand an arithmetic expression and pass it to the shell.
  - `echo "I am $[2020 - 1957] years old."`
- **Dollar Sign and Variable Name ($VAR)**: accesses the value of the variable used rather than the variable name.
  - `ls -l $BASH`

### Using Shell Variables
The shell itself stores information that may be useful to the user's shell session in what are called variables.
- You can see all variables set for your current shell by typing the `set` command.
- **Environment Variables**: variables that are exported to any new shells opened from the current shell.
  - Type `env` to see environment variables.
- Type `declare` to get a list of the current environment variables and their values along with a list of shell functions.

Using the `alias` command, you can effectively create a shortcut to any command and options that you want to run later.
- `alias p='pwd ; ls -CF'`
- `alias rm='rm -i'`
The `unalias` command removes aliases.

**Common Shell Environment Variables**

| Variable | Description |
| -------- | ----------- |
| BASH | This contains the full pathname of the `bash` command. This is usually `/bin/bash`. |
| BASH_VERSION | This is a number representing the current version of the `bash` command. |
| EUID | This is the effective user ID number of the current user. It is assigned when the shell starts, based on the user's entry in the `/etc/passwd` file. |
| FCEDIT | If set, this variable indicates the text editor used by the `fc` command to edit `history` commands. If this variable isn't set, the `vi` command is used. |
| HISTFILE | This is the location of your history file. It is typically located at `$HOME/bash_history`. |
| HISTFILESIZE | This is the number of history entries that can be stored. After this number is reached, the oldest commands are discarded. The default value is 1000. |
| HISTCMD | This returns the number of the current command in the `history` list. |
| HOME | This is your home directory. It is your current working directory each time you log in or type the `cd` command without any options. |
| HOSTTYPE | This is a value that describes the computer architecture on which the Linux system is running. For Intel-compatible PCs, the value is i386, i486, i586, i686, or something like i386-linux. For AMD 64-bit machines, the value is x86_64. |
| MAIL | This is the location of your mailbox file. The file is typically your username in the `/var/spool/mail` directory. |
| OLDPWD | This is the directory that was the working directory before you changed to the current working directory. |
| OSTYPE | This name identifies the current operating system. For Fedora Linux, the `OSTYPE` value is either linux or linux-gnu, depending on the type of shell you are using. (Bash can run on other operating systems as well). |
| PATH | This is the colon-separated list of directories used to find commands that you type. The default value for regular users varies for different distributions but typically includes the following: `/bin:/usr/bin:/usr/local/bin:/usr/bin/X11:/usr/X11R6/bin:~/bin`. You need to type the full path or a relative path to a command that you want to run which is not in your PATH. For the root user, the value also includes `/sbin`, `/usr/sbin`, and `/usr/local/sbin`. |
| PPID | This is the process ID of the command that started the current shell (for example, the Terminal window containing the shell). |
| PROMPT_COMMAND | This can be set to a command name that is run each time before your shell prompt is displayed. Setting `PROMPT_COMMAND=date` lists the current date/time before the prompt appears. |
| PS1 | This sets the value of your shell prompt. There are many items that you can read into your prompt (date, time, username, hostname, and so on). Sometimes a command requires additional prompts, which you can set with the variables `PS2`, `PS3`, and so on. |
| PWD | This is the directory that is assigned as your current directory. This value changes each time you change directories using the `cd` command. |
| RANDOM | Accessing this variable causes a random number to be generated. The number is between 0 and 99999. |
| SECONDS | This is the number of seconds since the time the shell was started. |
| SHLVL | This is the number of shell levels associated with the current shell session. When you log in to the shell, the SHLVL is 1. Each time you start a new bash command (by, for example, using `su` to become a new user, or by simply typing `bash`), this number is incremented. |
| TMOUT | This can be set to a number representing the number of seconds the shell can be idle without receiving input. After the number of seconds is reached, the shell exits. This security feature makes it less likely for unattended shells to be accessed by unauthorized people. (This must be set in the login shell for it actually to cause the shell to log out the user). |

To exit the shell when you are finished, type the `exit` command or press `Ctrl+D`. If this is done in a virtual console, the shell exits and returns you to a login prompt.

The `su` command opens a shell as a new user.

### Creating Your Shell Environment

**Bash Configuration Files**

| File | Description |
| ---- | ----------- |
| `/etc/profile` | This sets up user environment information for every user. It is executed when you first log in. This file provides values for your path in addition to setting environment variables for such things as the location of your mailbox and the size of your history files. Finally, `/etc/profile` gathers shell settings from configuration files in the `/etc/profile.d` directory. |
| `/etc/bashrc` | This executes for every user who runs the bash shell each time a bash shell is opened. It sets the default prompt and may add one or more aliases. Values in the file can be overridden by information in each user's `~/.bashrc` file. |
| `~/.bash_profile` | This is used by each user to enter information that is specific to his or her use of the shell. It is executed only once -- when the user logs in. By default, it sets a few environment variables and executes the user's `.bashrc` file. This is a good place to add environment variables because once set, they are inherited by future shells. |
| `~/.bashrc` | This contains the information that is specific to your bash shells. It is read when you log in and also each time you open a new bash shell. This is the best location to add aliases so that your shell picks them up. |
| `~/.bash_logout` | This executes each time you log out (exit the last bash shell). |

It is better to create an `/etc/profile.d/custom.sh` file to add system-wide settings instead of editing default files directly. Users can change the information in the `$HOME/.bash_profile`, `$HOME/.bashrc`, and `$HOME/.bash_logout` files in their own home directories.

To have the new information you just added to the file available from the current shell, type the following:
- `source $HOME/.bashrc`

#### Setting your Prompt
The `PS1` environment variable sets what the prompt contains and is what you will interact with most of the time.

You can use several special characters (indicated by adding a backslash to a variety of letters) to include different information in your prompt.

**Characters to Add Information to Bash Prompt**

| Special Character | Description |
| ----------------- | ----------- |
| `\!` | This shows the current command history number. This includes all previous commands stored for your username. |
| `\#` | This shows the command number of the current command. This includes only the commands for the active shell. |
| `\$` | This shows the user prompt ($) or root prompt (#), depending on which type of user you are. |
| `\W` | This shows only the current working directory base name. For example, if the current working directory was `/var/spool/mail`, this value simply appears as mail. |
| `\[` | This precedes a sequence of nonprinting characters. This can be used to add a Terminal control sequence into the prompt for such things as changing colors, adding blink effects, or making characters bold. (Your Terminal determines the exact sequences available). |
| `\]` | This follows a sequence of nonprinting characters. |
| `\\` | This shows a backslash. |
| `\d` | This displays the day name, month, and day number of the current date, for example, Sat Jan 23. |
| `\h` | This shows the hostname of the computer running the shell. |
| `\n` | This causes a newline to occur. |
| `\nnn` | This shows the character that relates to the octal number replacing nnn. |
| `\s` | This displays the current shell name. For the bash shell, the value would be bash. |
| `\t` | This prints the current time in hours, minutes, and seconds, for example, 10:14:39. |
| `\u` | This prints your current username. |
| `\w` | This displays the full path to the current working directory. |

To make a change to your prompt permanent, add the value of `PS1` to your `.bashrc` file in your home directory (assuming you are using the bash shell).

Reference for more options for `PS1`: http://www.tldp.org/HOWTO/Bash-Prompt-HOWTO

To modify the `PATH` variable by adding a new directory, use the following format example:
- `PATH=$PATH:/stuff/bin ; export PATH`

To set custom variables, use capital letters and export when done:
- `M=/work/time/files/info/memos ; export M`

### Manual Pages
Man pages are the most common means of getting information about commands as well as other basic components of a Linux system. Each man page falls into one of the categories listed below:

**Manual Page Sections**

| Section Number | Section Name | Description |
| -------------- | ------------ | ----------- |
| 1 | User Commands | Commands that can be run from the shell by a regular user (typically no administrative privilege is needed). |
| 2 | System Calls | Programming functions used within an application to make calls to the kernel. |
| 3 | C Library Functions | Programming functions that provide interfaces to specific programming libraries (such as those for certain graphical interfaces or other libraries that operate in user space). |
| 4 | Devices and Special Files | Filesystem nodes that represent hardware devices (such as Terminals or CD drives) or software devices (such as random number generators). |
| 5 | File Formats and Conventions | Types of files (such as a graphics or word processing file) or specific configuration files (such as the `passwd` or `group` file). |
| 6 | Games | Games available on the system. |
| 7 | Miscellaneous | Overviews of topics such as protocols, filesystems, character set standards, and so on. |
| 8 | System Administration Tools and Daemons | Commands that require root or other administrative privileges to use. |

To display the man page options for a command, use the `-k` option:
- `man -k passwd`

To see a specific man page, use that number as the argument:
- `man 5 passwd`

While displaying the man page, use the `/` forward slash to search for something in the page.
- Press `n` to repeat the search forward and `N` to repeat the search backward.
- To quit type `q`.

### Various Commands
- `date`: displays the current day, month, date, time, time zone, and year.
- `uname`: shows the type of system you are running.
- `id`: display information about your login identity.
- `who`: display information about the current login session.
***

## Moving Around the Filesystem
The Linux filesystem is the structure in which all of the information on your computer is stored.

In Linux, files are organized within a hierarchy of directories. Each directory can contain files as well as other directories.
- At the top is the *root* directory, which is represented by a single slash (`/`).

**Common Linux Directories**:
- `/bin`: Contains common Linux user commands.
- `/boot`: Has the bootable Linux kernel, initial RAM disk, and boot loader configuration files (GRUB).
- `/dev`: Contains files representing access points to devices on your systems.
- `/etc`: Contains administrative configuration files.
- `/home`: Contains directories assigned to each regular user with a login account.
- `/media`: Provides a standard location for auto-mounting devices.
- `/lib`: Contains shared libraries needed by applications in `/bin` and `/sbin` to boot.
- `/mnt`: A common mount point for many devices before it was supplanted by the standard `/media` directory.
- `/misc`: A directory sometimes used to auto-mount filesystems upon request.
- `/opt`: Directory structure available to store add-on application software.
- `/proc`: Contains information about system resources.
- `/root`: Represents the root user's home directory.
- `/sbin`: Contains administrative commands and daemon processes.
- `/sys`: Contains parameters for such things as tuning block storage and managing `cgroups`.
- `/tmp`: Contains temporary files used by applications.
- `/usr`: Contains user documentation, games, graphical files (X11), libraries (`lib`), and a variety of other commands and files that are not needed during the boot process.
- `/var`: Contains directories of data used by various applications, such as log files. On server computers, it is common to create the `/var` directory as a separate filesystem, using a filesystem type that can be easily expanded.

### Using Basic Filesystem Commands
**Commands to Create and Use Files**

| Command | Result |
| ------- | ------ |
| `cd` | Changes to another directory. |
| `pwd` | Prints the name of the current (or present) working directory. |
| `mkdir` | Creates a directory. |
| `chmod` | Changes the permission on a file or directory. |
| `ls` | Lists the contents of a directory. |

The `cd` command can be used with no options (to take you to your home directory) or with full or relative paths.
- The tilde (`~`) represents your home directory.

When you add a new user in RHEL-based distributions, the user is assigned to a group of the same name by default. This approach to assigning groups is referred to as the *user private group scheme.*

### Using Metacharacters and Operators
**Using File-Matching Metacharacters**
To save you some keystrokes and enable you to refer easily to a group of files, the bash shell lets you use meta-characters.
- `*`: Matches any number of characters.
- `?`: Matches any one character.
- `[...]`: Matches any one of the characters between the brackets, which can include a hyphen-separated range of letters or numbers.

**Using File-Redirection Metacharacters**
Commands receive data from standard input and send it to standard output. Using pipes (described earlier), you can direct standard output from one command to the standard input of another. With files, you can use less than (`<`) and greater than (`>`) signs to direct data to and from files.
- `<`: Directs the contents of a file to the command. In most cases, this is the default action expected by the command and the use of the character is optional; using `less <filename>` is the same as `less < <filename>`.
- `>`: Directs the standard output of a command to a file. If the file exists, the content of that file is overwritten.
- `2>`: Directs standard error (error messages) to the file.
- `&>`: Directs both standard output and standard error to the file.
- `>>`: Directs the output of a command to a file, adding the output to the end of the existing file.

Another type of redirection, referred to as *here text* (also called *here document*), enables you to type the text that can be used as standard input for a command. Here documents involve entering two less-than characters (`<<`) after a command, followed by a word. All typing following that word is taken as user input until the word is repeated on a line by itself.
```
$ mail root cnegus rjones bdecker << thetext
> I want to tell everyone that there will be a 10 a.m.
> meeting in conference room B. Everyone should attend.
>
> -- James
> thetext
$
```

**Using Brace Expansion Characters**
By using curly braces (`{}`), you can expand out a set of characters across filenames, directory names, or other arguments to which you give commands. For example, if you want to create a set of files such as `memo1` through `memo5`, you can do that as follows:
```
$ touch memo{1,2,3,4,5}
$ ls
memo1 memo2 memo3 memo4 memo5
```

### Listing Files and Directories
By default, when you type the `ls` command, the output shows you all non-hidden files and directories contained in the current directory. To list hidden files as well, use the long-listing (`-a`) option.
- As you look at the long listing, notice that the first character of each line shows the type of file:
  - A hyphen (`-`) indicates a regular file.
  - A `d` indicates a directory.
  - A `l` (lowercase L) indicates a symbolic link.
  - An executable file (a script or binary file that runs as a command) has execute bits turned on (x).

Special files in each directory:
- `.`: current directory.
- `..`: parent directory.

Instead of seeing the execute bit (x) set on an executable file, you may see an s in that spot instead. With an s appearing within either the owner (`-rwsr-xr-x`) or group (`-rwxr-sr-x`) permissions, or both (`-rwsr-sr-x`), the application can be run by any user, but ownership of the running process is assigned to the application's user/group instead of that of the user launching the command. This is referred to as a *set UID* or *set GID* program, respectively.

If a t appears at the end of a directory, it indicates that the *sticky bit* is set for that directory (for example, `drwxrwxr-t`). By setting the sticky bit on a directory, the directory's owner can allow other users and groups to add files to the directory but prevent users from deleting each other's files in that directory. With a set GID assigned to a directory, any files created in that directory are assigned the same group as the directory's group. (If you see a capital S or T instead fo the execute bits on a directory, it means that the set GID or sticky bit permission, respectively, was set, but for some reason the execute bit was not also turned on.)

If you see a plus sign at the end of the permission bits (for example, `-rw-rw-r--+`), it means that extended attributes (`+`), such as Access Control Lists (ACLs), are set on the file. A dot at the end (`.`) indicates that SELinux is set on the file.

Any file or directory beginning with a dot (`.`) is considered hidden and is not displayed by default with `ls`. These dot files are typically configuration files or directories that need to be in your home directory but don't need to be seen in your daily work.

### Understanding File Permissions and Ownership
The nine bits assigned to each file for permissions define the access that you and others have to your file. Permission bits for a regular file appear as `-rwxrwxrwx`. Those bits are used to define who can read, write, or execute the file. The first bit has different meanings based on the character, as shown below:
- `-`: indicates a regular file.
- `d`: indicates a directory.
- `l`: indicates a symbolic link.
- `b`: indicates a block device.
- `c`: indicates a character device.
- `s`: indicates a socket.
- `p`: indicates a named pipe.

Of the nine-bit permissions, the first three bits apply to the owner's permission, the next three apply to the group assigned to the file, and the last three apply to all others. The `r` stands for read, the `w` stands for write, and the `x` stands for execute permissions. If a dash appears instead of the letter, it means that permission is turned off for that associated read, write, or execute bit.

**Setting Read, Write, and Execute Permissions**

| Permission | File | Directory |
| ---------- | ---- | --------- |
| Read | View what's in the file. | See what files and subdirectories it contains. |
| Write | Change the file's content, rename it, or delete it. | Add files or subdirectories to the directory. Remove files of directories from the directory. |
| Execute | Run the file as a program. | Change to the directory as the current directory, search through the directory, or execute a program from the directory. Access file metadata (file size, time stamps, and so on) of files in that directory. |

Each permission (read, write, and execute) is assigned a number: (r=4, w=2, and x=1), and you use each set's total number to establish the permission.
- `chmod 777 file`: sets all permissions
- `chmod 755 file`: `rwxr-xr-x`
- `chmod 644 file`: `rw-r--r--`
- `chmod 000 file`: `---------`

The `chmod` command can be used recursively by using the `-R` option.

You can also turn file permissions on and off using plus (+) and minus (-) signs, respectively, along with letters to indicate what changes and for whom.
- `u`: user
- `g`: group
- `o`: other
- `a`: all users
- `r`: read
- `w`: write
- `x`: execute
Examples:
- `chmod go-rwx file`: Removes read, write, and execute permissions for the group and other permission settings.
- `chmod u+rw files`: Adds read and write to the user permissions.
- `chmod a+x files`: Adds execute to user, group, and other permissions.

When you create a file as a regular user, it's given permission `rw-rw-r--` by default. A directory is given the permission `rwxrwxr-x`. For the root user, file and directory permission are `rw-r--r--` and `rwxr-xr-x`, respectively. These default values are determined by the value of `umask`. Enter `umask` to see what your `umask` value is.
- Execute permissions are off by default for regular files.
- If you want to change your `umask` value permanently, add a `umask` command to the `.bashrc` file in your home directory (near the end of the file).

**Changing File Ownership**
As a regular user, you cannot change ownership of files or directories to have them belong to another user, but you can change ownership as the root user.
- `chown joe /home/joe/memo.txt`: changes the ownership of the file memo.txt to the user joe.

To change the group as well, put a colon (:) and the group after the username:
- `chown joe:joe /home/joe/memo.txt`

The `chown` command can be used recursively as well by using the `-R` option.

### Moving, Copying, and Removing Files
To change the location of a file, use the `mv` command. To copy a file from one location to another, use the `cp` command. To remove a file, use the `rm` command.
- `mv abc def`: renames the file abc to def.
  - The `-i` option asks you before overwriting existing files.
  - The `-b` option makes a backup copy of a file that already exists of the same name.
- `cp abc def`: copies the contents of the file abc into the new file def.
- `rm abc`: deletes the file abc.

To remove a directory, use the recursive `-r` option for `rm` or, for an empty directory, you can use the `rmdir` command.
- `rm -r /home/joe/randomdir`
- `rmdir /home/jow/randomdir`
***

## Working with Text Files
**VI Text Editor**
- `vi /tmp/test`: opens a file named test to be edited.

Vi has two main operating modes: command and input. The vi editor always starts in command mode. Before you can add or change text in the file, you have to type a command (one or two letters, sometimes preceded by an optional number) to tell vi what you want to do.

**Adding Text**
- `a`: The *add* command. With this command, you can input text that starts to the *right* of the cursor.
- `A`: The *add at end* command. With this command, you can input text starting at the end of the current line.
- `i`: The *insert* command. With this command, you can input text that starts to the *left* of the cursor.
- `I`: The *insert at beginning* command. With this command, you can input text that starts at the beginning of the current line.
- `o`: The *open below* command. This command opens a line below the current line and puts you in insert mode.
- `O`: The *open above* command. This command opens a line above the current line and puts you in insert mode.

When you are in insert mode, `-- INSERT --` appears at the bottom of the screen.

Remember the Esc key will place you back in command mode.

**Moving around in the text**
- **Arrow Keys**: Move the cursor up, down left, or right in the file one character at a time. To move left and right, you can also use Backspace and the spacebar, respectively.
- `h`: move one character to the left.
- `l`: move one character to the right.
- `j`: move one character down.
- `k`: move one character up.
- `w`: Moves the cursor to the beginning of the next word (delimited by spaces, tabs, or punctuation).
- `W`: Moves the cursor to the beginning of the next word (delimited by spaces or tabs).
- `b`: Moves the cursor to the beginning of the previous word (delimited by spaces, tabs, or punctuation).
- `B`: Moves the cursor to the beginning of the previous word (delimited by spaces or tabs).
- `0 (zero)`: Moves the cursor to the beginning of the current line.
- `$`: Moves the cursor to the end of the current line.
- `H`: Moves the cursor to the upper-left corner of the screen (first line on the screen).
- `M`: Moves the cursor to the first character of the middle line on the screen.
- `L`: Moves the cursor to the lower-left corner of the screen (last line on the screen).

**Deleting, copying, and changing text**
- `dw`: Deletes (d) a word (w) after the current cursor position.
- `db`: Deletes (d) a word (b) before the current cursor position.
- `dd`: Deletes (d) the entire current line (d).
- `c$`: Changes (c) the characters (actually erases them) from the current character to the end of the current line ($) and goes into input mode.
- `c0`: Changes (c) (again, erases), characters from the previous character to the beginning of the current line (0) and goes into input mode.
- `c1`: Erases (c) the current letter (1) and goes into input mode.
- `cc`: Erases (c) the line (c) and goes into input mode.
- `yy`: Copies (y) the current line (y) into the buffer.
- `y)`: Copies (y) the current sentence ( ) ), to the right of the cursor, into the buffer.
- `y}`: Copies (y) the current paragraph ( } ), to the right of the cursor, into the buffer.

**Pasting (putting) text**
- `P`: Puts the copied text to the left of the cursor if the text consists of letters or words; puts the copied text above the current line if the copied text contains lines of text.
- `p`: Puts the buffered text to the right of the cursor if the text consists of letters or words; puts the buffered text below the current line if the buffered text contains lines of text.

**Repeating commands**
After you delete, change, or paste text, you can repeat that action by typing a period (.).

**Exiting VI**
- `ZZ`: Saves the current changes to the file and exits from vi.
- `:w`: Saves the current file, but you can continue editing.
- `:wq`: Works the same as `ZZ`.
- `:q`: Quits the current file. This works only if you don't have any unsaved changes.
- `:q!`: Quits the current file and doesn't save the changes you just made to the file.

**Special Commands**
- `Esc`: Leaves input mode and returns to command mode.
- `u`: Undo the previous change you made.
- `Ctrl+R`: If you decide that you didn't want to undo the previous undo command, use `Ctrl+R` for Redo.
- `:!command`: You can run a shell command while you are in vi using `:!` followed by a shell command name.
- `Ctrl+g`: If you forget what you are editing, pressing these keys displays the name of the file you are editing and the current line that you are on at the bottom of the screen.

**Skipping around in the file**
- `Ctrl+f`: Pages ahead one page at a time.
- `Ctrl+b`: Pages back one page at a time.
- `Ctrl+d`: Pages ahead one-half page at a time.
- `Ctrl+u`: Pages back one-half page at a time.
- `G`: Goes to the last line of the file.
- `1G`: Goes to the first line of the file.
- `35G`: Goes to any line number (35, in this case).

**Searching for text**
To search for the next or previous occurrences of text in the file, use either the slash (/) or the question mark (?) character. Follow the slash or question mark with a pattern (string of text) to search forward or backward, respectively, for that pattern.
- `/hello`: searches forward for the word hello.
- `?goodbye`: searches backward for the word goodbye.

**Using ex mode**
The vi editor was originally based on the ex editor. When you type a colon and the cursor goes to the bottom of the screen, you are essentially in ex mode.
- `:g/Local`: Searches for the word Local and prints every occurrence of that line from the file.
- `:s/Local/Remote`: Substitutes Remote for the first occurrence of the word Local on the current line.
- `:g/Local/s//Remote`: Substitutes the first occurrence of the word Local on every line of the file with the word Remote.
- `:g/Local/s//Remote/g`: Substitutes every occurrence of the word Local with the word Remote in the entire file.
- `:g/Local/s//Remote/gp`: Substitutes every occurrence of the word Local with the word Remote in the entire file and then prints each line so that you can see the changes (piping it through less if the output fills more than one page).

`vimtutor`: opens a tutorial in the vim editor that steps through common commands.

### Finding Files
- `locate`: Find files and commands by name.
- `find`: Find files based on a lot of different attributes.
- `grep`: Search within text files to find lines in files that contain search text.

**Using locate to find files by name**
The `updatedb` command runs once per day to gather the names of files throughout your Linux system into a database. By running the `locate` command, you can search that database to find the location of files stored in it.
- Not every file in the filesystem is stored in the database. The contents of the `/etc/updatedb.conf` file limit which filenames are collected by pruning out select mount types, filesystem types, file types, and mount points.
- As a regular user you can't see any files from the locate database that you can't see in the filesystem normally.
- Using `locate -i`, filenames are found regardless of case.

**Searching for files with find**
When you run `find`, it searches your filesystem live, which causes it to run slower than `locate`, but it gives you an up-to-the-moment view of the files on your Linux system. However, you can also tell `find` to start at a particular point in the filesystem so that the search can go faster by limiting the area of the filesystem being searched.
- A long listing (ownership, permission, size, and so on) is printed with each file when you add `-ls` to the find command (similar to output of the `ls -l` command).
- To find files by name, you can use the `-name` and `-iname` options.
  - `find /etc -name passwd`
  - `find /etc -iname '*passwd'`
- The `-size` option enables you to search for files that are exactly, smaller than, or larger than a selected size.
  - `find /usr/share/ -size +10M`
- You can search for a particular owner (`-user`) or group (`-group`) when you try to find files. By using `-not` and `-or`, you can refine your search for files associated with specific users and groups.
  - `find /home -user chris`
  - `find /etc -group ntp`
- Use the `-perm` option to find files based on number or letter permissions.
  - `find /usr/bin -perm 755`
- Date and time stamps are stored for each file when it is created, when it is access, when its content is modified, or when its metadata is changed. Metadata includes owner, group, time stamp, file size, permissions, and other information stored in the file's inode.
  - `find /etc/ -mmin -10`: find what's changed in the last 10 minutes.
- The time options (`-atime`, `-ctime`, and `-mtime`) enable you to search based on the number of days since each file was accessed, changed, or had its metadata changed. The `min` options (`-amin`, `-cmin`, and `-mmin`) do the same in minutes. Numbers that you give as arguments to the `min` and `time` options are preceded by a hyphen (to indicate a time from the current time to that number of minutes or days ago) or a plus (to indicate time from the number of minutes or days ago and older). With no hyphen or plus, the exact number is matched.
- `find` has `-and`, `-not`, and `-or` options to add logic to a command.
  - `find /var/allusers/ -user joe -not -group joe`
- With the `-exec` option, the command you use is executed on every file found, without stopping to ask if that's okay. The `-ok` option stops at each matched file and asks whether you want to run the command on it. Use a set of curly braces to indicate where on the command line to read in each file that is found. To end the line, you need to add a backslash and a semicolon (`\;`).
  - `find /etc -iname passwd -exec echo "I found {}" \;`
  - `find /var/allusers -user joe -ok mv {} /tmp/joe/ \;`

**Searching in files with grep**
If you want to search for files that contain a certain search term, you can use the `grep` command. When you search, you can have every line containing the term printed on your screen (standard output) or just list the names of the files that contain the search term. By default, `grep` searches text in a case-sensitive way, although you can do case-insensitive searches as well. Also, instead of just searching files, you can use `grep` to search standard output as well. (Use the `-i` option).
- To search for lines that don't contain a selected text string, use the `-v` option.
- `grep desktop /etc/services`
- `grep -i desktop /etc/services`
- `grep -vi tcp /etc/services`
- To do recursive searches, use the `-r` option and a directory as an argument.
- The `-l` option just lists the files that include the search text, without showing the actual text.
- The `--color` option shows results in red.
- `grep -rli peerdns /usr/share/doc/`
- `grep -ri --color root /etc/sysconfig/`
- `ip addr show | grep inet`
***

## Managing Running Processes
Linux is a multitasking system. Multitasking means that many programs can be running at the same time. An instance of a running program is referred to as a process.

A process is a running instance of a command. A process is identified on the system by what is referred to as a *process ID* (PID). Each process, when it is run, is associated with a particular user account and group account. That account information helps determine what system resources that process can access.
- Each process stores its information in a subdirectory of `/proc`, named after the process ID of that process.

### Listing Processes
**Listing processes with ps**
The most common utility for checking running processes is the `ps` command.
- The `-u` option asks that usernames be shown, as well as other information such as the time the process started and memory and CPU usage for processes associated with the current user.
- The `STAT` column represents the state of the process, with `R` indicated a currently running process and `S` representing a sleeping process. A plus sign (`+`) indicates that the process is associated with the foreground operations.
- `VSZ` (Virtual Set Size) is the amount of memory allocated for the process, whereas `RSS` (Resident Set Size) is the amount that is actually being used. RSS memory represents physical memory that cannot be swapped.
- Background system processes perform such tasks as logging system activity or listening for data coming in from the network.
- `ps aux`
- Using the `-o` option, you can use keywords to indicate the columns you want to list with `ps`.
  - `ps -eo pid,user,uid,group,gid,vsz,rss,comm | less`
- If you want to sort by a specific column, you can use the `sort=` option.
  - `ps -eo pid,user,group,gid,vsz,rss,comm --sort=-vsz | head`

**Listing and changing processes with top**
The `top` command provides a screen-oriented means of displaying processes running on your system.
- If you want to be able to kill or renice any processes, you need to run `top` as the root user. If you just want to display processes and possibly kill or change your own processes, you can do that as a regular user.
- Press `h` to see help options, and then press any key to return to the `top` display.
- Press `M` to sort by memory usage instead of CPU, and then press `P` to return to sorting by CPU.
- Press the number `1` to toggle showing CPU usage of all your CPUs if you have more than one CPU on your system.
- Press `R` to reverse sort your output.
- Press `u` and enter a username to display processes only for a particular user.
- A process consuming too much CPU can be reniced to give it less priority to the processors.
- **Renicing a process**: Note the process ID of the process you want to renice and press `r`. When the `PID to renice` message appears, type the process ID of the process you want to renice. When prompted to `Renice PID to value`, type in a number from -20 to 19.
- **Killing a process**: Note the process ID of the process you want to kill and press `k`. Type `15` to terminate cleanly, or `9` to just kill the process outright.

### Managing Background and Foreground Process
**Starting background processes**
To place a program in the background at the time you run the program, type an ampersand (&) at the end of the command line.
- To check which commands you have running in the background, use the `jobs` command.
- To see the process ID for the background job, add a `-l` option to the `jobs` command.

**Using foreground and background commands**
You can bring any of the commands on the `jobs` list to the foreground using the `fg` command.
- `fg %1`
- To refer to a background job (to cancel or bring it to the foreground), use a percent sign (%) followed by the job number.
  - `%`: Refers to the most recent command put into the background (indicated by the plus sign when you type the jobs command).
  - `%string`: Refers to a job where the command begins with a particular string of characters.
  - `%?string`: Refers to a job where the command line contains a string at any point.
  - `%--`: Refers to the job stopped before the one most recently stopped.
- If a command is stopped, you can start it running again in the background using the `bg` command.
  - `bg %5`

### Killing and Renicing Processes
Although usually used for ending a running process, the `kill` and `killall` commands can actually be used to send any valid signal to a running process.

| Signal | Number | Description |
| ------ | ------ | ----------- |
| SIGHUP | 1 | Hang-up detected on controlling terminal or death of controlling process. |
| SIGINT | 2 | Interrupt from keyboard. |
| SIGQUIT | 3 | Quit from keyboard. |
| SIGABRT | 6 | Abort signal from abort(3). |
| SIGKILL | 9 | Kill signal. |
| SIGTERM | 15 | Termination signal. |
| SIGCONT | 19,18.25 | Continue if stopped. |
| SIGSTOP | 17,19,23 | Stop process. |

- `kill 10432`
- `kill -15 10432`
- `kill -SIGKILL 10432`
- The default signal sent by `kill` is 15 (SIGTERM).
- If something on your GNOME desktop were corrupted, you could send the `gnome-shell` a `SIGHUP` signal to reread its configuration files and restart the desktop.
- With the `killall` command, you can signal processes by name instead of by process ID.
  - Like the `kill` command, `killall` uses `SIGTERM` (signal 15) if you don't explicitly enter a signal number.
  - `killall -9 top`

**Setting processor priority with nice and renice**
When the Linux kernel tries to decide which running processes get access to the CPUs on your system, one of the things it takes into account is the nice value set on the process. Every process running on your system has a nice value between -20 and 19. By default, the nice value is set to 0.
- The lower the nice value, the more access to the CPUs the process has. In other words, the nicer the process is, the less CPU attention it gets. So, a -20 nice value gets more attention than a process with a 19 nice value.
- A regular user can set nice values only from 0 to 19.
- A regular user can set the nice value higher, not lower.
- A regular user can set the nice value only on the user's own processes.
- The root user can set the nice value on any process to any valid value, up or down.
- You can use the `nice` command to run a command with a particular nice value. When a process is running, you can change the nice value using the `renice` command.
  - `nice -n +5 updatedb &`
  - `renice -n -5 20284`

### Limiting Processes with cgroups
Setting the nice value for one process doesn't apply to child processes that a process might start up or any other related processes that are part of a larger service.

Cgroups can be used to identify a process as a task, belonging to a particular control group. Tasks can be set up in a hierarchy where, for example, there may be a task called daemons that sets default limitations for all daemon server processes, then subtasks that may set specific limits on a web server daemon (`httpd`) for FTP service daemon (`vsftpd`).
- The types of resources that can be limited by cgroups include the following:
  - **Storage (blkio)**: Limits total input and output access to storage devices (such as hard disks, USB drives, and so on).
  - **Processor scheduling (cpu)**: Assigns the amount of access a cgroup has to be scheduled for processing power.
  - **Process accounting (cpuacct)**: Reports on CPU usage. This information can be leveraged to charge clients for the amount of processing power they use.
  - **CPU assignment (cpuset)**: On systems with multiple CPU cores, assigns a task to a particular set of processors and associated memory.
  - **Device access (devices)**: Allows tasks in a cgroup to open or create (`mknod`) selected device types.
  - **Suspend/resume (freezer)**: Suspends and resumes cgroup tasks.
  - **Memory usage (memory)**: Limits memory usage by task. It also creates reports on memory resources used.
  - **Network bandwidth (net_cls)**: Limits network access to selected cgroup tasks. This is done by tagging network packets to identify the cgroup task that originated the packet and having the Linux traffic controller monitor and restrict packets coming from each cgroup.
  - **Network traffic (net_prio)**: Sets priorities of network traffic coming from selected cgropus and lets administrators change these priorities on the fly.
  - **Name spaces (ns)**: Separates cgroups into namespaces, so processes in one cgroup can only see the namespaces associated with the cgroup. Namespaces can include separate process tables, mount tables, and network interfaces.
- Cgroups can involve editing configuration files to create your own cgroups (`/etc/cgconfig.conf`) or set up limits for particular users or groups (`/etc/cgrules.conf`). Or you can use the `cgcreate` command to create cgroups, which resultes in the groups being added to the `/sys/fs/cgroup` hierarchy.
***

## Shell Scripting
A shell script is a group of commands, functions, variables, or just about anything else you can use from a shell. These items are typed into a plain-text file. That file can then be run as a command.

**Executing and debugging shell scripts**
You can execute a shell script in two basic ways:
- The filename is used as an argument to the shell (as in `bash myscript`). In this method, the file does not need to be executable; it just contains a list of shell commands.
- The shell script may also have the name of the interpreter placed in the first line of the script preceded by `#!` (as in `#!/bin/bash`) and have the execute bit of the file containing the script set (using `chmod +x filename`).

The pound sign (#) prefaces comments and can take up an entire line or exist on the same line after script code.

Shell Script Debugging tips:
- In some cases, you can place an `echo` statement at the beginning of the lines within the body of a loop and surround the command with quotes. This will print to `STDOUT` the result of the command rather than run it in the script context.
- You can place dummy `echo` statements throughout the code to determine execution steps.
- You can use `set -x` near the beginning of the script to display each command that is executed or launch your scripts using: `bash -x myscript`

**Understanding shell variables**
To store information used by a shell script in such a way that it can be easily reused, you can set *variables*. Variable names within shell scripts are case sensitive and can be defined in the following manner: `NAME=value`

Variables can contain the output of a command or command sequence. You can accomplish this by preceding the command with a dollar sign and open parenthesis, following it with a closing parenthesis. (e.g. `MYDATE=$(date)`).
- Enclosing the command in back-ticks can have the same effect. In this case, the `date` command is run when the variable is set and not each time the variable is read.

If you want to have the shell interpret a single character literally, precede it with a backslash (\). To have a whole set of characters interpreted literally, surround those characters with single quotes ('').
- Surround a set of text with double quotes if you want all but a few characters used literally.
- `echo '$HOME * $(date)'` => `$HOME * $(date)`
- `echo "$HOME * $(date)"` => `/home/user * Tue Jan 21 16:56:52 EDT 2020`
- `echo $HOME * $(date)` => `/home/user file1 file2 Tue Jan 21 16:56:52 EDT 2020`

**Special shell positional parameters**
There are special variables that the shell assigns for you. One set of commonly used variables is called *positional parameters* or *command-line arguments*, and it is referenced as `$0`, `$1`, `$2`, `$3`, .... `$n`.
- `$0` references the name of the script itself.

Another variable, `$#`, tells you how many parameters your script was given.
- The `$@` variable holds all of the arguments entered at the command line.
- `$?` stores the exit status of the last command executed.
  - 0 means the command exited successfully.
  - Anything other than 0 indicates and error of some kind.

**Reading in parameters**
Using the `read` command, you can prompt the user for information and store that information to use later in your script.
- `read -p "Type in an adjective, noun and verb (past tense): " adj1 noun1 verb1`

**Parameter expansion in bash**
Curly braces are used when the value of the parameter needs to be placed next to other text without a space. (`${CITY}`).
- `${var:-value}`: If variable is unset or empty, expand this to `value`.
- `${var#pattern}`: Chop the shortest match for `pattern` from the front of `var's` value.
- `${var##pattern}`: Chop the longest match for `pattern` from the front of `var's` value.
- `${var%pattern}`: Chop the shortest match for `pattern` from the end of `var's` value.
- `${var%%pattern}`: Chop the longest match for `pattern` from the end of `var's` value.

```
MYFILENAME=/home/user/myfile.txt      #Sets the value of MYFILENAME
FILE=${MYFILENAME##*/}                #FILE becomes myfile.txt
DIR=${MYFILENAME%/*}                  #DIR becomes /home/user
NAME=${FILE%.*}                       #NAME becomes myfile
EXTENSION=${FILE##*.}                 #EXTENSION becomes txt
```

**Performing arithmetic in shell scripts**
Bash used *untyped variables*, meaning it normally treats variables as strings of text, but you can change them on the fly if you want it to. You are not required to specify whether a variable is text or numbers.
- When you start trying to do arithmetic can be performed using the built-in `let` command or through the external `expr` or `bc` commands.

```
BIGNUM=1024
let RESULT=$BIGNUM/16
RESULT=$(expr $BIGNUM / 16)
RESULT=$(echo "$BIGNUM / 16" | bc)
let foo=$RANDOM; echo $foo
```

Another way to grow a variable incrementally is to use $(()) notation with ++I added to increment the value of I.

```
I=0
echo "The value of I after increment is $((++I))"
```

The `let` command insists on spaces between each operand and the mathematical operator, whereas the syntax of the `expr` command requires white space between each operand and its operator. In contrast to those, `bc` isn't picky about spaces, but it can be trickier to use because it does floating-point arithmetic.

**Using programming constructs in shell scripts**
The most commonly used programming construct is conditional execution, or the `if` statement. It is used to perform actions only under cetain conditions.
- `elif` (which stands for "else if") is used to test for an additional condition.

```
filename="$HOME"
if [ -f "$filename" ] ; then
  echo "$filename is a regular file"
elif [ -d "$filename" ] ; then
  echo "$filename is a directory"
else
  echo "I have no idea what $filename is"
fi
```

The condition you are testing is placed between square brackets `[]`. When a test expression is evaluated, it returns either a value of 0, meaning that it is true, or a 1, meaning that it is false.

**Operators for Test Expressions**

| Operator | What is Being Tested? |
| -------- | --------------------- |
| `-a file` | Does the file exist? |
| `-b file` | Is the file a block-special device? |
| `-c file` | Is the file character special (for example, a character device)? Used to identify serial lines and terminal devices. |
| `-d file` | Is the file a directory? |
| `-e file` | Does the file exist? (same as -a) |
| `-f file` | Does the file exist, and is it a regular file (for example, not a directory, socket, pipe, link, or device file)? |
| `-g file` | Does the file have the set group id (SGID) bit set? |
| `-h file` | Is the file a symbolic link? (same as -L) |
| `-k file` | Does the file have the sticky bit set? |
| `-L file` | Is the file a symbolic link? |
| `-n string` | Is the length of the string greater than 0 bytes? |
| `-O file` | Do you own the file? |
| `-p file` | Is the file a named pipe? |
| `-r file` | Is the file readable by you? |
| `-s file` | Does the file exist, and is it larger than 0 bytes? |
| `-S file` | Does the file exist, and is it a socket? |
| `-t fd` | Is the file descriptor connected to a terminal? |
| `-u file` | Does the file have the set user id (SUID) bit set? |
| `-w file` | Is the file writable by you? |
| `-x file` | Is the file executable by you? |
| `-z string` | Is the length of the string 0 (zero) bytes? |
| `expr1 -a expr2` | Are both the first expression and the second expression true? |
| `expr1 -o expr2` | Is either of the two expressions true? |
| `file1 -nt file2` | Is the first file newer than the second file (using the modification time stamp)? |
| `file1 -ot file2` | Is the first file older than the second file (using the modification time stamp)? |
| `file1 -ef file2` | Are the two files associated by a link (a hard link or a symbolic link)? |
| `var1 = var2` | Is the first variable equal to the second variable? |
| `var1 -eq var2` | Is the first variable equal to the second variable? |
| `var1 -ge var2` | Is the first variable greater than or equal to the second variable? |
| `var1 -gt var2` | Is the first variable greater than the second variable? |
| `var1 -le var2` | Is the first variable less than or equal to the second variable? |
| `var1 -lt var2` | Is the first variable less than the second variable? |
| `var1 != var2` | Is the first variable not equal to the second variable? |
| `var1 -ne var2` | Is the first variable not equal to the second variable? |

Two pipes (||) indicate that if the first command doesn't succeed, run the second.
- `CMD1 || CMD2`

Two ampersands (&&) will run the second command if the first one succeeds.
- `CMD1 && CMD2`

&& and || can be combined to make one-line `if..then..else` statements.

```
# dirname=mydirectory
[ -e $dirname ] && echo $dirname already exists || mkdir $dirname
```

The `case` command is similar to the `switch` statement in other programming languages. `esac` is used to end the `case` statement.

```
case $(date + %a) in
  "Mon")
    BACKUP=/home/myproject/data0
    TAPE=/dev/rft0
    ;;
  "Tue" | "Thu")
    BACKUP=/home/myproject/data1
    TAPE=/dev/rft1
    ;;
  "Wed" | "Fri")
    BACKUP=/home/myproject/data2
    TAPE=/dev/rft2
    ;;
  *)
    BACKUP="none"
    TAPE=/dev/null
    ;;
esac
```

The asterisk (*) is used as a catchall, similar to the `default` keyword in the C programming language.

The `for..do` loop iterates through a list of values, executing the body of the loop for each element in the list.

```
for VAR in LIST
do
  { body }
done
```

Bash allows you to use C syntax to control your loops:

```
LIMIT=10
# Double parentheses, and no $ on LIMIT even though it's a variable!
for ((a=1; a <= LIMIT ; a++)); do
  echo "$a"
done
```

The `while..do` loop executes while the condition is true. The `until..do` loop executes until the condition is true, in other words, while the condition is false.

```
while condition
do
  { body }
done

until condition
do
  { body }
done
```

**Text manipulation in scripts**
The `cut` command can extract fields from a line of text or from files.

The `tr` command is a character-based translator that can be used to replace one character or set of characters with another or to remove a character from a line of text.

The `sed` command is a simple script-able editor, so it can perform only simple edits, such as removing lines that have text matching a certain pattern, replacing one pattern of characters with another, and so on.
