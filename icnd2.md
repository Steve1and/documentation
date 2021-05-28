# Notes for the CCNA 200-125 Routing and Switching Test, book ICND2.
***

## Ethernet Virtual LANs

- A LAN includes all devices in the same broadcast domain.
- A Virtual LAN (VLAN) breaks up broadcast domains across a single switch and can be shared across multiple switches.
- Common reasons for creating VLANs:
  - To reduce CPU overhead on each device by reducing the number of devices that receive each broadcast frame.
  - To reduce security risks by reducing the number of hosts that receive copies of frames that the switches flood (broadcasts, multicasts, and unknown unicasts).
  - To improve security for hosts that send sensitive data by keeping those hosts on a separate VLAN.
  - To create more flexible designs that group users by department, or by groups that work together, instead of by physical location.
  - To solve problems more quickly, because the failure domain for many problems is the same set of devices as those in the same broadcast domain.
  - To reduce the workload for the Spanning Tree Protocol (STP) by limiting a VLAN to a single access switch.
- VLAN Trunking: causes switches to use a process called VLAN tagging, by which the sending switch adds another header to the frame before sending it over a trunk.
  - This extra trunking header includes a *VLAN identifier (VLAN ID)* field so that the sending switch can associate the frame with a particular VLAN ID, and the receiving switch can then know in what VLAN each frame belongs.
- VLAN Trunking Protocols:
  - ISL (Inter-Switch Link): Cisco proprietary
  - 802.1Q: Industry Standard (IEEE): inserts an extra 4-byte header into the frame header that includes a 12-bit VLAN ID field.
  - Supports up to 4094 VLANs.
- Normal-range VLANs: 1 - 1005
- Extended-range VLANs: 1006 - 4094
- Native VLAN: default VLAN 1, a trunking header is not added to the frames sent to this VLAN.
- Configuring a new VLAN on a Cisco Switch:
  1. Configuring the actual VLAN
    - From configuration mode, use the `vlan <vlan-id>` command to create the VLAN and to move the user into VLAN configuration mode.
    - (Optional) Use the `name <name>` command to list a name for the VLAN. If not configured, the VLAN name is VLANZZZZ where ZZZZ is the four-digit decimal VLAN-ID.
  2. For each access interface (each interface that does not trunk but instead belongs to a single VLAN), follow these steps:
    - Use the `interface <type> <number>` command to move into the configuration mode for the chosen interface.
    - Use the `switchport access vlan <id-number>` command to specify the VLAN number associated with that interface.
    - (Optional) Use the `switchport mode access` command in interface configuration mode to make this port always operate in access mode.
- The `show vlan brief` command displays VLAN information.
- VLAN Trunking Protocol (VTP): a Cisco proprietary tool on Cisco switches that advertises each VLAN configured in one switch (with the `vlan <number>` command) so that all the other switches in the network learn about that VLAN.
- Three VTP modes exist: server, client, or transparent.
  - Transparent disables VTP.
- The `show vtp status` command to see the current VTP settings.
- The `switchport mode trunk` command sets an interface to be a trunk port always.
- To set the trunk encapsulation scheme, use the `switchport trunk encapsulation {dot1q | isl | negotiate}` command.
  - Used to configure the Dynamic Trunking Protocol on a trunk interface.
- Trunking administrative mode options with the `switchport` mode command:

| Command Option | Description |
| -------------- | ----------- |
| access | Always act as an access port |
| trunk | Always act as a trunk port |
| dynamic desirable | Initiates negotiation messages and responds to negotiation messages to dynamically choose whether to start using trunking |
| dynamic auto | Passively waits to receive trunk negotiation messages, at which point the switch will respond and negotiate whether to use trunking |

- The `show interfaces swtichport` command lists the default administrative mode of switch interfaces.
- The `show interfaces trunk` command lists information about all interfaces that currently operationally trunk.
- To disable DTP negotiations, use the `switchport nonegotiate` command.
- Configuring Data and Voice VLANs for IP telephones.
  1. Create two VLANs (`vlan 10`, `vlan 11`)
  2. Choose the switch interface connected to the IP telephone (`interface FastEthernet0/1`)
  3. Set the port to be access only (`switchport mode access`)
  4. Set the data VLAN (For the workstation connected to the phone) (`switchport access vlan 10`)
  5. Set the voice VLAN (`switchport voice vlan 11`)
- To verify, use the `show interfaces FastEthernet0/1 switchport` command.
***

## Spanning Tree Protocol (STP) (802.1D)

- STP allows Ethernet LANs to have the added benefits of installing redundant links in a LAN, which allow a LAN to keep working even if some links fail.
- STP blocks certain ports on redundant links to prevent routing loops with two goals in mind:
  1. All devices in a VLAN can send frames to all other devices. In other words, STP does not block too many ports, cutting off some parts of the LAN from other parts.
  2. Frames have a short life and do not loop around the network indefinitely.
- STP adds an additional check on each interface before a switch sends or receives traffic:
  - If the port is in STP forwarding state in a specified VLAN, use it as normal; if it is in STP blocking state, however, block all user traffic and do not send or receive user traffic on that interface in the specified VLAN.
- Broadcast Storms: The forwarding of a frame repeatedly on the same links, consuming significant parts of the links' capacities.
- MAC table instability: a switches' MAC address tables keep changing due to frames with the same source MAC arriving on different ports.
- Multiple Frame Transmission: A side effect of looping frames in which multiple copies of one frame are delivered to the intended host, confusing the host.
- STP convergence: the process by which switches collectively realize that something has changed in the LAN topology and determine whether they need to change which ports block and which ports forward.
- The STP algorithm creates a spanning tree of interfaces that forward frames; resulting in a single path to and from each Ethernet link.
- STP uses three criteria to choose whether to put an interface in forwarding state:
  1. STP elects a root switch. STP puts all working interfaces on the root switch in a forwarding state.
  2. Each nonroot switch considers one of its ports to have the least administrative cost between itself and the root switch. The cost is called that switch's root cost. STP places its port that is part of the least root cost path, called that switch's root port (RP), in forwarding state.
  3. Many switches can attach to the same Ethernet segment, but in modern networks, normally two switches connect to each link. The switch with the lowest root cost, as compared with the other switches attached to the same link, is placed in forwarding state. That switch is the designated switch, and that switch's interface, attached to that segment, is called the designated port (DP).
- Summary of reasons STP puts a port in forwarding or blocking state:

| Characterization of the Port | STP State | Description |
| ---------------------------- | --------- | ----------- |
| All the root switch's ports | Forwarding | The root switch is always the designated switch on all connected segments. |
| Each nonroot switch's root port | Forwarding | The port through which the switch has the least cost to reach the root switch (lowest root cost). |
| Each LAN's designated port | Forwarding | The switch forwarding the Hello on to the segment, with the lowest root cost, is the designated switch for that segment. |
| All other working ports. | Blocking | The port is not used for forwarding user frames, nor are any frames received on these interfaces considered for forwarding. |

- STP bridge ID (BID): an 8-byte value unique to each switch. Consists of a 2-byte priority field and a 6-byte system ID, with the system ID being based on the switch's MAC.
- Bridge Protocol Data Units (BPDU): STP messages used to allow switches to exchange information with each other.
   - Hello BPDU information:

| Field | Description |
| ----- | ----------- |
| Root bridge ID | The bridge ID of the switch the sender of this Hello currently believes to be the root switch. |
| Sender's bridge ID | The bridge ID of the switch sending this Hello BPDU |
| Sender's root cost | The STP cost between this switch and the current root |
| Timer values on the root switch | Includes the Hello timer, MaxAge timer, and forward delay timer |

- Switches elect a root switch based on the BIDs in the BPDUs. The root switch is the one with the lowest numeric value for the BID.
  - If a tie occurs in the priority portion of the BPDU, the switch with the lowest MAC address portion is root.
- All switches start assuming they are the root switch until they hear a Hello BPDU with a lower BID, in which case they will forward this BID as the root switch.
- Switches add their local interface STP cost to the root cost listed in each received Hello BPDU and determine the root port based on the lowest number.
- The switch with the lower cost to reach the root, among all switches connected to a segment, becomes the designated port on that segment.
  - If advertised costs tie, the switch with the lowest BID becomes the designated port.
- Default Spanning Tree Protocol Port Cost:

| Ethernet Speed | IEEE Cost: 1998 (and Before) | IEEE Cost: 2004 (and After) |
| -------------- | ---------------------------- | --------------------------- |
| 10 Mbps | 100 | 2,000,000 |
| 100 Mbps | 19 | 200,000 |
| 1 Gbps | 4 | 20,000 |
| 10 Gbps | 2 | 2000 |
| 100 Gbps | N/A | 200 |
| 1 Tbps | N/A | 20 |

- The newer (2004 and later) standard can be enabled with the `spanning-tree pathcost method long` command.
- For switch interfaces connected to hosts or routers, which do not use STP, the switch still forwards Hellos on to those interfaces. This effectively makes that interface the designated port of that link.
- By default, the root switch sends a Hello BPDU every 2 seconds. Each nonroot switch forwards the Hello on all DPs after making their own changes.
- Steady-state operation when nothing is currently changing in the STP topology:
  1. The root creates and sends a Hello BPDU, with a root cost of 0, out all its working interfaces (those in a forwarding state).
  2. The nonroot switches receive the Hello on their root ports. After changing the Hello to list their own BID as the sender's BID, and listing that switch's root cost, the switch forwards the Hello out all designated ports.
  3. Steps 1 and 2 repeat until something changes.
- Convergence Process Timers:

| Timer | Default Value | Description |
| ----- | ------------- | ----------- |
| Hello | 2 seconds | The time period between Hellos created by the root. |
| MaxAge | 10 time Hello | How long any switch should wait, after ceasing to hear Hellos, before trying to change the STP topology. |
| Forward delay | 15 seconds | Delay that affects the process that occurs when an interface changes from blocking state to forwarding state. A port stays in an interim listening state, and then an interim learning state, for the number of seconds defined by the forward delay timer. |

- If a switch reaches its MaxAge timer, it begins to make all its STP choices again, thinking that something has gone wrong in the topology.
- When a port that was blocking needs to move to forwarding, the switch first puts the port through two intermediate interface states. These temporary states help prevent temporary loops:
  - Listening: Like the blocking state, the interface does not forward frames. The switch removes old MAC table entries.
  - Learning: Interfaces in this state still do not forward frames, but instead begin to populate their MAC tables.

### Rapid STP (802.1W)

- Similarities between RSTP and STP:
  - It elects the root switch using the same parameters and tiebreakers
  - It elects the root port on nonroot switches with the same rules
  - It elects the designated ports on each LAN segment with the same rules.
  - It places each port in either forwarding or blocking state, although RSTP calls the blocking state the discarding state.
- The primary reason for RSTP is to improve on convergence time. It takes a few to 10 seconds rather than the old 50 or more.
- RSTP is compatible with the original IEEE 802.1D STP.
- When RSTP has converged there are only 2 port states: discarding and forwarding.
- New RSTP features:
  - Adds a new mechanism to replace the root port, without any waiting to reach a forwarding state (in some conditions)
  - Adds a new mechanism to replace a designated port, without any waiting to reach a forwarding state (in some conditions)
  - Lowers waiting times for cases in which RSTP must wait.
  - Combines the listening and blocking states into just blocking to allow faster convergence time.
  - MaxAge is only three times the Hello timer
- RSTP port roles and functions:

| Function | Port Role |
| -------- | --------- |
| Nonroot switch's best path to the root | Root port |
| Replaces the root port when the root port fails | Alternate port |
| Switch port designated to forward onto a collision domain | Designated port |
| Replaces a designated port when a designated port fails | Backup port |
| Port that is administratively disabled | Disabled port |

- For a port to be an alternate port, both it and the root port must receive STP Hellos.
- RSTP switches tell each other (using messages) that the topology has changed, which direct neighboring switches to flush their MAC tables to remove all potentially loop-causing entries.
- Point-to-point ports: Ports that connect one switch to another.
- Edge ports: Ports that connect a switch to an end user device or server.
- Shared ports: Ports that connect to a hub.
- **Multiple Spanning Tree (MST)**: rides on top of RSTP so it converges very fast. The idea behind MST is that some VLANs can be mapped to a single spanning tree instance because most networks do not need more than a few logical topologies.

### Optional STP Features

- EtherChannel: combines multiple parallel segments of equal speed (up to eight) between the same pair of switches.
  - Treated as a single interface in regards to STP.
  - Switches use load balancing logic to send frames over an EtherChannel.
- PortFast: Allows a switch to immediately transition from blocking to forwarding, bypassing listening and learning states.
  - Can only be used on ports that no bridges, switches, or other STP-speaking devices are connected to.
- BPDU Guard: disables a port if any BPDUs are received on that port. (Good when combined with PortFast).
***

## Spanning Tree Protocol Implementation

- Best practice suggests that distribution layer and core layer switches have their Priority field set low enough to guarantee they are the root switches in an STP topology.
- Per-VLAN Spanning Tree Plus (PVST+): A cisco-proprietary improvement of the 802.1D standard that creates a different STP topology per VLAN.
  - Usually the default STP mode for Cisco switches.
  - Set with the command: `spanning-tree mode pvst`
- Rapid PVST+: PVST+ with RSTP support.
  - Set with the command: `spanning-tree mode rapid-pvst`
- Multiple Spanning Tree (MST): Third STP option supported by cisco switches.
  - Set with the command: `spanning-tree mode mst`
- STP System ID Extension: Priority field (4 bits) combined with a 12 bit VLAN ID.
- Setting a switch's priority field:
  - `spanning-tree vlan <vlan ID> priority #`
  - The number must be a multiples of 4096.
- For interfaces with mutiple speeds, Cisco switches base the port cost on the current speed.
- Can be manually configured with: `spanning-tree [vlan <vlan ID>] cost #`
- STP Configuration Option Summary

| Setting | Default | Command(s) to Change Default |
| ------- | ------- | ---------------------------- |
| BID Priority | Base: 32,768 | `spanning-tree vlan <vlan-ID> root {primary / secondary }` or `spanning-tree vlan <vlan ID> priority #` |
| Interface Cost | 100 for 10 Mbps / 19 for 100 Mbps / 4 for 1 Gbps / 2 for 10 Gbps | `spanning-tree vlan <vlan ID> cost #` |
| PortFast | Not enabled | `spanning-tree portfast` |
| BPDU Guard | Not enabled | `spanning-tree bpduguard enable` |

- `show spanning-tree vlan #`: Display the STP settings for a particular VLAN.
- `show spanning tree root`: Lists the root's BID for each VLAN.
- `show spanning-tree vlan # bridge`: Breaks out the BID into its component parts.
- `debug spanning-tree events`: Will monitor changes to STP.
- Instead of using numbers for STP root priority, the keywords `primary` and `secondary` can be used.
  - `spanning-tree vlan <vlan ID> root primary`: Tells a switch that its priority is low enough to become root.
  - `spanning-tree vlan <vlan ID> root secondary`: Tells a switch that its priority is worse than the root switch but better than other switches. (Sets the priority to 28,672)
- `show spanning-tree interface detail`: Lists a detailed account of the STP settings for a given interface.
- PortFast and BPDU Guard command summary:

