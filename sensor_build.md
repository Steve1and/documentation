# Sensor Build Notes
***

# Zeek

## Architecture

**Network** => (*Packets*) => **Event Engine** => (*Events*) => **Policy Script Interpreter** => (*Logs*) (*Notification*)

**Event Engine**: reduces the incoming packet stream into a series of higher-level *events*. These events reflect network activity in policy-neutral terms.
- For example, for each HTTP request seen on the wire, only the `http_request` event, the IP address and port, the URI being requested, and the HTTP version in use are reported.
- Contains the following sub-components:
  - **Input Sources**: ingest incoming network traffic from network interfaces.
  - **Packet Analysis**: processes lower-level protocols, starting all the way down at the link layer.
  - **Session Analysis**: handles application-layer protocols, such as HTTP, FTP, etc.
  - **Plugin-Architecture**: allows the adding of any outside capabilities to work with the core Zeek code base.

**Script Interpreter**: executes a set of *event handlers* written in Zeek's custom scripting language.
- All of Zeek's default output comes from scripts.
- Zeek's language allows scripts to maintain state over time, enabling them to track and correlate the evolution of what they observe across connection and host boundaries.
- Zeek scripts can generate real-time alerts and also execute arbitrary external programs on demand.

Without any major configuration, Zeek offers transaction data and extracted content data, in the form of logs summarizing protocols and files seen traversing the wire.
- Zeek can also provide some degree of alert data in the form of notices, and analysts can modify Zeek to create custom alerts.
***

## Installation

### Prerequisites and Dependencies

Run the following command to install the required dependencies:
- `sudo apt-get install cmake make gcc g++ flex bison libpcap-dev libssl-dev python3 python3-dev swig zlib1g-dev`

**Optional Dependencies**:
- `libmaxminddb`: Used for geolocating IP addresses. Install with: `sudo apt-get install libmaxminddb-dev`
  - Navigate to the following address and login: https://www.maxmind.com/en/accounts/594687/geoip/downloads
  - Download the latest `GeoLite2 ASN` database (`.mmdb` format).
  - Once Zeek is installed, a GeoIP subdirectoy will be created if `libmaxminddb` is available at build time.
  - Move the file to the GeoIP database directory. For Linux, it will be either `/usr/share/GeoIP` or `/var/lib/GeoIP`.
    - `mv GeoLite2-City.mmdb /usr/share/GeoIP`
    - `mv GeoLite2-City.mmdb /var/lib/GeoIP`
- `sendmail`: enables Zeek and ZeekControl to send mail
  - `sudo apt-get install sendmail`
- `curl`: used by a Zeek script that implements active HTTP
  - `sudo apt-get install curl`
- `gperftools`: tcmalloc is used to improve memory and CPU usage
  - `sudo apt-get install google-perftools`
- `jemalloc`: located at: https://github.com/jemalloc/jemalloc
  - `sudo apt-get install git autoconf`
  - `sudo git clone https://github.com/jemalloc/jemalloc.git`
  - Navigate into the source directory:
    - `cd /path/to/download/jemalloc`
    - `sudo ./autogen.sh`
    - `sudo make`
    - `sudo make install`
- `PF_RING`: allows speeding up the packet capture process by installing anew type of socket in Linux systems.
  - Supports 10Gbit hardware packet filtering using standard network adapters, and user-space DNA (Direct NIC Access) for fast packet capture/transmission.
  - Download and install on Ubuntu 20.04 LTS:
    - `apt-get install software-properties-common wget`
    - `add-apt-repository universe`
    - `wget https://packages.ntop.org/apt-stable/20.04/all/apt-ntop-stable.deb`
    - `apt install ./apt-ntop-stable.deb`
    - `apt-get clean all`
    - `apt-get update`
    - `apt-get install pfring-dkms nprobe ntopng n2disk cento`
  - You will be required to answer some install questions.
- `krb5` libraries and headers
  - `sudo apt-get install krb5*`
- `ipsumdump`: for trace-summary
  - `sudo git clone https://github.com/kohler/ipsumdump.git`
  - `cd /path/to/download/ipsumdump`
  - `sudo ./configure`
  - `sudo make`
  - `sudo make install`
- Python Packages for the `zkg` package manager:
  - `sudo apt-get install python3-git python3-semantic-version`

### Installing from Source

Download the source files and install:
- `git clone --recursive https://github.com/zeek/zeek`
- `./configure`
- `make`
- `make install`

### Configuring the Run-Time Environment

Adjust the PATH variable to include Zeek:
- `export PATH=/usr/local/zeek/bin:$PATH`
- If you installed from a binary source package, replace the directory with: `/opt/zeek/bin`
***

## Zeek Control

1. Set the interface to monitor in `/usr/local/zeek/etc/node.cfg`
  ```
  [zeek]
  type=standalone
  host=localhost
  interface=eth0
  ```
