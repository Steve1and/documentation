# Linux Server Administration Concepts
***

## Understanding Server Administration

Many servers are represented by processes that run continuously in the background and respond to requests that come to them. These processes are referred to as *daemon* processes. These processes have special attributes:
- **User and group permissions**: Daemon processes often run as users and groups other than root.
- **Daemon configuration files**: Often, a service has a configuration file for the daemon stored in the `/etc/sysconfig` directory. This is different than the service configuration file in that its job is often just to pass arguments to the server process itself rather than configure the service.
- **Port Numbers**: Used to relate network traffic to specific processes.

Traditionally, Linux servers have been configured by editing plain-text files in the `/etc/` directory (or subdirectories). Often, there is a primary configuration file; sometimes, there is a related configuration directory in which files ending in `.conf` can be pulled into the main configuration file.

There are two major facilities for managing services: `systemd` (used now by RHEL, Ubuntu, and Fedora) and SystemVinit scripts (used by Red Hat Enterprise Linux through RHEL 6.x).

## Managing Remote Access with the Secure Shell Service

The *Secure Shell tools* are a set of client and server applications that allow you to do basic communications between client computers and your Linux server. With Secure Shell tools, both the authentication process and all communications that follow are encrypted.
- If you are using the Fedora or RHEL distribution, the client and server software packages that contain the `ssh` tools are `openssh`, `openssh-clients`, and `openssh-server` packages.
- On Ubuntu, only the `openssh-clients` package is installed. It includes the functionality of the `openssh` package. If you need the server installed, use the `sudo apt-get install openssh-server` command.
  - `sudo dpkg --list | grep openssh`
  - `sudo apt-get install openssh-server`

**Commands to Determine sshd Status**

| Distribution | Command to Determine sshd Status |
| ------------ | -------------------------------- |
| RHEL 6 | `chkconfig --list sshd` |
| Fedora and RHEL 7 or later | `systemctl status sshd.service` |
| Ubuntu | `systemctl status ssh.service` |

**Commands to Start sshd**

| Distribution | Command to Start sshd |
| ------------ | --------------------- |
| RHEL 6 | `service sshd start` |
| Fedora and RHEL 7 or later | `systemctl start sshd.service` |
| Ubuntu | `systemctl start ssh.service` |

**Commands to Start sshd at Boot**

| Distribution | Command to Start sshd at Boot |
| ------------ | ----------------------------- |
| RHEL 6 | `chkconfig sshd on` |
| Fedora and RHEL 7 or later | `systemctl enable sshd.service` |
| Ubuntu | `systemctl enable ssh.service ` |

When you install `openssh-server` on Ubuntu, the `sshd` daemon is configured to start automatically on boot.

Unless using another port, modify your firewall settings to allow the `openssh-client` to access port 22.

Any further configurations for what the `sshd` daemon is allowed to do are handled in the `/etc/ssh/sshd_config` file. At a minimum, set the `PermitRootLogin` setting to no. This stops anyone from remotely logging in as root.

After you have changed the `sshd_config` file, restart the `sshd` service.

### Using SSH client tools

Use the `ssh` command from another Linux computer to test that you can log in to the Linux system running the `sshd` service.
- `ssh johndoe@10.140.67.23`
- If this is the very first time that you have logged in to that remote system using the `ssh` command, the system asks you to confirm that you want to connect. When you type `yes` to continue, you accept the remote host's public key. At that point, the remote host's public key is downloaded to the client in the client's `~/.ssh/known_hosts` file. Now data exchanged between these two systems can be encrypted and decrypted using RSA asymmetric encryption.
- When you are finished, type `exit` to end the remote connection.

Your server's public and private keys are stored in the `/etc/ssh` directory.
- If the keys to an ssh server have changed, in order to be able to `ssh` to that address again, just remove the host's key (the whole line) from the `known_hosts` file and then you can copy over the new key.

**Using ssh for remote execution**: Besides logging into a remote shell, the `ssh` command can be used to execute a command on the remote system and have the output returned to the local system.
- `ssh johndoe@10.140.67.23 hostname`
- If you run a remote execution command with `ssh` that includes options or arguments, be sure to surround the whole remote command line in quotes. Keep in mind that if you refer to files or directories in  your remote commands, relative paths are interpreted in relation to the user's home directory.
  - `ssh johndoe@10.140.67.23 "cat myfile"`
- Another type of remote execution that you can do with `ssh` is X11 forwarding. If X11 forwarding is enabled on the server (`X11Forwarding yes` is set in the `/etc/sshd/sshd_config` file), you can run graphical applications from the server securely over the SSH connection using `ssh -X`.
  - `ssh -X johndoe@10.140.67.23 system-config-printer`
  - If you want to run several X commands and don't want to have to reconnect each time, you can use X11 forwarding directly from a remote shell as well.
    - `ssh -X johndoe@10.140.67.23`
    - `system-config-printer &`
    - `gedit &`

**Copying files between systems with scp and rsync**: The `scp` command is similar to the old UNIX `rcp` command for copying files to and from Linux systems, except that all communications are encrypted.
- `scp /home/chris/memo johndoe@10.140.67.23:/tmp`
- You can do recursive copies with `scp` using the `-r` option. Instead of a file, pass a directory name to the `scp` command and all files and directories below that point in the filesystem are copied to the other system.
  - `scp johndoe@10.140.67.23:/usr/share/man/man1/ /tmp/`
- The `rsync` command is a better network backup tool because it can overcome some of the shortcomings of `scp` such as retaining file attributes, keeping symbolic links, and only copy files that have changed.
  - `rsync -avl johndoe@10.140.67.23:/usr/share/man/man1/ /tmp/`
    - `-a`: recursive archive
    - `-v`: verbose
    - `-l`: copy symbolic links
- If you want the remote and local directories to be mirrored, you would have to add the `--delete` option.
  - `rsync -avl /usr/shre/man/man1 localhost:/tmp`
  - `rsync -avl --delete johndoe@10.140.67.23:/usr/share/man/man1 /tmp`

**Interactive copying with sftp**: If you don't know exactly what you want to copy to or from a remote system, you can use the `sftp` command to create an interactive FTP-style session over the SSH service.
- `sftp` has nothing to do with the FTP protocol and doesn't use FTP services. It simply uses an FTP style of interaction between a client and an `sshd` server.
- `sftp johndoe@jd.example.com`

### Using key-based (passwordless) authentication

SSH allows you to set up key-based authentication.
- You create a public key and a private key.
- You guard the private key but copy the public key across to the user account on the remote host to which you want to do key-based authentication.
- With your key copied to the proper location, you use any SSH tools to connect to the user account on the remote host, but instead of asking you for a password, the remote SSH service compares the public key and the private key and allows you access if the two keys match.

Type the following to generate a local public/private key pair:
- `ssh-keygen`
- A private key (`id_rsa`) and a public key (`id_rsa.pub`) are copied to the `.ssh` directory in your local home directory. The next step is to copy the public key over to a remote user to that you can use key-based authentication each time you connect to that user account with `ssh` tools:
  - `ssh-copy-id -i ~/.ssh/id_rsa.pub johndoe@10.140.67.23`
  - The public key belonging to the user that created it is copied to the `authorized_keys` file in `johndoe's` `.ssh` directory on the remote system.
- After you have the keys in place on your remote system for everyone you want to allow to log in to that system, you can set the `sshd` service on the remote system to not allow password authentication by changing the `PasswordAuthentication` setting in the `/etc/ssh/sshd_config` file to `no`.
- Then restart the `sshd` service (`systemctl restart sshd`). After that, anyone with a valid key is still accepted.

## Configuring System Logging

The `rsyslog` service (`rsyslogd` daemon) provides the features to gather log messages from software running on the Linux system and direct those messages to local log files, devices, or remote logging hosts.
- In recent Red Hat Enterprise Linux and Fedora releases, the `rsyslog` facility leverages messages that are gathered and stored in the `systemd` journal. To display journal log messages directly from the `systemd` journal, instead of viewing them from files in the `/var/log` directory, use the `journalctl` command.

### Enabling system logging with rsyslog

Most of the files in the `/var/log` directory are populated with log messages directed to them from the `rsyslog` service.