| Action | Globally | One Interface |
| ------ | -------- | ------------- |
| Disable PortFast | `no spanning-tree portfast default` | `spanning-tree portfast disable` |
| Enable PortFast | `spanning-tree portfast default` | `spanning-tree portfast` |
| Disable BPDU Guard | `no spanning-tree portfast bpduguard default` | `spanning-tree bpduguard disable` |
| Enable BPDU Guard | `spanning-tree portfast bpduguard default` | `spanning-tree bpduguard enable ` |

- `show spanning-tree summary`: Shows the current global settings for STP.
- Manual EtherChannel Configuration Steps:
    1. Add the `channel-group # mode on` command to each interface designated to be in a channel group.
    2. Use the same number for all commands on the same switch, but the channel-group number on the neighboring switch can differ.
- `show etherchannel # summary`: Show the summary information for a given etherchannel.
- Cisco switches support the Port Aggregation Protocol (PAgP) and the Link Aggregation Control Protocol (LACP) to negotiate whether or not an interface becomes part of an etherchannel.
- To use PAgP: `channel-group 1 mode {desirable | auto}`
- To use LACP: `channel-group 1 mode {active | passive}`
- `show etherchannel # port-channel`: Display which negotiation protocol is in use.
- The Load value in a `show interface port-channel # etherchannel` command output lists the number of source-destination pairs on the link for an interface.
***

## LAN Troubleshooting

- Determining the root switch:
  1. Begin with a list or diagram of switches, and consider all as possible root switches.
  2. Rule out any switches that have an RP (`show spanning-tree`, `show spanning-tree root`), because root switches do not have an RP.
  3. Always try `show spanning-tree`, because it identifies the local switch as root directly: "This switch is the root" on the fifth line of output.
  4. Always try `show spanning-tree root`, because it identifies the local switch as root indirectly: The RP column is empty if the local switch is the root.
  5. Rather than try switches randomly, chase the RPs. For example, if starting with SW1, and Sw1's G0/1 is a RP, next try the switch on the other end of that port.
  6. Use the `show spanning-tree vlan #` command on a few switches and record the root switch, RP, and designated port (DP).
- When choosing a root port, a nonroot switch will try to find the lowest root cost per link, and if there is tie, it goes with the lowest BID of the neighbor at the end of each link.
- Three total tiebreakers:
  1. Choose based on the lowest neighbor bridge ID.
  2. Choose based on the lowest neighbor port priority.
  3. Choose based on the lowest neighbor internal port number.
- Default port priority is 128. This can be configured between 0 and 255 using the following command:
  - `spanning-tree vlan # port-priority #`
- Steps to figure out a Designated Port Switch:
  1. For switches connected to the same LAN segment, the switch with the lowest cost to reach the root, as advertised in the Hello they send onto the link, becomes the DP on that link.
  2. In case of a tie, among the switches that tied on cost, the switch with the lowest BID becomes the DP.
- Rules for a working EtherChannel configuration:
  1. On the local switch, all the `channel-group` commands for all the physical interfaces must use the same channel-group number.
  2. The channel-group number can be different on the neighboring switches.
  3. If using the `on` keyword, you must use it on the corresponding interfaces of both swithces.
  4. If you use the `desirable` keyword on one switch, the switch uses PAgP; the other switch must use either `desirable` or `auto`.
  5. If you use the `active` keyword on one switch, the switch uses LACP; the other switch must use either `active` or `passive`.
- Remember that, just like with trunking, one interface on the end of a PAgP or LACP link must be set to initiate dynamic configuration with either `desirable` or `active`.
- An interface must match all the below settings for the EtherChannel before it will be used as part of the channel-group:
  - Speed
  - Duplex
  - Operational access or trunking state (all must be access or trunks respectively)
  - If an access port, the access VLAN.
  - If a trunk port, the allowed VLAN list (per the `switchport trunk allowed` command)
  - If a trunk port, the native VLAN
  - STP interface settings
- The switch checks the neighboring switch's EtherChannel settings using PAgP or LACP if either is in use or CDP if not.
- **MAC Learning**: Frames received in a physical interface that is part of a PortChannel are considered to arrive on the PortChannel interface. So, MAC learning adds the PortChannel interface rather than the physical interface to the MAC address table.
- **MAC Forwarding**: The forwarding process will find a PortChannel port as an outgoing interface when matching the MAC address table. Then the switch must take the additional step to choose the outgoing physical interface, based on the load-balancing preferences configured for that PortChannel.
- VLAN Troubleshooting Steps:
  1. Identify all access interfaces and their assigned access VLANs and reassign into the correct VLANs if incorrect.
  2. Determine whether the VLANs both exist (either configured or learned with the VLAN Trunking Protocol (VTP)) and are active on each switch. If not, configure and activate the VLANs to resolve problems as needed.
  3. Check the allowed VLAN lists, on the switches on both ends of the trunk, and ensure that the lists of allowed VLANs are the same.
  4. Check for incorrect configuration settings that result in one switch operating as a trunk, with the neighboring switch not operating as a trunk.
  5. Check the allowed VLANs on each trunk, to make sure that the trunk has not administratively removed a VLAN from being supported on a trunk.
- Shutting down a VLAN on a switch causes it to not forward frames in that VLAN.
  - Done with the `shut vlan #` command.
***

## VLAN Trunking Protocol (VTP)

- The purpose of VTP is to share the configuration changes of a VLAN as they are modified on one switch with all other switches.
- A layer 2 protocol that causes all switches to synchronize their VLAN configuration to include the same VLAN IDs and VLAN names.
  - The process is similar to a routing protocol, with each switch sending periodic VTP messages.
  - VTP does not advertise the interface and access configurations of a VLAN.
- VTP limits the number of VLANs available from 1 to 1005.
- VTP servers allow for the creation of VLANs
- VTP clients do not allow VLAN configuration.
- VLAN server standard operating steps:
  1. For each trunk, send VTP messages, and listen to receive them.
  2. Check local VTP parameters versus the VTP parameters announced in the VTP messages received on a trunk.
  3. If the VTP parameters match, attempt to synchronize the VLAN configuration databases between the two switches.
- Done correctly, VTP causes all switches in the same administrative VTP domain, or the set of switches with the same domain name and password, to converge and have the exact same configuration of VLAN information.
- VTP uses a VLAN configuration database that has a configuration revision number.
  - This is incremented by 1 each time a server changes the VLAN configuration.
- VTP sends periodic messages every 5 minutes.
- Requirements for a new switch to join a VTP domain:
  - The link between the switches must be operating as a VLAN trunk (ISL or 802.1Q)
  - The two switches' case-sensitive VTP domain name must match.
  - If configured on at least one of the switches, both switches must have the same configured case-sensitive VTP password.
- VTP domains allow engineers to create separate groups of VTP configurations that are autonomous.  
- A transparent VTP setting allows a switch to not synchronize with other switches but it will still pass along VTP messages.
- VLAN Pruning: the appropriate switch trunk interfaces do not flood frames in a specific VLAN.
  - Done when a VLAN learned from other switches does not have access ports on a specific switch, so that one does not receive information for that VLAN's changes.

### VTP Configuration

- VTP Configuration Checklist:
  1. Use the `vtp mode {server | client}` command to enable VTP on a switch as either a server or client.
  2. On both clients and servers, use the `vtp domain <domain name>` command to establish the same domain name.
  3. (Optional) On both clients and servers, use the `vtp password <password>` command to create a domain password.
  4. (Optional) On servers, use the `vtp pruning` command to make the domain-wide VTP pruning choice.
  5. (Optional) On both clients and servers, use the `vtp version {1 | 2}` command in global configuration mode to tell the local switch whether to use VTP version 1 or 2.
- Best practice recommends at least two VTP server switches for redundancy.
- `show vtp status`: Shows the current VTP settings and configuration on a switch.
- When transmitting the VTP domain name or password, a switch will use MD5 to encrypt them before transmitting.
- `show vtp password`: Display the VTP password in clear text.
- `show vlan brief`: Display the Active VLANs to see if a switch learned them from VTP properly.
- VTP settings and the database are stored in the **vlan.dat** file in flash memory.
  - To delete, use the `delete flash:vlan.dat` command.
- VTP can be turned off using the `vtp mode off` command.
  - This is similar to transparent mode but the switch will not forward VTP messages as well.
- All devices need to have the same VTP version.

### VTP Troubleshooting

- VTP Troubleshooting Checklist:
  1. Confirm the switch names, topology (including which interfaces connect which switches), and switch VTP modes.
  2. Identify sets of two neighboring switches that should be either VTP clients or servers whose VLAN databases differ with the `show vlan` command.
  3. On each pair of two neighboring switches whose databases differ, verify the following:
    - Because VTP messages only flow over trunks, at least one operational trunk should exist between the two switches (use the `show interfaces trunk`, `show interfaces switchport`, or `show cdp neighbors` command).
    - The switches must have the same VTP domain name (`show vtp status`).
    - If configured, the switches must have the same VTP password (`show vtp password`)
    - The MD5 digest should be the same, as evidence that both the domain name and any configured passwords are the same on both switches (`show vtp status`).
    - While VTP pruning should be enabled or disabled on all servers in the same domain, having two servers configured with opposite pruning settings does not prevent the synchronization process.
  4. For each pair of switches identified in step 3, solve the problem by either troubleshooting the trunking problem or re-configuring a switch to correctly match the domain name or password.
***

## Miscellaneous LAN Topics

### IEEE 802.1x

- 802.1x security must be enabled on a switch and a AAA server (Authentication, Authorization, and Accounting) must be implemented.
  - The AAA server keeps usernames and passwords and will allow users who authenticate to it to use network resources.
  - The switch is configured with the AAA server's IP address and enables 802.1x on all interfaces connecting to end-user devices.
- Once implemented, a LAN switch acts as an 802.1x *authenticator*.
- End user devices use an 802.1x client called a *supplicant*.
- Protocols used to communicate between security devices:
  - Supplicant to Authenticator: Extensible Authentication Protocol (EAP) inside an Ethernet Frame.
  - Authenticator to Authentication Server: IP, UDP, RADIUS, EAP.

### AAA Authentication

- Cisco Access Control Server (ACS): A Cisco proprietary AAA server.
  - Uses TACACS+ (TCP) or RADIUS (UDP)
- TACACS+ and RADIUS comparisons:

| Features | TACACS+ | RADIUS |
| -------- | ------- | ------ |
| Most often used for | Network devices | Users |
| Transport protocol | TCP | UDP |
| Authentication port number(s) | 49 | 1645, 1812 |
| Protocol encrypts the password | Yes | Yes |
| Protocol encrypts entire packet | Yes | No |
| Supports function to authorize each user to a subset of CLI commands | Yes | No |
| Defined by | Cisco | RFC 2865 |

- `no aaa new-model`: Allows for default security settings. (Using `login` or `login local` to set local passwords or username/password combinations).
- `aaa new-model`: Enables AAA services on a local device.
- A AAA server must be configured after using the above command using the following steps:
  - Configure a single AAA server:
    - `tacacs server <server-name>`
    - `address ipv4 <address>`
    - `key <key value>`
    - `port <port number>`
  - Create a group of one or more servers:
    - `aaa group server <group-name>`
    - `server name <server-name-1>`
    - `server name <server-name-2>`
    - These are usually servers already configured on a device and put together as a group.
- To enable AAA login for device console or remote access, use the `aaa authentication login default <method 1> <method 2>` command.
  - `aaa authentication login default group WO-AAA-Group local`

### DHCP Snooping

- DHCP Snooping acts similar to a firewall or an ACL.
  - It watches for incoming messages on either all ports or some ports depending on the configuration.
  - It will look for DHCP messages and either allow or discard them based on snooping logic.
- DHCP snooping is a layer 2 feature, therefore it must be done on a device that sits between devices in the same VLAN.
- DHCP snooping begins with the assumption that end-user devices are untrusted while devices more in control of the IT department are trusted.
- Untrusted port meaning: Watch for incoming DHCP messages, and discard any that are considered to be abnormal for an untrusted port and therefore likely to be part of some kind of attack.
- DHCP snooping is first enabled globally on a switch or by VLAN. Once configured, all ports are considered untrusted until configured as trusted.
- Ports connected to legitimate DHCP servers should be trusted as well as ports connected to other networking devices.
- DHCP Snooping allows incoming Offer and Acknowledgment messages but filters messages if they arrive on untrusted ports.
  - This should discard any server-only traffic coming from non-server ports.
- DHCP snooping keeps state information in a DHCP Binding Table which is used to determine what traffic to discard.
- DHCP Snooping Features Summary:
  - **Trusted Ports**: Trusted ports allow all incoming DHCP messages.
  - **Untrusted ports, server messages**: Untrusted ports discard all incoming messages that are considered server messages.
  - **Untrusted ports, client messages**: Untrusted ports apply more complex logic for messages considered client messages. They check whether each incoming DHCP message conflicts with existing DHCP binding table information and, if so, discard the DHCP message. If the message has no conflicts, the switch allows the message through, which typically results in the addition of new DHCP binding table entries.
  - **Rate limiting**: Optionally limits the number of received DHCP messages per second, per port.

## Switch Stacking and Chassis Aggregation

- Switch Stacking: Make a stack of switches act like one switch.
  - The stack has a single management IP address.
  - Telnet and SSh can connect to just one switch, not multiple.
  - One configuration file would include the interfaces on all switches.
  - STP, CDP, and VTP would run on one switch, not multiple switches.
  - The switch ports would appear as if all are on the same switch.
  - The would be one MAC address table, and it would reference all ports on all physical switches.
- To make this happen, the switches are connected together using a special network. These use special stacking ports.
  - Usually gained by inserting a stacking module (Cisco FlexStack or FlexStack-Plus) into a switch and connect each with a stacking cable.
- The switches connect together to make a ring that is full duplex on each link and creates two paths to forward data between the physical switches in the stack.
- One switch acts as the stack master to control the rest of the switches.
  - The master switch does the MAC address matching to determine where to forward frames.
- **1:N Master Redundancy**: Every switch in the stack can act as the master. If the current master fails, another master is elected from the stack.
- Multichassis EtherChannels provide chassis redundancy in a VSS environment.

| | FlexStack | FlexStack-Plus |
| --- | --- | --- |
| Year Introduced | 2010 | 2013 |
| Switch model series | 2960-S, 2960-X | 2960-X, 2960-XR |
| Speed of single stack link, in both directions (full duplex) | 10 Gbps | 20 Gbps |
| Maximum number of switches in one stack | 4 | 8 |

- Chassis Aggregation: A type of Switch Stacking used for Switches in the Distribution and Core Layers.
  - Typically used for higher-end switches.
  - Does not require special hardware adapters, uses Ethernet interfaces instead.
  - Aggregates two switches
  - Arguably is more complex but also more functional.
- Advantages to using Switch Aggregation:
  - Multichassis EtherChannel (MEC): Uses the EtherChannel between the two physical switches.
  - Higher port density with better resource usage.
  - Active/Standby Control Plane: Simpler operation for control plane because the pair acts as one switch for the control plane protocols: STP, VTP, EtherChannel, ARP, routing protocols.
  - Active/Active date plane: Takes advantage of forwarding power of supervisors on both switches, with active Layer 2 and Layer 3 forwarding the xsupervisors of both switches. The switches synchronize their MAC and routing tables to support that process.
  - Single Switch Management: Simpler operation of management protocols by running management protocols (Telnet, SSH, SNMP) on the active switch; configuration is synchronized automatically with the standby switches.
  ***