2. In `/usr/local/zeek/etc/networks.cfg`, comment out the default settings and add the networks that Zeek will consider local to the monitored environment.
3. Start ZeekControl:
  - `sudo zeekctl`
4. For first time startup, run the `install` command.
5. Start Zeek: `start`
  - The `deploy` command combines `install` and `start` into one.
  - Stop the Zeek instance with `stop`

Whenever Zeek is stopped or the `LogRotationInterval` value is reached in `/usr/local/zeek/etc/zeekctl.cfg`, the current logs are compressed and moved to `/usr/local/zeek/logs/` in a folder named after the current day and time.
- While running, logs are stored in: `/usr/local/zeek/logs/current`

To write logs in a JSON format, add the following line to `/usr/local/zeek/share/zeek/site/local.zeek`:
- `@load tuning/json-logs.zeek`
***

## Configure the VM to allow Promiscuous mode to be set on interfaces

- Put these in the ethernet section of the `.vmx` file.
ethernet0.noPromisc = "FALSE"
ethernet1.noPromisc = "FALSE"
ethernet2.noPromisc = "FALSE"
***

# ELK Setup

## Installation

1. Add the PGP key used to sign the Elastic packages:
  - `wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -`
2. Add the `apt-transport-https` package:
  - `sudo apt-get install apt-transport-https`
3. Add the elastic repository to your source list:
  - `echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list`
4. Install the service:
  - `sudo apt-get update && sudo apt-get install elasticsearch`
5. Modify the bind address to `0.0.0.0` in the `/etc/elasticsearch/elasticsearch.yml` file.
  - Also add: `discovery.type: single-node` to the file to prevent bootstrap check errors.
6. Start Elasticsearch
  - `sudo service elasticsearch start`
7. Install kibana:
  - `sudo apt-get update && sudo apt-get install kibana`
8. Modify the address in `/etc/kibana/kibana.yml` to `0.0.0.0`.
9. Start the Kibana service:
  - `sudo service kibana start`
10. Download and install filebeat:
  - `sudo apt-get update && sudo apt-get install filebeat`
11. Modify the `/etc/filebeat/filebeat.yml` file:
  ```
  output.elasticsearch:
    hosts: ["192.168.157.128"]
    username: "elastic"
    password: "<password>"
  setup.kibana:
    host: "192.168.157.128"
  ```
12. Enable and configure the zeek module:
  - `sudo filebeat modules enable zeek`
  - `sudo filebeat modules enable suricata`
13. Modify the `/etc/filebeat/modules.d/zeek.yml` file to include the `var.paths: ["/usr/local/zeek/logs/current/<logname>.log"]` entry beneath each log type you wish to input.
14. Modify the `/etc/filebeat/modules.d/suricata.yaml` file to point to the `/var/log/suricata/eve.json` file.  
15. Start the filebeat service:
  - `sudo filebeat setup`
  - `sudo service filebeat start`
***

# Suricata

## Installation and Setup

1. Install dependencies:
  - `sudo apt-get install libpcre3-dbg libpcre3-dev autoconf automake libtool libpcap-dev libnet1-dev libyaml-dev libjansson4 libcap-ng-dev libmagic-dev libjansson-dev zlib1g-dev pkg-config rustc cargo`
2. Download and Install the Suricata update tool via `pip`:
  - `apt-get install python3-pip`
  - `pip3 install --upgrade suricata-update`
  - `ln -s /usr/local/bin/suricata-update /usr/bin/suricata-update`
3. Get the latest suricata download:
  - `sudo add-apt-repository ppa:oisf/suricata-stable`
  - `sudo apt update`
  - `sudo apt install suricata jq`
4. Modify the `/etc/suricata/suricata.yaml` file to include the desired interfaces to monitor and rules to use.
  - For multiple interfaces, make a new `-interface: <interface>` line beneath the first.
5. Update the signatures:
  - `suricata-update`
6. Restart Suricata:
  - `systemctl restart suritcata`
***

# Kali to Target Networking Settings

## SSH Access

**iptables**
- Set on Sensor/Redirector
- `iptables -I FORWARD -s 192.168.157.130 -d 172.16.193.129 -p tcp --dport 22 -j ACCEPT`
- `iptables -I FORWARD -s 172.16.193.129 -d 192.168.157.130 -p tcp --sport 22 -j ACCEPT`
- For a more secure rule, use: `iptables -I FORWARD -s 172.16.193.129 -d 192.168.157.130 -p tcp --sport 22  -m state --state ESTABLISHED,RELATED -j ACCEPT`

**ip routes**
- Set on Kali and Target:
- **Kali**: `ip route add 172.16.193.0/24 via 192.168.157.128 dev eth0`
- **Targets**: `ip route add 192.168.157.0/24 via 172.16.193.128 dev eth0`