**Understanding the rsyslog.conf file**: The `/etc/rsyslog.conf` file is the primary configuration file for the `rsyslog` service.
- Entries beginning with `module(load=` load the modules that follow. Modules that are currently disabled are preceded by a pound sign (#). The `imjournal` module lets `rsyslog` access the `systemd` journal. The `imuxsock` module is needed to accept messages from the local system. The `imklog` module logs kernel messages.
- Modules not enabled by default include the `immark` module, which allows `--MARK--` messages to be logged (used to indicate that a service is alive).
- In Ubuntu, you need to look in the `/etc/rsyslog.d` directory for this configuration information.
- Rules entries come in two columns. In the left column are designations of what messages are matched; the right column shows where matched messages go. Messages are matched based on facility (`mail`, `cron`, `kern`, and so on) and priority (starting at `debug`, `info`, `notice` and up to `crit`, `alert`, and `emerg`), separated by a dot (`.`).

**Understanding the messages log file**: The default message format in the `/var/log/messages` file is divided into five main parts:
1. The date and time that the message was logged.
2. The name of the computer from which the message came.
3. The program or service name to which the message pertains.
4. The process number (enclosed in square brackets) of the program sending the message.
5. The actual text message.

**Setting up and using a loghost with rsyslogd**: To redirect your computer's log files to another computer's `rsyslogd`, you must make changes to both the local and remote `rsyslog` configuration file `/etc/rsyslog.conf`.
- **On the client side**: To send the messages to another computer (the loghost) instead of a file, start by replacing the log file name with the @ character followed by the name of the loghost.
  - The messages are now sent to the `rsyslogd` running on the computer named loghost.
- **On the loghost side**: The loghost that is set to accept the messages must listen for those messages on standard ports (514 UDP, although it can be configured to accept messages on 514 TCP as well).
  - Edit the `/etc/rsyslog.conf` file on the loghost system and uncomment the lines that enable the `rsyslogd` daemon to listen for remote log messages. Uncomment the first two lines to enable incoming UDP log messages on port 514 (default); uncomment the two lines after that to allow messages that use TCP protocol (also port 514);
  ```
  module(load="imudp") # needs to be done just once
  input(type="imudp" port="514")
  module(load="imtcp") # needs to be done just once
  input(type="imtcp" port="514")
  ```
  - Open your firewall to allow new messages to be directed to your loghost.
  - Restart the `rsyslog` service (`service rsyslog restart` or `systemctl restart rsyslog.service`).

### Watching logs with logwatch

The `logwatch` service runs in most Linux systems that do system logging with `rsyslog`. To install the `logwatch` facility, enter the following:
- `yum install logwatch`

What `logwatch` does is gather messages once each night that look like they might represent a problem, put them in an email message, and send it to any email address the administrator chooses. The `logwatch` service runs from a `cron` job (`0logwatch`) placed in `/etc/cron.daily`. The `/etc/logwatch/conf/logwatch.conf` file holds local settings. The default options used to gather log messages are set in the `/usr/share/logwatch/default.conf/log-watch.conf` file.

## Checking System Resources with sar

The System Activity Reporter (`sar`) is one of the oldest system monitoring facilities created for early Unix systems -- predating Linux by a few decades. The `sar` command itself can display system activity continuously, at set intervals (every second or two), and display it on the screen. It can also display system activity data that was gathered earlier.
- The `sar` command is part of the `sysstat` package.
  - `systemctl enable systat`
  - `systemctl start systat`
- To read the data in the `/var/log/sa/sa??` files.
  - `sar -u | less`
  - The `-u` option shows CPU usage.
  - The output continues to show the activity every 10 minutes until the current time is reached.
- To see disk activity output, run the `sar -d` command.
  - `sar -d | less`
- If you want to run `sar` activity reports live, you can do that by adding counts and time intervals to the command line:
  - `sar -n DEV 5 2`
  - `-n DEV`: samplings of data were taken every 5 seconds and repeated twice.

## Checking System Space

### Displaying system space with df

You can display the space available in your filesystems using the `df` command.

To produce output in a more human-readable form, use the `-h` option:
- `df -h`

`df` enables you to do the following:
- Print only filesystems of a particular type (`-t type`).
- Exclude filesystems of a particular type (`-x type`).
- Include filesystems that have no space, such as `/proc` and `/dev/pts` (`-a`).
- List only available and used inodes (`-i`).
- Display disk space in certain block sizes (`--block-size=#`).

### Checking disk usage with du

To find out how much space is being consumed by a particular directory (and its subdirectories), use the `du` command. With no options, `du` lists all directories below the current directory, along with the space consumed by each directory. At the end, `du` produces total disk space used within that directory structure.
- By default, disk space is displayed in 1KB block sizes. To make the output friendlier (in kilobytes, megabytes, and gigabytes), use the `-h` option.
- `du -h /home/jake`
***

## Administering Networking

A computer must have a network interface (wired or wireless), an IP address, and assigned DNS server, and a route to the Internet (identified by a gateway device), in order to fully communicate on a network.

**Useful Commands**
- `ip addr show`: See information about each network interface on a local Linux system.
  - Add a `-s` option (`ip -s addr show`) to see statistics of packet transmissions and errors associated with each interface.
- `ifconfig <interface>`: shows similar information to `ip addr`, but also shows the number of packets received (RX) and transmitted (TX) by default, as well as the amount of data and any errors or dropped packets.
- `ping <host>`: Determine if a system can be reached on a network.
  - For Linux, remember to press `Ctrl + C` to end the `ping`.
- `ip route show`: Display routing information.
- `route`: provides similar information to `ip route show`.
  - `route -n`: Display routing information with only numerical addresses.
- `traceroute <host>`: Follow the entire route to a host from beginning to end.
- `hostname`: Display the hostname assigned to a local system.
- `dnsdomainname`: Display the domain portion of a hostname assigned to a local system.

The `nmtui` command (`yum install NetworkManager-tui`) provides a menu-based interface that runs in the shell.

### Network Configuration Files

In Fedora and RHEL, network interfaces and custom routes are set in files in the `/etc/sysconfig/network-scripts` directory.
- Open the `/usr/share/doc/initscripts/sysconfig.txt` file for descriptions of network-scripts configuration files (available from the `initscripts` package).
- One thing to be careful about is that NetworkManager believes that it controls the files in the network-scripts directory. So keep in mind that if you set manual addresses on an interface that NetworkManager has configured for DHCP, it could overwrite changes that you made manually to the file.

Configuration files for each wired, wireless, ISDN, dialup, or other type of network interface are represented by files in the `/etc/sysconfig/network-scripts` directory that begin with `ifcfg-interface`. Note that *interface* is replaced by the name of the network interface.

Here's an example of an `ifcfg-enp4s0` file for an interface configured to use DHCP:
```
DEVICE=enp4s0
TYPE=Ethernet
BOOTPROTO=dhcp
ONBOOT=yes
DEFROUTE=yes
UUID=f16259c2-f350-4d78-a539-604c3f95998c
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
NAME="System enp4s0"
PEERDNS=yes
PEERROUTES=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
```
In this `ifcfg-enp4s0` file, the first two lines set the device name and the type of interface to Ethernet. The `BOOTPROTO` variable is set to `dhcp`, which causes it to request address information from a DHCP server. With `ONBOOT=yes`, the interface starts automatically at system boot time. IPV6 settings say to initialize IPV6 and use the IPV6 settings that are presented, but the interface will continue to initialize if there is no IPV6 network available. Other settings say to use peer DNS automatically and route values that are detected.

Here's what a simple `ifcfg-enp4s1` file might look like for a wired Ethernet interface that uses static IP addresses:
```
DEVICE=enp4s1
HWADDR=00:1B:21:0A:E8:5E
TYPE=Ethernet
BOOTPROTO=none
ONBOOT=yes
USERCTL=no
IPADDR=192.168.0.140
NETMASK=255.255.255.0
GATEWAY=192.168.0.1
```
In this `ifcfg-enp4s1` file, because this is setting the address and other information statically, `BOOTPROTO` is set to `none`. Other differences are needed to set the address information that is normally gathered from a DHCP server. In this case, the IP address is set to 192.168.0.140 with a netmask of 255.255.255.0. The `GATEWAY=192.168.0.1` identifies the address of the router to the internet. Other interesting options:
- `PEERDNS`: Setting `PEERDNS=no` prevents DHCP from overwriting the `/etc/resolv.conf` file.
- `DNS?`: If an `ifcfg` file is being managed by NetworkManager, it sets the address of DNS servers using DNS? entries. For example, `DNS1=192.168.0.2` causes that IP address to be written to `/etc/resolv.conf` as the first DNS server being used on the system. You can have multiple DNS? entries (`DNS2=`, `DNS3=`, and so on).

**Other networking files**:
- `/etc/sysconfig/network`: System-wide settings associated with your local networking can be included in this file.
  - `GATEWAY=192.168.0.1`
- `/etc/hostname`: In RHEL and Fedora releases, the system's hostname is stored in the `/etc/hostname` file.
- `/etc/hosts`: Before DNS was created, translating to IP addresses was done by passing around a single hosts file. The `/etc/hosts` file is a way to set up names and addresses for a small local network or just create aliases in order to make it easier to access the systems that you use all the time.
- `/etc/resolv.conf`: DNS servers and search domains are set in the `/etc/resolv.conf` file. Each nameserver entry identifies the IP address of a DNS server. Another type of entry that you can add to this file is a search entry. A *search entry* lets you indicate domains to be searched when a hostname is requested by its base name instead of its entire fully qualified domain name.
  - `search example.com example.org example.net`
- `/etc/nsswitch.conf`: Unlike in earlier releases, the `/etc/nsswitch.conf` file is managed by the `authselect` command and should not be modified manually. To make changes, edit the `/etc/authselect/user-nsswitch.conf` file and run `authselect apply-changes`.
  - Settings in the `/etc/nsswitch.conf` file determine that hostname resolution is done by first searching the local `/etc/hosts` file (files) and then DNS servers listed in the `/etc/resolv.conf` file (`dns`). The `myhostname` value is used to ensure that an address is always returned for the host: `hosts:   files dns myhostname`.
  - You can add other locations, such as Network Information Service (`nis` or `nisplus`) databases, for querying hostname-to-IP-address resolution. You can also change the order in which the different services are queried.
  - If you want to check that your DNS servers are being queried properly, you can use the `host` or `dig` commands.
  - The `host` command produces simpler output for DNS queries.
  - The `dig` command shows information similar to what appears in the files that hold DNS records.
  - The `getent` command can be used to query any information setup in the `nsswitch.conf` file.

### Setting alias network interfaces

Network interface cards can listen on multiple IP addresses which are managed by creating alias files. To create an alias network interface in RHEL 6 and earlier Fedora releases, you just have to create another `ifcfg-` file with an interface name followed by `:#` where the number is a number from 0 up, (`ifcfg-eth0:0`). Place the file in the `/etc/sysconfig/network-scripts/` directory with the following contents:
```
DEVICE=eth0:0
ONPARENT=yes
IPADDR=192.168.0.141
NETMASK=255.255.255.0
```
This file creates an alias for the network interface `eth0` called `eth0:0`. Instead of `ONBOOT`, the `ONPARENT` entry says to bring up this interface if the parent (`eth0`) is started and listen on the address 192.168.0.141. You can add more IP addresses to that interface by creating more `ifcfg-eth0:?` files (`ifcfg-eth0:1`, `ifcfg-eth0:2`, and so on).

### Setting up Ethernet channel bonding

*Ethernet channel bonding* allows you to have more than one network interface card on a computer associated with a single IP address. In RHEL and Fedora on a computer with multiple NICs, you can set up Ethernet channel bonding by creating a few `ifcfg` files and loading the necessary module. You can start with one bonding file (for example, `ifcfg-bond0`) and then point multiple `ifcfg-eth?` files at that bond interface. Then you can load the bond module.
- Using the `BONDING_OPTS` variable, you define the mode and other bonding options (all of which are passed to the bonding module). You can read about the bonding module by entering `modinfo bonding`.
- `/etc/sysconfig/network-scripts/ifcfg-bond0`:
```
DEVICE=bond0
ONBOOT=yes
IPADDR=192.168.0.50
NETMASK=255.255.255.0
BOOTPROTO=none
BONDING_OPTS="mode=active-backup"
```
The `BONDING_OPTS` sets the bonding mode to active-backup. This means that only one NIC is active at a time, and the next NIC only takes over when the previous one falls (failover).
- For each slave device, use a file such as (`/etc/sysconfig/network-scripts/ifcfg-eth0`):
```
DEVICE=eth0
MASTER=bond0
SLAVE=yes
BOOTPROTO=none
ONBOOT=yes
```
With the `eth0` interface used as part of the `bond0` interface, there is no IP address assigned. That's because the `eth0` interface uses the IP address from the `bond0` interface by defining itself as a slave (`SLAVE=yes`) to `bond0` (`MASTER=bond0`). The last thing that you want to do is to make sure that the `bond0` interface is set to use the bonding module. To do that, create an `/etc/modeprobe.d/bonding.conf` file that contains the following entry:
- `alias bond0 bonding`

### Setting custom routes

To set a custom route in Fedora and RHEL, you create a configuration file in the `/etc/sysconfig/network-scripts` directory. In that route, you define the following:
- **GATEWAY?**: The IP address of the node on the local network that provides the route to the subnetwork represented by the static route.
- **ADDRESS?**: The IP address representing the network that can be reached by the static route.
- **NETMASK?**: The netmask that determines which part of the `ADDRESS?` represents the network and which represents the hosts that can be reached on that network.
The name of each custom route file is `route-interface`. So, for example, a custom route that can be reached through your `eth0` interface would be named `route-eth0`. It would contain contents like:
```
ADDRESS0=192.168.99.0
NETMASK0=255.255.255.0
GATEWAY=192.168.0.5
```
This route would take effect when the `eth0` network interface was restarted. If you wanted to add more custom routes, you could add them to this same `route-eth0` file. The next set of information would be named `ADDRESS1`, `NETMASK1`, `GATEWAY1`, and so on.

### Configuring Linux as a router

If you have more than one network interface on a computer (typically two more more NICs), you can configure Linux as a router. To make this happen, all that is needed is a change to one kernel parameter that allows packet forwarding.
- `cat /proc/sys/net/ipv4/ip_forward`
- `echo 1 > /proc/sys/net/ipv4/ip_forward`
- `cat /proc/sys/net/ipv4/ip_forward`
Packet forwarding (routing) is disabled by default, with the value of `ip_forward` set to 0. By setting it to 1, packet forwarding is immediately enabled. To make this change permanent, you must add that value to the `/etc/sysctl.conf` file, so that it appears as follows:
- `net.ipv4.ip_forward=1`
With that file modified as shown, each time the system reboots, the value for `ip_forward` is reset to 1.

### Configuring Linux as a DHCP server

DHCP service is provided by the `dhcp-server` package in Fedora and RHEL. The service is named `dhcpd`. The primary configuration file is `/etc/dhcp/dhcpd.conf` for IPv4 networks (there is a `dhcpd6.conf` file in the same directory to provide DHCP service for IPv6 networks). By default, the `dhcpd` daemon listens on UDP port 67. To configure a DHCP server, you could copy the `dhcpd.conf.example` file from the `/usr/share/doc/dhcp-server` directoryh and replace the `/etc/dhcp/dhcpd.conf` file. Then modify it as you like.

### Configuring Linux as a DNS server.

In Linux, most professional Domain Name System (DNS) servers are implemented using the Berkeley Internet Name Domain (BIND) service. This is implemented in Fedora and RHEL by installing the `bind`, `bind-utils`, and `bind-libs` packages. For added security, some people install the `bind-chroot` package. By default, `bind` is configured by editing the `/etc/named.conf` file. Hostname-to-IP-address mapping is done in zone files located in the `/var/named` directory. If you install the `bind-chroot` package, `bind` configuration files are moved under the `/var/named/chroot` directory, which attempts to replicate the files from `/etc/` and `/var` that are needed to configure `bind` so that the named daemon (which provides the service) is confined to the `/etc/named/chroot` directory structure.

### Configuring Linux as a proxy server

A proxy server provides a means of restricting network traffic from a private network to a public one, such as the Internet.
- A proxy server that supports SOCKS can provide a proxy service for different protocols outside of the local network. (*SOCKS* is a network protocol made to allow client computers to access the Internet through a firewall).
  - The port associated with the proxy service is, by default, 3128.

By physically setting up Linux as a router but configuring it as a proxy server, all of the systems on your home or business network can be configured to access the Internet using only certain protocols and only after you filter the traffic.

Using the Squid Proxy Server, which comes with most Linux systems (`squid` package in Fedora and RHEL), you can enable the system to accept requests to web servers (HTTP and HTTPS), file servers (FTP) and other protocols. Configuring a squid proxy server can be as simple as installing the `squid` package, editing the `/etc/squid/squid.conf` file, and starting the `squid` service.

To get the Squid proxy running in its basic form, do the following:
1. Install the package:
  - `sudo apt install squid`
2. Enable the service:
  - `sudo systemctl enable squid`
3. Copy the squid configuration file:
  - `sudo cp /etc/squid/squid.conf{,.orginal}`
4. Create a file to hold all the allowed IPs:
  - `sudo vim /etc/squid/allowed_ips.txt`
  - `192.168.0.9` (as many as will be using the web proxy, one on each line)
5. Edit the squid configuration file and add a new ACL. Then add a line to only allow the IP's listed in the file you made in the previous step:
  - `acl allowed_ips src "/etc/squid/allowed_ips.txt"` (should be placed in the section of the file with the other `acl` lines)
  - `http_access allow allowed_ips` (place below the line `http_access allow localhost`)
6. Restart Squid:
  - `sudo systemctl restart squid`
7. Remember to set the appropriate host machines to identify the proxy box as their preferred proxy.

- **Reference**: https://linuxize.com/post/how-to-install-and-configure-squid-proxy-on-ubuntu-20-04/

To allow the `apt` service to use a web proxy, perform the following steps on the host needing to update:
1. Create a file named `proxy.conf` in the `apt` config directory:
  - `sudo vim /etc/apt/apt.conf.d/proxy.conf`
2. Add the following to the file:
```
Acquire {
  HTTP::proxy "http://<proxy-address>:3128";
  HTTPS::proxy "http://<proxy-address>:3128"
}
```
3. Save the file and do and update to verify that it worked:
  - `sudo apt update`
***

## Starting and Stopping Services

Ongoing services offered by a Linux system, such as access to a printer service or login service, are typically implemented by what is referred to as a *daemon* process. Most Linux systems have a method of managing each daemon process as a *service* using one of the several popular initialization systems (also referred to as init systems).

### Understanding the Initialization Daemon (init or systemd)

The initialization daemon can be thought of as the "mother of all processes". This daemon is the first process to be started by the kernel on the Linux server. For Linux distributions that use SysVinit, the init daemon is literally named `init`. For `systemd`, the init daemon is named `systemd`.

The Linux kernel has a process ID (PID) of 0. Thus, the initialization process (`init` or `systemd`) daemon has a parent process ID (PPID) of 0, and a PID of 1. Once started, `init` is a responsible for spawning (launching) processes configured to be started at the server's boot time, such as the login shell (`getty` or `mingetty` process). It is also responsible for managing services.

#### SysVinit and BSD init process

Both the SysVinit daemon and the BSD `init` daemon's configuration information is taken at boot time from the `/etc/inittab` file. Below are the standard Linux runlevels:

| RunLevel # | Name | Description |
| ---------- | ---- | ----------- |
| 0 | Halt | All services are shut down, and the server is stopped. |
| 1 or S | Single User Mode | The root account is automatically logged in to the server. Other users cannot log in to the server. Only the command-line interface is available. Network services are not started. |
| 2 | Multiuser Mode | Users can log in to the server, but only the command-line interface is available. On some systems, network interfaces and services are started; on others they are not. Originally, this runlevel was used to start dumb terminal devices so that users could log in (but no network services were started). |
| 3 | Extended Multiuser Mode | Users can log in to the server, but only the command-line interface is available. Network interfaces and services are started. This is a common runlevel for servers. |
| 4 | User Defined | Users can customize this runlevel. |
| 5 | Graphical Mode | Users can log in to the server. Command-line and graphical interfaces are available. Network services are started. This is a common runlevel for desktop systems. |
| 6 | Reboot | The server is rebooted. |

The `/etc/inittab` file tells the `init` daemon which runlevel is the default runlevel. A *runlevel* is a categorization number that determines what services are started and what services are stopped.

The `init` command accepts any of the runlevel numbers 0-6, allowing you to switch your server quickly from one runlevel category to another. To see your Linux server's current runlevel, simply type in the command `runlevel`. The first item displayed is the server's previous runlevel. The second item displayed shows the server's current runlevel. The letter N stands for "Nonexistent" and indicates that the server was freshly booted to the current runlevel.
```
runlevel
3 5
```

In addition to the `init` command, you can use the `telinit` command, which is functionally the same.
- `telinit 6`

When a runlevel is chosen, the scripts located in the `/etc/rc.d/rc#.d` directory (where # is the chosen runlevel) are run.
- `ls /etc/rc.d/rc5.d`
- Some scripts located in the `rc#.d` directories start with a K and some start with an S. The K refers to a script that will kill (stop) a process. The S refers to a script that will start a process. Also, each K and S script has a number before the name of the service or daemon that they control. This allows the services to be stopped or started in a particular controlled order. The files in these directories are also symbolic links to scripts in the `/etc/rc.d/init.d` directory.
- `ls /etc/rc.d/init.d`
- Each script in `/etc/rc.d/init.d` handles starting, stopping, restarting, and displaying the status of a service.

After the runlevel scripts linked from the appropriate `/etc/rc.d/rc#.d` directory are executed, the SysVinit daemon's process spawning is complete. The final step the `init` process takes at this point is to do anything else indicated in the `/etc/inittab` file (such as spawn `mingetty` processes for virtual consoles and start the desktop interface, if you are in runlevel 5).

#### systemd initialization

System initialization time is reduced by `systemd` because it can start services in parallel. With the SysVinit daemon, services are stopped and started based upon runlevels. The `systemd` service is concerned with runlevels, but it implements them in a different way with what are called *target units*. Although the main job of `systemd` is to start and stop services, it can manage other types of things referred to as units. A *unit* is a group consisting of a name, type, and configuration file, and it is focused on a particular service or action. There are 12 systemd unit types:
- `automount`
- `device`
- `mount`
- `path`
- `service`
- `snapshot`
- `socket`
- `target`
- `timer`
- `swap`
- `slice`
- `scope`

A *service unit* is for managing daemons on your Linux server. A *target unit* is simply a group of other units. Each service unit name ends with `.service`. The target units shown have names like `sysinit`. (`sysinit` is used for starting up services at system initialization). The target unit names end with `.target`.
- `systemctl list-units | grep .service`
- `systemctl list-units | grep .target`
The Linux system unit configuration files are located in the `/lib/systemd/system` and `/etc/systemd/system` directories.
- `systemctl list-unit-files --type=service`
The unit configuration files are all associated with a service unit. Configuration files for target units can be displayed via the following method:
- `systemctl list-unit-files --type=target`
Units and Targets have statuses of static, enabled, and disabled. The enabled status means that the unit is currently enabled. The disabled status means that the unit is currently disabled. The next status, static, is slightly confusing. It stands for "statically enabled", and it means that the unit is enabled by default and cannot be disabled, even by root.

The service unit configuration files contain lots of information, such as what other services must be started, when this service can be started, which environmental file to use, and so on.
- `cat /lib/systemd/system/sshd.service`

The basic service unit configuration file has the following options:
- **Description**: A free-form description (comment line) of the service.
- **Documentation**: Lists the man pages for the `sshd` daemon and configuration file.
- **After**: Configures ordering. In other words, it lists which units should be activated before this service is started.
- **Environment File**: The service's configuration files.
- **ExecStart**: The command used to start this service.
- **ExecReload**: The command used to reload this service.
- **WantedBy**: The target unit to which this service belongs.

You can view the various units that a target unit will active by using the following command:
- `systemctl show --property "Wants" multi-user.target`
- `systemctl show --property "Wants" multi-user.target | fmt -10 | sed 's/Wants=//g' | sort`

A target unit has both *Wants* and requirements, called *Requires*. A *Wants* means that all of the units listed are triggered to activate (start). If they fail or cannot be started, no problem -- the target unit continues on its merry way. A *Requires* means that all of the units listed are triggered to activate (start). If they fail or cannot be started, the entire unit (group of units) is deactivated.
- `systemctl show --property "Requires" multi-user.target`
- The `target` units also have configuration files, as do the `service` units.
- `cat /lib/systemd/system/multi-user.target`

The `multi-user.target` basic target unit configuration file has the following options:
- **Description**: This is just a free-form description of the target.
- **Documentation**: Lists the appropriate systemd man page.
- **Requires**: If this `multi-user.target` gets activated, the listed target unit is also activated. If the listed target unit is deactivated or fails, then `multi-user.target` is deactivated. If there are no After and Before options, then both `multi-user.target` and the listed target unit activate simultaneously.
- **Conflicts**: This setting avoids conflicts in services. Starting `multi-user.target` stops the listed targets and services, and vice versa.
- **After**: This setting configures ordering. In other words, it determines which units should be activated before starting this service.
- **AllowIsolate**: This option is a Boolean setting of `yes` or `no`. If this option is set to `yes`, then this target unit, `multi-user.target`, is activated along with its dependencies and all others are deactivated.

At boot, `systemd` activates the `default.target` unit. This unit is aliased either to `multi-user.target` or `graphical.target`.

There are seven target unit configuration files specifically created for backward compatibility to SysVinit:
- `runlevel0.target`
- `runlevel1.target`
- `runlevel2.target`
- `runlevel3.target`
- `runlevel4.target`
- `runlevel5.target`
- `runlevel6.target`
These target unit configuration files are symbolically linked to target unit configuration files that most closely match the idea of the original runlevel.
- `ls -l /lib/systemd/system/runlevel*.target`

### Checking the Status of Services

#### Checking services for SysVinit systems

To see all of the services that are being offered by a Linux server using the classic SysVinit daemon, us the `chkconfig` command. Note that each runlevel (0-6) is shown for each service with a status of on or off.
- `chkconfig --list`

Using the `chkconfig` command, you cannot tell if a service is currently running. To do that, you need to use the `service` command.
- `service --status-all | grep running... | sort`

You can also use both the `chkconfig` and the `service` commands to view an individual service's settings.
- `chkconfig --list cups`
- `service cups status`

#### Checking services for systemd systems

To see all of the services that are being offered by a Linux server using `systemd`, use the following command:
- `systemctl list-unit-files --type=service | grep -v disabled`

To see if a particular service is running, use the following command:
- `systemctl status cups.service`

### Stopping and Starting Services

#### Stopping and Starting SysVinit services

The primary command for stopping and starting SysVinit services is the `service` command. With the `service` command, the name of the service that you want to control comes second in the command line. The last option is what you want to do to the service: `stop`, `start` `restart`, and so on.
- `service cups status`
- `service cups stop`
- `service cups status`

To start a service, you simply use a `start` option instead of a `stop` option on the end of the `service` command.
- `service cups start`
- `service cups status`

To restart a SysVinit service, the `restart` option is used.
- `service cups restart`
- `service cups status`
- When a service is already stopped, a `restart` generates a FAILED status but will start it up.

When you `reload` a service, the service itself is not stopped. Only the service's configuration files are loaded again.
- `service cups reload`
- `service cups status`
- Running the `reload` option on a stopped service will result in a FAILED message.

#### Stopping and Starting systemd services

For the systemd daemon, the `systemctl` command works for stopping, starting, reloading, and restarting services.
- `systemctl start cups.service`
- `systemctl stop cups.service`
- `systemctl status cups.service`
- `systemctl restart cups.service`

You can also perform a conditional restart of a service using `systemctl`. A conditional restart only restarts a service if it is currently running. Any service in an inactive state is not started.
- `systemctl condrestart cups.service`
- `systemctl status cups.service`

Just like SysVinit, you can `reload` a systemd service.
- `systemctl reload sshd.service`
- `systemctl status sshd.service`
- Doing a `reload` of a service, instead of a `restart` prevents any pending service operations from being aborted. A `reload` is a better method for a busy Linux server.

### Enabling Persistent Services

A *persistent service* is one that is started at server boot time or at a particular runlevel.

#### Configuring persistent services for SysVinit

To make a service persistent at a particular runlevel, the `chkconfig` command is used again. Instead of the `--list` option, the `--level` option is used, as shown in the following code:
- `chkconfig --level 3 cups on`
- `chkconfig --list cups`
- `ls /etc/rc.d/rc3.d/S*cups`

To make a service persistent on more than one runlevel, you can do the following:
- `chkconfig --level 2345 cups on`
- `chkconfig --list cups`
- `ls /etc/rc.d/rc?.d/S*cups`

Disabling a service is just as easy as enabling one with SysVinit. You just need to change the `on` in the `chkconfig` command to `off`.
- `chkconfig --level 5 cups off`
- `chkconfig --list cups`
- `ls /etc/rc.d/rc5.d/S*cups`

For the `systemd` daemon, again the `systemctl` command is used.

#### Configuring persistent services for systemd

Using the `enable` option on the `systemctl` command sets a service to always start at boot (be persistent).
- `systemctl status cups.service`
- `systemctl enable cups.service`
- `systemctl status cups.service`

You can use the `disable` option on the `systemctl` command to keep a service from starting at boot.
- `systemctl disable cups.service`
- `systemctl status cups.service`

When the `systemctl disable` command is used on `dbus.service`, it is simply ignored due to the service being static.

To disable a service in a way that prevents it from ever running on your system, you can use the `mask` option.
- `systemctl mask NetworkManager.service`
- As the output shows, the `NetworkManager.service` file in `/etc` is linked to `/dev/null`. So even if someone tried to run that service, nothing would happen. To be able to use the service again, you could type `systemctl unmask NetworkManager.service`.

### Configuring a Default Runlevel or Target Unit

#### Configuring the SysVinit default runlevel

You set the persistent runlevel for a Linux server using SysVinit in the `/etc/inittab` file. The `initdefault` line in the file holds the current default runlevel. This can be changed to any other, (recommend only 2, 3, 4, or 5 as 0 or 6 will keep a computer from starting properly).

#### Configuring the systemd default runlevel

The persistent target unit is set via a symbolic link to the `default.target` unit file.
- `ls -l /etc/systemd/system/default.target`
- `ls -l /lib/systemd/system/runlevel5.target`

The following `systemctl` example changes the server's persistent target unit from `graphical.target` to `multi-user.target`:
- `systemctl get-default`
- `systemctl set-default runlevel3.target`
- `systemctl get-default`

### Adding New or Customized Services

#### Adding new services to SysVinit

When adding a new or customized service to a Linux SysVinit server, you must complete three steps in order to have the service managed by SysVinit:
1. Create a new or customized service script file.
2. Move the new or customized service script to the proper location for SysVinit management.
3. Set appropriate permission on the script.
4. Add the service to a specific runlevel.

**Step 1: Create a new or customized service script file**
- If you are customizing a service script, simply make a copy of the original unit file from `/etc/rc.d/init.d` and add any desired customizations. If you are creating a new script, you need to make sure you handle all of the various options that you want the `service` command to accept for your service, such as `start`, `stop`, `restart`, and so on.
- `cat /etc/rc.d/init.d/cups`
- One line you should be sure to check and possibly modify in your new script is the `chkconfig` line that is commented out: for example:
  - `chkconfig 2345 25 10`
- When you add the service script in a later step, the `chkconfig` command reads that line to set runlevels at which the service starts (2, 3, 4, and 5), its run order when the script is set to start (25), and its kill order when it is set to stop (10).

**Step 2: Add the service script to /etc/rc.d/init.d**
- You can change the `chkconfig` line in the service script if you want the service to start earlier (use a smaller number) or later (use a larger number) in the list of service scripts.
  - `cp My_New_Service /etc/rc.d/init.d`

**Step 3: Set appropriate permission on the script**
- The script should be executable:
  - `chmod 755 /etc/rc.d/init.d/My_New_Service`

**Step 4: Add the service to runlevel directories**
1. To add the script baed on the `chkconfig` line in the service script, type the following:
  - `chkconfig --add My_New_Service`
  - `ls /etc/rc?.d/*My_New_Service`
2. After you have made the symbolic link(s), test that your new or modified service works as expected before performing a server reboot.
  - `service My_New_Service start`
  - `service My_New_Service stop`

#### Adding new services to systemd

When adding a new or customized service to a Linux `systemd` server, you have to complete three steps in order to have the service managed by `systemd`:
1. Create a new or customized service configuration unit file for the new or customized service.
2. Move the new or customized service configuration unit file to the proper location for `systemd` management.
3. Add the service to a specific target unit's Wants to have the new or customized service start automiatically with other services.

**Step 1: Create a new or customized service configuration unit file.**
- If you are customizing a service configuration unit file, simply make a copy of the original unit file from `/lib/systemd/system` and add any desired customizations.
- For new files, at a bare minimum, you need `Description` and `ExecStart` options for a service unit configuration file:
```
[Unit]
Description=My New Service
[Service]
ExecStart=/usr/bin/My_New_Service
```

**Step 2: Move the service configuration unit file**
- Before you move the new or customized service configuration unit file, you need to be aware that there are two potential locations to store service configuration unit files. The one you choose determines whether the customizations take effect and if they remain persistent through software upgrades. You can place your system service configuration unit file in one of the following two locations:
  - `/etc/systemd/system`
    - This location is used to store customized local service configuration unit files.
    - Files in this location are not overwritten by software installations or upgrades. Files here are used by the system even if there is a file of the same name in the `/lib/systemd/system` directory.
  - `/lib/systemd/system`
    - This location is used to store system service configuration unit files.
    - Files in this location are overwritten by software installations and upgrades.
- Files here are used by the system only if there is no file of the same name in the `/etc/systemd/system` directory.
- When you create a new or customized servicehttps://gist.github.com/jeanlescure/084dd6113931ea5a0fd9, in order for the change to take effect without a server reboot, you need to issue a special command. At the command line, type `systemctl daemon-reload`.

**Step 3: Add the service to the Wants directory**
- This final step is optional. It needs to be done only if you want your new service to start with a particular `systemd` target unit. For a service to be activated (started) by a particular target unit, it must be in that target unit's `Wants` directory.
- First, add the line `WantedBy=desired.target` to the bottom of your service configuration unit file, where `desired.target` is the target level that you want to use the service.
- To add a new service unit to a target unit, you need to create a symbolic link.
  - `ls /etc/systemct/system/multi-user.target.wants`
  - `ls -l /etc/systemd/system/multi-user.target.wants`
- The following illustrates the process of adding a symbolic link file for `My_New_Service`:
  - `ln -s /etc/systemd/system/My_New_Service.service /etc/systemd/system/multi-user.target.wants/My_New_Service.service`
- If you want to change the `systemd` target unit for a service, you need to change the symbol link to point to a new target `Wants` directory location. Use the `ln -sf` command to force any current symbolic link to be broken and the new designated symbolic link to be enforced.
***

## Configuring a Print Server

You can configure a printer as a native Linux printer in Fedora, RHEL, Ubuntu, and other Linux systems with the *Common UNIX Printing System (CUPS)*. To configure a printer to work as a Microsoft Windows style of print server, you can use the Samba service in Linux. CUPS has the following features:
- **IPP**: CUPS is based on the Internet Printing Protocol (`http://www.pwg.org/ipp`), a standard that was created to simplify how printers can be shared over IP networks. In the IPP model, printer servers and clients who want to print can exchange information about the model and features of a printer using the HTTP (that is, web content) protocol. A server can also broadcast the availability of a printer so that a printing client can easily find a list of locally available printers without configuration.
- **Drivers**: CUPS also standardized how printer drivers are created. That way, a manufacturer has only has to create a driver once to work for Linux, Mac OS X, and a variety of UNIX derivatives.
- **Printer Classes**: You can use printer classes to create multiple printer server entries that point to the same printer or one print server entry that points to multiple printers. In the first case, multiple entries can each allow different options (such as pointing to a particular paper tray or printing with certain character sizes or margins). In the second case, you can have a pool of printers so that printing is distributed. In this instance, a malfunctioning printer, or a printer that is dealing with very large documents, won't bring all printing to a halt. CUPS also supports *implicit classes*, which are print classes that form by merging identical network printers automatically.
- **Printer Browsing**: With printer browsing, client computers can see any CUPS printers on your local network with browsing enabled.

**Printing Directly from Windows to CUPS**: You can use a PostScript printer driver to print directly from a Windows system to print directly from a Windows system to your CUPS server. You can use CUPS without modification by configuring the Windows computer with a PostScript driver that uses http://printservername:631/printers/targetPrinter as its printing port.

To install CUPS on RHEL based systems, run the following command:
- `yum install cups cups-client`

To install the Print Settings tool on RHEL based systems, run the following command:
- `yum install system-config-printer`

### Configuring the CUPS Server (cupsd.conf)

The `cupsd` daemon process listens for requests to your CUPS print server and responds to those requests based on settings in the `/etc/cups/cupsd.conf` file.

No classification is set by default. With the classification set to `topsecret`, you can have Top Secret displayed on all pages that go through the print server:
- `Classification topsecret`
- Other classifications you can substitute for `topsecret` include `classified`, `confidential`, `secret`, and `unclassified`.

The term *browsing* refers to the act of broadcasting information about your printer on your local network and listening for other print servers; information. The `cups-browsed` setting is used to brose shared, remote printers. Browsing is on by default for all local networks (`@LOCAL`). You can allow CUPS browser information (`BrowseAllow`) for additional selected addresses. Browsing information is broadcast, by default, on address `255.255.255.255`. Here are examples of several browsing settings:
```
Browsing On
BrowseProtocols cups
BrowseOrder Deny,Allow
BrowseAllow from @LOCAL
BrowseAddress 255.255.255.255
Listen *:631
```
To enable web-based CUPS administration and to share printers with others on the network, the `cupsd` daemon can be set to listen on port 631 for all network interfaces to your computer based on this entry: `Listen *:631`. By default, it listens on the local interface only on many Linux systems (`Listen localhost:631`).

An access definition for a CUPS printer might appear as follows:
```
<Location /printers/ns1-hp1>
Order Deny,Allow
Deny From All
Allow From 127.0.0.1
AuthType None
</Location>
```
Here, printing to the `ns1-hp1` printer is allowed only for users on the local host (`127.0.0.1`). No password is needed (`AuthType None`). To allow access to the administration tool, CUPS must be configured to prompt for a password (`AuthType Basic`).

For Linux systems that use System V-style startup scripts, use the `chkconfig` command to turn on CUPS so that it starts at each reboot.
- `chkconfig cups on`
- `service cups start`

To start `cupsd` on boot with `systemd`-based systems, use the following commands:
- `systemctl enable cups.service`
- `systemctl start cups.service`

When a new printer is created from the Print Settings window, it is defined in the `/etc/cups/printers.conf` file. This is what a printer entry looks like:
```
<DefaultPrinter printer>
Info HP LaserJet 2100M
Location HP LaserJet 2100M in hall closet
DeviceURI parallel:/dev/lp0
State Idle
Accepting Yes
Shared No
JobSheets none none
QuotaPeriod 0
PageLimit 0
KLimit 0
</Printer>
```
The `Shared No` value is set because the printer is currently available only on the local system. The most interesting information relates to `DeviceURI`, which shows that the printer is connected to parallel port `/dev/lp0`. The state is `Idle` (ready to accept printer jobs), and the `Accepting` value is `Yes` (the printer is accepting print job by default).

The `DeviceURI` has several ways to identify the device name of a printer, reflecting where the printer is connected. Here are some examples listed in the `printers.conf` file:
```
DeviceURI parallel:/dev/plp
DeviceURI serial:/dev/ttyd1?baud=38400+size=8+parity=none+flow=soft
DeviceURI scsi:/dev/scsi/sc1d610
DeviceURI usb://hostname:port
DeviceURI socket://hostname:port
DeviceURI tftp://hostname/path
DeviceURI ftp://hostname/path
DeviceURI http://hostname[:port]/path
DeviceURI ipp:/hostname/path
DeviceURI smb://hostname/printer
```
The first four examples show the form for local printers (`parallel`, `serial`, `scsi`, and `usb`). The other examples are for remote hosts. In each case, `hostname` can be the host's name or IP address. Port numbers or paths identify the locations of each printer on the host.

### Using Printing Commands

You can use the `lp` command to print documents to both local and remote printers (provided the printers are configured locally). Document files can be either added to the end of the `lp` command line or directed to the `lp` command using a pipe (`|`).
- `lp doc1.ps`
- When you specify just a document file with `lp`, output is directed to the default printer. As an individual user, you can change the default printer by setting the value of the `PRINTER` variable. Typically, you add the `PRINTER` variable to one of your startup files, such as `$HOME/.bashrc`. Adding the following line to your `.bashrc` file, for example, sets your default printer to `lp3`:
  - `export PRINTER=lp3`
- To override the default printer, specify a particular printer on the `lp` command line. The following example uses the `-P` option to select a different printer:
  - `lp -P canyonps doc1.ps`
- The `lp` command has a variety of options that enable `lp` to interpret and format several different types of documents. These include `-# num`, where `num` is replaced by the number of copies to print (from 1 to 100) and `-l` (which causes a document to be sent in raw mode, presuming that the document has already been formatted).

Use the `lpstat -t` command to list the status of your printers.

Users can remove their own print jobs from the queue with the `lprm` command. Used alone on the command line, `lprm` removes all of the user's print jobs from the default printer. To remove jobs from a specific printer, use the `-P` option as follows:
- `lprm -P lp0`
- To remove all print jobs for the current user, type the following:
  - `lprm -`
- The root user can remove all of the print jobs for a specific user by indicating that user on the `lprm` command line.
  - `lprm -U mike`
- To remove an individual print job from the queue, indicate its job number on the `lprm` command line. To find the job number, type the `lpq` command.
  - The job number is listed under the `Job` column.

### Configuring a shared CUPS printer

To configure a printer entry manually in the `/etc/cups/printers.conf` file to accept print jobs from all other computers, make sure that the `Shared Yes` line is set.
```
<DefaultPrinter printer>
Info HP LaserJet 2100M
Location HP LasterJet 2100M in hall closet
DeviceURI parallel:/dev/lp0
State Idle
Accepting Yes
Shared Yes
JobSheets none none
QuotaPeriod 0
PageLimit 0
KLimit 0
</Printer>
```

### Configuring a shared Samba printer

When you configure Samba, the `/etc/samba/smb.conf` file is constructed to ensable all of your configured printers to be shared. Here are a few lines from the `smb.conf` file that relate to printer sharing:
```
[global]
...
  load printers = yes
  cups options = raw
  printcap name = /etc/printcap
  printing = cups
...
    comment = All Printers
    path = /var/spool/samba
    browseable = yes
    writeable = no
    printable = yes
```
The selected lines show that printers from `/etc/printcap` were loaded and that the CUPS service is being used. With `cups options` set to raw, Samba assumes that print files have already been formatted by the time they reach your print server. This allows the Linux or Windows clients to provide their own print drivers.
- The last few lines are the actual printers' definition. By changing the `browseable` option from `no` to `yes`, you give users the ability to print to all printers (`printable = yes`). You can also store Windows native print drivers on your Samba server. When a Windows client uses your printer, the driver automatically becomes available. You do not need to download a driver from the vendor's website. To enable the printer driver share, add a Samba share called `print$` that looks like the following:
```
[print$]
Comment = Printer Drivers
path = /var/lib/samba/drivers
browseable = yes
guest ok = no
read only = yes
write list = someone, qquffey
```
After you have the share available, you can start copying Windows print drivers to the `/var/lib/samba/drivers` directory.
***

## Configuring a Web Server

*Apache HTTPD* (also known as the *Apache HTTPD Server*) provides the service with which the client web browsers communicate. The daemon process (httpd) runs in the background on your server and waits for requests from web clients. Web browsers provide those connections to the HTTP daemon and send requests, which the daemon interprets, sending back the appropriate data (such as a web page or other content).

Apache HTTPD includes an interface that allows modules to tie into the process to handle specific portions of a request. Among other things, modules are available to handle the processing of scripting languages, such as Perl or PHP, within web documents and to add encryption to connections between clients and the server.

Apache can be installed on RHEL-based systems with the following command:
- `yum install httpd`

Ubuntu systems use the package `apache2` and can install the program with the following command:
- `apt update && apt install apache2`

The main configuration file is `/etc/httpd/conf/httpd.conf` for Apache. The `welcome.conf` file defines the default home page for your website, until you add some content. The `magic` file defines rules that the server can use to figure out a file's type when the server tries to open it.

The `/etc/logrotate.d/httpd` file defines how log files produced by Apache are rotated. The `/usr/lib/tmpfiles.d/httpd.conf` file defines a directory that contains temporary runtime files (no need to change that file).

Some Apache modules drop configuration files (`*.conf`) into the `/etc/httpd/conf.modules.d/` directory. Any file in that directory that ends in `.conf` is pulled into the main `httpd.conf` file and used to configure Apache. Most module packages that come with configuration files put those configuration files in the `/etc/httpd/conf.d` directory.

### Starting Apache

To get the Apache web server going, you want to enable the service to start on every reboot, and you want to start it immediately. For SysVinit systems use the following commands:
- `chkconfig httpd on`
- `service httpd start`
For systemd-based systems use the following commands:
- `systemctl enable httpd.service`
- `systemctl start httpd.service`
- `systemctl status httpd.service`

When the `httpd` service starts, five or six `httpd` daemon processes are launched by default (depending on your Linux system) to respond to requests for the web server. You can configure more or fewer `httpd` daemons to be started based on settings in the `httpd.conf` file.
- To change the behavior of the `httpd` daemon, you can edit the `httpd` service by running `systemctl edit httpd`.
- To have the Apache program send the maximum amount of log messages make the following changes to the service file:
  - `systemctl edit httpd`
  ```
  [Service]
  Environment=OPTIONS='-e debug'
  ```
  - Restart the service to make the changes take effect.

### Securing Apache

The `httpd` daemon process runs as the user `apache` and the group `apache`. By default, HTML content is stored in the `/var/www/html` directory (as determined by the value of `DocumentRoot` in the `httpd.conf` file).

If you have locked down your firewall in Linux, you need to open several ports for clients to be able to talk to Apache through the firewall. Standard web service (HTTP) is accessible over TCP port 80; secure web service (HTTPS) is accessible via TCP port 443.

To verify which ports are being used by the `httpd` server, use the `netstat` command:
- `netstat -tupln | grep httpd`

The following `iptables` rules will allow the default web ports to be used:
- `iptables -A INPUT -m state --state NEW,RELATED,ESTABLISHED -m tcp -p tcp --dport 80 -j ACCEPT`
- `iptables -A INPUT -m state --state NEW,RELATED,ESTABLISHED -m tcp -p tcp --dport 448 -j ACCEPT`

If *Security Enhanced Linux (SELinux)* is set to enforcing it adds another layer of security over your `httpd` service. In essence, SELinux actually sets out to protect the system from being damaged by someone who may have cracked the `httpd` daemon. SELinux does this by creating policies that do the following:
- Deny access to files that are not set to the right file contexts. For `httpd` in SELinux, there are different file context for content, configuration files, log files, scripts, and other `httpd`-related files. Any file that is not set to the proper context is not accessible to the `httpd` daemon.
- Prevent insecure features from being used, such as file uploading and clear-text authentication, by setting Booleans for such features to the off position. You can selectively turn on Booleans as they are needed if they meet your security requirements.
- Keep the `httpd` daemon from accessing nonstandard features, such as a port outside of the default ports the service would expect to use.

### Understanding the Apache configuration files

**Directives**: the general name for configuration options.

On RHEL-based systems, the basic Apache server's primary configuration file is in `/etc/httpd/conf/httpd.conf`. Besides this file, any file ending in `.conf` in the `/etc/httpd/conf.d` directory is also used for Apache configuration (based on an `Include` line in the `httpd.conf` file). In Ubuntu, the Apache configuration is stored in text files read by the Apache server, beginning with `/etc/apache2/apache2.conf`.

The scope of many configuration directives can be altered based on context. In other words, some parameters may be set on a global level and then changed for a specific file, directory, or virtual host. Other directives are always global in nature, such as those specifying on which IP addresses the server listens. Still others are valid only when applied to a specific location.

Locations are configured in the form of a start tag containing the location type and a resource location, followed by the configuration options for that location, and finishing with an end tag. This form is often called a *configuration block*, and it looks very similar to HTML code. A special type of configuration block, known as a *location block*, is used to limit the scope of directives to specific files and directories. These blocks take the following form:
```
<locationtag specifier>
(options specific to objects matching the specifier go within this block)
</locationtag>
```
The specifier included in the start tag is handled based on the type of location tag. The location tags that you generally use and encounter are `Directory`, `Files`, and `Location`, which limit the scope of the scope of the directives to specific directories, files, or locations, respectively.
- `Directory`: tags are used to specify a path based on the location on the location on the filesystem.
- `Files`: tags are used to specify files by name.
- `Location`: tags are used to specify the URI used to access a file or directory. `Location` tags are processed last and override the settings in `Directory` and `Files` blocks.

Match versions of these tags--`DirectoryMatch`, `FilesMatch`, and `LocationMatch`--have the same function but can contain regular expressions in the resource specifications.

Apache can also be configured to process configuration options contained within files with the name specified in the `AccessFileName` directive (which is generally set to `.htaccess`).

Access control files are useful for allowing users to change specific settings without having access to the server configuration files. The configuration directives permitted within an access configuration file are determined by the `AllowOverride` setting on the directory in which they are contained.

Three directives commonly found in location blocks and access control files are `DirectoryIndex`, `Options`, and `ErrorDocument`:
- `DirectoryIndex` tells Apache which file to load when the URI contains a directory but not a filename. This directive doesn't work in `Files` blocks.
- `Options` is used to adjust how Apache handles files within a directory. The `ExecCGI` option tells Apache that files in that directory can be run as CGI scripts, and the `Includes` option tells Apache that server-side includes (SSIs) are permitted. Another common option is the `Indexes` option, which tells Apache to generate a list of files if one of the filenames found in the `DirectoryIndex` setting is missing. An absolute list of options can be specified, or the list of options can be modified by adding + or - in front of an option name.
- `ErrorDocument` directives can be used to specify a file containing messages to send to web clients when a particular error occurs. The location of the file is relative to the `/var/www` directory. The directive must specify an error code and the full URI for the error document.
  - `ErrorDocument 404 /error/HTTP_NOT_FOUND.html.var`

Another common use for location blocks and access control files is to limit or expand access to a resource. The `Allow` directive can be used to permit access to matching hosts, and the `Deny` directive can be used to forbid it. Both of these options can occur more than once within a block and are handled based on the `Order` setting. Setting `Order` to `Deny,Allow` permits access to any host that is not listed in a `Deny` directive. A setting of `Allow,Deny` denies access to any host not allowed in an `Allow` directive. By adding the `Satisfy` option and some additional parameters, you can add password authentication.

#### Understanding default settings

The following settins show locations where the `httpd` server is getting and putting content by default:
```
ServerRoot "/etc/httpd"
Include conf.d/*.conf
ErrorLog logs/error_log
CustomLog "logs/access_log" combined
DocumentRoot "/var/www/html"
ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"
```
The `ServerRoot` directive identifies `/etc/httpd` as the location where configuration files are stored. At the point in the file, where the `Include` line appears, any files ending in `.conf` from the `/etc/httpd/conf.d` directory are included in the `httpd.conf` file. As errors are encountered and content is served, messages about those activities are placed in files indicated by the `ErrorLog` and `CustomLog` entries.

The `DocumentRoot` and `ScriptAlias` directives where content that is served by your `httpd` server is stored. Traditionally, you would place an `index.html` file in the `DocumentRoot` directory (`/var/www/html`, by default) as the home page and add other content as needed. The `ScriptAlias` directive tells the `httpd` daemon that any scripts requested from the `cgi-bin` directory should be found in the `/var/www/cgi-bin` directory.
```
Listen 80
User apache
Group apache
ServerAdmin root@localhost
DirectoryIndex index.html index.php
AccessFileName .htaccess
```
The `Listen 80` directive tells `httpd` to listen for incoming requests on port 80. By default, it listens on all network interfaces, although you could restrict it to selected interfaces by IP address (for example, `Listen 192.168.0.1:80`).

The `User` and `Group` directives tell `httpd` to run as `apache` for both the user and group. The value of `ServerAdmin` (`root@localhost`, by default) is published on some web pages to tell users where to email if they have problems with the server.

The `DirectoryIndex` lists files that `httpd` will serve if a directory is requested. For example, if a web browser requested http://host/whatever, `httpd` would see whether `/var/www/html/whatever/index.html` existed and serve it if it did. If it didn't exist, `httpd` would look for `index.php`. If that file couldn't be found, the contents of the directory would be displayed. An `AccessFileName` directive can be added to tell `httpd` to use the contents of the `.htaccess` file if it exists in a directory to read in settings that apply to access to that directory. For example, the file could be used to require password protection for the directory or to indicate that the contents of the directory should be displayed in certain ways. For this file to work, however, a `Directory` container would have to have `AllowOverride` opened.

### Virtual Hosts

Apache supports the creation of separate websites within a single server to keep content separate. Individual sites are configured on the same server in what are referred to as *virtual hosts*.

*Virtual hosts* are really just a way to have the content for multiple domain names available from the same Apache server. An Apache server that is doing virtual hosting may have multiple domain names that resolve to the IP address of the server. The content that is served to a web client is based on the name used to access the server. To use name-based virtual hosting, add as many `VirtualHost` containers as you like. After you enable your first `VirtualHost`, your default `DocumentRoot` (`/var/www/html`) is no longer used if someone accesses the server by IP address or some name that is not set in a `VirtualHost` container. Instead, the first `VirtualHost` container is used as the default location for the server.

Below are the steps to configure a virtual host:
1. In RHEL-based systems, create a file named `/etc/httpd/conf.d/example.org.conf` using this template:
  ```
  <VirtualHost *:80>
    ServerAdmin     webmaster@example.org
    ServerName      www.example.org
    ServerAlias     web.example.org
    DocumentRoot    /var/www/html/example.org/
    DirectoryIndex  index.php index.html index.htm
  </VirtualHost>
  ```
  - The `*:80` specification in the `VirtualHost` block indicates to what address and port this virtual host applies. With multiple IP addresses associated with your Linux system, the * can be replaced by a specific IP address. The port is optional for `VirtualHost` specifications but should alwys be used to prevent interference with SSL virtual hosts (which use port 443 by default).
  - The `ServerName` and `ServerAlias` lines tell Apache which names this virtual host should be recognized as, so replace them with names appropriate to your site. You can leave out the `ServerAlias` line if you do not have any alternate names for the server, and you can specify more than one name per `ServerAlias` line or have multiple `ServerAlias` lines if you have several alternate names.
  - The `DocumentRoot` specifies where the web documents (content served for the site) are stored. Although shown as a subdirectory that you create under the default `DocumentRoot` (`/var/www/html`), often sites are attached to the home directories of specific users (such as `/home/user/public_html`) so that each site can be managed by a different user.
2. With the host enabled, use `apachectl` to check the configuration, and then do a graceful restart:
  - `apachectl configtest`
  - `apachectl graceful`: instructs Apache to reload its configuration without disconnecting any active clients.

The first virtual host receives requests for site names that don't match any in your configuration. In a commercial web-hosting environment, it is common to create a special default virtual host that contains an error message indicating that no site by that name has been configured.

### Allowing Users to publish their own web content

The `mod_userdir` module in Apache, once enabled, will make the `public_html` in each user's home directory available to the web at http://servername/<username>/. For example, a user named `wtucker` on `www.example.org` stores web content in `/home/wtucker/public_html`. That content would be available from http://www.example.org/~wtucker.

Make the following changes to the `/etc/httpd/conf/httpd.conf` file to allow users to publish web content from their own home directories.
1. Create a `<IfModule mod_userdir.c>` block.
  ```
  <IfModule mod_userdir.c>
    UserDir enabled <username>
    UserDir public_html
  </IfModule>
  ```
2. Create a `<Directory /home/*/public_html>` directive block and change any settings you like. This is how it will look:
  ```
  <Directory "/home/*/public_html">
    Options Indexes Includes FollowSymLinks
    Requires all granted
  </Directory>
  ```
3. Have your users create their own `public_html` directories in their own home directories.
  - `mkdir $HOME/public_html`
4. Set the execute permission (as root) to allow the `httpd` daemon to access the home directory:
  - `chmod +x /home /home/*`
5. If SELinux is in enforcing mode, a proper SELinux file context (`httpd_user_content_t`) should already be set on the following directories so that SElinux allows the `httpd` daemon to access the content automatically: `/home/*/www`, `/home/*/web`, and `/home/*/public_html`. If for some reason the context is not set, you can set it as follows:
  ```
  ttpd_user_content_t to /home/*/
  chcon -R --reference=/var/www/html/ /home/*/public_html
  ```
6. Set the SELinux Boolean to allow users to share HTML content from their home directories:
  - `setsebool -P httpd_enable_homedirs true`
7. Restart or reload the `httpd` service.
- At this point, you should be able to access content placed in a user's `public_html` directory by pointing a web browser to http://hostname/~user.

### Securing web traffic with SSL/TLS

For web connections, the SSL connection is established first, and then normal HTTP communication is "tunneled" through it. Because SSL negotiation takes place before any HTTP communication, name-based virtual hosting (which occurs at the HTTP layer) does not work easily with SSL. As a consequence, every SSL virtual host you configure should have a unique IP address.

If you have installed the `mod_ssl` package on a RHEL-based system, a self-signed certificate and private key are created when the package is installed. Although the default configuration of `mod_ssl` allows you to have encrypted communications between your web server and clients, because the certificate is untrusted. To install the mod, use the following command:
- `yum install mod_ssl`
- The `mod_ssl` package includes the module needed to implement SSL on your web server (`mod_ssl.so`) and a configuration file for your SSL hosts: `/etc/httpd/conf.d/ssl.conf`.
- The SSL service is set to listen on standard SSL port 443 on all the system's network interfaces.
- A `VirtualHost` block is created that causes error messages and access messages to be logged to log files that are separate from the standard logs used by the server (`ssl_error_log` and `ssl_access_log` in the `/var/log/httpd/` directory). The level of log messages is set to `warn` and the SSLEngine is turned on.

#### Generating an SSL key and self-signed certificate

1. If the `openssl` package is not already installed, install it as follows:
  - `yum install openssl`
2. Generate a 2048-bit RSA private key and save it to a file:
  - `cd /etc/pki/tls/private`
  - `openssl genrsa -out server.key 2048`
  - `chmod 600 server.key`
  - Or, in a higher-security environment, encrypting the key by adding the `-des3` argument after the `genrsa` argument on the `openssl` command is a good idea. When prompted for a passphrase, press `Enter`:
    - `openssl genrsa -des3 -out server.key 1024`
3. To generate a self-signed certificate, run the following commands:
  - `cd /etc/pki/tls/certs`
  - `openssl req -new -x509 -nodes -sha1 -days 365 -key /etc/pki/tls/private/server.key -out server.crt`
4. Edit the `/etc/httpd/conf.d/ssl.conf` file to change the key and certificate locations to use the ones that you just created:
  ```
  SSLCertificateFile /etc/pki/tls/certs/server.crt
  SSLCertificateKeyFile /etc/pki/tls/private/server.key
  ```
5. Restart or reload the `httpd` server.
6. Open https://localhost from a local browser, click the Advanced and then View to see the certificate that was generated. Then select Accept the Risk and Continue to continue to the index page.
***

## Configuring an FTP Server

**FTP**: File Transfer Protocol

The version of FTP often used with various Linux distributions is the Very Secure FTP Daemon (`vsftpd` package).

### FTP Functionality

FTP operates in a client/server model. An FTP server daemon listens for incoming requests (on TCP port 21) from FTP clients. The client presents a login and password. If the server accepts the login information, the client can interactively traverse the filesystem, list files and directories, and then download (and sometimes upload) files. What makes FTP insecure is that everything sent between the FTP client and server is done in clear text. To secure file transfer communications use SSH services such as: `sftp`, `scp`, or `rsync`.

When users authenticate to an FTP server in Linux, their usernames and passwords are authenticated against the standard Linux user accounts and passwords. There is also a special, non-authenticated account used by the FTP server called anonymous. The anonymous account can be accessed by anyone because it does not require a valid password. In fact, the tern *anonymous FTP server* is often used to describe a public FTP server that does not require (or even allow) authentication of a legitimate user account.

After the authentication phase (on the control port, TCP port 21), a second connection is made between the client and server. FTP supports both active and passive connection types. With an *active FTP connection*, the server sends data from its TCP port 20 to some random port the server chooses above port 1023 on the client. With a *passive FTP connection*, the client requests the passive connection and requests a random port from the server.

After the connection is established between the client and server, the client's current directory is established. For the anonymous user, the `/var/ftp` directory is the home directory for RHEL-based systems, and it's `/srv/ftp` for Ubuntu and most Debian-based distributions. The anonymous user cannot go outside of the `/var/ftp` directory structure. If a regular user logs in to the FTP server, their home directory will be their current directory, but they cannot change to any part of the filesystem that they do not have permission.

Command-oriented FTP clients (such as `lftp` and `ftp` commands) go into an interactive mode after connecting to the server. From the prompt you can run many commands that are similar to those that you would use from the shell. You could use `pwd` to see your current directory, `ls` to list directory contents, and `cd` to change directories. When you see a file that you want, you use the `get` and `put` commands to download files from or upload them to the server, respectively.

With graphical tools for accessing FTP servers (such as a web browser), you type the URL of the site that you want to visit (such as ftp://docs.example.com) into the location box of the browser. If you don't add a username or password, an anonymous connection is made and the contents of the home directory of the site are displayed. Click links to directories to change to those directories. Click links to files to display or download those files to your local system.

### Installing and starting the vsftpd FTP Server

For RHEL-based systems:
- `yum install vsftpd`

For Ubuntu and other Debian-based systems:
- `sudo apt-get install vsftpd`

The main configuration file is `/etc/vsftpd/vsftpd.conf` (in RHEL and Fedora) or `/etc/vsftpd.conf` (in Ubuntu). The `ftpusers` and `user_list` (RHEL but not Ubuntu) files in the same directory store information about user accounts that are restricted from accessing the server. The `/etc/pam.d/vsftpd` file sets how authentication is done to the FTP server. The `/etc/logrotate.d/vsftpd` file configures how log files are rotated over time.

To start and enable `vsftpd`, run the following commands:
- `systemctl start vsftpd.service`
- `systemctl enable vsftpd.service`

To check the status of the program, use the following command:
- `systemctl status vsftpd.service`

In SysVinit systems, use the following commands to start and enable the command:
- `service vsftpd start`
- `chkconfig vsftpd on ; chkconfig --list vsftpd`

The `netstat` command can be used to check if the service is running:
- `netstat -tupln | grep vsftpd`

A quck way to check that `vsftpd` is working is to put a file in the `/var/ftp` directory and try to open it from your web browser on the local host:
- `echo "Hello From Your FTP Server" > /var/ftp/hello.txt`
- From a web browser on the local system, type the following into the location box of a web browser:
  - `ftp://localhost/hello.txt`
- Next, try this again from a web browser on another system, replacing `localhost` with your host's IP address or fully qualified host name. If that works, the `vsftpd` server is publicly accessible.

### Securing an FTP Server

The following `iptables` commands will allow FTP through your firewall as well as secure file transfer services via SSH:
```
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 21 -j ACCEPT
```

To apply these changes, run the following commands:
- SysVinit systems: `service iptables restart`
- `systemd` systems: `systemctl restart firewalld.service`

#### Configuring SELinux for an FTP Server

The `getenforce` command will display the current SELinux mode. If it's set to enforcing, the following changes need to be made.
- You can also run `grep ^SELINUX= /etc/sysconfig/selinux` command to check the value in the `SELINUX` environment variable which shows how SELinux is set when the system comes up.

Here are some examples of file contexts that must be set for SELinux to allow files and directories to be accessed by `vsftpd`:
- To share content so that it can be downloaded to FTP clients, that content must be marked with a `public_content_t` file context. Files created in the `/var/ftp` directory or its subdirectories inherit `public_content_t` file context automatically.
- To allow files to be uploaded by anonymous users, the file context on the directory to which you upload must be set to `public_content_rw_t`.

If you have files in the `/var/ftp` directory structure that have the wrong file contexts (which can happen if you move files there from other directories instead of copying them), you can change or restore the file context on those files os that they can be shared. For example, to recursively change the file context of the `/var/ftp/pub/stuff` directory so that the content can be readable from the FTP server through SELinux, enter the following:
- `semanage fcontext -a -t public_content_t "/var/ftp/pub/stuff(/.*)?"`
- `restorecon -F -R -v /var/ftp/pub/stuff`

This example tells SELinux to allow uploading of files to the `/var/ftp/pub/uploads` directory:
- `semanage fcontext -a -t public_content_rw_t "/var/ftp/pub/uploads(/.*)?"`
- `restorecon -F -R -v /var/ftp/pub/uploads`

For SELinux to allow anonymous users to read and write files and directories, you need to turn on the `allow_ftpd_anon_write` (SysVinit) or `ftpd_anon_write` (`systemd`) Boolean:
- `setsebool -P ftpd_anon_write on`

To be able to mount remote NFS or CIFS (Windows) shared filesystems and share them from your `vsftpd` server, you need to turn on the following two Booleans, respectively:
- `setsebool -P allow_ftpd_use_nfs on`
- `setsebool -P allow_ftpd_use_cifs on`

#### Relating Linux file permissions to vsftpd

The `vsftpd` server relies on standard Linux file permissions to allow or deny access to files and directories. As you would expect, for an anonymous user to view or download a file, at least read permission must be open for `other (------r--).` To access a directory, at least execute permission must be on for `other (--------x)`.

### Configuring an FTP Server

The `vsftpd` server comes with all local Linux users (those listed in the `/etc/passwd` file) configured to access the server and the anonymous user prevented. This is based on the following `vsftpd.conf` settings:
```
anonymous_enable=NO
local_enable=YES
```

Creating a user account that has no default shell (actually, `/sbin/nologin`) is how you can keep a user from logging into a shell but still allow FTP access.
- `user:x:1000:1000:Standard User:/home/user:/sbin/nologin`

The setting `userlist_enable=YES` in `vsftpd.conf` says to deny access to the FTP server to all accounts listed in the `/etc/vsftpd/user_list` file.

The `/etc/vsftpd/ftpusers` file always includes users who are denied access to the server.

One way to limit access to users with regular user accounts on your system is to use `chroot` settings. Here are examples of some `chroot` settings:
```
chroot_local_user=YES
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd/chroot_list
```
With the settings just shown uncommented, you could create a list of local users and add them to the `/etc/vsftpd/chroot_list` file. After one of those users logged in, that user would be prevented from going to places in the system that were outside of that user's home directory structure.

#### Allowing Uploading

To allow any form of writing to the `vsftpd` server, you must have `write_enable=YES` set in the `vsftpd.conf` file (which it is, by default). Because of that, if local accounts are enabled, users can log in and immediately begin uploading files to their own home directories. However, anonymous users are denied the ability to upload files by default. To allow anonymous uploads with `vsftpd`, you must have the first option in the following code example, and you may want the second line of code as well (both can be enabled by uncommenting them from the `vsftpd.conf` file). The first allows anonymous users to upload files; the second allows them to create directories:
```
anon_upload_enable=YES
anon_mkdir_write_enable=YES
```
The next step is to create a directory where anonymous users can write. A common thing is to create an uploads directory with permission open for writing.
- `mkdir /var/ftp/uploads`
- `chown ftp:ftp /var/ftp/uploads`
- `chmod 775 /var/ftp/uploads`

By setting the following two values, you can allow anonymous uploads. The result of these settings is that when an anonymous user uploads a file, that file is immediately assigned ownership of a different user. The following is an example of some `chown`settings that you could put in your `vsftpd.conf` file to use with your anonymous upload directory:
```
chown_uploads=YES
chown_username=user
```
If an anonymous user were to upload a file after `vsftpd` was restarted with these settings, the uploaded file would be owned by the user `user` and the `ftp` group. Permissions would be read/write for the owner and nothing for anyone else (`rw-------`).

#### Setting up vsftpd for the internet

To start with a configuration that is designed to share `vsftpd` files safely over the Internet, back up your current `/etc/vsftpd/vsftpd.conf` file and copy this file to overwrite your `vsftpd.conf`:
- `/sur/share/doc/vsftpd/EXAMPLE/INTERNET_SITE/vsftpd.conf`

The following settings in the first section set the access rights for the server:
```
# Access rights
anonymous_enable=YES
local_enable=NO
write_enable=NO
anon_upload_enable=NO
anon_mkdir_write_enable=NO
anon_other_write_enable=NO
```
Turning on `anonymous_enable (YES)` and turning off `local_enable (NO)` ensures that no one can log in to the FTP server using a regular Linux user account. No one can upload files (`write_enable=NO`). Then, the anonymous user cannot upload files (`anon_upload_enable=NO`), create directories (`anon_other_write_enable=NO`). Here are the Security settings:
```
# Security
anon_world_readable_only=YES
connect_from_port_20=YES
hide_ids=YES
pasv_min_port=50000
pasv_max_port=60000
```
Because the `vsftpd` daemon can read files assigned to the `ftp` user and group, setting `anon_world_readable_only=YES` ensures that anonymous users can see files where the read permission bit is turned on for `other` (`------r--`), but not write files. The `connect_from_port_20=YES` setting gives the `vsftpd` daemon slightly more permission to send data the way a client might request by allowing `PORT`-style data communications. Using `hide_ids=YES` hides the real permissions set on files, so to the user accessing the FTP site, everything appears to be owned by the `ftp` user. The two `pasv` settings restrict the number port on which to send data) to between 50000 and 60000.

The next section contains features of the `vsftpd` server:
```
# Features
xferlog_enable=YES
ls_recurse_enable=NO
ascii_download_enable=NO
async_abor_enable=YES
```
With `xferlog_enable=YES`, all file transfers to and from the server are logged to the `/var/log/xferlog` file. Setting `ls_recurse_enable=NO` prevents users from recursively listing the contents of an FTP directory (in other words, it prevents the type of listing that you could get with the `ls -R` command) because on a large site, that could drain resources. Disabling ASCII downloads forces all downloads to be in binary mode (preventing files from being translated in ASCII, which is inappropriate for binary files). The `async_abor_enable=YES` setting ensures that some FTP clients, which might hang when aborting a transfer, will not hang.

The following settings have an impact on performance:
```
# Performance
one_process_model=YES
idle_session_timeout=120
data_connection_timeout=300
accept_timeout=60
connect_timeout=60
anon_max_rate=50000
```
With `one_process_model=YES` set, performance can improve because `vsftpd` launches one process per connection. Reducing the `idle_session_timeout` from the default 300 seconds to 120 seconds causes FTP clients that are idle more than 2 minutes to be disconnected. Thus, less time is spent managing FTP sessions that are no longer in use. If a data transfer stalls for more than `data_connection_timeout` seconds (300 seconds here), the connection to the client is dropped. The `accept_timeout` setting of 60 seconds allows 1 minute for a PASV connection to be accepted by the remote client. The `connect_timeout` sets how long a remote client has to respond to a request to establish a `PORT`-style data connection. Limiting the transfer rate to 50000 (bytes per second) with `anon_max_rate` can improve overall performance of the server by limiting how much bandwidth each client can consume.

### Using FTP Clients to Connect to an FTP Server

To log in to an FTP server as a particular user from a web browser, you can precede the host name with a `username:password@` notation, as shown in the command below:
- ftp://user:password@localhost

To test your FTP server from the command line, you can use the `lftp` command, which can be installed on RHEL servers with the command: `yum install lftp`. If you use the `lftp` command with just the name of the FTP server you are trying to access, the command tries to connect to the FTP server as the anonymous user. By adding the `-u username`, you can enter the user's password when prompted and gain access to the FTP server as the user you logged in as.
- `lftp -u user localhost`
- To have the commands you run interpreted by the client system, you can simply put an exclamation mark (!) in front of a command. For example, running `!pwd` shows that the current directory on the system that initiated the `lftp` is `/root`.
***

## Configuring a Windows File Sharing (Samba) Server

Samba is the project that implements open source versions of protocols used to share files and printers among Windows systems as well as authenticate users and restrict hosts. Samba offers a number of ways to share files among Windows, Linux, and MacOS systems that are well known and readily available to users of those systems. With Samba, you can use standard TCP/IP networking to communicate with clients. For name service, Samba supports regular TCP/IP hostnames as well as NetBIOS names. For that reason, Samba doesn't require the NetBEUI (Microsoft Raw NetBIOS frame) protocol. File sharing is done using Server Message Block (SMB) protocol, which is sometimes referred to as the *Common Internet File System (CIFS)*.

### Installing Samba

Among other components, the `samba` package includes the Samba service daemon (`/usr/sbin/smbd`) and NetBIOS name server daemon (`/usr/sbin/nmbd`). Installing the `samba` package pulls in the `samba-common` package, which contains server configuration files (`smb.conf`, `lmhosts`, and others) and commands for adding passwords and testing configuration files, along with other Samba features.
- `samba-client` **package**: Contains command-line tools such as `smbclient` (for connecting to Samba or Windows shares), `nmblookup` (for looking up host addresses), and `findsmb` (to find SMB hosts on the network).
- `samba-winbind` **package**: Includes components that allow your Samba server in Linux to become a complete member of a Windows domain, including using Windows user and group accounts in Linux.

To install the service on RHEL-based systems, run the following command:
- `yum install samba sabma-client samba-windbind`

### Starting and Stopping Samba

Two main services are associated with a Samba server, each of which has its own service daemon:
- **smb**: This service controls the `smbd` daemon process, which provides the file and print sharing services that can be accessed by Windows clients.
- **nmb**: This service controls the `nmbd` daemon. By providing NetBIOS name service name-to-address mapping, `nmbd` can map requests from Windows clients for NetBIOS names so that they can be resolved into IP addresses.

For `systemd`-based systems, use the following commands to enable at boot, start, and check the status of the `samba` service:
- `systemctl enable smb.service`
- `systemctl start smb.service`
- `systemctl status smb.service`
- The service file is located at `/usr/lib/systemd/system/smb.service`.

The Samba daemon process (`smbd`) starts up after the `network`, `network-online`, `nmb`, and `winbind` targets. The `/etc/sysconfig/samba` file contains variables that are passed as arguments to the `smbd`, `nmbd`, and `winbindd` daemons when they start. No options are set by default for any of those daemons.

In SysVinit-based systems, use the following commands to accomplish the same as the above `systemd` commands:
- `service smb start`
- `chkconfig smb on`
- `service smb status`
- `chkconfig --list smb`

You can check access to the Samba server using the `smbclient` command (from the `samba-client` package):
- `smbclient -L localhost`
  - You must enter the password for the user you are currently logged-in as if it is required by the current Samba settings.
  - The `smbclient` output allows you to see what services are available from the server. By default, anonymous login is allowed when querying the server (so you can press Enter when prompted for a password).

#### Starting the NetBIOS (nmbd) name server

To start the `nmb` service (`nmbd` daemon) in `systemd`-based system:
- `systemctl enable nmb.service`
- `systemctl start nmb.service`
- `systemctl status nmb.service`

To start the `nmb` service on SysVinit-based systems:
- `service nmb start`
- `service nmb status`
- `chkconfig nmb on`
- `chkconfig --list nmb`

Regardless of how the NetBIOS service was started, the `nmbd` daemon should now be running and ready to serve NetBIOS name-to-address mapping. Run the `smbclient -L <server-IP>` command and check the `Workgroup` section of the output which should now list the NetBIOS name of the server.

To query the `nmbd` server for the IP address of a Samba server, run the following command:
- `nmblookup -U localhost <NetBIOS name>`

#### Stopping the Samba (smb) and NetBIOS (nmb) services

To stop the services on a `systemd`-based machine:
- `systemctl stop smb.service`
- `systemctl stop nmb.service`

To stop the services on a SysVinit-based machine:
- `service smb stop`
- `service nmb stop`

To prevent the `smb` and `nmb` services from starting the next time the system reboots, enter the following commands for `systemd`-based systems:
- `systemctl disable smb.service`
- `systemctl disable nmb.service`

To prevent the same as above for SysVinit-based systems, run the following commands:
- `chkconfig smb off`
- `chkconfig nmb off`

### Configuring firewalls for Samba

These are the ports that you should open for the Samba service:
- **TCP port 445**: This is the primary port on which the Samba `smbd` daemon listens. Your firewall must support incoming packet requests on this port for Samba to work.
- **TCP port 139**: The `smbd` daemon also listens on TCP port 139 in order to handle sessions associated with NetBIOS hostnames. It is possible to use Samba over TCP without opening this port, but it is not recommended.
- **UDP ports 137 and 138**: The `nmbd` daemon uses these two ports for incoming NetBIOS requests. If you are using the `nmbd` daemon, these two ports must be open for new packet requests for NetBIOS name resolution.

Below are some `iptables` rules that can be used to allow the above ports to be open:
```
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
iptables -A INPUT -i lo -j ACCEPT  
iptables -A INPUT -m state --state NEW -m udp -p udp --dport 137 -j ACCEPT
iptables -A INPUT -m state --state NEW -m udp -p udp --dport 138 -j ACCEPT
iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 139 -j ACCEPT
iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 445 -j ACCEPT
```

To get the above rules to take effect, restart the service:
- `systemd`: `systemctl restart iptables.service`
- SysVinit: `service restart iptables`

### Configuring SELinux for Samba

File context must be properly set on a directory that is shared by Samba. Booleans allow you to override the secure-by-default approach to certain Samba features.

To use the `semanage` command to list Samba-related Booleans, enter the following:
- `semanage boolean -l | egrep "smb|samba"`

The following is a list of SELinux Booleans that apply to Samba and their descriptions.
- **samba_run_unconfined**: Allows samba to run unconfined scripts from Samba shares.
- **smbd_anon_write**: Allows Samba to let anonymous users modify public files used for public file transfer services. Files and directories must be labeled `public_content_rw_t`.
- **samba_enable_home_dirs**: Allows Samba to share users; home directories.
- **samba_export_all_ro**: Allows Samba to share any file and directory read-only.
- **use_samba_home_dirs**: Allows a remote Samba server to access home directories on the local machine.
- **samba_create_home_dirs**: Allows Samba to create new home directories (for example, via PAM).
- **samba_export_all_rw**: Allows Samba to share any file or directory read/write.

The following Booleans affect Samba's ability to share directories that are themselves mounted from other remote services (such as NFS) or to act as a Windows domain controller:
- **samba_share_fusefs**: Allows Samba to export `ntfs/fusefs` volumes.
- **samba_share_nfs**: Allows Samba to export NFS volumes.
- **samba_domain_controller**: Allows Samba to act as the domain controller, add users and groups, and change passwords.

The `setsebool` command is used to turn the SELinux Booleans on or off. Used with the `-P` option, `setsebool` sets the Boolean you indicate permanently. For example, to allow Samba to share any file or directory with read-only permission from the server, you could type the following from a shell as root user:
- `setsebool -P samba_export_all_ro on`
- `getsebool samba_export_all_ro`

To find files and directories associated with the Samba service and `smbd` daemon that have file contexts preset, run the following:
- `semanage fcontext -l | grep -i samba`
- `semanage fcontext -l | grep -i smb`

The file context portion in which you are interested ends with `_t`: for example, `samba_etc_t`, `samba_log_t`, and `samba_var_t` for the `/etc/samba`, `/var/log/samba`, and `/var/lib/samba` directories, respectively.

You can change a file context permanently by creating a new file context rule and then applying that rule to the file or directory for which it is intended. You can do that with the `semanage` command (to make the rule) and `restorecon` command (to apply the rule). For example, if you wanted to share a directory, `/mystuff`, you would create that directory with the proper permissions and run the following command to make it available for read/write access from Samba:
- `semanage fcontext -a -t samba_share_t "/mystuff(/.*)?"`
- `restorecon -v /mystuff`
After those commands are run, the `/mystuff` directory, along with any files and directories below that point, have the file context of `samba_share_t`. It is then up to you to assign the correct Linux ownership and file permissions to allow access to the users you choose.

### Configuring Samba

The file `/etc/samba/smb.conf` consists of the following predefined sections:
- **[global]**: Settings that apply to the Samba server as a whole are placed in this section. This is where you set the server's description, its workgroup (domain), the location of log files, the default type of security, and other settings.
- **[homes]**: This section determines whether users with accounts on the Samba server can see their home directories (browse-able) or write to them.
- **[printers]**: In this section, settings tell Samba whether to make printers available through Samba that are configured for Linux printing (CUPS).
- **[print$]**: This section configures a directory as a shared printer drivers folder.

#### Configuring the [global] section

Here is an example of a `[global]` section of the `smb.conf` file:
```
[global]
  workgroup = SAMBA
  security = user
  passdb backend = tdbsam
  printing = cups
  printcap name = cups
  load printers = yes
  cups options = raw
; netbios name = MYSERVER
; interfaces = lo eth0 192.168.12.2/24 192.168.13.2/24
; hosts allow = 127. 192.168.12 192.168.13
```
The `workgroup` (also used as the domain name) is set to `SAMBA` in this example. When a client communicates with the Samba server, this name tells the client which workgroup the Samba server is in. The default `security` type is set to `user` (Samba usernames and passwords). The `passdb backend = tdbsam` specifies to use a Samba backend database to hold passwords. You can use the `smbpasswd` command to set each user's password. Setting `printing = cups` and `printcap name = cups` indicates to use the `printcap` created by the CUPS printing service. WHen you set `load printers = yes`, Samba knows to share any printers configured by your local CUPS printing service from Samba. The `cups options` lets you pass any options that you like to the CUPS printers served by your Samba server. By default, only `raw` is set, which allows Windows clients to use their own print drivers. Printers on your Samba server print the pages they are presented in raw form. By default, your server's DNS hostname (enter `hostname` to see what it is) is used as your Samba server's NetBIOS name as well. You can override that and set a separate NetBIOS name by uncommenting the `netbios name` line and adding the server name you want. If you want to restrict access to the Samba server so that it only responds on certain interfaces, you can uncomment the `interfaces` line and add either the IP address or name (`lo`, `eth0`, `eth1`, and so on) of the network interfaces you want. You can restrict access to the Samba server to specific hosts as well. Uncomment the `hosts allow` line (remove the semicolon) and insert the IP addresses of the hosts that you want to allow. To enter a range of addresses, simply end the subnetwork portion of the address, followed by a dot. For example, `127.` is associated with IP addresses that point to the local host. The `192.168.12.` entry matches all IP address from `192.168.12.1` to `192.168.12.254`.

#### Configuring the [homes] section

The `[homes]` section is configured, by default, to allow any Samba user account to be able to access its own home directory via the Samba server. Here is what the default homes entry looks like:
```
[homes]
  comment = Home Directories
  valid users = %S, %D%w%S
  browseable = No
  read only = No
  inherit acls = Yes
```
Setting `valid users` to `%S` substitutes the current service name, which allows any valid users of the service to access their home directories. The valid users are also identified by domain or workgroup (`%D`), winbind separator (`%w`), and name of current service (`%S`). The `browseable = No` setting prevents the Samba server from displaying the availability of the shared home directories. Users who can provide their own Samba usernames and passwords can read and write in their own home directories (`read only = no`). With `inherit acls` set to `Yes`, access control lists can be inherited to add another layer of security on the shared files.
- To tell SELinux to allow Samba users to access their home directories as Samba shares, turn on the `samba_enable_home_dirs` Boolean by entering the following as root from a shell:
  - `setsebool -P samba_enable_home_dirs on`

#### Configuring the [printers] section

Any printer that you configure for CUPS printing on your Linux system is automatically shared to others over Samba, based on the `[printers]` section that is added by default. The global `cups options = raw` setting makes all printers raw printers (meaning that the Windows client needs to provide the proper printer driver for each shared printer). Here's what the default printers section looks like in the `smb.conf` file:
```
[printers]
  comment = All Printers
  path = /var/tmp
  printable = Yes
  create mask = 0600
  browseable = No
```
The `path` tells Samba to store temporary print files in `/var/tmp`. The `printable = Yes` line causes all of your CUPS printers on the local system to be shared by Samba. The `create mask = 0600` setting used here has the effect of removing write and execute bits for group and other, within the ACL, when files are created in the path directory. To see that local printers are available you could run the `smbclient -L` command from a command line.

#### Creating a Samba shared folder

Before you can create a shared folder, that folder (directory) must exist and have the proper permissions set. In this example, the `/var/mystuff` directory is shared. You want the data to be writable by the user named `user` but visible to anyone on your network. To create that directory and set the proper permissions and SELinux file contexts, type the following as root user:
```
mkdir /var/mystuff
chmod 775 /var/mystuff
chown user:user /var/mystuff
semanage fcontext -a -t samba_share_t /var/mystuff
restorecon -v /var/mystuff
touch /var/mystuff/test
ls -lZ /var/mystuff/test
```
With the `/var/mystuff` directory created and properly configured to be shared by Samba, here is what the shared folder (called `mystuff`) might look like in the `smb.conf` file:
```
[mystuff]
  comment = My Stuff
  path = /var/mystuff
  read only = no
;  browseable = yes
  valid users = user
```
After `user` is connected to the share, `user` has read and write access to it (`read only = no`).

For the changes to your Samba configuration to take effect, you need to restart the `smb` service:
- `systemctl restart smb.service`
- `smbclient -L localhost -U user` or `smbclient -U user //localhost/mystuff`
- Once connected, you could run the following commands:
  - `smb: \> lcd /etc`
  - `smb: \> put file`
  - `smb: \> ls`
  - `smb: \> quit`
- A Samba share is in the form `//host/share` or `\\host\share`. However, when you identify a Samba share from a Linux shell in the latter case, the backslashes need to be escaped. So, as an argument, the first example of the share would have to appear as `\\\\localhost\\mystuff`. Thus, the first form is easier to use.

Particular Samba users and groups can be allowed access to specific Samba shares by identifying those users and groups within a share in the `smb.conf` file. Aside from guest users, which you may or may not allow, the default user authentication for Samba requires you to add a Samba (Windows) user account that maps into a local Linux user account. To allow a user to access the Samba server, you need to create a password for the user. Here is an example of how to add a Samba password for the user `user`:
- `smbpasswd -a user`

The `/var/lib/samba/private/passdb.tdb` file holds the password just entered for `user`. After that, the user `user` can change the password by simply typing `smbpasswd` when he is logged in. The root user can change the password by returning the command shown in the example by dropping the `-a` option. If you wanted to give `user` access to a share, you could add a `valid users` line to that shared block in the `smb.conf` file. For example, to provide both `user` and `user2` access to a share, you could add the following line:
- `valid users = user, user2`
- If `read only` is set to `yes`, you could still allow access to `user` and `user2` to write files by adding a `write list` line as follows:
  - `write list = user, user2`
- You can add write permission for a group by putting a plus (+) character in front of a name. For example, the following adds write access for the `users` group to the share with which this line is associated:
  - `write list = user, user2, +users`

### Accessing Samba Shares

With the Nautilus window manager displayed, select Other Location in the left navigation bar. Available networks (such as Windows Network) should appear. Look to the box at the bottom of the window identified as Connect to Server, and then enter the location of an available Samba share. Given the previous example, you would be able to use either of these shares:
- smb://192.168.1.10/user
- smb://192.168.1.10/mystuff
- You will have to enter the username, password, and Domain that the share is associated with to access it.

#### Mounting a Samba share from a Linux command line

Using the standard `mount` command (with `cifs-utils` installed), you can mount a remote Samba share as a CIFS filesystem in Linux. This example mounts the `mystuff` share from the host at IP address 192.168.1.11 on the local directory `/mnt/mystuff`:
```
yum install cifs-utils -y
mkdir /mnt/mystuff
mount -t cifs -o user=user //192.168.1.11/mystuff /mnt/mystuff
```
- Enter the password for `user`.

If you want the share to be mounted permanently (that is, every time the system boots up) in the same location, you can do some additional configuration. First, open the `/etc/fstab` file and add an entry similar to the following:
- `//192.168.1.10/mystuff /mnt/sales cifs credentials=/root/cif.txt 0 0`
- Next, create a credentials file (in this example, `/root/cif.txt`). In that file, put the name of the user and the user's password that you want to present when the system tries to mount the filesystem. Here is an example of the contents that could exist in the file:
```
user=user
pass=mypass
```
***

## Configuring an NFS File Server

Instead of representing storage devices as drive letters (A, B, C, and so on), as they are in Windows, Linux systems invisibly connect filesystems from multiple hard disks, USB drives, CD-ROMs, and other local devices to form a single Linux filesystem. The Network File System (NFS) facility enables you to extend your Linux filesystem to connect filesystems on other computers to your local directory structure.

To run an NFS server, you need a set of kernel modules (which are delivered with the kernel itself), plus some user-level tools to configure the service, run daemon processes, and query the service in various ways. To install the service on RHEL-based systems, use the following commands:
- `yum install nfs-utils`

### Starting the NFS service

Starting the NFS server involves launching several service daemons. The basic NFS service is called `nfs-server`. To start that service, enable it (if you want it to start on boot) and check the status to ensure it's running with the following commands for `systemd`-based systems:
- `systemctl start nfs-server.service`
- `systemctl enable nfs-server.service`
- `systemctl status nfs-server.service`

The NFS service also requires that the RPC service be running (`rpcbind`). The `nfs-server` service automatically starts the `rpcbind` service, if it is not already running. For SysVinit-based systems, the following commands accomplish the same as the above `systemd`-based commands but also start the `rpcbind` service:
- `service rpcbind start`
- `service nfs start`
- `chkconfig rpcbind on`
- `chkconfig nfs on`

### Sharing NFS Filesystems

To share an NFS filesystem from your Linux system, you need to export it from the server system. Exporting is done in Linux by adding entries into the `/etc/exports` file. Here is the basic format of entries in this file:
- `Direcotry    Host(Options...)    Host(Options...)    # Comments`
- The `Directory` piece is the name of the directory that you want to share, and `Host` indicates the client computer to which the sharing of this directory is restricted. `Options` can include a variety of options to define the security measures attached to the shared directory for the host. (You can repeat `Host` and `Option` pairs). `Comments` are any optional comments that you want to add (following the # sign).

Here is an example of an `/etc/exports` file:
```
/cal    *.linuxtoys.net(rw)         # Company events
/pub    *(ro,insecure,all_squash)   # Public dir
/homes  remote(rw,root_squash)  remote2(rw,root_squash)
```
- The `/cal` entry represents a directory that contains information about events related to the company. Any computer in the company's domain (`*.linuxtoys.net`) can mount that NFS share. Users can write files to the directory as well as read them (indicated by the `rw` option). The comment (`# Company events `) simply serves to remind you of what the directory contains.
- The `/pub` entry represents a public directory. It allows any computer and user to read files from the directory (indicated by the `ro` option) but not to write files. The `insecure` option enable any computer, even one that doesn't use a secure NFS port, to access the directory. The `all_squash` option causes all users (UIDs) and groups (GIDs) to be mapped to the `nobody` user (UID 65534), giving them minimal permission to files and directories.
- The `/home` entry enables a set of users to have the same `/home` directory on different computers. Suppose, for example, that you are sharing `/home` from a computer named `local`. The computers named `remote` and `remote2` could each mount that directory on their own `/home` directories. If you gave all users the same username/UID on all machines, you could have the same `/home/user` directory available for each user, regardless of which computer they are logged into. The `root_squash` is used to exclude the root user from another computer from having root privilege to the shared directory.

#### Hostnames in /etc/exports

You can indicate in the `/etc/exports` file which host computers can have access to your shared directory. For example:
- `/usr/local remote(rw) remote2(ro,root_squash)`

You can identify hosts in several ways:
- **Individual host**: Enter one or more TCP/IP hostnames or IP addresses. If the host is in your local domain, you can simply indicate the hostname. Otherwise, use the full `host.domain` format. These are valid ways to indicate individual host computers:
  - `remote`
  - `remote.example.com`
  - `10.0.0.12`
- **IP Network**: Allow access to all hosts from a particular network address by indicating a network number and its netmask, separated by a slash (/). Here are valid ways to designate network numbers:
  - `10.0.0.0/255.0.0.0 172.16.0.0/255.255.0.0`
  - `192.168.18.0/255.255.255.0`
  - `192.168.18.0/24`
- **TCP/IP domain**: Using wildcards, you can include all or some host computers from a particular domain level. Here are some valid uses of the asterisk and question mark wildcards:
  - `*.example.com`
  - `*remote.example.com`
  - `???.example.com`

#### Access options in /etc/exports

You can add options that allow or limit access by setting read/write permission:
- **ro**: Client can mount this exported filesystem read-only. The default is to mount the filesystem read/write.
- **rw**: Explicitly asks that a shared directory be shared with read/write permissions. (If the client chooses, it can still mount the directory as read-only).

#### User mapping options in /etc/exports

Here are some methods of setting user permissions and the `/etc/exports` option that you use for each method:
- **root user**: The client's root user is mapped by default into the `nobody` username (UID 65534). This prevents a client computer's root user from being able to change all files and directories in the shared filesystem. If you want to the client's root user to have root permission on the server, use the `no_root_squash` option.
- **nfsnobody or nobody user/group**: By using the 65534 user ID and group ID, you essentially create a user/group with permissions that do not allow access to the files that belong to any real users on the server, unless those users open permission to everyone. However, files created by the 65534 user or group are available to anyone assigned as the 65534 user or group. To set all remote users to the 65534 user/group, use the `all_squash` option. The 65534 UIDs and GIDs are used to prevent the ID from running into a valid user or group ID. Using `anonuid` or `anongid` options, you can change the 65534 user or group, respectively. For example, `anonuid=175` sets all `anonymous` users to UID 175, and `anongid=300` sets the GID to 300. (Only the number is displayed when you list file permissions unless you add entries with names to `/etc/passwd` and `/etc/group` for the new UIDs and GIDs).
- **User mapping**: If a user has login accounts for a set of computers (and has the same ID), NFS by default, maps that ID. This means that if the user named `user` (UID 110) on `remote` has an account on `remote2` (`user`, UID 110), he can use his own remotely mounted files on either computer from either computer. If a client user who is not set up on the server creates a file on the mounted NFS directory, the file is assigned to the remote client's UID and GID. (An `ls -l` on the server shows the UID of the owner).

#### Exporting the shared filesystems

After you have added entries to your `/etc/exports` file, run the `exportfs` command to have those directories exported (made available to other computers on the network). Reboot your computer or restart the NFS service, and the `exportfs` command runs automatically to export your directories. Running the `exportfs` command after you change the exports file is a good idea. If any errors are in the file, `exportfs` identifies them for you.
- `/usr/sbin/exportfs -a -r -v`
- The `-a` option indicates that all directories listed in `/etc/exports` should be exported. The `-r` resyncs all exports with the current `/etc/exports` file (disabling those exports no longer listed in the file). The `-v` option says to print verbose output.

### Securing an NFS Server

- **NFSv4**: By integrating Kerberos support, NFSv4 lets you configure user access based on each user obtaining a Kerberos ticket. For you, the extra work is configuring a Kerberos server. As for exposing NFS share locations, with NFSv4 you can bind shared directories to an `/exports` directory, so when they are shared, the exact location of those directories is not exposed.

#### Firewall options for NFS

For the default NFSv4 service, TCP and UDP ports 2049 (`nfs`) and 111 (`rpcbind`) must be open for an NFS server to perform properly. The server must also open TCP and UDP ports 20048 for the `showmount` command to be able to query available NFS shared directories from `rpc.mounted` on the server. Below are some `iptables` commands for opening up the necessary ports for NFS:
```
iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 111 -j ACCEPT
iptables -A INPUT -m state --state NEW -m udp -p udp --dport 111 -j ACCEPT
iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 2049 -j ACCEPT
iptables -A INPUT -m state --state NEW -m udp -p udp --dport 2049 -j ACCEPT
iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 20048 -j ACCEPT
iptables -A INPUT -m state --state NEW -m udp -p udp --dport 20048 -j ACCEPT
```
To make the process of locking down NFS server ports easier, entries in the `/etc/sysconfig/nfs` file can be added to assign specific port numbers to services. The following are examples of options in the `/etc/sysconfig/nfs` file with static port numbers set:
```
RQUOTAD_PORT=49001
LOCKD_TCPPORT=49002
LOCKD_UDPPORT=49003
MOUNTD_PORT=49004
STATD_PORT=49005
STATD_OUTGOING_PORT=49006
RDMA_PORT=49007
```

#### Configuring SELinux for an NFS Server

Here are a few SELinux file contexts associated with NFS that you might need to know about:
- `nfs_export_all_ro`: With this Boolean set to on, SELinux allows you to share files with read-only permission using NFS. NFS read-only file sharing is allowed with this on regardless of the SELinux file context set on the shared files and directories.
- `nfs_export_all_rw`: With this Boolean set to on, SELinux allows you to share files with read/write permission using NFS.As with the previous Boolean, this works regardless of the file context set on the shared files and directories.
- `use_nfs_home_dirs`: To allow the NFS server to share your home directories via NFS, set this Boolean to on.
Of the Booleans just described, the first two are on by default. The `use_nfs_home_dirs` Boolean is off. To turn on the `use_nfs_home_dirs` directory, you could type the following:
- `setsebool -P use_nfs_home_dirs on`

The `public_content_t` and `public_content_rw_t` file contexts can be set on any directory that you want to share via NFS (or other file share protocols, such as HTTP, FTP, and others, for that matter). For example, to set the rule to allow the `/mystuff` directory and its subdirectories to be shared read/write via NFS, and then to apply that rule, enter the following:
- `semanage fcontext -a -t public_content_rw_t "/mystuf(/.*)?"`
- `restorecon -F -R -v /mystuff`
- If you wanted to allow users just to be able to read files from a directory, but not write to it, you could assign the `public_content_t` file context to the directory instead.

### Using NFS Filesystems

After a server exports a directory over the network using NFS, a client computer connects that directory to its own filesystem using the `mount` command. The `mount` command enables a client to mount NFS directories added to the `/etc/fstab` file automatically, just as it does with the local disks. NFS directories can also be added to the `/etc/fstab` file in such a way that they are not automatically mounted (so you can mount them manually when you choose). With a `noauto` option, an NFS directory listed in `/etc/fstab` is inactive until the `mount` command is used, after the system is up and running to mount the filesystem. In addition to the `/etc/fstab` file, you can set mount options using the `/etc/nfs-mount.conf` file. Before you set about mounting NFS shared directories, however, you probably want to check out what shared directories are available via NFS using the `showmount` command:
- `showmount -e server.example.com`

#### Manually mounting an NFS filesystem

The following is an example of mounting the `/mystuff` directory from a computer named `remote` on your local computer:
- `mkdir /mnt/mystuff`
- `mount remote:/mystuff /mnt/mystuff`

To ensure that the NFS mount occurred, type `mount -t nfs4`. This command lists all mounted NFS filesystems.

#### Mounting an NFS filesystem at boot time

Here's the format for adding an NFS filesystem to your local system:
- `host:directory     mountpoint    nfs4  options     0   0`
- The first item (`host:directory`) identifies the NFS server computer and shared directory, `mountpoint` is the local mount point on which the NFS directory is mounted. It is followed by the filesystem type (`nfs`). Any options related to the mount appear next in a comma-separated list. (the last two zeros configure the system not to dump the contents of the filesystem and not to run `fsck` on the filesystem).

Below are some examples of NFS entries in `/etc/fstab`:
- `remote:/mystuff    /mnt/mystuff    nfs     bg,resize=8192,wsize=8192    0  0`
- `remote2:/apps      /mnt/apps       nfs     noauto,ro     0   0`

A `noauto` filesystem can be mounted manually. The advantage is that when you type the `mount` command, you can type less information and have the rest filled in by the contents of the `/etc/fstab` file. For example, you could type: `mount /mnt/apps` based on the above example.

You can add several `mount` options to the `/etc/fstab` file (or to a `mount` command line itself) to influence how the filesystem is mounted. When you add options to `/etc/fstab`, they must be separated by commas. The following are some options that are valuable for mounting NFS filesystems:
- **hard**: If this option is used and the NFS server disconnects or goes down while a process is waiting to access it, the process hangs until the server comes back up. (This is the default behavior).
- **soft**: If the NFS server disconnects or goes down, a process trying to access data from the server times out after a set period when this option is on. An input/output error is delivered to the process trying to access the NFS server.
- **rsize**: This is the size of the blocks of data (in bytes) that the NFS client will request be used when it is reading data from an NFS server. The default is 1024.
- **wsize**: This is the size of the blocks of data (in bytes) that the NFS client will request to be used when it is writing data to an NFS server. The default is 1024.
- **timeo=#**: This sets the time after an RPC time-out occurs that a second transmission is made, where # represents a number in tenths of a second. The default value is seven-tenths of a second. Each successive time-out causes the time-out value to be doubled (up to 60 seconds maximum).
- **retrans=#**: This sets the number of minor time-outs and retransmissions that need to happen before a major time-out occurs.
- **retry=#**: This sets how many minutes to continue to retry failed mount requests, where # is replaced by the number of minutes to retry. The default is 10,000 minutes (which is about one week).
- **bg**: If the first mount attempt times out, try all subsequent mounts in the background.
- **fg**: If the first mount attempt times out, try subsequent mounts in the foreground. This is the default behavior.

Not all NFS mount options need to go into the `/etc/fstab` file. On the client side, the `/etc/nfsmount.conf` file can be configured for Mount, Server, and Global sections. In the Mount section, you can indicate which mount options are used when an NFS filesystem is mounted to a particular mount point. The Server sections lets you add options to any NFS filesystem mounted from a particular NFS server. Global options apply to all NFS mounts from this client.

To set default options for all NFS mounts for your systems, you can uncomment the `NFS-Mount_Global_Options` block. In that block, you can set such things as protocols and NFS versions as well as transmission rates and retry settings. Here is an example of an `NFSMount_Global_Options` block:
```
[ NFSMount_Global_Options ]
# This sets the default version to NFS 4
Defaultvers=4
# Sets the number of times a request will be retried before generating a timeout
Retrans=2
# Sets the number of minutes before retrying a failed mount to 2 minutes
Retry=2
```
In the example above, the default NFS version is 4. Data is retransmitted twice (2) before generating a time-out. The wait time is 2 minutes before retrying a failed transmission.

#### Using autofs to mount NFS filesystems on demand

`autofs` (short for *automatically mounted filesystems*), mounts network filesystems on demand when someone tries to use the filesystems.
- On RHEL-based systems, install with: `yum install autofs`
- On Ubuntu or Debian-based systems: `apt-get install autofs`

With `autofs` enabled, if you know the hostname and directory being shared by another host computer, simply change to the `autofs` mount directory (`/net` or `/var/autofs` by default). This causes the shared resource to be automatically mounted and made accessible to you. The following steps explain how to turn on the `autofs` facility in RHEL-based systems:
1. As the root user, open the `/etc/auto.master` file and look for the following line:
  - `/net -hosts`
  - This causes the `/net` directory to act as the mount point for the NFS shared directories that you want to access on the network. (If there is a comment character at the beginning of the line, remove it).
2. To start the `autofs` service on a `systemd`-based system:
  - `systemctl start autofs.service`
3. Enable the service to start on boot:
  - `systemctl enable autofs`

If you know that the `/usr/local/share` directory is being shared from the computer on your network named `remote`, you can do the following:
- `cd /net/shuttle/`

You can use any name or IP address that identifies the location of the NFS server computer. For example, instead of `shuttle`, you might have used `shuttle.example.com` or an IP address such as `192.168.0.2`.

The following procedure illustrates how to set up a user account on an NFS server and share the home directory of a user named `user` from that server so that it can be automounted when `user` logs into a different computer. In this example, instead of using a central authentication server, matching accounts are created on each system.
1. On the NFS server (`nfs.example.com`) that provides a centralized user home directory for the user named `user`, create a user account for `user` with a home directory of `/home/shared/user` as its name. Also find `user`'s user ID number from the `/etc/passwd` file (third field) so that you can match it when you set up a user account for `user` on another system.
  - `mkdir /home/shared`
  - `useradd -c "User" -d /home/shared/user user`
2. On the NFS server, export the `/home/shared/` directory to any system on your local network (I use `192.168.0.*`), so that you can share the home directory for `user` and any other users you create by adding this line to the `/etc/exports` file:
  - `# /etc/exports file to share directories under /home/shared only to other systems on the 192.168.0.0/24 network:`
  - `/home/shared 192.168.0.* (rw,insecure)`
  - In the exports file example above, the `insecure` option allows clients to use ports above port 1024 to make mount requests. Some NFS clients require this because they do not have access to NFS-reserved ports.
3. On the NFS server, restart the `nfs-server` service, or if it is already running, you can simply export the shared directory as follows:
  - `exportfs -a -r -v`
4. On the NFS server, make sure that the appropriate ports are open on the firewall.
5. On the NFS client system, add an entry to the `/etc/auto.master` file that identifies the mount point where you want the remote NFS directory to be mounted and a file (of your choosing) where you will identify the location of the remote NFS directory. Below is an example entry in the `auto.master` file:
  - `/home/remote /etc/auto.user`
6. On the NFS client system, add an entry to the file you just noted (`/etc/auto.user` is what I used) that contains an entry like the following:
  - `user     -rw     nfs.example.com:/home/shared/user`
7. On the NFS client system, restart the `autofs` service:
  - `systemctl restart autofs.service`
8. On the NFS client system, create a user named `user` using the `useradd` command. For that command line, you need to get the UID for `user` on the server (501 in this example) so that `user` on the client system owns the files from `user`'s NFS home directory. When you run the following command, the `user` user account is created, but you will see an error message stating that the home directory already exists (which is correct):
  - `useradd -u 501 -c "User" -d /home/remote/user user`
  - `passwd user`
9. On the NFS client system, log in as `user`. If everytihng is working properly, when `user` logs in and tries to access his home directory (`/home/remote/user`), the directory `/home/share/user` should be mounted from the `nfs.example.com` server.

### Unmounting NFS filesystems

After an NFS filesystem is mounted, it can be unmounted using the `umount` command:
- `umount remote:/stuff`
- `umount /mnt/stuff`
- If you get the message `device is busy` when you try to unmount a filesystem, it means that the unmount failed because the filesystem is being access.
- If an NFS filesystem doesn't unmount, you can force it (`umount -f /mnt/stuff`) or unmount and clean up later (`umount -l /mnt/stuff`). The `-l` option is usually the better choice because a forced unmount can disrupt a file modification that is in process.
  - Run `fuser -v <mountpoint>` to see what users are holding your mounted NFS share open and then `fuser -k <mountpoint>` to kill all of those processes.
***