## Open Shortest Path First (OSPF) version 2 (IPV4)

- Routers add IP routes to their routing tables using three methods:
  1. Connected Routes
  2. Static Routes
  3. Routes learned by using dynamic routing protocols.
- Routing Protocol: A set of messages, rules, and algorithms used by routers for the overall purpose of learning routes. This process includes the exchange and analysis of routing information. Each router chooses the best route to each subnet (path selection) and finally places those best routes in its IP routing table. Examples include: RIP, EIGRP, OSPF, and BGP.
- Routed protocol and rout-able protocol: Both terms refer to a protocol that defroutingines a packet structure and logical addressing, allowing routers to forward or route the packets. Routers forward packets defined by routed and rout-able protocols. Examples include IP versions 4 and 6.
- Routing protocol functions:
  1. Learn routing information about IP subnets from neighboring routers.
  2. Advertise routing information about IP subnets to neighboring routers.
  3. If more than one possible route exists to reach one subnet, pick the best route based on a metric.
  4. If the network topology changes, (a link fails, etc.), react by advertising that some routes have failed and pick a new currently best route. (Known as convergence)
- Routing Protocol Algorithm: the logic and processes used by a routing protocol to solve the problem of learning all routes, choosing the best route to a subnet, and converging in reaction to changes in the internetwork.
  - Distance Vector
  - Advanced Distance Vector
  - Link-state
- Metrics used by each Interior Gateway Protocol (IGP):

| IGP | Metric | Description |
| --- | ------ | ----------- |
| RIPv2 | Hop Count | The number of routers (hops) between a router and the destination subnet. |
| OSPF | Cost | The sum of all interface cost settings for all links in a route, routing with the cost defaulting to be based on interface bandwidth. |
| EIGRP | Composite of bandwidth and delay | Calculated based on the route's slowest link and the cumulative delay associated with each interface in the route. |

- Route redistribution: When two or more different routing protocols are used, the routes learned by a router through one routing protocol are advertised through another routing protocol to ensure all routers learn all possible routes.
- Link-state protocols build IP routes with a couple of major steps:
  1. The routers together build a lot of information about the network: routers, links, IP address, status information, and so on.
  2. The routers then flood the information so that all routers know the same information and then each router can calculate routes to all subnets from their own perspective.
- Link-state advertisements (LSA): The messages Link-state routing protocols send out to exhange the information they know about the network.
- Link-state database (LSDB): The collection of all LSAs known to a router.
- `show ip ospf database`: Display the details contained in a router's LSDB.
- Before flooding an LSA to its neighbors, a router will communicate by asking the neighboring routers if they already have a copy.
- The default aging timer for an LSA reflood is 30 minutes.
- Dijkstra Shortest Path First (SPF) algorithm: analyzes the LSDB and builds the rroutingoutes that the local router should add to the IP routing table.
  - Just like other protocols, these routes have a subnet number and mask, an outgoing interface, and a next-hop router IP address.
- OSPF Neighbors: routers that both use OSPF and both sit on the same data-link. (Connected to the same VLAN or on opposite ends of a serial link).
  - These routers send OSPF hello messages, and, if their features are compatible, they will become neighbors.
  - `show ip ospf neighbor`: Display OSPF neighbors to a router.
  - This allows new routers to be installed on a network without every router needing to be re-configured.
  - OSPF hellos contain a router's router ID (32 bit numbers) which can be configured and are based off on an active interface IPv4 address.
- OSPF Hello Message features:
  - Follows the IP packet header with IP protocol type 89.
  - Hello packets are sent to multicast IP address 224.0.0.5
  - OSPF routers listen for packets sent to IP multicast address 224.0.0.5
- Once two OSPF routes exchange hellos, they move to a 2-way state, whcih implies:
  - The router received a Hello from the neighbor, with that router's own RID listed as being seen by the neighbor.
  - The router has checked all the parameters in the Hello received from the neighbor, with no problems. The router is willing to become a neighbor.
  - If both routers reach a 2-way state with each other, it means that both routers meet all OSPF configuration requirements to become neighbors. Effectively, at that point, they are neighbors, and ready to exchange their LSDB with each other.
- Exchanging the LSDB between neighbors:
  - First the neighbors tell each other about the list of LSAs in their respective databases. (Using a database description, or DD packet)
  - If a neighbor has an LSA that the other doesn't, that neighbor asks for it to be exchanged. (In the form of a link-state request packet).
  - Link-state update (LSU): packets that send LSAs to neighbor routers.
  - This exchange results in the routers reaching what is known as a full-state.
- Dead interval: a timer that is usually 4 times the length of the hello interval and lets a router know its neighbor has likely failed.
- OSPF elects one of the routers on the same subnet to act as they *designated router (DR)*.
  - Done specifically on Ethernet links.
  - The DR manages the LSDB exchange process between all routers on the link.
  - A backup DR (BDR) is also nominated and takes over if the DR fails.routing
- Calculating the Best Routes with SPF:
  - SPF will calculate the best route to a subnet based on the collective metrics of each route link in each LSA.
- OSPF Areas: Designated with a specific number on an interface, OSPF routers in a single area will only perform SPF calculations for routers in their area.
- OSPF Area Design Principles:
  - Put all interfaces connected to the same subnet inside the same area.
  - An area should be contiguous
  - Some routers may be internal to an area, with all interfaces assigned to that single area.
  - Some routers may be Area Border Routers (ABR), because some interfaces connect to the backbone area, and some connect to nonbackbone areas.
  - All nonbackbone areas must connect to the backbone area (area 0) by having at least one ABR connected to both the backbone area and the nonbackbone area.

### Implementing OSPF version 2

- Configuration Checklist:
  1. Use the `router ospf <process-id>` global command to enter OSPF configuration mode for a particular OSPF process.
  2. (Optional) Configure the OSPF router ID by doing the following:
    - Use the `router-id <id-value>` command to define the router ID.routing
    - Use the `interface loopback <number>` global command, along with an `ip address <address> <mask>` command, to configure an IP address on a loopback interface (chooses the highest IP address of all working loopbacks).
    - Rely on an interface IP address (chooses the highest IP address of all working nonloopbacks)
  3. Use one or more `network <ip-address> <wildcard-mask> area <area-id>` to enable OSPFv2 on any interfaces matched by the configured address and mask, enabling OSPF on the interface for the listed area.
  4. (Optional) Use the `passive-interface <type> <number>` command to configure any OSPF interfaces as passive if no neighbors can or should be discovered on the interface.
- The wildcard mask is the same concept as the one used for ACLs.
- `show ip ospf neighbor`: Display neighbor information such as: Interface connected to it, address, state (full/two-way/etc.), and neighbor ID.
- `show ip ospf database`: Shows the entire contents of the OSPF LSDB (i.e. all LSAs).
- `show ip route`: Display all routes learned via OSPF.
- The process Cisco routers used to choose their OSPF router ID:
  1. If the `router-id <rid>` OSPF command is configured, this value is used as the RID.
  2. IF any loopback interfaces have an IP address configured, and the interface has an interface status of up, the router picks the highest numeric IP address among these loopback interfaces.
  3. The router picks the highest numeric IP address from all other interfaces whose interface status code (first status code) is up. (In other words, and interface in up/down state will be included by OSPF when choosing its router ID).
- Loopback interfaces can be configured with the `interface loopback <interface-numberrouting>` command.
- To restart the OSPF process, use the `clear ip ospf process` command.
- `show ip ospf`: display the OSPF Router ID.
- `passive-interface default`: sets all interfaces to default to passive for OSPF.
  - Individual interfaces can be set to not be passive with the `no passive-interface <interface number>` command.
- `show ip ospf interface`: lists a single line that mentions that an interface is passive or not.
- `show ip ospf interface brief`: lists all interfaces on which OSPF is enabled.
- For multi-area OSPF configurations, configure the internal routers the same way you would for a single-area configuration. The ABR should be configured to connect at least one interface to each area and one to the backbone area (0).
  - This is usually done using all 0 wildcard addresses to make each link advertise to its assigned area.
  - `network 10.1.1.1 0.0.0.0 area 0`
  - The real work comes down to planning the actual areas and assigning numbers appropriately.
- The best place to verify proper multi-area configuration is from the ABR. (Use the `shroutingow ip ospf interface brief` command and make sure all interfaces are in the appropriate states).
- DRs and BDRs are elected on a link upon the first setup of OSPF, using the highest Router ID to choose each one.
  - New routers added later do not cause a re-election.
- Typical Enterprise Network Design Goals:
  - All routers learn specific routes for subnets inside the company; a default router is not needed when forwarding packets to these destinations.
  - One router connects to the internet, and it has a default route that points toward the internet.
  - All routers should dynamically learn a default route, used for all traffic going to the Internet, so that all packets destined to locations in the Internet go to the one router connected to the Internet.
- The `default-information originate` command makes a router advertise a default route using OSPF.
  - The `default-information originate always` command tells a router to advertise its default route even if it is down.
- The `ip ospf cost <#>` command sets an interface's OSPF advertised cost.
- Reference Bandwidth: A settable OSPF value
- IOS formula for calculating OSPF cost: reference_bandwidth / interface_bandwidth
- To manipulate this forumla's output, the `bandwidth <speed>` command can be used on an interface to change the bandwidth so that it might get a different output for the OSPF formula.
- Rules for how a router sets its OSPF interface costs:
  1. Set the cost explicitly, using the `ip ospf cost <x>` interface subcommand, to a value between 1 and 65,535.
  2. Change the interface bandwidth with the `bandwidth <speed>` command, with speed being a number in kilobits per second (kbps)
  3. Change the reference bandwidth, using router OSPF subcommand `auto-cost reference-broutingandwidth <ref-bw>` with a unit of megabits per second (Mbps)
- OSPF can use load-balancing across multiple paths of the same cost using the `maximum-paths <#>` command.
- OSPF can be directly configured on an interface using the `ip ospf` command.
***

## Enhanced Interior Gateway Routing Protocol (EIGRP)

- Primary benefits of EIGRP:
  - Uses a robust metric based on both link bandwidth and link delay, so routers make good choices about the best route to use.
  - Converges quickly.
- Distance Vector routing protocol route information:
  - Distance: The metric for a possible route
  - Vector: The direction, based on the next-hop router for a possible route.
- EIGRP advertises changes to routes in *update* messages.
- Split Horizon: A distance vector feature that tells a routing protocol to not include routes that list the same interface as the outgoing interface in regular updates.
- Route poisoning: advertising a failed route with a metric of infinity so that other routers know that the route is no longer working.
  - Infinity is usually 16 for RIP as 15 is the highest hop count it supports.
  - For EIGRP, 2^32 - 1 is infinity (a little over 4 billion).
  - OSPFv2 uses 2^24 - 1 as infinity.
- EIGRP sends information about each route once, as it is learned, and then sends partial updates.
  - Partial Updates: sent when routes change (or new routes become available) to neighboring EIGRP routers.
- EIGRP Hello message: used to monitor the state of EIGRP neighbors set to a default interval.
  - A router must receive a Hello from a neighbor within a time called the Hold interval, which is usually three times the Hello interval.
- EIGRP has a multicast address of 224.0.0.10 that messages are sent to.
- EIGRP Operational Steps:
  1. **Neighbor Discovery**: EIGRP routers send Hello messages to discover potential neighboring EIGRP routers and perform basic parameter checks to determine which routers should become neighbors. Neighbors that pass all parameter checks are added to the EIGRP neighbor table.
  2. **Topology Exchange**: Neighbors exchange full topology updates when the neighbor relationship comes up, and then only partial updates as needed based on changes to the network topology. The data learned in these updates is added to the router's EIGRP topology table.routing
  3. **Choosing Routes**: Each router analyzes its respective EIGRP topology tables, choosing the lowest-metric route to reach each subnet. EIGRP places the route with the best metric for each destination into the IPv4 routing table.
- Conceptually, an EIGRP neighbor is another EIGRP-speaking router, connected to a common subnet, with which the first router is willing to exchange EIGRP topology information. EIGRP uses EIGRP Hello messages to dynamically discover potential neighbors, sending those updates to multicast address 224.0.0.10.
- EIGRP Neighbor determination checklist:
  - It must pass the authentication process if used.
  - It must use the same configured autonomous system number (a configuration setting)
  - The source IP address used by the neighbor's Hello must be in the same subnet as the local router's interface IP address/mask.
  - The router's EIGRP K-values must match.
- EIGRP sends update messages using the Layer 4 Reliable Transport Protocol (RTP) which provides a mechanism to resend any EIGRP messages that are not received by a neighbor.
- EIGRP Metric Calculation:
  - A composite metric that feeds multiple inputs into a math equation. (Typically link bandwidth and delay resulting in an integer value).
  - The bandwidth and delay for each link can be set using the `bandwidth` and `delay` commands.
- `show ip eigrp topology`: Display the delay settings in the number of microseconds.
- Feasible Distance (FD): The local router's composite metric of the best route to reach a subnet, as calculated on the local router.
- Reported Distance (RD): The next-hop router's best composite metric for that same subnet.
- For a particular subnet, the route with the best metric is called the *successor* whose metric is the Feasible Distance (FD).
- Similar routes to the same subnet are run through an algorithm to determine the best one, which is called the *feasible successor* route.
  - This route is chosen if its RD is less than the FD for that route.
- This process allows for near-immediate convergence if a route should fail.
- Diffusing Update Algorithm (DUAL): Used by EIGRP to choose a replacement route if a route fails and has no feasible successor.
  - A router sends EIGRP query messages to its working neighbors who send EIGRP reply messages if they have a working route to the subnet that the original sender needs to reach.

### EIGRP Configuration

- Configuration Checklist
  1. Use the `router eigrp <as-number>` command to enter EIGRP configuration mode and define the EIGRP autonomous system number (ASN).
  2. Configure one or more `network <ip-address> <wildcard-mask>` commands which will cause EIGRP to match any interface that has an address on the configured subnet.
  3. (Optional): Use the `eigrp router-id <value>` command to set the EIGRP router ID (RID) explicitly.
  4. (Optional): Use the `ip hello-interval eigrp <asn> <time>` and `ip hold-time eigrp <asn> <time>` commands to change the interface Hello and Hold timers.
  5. (Optional): Use the `bandwidth <value>` and `delay <value>` commands to impact metric calculations by tuning bandwidth and delay.
  6. (Optional): Use the `maximum-paths <number>` and `variance <multiplier>` commands to configure support for multiple equal-cost routes.
  7. (Optional): Use the `auto-summary` command to enable automatic summarization of routes at the boundaries of classful IPv4 networks.
- Routers that do not use the same ASN will not become neighbors.
- EIGRP tracks its current configuration with three tables: The Neighbor Table, the Topology table, and the IPv4 Routing Table.
- To check the overall EIGRP configuration, use the `show running-config` command.
- Show commands to check interfaces for EIGRP:
  - `show ip eigrp interfaces`
  - `show ip eigrp interfaces detail`routing
  - `show ip eigrp interfaces <interface>`
  - `show ip protocols`
