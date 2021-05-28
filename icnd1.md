# Notes for the CCNA 200-125 Routing and Switching Test, book ICND1.
***

## TCP/IP Model

| Application |
| ----------- |
| **Transport** |
| **Network** |
| **Data Link** |
| **Physical** |

- Defined in RFC 1122
- Adjacent-Layer Interaction: The concept of how adjacent layers in a networking model, on the same computer, work together.
- Same-Layer Interaction: When a particular layer on one computer wants to communicate with the same layer on another computer, the two computers use headers to hold the information that they want to communicate.
- Encapsulation: The placement of data from a higher-layer protocol behind the header, or between a header and trailer, of the next-lower-layer protocol.
- De-encapsulation: On a computer that receives data over a network, the process in which the device interprets the lower-layer headers and, when finished with each header, removes the header, revealing the next-higher-layer PDU.
- Application Layer: Provides services to the application software running on a computer.
  - HTTP, POP3, SMTP
- Transport Layer: Provides services to the application layer protocols that reside on the Application layer
  - TCP, UDP
- Network Layer: Provides the services of the IP protocol including addressing and routing.
- Link Layer: Refers to the physical connections, or links, between two devices and the  protocols used to control those links.
***

## OSI Model

| Application |
| ----------- |
| **Presentation** |
| **Session** |
| **Transport** |
| **Network** |
| **Data Link** |
| **Physical** |

- Application Layer: Provides an interface from the application to the network by supplying a protocol with actions meaningful to the application.
  - Telnet, HTTP, FTP, SMTP, POP3, VoIP, SNMP
  - Hosts, firewalls
- Presentation Layer: This layer negotiates data formats, such as ASCII text, or image types like JPEG
- Session Layer: This layer provides methods to group multiple bidirectional messages into a workflow for easier management and easier back-out of work that happened if the entire workflow fails.
- Transport Layer: In function, much like the TCP/IP's transport layer. This layer focuses on data delivery between the two endpoint hosts (e.g. error recovery).
  - TCP, UDP
  - Hosts, firewalls
- Network Layer: Like the TCP/IP network (internet) layer, this layer defines logical addressing, routing (forwarding), and the routing protocols used to learn routes.
  - IP
  - Routers
- Data Link Layer: Like the TCP/IP data link layer, this layer defines the protocols for delivering data over a particular single type of physical network (for example, the Ethernet data link protocols).
  - Ethernet (IEEE 802.3), HDLC
  - LAN switch, wireless access point, cable modem, DSL modem.
- Physical Layer: This layer defines the physical characteristics of the transmission medium, including connectors, pins, use of pins, electrical currents, encoding, light modulation, and so on.
  - RJ-45, Ethernet (IEEE 802.3)
  - LAN hub, LAN repeater, cables.
- Protocol Data Unit (PDU): represents the bits that include the headers and trailers for a particular layer, as well as the encapsulated data.
  - IP Packet, Ethernet Frame, TCP/UDP Segment, etc.
***

## Ethernet

- A family of standards in regards to copper cable LANs and WANs.
  - 802.3 IEEE standard
- Uses the same layer 2 frame format over all link types.
  - Header and Trailer
  - MAC Addresses
- Crosstalk: When the EMI of one cable interferes with the cables around it.
  - Solved by twisting the cables together.

| Speed | Common Name | Informal IEEE Standard Name | Formal IEEE Standard Name | Cable Type, Maximum Length |
| ----- | ----------- | --------------------------- | ------------------------- | -------------------------- |
| 10 Mbps | Ethernet | 10Base-T | 802.3 | Copper, 100m |
| 100 Mbps | Fast Ethernet | 100Base-T | 802.3u | Copper, 100m |
| 1000 Mbps | Gigabit Ethernet | 1000Base-LX | 802.3z | Fiber, 5000m |
| 1000 Mbps | Gigabit Ethernet | 1000Base-T | 802.3ab | Copper, 100m |
| 10 Gbps | 10 Gig Ethernet | 10GBase-T | 802.3an | Copper, 100m | The user PC will not have any access to the corporate network through the switch.

- Straight-Through Cable: Used when connecting two different networking devices
- Cross-over cable: Used when connecting like devices
  - This is done because the devices use the same pins in a cable for transmitting and receiving.

| Transmits on Pins 1 & 2 | Transmits on Pins 3 & 6 |
| ----------------------- | ----------------------- |
| PC NICs | Hubs |
| Routers | Switches |
| Wireless AP Ethernet Interface | -- |

- This table only applies if the devices are using 10Base-T or 100Base-T cabling.
  - Cisco Switches have a feature called *auto-mdix* that notices when a wrong cable is being used and automatically changes its logic to make the link work.
- For 1000Base-T cabling, four wire pairs are required, otherwise, the general idea for straight-through and crossover cabling is the same.

### Ethernet Data-Link Protocols

- The Ethernet frame consists of the following parts:

| Field | Bytes | Description |
| ----- | ----- | ----------- |
| Preamble | 7 | Synchronization |
| Start Frame Delimiter (SFD) | 1 | Signifies that the next byte begins the Destination MAC Address field. |
| Destination MAC Address | 6 | Identifies the intended recipient of this frame. |
| Source MAC Address | 6 | Identifies the sender of the frame. |
| Type | 2 | Defines the type of protocol listed inside the frame; today, most likely identifies IP version 4 (IPv4) or IP version 6 (IPv6). |
| Data and Pad | 46-1500 | Holds data from a higher layer, typically an L3PDU (usually an IPv4 or IPv6 packet). The sender adds padding to meet the minimum length requirement for this field (46 bytes). |
| Frame Check Sequence (Trailer) | 4 | Provides for the receiving NIC to determine whether the frame experienced transmission errors. |

- MAC (Media Access Control) Addresses: 6 byte long binary numbers.
  - For networking devices, most MACs represent a single NIC or other Ethernet port. These are known as *unicast* Ethernet addresses.
  - All devices, upon manufacture, are assigned a universally unique MAC address.
  - The IEEE assigns each organization a unique 3-byte code, called the organizationally unique identifier (OUI) which makes up the first half of the MAC address.
  - The manufacturer themselves assign each device a unique address for the last half of the device's MAC address.
  - Broadcast Address: Frames sent to this address should be delivered to all devices on the Ethernet LAN. It has the value of FFFF.FFFF.FFFF
  - Multicast Address: Frames sent to a multicast address will be copied and forwarded to a subset of devices on the LAN that volunteers to receive frames sent to a specific multicast address.
- Common EtherTypes:
  - IPv4: 0800
  - IPv6: 86DD
- If the FCS results are different for the receiver than what the sender initially calculated, the frame is discarded.
- Half Duplex: The devices must wait to send if it is currently receiving a frame; in other words, it cannot send and receive data at the same time.
- Full Duplex: The device does not have to wait before sending; it can send and receive at the same time.
- Hubs repeat all signals received out all other connected ports, except the one that received the signal.
- CSMA/CD (Carrier Sense Multiple Access with Collision Detection):
  1. A device with a frame to send listens until the Ethernet is not busy.
  2. When the Ethernet is not busy, the sender begins sending the frame.
  3. The sender listens while sending to discover whether a collision occurs; collisions might be caused by many reasons, including unfortunate timing. If a collision occurs, all currently sending nodes do the following:
    - They send a jamming signal that tells all nodes that a collision happened.
    - They independently choose a random time to wait before trying again, to avoid unfortunate timing.
    - The next attempt starts again at step 1.
***

## Fundamentals Of WANs

- Leased-Line WANs: works similar to an Ethernet crossover cable to connect two routers in that it connects one LAN with the rest of an organization's TCP/IP network.
  - Delivers traffic in both directions at a predetermined speed using full-duplex logic.
  - Uses two pairs of wires, one pair for each direction of sending data.
  - Telecomms put their equipment in buildings called central offices (CO). They install cables from the CO to most every other building in a city.
  - Customer Premises Equipment (CPE): Teleco company property on a customer's site that connects the customer to the telecomm WAN. Normally involves the router, serial interface card, and CSU/DSU (channel service unit/data service unit).
  - Data Terminal Equipment (DTE) cables: serial cables normally used between a router and an external CSU/DSU.
  - Data Communication Equipment (DCE) cable: has a female connector to attach to a DTE.
- High-Level Data Link Control (HDLC): Layer 2 protocol used by lease-lines to send frames from one end of a line to another.
  - Cisco has its own proprietary version.

| HDLC Field | Ethernet Equivalent | Description |
| ---------- | ------------------- | ----------- |
| Flag | Preamble, SFD | Lists a recognizable bit pattern so that the receiving nodes realize that a new frame is arriving. |
| Address | Destination Address | Identifies the destination device. |
| Control | N/A | Mostly used for purposes no longer in use today for links between routers. |
| Type | Type | Identifies the type of Layer 3 packet encapsulated inside the frame. |
| FCS | FCS | A field used by the error detection process. (It is the only trailer field in this table.) |

- Ethernet WANs: Uses fiber ethernet links to connect to Points of Presence (PoP). These are typically ethernet switches that connect a client to a service provider's ethernet WAN.
  - Ethernet emulation or Ethernet over MPLS (EoMPLS) (Multiprotocol Label Switching): Ethernet WAN service that acts like one Ethernet link.
    - Provides a point-to-point connection between two customer devices
    - Behavior as if a fiber Ethernet link existed between the two devices.
- Internet Core: middle of the internet that is owned and operated by ISP's.
  - Consists of LANs and WANs.
- Internet Access Link: A WAN link used to connect a client to one of the ISP's that provide internet access.
  - All internet access lines use a pair of routers, one at the customer's side and one at the ISP side.
- Digital Subscriber Line (DSL): creates a short (miles long) high-speed link WAN between a Telecomm customer and an ISP.
  - Uses the same single-pair telephone line used for a typical home phone line.
  - DSL Access Multiplexer (DSLAM): Splits out the data over a router which completes the connection to the internet. It also splits out the voice signals over the voice switch used by phones.
- Public Switched Telephone Network (PSTN): The worldwide voice network for telephones.
- Cable Internet: uses existing cable TV cables to send data using asymmetric speeds.
  - Generally faster than DSL.
- **Dense Wavelength-division multiplexing (DWDM)**: multiplies the amount of bandwidth that a single strand of fiber can support.
  - Used in all modern submarine communications cable systems and other long-haul circuits.
  - Supports both the SDH and SONET standards.
  - Allows a single strand of fiber to support bidirectional communications.
***

## Fundamentals of IPv4 Addressing and Routing

- Network Layer Routing (Forwarding) Logic: Host OS's send IP Packets to nearby routers who then decide where the packet should go next.
  - Path Selection: the routing process and/or protocols that selects the best route to the desired destination.
  - The default router used by devices in a LAN is referred to as a default gateway.
  - Each router keeps an IP routing table that lists IP address groupings, called IP networks and IP subnets.
- ARP (Address Resolution Protocol): dynamically learns the data-link address of an IP host connected to a LAN.
  - Used by a router to find a device in it's LAN when it only has the IP address but not the required Layer 2 information.
- IP Packets have a 20 byte header with source and destination IP addresses.
- Routing protocols allow routers that share connections to learn the routes to each other.
- IP Packet Header:
  1. Version
  2. Length
  3. DS Field
  4. Packet Length
  5. Identification
  6. Flags
  7. Fragment Offset
  8. Time to Live (TTL)
  9. Protocol
  10. Header Checksum
  11. Source IP Address
  12. Destination IP Address

### IPv4 Addressing

- IP Host: any device that has at least one interface with an IP address that can send and receive IP packets.
- IP addresses consist of a 32-bit number written in dotted decimal notation with four decimal octets each representing 8 bits.
  - The range of numbers in each octet is 0 to 255.
  - All IP addresses in the same group must not be separated from each other by a router.
  - IP addresses separated from each other by a router must be in different groups.
- Class A IP Addresses: 1 - 126 in the first octet.
  - 8 network bits, 24 host bits by default.
- Class B IP Addresses: 128 - 191 in the first octet.
  - 16 network bits, 16 host bits by default.
- Class C IP Addresses: 192 - 223 in the first octet.
  - 24 network bits, 8 host bits by default.
  - These all define unicast addresses that correspond with a single host interface.
- Class D IP Addresses: 224 - 239 in the first octet. Used for multicast addresses.
- Class E IP Addresses: 240 - 255 in the first octet. Reserved for future use.
- Network ID (Identifier): A reserved octet value per network that defines the IP network.
  - Cannot be used by a host as an IP address.
  - Consists of the set network bits and the unset host bits, e.g: 8.0.0.0
- Subnetting: the method used to further subdivide the IPv4 address space into groups that are smaller than a single IP network.

### IPv4 Routing

- Host routing logic:
  1. If the destination IP address is in the same IP subnet as the sending host, then send the packet directly to the destination host.
  2. Otherwise, send the packet to the default gateway.
- Router Forwarding Logic:
  1. Use the data-link FCS field to ensure that the frame had no errors. If errors occurred, discard the frame.
  2. Assuming that the frame was not discarded at Step 1, discard the old data-link header and trailer, leaving the IP packet.
  3. Compare the IP packet's destination IP address to the routing table, and find the route that best matches the destination address. This route identifies the outgoing interface of the router, and possibly the next hop router IP address.
  4. Encapsulate the IP packet inside a new data-link header and trailer, appropriate for the outgoing interface, and forward the frame.
- Goals of a routing protocol:
  - To dynamically learn and fill the routing table with a route to each subnet in the internetwork.
  - If more than one route to a subnet is available, to place the best route in the routing table.
  - To notice when routes in the table are no longer valid, and to remove them from the routing table.
  - If a route is removed from the routing table and another route through another neighboring router is available, to add the route to the routing table.
  - To work quickly when adding new routes or replacing lost routes.
  - To prevent routing loops.
- Convergence time: the time between losing a route and finding a working replacement.
- Steps used by routing protocols generally to learn new routes:
  1. Each router, independent of the routing protocol, adds a route to its routing table for each subnet directly conected to the router.
  2. Each router's routing protocol tells its neighbors about the routes in its routing table, including the directly connected routes and routes learned from other routers.
  3. After learning a new route from a neighbor, the router's routing protocol adds a route to its IP routing table, with the next hop router of that route typically being the neighbor from which the route was learned.
- Domain Name System (DNS): Dynamically resolves hostnames to IP addresses to direct IP traffic.
  - DNS queries are sent to a DNS server which sends back a DNS reply that lists the hostname's IP address.
- Address Resolution Protocol (ARP) uses ARP Requests to ask what MAC address belongs to an IP Address and ARP Reply to list both the original IP address and the matching MAC address. Can be used by both hosts and routers.
  - Results are kept inside an ARP cache or ARP table.
- Ping (Packet Internet Groper): uses the Internet Control Message Protocol (ICMP) to send an ICMP echo request to another IP address and, if successful, the sending PC should receive an ICMP echo reply. If this doesn't happen it means that the sending PC cannot reach the destination PC for various reasons.
***

## TCP and UDP

- The fundamental difference between TCP and UDP is that TCP provides a wide variety of services to applications, whereas UDP does not.
  - TCP provides re-transmission (error recovery) and helps avoid congestion with flow control.
  - VoIP and Video over IP both use UDP as they do not need error recovery.
- Features supported by TCP/UDP:
- The layer 4 PDU is referred to as a segment.

| Function | Description |
| -------- | ----------- |
| Multiplexing using ports | Function that allows receiving hosts to choose the correct application for which the data is destined, based on the port number. |
| Error recovery (reliability) | Process of numbering and acknowledging data with Sequence and Acknowledgment header fields |
| Flow control using windowing | Process that uses window sizes to protect buffer space and routing devices from being overloaded with traffic. |
| Connection establishment and termination | Process used to initialize port numbers and Sequence and Acknowledgment fields. |
| Ordered data transfer and data segmentation | Continuous stream of bytes from an upper-layer process that is "segmented" for transmission and delivered to upper-layer processes at the receiving device, with the bytes in the same order. |

### Transmission Control Protocol (TCP)

- Defined in RFC 793
- TCP relies on IP for end-to-end delivery of the data, including routing issues.
- TCP Header fields:
  1. Source Port
  2. Destination Port
  3. Sequence Number
  4. Acknowledgment Number
  5. Offset
  6. Reserved
  7. Flag Bits
  8. Window
  9. Checksum
  10. Urgent
- TCP uses multiplexing to determine which application receives what traffic based on destination port numbers.
- Multiplexing relies on a concept called a socket which consists of three things:
  1. An IP Address
  2. A transport protocol
  3. A port number
  - For example, a web application socket would be (10.1.1.2, TCP, port 80).
- Well known port numbers are 1 - 1023.
  - Well known port numbers are used by servers to host services like FTP.
  - Other port numbers are used by clients.
- Popular TCP/IP Applications:

| Port Number | Protocol | Application |
| ----------- | -------- | ----------- |
| 20 | TCP | FTP Data |
| 21 | TCP | FTP Control |
| 22 | TCP | SSH |
| 23 | TCP | Telnet |
| 25 | TCP | SMTP |
| 53 | TCP/UDP | DNS |
| 67 | UDP | DHCP Server |
| 68 | UDP | DHCP Client |
| 69 | UDP | TFTP |
| 80 | TCP | HTTP (WWW) |
| 110 | TCP | POP3 |
| 161 | UDP | SNMP |
| 443 | TCP | SSL |
| 514 | UDP | Syslog |