- Show commands to see the current neighbor table:
  - `show ip eigrp neighbors`
  - `show ip eigrp neighbors <interface>`
  - `show ip protocols`
- Show commands to see the current EIGRP topology table:
  - `show ip eigrp topology`
  - `show ip eigrp topology <subnet/prefix>`
  - `show ip eigrp topology | section <subnet>`
- Show the current route metric calculations:
  - `show ip route`
  - `show ip route eigrp`
  - `show ip route <subnet> <mask>`
  - `show ip route | section <subnet>`
- EIGRP also allows for passive interfaces using the `passive-interface <interface>` command.
- EIGRP Router ID choice process:
  1. The value configured with the `eigrp router-id <#>` command.
  2. The numerically highest IP address of an up/up loopback interface at the time the EIGRP process comes up.
  3. The numerically highest IP address of a nonloopback interface at the time the EIGRP process comes up.
- Equal-cost load balancing: tells EIGRP to treat all the routes equally that tie as successor routes.
- Variance: allows routes who's metrics are relatively close in value to be considered equal, allowing multiple unequal-metric routes to the same subnet to be added to the routing table.
  - Set using the `variance <multiplier>` command.
  - Multiplied by the current FD.
  - Any FS routes whose calculated metric is less than or equal to the variance are added to the routing table, depending on the *maximum-paths* setting.
***

## Troubleshooting IPv4 Routing Protocols

- General Troubleshooting Steps:
  1. Examine the internetwork design to determine on which interfaces the routing protocol should be enabled and which routers are expected to become neighbors.
  2. Verify whether the routing protocol is enabled on each interface. If it isn't, determine the root cause and fix the problem.
  3. Verify that each router has formed all expected neighbor relationships. If it hasn't, find the root cause and fix the problem.
- For any interfaces matched by a `network` command, a routing protocol will try the following two actions:
  1. Attempt to find potential neighbors on the subnet connected to the interface.
  2. Advertise the subnet connected to the interface.
- A `passive-interface` command will disable a router finding neighbors through an interface.
- Show commands to find if a routing protocol is set on an interface:

| Command | Key Information | Lists Passive Interfaces? |
| ------- | --------------- | ------------------------- |
| `show ip eigrp interfaces` | Lists the interfaces on which EIGRP is enabled (based on the `network` commands), excluding passive interfaces. | No |
| `show ip ospf interface brief` | Lists the interfaces on which the OSPFv2 is enabled (based on the `network` router subcommands or `ip ospf` interface subcommands), including passive interfaces. | Yes |
| `show ip protocols` | Lists the contents of the `network` configuration commands for each routing process, and lists enabled but passive interfaces. | Yes |

- EIGRP and OSPF Neighbor Requirements:

| Requirement | EIGRP | OSPF |
| ----------- | ----- | ---- |
| Interfaces must be in an up/up state. | Yes | Yes |
| Interfaces must be in the same subnet. | Yes | Yes |
| Access control lists (ACL) must not filter routing protocol messages. | Yes | Yes |
| Must pass routing protocol neighbor authentication (if configured). | Yes | Yes |
| Must use the same ASN/PID on the `router` configuration command. | Yes | No |
| Hello and hold/dead timers must match | No | Yes |
| Router IDs (RID) must be unique | No | Yes |
| K-values must match. | Yes | N/A |
| Must be in the same area. | N/A | Yes |

- Commands to Isolate EIGRP Neighbor Problems:

| Requirement | Best Commands to Isolate the Problem |
| ----------- | ------------------------------------ |
| Must be in the same subnet. | `show interfaces, show ip interface` |
| Must use the same ASN on the `router` configuration command. | `show ip eigrp interfaces`, `show ip protocols` |
| Must pass EIGRP neighbor authentication | `debug eigrp packets` |
| K-values must match. | `show ip protocols` |

- Commands to Isolate OSPF Neighbor Problems:

| Requirement | Best Show Command | Best Debug Command |
| ----------- | ----------------- | ------------------ |
| Must be in the same subnet. | `show interfaces` | `debug ip ospf hello` |
| Hello and dead timers must match. | `show ip ospf interface` | `debug ip ospf hello` |
| Must be in the same area. | `show ip ospf interface breif` | `debug ip ospf adj` |
| Router IDs must be unique. | `show ip ospf` | N/A (use log messages) |
| Must pass any neighbor authentication. | `show ip ospf interface` | `debug ip ospf adj` |

- To restart the OSPF process after fixing an issue, use the `clear ip ospf process` command.
- OSPF and EIGRP can be started and stopped without losing configuration by using the `shut/no shut` commands.
- To change the MTU size for an interface, use the `ip mtu` or `ipv6 mtu` for IPv4 and 6 respectively.
***

## External Border Gateway Protocol (BGP)

- The only option for an exterior gateway protocol (EGP) in use today.
- **Reach-ability**: A EGP concept that is similar to learning routes in IGP's.
  - Used to learn all public IP address prefixes reachable in the Internet.
- BGP routers send out BGP protocol messages called *updates* to exchange routing information with other routers.
- Other routers are called *BGP Neighbors* or *BGP Peers*.
- BGP advertises routes to routers in other companies.
- **Network Layer Reachability Information (NLRI)**: BGP term for subnets.
- Typically, BGP is used for a company to advertise its public IPv4 prefix to its local ISP.
- Autonomous system numbers (ASN) are used for making BGP best path selection processes and to prevent routing loops.
- *iBGP*: uses BGP to advertise routes to other routers inside the same ASM.
- **Path Attributes**: different facts about a route to reach a subnet.
- **Best Path Selection**: The process by which BGP makes comparisons (about 10) until a route from a group of routes to the same subnet is chosen as the best route.
  - One path attribute is the *AS_Path* which is similar to hop counts but the hops are entire ASNs.
- **Internet Edge**: the connection between an ISP customer and an ISP.
- **Single-Homed**: an Internet edge design with a single link to one ISP.
- Typically EGP will advertise just one route to a public block of IPv4 address to reduce the amount of Internet routes.
- ISPs will typically advertise a default route via EGP to ASNs in order for all of their unknown traffic to be sent to the ISP.

### BGP Configuration

- BGP uses TCP to transport its messages between two BGP peers using well-known port `179`.
  - Once BGP is configured, it opens port `179` and waits for incoming connection requests from other routers. Once a peer connects, the TCP connection is formed.
- BGP Neighbor establishment and update process steps:
  1. Because of proper neighbor configuration in two routers, the routers create a TCP connection with each other and become BGP peers.
  2. Because of an additional configuration, possibly the BGP `network` command, one router adds one or more NLRI and associated PAs to its local BGP table.
  3. eBGP on that router advertises all the best routes in its BGP table, that is, the routes it considers as best for each NLRI, in a BGP update message sent to the other router.
  4. BGP on the other router processes that received update, adding a BGP table entry for the new subnet.
- `router bgp <asn>`: Begins BGP configuration.
- `neighbor <peer-ip-address>`: subcommand that defines the IP address of a BGP neighbor.
- To establish more peers, use the `bgp <peer-ip-address> remote-as <asn>` command.
- To disable a BGP peer on a link, use the `bgp <peer-ip-address> shutdown` command.
- `show tcp brief`: show all TCP connections in order to display a working BGP connection.
- `show ip bgp summary`: Lists each BGP peer on one line each.
- BGP router ID selection process:
  1. Use the value in the `bgp router-id <rid>` command.
  2. If unset per Step 1, choose the highest IPv4 address among all loopback interfaces in an interface up state.
  3. If unset per Steps 1 and 2, use the same logic as Step 2, but for all nonloopback interfaces in an interface up state.
- BGP Neighbor States:

| BGP Neighbor State | Typical Reasons |
| ------------------ | --------------- |
| Idle | The neighbor has been administratively disabled (`neighbor shutdown`), or the router is waiting before the next retry. |
| Connect | The TCP connection is being attempted but has not completed. |
| Active | The TCP connection has completed, but no BGP messages have been sent yet. |
| Opensent | The TCP connection exists, and this router has sent the first message to establish the BGP neighbor relationship (a BGP open message). |
| Openconfirm | The TCP connection exists and the local router has received an Open message from the other router. The neighbor relationship may still be rejected. |
| Established | The routers are now neighbors/peers and can exchange update messages. |

- `network <prefix> mask <mask>`: BGP subcommand to list the address prefix to advertise.
- Multiple network commands can be used to advertise subnets instead of an entire classful address block.
- Discard route: static route used by BGP to discard any packets that match the route.
  - `network <address> mask <mask> null0`
  - Used to force BGP to add a route to its BGP table when multiple subnets for a public address block exist.
- To advertise an ISP learned default route inside an AS, use the `default-information originate` command.
***

## Wide-Area Networks

### Implementing Point-to-Point WANs

- A physical leased-line WAN works a lot like an Ethernet crossover cable connecting two routers, but with no distance limitations.
- Different Names for a Leased Line:

| Name | Meaning or Reference |
| ---- | -------------------- |
| Leased circuit, circuit | The words *line* and *circuit* are often used as synonyms in telco terminology; circuit makes reference to the electrical circuit between the two endpoints. |
| Serial link, serial line | The words *link* and *line* are also often used as synonyms. *Serial* in this case refers to the fact that the bits flow serially and that routers use serial interfaces. |
| Point-to-point link, point-to-point line | Refers to the fact that the topology stretches between two points, and two points only. (Some older leased lines allowed more than two devices.) |
| T1 | A specific type of leased line that transmits data at 1.544 megabits per second (1.544 Mbps) |
| WAN link, link | Both these terms are very general, with no reference to any specific technology. |

- **Customer Premises Equipment (CPE)**: This telco term refers to the gear that sits at their customer's sites on the ends of the link.
- **Channel service unit/data service unit (CSU/DSU)**: This device provides a function called *clocking*, in which it physically controls the speed and timing at which the router serial interface sends and receives each bit over the serial cable.
- **Serial Cable**: This is a short cable that connects the CSU and the router serial interface.
- **WAN interface cards (WIC), High-speed WICs (HWIC), or Network Interface Modules (NIM)**: Names used for the removable card on a router that provides a serial interface.
- **Time-Division Multiplexing (TDM)**: Technology that lets telcos combine multiple T-carrier system speeds onto a single line.
  - **Digital Signal Level 1 (DS1) or (T1)**: combines 24 DSOs (at 64 Kbps) plus 8 Kbps of overhead onto one physical line that runs at 1.544 Mbps.
  - Essentially, each line could run at slower or faster speeds as long as they are multiples of 64 Kbps.
- T-series Leased line speeds:

| Names of Line | Bit Rate |
| ------------- | -------- |
| DSO | 64 Kbps |
| Fractional T1 | Multiples of 64 Kbps, up to 24X |
| DS1 (T1) | 1.544 Mbps (24 DS0s, for 1.536 Mbps, plus 8 Kbps overhead) |
| E1 (Europe) | 2.048 Mbps (32 DS0s) |
| Fractional T3 | Multiples of 1.536 Mbps, up to 28X |
| DS3 (T3) | 44.736 Mbps (28 DS1s, plus management overhead) |
| E3 (Europe) | Approx. 34 Mbps (16 E1s, plus management overhead) |

- **Data Circuit-Terminating Equipment (DCE)**: Controls the speed of the router serial interface.
- **Data Terminal Equipment (DTE)**: Controlled by the clocking singals from the CSU (DCE).
- **High-Level Data-Link Control (HDLC)**: Provides a layer 2 protocol for leased lines.
  - The frame header lets the receiving router know that a new frame is coming.
  - Includes a Frame Check Sequence (FCS) for error detection.
  - Cisco adds a type field in the HDLC header to support multiple types of network layer packets to cross the HDLC link. (I.E. IPv4 and IPv6).
- HDLC Config Checklist:
  1. Use the `ip address <address> <mask>` command in interface configuration mode to configure the interface IP address.
  2. The following tasks are required only when the specifically listed conditions are true:
    - If an `encapsulation <protocol>` interface subcommand already exists, for a non-HDLC protocol, use the `encapsulation hdlc` command in interface configuration mode to enable HDLC. Alternatively, use the `no encapsulation <protocol>` command in interface configuration mode to use the default setting of HDLC as the data link protocol.
    - If the interface line status is administratively down, use the `no shutdown` command in interface configuration mode to enable the interface.
    - If the serial link is a back-to-back serial link in a lab (or a simulator), use the `clock rate <speed>` command in interface configuration mode to configure the clocking rate. Use this command only on the one router with the DCE cable (per the `show controllers serial <number>` command).
  3. The following steps are always optional and have no impact on whether the link works and passes IP traffic:
    - Use the `bandwidth <speed in kbps>` command in interface configuration mode to configure the link's documented speed so that it matches the actual clock rate of the link.
    - For documentation purposes, use the `description <text>` command in interface configuration mode to configure a description of the purpose of the interface.
- **Point-to-Point Protocol (PPP)**: Plays a similar role to HDLC as a data-link protocol for use on serial links but it was designed with routers, TCP/IP, and other network layer protocols in mind.
- PPP Concepts:
  - Definition of a header and trailer that allows delivery of a data frame over the link.
  - Support for both synchronous and asynchronous links.
  - A protocol Type field in the header, allowing multiple Layer 3 protocols to pass over the same link.
  - Built-in authentication tools: **Password Authentication Protocol (PAP)** and **Challenge Handshake Authentication Protocol (CHAP)**.
  - Control protocols for each higher-layer protocol that rides over PPP, allowing easier integration and support of those protocols.
- PPP Framing: defines a protocol field that identifies the type of packet inside the frame.
  - Also includes an FCS for error detection
- PPP Control Protocols:
  - **Link Control Protocol (LCP)**: Includes several functions that each are focused on the data link itself, ignoring the Layer 3 protocol sent across the link.
  - **Network Control Protocols (NCP)**: This is a category of protocols, one per network layer protocols, one per network layer protocol. Each protocol performs functions specific to its related Layer 3 protocol.
    - **IP Control Protocol (IPCP)**
    - **IPv6CP**
    - **CDPCP**: for CDP
- LCP Functions, Features, and Descriptions:

| Function | LCP Feature | Description |
| -------- | ----------- | ----------- |
| Looped link detection | Magic number | Detects whether the link is looped, and disables the interface, allowing rerouting over a working route. |
| Error detection | Link-quality monitoring (LQM) | Disables an interface that exceeds an error percentage threshold, allowing rerouting over better routes. |
| Multilink support | Multilink PPP | Load balances traffic over multiple parallel links |
| Authentication | PAP and CHAP | Exchanges names and passwords so that each device can verify the identity of the device on the other end of the link. |

- WAN Authentication is most often needed when dial lines are used.   
  - PAP works by having the to-be-authenticated device starting the messages, claiming to be legitimate by listing a secret password in clear text. The receiving router sends back an acknowledgment that the sending router has passed the authentication process.
  - CHAP has the device doing the authentication start with a challenge message, which requires the device to authenticate to send a reply which includes a hashed version of the password. The router doing the authentication has been pre-configured with the other router's name and password in order to validate it. It will send a third message to confirm the authentication is successful.
    - Also uses a randomly generated number to hash the password so that it is different each time authentication is required.