- TCP Connection Establishment: the process of initializing Sequence and Acknowledgment fields and agreeing on the port numbers used.
  - The three-way handshake: SYN, SYN/ACK, ACK
  - SYN means "synchronize the sequence numbers"
- Connection Termination: a four-way sequence, ACK/FIN, ACK, ACK/FIN, ACK
- Connection Oriented Protocol: A protocol that requires an exchange of messages before data transfer begins, or that has a required pre-established correlation between two endpoints.
- Connectionless Protocol: A protocol that does not require an exchange of messages and that does not require a pre-established correlation between two endpoints.
- Error Recovery and Reliability: TCP numbers data bytes using the Sequence and Acknowledgment fields in the TCP header. Using Sequence in one direction and Acknowledgment in another.
- Forward Acknowledgment: The convention of acknowledging by listing the next expected byte, rather than the number of the last byte received.
- Flow Control using Windowing: the window concept is where TCP determines the amount of data that can be outstanding and awaiting acknowledgment at any one point in time.
  - This allows the receiving host to tell the sender how much data it can receive at any given time.
  - The receiver can slide the window size up and down, known as sliding or dynamic window.

### User Datagram Protocol (UDP)

- Provides a service for applications to exchange messages.
- UDP is connectionless and provides no reliability, no windowing, no re-ordering of the received data, and no segmentation of large chunks of data into the right size for transmission.
- Provides functions such as data transfer and multiplexing using port numbers.
- UDP Header format:
  1. Source Port
  2. Destination Port
  3. Length
  4. Checksum

### TCP/IP Applications

- Web servers: consist of web server software running on a computer, store information (in the form of web pages) that might be useful to different people.
- Web browser: software installed on an end user's computer, provides the means to connect to a web server and display the web pages stored on the web server.
- Links: clickable items on a web page which in turn bring you to another web page.
- Uniform Resource Identifier (URI): What a link actually refers to. Three key components:
  1. Scheme (http://)
  2. Authority (www.certskills.com)
  3. Path (/blog)
- Before a browser can send packets to a web server, it has to resolve the Authority's IP address by sending a DNS request (UDP port 53) to a DNS server.
- Hyper text transfer protocol (HTTP): defines how files can be transferred between two computers, specifically between web servers and web clients.
  - Uses the HTTP GET request to request a file rom a web server.
  - The web server sends a return code of 200 with the file's contents.
  - The web server sends the initial web page object as well as all other objects mentioned in URI's on the initial web page requested.
- Layer 3 packet headers use 6 in the protocol field to identify TCP and 17 to identify UDP.
***

## Cisco Catalyst Switch CLI

- Cisco refers to a switch's physical connectors as either interfaces or ports, with an interface type and interface number.
  - The interface type, as used in commands on the switch, is either 10BaseT, 100BaseT, 1000BaseT or so on.
  - Some catalyst switches use a two-digit interface number (x/y), while others have a three-digit number (x/y/z).
- The Cisco networking device operating system is called the Internetwork Operating System (IOS).
- The CLI can be accessed through three methods:
  1. The console
  2. Telnet
  3. SSH
- Console port: on modern devices is just a usb port marked console. On older devices a console cable connected to either a serial port on a host machine or combining with a USB converter is needed.
- Default console port settings for terminal emulator software:
  - 9600 bits/second
  - No hardware flow control
  - 8-bit ASCII
  - No parity bits
  - 1 stop bit
- When accessing the CLI for the first time, you are placed in EXEC mode, also known as user mode.
  - This has limited privileges.
  - Has a `>` after the username for a command prompt.
- The next level of user is called enable mode, also known as privileged mode or privileged exec mode.
  - Accessed by entering `enable` from exec mode.
  - Has a `#` after the username for a command prompt.
- Normally, a switch doesn't require password access for enable mode for someone accessing it from a console port.
- Password protection can be configured at two points in the login process from the console:
  1. When the user connects from the console
  2. When any user moves to enable mode.
- In a running config, accessed with the `show running-config` command, lines that begin with `!` are comments.
- The `enable secret <word>` line in a config is the password all users must enter to access enable mode.
- Command line help:

| What you enter | What help you get |
| -------------- | ----------------- |
| ? | Help for all commands available |
| command ? | With a space between the command and the ?, the switch lists text to describe all the first parameter options for the command. |
| com? | A list of commands that start with com. |
| command parm? | Lists all parameters beginning with the parameter typed so far. |
| command parm <TAB> | Pressing the Tab key causes IOS to spell out the rest of the word, assuming that you have typed enough of the word so there is only one option that begins with that string of characters. |
| command parm1 ? | If a space is inserted before the question mark, the CLI lists all the next parameters and gives a brief explination of each. |

- The amount of help displayed depends on the current user context. (EXEC vs. Enable)
- Access the command history:
- The `show` command can display information about almost every aspect of a cisco device.
  - `show mac address-table dynamic`
- The `debug` command tells the user about the operation of the switch in the form of a live update feed, issuing messages any user can see.
- The `password` command defines the password the console user must type when prompted.
- The `login` command tells IOS to perform simple password checking at the console.
- The `reload` command tells the switch to reinitialize or reboot Cisco IOS. Can only be done from enabled mode.
- The `disable` command switches from enable mode to user mode.
- The `end` command switches from a configuration mode to enable mode.
- The `quit` command ends a CLI session.
- The `do` command put before any EXEC level command in configuration contexts will run as if in EXEC mode.

### Configuring Cisco IOS Software

- Configuration Mode: A user context that accepts configuration commands.
  - These changes occur immediately each time enter is pressed after a command.
- Configuration mode consists of the initial mode or global configuration mode, and subcommand modes.
  - Context setting commands move from one subcommand mode to another.
- To get to the global configuration mode, type: `configure terminal`. (Sometimes just `config t`)
- The `interface` command is used to enter the configuration context for a specified interface.
  - `interface FastEthernet 0/1`
  - `line console 0` (console port)
    - `password hope` (sets the password to hope for the console port)
- Type `exit` to leave the configuration subcontext back to the global context.
- Common configuration mode prompts:

| Prompt | Name of Mode | Context-Setting Command(s) to reach this mode |
| ------ | ------------ | --------------------------------------------- |
| hostname(config)# | Global | None--first mode after `configure terminal` |
| hostname(config-line)# | Line | `line console 0`, `line vty 0 15` |
| hostname(config-if)# | Interface | `interface type number` |
| hostname(vlan)# | VLAN | `vlan number` |

- The `end` command leaves global configuration mode and back into enable mode.
- The four types of memory found in a switch:
  1. RAM (The running active config is stored here)
  2. Flash Memory (Stores the default Cisco IOS image)
  3. ROM (Stores the bootstrap program loaded at power up).
  4. NVRAM (Non-volatile RAM. Stores the initial startup configuration file)
- Two configuration files used by switches:
  1. startup-config: stores the initial configuration used anytime the switch reloads. Stored in NVRAM
  2. running-config: Stores the currently used configuration commands. Stored in RAM.
- To display the startup config, use the `show startup-config` command.
- The exec mode command: `copy running-config startup-config` copies over the startup configuration file with the current running config.
- To revert the running config to the startup config without reloading, type `copy startup-config running-config`
- To get rid of all existing configurations in order to start with a clean configuration, run the following commands:
  - `write erase`
  - `erase startup-config`
  - `erase nvram:`
- Rebooting after removing the startup config will reload the switch with an empty configuration.
- There is no command to delete the running config. You can only erase the startup config and reload the switch.
***

## LAN Switching Concepts

- The primary purpose of a LAN switch is to forward ethernet frames to the destination MAC address.
- The three actions of a switch using switching logic:
  1. Deciding when to forward a frame or when to filter (or not forward) a frame, based on the destination MAC address
  2. Preparing to forward frames by learning MAC addresses by examining the source MAC address of each frame received by the switch.
  3. Preparing to forward only one copy of the frame to the destination by creating a (layer 2) loop-free environment with other switches by using Spanning Tree Protocol (STP).
- Switches compare a frame's destination MAC address to its MAC address table to decide whether to forward the frame or ignore it.
  - If the MAC is reachable it will send it out of the port that can reach the destination and choose not to send the frame out of other ports.
  - Also known as the forward-versus-filter decision
- Switches build their MAC address table by listening to incoming frames and examining the source MAC address. It assigns that address with the interface it came in on to know how to reach it in the future.
- Flooding: when a switch receives a frame and does not know where it's destination MAC is located. It sends the frame out of all interfaces except the one it came in on.
- Broadcast Frames: Frames with the destination MAC of FFFF.FFFF.FFFF which get sent to all connected devices.
- Spanning Tree Protocol: prevents looping frames by blocking some ports from forwarding frames so that only one active path exists between any pair of LAN segments.
- Default settings of a Catalyst Switch:
  1. The interfaces are enabled by default, ready to start working once a cable is connected.
  2. All interfaces are assigned to VLAN1.
  3. 10/100 and 10/100/1000 interfaces use auto-negotiation by default
  4. The MAC learning, forwarding, flooding logic all works by default.
  5. STP is enabled by default.
- To list all known MAC addresses and ensure that the switch is working correctly, run the `show mac address-table` command.
- To see all the dynamically learned MAC address only, run the `show mac address-table dynamic` command.
- To see the status of all interfaces, enter the `show interfaces status` command.
- To see all MAC's associated with the Fa0/1 interface run the `show mac address-table dynamic interface fastEthernet 0/1` command.
- The default for removing an unused MAC on a switch MAC table is 300 seconds.
  - The inactivity timer for each MAC entry is reset to 0 if a new frame arrives with that MAC as the source address.
- If the MAC table fills, the oldest entries are removed first.
- CAM (Content-addressable memory): a physical memory that holds the MAC address table on the switch.

***

## Configuring Basic Switch Management

- Data Plane: the work a switch does to forward frames generated by the devices connected to the switch.
- Control Plane: the configuration and processes that control and change the choices made by the switch's data plane.
- Management Plane: The management of the device itself.

### Securing the Switch CLI

- By default, a Cisco catalyst switch allows anyone to connect to the console port, access user mode, and then move to enable and configuration modes without any kind of security.
- Remotely accessing a switch requires a switch's IPv4 configuration to be setup and working.
- Shared password: Uses just a shared password with no username to access user mode on a switch.
  - One password for console login and another for remote login.
- Telnet passwords are known as vty passwords as they sit in the vty line configuration mode.
- Enable mode can be protected with its own password, known as the enable password.
- To configure a console password as hope, run the following commands:
  - `line console 0`
  - `login`
  - `password hope`
- To configure a vty password as love, run the following commands:
  - `line vty 0 15`
  - `login`
  - `password love`
- To configure the enable password as reason, run the following command in exec mode:
  - `enable secret reason`
- The `login` command tells IOS to setup the current line security with a shared password.
- Cisco switches support username/password combinations for console, telnet, and ssh access, but only has one shared secret for enable mode.
- To add a username and password to a current configuration line context, use the `username` <username> `secret` <password> commands.
  - `line con 0`
  - `login local`
  - `username wendall secret odom`
- The above commands set the username wendall with the password odom to login with console access. `login local` tells IOS to require a username and password.
- A cisco switch can be configured to work with an Authentication, Authorization, and Accounting (AAA) server, such as RADIUS or TACACS+.
- The login local configurations for Telnet also apply to SSH.
- To actually set up SSH as an option on a switch, use the following configuration commands:
  - `hostname <hostname>`
  - `ip domain-name <word.com>`
  - `crypto key generate rsa`
  - `modulus 2048`
- SSH servers use the fully qualified domain name (FQDN) of the switch as input to create the traffic encryption keys.
- To disable telnet, ssh, all or none, use the following commands:
  - `transport input all`
  - `transport input telnet`
  - `transport input ssh`
  - `transport input none`
- To tell the switch what version of ssh to use, run the following command:
  - `ip ssh version #`
- To display the current status of SSH, run the `show ip ssh` command.
- To display the currently connected SSH users, run the `show ssh` command.

### Enabling IPv4 for Remote Access

- A switch needs an IP address to be accessed remotely. This is assigned to a single interface managed by a virtual NIC.
- Switched Virtual Interface (SVI): Also known as a VLAN interface, used to manage the interfaces on a switch.
- In order for a switch to be able to communicate outside of its local subnet it needs to have a default gateway set.
- To setup an IPv4 address on a Switch interface, run the following commands:
  - `configure terminal`
  - `interface vlan #` (Choose the vlan the ip address will rest on)
  - `ip address <ip> <ip subnet mask>` (assign the ip address and subnet mask)
  - `no shutdown` (enables the interface)
  - `exit`
  - `ip default-gateway <ip for local router>` (Tells the switch the IP of it's local router)
- If DHCP is setup on the network, you can enable DHCP address learning on an interface with the following commands:
  - `configure terminal`
  - `interface vlan #`
  - `ip address dhcp`
  - `no shutdown`
  - `CTRL+Z` (Return to enable mode)
- To view the status of interfaces assigned to a specific VLAN, run the `show interfaces vlan #` command.
- To see the currently leased DHCP addresses, run the `show dhcp lease` command.
- To see the default gateway, run the `show ip default gateway`
- To see the current command history, run the `show history` command.
- To set the history buffer size for a terminal session, use the `terminal history size #` command.
- To set the default history size, run the `history size #` command.
- To stop IOS from displaying log messages in the console, run the `no logging console` command.
- To enable IOS to display logging messages to the console, run the `logging console` command.
- To tell IOS to only display logging messages when you are not in the middle of a command, enter the `logging synchronous` command.
- By default, a switch has a 5 minute terminal timeout setting for login sessions. To change this, use the `exec-timeout <minutes> <seconds>` command.
  - `exec-timeout 0 0` sets sessions to never timeout.
- To disable DNS name resolution, use the `no ip domain-lookup` command.
***

## Configuring Switch Interfaces

- To change the speed, description, and duplex settings of an interface, run the following commands:
  - `configure terminal`
  - `interface <interface>`
  - `duplex {auto | full | half }`
  - `speed {auto | 10 | 100 | 1000 }`
  - `description <description string>`
  - `exit`
- You can also configure the description for a range of interfaces with the following command example:
  - `interface range FastEthernet 0/11 - 20`
  - `description <description string>`
  - `exit`
- The duplex command sets the duplex mode for an interface.
- The speed command sets the default speed for an interface.
- The description command associates a string with an interface or range of interfaces.
- The range command when used with the interface command can be used to enter a subcontext to configure a group of interfaces at once.
  - Can be done if all interfaces referenced are numbered consecutively.
- You can revert changes done by a particular command by putting `no` infront of it.
  - For example, `no duplex` resets an interface's duplex settings back to the default.
- Auto-negotiation: for interfaces that can run at different speeds, Cisco switches with interfaces set to auto will attempt to automatically determine the speed and duplex setting to use.
  - Ensure that both ends are set to use autonegotiation for it to work properly.

### Port Security

- Port security is used to ensure only devices with approved MAC addresses are allowed to forward frames through a switch.
- Switches enable port security per port.
  - Each port has a maximum number of allowed MAC addresses.
- The default port security action is discarding all future incoming traffic on the offending port.
- Common port security ideas:
  - Define a maximum number of source MAC addresses allowed for all frames coming in the interface.
  - Watch all incoming frames, and keep a list of all source MAC addresses, plus a counter of the number of different source MAC addresses.
  - When adding a new source MAC address to the list, if the number of MAC addresses pushes past the configured maximum, a port security violation has occurred. The switch takes action (the default action is to shut down the interface).
- Sticky secure MAC addresses: a method to discover the MAC addresses used off each port and store them in a port security configuration in the running config file.
- Steps to enable and configure port security on an interface:
  1. Make the switch interface either a static access or trunk interface using the `switchport mode access` or the `switchport mode trunk` interface subcommands, respectivley.
  2. Enable port security using the `switchport port-security` command.
  3. (Optional) Override the default maximum number of allowed MAC addresses associated with the interface (1) by using the `switchport port-security maximum #` command.
  4. (Optional) Override the default action to take upon a security violation (shutdown) using the `switchport port-security violation {protect | restrict | shutdown}` command.
  5. (Optional) Predefine any allowed source MAC addresses for this interface using the `switchport port-security mac-address <mac-address>` command. You can use this command multiple times for multiple MAC's.
  6. (Optional) Tell the switch to "sticky learn" dynamically learned MAC addresses with the `switchport port-security mac-address sticky` command.
- To verify port security settings, use the `show port-security interface` command.
- Port security can perform three different actions with different behaviors as described in the table below:

| Option on the switchport port-security violation Command | Protect | Restrict | Shutdown |
| -------------------------------------------------------- | ------- | -------- | -------- |
| Discards offending traffic | Yes | Yes | Yes |
| Sends log and SNMP messages | No | Yes | Yes |
| Increments the violation counter for each violating incoming frame | No | Yes | Yes |
| Disables the interface by putting it in an err-disabled state, discarding all traffic | No | No | Yes |

- To bring an interface out of the err-disabled state, the `shutdown` and `no shutdown` commands have to be used.
- The `show mac address-table secure` command lists MAC addresses associated with ports that use port security.
- The `show mac address-table static` command lists MAC addresses associated with ports that use port security as well as any other statically defined MAC addresses.
***

## Analyzing Ethernet LAN Designs

- An Ethernet HUB receives traffic on one port and forwards it out all other ports.
- Collision Domain: The set of NICs and device ports for which if they sent a frame at the same time, the frames would collide.
- Ethernet Transparent Bridges: Sit between hubs and divide the network into multiple collision domains.
- For a LAN switch, each single link to the switch is considered its own collision domain.
- Full duplex cabling eliminates the need to worry about collisions.
- Broadcast Domain: the set of devices to which a broadcast can be delivered.
  - Routers naturally divide broadcast domains.
- A good definition of a LAN is that it consists of all devices in the same broadcast domain.
  - Switches can divide up broadcast domains by putting hosts into separate VLANs.
  - While configuring a port on a catalyst switch, the command `switchport access vlan 2` will put that interface into VLAN 2.
- Campus LAN: A LAN created to support the devices in a building or in multiple buildings in somewhat close proximity to one another.
  - Access Switches: connect directly to end users, providing user device access to the LAN.
  - Distribution switches: Provide a path through which access switches can forward traffic to each other. Usually each access switch connects to at least one distribution switch.
  - Core Switches: Have one function, connect the distribution switches.
- Four common network topology designs:
  1. Star: One central device connects all other devices.
  2. Full Mesh: All devices are connected directly to each other.
  3. Partial Mesh: Connects a link between some pairs of nodes, but not all.
  4. Hybrid: Combines topology design concepts into a larger design.
- Two-tier design: A network layout that uses a layer of access switches and a connecting layer of distribution switches.
  - Also known as a collapsed core which means it does not have a core tier.
- TIA Cabling Quality: a category assigned to each type of Ethernet cabling that lists the minimum category that the standard supports.

| Ethernet Type | Media | Maximum Segment Length |
| ------------- | ----- | ---------------------- |
| 10Base-T | TIA CAT3 or better, 2 pairs | 100m (328 feet) |
| 100Base-T | TIA CAT5 UTP or better, 2 pairs | 100m (328 feet) |
| 1000Base-T | TIA CAT5e UTP or better, 4 pairs | 100m (328 feet) |
| 10GBASE-T | TIA CAT6a UTP or better, 4 pairs | 100m (328 feet) |
| 10GBASE-T | TIA CAT6 UTP or better, 4 pairs | 38-55m (127-180 feet) |
| 1000BASE-SX | Multimode fiber | 550m (1800 feet) |
| 1000BASE-LX | Multimode fiber | 550m (1800 feet) |
| 1000BASE-LX | 9-micron single-mode fiber | 5 km (3.1 miles) |

- Multimode fiber: has a larger core than single mode cabling, allowing light to enter at multiple angles. Has a lower bandwidth but uses LED rather than laser.
- Single-mode fiber: has a narrow core that allows light to enter at a single angle. Has a higher bandwidth but normally uses a laser.
- Wireless routers perform three device functions:
  1. An ethernet switch for wired connections.
  2. A wireless access point to communicate with the wireless devices and forward the frames to/from the wired network.
  3. A router, to route IP packets to/from the LAN and WAN interfaces.
- Wireless LAN Controller: controls and manages all AP functions such as roaming, defining WLANs, authentication, etc.
- Lightweight AP (LWAP): Forwards data between the wired and wireless LAN, and specifically forwarding data through the WLC using a protocol like Control and Provision of Wireless Access Points (CAPWAP).
***

## Implementing Ethernet Virtual LANs

- Without VLANs, a switch considers all its interfaces to be in the same broadcast domain.
- The most common reasons to create VLANs:
  - To reduce CPU overhead on each device by reducing the number of devices that receive each broadcast frame.
  - To reduce security risks by reducing the number of hosts that receive copies of frames that the switches flood (broadcasts, multicasts, and unknown unicasts).
  - To improve security for hosts that send sensitive data by keeping those hosts on a separate VLAN.
  - To create more flexible designs that group users by department, or by groups that work together, instead of by physical location.
  - To solve problems more quickly, because the failure domain for many problems is the same set of devices as those in the same broadcast domain.
  - To reduce the workload for the Spanning Tree Protocol (STP) by limiting a VLAN to a single access switch.
- VLAN Trunking: causes connected switches to use a process called VLAN tagging, by which the sending switch adds another header to the frame before sending it over the trunk.
  - This extra trunking header includes a VLAN identifier (VLAN ID) field so that the sending switch can associate the frame with a particular VLAN ID, and the receiving switch can then know in what VLAN each frame belongs.
- VLAN trunking creates one link between switches that supports as many VLANs as you need.
- Switches treat a trunk link as if it were a part of all the VLANs.
- Trunks also keep the VLAN traffic separate.
- Cisco Trunking Protocols:
  - 802.1Q: An IEEE Trunking protocol that inserts an extra 4-byte VLAN header into an ethernet frame header after the source address, known as a Tag.
    - The tag is 12 bytes long that supports up to 4094 VLANs.
  - ISL: Inter-Switch Link. A cisco proprietary protocol invented before 802.1Q.
- Normal Range: VLAN ID's ranging from 1 to 1005.
  - All switches can use the normal range.
- Extended Range: VLAN ID's ranging from 1006 to 4094.
  - The rules for which switches can use extended-range VLANs depend on the configuration of the VLAN Trunking Protocol (VTP).
- Native VLAN: Default VLAN 1, defined by 802.1Q where the trunking protocol does not add a VLAN header to frames in this VLAN.
  - Switches receiving frames over a trunk with no VLAN header know that the frame is destined for the native VLAN.
- VLAN Trunks can be created between switches and routers. Sometimes referred to as a router-on-a-stick.
- Multilayer Switches: Switches with Layer 3 capabilities.
  - Typically used to route packets between VLANs to save on overhead.
- Configuring a switch to use VLANs:
  1. From configuration mode, use the `vlan #` command in global configuration mode to create the VLAN and to move the user into VLAN configuration mode.
  2. (Optional): Use the `name <name>` command in VLAN configuration mode to list a name for the VLAN. If not configured, the VLAN name is VLAN####, where #### is the 4 digit decimal VLAN ID.
- For each access interface (each interface that does not trunk, but belongs to a single VLAN):
  1. Use the `interface type number` command in global configuration mode to move into interface configuration mode for a desired interface.
  2. Use the `switchport access vlan #` command in interface configuration mode to specify the VLAN number associated with that interface.
  3. (Optional): Use the `switchport mode access` command in interface configuration mode to make this port always operate in access mode (that is, to not trunk).
- Cisco VLAN Trunking Protocol (VTP): a proprietary tool on Cisco switches that advertises each VLAN configured in one switch (with the vlan number command) so that all the other switches in the campus learn about that VLAN.
- Things `show vtp status` can display:
  - The server switches can configure VLANs in the standard range only (1-1005)
  - The client switches cannot configure VLANs.
  - Both servers and clients may be learning new VLANs from other switches, and seeing their VLANs deleted by other switches, because of VTP.
  - the `show running-config` command does not list any vlan commands.
- Switch Trunk Settings:
  - The type of trunking: IEEE 802.1Q, ISL, or negotiate which one to use.
  - The administrative mode: Whether to always trunk, always not trunk, or negotiate
- Dynamic Trunking Protocol (DTP): Used to negotiate what type of trunking to use.
- `switchport trunk encapsulation {dot1q / isl/ negotiate}`: Set the type of trunking to use on a trunk interface
- Options for administrative settings for the `switchport mode` command:

| Command Option | Description |
| -------------- | ----------- |
| access | Always act as an access (nontrunk) port |
| trunk | Always act as a trunk port |
| dynamic desirable | Initiates negotiation messages and responds to negotiation messages to dynamically choose whether to start using trunking |
| dynamic auto |  Passively waits to receive trunk negotiation messages, at which point the switch will respond and negotiate whether to use trunking |

- The below table lists the administrative modes for interfaces and the expected operational mode:

| Administrative Mode | Access | Dynamic Auto | Trunk | Dynamic Desirable |
| ------------------- | ------ | ------------ | ----- | ----------------- |
| access | Access | Access | Do Not Use | Access |
| dynamic auto | Access | Access | Trunk | Trunk |
| trunk | Do Not Use | Trunk | Trunk | Trunk |
| dynamic desirable | Access | Trunk | Trunk | Trunk |

- DTP negotiations can be disabled altogether using the `switchport nonegotiate` command.
- IP Telephony: where telephones use IP packets to send and receive voice as represented by the bits in the data portion of the IP packet.
- Most IP Telephones include a small 3 port switch used to connect the phone to the switch and a PC to the phone.
- To put phones in one VLAN and PC's in another, phone switch ports act like an access link for the PC's traffic and like a trunk for the phone's traffic.
  - Data VLAN: Defined as a VLAN on a link for forwarding traffic for the device connected to the phone on the same desk.
  - Voice VLAN: Defined on the link for forwarding the phone's traffic, usually tagged with an 802.1Q header.
- Configuring a switch port to use voice VLANs:
  1. Use the `vlan #` command to create two VLANs, one for data and one for voice.
  2. Configure the VLAN like an access VLAN as usual:
    - Use the `interface type number` command to move into the interface connected to an IP phone.
    - Use the `switchport access vlan #` command to define the Data VLAN.
    - Use the `switchport mode access` command to set the port to operate in access mode.
  3. Use the `switchport voice vlan #` command to set the voice VLAN ID.
***

## Troubleshooting Ethernet LANs

- Four key troubleshooting topics:
  1. Analyzing switch interfaces and cabling
  2. Predicting where switches will forward frames
  3. Troubleshooting port security
  4. Analyzing VLANs and VLAN trunks
- Important troubleshooting methodology concepts:
  1. Problem isolation and documentation: the process of narrowing down and isolating the exact problem and documenting any discoveries.
  2. Resolve or escalate: Try to fix the problem and if you can't, move it up the chain to someone who can.
  3. Verify or monitor: Ensure that a fix is working either through some commands or watching to ensure the problem does not return.
- To display the status codes for interfaces, use the `show interfaces` and `show interfaces description` commands.
  - Line status: refers to whether Layer 1 is working or not.
  - Protocol status: refers to whether Layer 2 is working or not.
- A single code interface status corresponds to different combinations of the traditional two-code interface status codes.

| Line Status | Protocol Status | Interface Status | Typical Root Cause |
| ----------- | --------------- | ---------------- | ------------------ |
| administratively down | down | disabled | The shutdown command is configured on the interface. |
| down | down | notconnect | No cable; bad cable; wrong cable pinouts; speed mismatch; neighboring device is (a) powered off, (b) shutdown, or (c) error disabled. |
| up | down | notconnect | Not expected on LAN switch physical interfaces |
| down | down (err-disabled) | err-disabled | Port security has disabled the interface |
| up | up | connected | The interface is working |

- To see the speed and duplex settings of an interface, use the `show interfaces status` command.
  - Autonegotiated speeds are prefixed with an a-
- If one device is using autonegotiation and another isn't, the default decisions are below:
  - If the speed is not known through any means, use 10Mbps and half duplex.
  - If the switch successfully senses the speed without IEEE autonegotiation, by just looking at the signal on the cable:
    - If the speed is 10 or 100 Mbps, default to use half duplex.
    - If the speed is 1,000 Mbps, default to use full duplex.
- Duplex mismatch can be difficult to determine. If the duplex settings do not match on the ends of an Ethernet segment, the switch interface will still be in a connected (up/up) state.
- Cisco switches keep a count of various elements of traffic:
  - Runts: frames that did not meet the minimum frame size requirements (64 bytes, including the 18-byte destination MAC, source MAC, type, and FCS). Can be caused by collisions.
  - Giants: frames that exceed the maximum frame size requirement (1518 bytes, including the 18-byte destination MAC, source MAC, type, and FCS).
  - Input Errors: A total of many counters, including runts, giants, no buffer, CRC, frame, overrun, and ignored counts.
  - CRC (Cyclic Redundancy Check): Received frames that did not pass the FCS math; can be caused by collisions.
  - Frame: Received frames that have an illegal format, for example, ending with a partial byte; can be caused by collisions.
  - Packets Output: Total number of packets (frames) forwarded out the interface.
  - Output Errors: Total number of packets (frames) that the switch port tried to transmit, but for which some problem occurred.
  - Collisions: Counter of all collisions that occur when the interface is transmitting a frame.
  - Late Collisions: The subset of all collisions that happen after the 64th byte of the frame has been transmitted. (In a properly working Ethernet LAN, collisions should occur within the first 64 bytes; late collisions today often point to a duplex mismatch.)
- Switch Forwarding Logic:
  1. Process functions on the incoming interface, if the interface is currently in an up/up (connected) state, as follows:
    - If configured, apply port security logic to filter the frame as appropriate.
    - If the port is an access port, determine the interface's access VLAN.
    - If the port is a trunk, determine the frame's tagged VLAN.
  2. Make a forwarding decision. Look for the frame's destination MAC address in the MAC address table, but only for entries in the VLAN identified in Step 1. If the destination MAC is:
    - Found (unicast): forward the frame out the only interface listed in the matched address table entry.
    - Not found (unicast): flood the frame out all other access ports (except the incoming port) in that same VLAN, plus out trunks that have not restricted the VLAN from that trunk.
    - Broadcast: flood the frame, with the same rules as the previous step.
- Steps for identifying a port security related issue:
  1. Use `show running-config` and `show port-security` to determine which interfaces have port security enabled.
  2. Determine whether a security violation is currently occurring based in part on the violation mode of the interface's port security configuration:
    - shutdown: the interface will be in err-disabled mode, and the port security port status will be secure-down.
    - restrict: the interface will be in a connected state, the port security status will be secure-up, but the `show port-security interface` command will show an incrementing violations counter.
    - protect: the interface will be in a connected state, and the `show port-security interface` command will not show an incrementing violations counter.
  3. In all cases, compare the port security configuration to the diagram and to the Last Source Address field in the output of the `show port-security interface` command.
- To recover from an err-disabled state, use `shut` `no shut`
- Four potential issues that could cause VLAN problems:
  1. Identify all access interfaces and their assigned access VLANs and reassign into the correct VLANs as needed.
  2. Determine whether the VLANs both exist (configured or learned with VTP) and are active on each switch.; IF not, configure and activate the VLANs to resolve problems as needed.
  3. Check the allowed VLAN lists, on the switches on both ends of the trunk, and ensure that the lists of allowed VLANs are the same.
  4. Check for incorrect configuration settings that result in one switch operating as a trunk, with the neighboring switch not operating as a trunk.
- Commands to identify VLAN issues:

| EXEC Command | Description |
| ------------ | ----------- |
| show vlan brief / show vlan | Lists each VLAN and all interfaces assigned to that VLAN (but does not include operational trunks) |
| show vlan id num | Lists both access and trunk ports in the VLAN |
| show interfaces type # switchport | Identifies the interface's access VLAN and voice VLAN, plus the configured and operational mode (access or trunk) |
| show mac address-table | Lists MAC table entries, including the associated VLAN |

- To disable a vlan, use the `shutdown vlan #` command.
- To enable a vlan, use the `no shutdown vlan #` command.
- To disable DTP negotiation, use the `switchport nonegotiate` command.
***

## Subnetting IPv4

- The three phases of IP design and deployment:
  1. Analyze Needs
    - Number of subnets
    - Number of hosts on each subnet
    - 1 size subnet
  2. Design Subnets
    - Choose Network
    - Choose 1 Mask
    - List all subnets
  3. Plan Implementation
    - Subnets --> Locations
    - Static IP
    - DHCP Ranges
- Rules for a subnet:
  - Addresses in the same subnet are not separated by a router.
  - Addresses in different subnets are separated by at least one router.
- One subnet should be used for every:
  - VLAN
  - Point-to-point serial link
  - Ethernet emulation WAN link (EoMPLS)
- The number of hosts in a subnet can be defined with the formula: 2^H - 2 where H is the number of host bits available.
- Network address translation (NAT) translates the IP addresses inside of packets as they go from inside a private network to the internet with an assigned public IP address.
- The private IP ranges:

| Class of Networks | Private IP Networks | Number of Networks |
| ----------------- | ------------------- | ------------------ |
| A | 10.0.0.0 | 1 |
| B | 172.16.0.0. through 172.31.0.0 | 16 |
| C | 192.168.0.0 through 192.168.255.0 | 256 |

- All addresses in a network have network bits, set as 1's, and host bits, set as 0's.
  - Two addresses in the same network have the same value in the network portion.
  - Their host values are different.
- Host bits per network:
  Class A: Network bits: 8, Host bits: 24
  Class B: Network bits: 16, Host bits: 16
  Class C: Network bits: 24, Host bits: 8
- To create subnets, host bits are borrowed and added to the network bits portion.
- The number of bits in the subnet portion identify the possible number of subnets available.
  - For example: 2 host bits used for subnet bits means: 2^2 = 4 subnets.
- Each subnet contains the following:
  - Subnet ID: the number that identifies the subnet. It is the first available host address available.
  - Subnet Broadcast: The number used for broadcast messages in the subnet. It is the last available address in the subnet range.
  - IP addresses: The avaiable host addresses between the ID and broadcast.

| | Class A | Class B | Class C |
| ----- | ----- | ---- | ---- |
| First octet range | 1 - 126 | 128 - 191 | 192 - 223 |
| Valid network numbers | 1.0.0.0 - 126.0.0.0 | 128.0.0.0 - 191.255.0.0 | 192.0.0.0 - 223.255.255.0 |
| Total networks | 2^7 -2 = 126 | 2^14 = 16,384 | 2^21 = 2,097,152 |
| Hosts per network | 2^24 - 2 | 2^16 - 2 | 2^8 - 2 |
| Octets (bits) in network part | 1(8) | 2(16) | 3(24) |
| Octets (bits) in host part | 3(24) | 2(16) | 1(8) |
| Default mask | 255.0.0.0 | 255.255.0.0 | 255.255.255.0 |

- Both 0.0.0.0 and 127.0.0.0 are reserved. (Any IP on a host's interfaces and loopback respectively)
- When thinking of ip addresses as purely classful with no subnetting, remember that each network will have a prefix (network part) and host part.
  - The addresses in the same network have the same values in the network part
  - The addresses in the same network have different values in the host part
- Every classful network has four key numbers that describe the network:
  - Network number
  - First (numerically lowest) usable address
  - Last (numerically highest) usable address
  - Network broadcast address

### Subnet Masks

- Three rules for subnet masks:
  1. The value must not interleave 1s and 0s
  2. If 1s exist, they are on the left
  3. If 0s exist, they are on the right
- Classless Interdomain Routing (CIDR): also known as the prefix mask, it is a forward slash placed after an IP address to represent the number of network bits in the subnet.
  - 11111111.11111111.11000000.00000000 = /18
- Dotted Decimal Number (DDN): contains four decimal numbers, separated by dots to represent 8 bits per decimal number.

| Binary Mask Octet | Decimal Equivalent | Number of Binary 1s |
| ----------------- | ------------------ | ------------------- |
| 00000000 | 0 | 0 |
| 10000000 | 128 | 1 |
| 11000000 | 192 | 2 |
| 11100000 | 224 | 3 |
| 11110000 | 240 | 4 |
| 11111000 | 248 | 5 |
| 11111100 | 252 | 6 |
| 11111110 | 254 | 7 |
| 11111111 | 255 | 8 |

- Classless addressing: the concept that an IPv4 address has two parts - the prefix and host parts - as defined by the mask with no consideration of the class (A, B, or C).
- Classlful addressing: the concept that an IPv4 address has three parts - network, subnet, and host - as defined by the mask and Class A, B, and C rules.
- Subnet Total = Prefix - Class Network Bits
  - /30 Class C: 30 - 24 = 6 subnet bits

### Analyzing Existing Subnets

- An IP subnet is a subset of a classful network.   
  - The subnet has to contain a set of consecutive numbers
  - The subnet holds 2^H numbers, where H is the number of host bits defined by the subnet mask
  - Two special numbers in the range cannot be used as IP addresses:
    - The first (lowest) number acts as the Network ID
    - The last (highest) number acts as the network broadcast address
  - The remaining addresses, whose values sit between the subnet ID and subnet broadcast address, are used as Unicast IP addresses.
- Computers use Boolean math to subnet IP addresses:
  - Perform a Boolean AND of the IP address and mask. This process converts all host bits to binary 0.
  - Invert the mask, and then perform a Boolean OR of the IP address and inverted subnet mask. This process converts all host bits to binary 1.
- Mask: 255.255.128.0 => Pattern: Multiples of 128 => /17
- Mask: 255.255.192.0 => Pattern: Multiples of 64 => /18
- Mask: 255.255.224.0 => Pattern: Multiples of 32 => /19
- Mask: 255.255.240.0 => Pattern: Multiples of 16 => /20
- Mask: 255.255.248.0 => Pattern: Multiples of 8  => /21
- Mask: 255.255.252.0 => Pattern: Multiples of 4  => /22
- Mask: 255.255.254.0 => Pattern: Multiples of 2  => /23
- Multiple number can be calculated as: 256 - mask decimal number
  - 256 - 192 = 64

| | | | | | | | | |
| -- | -- | -- | -- | -- | -- | -- | -- | -- |
| Prefix, interesting octet 2 | /9 | /10 | /11 | /12 | /13 | /14 | /15 | /16 |
| Prefix, interesting octet 3 | /17 | /18 | /19 | /20 | /21 | /22 | /23 | /24 |
| Prefix, interesting octet 4 | /25 | /26 | /27 | /28 | /29 | /30 | /31 | /32 |
| Magic number | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |
| DDN mask in the interesting octet | 128 | 192 | 224 | 240 | 248 | 252 | 254 | 255 |

***

## Implementing IPv4

- Most modern routers include CSU/DSU units built into their serial link hardware.
- Network Interface Modules (NIMS)
- Integrated Services Router (ISR)
- Steps to install a router:
  1. Connect any LAN cables to the LAN ports.
  2. If using an external CSU/DSU, connect the router's serial interface to the CSU/DSU and the CSU/DSU to the line from the telco.
  3. If using an internal CSU/DSU, connect the router's serial interface to the line from the telco.
  4. Connect the router's console port to a PC (using a rollover cable), as needed, to configure the router.
  5. Connect a power cable from a power outlet to the power port on the router.
  6. Power on the router.
- Cable modems and DSL modems convert between CATV Layer 1 and Layer 2 standards to Ethernet and vice versa or DSL signals over a home telephone line and Ethernet.
- A SOHO router does all the following network functions:
  - Router
  - Switch
  - Cable or DSL modem
  - Wireless access point
  - Hardware-enabled encryption
- Connecting to a router uses the same steps as connecting to a switch via the console port.
- Router CLIs have the following differences with Switch CLIs:
  - The configuration of IP addresses differs in some ways, with switches using a VLAN interface and routers using an IP address configured on each working interface.
  - Many Cisco router models have an auxiliary (AUX) port, intended to be connected to an external modem and phone line to allow remote users to dial in to the router, and access the CLI, by making a phone call.
  - Router IOS defaults to disallow both Telnet and SSH into the router because of the default setting of `transport input none` in vty configuration mode.
  - Switches allow the `show mac address-table` command while routers allow the `show ip route` command.
- Each point-to-point serial link can use the High-Level Data Link Control (HDLC, the default), or Point-to-Point Protocol (PPP).
- Examples of refrencing router interfaces on the CLI:
  - `interface ethernet 0`
  - `interface fastEthernet 0/1`
  - `interface gigabitethernet 0/0`
  - `interface serial 1/0/1`
- The most common ways to display interfaces on a router are the `show ip interface brief` and `show interfaces` commands.
- Each interface has two status codes: The first refers to whether Layer 1 is working and the second usually refers to whether the data link layer protocol is working.

| Name | Location | General Meaning |
| ---- | -------- | --------------- |
| Line status | First status code | Refers to the Layer 1 status. (For example, is the cable installed, is it the right/wrong cable, is the device on the other end powered on?) |
| Protocol status | Second status code | Refers generally to the Layer 2 status. It is always down if the line status is down. If the line status is up, a protocol status of down is usually caused by a mismatched data link layer configuration. |

- Interface status code combinations:

| Line Status | Protocol Status | Typical Reasons |
| ----------- | --------------- | --------------- |
| Administratively Down | Down | The interface has a `shutdown` command configured on it. |
| Down | Down | The interface is not `shutdown`, but the physical layer has a problem. For example, no cable has been attached to the interface, or with Ethernet, the switch interface on the other end of the cable is shut down for the switch is powered off. |
| Up | Down | Almost always refers to data link layer problems, most often configuration problems. For example, serial links have this combination when one router was configured to use PPP and the other defaults to use HDLC. |
| Up | Up | Layer 1 and Layer 2 of this interface are functioning. |

- Cisco routers require some configuration, unlike Switches, beyond their default configuration to work.
  - Most Cisco router interfaces default to a disabled (`shutdown`) state and should be enabled with the `no shutdown` interface subcommand.
  - Cisco routers do not route IP packets in or out an interface until an IP address and mask have been configured; by default, no interfaces have an IP address and mask.
  - Cisco routers attempt to route IP packets for any interfaces that are in an up/up state and that have an IP address/mask assigned.
- To configure the address and mask, use the `ip address <address> <mask>` command while in the interface's configuration sub-context.
- The `show protocols` command shows the various states of each interface and global protocls running.
- Routers physical set themselves to the speed dictated by the CSU/DSU through a process called clocking.
- The `clock rate` subcommand used for serial interfaces tells the router which speed to transmit at.
  - The default clock rate is: `clock rate 2000000`
- The `show controllers` command used on serial interfaces shows the current clock rate.
- The `bandwidth` command is used to set the bandwidth on a serial link.
- Aux ports can be configured using the `line aux 0` command.

### Configuring IPv4 Addresses and Static Routes

- Routers use three methods to add IPv4 routes to their IPv4 routing tables.
  - Connected Routes: routes for subnets attached to a router interface.
  - Static Routes: routes created through a configuration command `ip route` that tells the router what route to put in the IPv4 routing table.
  - Routing Protocol: routers tell each other about their known routes so that routers can learn the next hops near them.
- Steps to a router's routing logic:
  1. For each received data-link frame, choose whether or not to process the frame.
    - Check to see if the frame passes FCS math and the Destination MAC is the router.
  2. De-encapsulate the frame to the packet level.
  3. Make a routing decision by comparing the destination IP address to the routing table and find the route that matches the destination.
  4. Encapsulate the packet into a data-link frame appropriate for the outgoing interface.
    - When forwarding out LAN interfaces, use ARP as needed to find the next device's MAC address.
  5. Transmit the frame out the outgoing interface.
- A Cisco router automatically adds a route to its routing table for the subnet connected to each interface, assuming that the following two facts are true:
  1. The interface is in a working state.
  2. The interface has an IP address assigned through the `ip address` command.
- To clear a cisco device of its ARP table entries, use the `clear ip arp` command.
- In order for a router to route a packet out of one VLAN into another it needs to have an IP address in each subnet used by each VLAN and a connected route to each subnet.
- There are three options for connecting a router to each subnet on a VLAN:
  1. Use a router, with one router LAN interface and cable connected to the switch for each and every VLAN.
  2. Use a router, with a VLAN trunk connecting to a LAN switch.
  3. Use a Layer 3 switch.
- Router-on-a-Stick (ROAS): The process where one physical interface is connected to a Switch and configured to be a trunk interface. The router side is configured with virtual subinterfaces to allow for one address in each subnet to travel between multiple VLANs.
  - Uses the same 802.1Q protocl used by switches for trunking.
  - Most routers do not attempt to negotiate trunking, so typically both devices will have their trunk settings configured. (Use the `switchport mode trunk` command on both interfaces.)
  - Configuration checklist:
    1. Use the `interface <type> <number.subint>` command to create a unique subinterface for each VLAN that needs to be routed.
    2. Use the `encapsulation dot1q <vlan_id>` command to enable 802.1Q and associate a specific VLAN with a subninterface.
    3. Use the `ip address <address> <mask>` command to configure IP settings (address and mask).
- To use the native VLAN for each 802.1Q trunk, follow these steps:
  1. Configure the `ip address` command on the physical interface, but without an encapsulation command so that the router considers the physical interface to be using the native VLAN.
  2. Configure the `ip address` command on a subinterface, and use the `encapsulation dot1q <id> native` command.
- To configure a Layer 3 switch to route packets between VLANs, use these 5 steps:
  1. On some older models of switches, enable hardware support for IPv4 routing. For example, on 2960 switches, use the `sdm prefer lanbase-routing` and then the `reload` command.
  2. Use the `ip routing` command to enable routing on the switch.
  3. Use the `interface vlan <vlan_id>` command to create VLAN interfaces for each VLAN that the switch will route packets to.
  4. Use the `ip address <address> <mask>` command to configure an address on the VLAN interface to enable IPv4 on that interface.
  5. Use the `no shutdown` command to enable the VLAN interface.
- To add a static route, use the `ip route <subnetID> <mask> (<outgoing interface> or <next hop address>)` command.
  - `ip route 172.16.2.0 255.255.255.0 172.16.4.2`
  - `ip route 172.16.3.0 255.255.255.0 S0/0/1`
  - `ip route 10.1.1.9 255.255.255.255 10.9.9.9` (Add a single host route)
- To view all static routes, use the `show ip route static` command.
- If a packet matches multiple routes, it uses the route with the longest prefix.
- Before a router adds a static route, it considers the following:
  - For `ip route` commands that list an outgoing interface, that interface must be in an up/up state.
  - For `ip route` commands that list a next-hop IP address, the local router must have a route to reach that next-hop address.
- To have a router ignore its basic checks, use the `permanent` keyword with `ip route` commands.
- If a competing route exists, a router uses administrative distance, or which route is closer to the destination, to send the packet.
  - Static routes have an administrative distance of 1.
- Floating static route: a static route is ignored if a better routing protocol route is known. To change the distance on a static route, append a new administrative distance number to the end of the `ip route` command.
  - `ip route 172.16.2.0 255.255.255.0 172.16.5.3 130`
- To configure a default static route, use special values in the `ip route` command.
  - `ip route 0.0.0.0 0.0.0.0 <interface>`
  - The gateway of last resort is the chosen default route from the candidates of default routes.
- Troubleshooting incorrect static routes that appear in the IP routing table:
  - Check for subnetting math errors, whether or not the next hop is correct, or the outgoing interface is correct.
- The static route does not appear in the ip routing table:
  - The outgoing interface listed in the `ip route` command is not up/up.
  - The next-hop router IP address listed in the `ip route` command is not reachable
  - A better competing route exists with lower administrative distance.
- The correct static route appears but works poorly;
  - Check to see if routes with the permanent key word are still relevant.

### Routing Information Protocol (RIP) version 2

- The core features of all routing protocols:
  1. Learn routing information about IP subnets from other neighboring routers.
  2. Advertise routing information about IP subnets to other neighboring routers.
  3. If a router learns of more than one route to reach one subnet, choose the best route based on that routing protocol's concept of a metric.
  4. React to changes when the network topology changes.
- RIP version 1 was the first popularly used IP routing protocol, with the Cisco proprietary Interior Gateway Routing Protocol (IGRP) being introduced later.
  - These are sometimes known as distance vector protocols.
- Enhanced Interior Gateway Routing Protocol (EIGRP), OSPF version 2, and RIP version 2 represent the next phase of routing protocols for IPv4.
- OSPF version 3, EIGRP version 6, and RIP next generation were designed for IPv6.
- OSPF v3 was later updated to handle both IPv4 and IPv6.
- Interior Gateway Protocols (IGP): routing protocols used inside an organization.
- Exterior Gateway Protocols (EGP): protocols used between organizations and between ISPs.
- Major comparison points for IGPs:
  - The underlying routing protocol algorithm: specifically, whether the routing protocol uses logic referenced as distance vector (DV) or link state (LS).
  - The usefulness of the metric: The routing protocol chooses which route is best based on its metric; so the better the metric, the better choices are made by the protocol.
  - The speed of convergence: How long it takes all routers to learn about a change in the network and update their routing tables.
  - Whether the protocol is a public standard or a proprietary standard: RIP and OSPF are public, but EIGRP is Cisco specific.
- RIP uses a hop count metric and treats each router as a hop. A hop count consists of how many routers or hops it takes to reach a destination.
  - RIP chooses the route with the least hops, although these hops can have the slowest links.
  - More advanced routing protocols may include in part of their metric link bandwidth and are therefore capable of making better routing decisions.
- Distance Vector (DV): What a router knows about each route.
  - Can include: the destination subnet, the distance, and the vector (the link and next-hop router).
  - Calls for each router to send its entire routing table in each update, but only to its neighbors.
- RIP sends out a periodic routing update based on a relatively short timer and repeats the update over and over even if nothing changes.
  - These updates can consist of directly connected subnets to the router or routes it learned from other routers.
- Split horizon: a DV feature that tells a router to omit some routes from an update sent out an interface.
  - Typically includes routes learned on an interface to not go out that interface.
- Route poisoning: a practice of advertising a failed route, but with a special metric value called infinity, which routers will innately consider to be a failed route.
  - For RIP 16 is the infinite metric, therefore, 15 is the largest metric a route can be assigned and still be usable.
- RIP will send its routing update message to the 224.0.0.9 multicast IP address.
- RIPv2 has support for variable length subnet masks (VLSM), which refers to a classful IP address having multiple subnet masks for the various subnets that make it up.
- RIPv2 requires three basic commands and configuration steps:
  1. Use the `router rip` command to move into RIP configuration mode.
  2. Use the `version 2` command in RIP configuration mode to tell the router to use RIP Version 2 exclusively.
  3. Use one or more `network <net-number>` commands in RIP configuration mode to enable RIP on the correct interfaces.
- When using the network command, include some classful IP address as the parameter and IOS will choose the interfaces with an address in the range of the network to enable RIP on.
- Once enabled on an interface, RIP takes three separate actions:
  1. The router sends routing updates out the interface.
  2. The router listens for and processes incoming updates on that same interface.
  3. The router advertises about the subnet connected to the interface.
- Helpful show commands to see if RIPv2 is working:

| Command | Purpose |
| ------- | ------- |
| show ip route [rip] | Routes: This command lists IPv4 routes as learned by RIP. The show ip route command lists all IPv4 routes, and the show ip route rip command lists RIP-learned routes only. |
| show ip protocols | Configuration: This command lists information about the RIP configuration, plus the IP addresses of neighboring RIP routers from which the local router has learned routes. |
| show ip rip database | Best routes: This command lists the prefix/length of all best routes known to RIP on this router, including routes learned from neighbors and connected routes for interfaces on which RIP has been enabled. |

- IOS standard metrics assigned to each routing protocol (also known as administrative distance):

| Route Source | Administrative Distance |
| ------------ | ----------------------- |
| Connected routes | 0 |
| Static routes | 1 |
| EIGRP | 90 |
| OSPF | 110 |
| RIP (v1 and v2) | 120 |
| DHCP default route | 254 |
| Unknown or unbelievable | 255 |

- The `show ip rip database` command is good for seeing routes for subnets learned from other RIP routers and routes for connected subnets for which RIP is enabled on interfaces due to the RIP network commands.
- The RIPv2 `passive-interface` command can be used to stop all RIPv2 updates from being sent out the interface that is matched by a network command.
  - `passive-interface G0/1`
- The `passive-interface default` command sets all interfaces to passive.
- The `no passive-interface` command will set individual interfaces to broadcast RIPv2 messages.
- RIPv2 supports equal-cost routes with maximum paths by spreading the load over routes that have the same metric and go to the same destination. Also known as equal-cost load balancing.
  - RIPv2 controls this behavior with the `maximum-paths <number of paths>` command.
  - Setting maximum paths to 1 disables the feature.
- The `no auto-summary` command disables classful routing.
- A routing protocol that uses autosummarization automatically creates a summary route under certain conditions. That automatic process happens when:
  - That one router connects to subnets of multiple different classful networks.
  - That router uses a routing protocol that uses the autosummary feature.
  - The summary route is essentially a route to the entire classful range of IPs even if it is broken up into subnets.
- Contiguous network: A network topology in which subnets of network X are not separated by subnets of any other classful network.
- Discontiguous network: A network topology in which subnets of network X are separated by subnets of some other classful network.
- The `default-information originate` command tells the router that if the IPv4 routing table has a default route in it, advertise a default route with RIP, with this local router as the eventual destination of those default routes.
  - This will also advertise default routes learned via DHCP.
- The symptoms of a missing `network` command in a RIP configuration can be:
  - The router does not advertise about the subnets on those interfaces.
  - The router does not exchange routing information with other routers on those interfaces.
- The `passive-interface` command should never be used on an interface connected to another RIP enabled router.
- Miss use of the `no auto-summary` command can cause a router to advertise an entire classful network instead of a single subnet with a mask smaller than the default class mask.
- RIP uses UDP port 520 to advertise its updates.

### DHCP and IP Networking on Hosts

- Any host that uses IPv4 needs four settings to work properly:
  1. IP address
  2. Subnet Mask
  3. Default routers
  4. DNS server IP address
- Dynamic Host Configuration Protocol (DHCP): A protocol used to manage the assignment of IP addresses to hosts from a DHCP server.
  - The configuration of host IP settings sits in the DHCP server, which sends DHCP messages to client hosts so that they learn their respective addreses.
  - The DHCP server can permanently assign an address or temporarily lease one.
- The DHCP process of a client finding a DHCP server and obtaining an address takes on four steps:
  1. Discover: Sent by the DHCP client to find a willing DHCP server
  2. Offer: Sent by a DHCP server to offer to lease to that client a specific IP address (and inform the client of its other parameters).
  3. Request: Sent by the DHCP client to ask the server to lease the IPv4 address listed in the Offer message.
  4. Acknowledgment: Sent by the DHCP server to assign the address, and to list the mask, default router, and DNS server IP addresses.
- A DHCP client by default does not yet have an IP address, and instead uses two unique IP addresses:
  - 0.0.0.0: An address reserved for use as a source IPv4 address for hosts that do not yet have an IP address.
  - 255.255.255.255: The local broadcast IP address. Packets sent to this destination address are broadcast on the local data link, but routers do not forward them.
- The host will send a Discover packet using 0.0.0.0 as the source address and 255.255.255.255 as the destination, hoping that there is a DHCP server on the local subnet.
- The server will send its Offer packet to the destination address 255.255.255.255 with its address as the source address.
- The original Discover message lists a client ID number based on the requesting host's MAC address. This allows the host to know that the Offer message sent to the broadcast is meant for it.
- To establish a remote, centralized DHCP server, the routers connected to the remote LAN subnets need an interface subcommand: `ip helper-address <server-ip>`.
- The `ip helper-address <server-ip>` subcommand tells the router to do the following for the messages coming in an interface, from a DHCP client:
  1. Watch for incoming DHCP messages, with the destination IP address 255.255.255.255
  2. Change that packet's source IP address to the router's incoming interface IP address.
  3. Change that packet's destination IP address to the address of the DHCP server (as configured in the `ip helper-address` command).
  4. Route the packet to the DHCP server.
  - This feature is known as DHCP Relay.
  - The router set up to do this is known as a DHCP relay agent.
- When the DHCP server receives the Discover packet, it reverses the source and destination IP addresses in the packet in its Offer Message.
  - The DHCP relay agent then broadcasts the Offer message onto the local subnet from where the initial Discover message came from.
- The `ip helper-address <server-ip>` command should be included on every interface for routers that are expected to have hosts requesting DHCP addresses to a centralized DHCP server.
- All DHCP servers need the following configuration settings:
  - Subnet ID and mask: The DHCP server can use this information to know all addresses in the subnet. (The DHCP server knows to not lease the subnet ID or subnet broadcast address).
  - Reserved (excluded) addresses: The server needs to know which addresses in the subnet to not lease. This list allows the engineer to reserve addresses to be used as static IP addresses. For example, most router and switch IP addresses, server addresses, and addresses of most anything other than user devices use a statically assigned IP address. Most of the time, engineers use the same convention for all subnets, either reserving the lowest IP address in all subnets, or reserving the highest IP addresses in all subnets.
  - Default router(s): This is the IP address of the router on that subnet.
  - DNS IP address(es): This is a list of DNS server IP addresses.
- Some optional settings include address lease time and TFTP server address allocation.
- DHCP uses three allocation modes:
  1. Dynamic allocation: Timed leasing based on requests from clients.
  2. Automatic allocation: sets the DHCP lease time to infinite.
  3. Static allocation: pre-configures the specific IP address for a client based on the client's MAC address.
- Each subnet configured on a DHCP server can have a DHCP pool, which is the pool of addresses that can be assigned to hosts.
- Cisco IOS DHCP server configuration checklist:
  1. Use the `ip dhcp excluded-address <first> <last>` command in global configuration mode to list addresses that should be excluded.
  2. Use the `ip dhcp pool <name>` command in global configuration mode to both create a DHCP pool for a subnet and to navigate into the DHCP pool configuration mode.
    - Use the `network <subnet-id> <mask>` or `network <subnet-id> <prefix-length>` command in DHCP pool configuration mode to define the subnet for this pool.
    - Use the `default-router <address1> <address2>...` command to define default router IP address(es) in that subnet.
    - Use the `dns-server <address1> <address2>...` command in DHCP pool configuration mode to define the list of DNS server IP addresses used by hosts in this subnet.
    - Use the `lease <days> <hours> <minutes>` command to define the length of the lease, in days, hours, and minutes.
    - Use the `domain-name <name>` command in DHCP pool configuration mode to define the DNS domain name.
    - Use the `next-server <ip-address>` command to define the TFTP server IP address used by any hosts that need one.
- IOS DHCP Server show commands:
  - `show ip dhcp binding`: Lists state information about each IP address currently leased to a client.
  - `show ip dhcp pool <pool name>`: Lists the configured range of IP addresses, plus statistics for the number of currently leased addresses and the high-water mark for leases from each pool.
  - `show ip dhcp server statistics`: Lists DHCP server statistics.
  - `show ip dhcp conflict`: List known DHCP conflicts.
- DHCP Troubleshooting Steps:
  1. If using a centralized DHCP server, at least one router on each remote subnet that has DHCP clients must act as DHCP relay agent, and have a correctly configured `ip helper-address <address>` command on the interface connected to that subnet.
  2. If using a centralized IOS DHCP server, make sure the DHCP pools' `network` command match the entire network's list of router interfaces that have an `ip helper-address` command pointing to this DHCP server.
  3. Troubleshoot for any IP connectivity issues between the DHCP relay agent and the DHCP server, using the relay agent interface IP address and the server IP address as the source and destination of the packets.
  4. Troubleshoot for any LAN issues between the DHCP client and the DHCP relay agent.
- DHCP servers detect conflicts, (a host statically assigns an address in the DHCP pool), using pings. If the ping returns a result before an address is assigned, the server offers another address and notes the conflict.
  - DHCP clients use ARP to detect conflicts.
- IP Configuration check commands on host OS's:
  - Windows: `ipconfig /all`
  - MacOS: `ifconfig -a`
  - Linux: `ip a`
- `netstat -rn`: Check the default gateway on a host.
- Types of broadcasts:
  - Local broadcast address: 255.255.255.255, also known as a limited broadcast.
  - Subnet broadcast address: highest numerically available address in a subnet, also known as a directed broadcast.
  - Network broadcast address: highest numerically available address in a classful network, also known as an all-subnets broadcast.
- When using multicast, all hosts still use their individual unicast IP address for their normal traffic, while also using the same multicast (class D) address for the multicast application.
- Multicast addresses may be used as destination IP addresses only.
- To encapsulate a multicast packet over an Ethernet LAN, multicast calculates the destination MAC address by copying the last 23 bits of the IP address behind a reserved 25-bit prefix to form the 48-bit destination MAC address.
  - This multicast MAC address begins with the hex 01005E.
  - A switch receiving one of these frames either floods it out like a broadcast, or use other ethernet multicast features that flood the frame only to those devices that registered to receive a copy.
***

## Subnet Design

- To determine the best subnet mask for a network design, follow these three steps:
  1. Determine the number of network bits (N) based on the class.
    - Class A: 8, Class B: 16, Class C: 24
  2. Determine the smallest value of S, so that 2^S => X, where X represents the required number of subnets.
  3. Determine the smallest value of H, so that 2^H - 2 => Y, where Y represents the required number of hosts/subnet.
- Three possible situations: No Mask works, One mask works, or multiple masks work.
  - In a multiple mask case, the extra, unassigned bits can be given to the subnet or host portions, depending on the needs of the engineer.
- Finding all the masks, Math:
  1. Calculate the shortest prefix mask (/P) based on the minimum value of S, where P = N + S
  2. Calculate the longest prefix mask (/P) based on the minimum value of H, where P = 32 - H
  3. The range of valid masks include all /P values between the two values calculated in the previous steps.
- Choosing the best mask:
  - To maximize the number of host/subnet: use the shortest prefix (the smallest /P value), which will have the largest host part.
  - To maximuze the number of subnets: use the longest prefix.
  - To increase both as much as possible: Choose the prefix in the middle of the range.
- Formal Subnet Design Process:
  1. Find the number of network bits (N) per class rules.
  2. Calculate the minimum number of subnet bits (S) so that 2^S => the number of required subnets.
  3. Calculate the minimum number of host bits (H) so that 2^H -2 => the number of required hosts/subnet.
  4. If N + S + H > 32, no mask meets the need.
  5. If N + S + H = 32, One mask meets the need. Calculate the mask as /P, where P = N + S
  6. If N + S + H < 32, multiple masks meet the need:
    - Calculate mask /P based on the minimum value of S, where P = N + S. This mask maximizes the number of hosts/subnet.
    - Calculate mask /P based on the minimum value of H, where P = 32 - H. This mask maximizes the number of possible subnets.
    - Note that the complete range of masks includes all prefix lengths between the two values calculated in the previous two steps.
- The Zero Subnet: Take the network ID of the classful network and use it as the first subnet ID.
- Related IOS Commands:
  - `ip subnet-zero`: allows the configuration of addresses in the zero subnet.
  - `no ip subnet-zero`: prevents the configuration of addresses in the zero subnet.
- Magic number: 256 minus the mask's decimal value.
  - Identifies the pattern of the subnet IDs.
- Formal process to find all subnet IDs:
  1. Write down the subnet mask, in decimal.
  2. Identify the interesting octet, which is the one with a value other than 255 or 0.
  3. Calculate and write down the magic number by subtracting the subnet mask's interesting octet from 256.
  4. Write down the classful network number, which is the same number as the zero subnet.
  5. To find each successive subnet number:
    - For the three uninteresting octets, copy the previous subnet number's values.
    - For the interesting octet, add the magic number to the previous subnet number's interesting octet.
  6. When the sum calculated in the previous step reaches 256, stop the process.
- Cases with exactly 8 subnet bits:
  - Class A networks with a mask of 255.255.0.0 have the entire second octet containing subnet bits.
  - Class B networks with mask 255.255.255.0 have the entire third octet containing subnet bits.
- Since the magic number is: 256 - 255 = 1, each subnet ID is 1 larger than the previous subnet ID.
  - E.g) 172.16.0.0 255.255.255.0: 172.16.0.0, 172.16.1.0, 172.16.2.0, and so on.
- Finding the subnets with 9 to 16 subnet bits:
  1. Calculate subnet IDs using the 8-subnet-bits-or-less process. However, when the total adds up to 256, move to the next step; consider the subnet IDs listed so far as a subnet block.
  2. Copy the previous subnet block, but add 1 to the just-left octet in all subnet IDs in the new block.
  3. Repeat step 2 until you create the block with a just-left octet of 255, but go no further.
- E.g: 130.4.0.0 255.255.255.192
       130.4.0.0, 130.4.0.64, 130.4.0.128, 130.4.0.192
       130.4.1.0, 130.4.1.64, 130.4.1.128, 130.4.1.192
       ......
       130.4.255.0, 130.4.255.64, 130.4.255.128, 130.4.255.192
- Finding the subnets with 17 or more subnet bits:
  - Requires a class A network, with the subnet portion consisting of the entire second and third octets, plus part of the fourth octet.
  - Same process as before, just with a lot more subnets.

### Variable-Length Subnet Masks (VLSM)

- VLSM occurs when an internetwork uses more than one mask for different subnets of a single Class A, B, or C network.
  - For example, Class A address 10.0.0.0 being divided up with subnets using a /24 prefix, or 10.2.1.0/24, and serial links between routers using a /30, or 10.1.4.0/30.
  - The masks must all be used in the same classful address in order to qualify as VLSM.
- Before VLSM can be deployed, a routing protocol must be used that supports VLSM.
  - Classless routing protocols advertise the mask with each advertised routes, whereas classful ones do not.
  - Routing Protocols that support VLSM: RIPv2, EIGRP, OSPF
- Even with VLSM, subnets used in a network design should not overlap address ranges.
  - An overlap can be easily identified by noticing that two subnets have the same subnet IDs.
- Steps for adding a new subnet to an exisitng VLSM structure:
  1. Pick the subnet mask (prefix length) for the new subnet, based on the design requirements (if not already listed as part of the question).
  2. Calculate all possible subnet numbers of the classful network using the mask from Step 1, along with the subnet broadcast addresses.
  3. Make a list of existing subnet IDs and matching subnet broadcast addresses.
  4. Compare the existing subnets to the candidate new subnets to rule out overlapping new subnets.
  5. Choose the new subnet ID from the remaining subnets identified at step 4, paying attention to whether the question asks for the numerically lowest or numerically highest subnet ID.

### Network Troubleshooting

- If a `ping` command succeeds, as in an echo request was sent and an echo reply was received, the device knows that the packets made it to the destination and back.
  - Best practice is to issue ping commands from the router closest to the device having issues.
  - E.g.) Device X cannot reach device Y, so the engineer pings device X from the nearest router to the device.
- If an initial ping request fails to reach the target but others succeed, it is usually due to a device not having a correct entry in their ARP table.
- Access Control Lists (ACLs) do not filter packets formed on their own device.
- Cisco IOS has an extended ping feature. To use, type `ping` and hit enter. You will then be prompted for the various parameters to use to craft the full command.
  - Enter `y` when asked for extended commands to supply a different source address than the default one used by the router's nearest interface to the ping target.
  - Useful for testing a connection as close to a host with an issue as possible from their default router.
- If all goes well pinging a distant subnet, try pinging devices in the same subnet as the host with issues from the same default router.
- Use extended ping to force a device to use its default router setting by supplying an IP on an interface in a different subnet as the source IP.
- Ping can be used to test that DNS settings are working by pinging a hostname instead of an IP address.
- The `traceroute` command identifies the routers in the path from source host to destination host. Specifically, it lists the next-hop IP address of each router that would be in each of the individual routes.
- The traceroute command gathers information by generating packets that trigger error messages from routers; these messages identify the routers, letting the traceroute command list the routers’ IP addresses in the output of the command. That error message is the ICMP Time-to-Live Exceeded (TTL Exceeded) message, originally meant to notify hosts when a packet had been looping around a network.
  -  The original host that creates the packet sets an initial TTL value. Then, each router that forwards the packet decrements the TTL value by 1. When a router decrements the TTL to 0, the router perceives the packet is looping, and the router discards the packet. The router also notifies the host that sent the discarded packet by sending an ICMP TTL Exceeded message.
- Traceroute also has an extended version on Cisco IOS and can be accessed the same way as extended ping by typing `traceroute` and hitting enter followed by supplying input for each parameter.
  - Windows uses `tracert` and `pathping`, not `traceroute`
- The last router listed in the output of a traceroute command’s output tells us where to look next to isolate the problem, as follows:
  - Connect to the CLI of the last router listed, to look at forward route issues.
  - Connect to the CLI of the next router that should have been listed, to look for reverse route issues.
- If routers are having issues talking to each other, preventing a host machine from telnetting or ssh'ing to them, use `telnet` or `ssh` to connect to the host machine's default router and then the same commands to connect from one Cisco device to another over each connected link to reach the problem machines.
- When checking a host's ip configuration for issues:
  1. Check the host’s list of DNS server addresses against the actual addresses used by those servers.
  2. Check the host’s default router settings against the router’s LAN interface configuration, for the ip address command.
  3. Check the subnet mask used by the router and the host; if they use a different mask, the subnets will not exactly match, which will cause problems for some host addresses.
  4. The host and router should attach to the exact same subnet—same subnet ID and same range of IP addresses. So, use both the router’s and host’s IP address and mask, calculate the subnet ID and range of addresses, and confirm they are in the same subnet as the subnet implied by the address/mask of the router’s ip address command.
- Steps for troubleshooting DHCP problems:
  1. If using a centralized DHCP server, at least one router on each remote subnet that has DHCP clients must act as DHCP relay agent, and have a correctly configured ip helper-address address subcommand on the interface connected to that subnet.
  2. Troubleshoot for any IP connectivity issues between the DHCP relay agent and the DHCP server, using the relay agent interface IP address and the server IP address as the source and destination of the packets.
  3. Whether using a local DHCP server or centralized server, troubleshoot for any LAN issues between the DHCP client and the DHCP relay agent.
  4. Troubleshoot incorrect server configuration.
- If routes overlap, the one with the longest prefix is used. For example, /24 would be used over /16.
- A router can display a route it knows to a specific address using the `show ip route <address>` command.
- IP subnetting rules require that the address ranges in the subnets used in an internetwork should not overlap. IOS sometimes can recognize when a new ip address command creates an overlapping subnet, but sometimes not, as follows:
  - Preventing the overlap on a single router: IOS detects the overlap when the ip address command implies an overlap with another ip address command on the same router.
  - Allowing the overlap on different routers: IOS cannot detect an overlap when an ip address command overlaps with an ip address command on another router.
***

## IPv4 Access Control Lists (ACLs)

- ACLs can be used to perform packet filtering and implementing Quality of Service (QoS) features.
- ACLs can be implemented on a router to apply to packets either entering an interface or leaving an interface.
- Matching Packets: refers to how to configure ACL commands to look at each packet, listing how to identify which packets should be discarded, and which should be allowed through.
- Each IP ACL consists of one or more configuration commands, with each command listing details about values to look for inside a packet's headers.
- General ACL command logic: "Look for these values inside the packet header, and if found, discard the packet or allow the packet".
  - ACLs typically are set up to look at IP addresses and TCP/UDP port numbers.
- The two actions available for an ACL to take exist in the keywords: `deny` and `permit`.
- A standard Cisco filter (ACL) is considered to only match the source IP address of a packet and is configured to identify the ACL using numbers rather than names (numbered).
- A single ACL is both a single entity and at the same time, a list of one or more configuration commands. As a single entity, the configuration enables the entire ACL on an interface, in a specific direction. As a list of commands, each command has different matching logic that the router must apply to each packet when filtering using that ACL.
- When doing ACL processing, the router processes the packet, compared to the ACL as follows:
  - ACLs use first-match logic. Once a packet matches one line in the ACL, the router takes the action listed in the ACL and stops looking further in the list.
- Sequence of processing and ACL as a list: IP, other protocols, standard or extended, named or numbered.
- If a packet does not match any of the items in an ACL, the packet is discarded.
  - **EVERY IP ACL has a deny all statement implied at the end.**
- Standard numbered IP ACL global command format:
  - `access list {1-99 | 1300-1999} {permit | deny} <matching-parameters>`
  - Standard ACLS are in the 1 to 99 and 1300 to 1999 number ranges.
- Each standard ACL has one or more `access-list` commands with the same number from the ranges shown in the command format.
  - Besides the ACL number, each `access-list` command also lists the action (`permit` or `deny`), plus the matching logic.
- Matching an exact source IP address:
  - Type the IP address at the end of the command.
  - Example: `access-list 1 permit 10.1.1.1`
  - Earlier versions of IOS required the `host` keyword before the IP address: `access-list 1 permit host 10.1.1.1`
- Matching a subset of an address with wildcards:
  - Wildcard mask: used to tell IOS to ignore parts of an address when making comparisons.
  - Decimal 0: The router must compare this octet as normal.
  - Decimal 255: The router ignores this octet, considering it to already match.
  - Examples:
    - `access-list 1 permit 10.1.1.1`
    - `access-list 1 deny 10.1.1.0 0.0.0.255` (Check the first three octets and ignore the fourth)
    - `access-list 1 permit 10.0.0.0 0.255.255.255` (Check the first octet but ignore the last three)
  - When using a wildcard mask, any octet intended to be ignore with a 255 in the wildcard mask should be left as 0 in the source address portion.
- Matching a wildcard mask to a specific subnet (With a classless subnet mask):
  - Use the subnet number as the source value in the `access-list` command.
  - Use a wildcard mask found by subtracting the subnet mask from 255.255.255.255.
  - For example, the subnet 172.16.8.0 255.255.252.0 would have the wildcard mask: 0.0.3.255 due to: 255 - 252 = 3.
    - Rule for the actual ACL: `access-list 1 permit 172.16.8.0 0.0.3.255`
- Matching Any/All Addresses:
  - ACLs can match any packet using the `any` keyword.
  - `access-list 1 permit any`
  - Typically this is done at the end of an ACL to override the implicit deny all command at the end of each ACL.
  - You can also explicitly deny all packets with the rule: `access-list 1 deny any`
    - This isn't necessary for logic purposes but it will allow the appropriate `show` commands to display metrics for packets denied at the end of an ACL.
- Implementing standard IP ACLs:
  - Summary of the `access-list` command: `access-list <access-list-number> {deny | permit} <SourceIP> [source-wildcard]`
  1. Plan the location (router and interface) and direction (in or out) on that interface:
    - Standard ACLs should be placed near to the destination of the packets so that they do not unintentionally discard packets that should not be discarded.
    - Because standard ACLs can only match a packet's source IP address, identify the source IP addresses of packets as they go in the direction that the ACL is examining.
  2. Configure one or more `access-list` commands to create the ACL, keeping the following in mind:
    - The list is searched sequentially, using first-match logic.
    - The default action, if a packet does not match any `access-list` commands, is to deny the packet.
  3. Enable the ACL on the chosen router interface, in the correct direction, using the `ip access-group <number> {in | out}` interface subcommand.
- Show commands:
  - `show running-config`: Will display the access list in the order it was entered.
  - `show ip access-lists`: lists details about IPv4 ACLs only.
  - `show access-lists`: lists details about IPv4 ACLs plus any other type of ACLs that are currently configured.
  - `show ip interface <interface#>`: Will display the Inbound and Outgoing access lists.
- Access lists can have a comment using the `remark` keyword:
  - `access-list 1 remark <string>`
- If the `log` keyword is added to the end of an `access-list` command, IOS will issue log messages with occasional statistics about matches of a paticular line in the ACL.
  - Good for troubleshooting.
  - `access-list 2 permit 10.2.2.1 log`
- To reverse an ACL wildcard command to figure out what address range it is looking at, add the wildcard mask to the source IP:
  - `access-list 1 permit 172.16.200.0 0.0.7.255` => 172.16.200.0 + 0.0.7.255 = 172.16.207.255

### Extended IPv4 ACLs

- Extended ACLs are enabled the same way Standard ACLs are and also first-match logic.
- Extended ACLs use the `access-list` command and the syntax is the same up through the `permit` or `deny` keyword.
  - The rest of the command requires three matching parameters: IP protocol type, source IP address, and the destination IP address.
  - For the protocol type, you can use the `ip`, `tcp`, `udp`, or `icmp` keywords. (The `ip` keyword matches all IPv4 packets).
  - Extended ACLs use the numbers 100 - 199 and 2000 - 2699
  - For source and destination IP addresses, Extended ACLs require the `host` keyword before each address, unlike Standard ACLs.
- Examples:
  - `access-list 101 deny tcp any any` (Matches any packet that has a tcp header)
  - `access-list 101 deny ip host 1.1.1.1 host 2.2.2.2` (All IP packets from host 1.1.1.1 going to host 2.2.2.2)
  - `access-list 101 deny udp 1.1.1.0 0.0.0.255 any` (All IP packets that have a UDP header from subnet 1.1.1.0/24 going to any destination)
  - `access-list 101 deny tcp any gt 1023 host 10.1.1.1 eq 23` (TCP packets coming from any IP address with a source port greater than 1023 going to IP host 10.1.1.1 on port 23)
- Extended ACLs have optional parameters for source and destination ports, each placed after their respective IP addresses.
  - Commands that use these paramters MUST have the `tcp` or `udp` keywords to allow port checking.
- General Extended ACL syntax:
  - `access-list {100 - 199 | 2000 - 2699} {permit | deny} {ip | tcp | udp | icmp} <source IP> <source wildcard> {gt | lt | eq} <source port> <destination IP> <destination wildcard>  {gt | lt | eq} <destination port> {log | log-inpug}`

### Naming and Editing ACLs

- Using a named ACL list allows for the use of access-list commands as subcommands under one list name.
- Unlike the global ACL commands, named ACLs use the `ip access-list` command to begin a named ACL, followed by the parts of the `access-list` command after access-list.
  - Example: `ip access-list extended primary` (Creates a named extended ACL named primary)
  - `permit tcp host 10.1.1.2 eq www any`
- Can be assigned to an interface using:
  - `interface serial1`
  - `ip access-group primary out` (Sets the list to apply to Serial1 outgoing)
- General named ACL configuration:
  - `ip access-list {standard | extended} <name>`
- To delete a line from a named ACL, use the following command:
  - For Extended: `no {permit | deny} {ip | tcp | udp | icmp} <source IP> <source wildcard> {eq | gt | lt} <source port> <destination IP> <destination wildcard> {eq | gt | lt} <destination port>`
  - For Standard: `no {permit | deny} <source IP> <source wildcard>`
- Use the `show access-list` command to check your named list for accuracy.
- Numbered ACLs are configured in a similar way to named ACLs, but with a number instead of a name:
  - `ip access-list {standard | extended} <num>`
  - Remember: the number should be in the range for either standard or extended, depending on which kind is chosen.
- To delete a line from a numbered ACL, use the `no <sequence number>` command.
- Normally, non-named ACLs will be sequentially numbered automatically, to specifically number a line, use the following command:
  - `<sequence number> {permit | deny} <source IP> ....`
  - This requires you to be in the configuration context for a specific numbered ACL.
  - Old style: `access-list <number> {permit | deny} <source IP> ....`
- To display a numbered ACL, use the following command:
  - `show ip access-lists <number>`
- ACL Implementation Considerations:
  - Place extended ACLs as close as possible to the source of the packet to save on bandwidth.
  - Place standard ACLs as close as possible to the destination of the packet.
  - Place more specific statements early in the ACL.
  - Disable an ACL from its interface (using the `no ip access-group` command) before making changes to the ACL.
- ACLs can be resequenced by using the `ip access-list resequence <access-list-name> <starting-sequence-number-increment>` command.

### Troubleshooting IPv4 ACLs

- ACL Troubleshooting Steps:
1. Determine on which interfaces ACLs are enabled, and in which direction (`show running-config,`, `show ip interfaces`)
2. Find the configuration of each ACL (`show access-lists`, `show ip access-lists`, `show running-config`)
3. Analyze the ACLs to predict which packets should match the ACL, focusing on the following points:
  - **Misordered ACLs**: Look for misordered ACL statements. IOS uses first-match logic when searching an ACL.
  - **Reversed source/destination addresses**: Analyze the router interface, the direction in which the ACL is enabled, compared to the location of the IP address ranges matched by the ACL statements. Make sure the source IP address field could match packets with that source IP address, rather than the destination, and vice versa for the destination IP address field.
  - **Reversed source/destination ports**: For extended ACLs that reference UDP or TCP port numbers, continue to analyze the location and direction of the ACL versus the hosts, focusing on which host acts as the server using a well-known port. Ensure that the ACL statement matches the correct source or destination port depending on whether the server sent or will receive the packet.
  - **Syntax**: Remember that extended ACL commands must use the `tcp` and `udp` keywords if the command needs to check the port numbers.
  - **Syntax**: Note that ICMP packets do not use UDP or TCP; ICMP is considered to be another protocol matchable with the `icmp` keyword.
  - **Explicit deny any**: Instead of using the implicit `deny any` at the end of each ACL, use an explicit configuration command to deny all traffic at the end of an ACL so that the `show` command counters increment when that action is taken.
  - **Dangerous inbound ACLs**: Watch for inbound ACLs, especially those with deny all logic at the end of the ACL. These ACLs may discard incoming overhead protocols, like routing protocol messages.
  - **Standard ACL location**: Standard ACLs enabled close to the source of matched addresses can discard the packets as intended, but also discard packets that should be allowed through. Always pay close attention to the requirements of the ACL in these cases.
- Routing protocols bypass the logic for outgoing ACLs, but not incoming ACLs.
- Useful `access-list` commands to allow routing protocol traffic:
  - `ip access-list extended RoutingProtocolAllow`
    - `10 permit udp any any eq 520`
    - `20 permit ospf any any`
    - `30 permit eigrp any any`
- A self ping actually sends the echo request to the destination at the end of the link which then sends back an echo request to the router's interface.

***
## Network Address Translation

- Classless Inter-domain Routing (CIDR): a global address assignment convention that defines how the Internet Assigned Numbers Authority (IANA), its member agencies, and ISPs should assign the globally unique IPv4 address space to individual organizations.
  - Allows for the reduction in routes through route summarization. For example, if one organization owned all IPs beginning with 198 in the first octet they could give a route of: 198.0.0.0/8 and then divide up the range as they need to internally.
  - This also allows Regional Internet Registries (RIR) to allocate only the amount of addresses an organization needs rather than an entire classful network range.
- Private Addressing: Allows the use of three unregistered Classful Address Ranges (One class A, B, and C) inside an organization with the intention that they will not actually be used to communicate accross the internet.

| Range of IP Addresses | Network(s) | Class of Networks | Number of Networks |
| --------------------- | ---------- | ----------------- | ------------------ |
| 10.0.0.0 to 10.255.255.255 | 10.0.0.0 | A | 1 |
| 172.16.0.0 to 172.31.255.255 | 172.16.0.0 - 172.31.0.0 | B | 16 |
| 192.168.0.0 to 192.168.255.255 | 192.168.0.0 - 192.168.255.0 | C | 256 |

- Network Address Translation (NAT): changes a private IP address to a valid registered public IP address before the internal traffic leaves the internal LAN for the internet.
- Cisco refers to IP addresses as *inside local* and *inside global* for public IP addresses.
- *Outside global* refers to the one address used by the NAT'd network that doesn't change.
- Dynamic NAT: sets up a pool of possible inside global addresses and defines matching criteria to determine which inside local IP addresses should be translated with NAT.
  - Useful if only a set number of hosts need to be mapped to an equal number of public IP addresses.
- The `clear ip nat translation *` command will clear dynamic entries from the nat table.
- Port Address Translation (PAT): Also known as NAT overload, assigns a unique port number to each host needing NAT to allow for multiple hosts to use one inside global address.
- Static NAT Configuration:
  1. Use the `ip nat inside` command for the interfaces meant to be in the inside part of the NAT design.
  2. Use the `ip nat outside` command for the interfaces meant to be in the outside part of the NAT design.
  3. Use the `ip nat inside source static <inside-local> <inside-global>` in global configuration mode to configure the static mappings.
- Show commands:
  - `show ip nat translations`: list static NAT entries created in the configuration.
  - `show ip nat statistics`: list NAT statistics like current active translations.
- Dynamic NAT Configuration:
  1. Use the `ip nat inside` command for interfaces on the inside of the NAT design.
  2. Use the `ip nat outside` command for interfaces on the outside of the NAT design.
  3. Configure an ACL that matches the packets entering inside interfaces for which NAT should be performed.
    - `access-list 1 permit <IP>`
  4. Use the `ip nat pool <name> <first address> <last address> netmask <subnet mask>` command in global configuration mode to configure the pool of public registered IP addresses.
    - `ip nat pool test 200.1.1.1 200.1.1.2 netmask 255.255.255.252`
  5. Use the `ip nat inside source list <acl-number> pool <pool-name>` command in global configuration mode to enable dynamic NAT. Uses the ACL from step 3 and pool from step 4.
    - `ip nat inside source list 1 pool test`
- Show commands:
  - `show ip nat translations`: display the NAT table entries.
  - `show ip nat statistics`: shows how many times NAT has created a NAT table entry.
- The `debug ip nat` command causes the router to issue a message every time a packet has its address translated for NAT.
- NAT Overload (PAT) Configuration:
  1. Use the `ip nat inside` command for interfaces on the inside of the NAT design.
  2. Use the `ip nat outside` command for interfaces on the outside of the NAT design.
  3. Configure an ACL that matches the packets entering the inside interfaces.
  4. Configure the `ip nat inside source list <acl-number> interface <type/number> overload` configuration command, referring to the ACL created in step 3 and the interface whose IP address will be used for translations.
    - `ip nat inside source list 1 pool test overload`
- Show commands:
  - `show ip nat translations`: show the number of translations and port number for each translation.
- NAT Troubleshooting:
  - **Reversed inside and outside**
  - **Static NAT**: ensure that the `ip nat inside source static` command lists the inside local address first, and the inside global command second.
  - **Dynamic NAT (ACL)**: Ensure that the ACL is configured to match the packets sent by the inside hosts.
  - **Dynamic NAT(pool)**: Ensure that the pool has enough IP addresses.
  - **PAT**: Ensure the overload keyword is used.
  - **ACL**: Check ACLs to make sure they are not filtering traffic not intended to be.
  - **IPv4 Routing**: Check routing troubleshooting steps.
***

## IPv6

- 128-bit addressing scheme.
- Normally written in Hex.
- The core IPv6 protocol defines a packet concept, addresses for those packets, and the role of hosts and routers.
- Protocols that support IPv6:
  - OSPF v3
  - ICMP v6
  - Neighbor Discovery Protocol (NDP): Replaces ARP for IPv6.
  - EIGRP v6
  - RIPng
  - MP BGP-4
- IPv6 Header Format:
  - Version
  - Class
  - Flow Label
  - Payload Length
  - Next Header
  - Hop Limit
  - Source Address (16 bytes)
  - Destination Address (16 bytes)
- Dual Stack: running both IPv4 and IPv6.
- IPv6 format consists of eight sets of four hex digits, each with a colon seperating them.
  - `2340:1111:AAAA:0001:1234:5678:9ABC:1234`
  - These sets are known individually as a quartet, with 8 quartets making up an IPv6 address.
- IPv6 Abbreviating Rules:
  1. Inside each quartet of four hex digits, remove the leading 0's (on the left side) in the three positions on the left. (Four 0s would leave a single 0).
  2. Find any string of two or more consecutive quartets of all hex 0s, and replace that set with a double colon (::). This can only be done once per address.
    - `FE00:0000:0000:0001:0000:0000:0000:0056` can be represented as: `FE00:0:0:1:0:0:0:56`
- IPv6 uses a mask concept called a *prefix length*.
  - Written as a `/` followed by a decimal number.
  - Represents the number of bits of the IPv6 address that define the prefix, which is similar to an IPv4 subnet ID.
  - The legal value of the range for prefix lengths is 0 to 128, inclusive.
- To find the prefix, use these rules:
  1. Copy the first prefix length bits as they are in the IPv6 address.
  2. Change the rest of the bits to 0.
- If the prefix length happens to be a multiple of 4 it means that each hex digit is either copied, or changed to 0. The process is:
  1. Identify the number of hex digits in the prefix by dividing the prefix length (in bits) by 4.
  2. Copy the hex digits determined to be in the prefix per the first step.
  3. Change the rest of the hex digits to 0.

### IPv6 Addressing and Subnetting

- Global unicast: Addresses that work like public IPv4 addresses. The organization that needs IPv6 addresses asks for a registered IPv6 address block, which is assigned as a global routing prefix. After that, only that organization uses the addresses inside that block of addresses; the ones beginning with the assigned prefix.
- Unique local: Works somewhat like private IPv4 addresses, with the possibility that multiple organizations use the exact same addresses, and with no requirement for registering with any numbering authority.
- Global routing prefixes can be thought of as similar to Class A, B, or C IPv4 network numbers from a range of public IPv4 addresses.
  - This is partly done so that internet routers can have one route that refers to all the addresses inside the address block.
- IPv6 address prefixes and their purposes:

| Address Type | First Hex Digits |
| ------------ | ---------------- |
| Global unicast | 2 or 3 (originally); all not otherwise reserved (today) |
| Unique local | FC |
| Multicast | FF |
| Link local | FE80 |

- The /64 prefix is the most commonly used prefix for subnetting a global unicast block of IPv6 addresses.
- The subnetting concepts are similar to IPv4, including deciding where to put each subnet: one for each VLAN and one for each point-to-point WAN connection.
- Subnetting in IPv6:
  - All subnets begin with the global routing prefix (These function similarly to the network portion of an IPv4 address).
  - Subnet bits are taken from the Interface ID (Host bits) like in IPv4.
  - The address ends with the Interface ID which is similar to the IPv4 host bit section.
- IPv6 has no concept of address classes. Therefore there are no preset rules that determine the prefix length of the global routing prefix.
- Example: Prefix 2001:0DB8:1111::/48
  - 64 bit interface ID
  - Subnet field of 16 bits, allowing for 2^16 IPv6 subnets
  - Unicast IP possible: 2001:DB8:1111:1::1/64
  - Subnet ID: 2001:DB8:1111:1::/64 (All Interface ID bits are flipped to 0)
- Subnet ID Rules:
  - All subnet IDs begin with the global routing prefix
  - Use a different value in the subnet field to identify each different subnet
  - All subnet IDs have all 0s in the interface ID.
- Just as with IPv4, the subnet ID is reserved and is not to be assigned to an interface.
- IPv6 addresses assigned to interfaces are done with each unique address followed by the prefix length and also include the default router and DNS server IPv6 addresses.
- IPv6 has a built-in mechanism called Stateless Address Auto-configuration (SLAAC) that assists hosts in learning information similar to DHCP such as the DNS server.
- Rules for creating Unique Local IPv6 addresses:
  - Use FC as the first two hex digits
  - Choose a unique 40-bit global ID
  - Append the global ID to the FC to create a 48-bit prefix, used as the prefix for all of the addresses.
  - Use the next 16 bits as a subnet field
  - The remaining 64 bits are the interface ID
- Subnetting for Unique local addresses is the same as with global unicast addresses.
   - Example: FC + Global ID of 00:0001:0001 = FC00:0001:0001 as the global routing prefix.
   - Subnet ID example: FC00:1:1:1::/64

### Implementing IPv6 Addressing on Routers

- To configure a full 128-bit unicast address, either global unicast or unique local, the router needs an `ipv6 address <address/prefix-length>` interface subcommand on each interface.
  - `ipv6 unicast-routing`
  - `interface Serial0/0/0`
  - `ipv6 address 2001:0db8:1111:0002:0000:0000:0000:0001/64`
- On Cisco routers, the `ipv6 unicast-routing` command enables IPv6 routing so that IPv6 will function.
- Show commands:
  - `show ipv6 interface brief`: display interface IPv6 address information, but not prefix length information.
  - `show ipv6 interface`: display details of IPv6 interface settings.
  - `show ipv6 route connected`: shows the added IPv6 routes for the router's interfaces that are configured with IPv6 unicast addresses.
- EUI-64: (Extended Unique Identifier): Rules a router follows to dynamically create the Interface ID for IPv6 Addresses.
  1. Split the 6-byte (12-hex-digit) MAC address in two halves (6 hex digits each).
  2. Insert FFFE in between the two, making the interface ID now have a total of 16 hex digits (64 bits).
  3. Invert the seventh bit of the interface ID.
- To enable eui-64 on a cisco router, use the `ipv6 address <address/prefix-length eui-64` interface subcommand.
  - This tells the router to find the interface MAC address and do the EUI_64 conversion math to find the interface ID.
  - `ipv6 unicast-routing`
  - `interface Serial0/0/0`
  - `ipv6 address 2001:DB8:1111:2::/64 eui-64`
  - For interfaces that don't have a MAC address, (such as serial interfaces), the router chooses the MAC of the lowest-numbered router interface that does have a MAC.
- Cisco routers support two ways for the router interface to dynamically learn an IPv6 address to use:
  - Stateful DHCP
  - Stateless Address Autoconfiguration (SLAAC)
- Stateful DHCP configuration:
  - `interface FastEthernet0/0`
  - `ipv6 address dhcp`
- SLAAC configuration:
  - `interface FastEthernet0/0`
  - `ipv6 address autoconfig`
- Link-Local Addresses: IPv6 addresses used by some overhead protocols and for routing.
  - Used by each IPv6 host.
  - Packets sent to a link-local address do no leave the IPv6 subnet because routers do not forward packets sent to a link-local address.
  - Neighbor Discover Protocol (NDP) uses link-local addresses for its functionality.
  - Routers use link-local addresses as the next-hop IP addresses in IPv6 routes.
  - Hosts refer to a router's link-local address for their default gateway.
- Rules for hosts calculating their own link-local addresses:
  - All link-local addresses start with the same prefix. The address must begin with FE8, FE9, FEA, or FEB.
    - For example, if FE8 is chosen, the first 10 bits must match prefix FE80::/10,
  - The Interface ID, for Cisco routers, is configured using EUI-64
    - Other OS's randomly generate the interface ID.
  - Ultimately, the link-local address can be simply configured.
    - `ipv6 address <address> link-local`
- IOS creates a link-local address for any interface that has configured at least one other unicast address using the `ipv6 address` command.
- Cisco routers can enbale IPv6 on an interface without using a global unicast address or unique local address at all using the `ipv6 enable` command.
  - This command will at least create a link-local address on the interface.
- IPv6 multicast addresses are those that begin with FF

| Short Name | Multicast Address | Meaning | IPv4 Equivalent |
| ---------- | ----------------- | ------- | --------------- |
| All-nodes | FF02::1 | All-nodes (all interfaces that use IPv6 that are on the link) | A subnet broadcast address |
| All-routers | FF02::2 | All-routers (all IPv6 router interfaces on the link) | None |
| All-OSPF, All-OSPF-DR | FF02::5, FF02::6 | All OSPF routers and all OSPF-designated routers, respectively | 224.0.0.5, 224.0.0.6 |
| RIPng Routers | FF02::9 | All RIPng routers | 224.0.0.9 |
| EIGRPv6 Routers | FF02::A | All routers using EIGRP for IPv6 (EIGRPv6) | 224.0.0.10 |
| DHCP Relay Agent | FF02::1:2 | All routers acting as a DHCPv6 relay agent | None |

- There are two kinds of special IPv6 multicast addresses:
  - Local Scope Multicast Addresses: Multicast addresses that begin with FF02::/16 have a link-local scope, meaning that the routers will not forward these packets outside the local subnet. Addresses that begin with FF08::/16 have an organizational-local scope, meaning that packets sent to these addresses are forwarded throughout the organization but not out into the Internet.
  - Solicited-Node Multicast Addresses: Exists on each interface using IPv6 in addition to the usual unicast address, but has the following aspects:
    - Multicast
    - Link-local (Does not route outside the local subnet)
    - Calculated (generated based on the unicast IPv6 address of the host, specifically the last 6 hex digits)
    - Operation: (Each host interface listens for packets being sent to this address)
    - Overlap (Some hosts might have the same solicited-node multicast address)


- Anycast Addresses: A set address used by a service implemented across several routers that allows a router receiving a packet to the Anycast address to route the packet to the nearest routers that support the address as well. Two steps:
  1. Two routers configure the exact same IPv6 address, designated as an anycast address, to support some service.
  2. In the future, when any router receives a packet for that anycast address, the other routers simply route the packet to the nearest of the routers that support the address.
    - Normally implemented with a /128 prefix.
    - Configuration Example:
      - `interface FastEthernet0/0`
      - `ipv6 address 2001:1:1:1::1/64`
      - `ipv6 address 2001:1:1:1:2::99/128 anycast`
    - The subnet router anycast address is a special anycast address on each subnet, reserved by routers as a way to send a packet to any router on the subnet. The address value is the same number as the subnet ID.
- Miscellaneous IPv6 Addresses:
  - Unknown (unspecified) IPv6 address, ::, or all 0s
  - Loopback IPv6 address, ::1, or 127 binary 0s with a single 1

### Implementing IPv6 Addressing on Hosts

- Four settings required to configure IPv6 on a host:
  1. Interface Unicast IPv6 Address
  2. Associated Prefix Length
  3. Default Router IPv6 Address
  4. DNS Server IPv6 Address(es)
- Network Discovery Protocol (NDP) IPv6 related addressing functions:
  - Stateless Address Auto Configuration (SLAAC): the host uses NDP messages to learn the first part of its address, plus the prefix length
  - Router Discovery: Hosts learn the IPv6 addresses of the available IPv6 routers in the same subnet using NDP messages.
  - Duplicate Address Detection: No matter how a host sets or learns its IPv6 address, the host waits to use the address until the host knows that no other host uses the same address.
  - Neighbor MAC Discovery: After a host has passed the DAD process and uses its IPv6 address, a LAN-based host will need to learn the MAC address of other hosts in the same subnet. NDP replaces ARP, providing messages that act similarly to ARP request and reply messages.
- ICMPv6 Messages for NDP:
  - Router Solicitation (RS): This message is sent to the "all-IPv6-routers" local-scope multicast address of FF02::2 so that the message asks all routers, on the local link only, to identify themselves.
  - Router Advertisement (RA): This message, sent by the router, lists the router's information including the link-local IPv6 address of the router. When unsolicited, it is sent to the all-IPv6-hosts local-scope multicast address of FF02::1. When sent in a response to an RS message, it flows back to either the unicast address of the host that sent the RS or to the all-IPv6-hosts address FF02::1.
- IPv6 does not use broadcasts but instead uses multicasts for various purposes.
- NDP also has another pair of solicitation and advertisement messages:
  - Neighbor Solicitation (NS): This message asks a host with a particular IPv6 address (the target address) to send back an NA with its MAC address listed. The NS message is sent to the solicited-node multicast address associated with the target address, so the message is processed only by hosts whose last six hex digits match the address that is being queried.
  - Neighbor Advertisement (NA): This message lists the sender's address as the target address, along with the matching MAC address. It is sent back to the unicast address of the host that sent the original NS message.
- To view a host's NDP neighbor table, use these commands:
  - Windows: `netsh interface ipv6 show neighbors`
  - Linux: `ip -6 neighbor show`
  - MAC OS: `ndp -an`
- The NDP Duplicate Address Detection process uses NDP NS and NA messages.
  - A host sends an NS message but lists the address it wants to use as the target address.
  - If no host replies, the address is considered to be not in use.
- DHCPv6 gives an IPv6 host a way to learn host IPv6 configuration settings, using the same general concepts as DHCP for IPv4.
- A router acting as a relay must be supplied with a remote DHCP server's IPv6 address as well as its IPv4 address if running a dual stack.
- Stateful DHCPv6 acts much like DHCP for IPv4 where it keeps track of clients who request addresses.
  - Stateful DHCPv6 does not supply the default router to the client. The client instead learns this from RA NDP messages from the router itself.
- DHCPv6 uses the following messages in this order: Solicit, Advertise, Request, Reply. (Instead of the DORA format used by DHCP for IPv4.)
- For a DHCPv6 Solicit message sent by a client, the IPv6 header includes the following information:
  - Source of link-local: The client uses its own link-local address as the source address of the packet.
  - Destination address of "all-DHCP-agents" FF02::1:2: This link-local scope multicast address is used to send packets to two types of devices: DHCP servers and routers acting as DHCP relay agents.
- When using Stateless Address Auto Configuration, a host learns the prefix for its IPv6 address and configures the rest on its own using the following steps:
  1. Learn the IPv6 prefix used on the link, from any router, using NDP RS/RA messages.
  2. Choose its own IPv6 address by making up the interface ID value to follow the just-learned IPv6 prefix.
  3. Before using the address, first use DAD to make sure that no other host is already using the same address.
- When using SLAAC to generate its address, a host learns the prefix, prefix length, and default router from NDP messages and the DNS server's address via DHCPv6 but only asks for the server address, not a full Solicit, Advertise, Request, Reply message chain.
- In MAC OSX and Linux, the `ping6` and `traceroute6` commands can be used for IPv6 addresses in the same way their IPv4 counterparts can be used for IPv4 addresses.
- The `show ipv6 neighbors` command lists a Router's Neighbors learned via NDP.
- The `clear ipv6 neighbor` command clears the NDP table.
- The `show ipv6 routers` command lists any other routers but not the local router learned via RA NDP messages.

### IPv6 Routing

- Cisco Routers add IPv6 routes to their routing tables based on the following circumstances:
  - The configuration of IPv6 addresses on working interfaces. (Connected and local routes.)
  - The direct configuration of a static route
  - The configuration of a routing protocol, like OSPFv3, on routers that share the same data link (dynamic routes).
- Routers do not create IPv6 routes for link-local addresses.
- Connected routes represent the subnet connected to an interface.
- Local routes are host routes for only a specific IPv6 address configured on an interface.
- Steps a router follows to create routes for a configured IPv6 unicast address on an interface:
  1. Routers create IPv6 routes based on each unicast IPv6 address on an interface, as configured with the `ipv6 address` command, as follows:
    - The router creates a route for the subnet (a connected route).
    - The router creates a host route (/128 prefix length) for the router IPv6 address (a local route).
  2. Routers do not create routes based on the link-local addresses associated with the interface.
  3. Routers remove the connected and local routes for an interface if the interface fails, and they re-add these routes when the interface is again in an up/up state.
- The `show ip route` command will display known routes for IPv6 addresses.
- The `show ip route local` command will display known local routes.
- The `ipv6 route <prefix></prefix length> {outgoing interface | next-hop global unicast address}` command acts similar to how the `ip route` command works for IPv4 for adding static routes.
- When adding a static route using a link-local address for the next hop router, the outgoing interface on the device being configured must be included as well.
  - `ipv6 route 2001:db8:1111:2::/64 S0/0/0 FE80::FF:FE00:2`
- The `show ipv6 route static` command lists the known static routes.
- A static default route is configured with the following command: `ipv6 route ::/0 <outgoing interface>`
- A static host route is configured with a /128 prefix length.
  - `ipv6 route 2001:db8:1111:2::22/128 S0/0/0 FE80::FF:FE00:2`
- Just like with IPv4 floating routes, an IPv6 route can have its administrative distance changed to allow routing protocol routes to go first.
  - `ipv6 route 3444:4:4:4::/64 3444:2:2:2::2 130`
- SLAAC can be configured on an interface to allow it to not only configure its own address but also to not add a default route:
  - `ipv6 address autoconfig default`
  - Useful for links between an edge router and an ISP router.
- What happens when a router receives an NDP RA message in regards to routing:
  1. **Interface Address**: Builds its own interface IPv6 address using the SLAAC process, based on the prefix in the RA.
  2. **Local /128 Route**: Adds a local (/128) IPv6 route for the address, as it would for any interface IPv6 address.
  3. **Connected Route for Prefix**: Adds a connected (/64) route for the prefix learned in the NDP RA message.
  4. **Default Route**: R1 adds a default route, to destination ::/0, with the next hop address of the router on the other end of the link as learned from the RA sent by said router.
- IPv6 routing troubleshooting steps:
  1. Prefix/Length: Does the `ipv6 route` command reference the correct subnet ID (prefix) and mask (prefix length)?
  2. If using a next-hop IPv6 address that is a link-local address:
    - Is the link-local address an address on the correct neighboring router?
    - Does the `ipv6 route` command also refer to the correct outgoing interface on the local router?
  3. If using a next-hop IPv6 address that is a global unicast or unique local address, is the address the correct unicast address of the neighboring router?
  4. If referencing an outgoing interface, does the `ipv6 route` command reference the interface on the local router?
***

## Device Management Protocols

### Syslog

- By default, IOS shows log messages to console users for all severity levels of messages. The command to set this mode is: `logging console`.
- For other users, (Telnet and SSH), the `logging monitor` command must be used to allow them to see logging messages as well.
  - For these users, they must also use the `terminal monitor` command for each established terminal session as well.
- By default, IOS discards log messages. To enable storing in RAM, use the `logging buffered` command.
  - To display these stored log messages, use the `show logging` command.
- If a syslog server exists on a network, use the `logging {address|hostname}` command to configure the Cisco device to send syslog messages to the designated server.
- Log message format:
  - **Timestamp** Dec 18 17:10:15.079
  - **The facility on the router that generated the message**: %LINEPROTO
  - **The severity level**: 5
  - **A mnemonic for the message**: UPDOWN
  - **The description of the message**: Line protocol on the Interface FastEthernet0/0, changed state to down.
- To turn off log message timestamps, use the `no service timestamps` command.
- To enable log message sequence numbers, use the `service sequence-numbers` command.
- Log message severity levels go from 1 to 8 with the lower the number the more severe the issue.
  - 0 & 1: Most Severe
  - 2: Critical
  - 3: Error
  - 4: Warning
  - 5 & 6: User notification
  - 7: debug

| Service | To Enable Logging | To Set Message Levels |
| ------- | ----------------- | --------------------- |
| Console | logging console | logging console {level-name / level-number} |
| Monitor | logging monitor | logging monitor {level-name / level-number} |
| Buffered | logging buffered | logging buffered {level-name / level-number} |
| Syslog | logging host {address / hostname} | logging trap {level-name / level-number} |

- To clear old logging messages use the `clear logging` command.
- The `debug` command can be used to ask IOS to monitor for certain events and create log messages accordingly.
  - Example: `debug ip rip`
  - Disable: `no debug ip rip`

### Network Time Protocol (NTP)

- NTP provides protocol messages that devices use to learn the timestamp of other devices. Devices send timestamps to each other with NTP messages, continually exchanging messages, with one device changing its clock to match the other, eventually synchoronizing clocks.
- Best practice is to set the time on the device as close to current time as possible before enabling the NTP client function.
- To set the timezone, use the `clock timezone <zone> +/- <UTC>` with the +-<UTC> portion being how many hours behind UTC the timezone is.
  - `clock timezone EST -5`
- To set daylight savings time, use the `clock summer-time <zone> recurring` command.
  - `clock summer-time EDT recurring`
- To set the actual time on the device, use the `clock set <H:M:S Day Month Year>` command.
  - `clock set 20:52:49 21 October 2015`
- To display the current time settings, use the `show clock` command.
- General rules for NTP on IOS devices:
  - NTP clients adjust their own time based on what they hear from an NTP server.
  - NTP servers supply time information to clients but do not adjust their time as a result.
  - NTP client/servers play both roles. As a client, the devices connects to an NTP server to synchronize its time, and as a server, it supplies time information to other devices.
- To configure a device as an NTP client, use the `ntp server address | hostname` command.
  - This tells the device to act as an NTP client.
  - It also tells the device to act as an NTP server after it has synchronized its time with the server specified in the above command.
- To make an IOS device function as the default clock source, use the `ntp master` command.
  - A stratum level, or the number used to determine a clock source's reliability, can be specified as well: `ntp master #`
- `show ntp associations`: Show peered NTP servers associated with this device.
- `show ntp status`: List NTP details.
- Loopback interfaces can be configured on a router using the `interface loopback #` command.
  - Can be assigned an IP address, be advertised by routing protocols, and be ping-ed/tracerouted.
  - Useful for allowing NTP to continue functioning even if an interface goes down.
  - `interface loopback 0`
  - `ip address 172.16.9.9 255.255.255.0`
  - `ntp master 4`
  - `ntp source loopback 0`

### Analyzing Topology Using CDP and LLDP

- Cisco Discovery Protocol (CDP): discovers basic information about neighboring routers and switches without needing to know the passwords for the neighboring devices.
  - Routers and Switches send CDP messages out each of their interfaces containing device information.
  - Uses Type Length Values (TLVs) to share device information.
  - Runs on the data-link layer only.
- CDP message information:
  - **Device Identifier**: Typically the host name
  - **Address List**: Network and data link addresses
  - **Port Identifier**: The interface on the remote router or switch on the other end of the link that sent the CDP advertisement.
  - **Capabilities List**: Information on what type of device it is (for example, a router or a switch)
  - **Platform**: The model and OS level running on the device.
- Cisco IP Phones use CDP to learn the data and voice VLAN IDs as configured on the access switch.
- `show cdp neighbors <type number>`: Lists one summary line of information about each neighbor or just the neighbor found on a specific interface if an interface was listed.
- `show cdp neighbors detail`: Lists one large set (approximately 15 lines) of information, one set for every neighbor.
- `show cdp entry <name>`: Lists the same information as the `show cdp neighbors detail` command, but only for the named neighbor.
- CDP can be disabled on an interface using the `no cdp enable` command and re-enabled with `cdp enable`.
- To disable and re-enable CDP globally, use the `no cdp run` and `cdp run` global commands, respectfully.
- `show cdp`: states whether CDP is enabled globally, and lists the dfault update and holdtime timers.
- `show cdp interface <type number>`: States whether CDP is enabled on each interface, or a single interface if the interface is listed, and states update and holdtime timers on those interfaces.
- `show cdp traffic`: Lists global statistics for the number of CDP advertisements sent and received.

- Link Layer Discover Protocol (LLDP): Has many of the same features as CDP but was developed to be a standardized protocol that isn't Cisco specific.
  - Runs on the data-link layer.
  - Uses mandatory TLVs to discover neighboring devices.
  - TLVs can share device management addresses and device types.
- `show lldp neighbors`: display neighboring devices learned via LLDP.
- `lldp run`: Enable LLDP globally on a device.
- To enable LLDP on an interface, use the `lldp transmit` and `lldp receive` commands on that interface.
- `show lldp interface`: List the interfaces on which LLDP is enabled.
- The LLDP-MED extension provides support for VoIP.
- LLDP is configured in global configuration mode.
***

## Cisco Device Security Features

- Enable mode password: `enable secret <password>`
- Telnet (Password only):
  - `line vty 0 15`
  - `transport input telnet`
  - `login`
  - `password <password`
- SSH and Telnet:
  - `username <user> secret <password>`
  - `hostname <hostname>`
  - `ip domain-name <domain-name>`
  - `crypto key generate rsa`
  - `line vty 0 15`
  - `transport input all`
  - `login local`
- To encrypt passwords normally stored in clear-text, use the `service password-encryption` command.
- The `no service password-encryption` command keeps encrypted passwords as they are until they are changed.
- The `enable secret` command uses MD5 to encrypt passwords.
- The `no enable secret` command will delete the old password.
- A specific hashing algorithm can be specified for `enable secret`:
  - `enable algorithm-type <type> secret <password>`
  - `enable algorithm-type sha-256 secret password`
- The `banner` command can be used to set a display banner for users to see upon logging in for one of three situations:
  - `banner #`: Sets a "Message of the Day (MOTD)" banner that is ended with the `#` symbol.
  - `banner login #`: Sets a message to display whenever a user logs in for the first time.
  - `banner exec #`: Sets a message to display whenever a user enters enable mode.
- Securing Unused Switch Interfaces:
  - Administratively disable the interface using the `shutdown` interface command.
  - Prevent VLAN trunking by making the port a nontrunking interface using the `switchport mode access` interface command.
  - Assign the port to an unused VLAN using the `switchport access vlan #` command.
  - Set the native VLAN so that it is not VLAN1 but instead is an unused VLAN, using the `switchport trunk native vlan <vlan-id>` command.
- When an external user connects to a router or switch using Telnet or SSH, IOS uses a vty line to represent that user connection.
  - IOS can apply an ACL to those inbound connections by applying an ACL to the vty line to filter the addresses that hosts can use to telnet or SSH into the device.
  - `line vty 0 4`
  - `login`
  - `password cisco`
  - `access-class 3 in`
  - `access-list 3 permit 10.1.1.0 0.0.0.255`
  - If using an outbound ACL to filter moving from one device to another, remember that the outbound rule will look at the destination IP, or where the connection is trying to go.
- Typically, firewalls sit in the forwarding path of all packets and have the following features:
  - Like router IP ACLs, match the source and destination IP addresses.
  - Like router IP ACLs, identify applications by matching their static well-known TCP and UDP ports.
  - Watch application-layer flows to know what additional TCP and UDP ports are used by a particular flow, and filter based on those ports.
  - Match the text in the URI of an HTTP request and match patterns to decide whether to allow or deny the download of a web page.
  - Keep state information by storing information about each packet, and make decisions about filtering future packets based on the historical state information (called stateful inspection, or being a stateful firewall).
- Firewall zones allow a firewall to place multiple interfaces into the same zone and apply the same security rules.
- A Demilitarized Zone (DMZ): Used for devices that need to be publicly accessible to the internet but need some protection as well.
***

## Managing IOS Files

- IOS exists as a single file that is loaded into RAM to use as the OS.
- For each physical memory device in a piece of Cisco equipment, IOS creates a simple IOS file system.
  - To see the file system, use the `show file systems` command.
- Types of IOS file systems:
  - **Opaque**: To represent logical internal file systems for the convenience of internal functions and commands.
  - **Network**: To represent external file systems found on different types of servers for the convenience of reference in different IOS commands.
  - **Disk**: For flash
  - **Usbflash**: For USB flash
  - **NVRAM**: A special type for NVRAM memory, the default location of the startup-config file
- To display a file on disk, use the `more <file sytem:/path/to/file` command.
- The `show flash` command displays the default IFS flash (usually flash0:)
- Steps to upgrade an IOS image into flash memory:
  1. Obtain the IOS image from Cisco.
  2. Place the IOS image someplace that the router can reach. (TFTP server or USB drive).
  3. Issue the `copy` command from the router, copying the file into the flash memory.
  4. Add a `boot system` command to refer to the new file.
  5. Run `copy running-config startup-config`
  6. Reload the router using the `reload` command.
    - `copy tftp flash`
- To display the contents of a directory on device, use the `dir <directory>` command.
  - `dir flash0:`
- To verify that an image is legitimate, copy the md5 hash from cisco's website to the end of the following command:
  - `verify /md5 <IOS_Update_file.bin> <md5 hash>`
- IOS can use TFTP, FTP, and SCP to copy files over the network.
- To configure an ftp username and password in the router, use the following commands:
  - `ip ftp username <username>`
  - `ip ftp password odom`
- IOS can act as an SCP server by using the `ip scp server enable` command.
  - SSH needs to already be supported.
  - Give an SSH user direct access to privileged mode with: `username <user> privilege-level 15 password <password>`
- Cisco devices have a special-purpose OS called ROMMON that is used for things like password recovery.
- The IOS boot process:
  1. The router performs a POST (Power-on self-test) process to discover the hardware components and verify that everything is working properly.
  2. The router copies a bootstrap program from ROM into RAM and runs the bootstrap program.
  3. The bootstrap program decides which IOS image (or the ROMMON OS) to load into RAM, and then the bootstrap program loads the OS. After loading, the bootstrap program hands over control of the router hardware to the newly loaded OS.
  4. If the bootstrap program happened to load IOS, once IOS is running, it finds the startup-config file and loads it into RAM as the running-config.
- Routers use a *configuration register* to find some configurations settings at boot time.
  - Consists of 4 hex digits (16 bits).
  - Can be changes with the `config-register` command.
  - Setting it to: `config-register 0x2100` tells the router to load ROMMON.
  - Default setting is 0x2102 which sets the console speed at 9600 bps and loads an IOS image.
- The device chooses the OS to load based on two factors:
  - The last hex digit in the configuration register (called the boot field).
  - Any `boot system` global command in the startup-config file.
- Boot field values:
  - 0: use ROMMON
  - 1: load the first IOS file found in flash memory
  - 2 - F: Try each `boot system` command in the startup-config, and if none works, load the first IOS file found in flash memory.
  - If all other attempts fail, load ROMMON.
- Each new `boot system` command is appended to the end of a list of `boot system` commands.

| Boot System Command | Result |
| ------------------- | ------ |
| boot system flash | The first file from system flash memory is loaded |
| boot system flash filename | IOS with the name <filename> is loaded from system flash memory |
| boot system tftp filename 10.1.1.1 | IOS with the name <filename> is loaded from the TFTP server at address 10.1.1.1 |

- Use the `show version` command to verify that the correct IOS file was loaded.

### Password Recovery

- If the ignore configuration bit is set (the 9th bit) is set to 1, the router will ignore the startup-config the next time the router is loaded.
  - `config-register 0x2142`
- Steps to recover passwords:
  1. Boot ROMMON, either by breaking into the boot process from the console or removing all the flash memory.
  2. Set the configuration register to ignore the startup-config (for example, `confreg 0x2142`)
  3. Boot the router with an IOS. The router boots with no configuration.
- Following this, select no when asked to do initial configuration, enter enable mode, copy the startup config to the running config, enter global configuration mode, set a new enable password, reconfigure the configuration register (0x2102), then copy the running config to the startup config.

### Managing Config Files

- It is considered a good practice to make copies of a router or switch config and save them elsewhere.
  - `copy running-config usbflash1:temp-copy-of-config`
- Using the copy command to backup and restore config files:
  1. To back up: Copy the running-config to an external server; for instance: `copy running-config tftp`
  2. To restore:
    - Copy the backup configuration into the startup-configuration file using the `copy` command, which replaces the startup-config file; for instance, `copy tftp startup-config`.
    - Issue the `reload` command, which reloads, or reboots, the router. That process erases all running config in RAM and then copies the startup-config into RAM as part of the reload process.
- IOS can be configured to archive config files to a pre-determined location periodically:
  - `archive`
  - `path ftp://username:password@192.168.1.170/`
  - `time-period 1440`
  - `write-memory`
- The `archive config` command can be used to manually archive the config files.
- The `show archive` command displays the currently saved archives.
- The `configure replace <source>` command will replace the current running-config with an archive without having to reload the device.
  - `configure replace ftp://wendell:odom@192.168.1.170/ -Oct-24-09-46-43.165-2`
- Setup mode, started with the `setup` command, will walk someone through configuring a device.
***

## IOS License Management

- Product Authorization Key (PAK): used for Cisco license management methods.
- From the beginning of Cisco until about 2008, they created each IOS image for a particular router model, version and release, and feature set.
- Major revisions to IOS software are called versions and smaller changes are called releases.
  - Cisco also created one image for each combination of IOS feature sets that were allowed on each router.
- After 2008, Cisco moved to a universal image which includes all feature sets.
- Today, in order to download new Cisco updates, a user must have an account with the right privileges.
- New Versions of IOS must be activiated now to unlock feature sets, enable the IP Base, and verify legal rights.

| Technology Package License | Features |
| -------------------------- | -------- |
| IP Base | Entry-level IOS functionality |
| Application Experience | Performance routing (PfR), WAAS, NBAR2 |
| Unified Communications | VoIP, IP Telephony |
| Security | IOS firewall, IPsec, DMVPN, VPN |

- Cisco License Manager (CLM): A free software package that performs the following functions:
  - Communicates with Cisco's Product License Registration Portal over the Internet
  - Takes as input information about feature licenses purchased from any Cisco reseller
  - Communicates with the company's routers and switches to install license keys, enabling feaures on the correct devices.
- Each Cisco device that supports software licensing ha9+
s a unique identifying number called the Unique Device Identifier (UDI).
  - Two main components: the product ID (PID) and the serial number (SN).
- The `show license udi` command displays the device UDI.
- Steps for obtaining a new license key and updating a device.
  1. At the Cisco Product License Registration Portal, input the UDI of the router.
  2. At that same portal, type in the PAK for the license you purchased, as learned from your reseller or directly from Cisco.
  3. Copy the license key file (download or email) when prompted at Cisco's Product License Registration Portal website.
  4. Make the file available to the router via USB or some network server.
  5. From the router CLI, issue the `license install <url>` command to install the license key file into the router. (The URL points to the file).
  6. Reload the router to pick up the changes.
- The `show license` command shows the licene's features.
- The `show license feature` command lists one line of output per feature.
- Installing a license key that's on a usb stick:
  - `show version` (To find the usb drive name)
  - `dir usbflash1:` (Display the files in the usb drive to get the license key name)
  - `license install usbflash1:<filename>.lic`
  - `reload`
- A `show license` command on reboot should display the new license has been loaded.
- To try the features of a license without buying a PAK, use the `license boot module` command.
  - `license boot module c2900 technology-package securityk9`
  - `reload`
  - The feature will have a 60 day trial limit.