- Implementing PPP: Uses the same setup as HDLC but requires the `encapsulation ppp` command on both ends of a link.
- Configure PPP with CHAP:
  1. Use the `encapsulation ppp` command in interface configuration mode, on the serial interfaces on both routers, to enable PPP on the interfaces.
  2. Define the usernames and passwords used by the two routers:
    - Use the `hostname <name>` command in global configuration mode on each router, to set the local router's name to use when authenticating.
    - Use the `username <name> password <password>` command in global configuration mode on each router, to define the name (case-sensitive) used by the neighboring router, and the matching password (case-sensitive). (The name in the `username` command should match the name in the neighboring router's `hostname` command.)
  3. Use the `ppp authentication chap` command in the interface configuration mode on each router to enable CHAP on each interface.
- Configure PPP with PAP: Requires the `ppp authentication pap` command and the `ppp pap sent-username <username> password <password>` command. The receiving router will compare the username and password sent to the various username/password commands entered on it.
- **Multilink PPP (MLPPP)**: A PPP feature useful when using multiple parallel serial links between two devices.
  - Provides reduced Layer 3 complexity by making the multiple serial interfaces on each router look like a single interface from a layer 3 perspective. It does this by using a virtual interface called a *multilink interface*.
  - Balances the frames sent at Layer 2 over the multiple links by fragmenting the frame into multiple smaller frames, one per active link.
- Configuring MLPPP:
  1. Configure matching multilink interfaces on the two routers, configuring the interface subcommands for all layer 3 features (IPv4, IPv6, and routing protocol) under the multilink interfaces (and not on the serial interfaces).
  2. Configure the serial interfaces with all Layer 1 and 2 commands, like `clock rate` and `ppp authentication`.
  3. Configure some PPP commands on both the multilink and serial interfaces, to both enable MLPPP and associate the multilink interface with the serial interfaces.
  - Example:
    - Layer 3: `interface multilink 1`, `encapsulation ppp`, `ppp multilink`, `ip address 192.168.5.1 255.255.255.0`, `ppp multilink group 1`.
    - Layer 2: `interface serial0/0/0`, `encapsulation ppp`, `ppp multilink`, `no ip address`, `ppp multilink group 1`.
- `show ppp all`: Verify PPP settings.
- `show ppp multilink`: Show PPP Multilink settings.
- **Link-quality monitoring (LQM)**: Disables an interface that exceeds an error percentage threshold, allowing rerouting over better routes.
- **Magic Number**: Detects whether the link is looped, and disables the interface, allowing rerouting over a working route.
- Serial Link Verification and Troubleshooting process:
  1. From one router, ping the other router's serial IP address.
  2. If the ping fails, examine the interface status on both routers and investigate problems related to the likely problem.
  3. If the ping works, also verify that any routing protocols are exchanging routes over the link.
- HDLC and PPP link behavior when IP addresses on each end to not reside in the same subnet:

| Symptoms When IP Addresses on a Serial Link are in different subnets | HDLC | PPP |
| -------------------------------------------------------------------- | ---- | --- |
| Does a ping of the other router's serial IP address work? | No | Yes |
| Can routing protocols exchange routes over the link? | No | No |

### Private WANs with Ethernet and MPLS

- **Metro Ethernet (MetroE)**: includes a variety of WAN services with some common features.
  - Each MetroE service uses Ethernet physical links to connect the customer's device to the service provider's device.
  - MetroE acts much as if the WAN service were created by one Ethernet switch.
  - To use a Metro Ethernet service, each site needs to connect to the service with (at least) one Ethernet link.
    - Facilitated by the ISP by connecting customer links to an Ethernet switch that is as physically near the customer Point of Presence (PoP) as possible.
- Metro Ethernet Services:

| MEF Service Name | MEF Short Name | Topology Terms | Description |
| ---------------- | -------------- | -------------- | ----------- |
| Ethernet Line Service | E-Line | Point-to-point | Two customer premise equipment (CPE) devices can exchange Ethernet frames, similar in concept to a leased line. |
| Ethernet LAN Service | E-LAN | Full mesh | Acts like a LAN, in that all devices can send frames to all other devices. |
| Ethernet Tree Service | E-Tree | Hub-and-spoke; partial mesh; point-to-multipoint | A central site can communicate to a defined set of remote sites, but the remote sites cannot communicate directly. |

- **Ethernet Virtual Connection (EVC)**: Defines which user (customer) devices can communicate with which.
- **Committed Information Rate (CIR)**: Defined by a bandwidth profile to set a speed that will be charged to a customer.
- Quality of Service Policing: watch incoming frames and identify the frames associated with each EVC. If the bytes sent are over the CIR, then discard enough of the currently arriving frames to keep it down to the CIR.
- Quality of Service Shaping: tells routers on the MetroE access link to send some frames, then wait, then send more.
- A MPLS Layer 3 virtual private network (VPN) creates a layer 3 WAN service.
  - Uses a different subnet on each access link.
- Routers on the edge of the MPLS network add and remove an MPLS header as packets enter and exit the MPLS network in order to keep customer traffic separate.
  - Devices inside the MPLS network then use the label field inside the MPLS header when forwarding data across the MPLS network.
- **Customer Edge (CE)**: typically a router that sits at a customer site.
- **Provider Edge (PE)**: a device that sits at the edge of the SP's network on the other end of the access link connected to a CE.
- MPLS offers QoS features for VoIP and similar traffic.
- A CE router does become neighbors with the PE router on the other end of the access link.
- A CE router does not become neighbors with other CE routers.
- The MPLS network will advertise the customer's routes between the various PE routers, so that the CE routers can learn all customer routes through their PE-CE routing protocol neighbor relationship.
- To advertise the customer routes between the PE routers, the PE routers use another routing protocol along with a process called *route redistribution*. This happens inside one router, taking routes from one routing protocol process and injecting them into another.
  - Uses a variation of BGP called **Multiprotocol BGP (MPBGP)**.
- For OSPF, the MPLS PE's form a backbone area by the name of a super backbone.
  - Each PE-CE link can be any area, a non-backgone area or the backbone area.
- For EIGRP, it is more efficient to use the same ASN at all sites when using MPLS.
- MPLS uses labels to separate and forward customer traffic.
- MPLS can carry multiple protocols, including IPv4 and IPv6
- MPLS operates between Layer 2 and Layer 3.

### Private WANs with Internet VPN

- **DSL Access Multiplexer (DSLAM)**: used to split the signals of a DSL line between analog and digital.
- **VPN Tunnel Concepts**: Two devices near the edge of the internet create a VPN by adding headers to packets that include fields that allow the VPN devices to make the traffic secure. They encrypt the original IP packet as it travels between the devices.
  - *Tunnel* refers to any protocol's packet that is sent by encapsulating the packet inside another packet.
- **Site-to-site VPN**: Connects two sites of an organization.
- **IPsec**: an architecture or framework for security services for IP networks.
  - Defines how two devices, both of which connect to the Internet, can achieve the main goals of a VPN: confidentiality, authentication, data integrity, and anti-replay.
  - Uses a pair of encryption algorithms, one to encrypt the data and one to decrypt it.
- **Cisco AnyConnect Secure Mobility Client (or AnyConnect Client)**: software that runs on a host PC and uses SSL to create one end of a VPN tunnel.
- **Generic Routing Encapsulation (GRE)**: defines an additional header used by GRE to perform tunneling, along with the new IP header, that encapsulates the original packet.
  - Two routers work together, with matching configuration settings, to create a GRE IP tunnel.
  - IPsec configuration can be added to encrypt the traffic.
  - Routers use virtual interfaces called *tunnel interfaces* which have IP address assigned to them in the same subnet.
    - These interfaces do not exist until the `interface tunnel #` command is used.
  - GRE uses a 20-byte IP header called a *delivery header* that uses IP addresses from the unsecure network. This is used by internet routers to route the packet normally.
- GRE Tunnel Config Checklist:
  1. Use the `interface tunnel #` command to create a tunnel interface. The interface numbers have local meaning only and do not have to match between the two routers.
  2. (Optional) Use the `tunnel mode gre ip` command in tunnel interface mode to tell IOS to use GRE encapsulation on the tunnel. (This is the default setting).
  3. Use the `ip address <address> <mask>` interface subcommand to assign an IP address to the tunnel interface, using a subnet from the secure network's address range. The two routers on the tunnel should use addresses from the same subnet.
  4. Configure the tunnel's source IP address in the unsecured part of the network in one of two ways. Regardless of the method, the local router's source IP address must match the other router's tunnel destination.
    1. Use the `tunnel source <ip-address>` command to directly set the tunnel's source IP address.
    2. Use the `tunnel source <interface-id>` command to indirectly set the tunnel's source IP address by referencing an interface on the local router.
  5. Use the `tunnel destination {<ip address> | <hostname>}` command to configure the tunnel's destination IP address in the unsecured part of the network. (This value must match the IP address used by the other router as its tunnel source IP address).
  6. Add routes that use the tunnel by enabling a dynamic routing protocol on the tunnel or by configuring static IP routes.
- Verify the tunnel is working by using the `show ip interface brief` command or the `show interfaces tunnel <interface number>` command.
  - `traceroute` can also be used to verify a tunnel is working.
- For ACLs on tunnel interfaces, make sure a rule specifically allowing GRE traffic is set.
- Reasons that can cause a GRE tunnel to be in an up/down state:
  - The tunnel address is routed through the tunnel itself.
  - The tunnel interface is down.
  - A valid route to the destination address is missing from the routing table.
- **Dynamic Multipoint VPN (DMVPN)**: An IOS feature that uses the *Next Hop Resolution Protocol (NHRP)*. NHRP works as follows:
  1. One site on the tunnel acts as the hub site and as NHRP server.
  2. The spoke sites initially can communicate only with the hub site.
  3. The spoke sites (as NHRP clients) register their matching public and private IP addresses with the NHRP server (using NHRP protocol messages).
- **PPP over Ethernet (PPPoE)**: extends the PPP protocol to be sent over Ethernet links.
  - Has a typical Ethernet header, a short PPPoE header, and then the usual PPP frame.
  - PPP Authentication is performed during the PPP Session phase.
- Dialer interfaces can be dynamically bound to use another interface. For example, for PPPoE, a dialer interface will hold configuration for IP and PPP, but it is not a physical Ethernet interface.
- PPPoE Configuration Checklist:
  1. Configure Layer 1 details:
    - Configure a dialer interface:
      - Use the `interface dialer #` command to create the dialer interface; choose a number not already used on the local router by some other dialer interface.
      - Use the `dialer pool #` command to refer to a pool of Ethernet-family interfaces that can be used for PPPoE.
    - Configure the physical interface(s):
      - Use the `pppoe-client dial-pool-number #` command to add the interface to the same pool number configured on the dialer interface.
  2. Configure Layer 2 details.
    - Configure PPP on the dialer interface.
      - Use the `encapsulation ppp` interface subcommand to enable PPP on the dialer interface.
      - Use the `ppp chap hostname <name>` interface subcommand to define the username with which to authenticate to the ISP.
      - Use the `ppp chap password <password>` interface subcommand to define the password with which to authenticate to the ISP.
    - Configure PPPoE on the Ethernet interface(s):
      - Use the `pppoe enable` interface subcommand to enab,le PPPoE. (IOS adds this command automatically when the `pppoe-client` interface subcommand is configured).
  3. Configure Layer 3 details.
    - Configure IP on the dialer interface.
      - Use the `ip address negotiated` interface subcommand to tell the dialer interface to use PPP's IPCP to learn the IP address to use.
      - Use the `mtu 1492` interface subcommand to change from the default of 1500 to allow for the 8 extra header bytes used by PPPoE.
    - Disable IP on the Ethernet Interface(s).
      - Use the `no ip address` interface subcommand to disable IP routing on the physical interface and remove the IPv4 address from the interface.
- Show commands for PPPoE verification:
  - `show interfaces dialer #`: lists output for dialer interfaces and their associated virtual-access interfaces.
  - `show interfaces virtual-access #`: show details specifically about a virtual-access interface.
  - `show interfaces virtual-access # configuration`: shows the configuration for a dynamically created virtual-access interface.
  - `show pppoe session [interface <interface number>]`: Show the status of a PPPoE session on a designated interface.
  - `show ip route`: verify that a dialer interface has a connected route between the two routers.
  - `show ip interface brief dialer #`: show a brief output of a dialer interface status.
- The virtual-template interface contains the information used to assign an IP address to a PPPoE client which is contained in the PPPoE server configuration.
***

## IPv4 Services: ACLs and QoS

- Cisco routers can apply ACL logic to packets at the point at which the IP packets enter an interface, or the point at which they exit an interface.
- Two ACL actions: `deny` and `permit`.
- Types of ACLS:
  - Standard numbered ACLs (1-99)
  - Extended numbered ACLs (100-199)
  - Additional ACL numbers (1300-1999 standard, 2000-2699 extended)
  - Named ACLs
  - Improved editing with sequence numbers.
- Standard ACLs only match the Source IP of a packet.
- A single ACL is both a single entity and, at the same time, a list of one or more configuration commands.
  - They choose the first entry that matches a packet starting from the top of the list.
  - There is an implicit deny all at the bottom of all ACLs.
- Standard Numbered ACL Syntax: `access-list {1-99 | 1300-1999} {permit | deny} <source IP> <wildcard mask>`
  - The same number ACL can be added to by additional `access-list` commands using the same number.
- In a wildcard mask, `0` in an octet means that the router must compare the octet as normal and `255` in an octet means that the router ignores the octet, considering it already a match.
  - When using a wildcard mask, the `access-list` command's loosely defined source parameter should be a 0 in any octets where the WC mask is a 255.
- Finding the right wildcard mask to match a subnet:
  1. Use the subnet number as the source value in the `access-list` command.
  2. Use a wildcard mask found by subtracting the subnet mask from 255.255.255.255
    - 255.255.255.255 - 255.255.252.0 = 0.0.3.255
    - `access-list 1 permit 172.16.8.0 0.0.3.255`
- To match any and all packets with an ACL command, use the `any` keyword.
- Implementing Standard IP ACLs:
  1. Plan the location (router and interface) and direction (in or out) on that interface:
    - Standard ACLs should be placed near to the destination of the packets so that they do not unintentionally discard packets that should not be discarded.
    - Because standard ACLs can only match a packet's source IP address, identify the source IP addresses of packets as they go in the direction that the ACl is examining.
  2. Configure one or more `access-list` global configuration commands to create the ACL, keeping the following in mind:
    - The list is searched sequentially, using first-match logic.
    - The default action, if a packet does not match any of the `access-list` commands, is to `deny` (discard) the packet.
  3. Enable the ACL on the chosen router interface in the correct direction, using the `ip access-group # {in | out}` interface subcommand.
- `show ip access-lists` / `show access-lists`: Show the access lists configured on a router.
- `show ip interface #`: Show if an ACL is set on an interface.
- `access-list # remark <string>`: Leave a comment for an ACL for other people to read.
- Extended ACLs require the use of the `host` keyword when setting IP addresses.
- Extended ACL Configuration Syntax: `access-list {100-199 | 2000-2699} {deny | permit} {tcp | udp | icmp} <source IP> <source wildcard> <destination IP> <destination wildcard> [log | log-input]`
- Extended ACL Configuration Sytax for TCP/UDP specifically: `access-list {100-199 | 2000-2699} {deny | permit} {tcp | udp} <source IP> <source wildcard> [gt | lt | eq | ne | range] [port #] <destination IP> <destination wildcard> [gt | lt | eq | ne | range] [port #] [established] [log]`
- Extended ACL Configuration Checklist:
  - Place extended ACLs as close as possible to the source of the packets that will be filtered. Filtering close to the source of the packets saves some bandwidth.
  - Remember that all fields in one `access-list` command must match a packet for the packet to be considered to match that `access-list` statement.
  - Use numbers of `100-199` and `2000-2699` on the `access-list` commands.
- Named ACLs begin with a `ip access-list {standard | extended} <name>` command.
  - List entries take the form of: `{permit | deny} {standard ACL parameters | extended ACL parameters}`.
  - The `remark <string>` entry can be used as well to add a comment.
- Entries in a Named ACL are deleted using the `no <ACL rule>` access list subcommand.
- Numbered ACLs can remove entries by using the `no <sequence-number>` command with the corresponding rule sequence number.
  - Use the `show ip access-lists` command to determine the right number.
- To start a numbered ACL, use the `ip access-list {standard | extended} #` command, with the number corresponding to the appropriate range for standard or extended ACLs.

### Quality of Service (QoS)

- **Quality of Service (QoS)**: refers to tools that network devices can use to manage several related characteristics of what happens to a packet while it flows through a network.
- QoS tools are used to manage: *Bandwidth*, *Delay*, *Jitter*, and *Loss*.
- **Bandwidth**: the speed of a link in bits per second (bps). Also consists of the capacity of the link.
- **Delay**: Consists of *One-Way Delay*, or the time between sending one packet and that same packet arriving at the destination host, and *Round-Trip Delay*, or the one-way delay plus the time for the receiver of the first packet to send back a packet.
- **Jitter**: The variation in one-way delay between consecutive packets sent by the same packet.
- **Loss**: The amount of lost messages, usually as a percentage of packets sent.
- **Quality of Experience (QoE)**: a term referring to user's perception of their use of the application on the network.
- Types of traffic considered by QoS: Network Application, Data application (batch traffic), voice, and video.
- Voice Over IP (VoIP) Process:
  1. The phone user makes a phone call and begins speaking.
  2. A chip called a *codec* processes (digitizes) the sound to create a binary code (160 bytes with the G.711 codec, for example) for a certain time period (usually 20 ms).
  3. The phone places the data into an IP packet.
  4. The phone sends the packet to the destination IP phone.
- Recommended QoS settings for Voice Applications:
  - **Delay (one-way)**: 150 ms or less
  - **Jitter**: 30 ms or less
  - **Loss**: 1% or less
- Recommended QoS settings for Video Applications:
  - **Bandwidth**: 384 kbps to 20+ Mbps
  - **Delay (one-way)**: 200-400 ms
  - **Jitter**: 30-50 ms
  - **Loss**: 0.1% - 1%
- **Classification**: The process of matching the fields in a message to make a choice to take some QoS action.
- **Marking**: When QoS tools change one or more header fields, setting a value in the header.
  - - **Type of Service (ToS)**: The 8-bit field in an IP packet header used for QoS.
  - **Differentiated Services Code Point (DSCP)**: The first 6-bits of the ToS field in the IP header meant for QoS marking.
- **Network Based Application Recognition (NBAR)**: matches packets for classification in a large variety of ways that are very useful for QoS. Cisco specific.
- **802.1Q Header**: A 4 byte header with a significant part that sits in the third byte of the field as a 3-bit field, supplying eight possible values to mark. This field is known as *Class of Service or CoS* and *Priority Code Point or PCP*.
- QoS Fields:

| Field Name | Header(s) | Length (bits) | Where Used |
| ---------- | --------- | ------------- | ---------- |
| DSCP | IPv4, IPv6 | 6 | End-to-end packet |
| IPP | IPv4, IPv6 | 3 | End-to-end packet |
| CoS | 802.1Q | 3 | Over VLAN trunk |
| TID | 802.11 | 3 | Over Wi-Fi |
| EXP | MPLS Label | 3 | Over MPLS WAN |

- **Trust Boundary**: The point in the path of a packet flowing through the network at which the networking devices can trust the current QoS markings.
  - Traffic that arrives with a tag is untagged at the edge of an administrative domain.
- **Expedited Forwarding (EF)**: A DiffServ defined DSCP value suggested for use for packets that need low latency (delay), low jitter, and low loss. Decimal 46.
- **Assured Forwarding (AF)**: A DiffServ defined set of 12 DSCP values meant to be used in concert with each other. Includes four separate queues in a queuing system and three levels of drop priority within each queue for use with congestion avoidance tools.
  - Set as AFXY, with X referring to the queue (1 through 4) and Y referring to the drop priority (1 through 3).
- **Class Selector (CS)**: A DiffServ defined DSCP value that consists of the Type of Service (ToS) byte and eight defined values. (CS0-7)
- **Congestion management (queuing)**: the QoS toolset for managing the queues that hold packets while they wait their turn to exit an interface (and in other cases in which a router holds packets waiting for some resource).
- **Prioritization**: giving priority to one queue over another in some way.
- **Round Robin Scheduling (Prioritization)**: Cycle through each queue in order, taking turns with each queue. Also includes the concept of *weighting* where the scheduler takes a different number of packets (or bytes) from each queue, giving more preference to one queue over another.
- **Low Latency Queuing**: treats one or more queues as special *priority queues*.
  - **Queue Starvation**: When to much traffic comes in a priority queue resulting in other queues never being serviced. Solved using **policing**.
- **Policing**: Traffic arrives at networking devices at a varying rate, with valleys and spikes. The policer measures that rate, and make a similar measurement. If traffic volume is over the configured rate, the policer begins to discard packets.
  - Policers tend to allow a burst of traffic beyond the policing rate for a short time after a period of low activity.
  - Typically used at the edge between networks.
  - Policers can also re-mark packets that have been modified earlier in the policing process.
- **Shaping**: The process of slowing messages down by queuing them. The shaper then services the shaping queues based on the shaping rate. This is used with policing to ensure that a sending rate does not exceed the shaping rate.
  - For voice and video traffic, configure a short time interval for shaping (Tc). Use a 10-ms time interval to support voice and video.
- **Congestion Avoidance**: A QoS feature that attempts to reduce overall packet loss by preemptively discarding some packets used in TCP connections.
  - Relates to the **TCP Window**, or a number of bytes the sender can send over a TCP connection before receiving an acknowledgment.
  - Typically when data loss is detected, the TCP Window decreases by one half.
  - Congestion Avoidance tools will deliberately discard some TCP packets to purposely reduce the transmission of TCP streams to prevent *tail drop*, or when packets arrive to a full queue and are discarded.
- **Class-Based WFQ (CBWFQ)**: extends the standard WFQ functionality to provide support for user-defined traffic classes.
- **Priority Queuing (PQ)**: packets belonging to one priority class of traffic are sent before all lower priority traffic to ensure timely delivery of those packets.
  - Can be used to optimize voice traffic on a network that is primarily intended for data traffic.
- **Committed Access Rate (CAR)**: is only used for bandwidth limitation by dropping excessive traffic.
- **Resource Reservation Protocol (RSVP)**: allows applications to reserve bandwidth for their data flows. Good for reserving bandwidth for VoIP calls across a call path.
***

## IPv4 Routing in the LAN

- **Router on a stick (ROAS)**: Configuring an ethernet interface on a router to act as a trunk and set up subinterfaces with one in each vlan to send traffic between vlans.
- ROAS Config checklist:
  1. Use the `interface <type> <number.subint>` command in global configuration mode to create a unique subinterface for each VLAN that needs to be routed.
  2. Use the `encapsulation dot1q <vlan_id>` command in subinterface configuration mode to enable 802.1Q and associate one specific VLAN with the subinterface.
  3. Use the `ip address <address> <mask>` command in subinterface configuration mode to configure IP settings (address and mask).
- To set a router interface for the native VLAN, two options exist:
  - Configure the `ip address` command on the physical interface, but without an `encapsulation` command; the router considers this physical interface to be using the native VLAN.
  - Configure the `ip address` command on a subinterface, and use the `encapsulation dot1q <vlan-id> native` subcommand to tell the router both the VLAN ID and the fact that it is the native VLAN.
- To Verify ROAS, use the `show running-config`, `show ip route [connected]`, and `show vlans` commands.
- **Layer 3 Switch**: Also known as a multilayer switch, is one device, but executes logic at two layers: Layer 2 LAN switching and Layer 3 IP routing.
  - Used to route IP traffic between VLANs.
  - Layer 3 switching functionality needs a virtual interface connected to each VLAN internal to the switch. These *VLAN Interfaces* act like router interfaces, with an IP address and mask.
  - The Layer 3 switch has an IP routing table, with connected routes off each of these VLAN interfaces.
  - These interfaces are also referred to as **Switched Virtual Interfaces (SVI)**.
- Layer 3 Config checklist using SVIs:
  1. Enable IP routing on the switch, as needed:
    - Use the `sdm prefer lanbase-routing` command (or similar) in global configuration mode to change the switch forwarding ASIC settings to make space for IPv4 routes at the next reload of the switch.
    - Use the `reload` EXEC command in enable mode to reload (reboot) the switch to pick up the new `sdm prefer` command setting.
    - Once reloaded, use the `ip routing` command in global configuration mode to enable the IPv4 routing function in IOS software and to enable key commands like `show ip route`.
  2. Configure each SVI interface, one per VLAN for which routing should be done by this Layer 3 switch:
    - Use the `interface vlan <vlan_id>` command in global configuration mode to create a VLAN interface, and to give the switch's routing logic a Layer 3 interface connected into the VLAN of the same number.
    - Use the `ip address <address> <mask>` command in VLAN interface configuration mode to configure an IP address and mask on the VLAN interface, enabling IPv4 routing on that VLAN interface.
    - (As needed) Use the `no shutdown` command in interface configuration mode to enable the VLAN interface (if it is currently in a shutdown state).
- Use the `show ip route` command to verify that the interfaces are active and adding routes to the routing table.
- The Layer 3 switch would also need additional routes to the rest of the network, either by static routes or enabling a routing protocol.
- **Routed Port**: A port configured on a switch to not perform layer 2 switching logic on a frame sent to it's MAC address and instead do the following:
  1. Strip off the incoming frame's Ethernet data link header/trailer.
  2. Make a Layer 3 forwarding decision by comparing the destination IP address to the IP routing table.
  3. Add a new Ethernet data link header/trailer to the packet.
  4. Forward the packet, encapsulated in a new frame.
- **Layer 3 EtherChannels**: EtherChannels that are also routed ports instead of switched ports.
- To enable a switch interface to be a routed interface on a Layer 3 switch, just use the `no switchport` subcommand on the physical interface.
- This can be verified using the `show interfaces`, `show interfaces status`, `show ip route`, and `show interfaces <type> <number> switchport` commands. The routed port should show up differently than a switch port in all these command outputs.
- Configuring a Layer 3 Switch Routed Interface EtherChannel:
  1. Configure the physical interfaces as follows, in interface configuration mode:
    - Add the `channel-group <number> mode on` command to add it to the channel. Use the same number for all physical interfaces on the same switch, but the number used (the channel-group number) can differ on the two neighboring switches.
    - Add the `no switchport` command to make each physical port a routed port.
  2. Configure the PortChannel interface:
    - Use the `interface port-channel <number>` command to move to port-channel configuration mode for the same channel number configured on the physical interfaces.
    - Add the `no switchport` command to make sure the port-channel interface acts as a routed port. (IOS may have already added this command).
    - Use the `ip address <address> <mask>` command to configure the address and mask.
- To verify the EtherChannel, use the `show etherchannel summary` command.
- **First Hop Redundancy Protocol (FHRP)**: A category of protocols that can be used so that the hosts take advantage of redundant default routers.
  - All hosts act like they always have, with one default router setting that never has to change.
  - The default routers share a virtual IP address in the subnet, defined by the FHRP.
  - Hosts use the FHRP virtual IP address as their default router address.
  - The routers exchange FHRP protocol messages, so that both agree as to which router does what work at any point in time.
  - When a router fails, or has some other problem, the routers use the FHRP to choose which router takes over responsibilities from the failed router.
- FHRP Protocol Options:

| Acronym | Full Name | Origin | Redundancy Approach | Load Balancing |
| ------- | --------- | ------ | ------------------- | -------------- |
| HSRP | Hot Standby Router Protocol | Cisco | Active/standby | Per subnet |
| VRRP | Virtual Router Redundancy Protocol | RFC 5798 | Active/standby | Per subnet |
| GLBP | Gateway Load Balancing Protocol | Cisco | Active/active | Per host |

- **Hot Standby Router Protocol (HSRP)**: allows two or more routers to cooperate, all being willing to act as the default router.
  - The HSRP active router implements a virtual IP address and matching virtual MAC address.
  - This virtual IP address is in the same subnet as the interface IP address, but different from the interface.
  - When a router fails, the standby router sends an Ethernet frame with the virtual interface as the source MAC to update the mac address tables on the switches.
  - HSRP can load balance by preferring different routers as the default router for different VLANs.
- To configure HSRP on the routers that will share the default router responsibility, use the `standby <group> ip <virtual-ip>` interface subcommand.
  - The first value defines the HSRP group number, which must match on both routers.
- `show standby <brief>`: Display HSRP settings.
- Routers with higher HSRP priority become the active router.
- `standby # priority #`: Sets the priority for a HSRP group.
- `no standby # preempt`: Tells a router to not challenge an already active router to become the active router.
- `standby # preempt`: Tells a router to negotiate for active router when it discovers another HSRP configured router.
- `standby # name <string`: Sets a character name for a HSRP group.
- `standby version {1 | 2}`: Sets HSRP to either version 1 or 2.
- HSRP Configuration Mistakes and Symptoms:

| Scenario | Routers Both Become Active? | Duplicate Address Detected? | VIP Changes Depending on Active Router? |
| -------- | --------------------------- | --------------------------- | --------------------------------------- |
| HSRP version mismatch | Yes | Yes | N/A |
| HSRP group number mismatch | Yes | Yes | N/A |
| ACL blocks HSRP packets | Yes | No | N/A |
| Routers configure different VIPs | No | N/A | Yes |

- HSRP Uses port 1985.
***

## IPv6 Routing Operation and Troubleshooting

- **Global Unicast Addresses**: work like public IPv4 addresses in that the enterprise obtains a unique prefix with all addresses inside the enterprise beginning with that prefix.
  - To assign a Global Unicast Address, a company starts with the **global routing prefix**, which was assigned to them, and then breaks the address into the prefix, the subnet bits, and the interface ID of each device.
  - Typically the Global Routing Prefix and the Subnet bits make up the first 64 bits of the address.
- **Unique Local Unicast**: work like private IPv4 addresses.
- **Link-Local Addresses**: Special addresses created by each device for each interface to use for traffic that exists only on the local subnet.
  - Each link local address begins with the `FE80` 16-digit prefix.
  - Most hosts will use the **EUI-64** process to create their Interface IDs.

| Type | First Digits | Similar to IPv4 Public or Private? |
| ---- | ------------ | ---------------------------------- |
| Global Unicast | 2 or 3 | Public |
| Unique local unicast | FD | Private |
| Link-local | FE80 | N/A |

- IPv6 hosts have three options to set their IPv6 settings: static configuration, stateful DHCP, and SLAAC.
- **Stateful DHCPv6**: Same general process as DHCP for IPv4
  1. A DHCP server or servers exist somewhere in the internetwork.
  2. User hosts use DHCP messages to ask for a lease of an IP address and information about other settings.
  3. The server replies, assigning an address to the host and informing the host of the other settings.
- The primary difference between DHCPv4 and Stateful DHCPv6 is that DHCPv6 does not supply the default router information, instead NDP does this.
- If the DHCPv6 server sits on another subnet, the router on the subnet with the requesting host must be configured as a DHCPv6 relay agent in order to obtain an address for the requesting host.
- **Stateless Address Autoconfiguration (SLAAC)**: defines an overall process that also uses NDP and DHCPv6 with a stateless service.
  - NDP informs a host of the subnet prefix, prefix length, and default router.
  - The Stateless DHCPv6 server informs the host of the available DNS servers.
- To build it's own address using SLAAC, a host uses the following steps:
  1. Learn the IPv6 prefix used on the link, from any router, using NDP Router Solicitation (RS) and Router Advertisement (RA) messages.
  2. Choose an interface ID value to follow the just-learned IPv6 prefix, either by randomly choosing a number, or by using the host's MAC address and using EUI-64 rules.
- **EUI-64**: Steps:
  1. Split the 6-byte (12 hex digits) MAC address in two halves (6 hex digits each).
  2. Insert FFFE in between the two, making the interface ID now have a total of 16 hex digits (64 bits).
  3. Invert the seventh bit of the first byte.
- IPv6 Routing Config Checklist:
  1. Use the `ipv6 unicast-routing` global command to enable IPv6 routing.
  2. Use the `ipv6 address <address/prefix length>` subcommand in interface configuration mode to enable IPv6 on each desired interface, and set the IPv6 interface address and prefix length.
- **Dual-Stack**: Running both IPv4 and IPv6 on a single network.
- To have an interface use EUI-64 to figure out its own interface ID, use the `ipv6 address <address/prefix> eui-64` interface subcommand.
  - `ipv6 address 2001:db8:1111:1::/64 eui-64`
- Three basic options for IPv6 static routes about how to tell a router where to send packets next:
  1. Direct packets out of an interface on the local router.
    - `ipv6 route 2001:db8:1111:3::/64 S0/0/1`
  2. Direct the packets to the unicast address of a neighboring router.
    - `ipv6 route 2001:db8:1111:3::/64 2001:db8:1111:2::2`
  3. Direct the packets to the link-local address of a neighboring router (requires the outgoing interface, as well).
    - `ipv6 route 2001:db8:1111:3::/64 S0/0/1 FE80::FF:FE00:2`
- **Troubleshooting Focused Issues**:
  - **Host-Focused Issues**:
    1. Hosts should be in the same IPv6 subnet as their default router.
    2. Hosts should use the same prefix length as their default router.
    3. Hosts should have a default router setting that points to a real router's address.
    4. Hosts should have correct Domain Name Service (DNS) server addresses.
  - **Router-Focused Issues**:
    1. Router interfaces in use should be in an up/up state.
    2. Two routers that connect to the same data link should have addresses in the same IPv6 subnet.
    3. Routers should have IPv6 routers to all IPv6 subnets as per the IPv6 subnet design.
  - **Filtering Issues**:
    1. Watch for MAC address filtering on the LAN switches.
    2. Watch for missing VLANs in switches.
    3. Watch for IPv6 access control lists (ACL) in routers.
***

## Implementing OSPFv3 for IPv6

- In 2010, OSPFv3 added support for IPv4 through a feature called *address families*.
- Routers will have separate Link-state databases (LSDB) for IPv4 and IPv6 along with separate SPF calculations, separate neighbor relationships and separate OSPF messages between routers.
  - Shortest Path First (SPF)
  - Link-state advertisements (LSA)
- **OSPFv3 Config Checklist**:
  1. Use the `ipv6 router ospf <process-id>` global command to create an OSPFv3 process number and enter OSPF configuration mode for that process.
  2. Ensure that the router has an OSPF router ID, through either:
    - Configuring the `router-id <id-value>` router subcommand in OSPFv3 configuration mode.
    - Configuring an IPv4 address on a loopback interface (chooses the highest IPv4 address of all working loopbacks)
    - Relying on an interface IPv4 address (chooses the highest IPv4 address of all working nonloopbacks).
  3. Configure the `ipv6 ospf <process-id> area <area-number>` interface subcommand on each interface on which OSPFv3 should be enabled, to both enable OSPFv3 on the interface and set the area number for the interface.
  4. (Optional) Use the `passive-interface <type> <number>` router subcommand to configure any OSPFv3 interfaces as passive if no neighbors can or should be discovered on the interface.
- OSPFv3 Router IDs are set the same way as OSPFv2 as a 12-bit number that looks like an IPv4 address.
  - `router-id 3.3.3.3`
- Remember: All IPv6 configurations should begin with the `ipv6 unicast-routing` command.
- Methods to influence the metric for an OSPFv3 route:
  1. Set the cost explicitly using the `ipv6 ospf cost #` interface subcommand to a value between 1 and 65,535, inclusive.
  2. Change the interface bandwidth with the `bandwidth <speed>` command, with speed being a number in kilobits per second (Kbps), and let the router calculate the value based on the OSPFv3 reference-bandwidth / interface-bandwidth.
  3. Change the reference bandwidth with router OSPFv3 subcommand `auto-cost reference-bandwidth <ref-bw>`, with a unit of megabits per second (Mbps).
- OSPFv3 uses the `maximum-paths #` router subcommand to define how many equal-cost routes it will add to its IPv6 routing table in order to perform load balancing.
- The `default-information originate` command in OSPFv3 configuration mode will enable a router to advertise its default route.
  - The prefix for a default route with IPv6 is ::/0, with a prefix length of 0, somewhat like the 0.0.0.0/0 used with IPv4.
- **OSPFv3 Verification and Troubleshooting Useful Commands**:
  - **Config**:
    - `show running-config`
  - **Enabled Interfaces**:
    - `show ipv6 protocols`
    - `show ipv6 ospf interface`
    - `show ipv6 ospf interface <type> <number>`
    - `show ipv6 ospf interface brief`
  - **Neighbors**:
    - `show ipv6 ospf neighbor`
    - `show ipv6 ospf neighbor <type> <number>`
  - **LSDB**:
    - `show ipv6 ospf databse`
  - **Routes**:
    - `show ipv6 route`
    - `show ipv6 route ospf`
    - `show ipv6 route <subnet> <mask>`
    - `show ipv6 route | section <subnet>`
- The Maximum Transmission Unit (MTU) size can be changed for IPv6 using the `ipv6 mtu <size>` interface subcommands.
  - The MTU settings must match between two interfaces on a link for a neighbor relationship to occur.
***

## Implementing EIGRP for IPv6

- EIGRP for IPv6 uses many of the same conventions as EIGRP for IPv4, such as **successors** and **feasible successors (FS)** as well as using **Diffusing Update Algorithm (DUAL)** processing when no FS exists.
- Config checklist:
  1. `ipv6 router eigrp #`: Define Autonomous System Number
  2. `eigrp router-id #.#.#.#`: Set the router ID (Optional)
  3. `interface <type> <number>`: Enter an interface's configuration mode.
  4. `ipv6 eigrp #`: Enable EIGRP for IPv6 on an interface, using the same ASN as defined in step 1.
- Just like OSPFv3, EIGRP for IPv6 doesn't use network commands to determine which interfaces will use EIGRP, and instead does the enabling and configuration on an interface-by-interface basis.
- EIGRP for IPv4 and IPv6 Commands and Functions:

| Function | EIGRP for IPv4 | EIGRP for IPv6 |
| -------- | -------------- | -------------- |
| Create process, define ASN | `router eigrp <as-number>` | `ipv6 router eigrp <as-number` |
| Define router ID explicitly (router mode) | `eigrp router-id #` | Same |
| Change number of concurrent routes (router mode) | `maximum-paths` # | Same |
| Set the variance multiplier (router mode) | `variance <multiplier>` | Same |
| Influence metric calculation (interface mode) | `bandwidth <value> / delay <value>` | Same |
| Change Hello and hold timers (interface mode) | `ip hello-interval eigrp <asn> <time>` / `ip hold-time eigrp <asn> <time>` | Change `ip` to `ipv6` |
| Enable EIGRP on an interface | `network <ip-address> [wildcard-mask]` | `ipv6 eigrp <as-number>` (interface subcommand) |
| Disable and enable automatic summarization (router mode) | `[no] auto-summary` | Not needed for EIGRP for IPv6 |

- EIGRP for IPv6 routers can become neighbors even if they have IPv6 addresses in different subnets.
- **EIGRP for IPv6 Verification and Troubleshooting Useful Commands**:
  - **Config**:
    - `show running-config`
  - **Enabled Interfaces**:
    - `show ipv6 eigrp interfaces`
    - `show ipv6 eigrp interfaces detail`
    - `show ipv6 eigrp interfaces <type> <number>`
    - `show ipv6 protocols`
  - **Neighbors**:
    - `show ipv6 eigrp neighbors`
    - `show ipv6 eigrp neighbors <type> <number>`
    - `show ipv6 protocols`
  - **Topology**:
    - `show ipv6 eigrp topology`
    - `show ipv6 eigrp topology <subnet/prefix>`
    - `show ipv6 eigrp topology | section <subnet>`
  - **Routes**:
    - `show ipv6 route`
    - `show ipv6 route eigrp`
    - `show ipv6 route <subnet>/<prefix>`
    - `show ipv6 route | section <subnet>/<prefix>`
***

## IPv6 Access Control Lists

- Differences between IPv4 and IPv6 ACLs:
  - IPv4 ACLs can only match IPv4 packets and IPv6 ACLs can only match IPv6 packets.
  - IPv4 ACLs can be identified by number or name, while IPv6 ACLs use names only.
  - IPv4 ACLs identify that an ACL is either standard or extended based on the ACL number range or by using the `standard` or `extended` keyword. IPv6 ACLs have a similar standard and extended ACL concept, but do not differentiate their styles with a different configuration keyword.
  - IPv4 ACLs can match on specific values unique to an IPv4 header (e.g., option, precedence, ToS TTL, fragments)
  - IPv6 ACLs can match on specific values unique to an IPv6 header (e.g., flow label, DSCP) as well as extension and option header values.
  - IPv6 ACLs have some implicit `permit` statements at the end of each ACL, just before the implicit deny all at the end of the ACL, while IPv4 ACLs do not have implicit `permit` statements.
- Be sure not to block all ICMPv6 messages, such as NDP, as this would inhibit IPv6 from working properly.
- For IPv6 ACLs, insteald of using a wildcard mask you use a prefix notation, (/48, /52, /64, etc.) to indicate the prefix fixed bits to be examined.
- All IPv6 ACLs are named.
- Standard IPv6 ACL commands:
  - Begin with: `ipv6 access-list <name>`
  - `[permit | deny] ipv6 {source-ipv6-prefix/prefix-length | any | host source-ipv6-address} {destination-ipv6-prefix/prefix-length | any | host destination-ipv6-address} [log]`
- Set the same way as IPv4 ACLs with: `ipv6 traffic-filter <name> {in | out}`
- Useful show commands:
    - `show running-config`
    - `show ipv6 interface <type> <number>`
    - `show ipv6 interface | include line | list`
    - `show ipv6 access-list`
- Extended IPv6 ACL commands:
    - Begin with: `ipv6 access-list <name>`
    - `[permit | deny] protocol {source-ipv6-prefix/prefix-length | any | host source-ipv6-address} [operator [port-number]] {destination-ipv6-prefix/prefix-length | any | host destination-ipv6-address} [operator [port-number]] [dest-option-type [doh-number | doh-type]] [dscp <value>] [flow-label <value>] [fragments] [log] [log-input] [mobility] [mobility-type [mh-number | mh-type]] [reflect <name> [timeout <value>]] [routing] [routing-type <routing-number>] [sequence <value>] [time-range <name>]`
- Extended IPv6 ACLs specific to each protocol:
  - **TCP**: `[permit | deny] tcp [eq | gt | lt | neq {port | protocol}] [range {port | protocol]}]`
  - **UDP**: `[permit | deny] udp [eq | gt | lt | neq {port | protocol}] [range {port | protocol]}]`
  - **ICMP**: `[permit | deny] icmp [icmp-type [icmp-code] | icmp-message]`
- The three implicit rules on IPv6 ACLs (To allow NDP to work):
  - `permit icmp any any nd-na`
  - `permit icmp any any nd-ns`
  - `deny ipv6 any any`
- To allow NDP RS and RA messages, add the following lines to an ACL:
  - `permit icmp any any router-advertisement`
  - `permit icmp any any router-solicitation`
- (Or just apply the ACL in an outbound direction to not block messages generated by the router).
***

## Network Management

### SNMP

- **Simple Network Management Protocol (SNMP)**: an application layer protocol that provides a message format for communication between what are termed *managers* and *agents*.
  - **SNMP Manager**: a network management application running on a PC or server, with that host typically being called a **Network Management Station (NMS)**.
  - **SNMP Agent**: software running inside each device (router, switch, etc.) with knowledge of all the variables on that device that describe the device's configuration, status, and counters.
  - The SNMP manager uses SNMP protocols to communicate with each SNMP agent.
  - **Management Information Base (MIB)**: a database kept by each SNMP Agent that keeps track of variables that make up the parameters, status, and counters for the operations of the device.
- The NMS uses the **SNMP Get**, **GetNext**, and **GetBulk** messages to ask for information from an agent.
- The NMS sends an **SNMP Set** message to write variables on the SNMP agent
- SNMP agents send a Trap or Inform SNMP message to the NMS to list the state of certain MIB variables when those variables reach a certain state.
  - SNMP Trap messages use a fire-and-forget process; SNMP Inform messages also use UDP but add application layer reliability. The NMS must acknowledge receipt of the Inform or the SNMP agent will time out and resend the Inform.
- The MIB defines each variable as an object ID (OID) which are organized into a hierarchy, usually shown as a tree. The tree structure sequence can be described either by name or by number.
- **SNMP Community**: A common password that needs to be known by both the SNMP agent and the SNMP manager in order to communicate. Used in SNMPv1 and SNMPv2c
  - **Read-Only (RO) Community**: Allows Get messages.
  - **Read-Write (RW) Community**: Allows both reads and writes (Gets and Sets).
- **SNMPv3** replaces communities with the following features:
  - **Message integrity**: This mechanism, applied to all SNMPv3 messages, confirms whether or not each message has been changed during transit.
  - **Authentication**: An optional feature that adds authentication with both a username and password, with the password never sent as clear text. Instead, it uses a hashing method.
  - **Encryption (privacy)**: An optional feature that encrypts the contents of SNMPv3 messages.
- **Securing SNMP**:
  1. Use ACLs to limit SNMP messages to those from known servers only.
  2. Use SNMPv3
- **SNMPv2c Config Checklist**
  1. Use the `snmp-server community <communitystring> RO [ipv6 acl-name][acl-name]` command in global configuration mode to enable the SNMP agent, set the read-only community string, and restrict incoming SNMP messages based on the optional referenced IPv4 or IPv6 ACL.
  2. (Optional) Use the `snmp-server community <communitystring> RW [ipv6 acl-name][acl-name]` command in global configuration mode to enable the SNMP agent, set the read-write community string, and restrict incoming SNMP messages based on the optional referenced IPv4 or IPv6 ACL.
  3. (Optional) If referenced by an `snmp-server community` command, configure an IPv4 or IPv6 ACL, with the same name or number referenced by the `snmp-server community` command, with the ACL permitting by matching the source IPv4 or IPv6 address of the allowed SNMP management hosts.
  4. (Optional) Use the `snmp-server location <text-describing-location>` command in global configuration mode to document the location of the device.
  5. (Optional) Use the `snmp-server contact <contact-name>` command in global configuration mode to document the person to contact if problems occur.
- **Configuring SNMPv2c Support for Trap and Inform**:
  1. Use the `snmp-server host {hostname | ip-address} [informs] version 2c <notification-community>` command in global configuration mode to configure the SNMP agent to send either SNMPv2c Traps (default) or Informs to the listed host. Use this command once for each host to which this device should send Traps.
  2. Use the `snmp-server enable traps` command in global configuration mode to enable the sending of all supported types of Trap and Inform messages.
- Useful Show Commands:
  - `show snmp`: Shows status and counter information
  - `show snmp community`: show community string values
  - `show snmp host`: lists the IP address or hostname of the NMS
  - `show snmp location`
  - `show snmp contact`
- **SNMPv3 Config Checklist**:
  1. Use the `snmp-server group <groupname> v3 {noauth | auth | priv} [write v1default] [access [ipv6] acl-name]` command in global configuration mode to enable the SNMP agent, create a named SNMPv3 group of security settings, get the security level, optionally override the default write view with the same view as defaulted for use as the read MIB view (v1default), and optionally restrict incoming SNMP messages based on the optional referenced IPv4 or IPv6 ACL.
  2. To configure users whose referenced SNMPv3 group has a security level of `noauth`, use the `snmp-server user <username> <groupname> v3` command in global configuration mode, making sure to reference an SNMPv3 group with security level of `noauth` configured.
  3. To configure users whose referenced SNMPv3 group use the security level of `auth`:
    - Use the `snmp-server user <username> <groupname> v3 auth {sha | md5} <password>` command in global configuration mode to configure the user and authentication password.
  4. To configure users that use the security level of `priv`, you will add parameters to the end of the `snmp-server user` command syntax as configured in Step 3, as follows:
    - Add the `priv des <encryption-key>`, `priv 3des <encryption-key>`, or `priv aes {128 | 192 | 256} <encryption-key>` parameters to the end of the `snmp-server user` command based on the desired level of encryption.
  5. Enable the SNMP agent to send notification messages (Traps and/or Informs) to an NMS as follows:
    - Use the `snmp-server host {hostname | ip-address} [informs | traps] version 3 {noauth | auth | priv} <username>` command in global configuration mode to configure the SNMP agent to send SNMPv3 traps and/or informs while using the same security level setting as the link SNMPv3 group.
    - Use the `snmp-server enable traps` command in global configuration mode to enable the sending of all supported notifications to all hosts defined in `snmp-server host` commands.

### IP Service Level Agreement (SLA)

- The IP Service Level Agreement (IP SLA) feature of Cisco routers provides a means to measure and display several key performance and availability indicators.
  - Typically used as measurements to determine if an SLA has been met over some period of time.
- The IP SLA runs on a router and generates traffic that mimics user traffic, but without placing any software on end-user devices.
- Many IP SLA probes rely on one router to generate the packets (the IP SLA source) and another router replying back (the IP SLA responder).
- IP SLA Example (echo from one subnet to another):
  - `ip sla 1`
    - `icmp-echo 10.1.3.2 source-ip 10.1.1.1`
    - `frequency 60`
    - `threshold 300`
    - `history filter all`
    - `history buckets-kept 6`
    - `history lives-kept 1`
  - `ip sla schedule 1 life forever start-time now`
  - Use `no ip sla schedule 1` to stop this IP SLA probe.
  - `show ip sla summary`: Display IP SLA activity information.
  - `show ip sla statistics #`: Show the statistics of a specified IP SLA.
  - `ip sla restart #`: Restarts a specified IP SLA probe.
  - `show ip sla history #`: Displays the activity history of a specified IP SLA operation.

### SPAN (Switch Port Analyzer)

- Enables a switch to make copies of some of the Ethernet frames flowing through the switch, sending them out a specific port.
- **SPAN Session**: A collection of SPAN rules that define one or more source ports, as well as define the direction of traffic on each port that should be then copied to the destination port.
- SPAN supports monitoring VLANs by monitoring all access ports to a VLAN on a switch. When more ports are added for that VLAN, the SPAN logic will adjust without requiring reconfiguration.
- **Remote SPAN (RSPAN)**: forwards the SPAN traffic over a VLAN at Layer 2.
- **Encapsulated RSPAN (ERSPAN)**: expects Layer 3 switches and a Layer 3 forwarding path between the switches, so it encapsulates the SPAN traffic in a GRE tunnel to forward the traffic to another switch.
- **Local SPAN Configuration**:
  - Use the `monitor session` global commands to configure SPAN, with each command defining one ore more SPAN sources of the same type (either port or VLAN, but not a mix), plus a single SPAN destination port.
  - Dependencies:
    - A SPAN destination port can be used with only one SPAN session at a time.
    - A SPAN destination port cannot also be a SPAN source port.
    - When configured as a SPAN destination port, the switch no longer treats the port as a normal port. That is, the switch does not learn MAC addresses for received frames, or send frames based on matching the MAC table, for that port.
    - A SPAN destination port can be unconfigured from one monitor session (`no monitor session # destination interface <type> <number>`) and then added to another monitor session.
    - Multiple SPAN sources can be used in a single SPAN session.
    - One SPAN session cannot mix interfaces and VLAN sources; that is, the sources must all be interfaces or all be VLANs.
    - One SPAN session can use any combination of directions (transmit, receive, and both) as applied to different SPAN sources.
    - EtherChannel interfaces can be used as source ports. Frames for all ports in the EtherCHannel will be considered by SPAN.
    - Trunks can be used as source ports. When used, by default SPAWN includes frames from all VLANs on that trunk, but SPAN VLAN filtering can limit the VLANs included.
  - Config Checklist:
    1. Use the `monitor session # source interface <type> <number> [- last-in-range] [rx | tx | both]` command in global configuration mode to identify a SPAN session by number, and define one SPAN source interface. Repeat this command to define all SPAN source ports for this session.
    2. Use the `monitor session # source vlan <vlan-id> [rx | tx | both]` command in global configuration mode to identify a SPAN session by number, and define one SPAN source VLAN. Repeat this command to define all SPAN source VLANs for this session.
    3. Use the `monitor session # destination interface <type> <number>` global command to define the one SPAN destination port for the monitor session.
- `show monitor session all`: Display the settings for SPAN sessions.
- `show monitor detail`: Display more detailed information about SPAN sessions.
***

## Cloud Computing

- Characteristics for an IT service to be considered cloud computing (derived from the definition provided by the U.S. National Institute of Standards and Technology (NIST)):
  - **On-demand self-service**: It can be requested on-demand
  - **Rapid elasticity**: It can be dynamically scaled
  - **Resource pooling**: It uses a pool of resources
  - **Broad network access**: It has a variety of network access options
  - **Measured service**: It can be measured and billed back to the user based on the amount used.
- Cisco Server Hardware Attributes:
  - **No KVM**: For most servers, there is no permanent user that sits near the server; all the users and administrators connect to the server over the network. As a result, there is no need for a permanent keyboard, video display, or mouse (collectively referred to as KVM).
  - **Racks of servers in a data center**: Most modern servers sit in a single space in racks.
- **Unified Computing System (UCS)**: Cisco's server hardware product line.
- **Virtual Machine**: A virtual instance of an OS run apart from the hardware on a machine.
- **Hypervisor**: manages and allocates the host hardware (CPU, RAM, etc.) to each VM based on the settings for each VM.
  - Each VM runs as if it is running on a self-contained physical server.
  - The hypervisor can treat each available CPU core thread as a virtual CPU (vCPU), and give each VM a number of vCPUs.
- Examples of Hypervisors:
  - VMware vCenter
  - Microsoft HyperV
  - Citrix XenServer
  - Red Hat KVM
- For networking, most VMs are assigned one or more virtual NICs and the Hypervisor uses some kind of an internal Ethernet switch concept, often called a virtual switch.
- **Top of Rack (ToR) switches**: typically two switches placed at the top of a server rack to provide redundant paths into the LAN. Each one acts as an access layer switch from a design perspective.
- **End of Row (EoR) switch**: Acts as a distribution switch, and also connects to the rest of the network.
- **Private Cloud**: creates a service, inside a company, to internal customers that meets the five cloud computing criteria.
- **Cloud Services Catalog**: a web application that lists anything that can be requested via the company's cloud infrastructure.
- **Public Cloud**: offers services, selling those services to consumers in other companies.
- **Infrastructure as a Service (IaaS)**: The user specifies the amount of hardware performance/capacity desired and the cloud service returns a VM to use.
- **Software as a service (SaaS)**: The cloud provider licenses, installs, and supports whatever software is required. The consumer chooses to use the application, signs up for the service, and starts using the application.
- **(Development) Platform as a Service (PaaS)**: a development platform, prebuilt as a service. Typically comes with many more software tools beyond what is provided by an IaaS.
- **Connecting to a Public Cloud with Internet**:
  - **Pros**:
    - **Agility**: An enterprise can get started using public cloud without having to wait to order a private WAN connection to the cloud provider because cloud providers support Internet connectivity.
    - **Migration**: An enterprise can switch its workload from one cloud provider to another more easily because cloud providers all connect to the Internet.
    - **Distributed users**: The enterprise's users are distributed and connect to the Internet with their devices.
  - **Cons**:
    - **Security**: Risk of Man-in-the-middle attacks.
    - **Capacity**: Moving an internal application to the public cloud increases network traffic.
    - **Quality of Service (QoS)**: The Internet does not provide QoS, whereas private WANs can.
    - **No WAN SLA**: ISPs typically will not provide a service level agreement (SLA) for WAN performance and availability to all destinations of a network.
- **Cloud Services Router (CSR)**: a router that runs as a VM in a cloud service, controlled by the cloud consumer. Typically used to terminate VPNs.
- **Private WAN and Internet VPN Access to Public Cloud**:
  - **Pros**:
    - Improved security.
    - Reduced Traffic compared to public cloud access via just internet.
  - **Cons**:
    - Internet VPN does not provide QoS.
    - Installing new Private WAN connections
    - Increased cost
- **Intercloud Exchanges**: a company that creates a private network as a service.
  - Connects to multiple cloud providers on one side.
  - On the other side, connects to cloud consumers.
- Summary Table:

| | Internet | Internet VPN | MPLS VPN | Ethernet WAN| Intercould Exchange |
| --- | --- | --- | --- | --- | --- |
| Secure | No | Yes | Yes | Yes | Yes |
| QoS | No | No | Yes | Yes | Yes |
| Requires capacity planning | Yes | Yes | Yes | Yes | Yes |
| Easier migration to new provider | Yes | Yes | No | No | Yes |
| Can begin using public cloud quickly | Yes | Yes | No | No | No |

- **Virtual Network Function (VNF)**: a virtual instance of a traditional networking device that the consumer can choose to use in a cloud.
***

## SDN and Network Programmability

- **Software Defined Networking**: the concept of software control of the network, rather than the more static configuration-controlled networking.
- **Data Plane**: The tasks that a networking device does to forward a message. Also known as the *forwarding plane*.
- **Control Plane**: Any action that controls the data plane. Usually creating the tables used by the data plane, (ARP tables, Routing tables, MAC tables, etc.).
- **Management Plane**: Protocols that allow network engineers to manage the devices. (Telnet, SSH, SNMP, Syslog, etc.)
- **Application-specific integrated circuit (ASIC)**: a chip built for specific purposes, such as for message processing in a networking device.
- **Ternary Content-addressable memory (TCAM)**: memory that does not require the ASIC to search the table but returns field request results to the ASIC.
- **Distributed Control Plane**: When control plane processes are distributed among multiple devices.
- **Centralized Control Plane**: Running all control plane logic on one device.
- **Controller/SDN Controller**: centralizes the control of the networking devices.
- **Southbound Interface (SBI)**: The interface between the controller and the devices it manages.
- **Northbound Interface (NBI)**: Opens the controller so its data and functions can be used by other programs, enabling network programmability.
- **REST API (Representational State Transfer)**: allows applications to sit on different hosts, using HTTP messages to transfer data over the API.
- **Open SDN**: Made by the Open Networking Foundation that acts as a consortium of users and vendors to help establish SDN in the marketplace.
  - OpenFlow acts as the SBI, with many APIs as the NBI.
- **OpenDaylight Controller**: an open-source project of the Linux foundation that has a variety of SBIs
- **Cisco Open SDN Controller**: Cisco's commercial version of ODL.
- **Application Centric Infrastructure (ACI)**: A solution to IT Infrastructure problems that begins by thinking about the applications and what they need and then building networking concepts around application architectures.
  - *Endpoints*: VMs or traditional servers with the OS running directly on the hardware.
  - *Policies*: defined about which endpoint groups can communicate with whom.
  - **Application Policy Infrastructure Controller (APIC)**: The centralized controller for ACI.
- **APIC Enterprise Module (APIC-EM)**: The Cisco SDN offering for the enterprise.
  - RESTful NBI
  - Telnet, SSH, and SNMP SBI

| Criteria | Open SDN | ACI | APIC Enterprise |
| -------- | -------- | --- | --------------- |
| Changes how the device control plane works versus traditional networking | Yes | Yes | No |
| Creates centralized point from which humans and automation control the network. | Yes | Yes | Yes |
| Degree to which the architecture centralized the control plane | Mostly | Partially | N/A |
| SBIs used | OpenFlow | OpFlex | CLI, SNMP |
| Controllers | OpenDaylight, Cisco OSC | APIC | APIC-EM |
| Organization that is the primary definer/owner | ONF | Cisco | Cisco |

- **APIC-EM Path Trace App**: A tool that predicts what happens in the data plane of the various devices in the network.
  1. Before using Path Trace, another APIC-EM app called Discovery discovers the network topology.
  2. From the Path Trace part of the GUI, the user can type in a source and destination address of a packet.
  3. The Path Trace app examines information pulled by APIC-EM from the devices in the network: The MAC tables, IP routing tables, and other forwarding details in the devices, to analyze where this imaginary packet would flow if sent in the network right now.
  4. The Path Trace GUI displays the path, with notes, overlaid on a map of the network.
- **APIC-EM Path Trace ACL Analysis Tool**: Looks for any enabled ACLs for the path trace app.
