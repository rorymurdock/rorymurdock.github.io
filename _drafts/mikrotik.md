Quick Start
# CHANGE THESE!!!!
/user set admin
password=fa6f99dae2b7284c13709aa2f0e74d223681d17da288c44a7eb762b07aaf2b87
/system identity set name=CHANGEME
# Adjust these to the right subnet or interface
/ip address add address=172.16.1.1 network=172.16.1.0 interface=ether1
/ip pool
add name=dhcp_pool1 ranges=172.16.1.100-172.16.1.254
/ip dhcp-server
add add-arp=yes address-pool=dhcp_pool1 disabled=no interface=ether1
name=LAN
/ip dhcp-server network
add address=172.16.1.0/24 dns-server=172.16.1.1 gateway=172.16.0.1
/ip firewall filter
add action=drop chain=input comment=\&quot;Drop Malicious requests from
internet\&quot; \\
dst-port=53,123,161 in-interface=ether1 protocol=udp
add action=drop chain=input comment=\&quot;Drop Malicious requests from
internet\&quot; \\
dst-port=22,53,123,161 in-interface=ether1 protocol=tcp
/ip cloud set ddns-enabled=yes
/ip dns set allow-remote-requests=yes servers=8.8.8.8,8.8.4.4
/ip service
set telnet disabled=yes
set ftp disabled=yes
set www disabled=yes
set ssh disabled=yes
set api disabled=yes
set api-ssl disabled=yes
/ip firewall nat
add action=masquerade chain=srcnat out-interface=ether1
/system clock set time-zone-name=Australia/Brisbane
/tool graphing interface add
/tool graphing queue add
/tool graphing resource add
/ip cloud print

Basics
Routerboard devices come with a default IP address of 192.168.88.1/24 and DHCP enabled on
ETH1. Connect to a ETH1 port if it\&#39;s not a WAN port otherwise connect on EHT2. You will
receive an IP and will be able to connect over Layer 3 (IP address) or Layer 2 (MAC address)
using WinBox you can configure the Mikrotik. There is also a web interface called [1]
Default Username is admin password &lt;blank&gt;
RouterBoard Naming Convention

Frequently used ports
Port Service Protocol

21 FTP TCP
22 SSH TCP
25 SMTP TCP
53 DNS TCP/UDP
80 HTTP TCP
123 NTP UDP
143 IMAP TCP
161 SNMP UDP
443 HTTPS TCP
548 AFP TCP
3283 ARD TCP/UDP
587 SMTP Submission TCP
993 IMAP SSL TCP
1701 L2TP UDP
1723 PPTP TCP
2195 APNS TCP
2196 APNS TCP
3306 MySQL TCP
3389 RDP TCP/UDP
5900 VNC TCP/UDP
8291 WinBox TCP
CHR
```NDC``` Deploy using the ROS template, add ether1 to VLAN 30, add ether2 to whichever
VLAN you want.
1. Change the ROS password
/user set admin password=SECURE

1. Add an IP address to ether1 via
/ip address add address=103.219.214.x network=103.219.214.129
interface=ether1

1. Add a default route
/ip route add distance=100 gateway=103.219.214.129
Once that\&#39;s done you can connect via winbox.
Configuring
First things to do when setting up a new MikroTik
1. Clear default config and reconnect over MAC address
2. Change admin password via System -&gt; Password
3. Set clock
4. Change the name via System -&gt; Identity

5. Configure WAN interface
● PPPoE
● Fibre
6. Add the following firewall rules - This could be either \&#39;\&#39;\&#39;PPPoE\&#39;\&#39;\&#39; interface or the eth
interface depending on the service type.
● Block all TCP 22,53,123,161 on WAN interface
● Block all UDP 53,123,161 on WAN interface
7. Allow remote DNS requests
8. Add Masq rule to allow
The below script will set time &amp; name, enable DNS server and add the firewall rules to an
interface named WAN
/system clock set time-zone-name=Australia/Brisbane
/system identity set name=EST-NDC-RT2
/ip dns set allow-remote-requests=yes servers=8.8.8.8,8.8.4.4
/ip firewall filter
add action=drop chain=input comment=\&quot;Drop Malicious requests from
internet\&quot; \\
dst-port=53,123,161 in-interface=WAN protocol=udp
add action=drop chain=input comment=\&quot;Drop Malicious requests from
internet\&quot; \\
dst-port=22,53,123,161 in-interface=WAN protocol=tcp
/ip firewall nat
add action=masquerade chain=srcnat out-interface=WAN

DynDNS
Mikrotiks have a built in dyndns service you can enable it and view the hostname via:
/ip cloud set ddns-enabled=yes
/ip cloud print

Safe Mode
When Safe Mode is enabled any changes made will be rolled back if WinBox or Console are
disconnected for 9min.
Use with caution as sometimes it does not work as it should / at all.
Tips
● You can use telnet to verify a port\&#39;s connectivity
● Under IP -&gt; Cloud there is a built in DynDNS updater that uses the router serial number +
.sn.mynetname.net
● If you hover over the letters in fields such as status in routing &amp; DHCP then a key will be
shown with what the letters mean
● D - Dynamic

● A - Active
● S - Static
● You can set a static DNS entry via IP -&gt; DNS -&gt; Static
This will create both A and PTR records
● You can torch an interface to track down what\&#39;s using bandwidth
● To limit one IPs bandwidth add a queue for it and set a DHCP reservation. This can
also be done for IP ranges
● Tools -&gt; Graphing
Enable graphing all on interfaces and queues to track bandwidth usage over time
Port Forwards
● IP -&gt; Firewall -&gt; NAT
Chain: dstnat
Add the IP or the interface for the port (e.g. 203.1.1.1 or ether1)
TCP or UPD
Destination Port (e.g. port 3389)
Action: dst-nat
To Address (e.g. 172.16.0.1)
add action=dst-nat chain=dstnat comment=LI-NDC-SQL dst-
address=210.10.238.155 dst-port=3393 protocol=tcp to-
addresses=172.16.106.10 to-ports=338

Routing
What is routing? When a packet\&#39;s destination is beyond the current network the
packet is sent to the network gateway. The gateway checks its routing tables and
forwards the packet on to the next router. This is called a hop.
Example 192.168.1.1/24 is another network connected via VPN.
To route to this network a route must be added as so
route: 192.168.1.1/24 192.168.1.254 (Gateway for that network)
Because the MikroTik has IP on that network it can communicate directly with that
router and forward the packets.
● Distance
The router checks it\&#39;s routing table in ascending order of distance. A VLAN (e.g.
172.16.0.1/24) would have a distance of 0 and a WAN a distance of 100. That way all of
the other routes will be checked first and then as a catch all all other traffic (0.0.0.0/0) will
be passed through the WAN
● WAN Failover

Quite simple really, just set the PPPoE interface to add a default gateway with a higher
distance than the primary gateway.
PPPoE
Interface -&gt; Add PPPoE Client
or
/interface pppoe-client
add add-default-route=yes default-route-distance=100
disabled=no interface=\\
ether1 max-mru=1480 max-mtu=1480 mrru=1600 name=WAN
password= \\
use-peer-dns=yes user=

EoIP
PPTP VPN
1. Create Server interface
/interface pptp-server
add name=\&quot;PPTP\&quot;

1. Create profile
/ppp profile
add change-tcp-mss=yes local-address=172.16.10.1 name=PPTP

1. Enable PPTP Server + Set profile (Make sure local address is
router IP/Remote address is VPN Pool)
/interface pptp-server server
set default-profile=PPTP enabled=yes

1. Add user
/ppp secret
add name=vpn password=&lt;password&gt; profile=PPTP remote-
address=172.16.10.201 \\
service=pptp

L2TP VPN
Interfaces -&gt; L2TP Server (button) -&gt;
1. Tick enabled

2. Select only mschap2
Go to the profiles tab and edit default-encryption
1. Set the local address to the routers LAN IP
2. Select the DHCP Pool for remote address
3. Set DNS accordingly
4. Set Change TCP MSS to yes
1. Go to the PPP -&gt; Secrets tab
1. IP -&gt; IPSec
2. New Peer
3. Set exchange mode to main-l2tp
4. Tick NAT Traversal
https://firstdigest.com/2015/01/mikrotik-l2tp-with-ipsec-for-mobile-clients/
Graphing
By enabling graphing on all interfaces you can review traffic over time.
/tool graphing interface add
/tool graphing queue add
/tool graphing resource add
File:Mikrotik Graph.png
Backup Script
Run this command via Terminal
/system scheduler
add interval=1d name=\&quot;Email Backup\&quot; on-event=backup
policy=\\
read,write,policy,test,password,sniff,sensitive start-
date=jul/20/2016 \\
start-time=21:00:00
/system script
add name=backup owner=admin policy=\\
ftp,reboot,read,write,policy,test,password,sniff,sensitive
source=\&quot;:log in\\
fo (\\\&quot;Starting Backup of \\\&quot;.[/system identity get
name])\\r\\
\
:global backupfile ([/system identity get name] . \\\&quot;-\\\&quot;
. [/system clock\\
\\_get time])\\r\\
\
/system backup save name=\\$backupfile\\r\\
\
:delay 10s\\r\\
\
:log info \\\&quot;Backup sending\\\&quot;\\r\\
\

/tool e-mail send to=\\\&quot;mikrotikbackup@estorm.com.au\\\&quot;
subject=([/system \\
identity get name] . \\\&quot; Backup \\\&quot;. \\\&quot; \\\&quot;
.[/system clock get date]) from=m\\
ikrotikbackup@estorm.com.au file=\\$backupfile
server=132.245.42.242\\r\\
\
:delay 30s\\r\\
\
/file remove \\$backupfile\\r\\
\
:log info \\\&quot;Backup completed\\\&quot;\&quot;
/tool e-mail
set address=132.245.42.242
from=mikrotikbackup@estorm.com.au password=\\
czZMNVVya3hs port=587 start-tls=yes
user=mikrotikbackup@estorm.com.au
This will
1. Create a schedule to run the job
2. Created the backup script
3. Set the email details
Wireless
Change the SSID and the wpa2-pre-shared-key
/interface wireless security-profiles
set [ find default=yes ] authentication-types=wpa2-psk eap-
methods=\&quot;\&quot; mode=\\
dynamic-keys supplicant-identity=MikroTik wpa2-pre-
shared-key=wikirule4lyfe
add authentication-types=wpa2-psk eap-methods=\&quot;\&quot; group-
ciphers=tkip,aes-ccm \\
management-protection=allowed mode=dynamic-keys
name=WPA2 \\
supplicant-identity=\&quot;\&quot; unicast-ciphers=tkip,aes-ccm
wpa2-pre-shared-key=wikirule4lyfe
/interface wireless
set [ find default-name=wlan1 ] band=2ghz-b/g/n
country=australia disabled=no \\
distance=indoors frequency-mode=regulatory-domain
mode=ap-bridge \\
security-profile=WPA2 ssid=SSID wireless-
protocol=802.11 wmm-support=\\
enabled wps-mode=disabled
Note that you will need to create a bridge between the wireless interface
and the ether switch otherwise the two will be isolated. You can do this by
/interface bridge port
add bridge=\&quot;LAN Bridge\&quot; interface=ether2
add bridge=\&quot;LAN Bridge\&quot; interface=wlan1

Assuming wlan1 is your wireless interace and ether2 is your LAN interface.
If ether2 is running as a master port all other ports that are slaves will
automatically be added to the bridge. You will need to change your DHCP
interface to LAN Bridge if you had a static bind to ether2.
Updating
Both the firmware and the system need updating.
\&#39;\&#39;\&#39;Firmware\&#39;\&#39;\&#39;
You can update the firmware via System -&gt; Routerboard.
\&#39;\&#39;\&#39;System\&#39;\&#39;\&#39;
You can update the system via QuickSet -&gt; Check for Updates.
Security
WinBox Layer 2 access should be disabled on WAN interfaces via Tools -&gt;
MAC
A strong admin password should be set
The following Firewall rules are recommended
Add the following firewall rules
1. Block all TCP 22,53,123,161 on WAN interface
2. Block all UDP 53,123,161 on WAN interface
Ports 53 UDP &amp; TCP are used for DNS and blocking them will prevent DNS
amplification attacks in which DNS queries are spoofed to appear as though
they are coming from the target. The router will respond to the query and
send the results to the target. When enough packets are aimed at the target
the incoming traffic will exceed their connection ability resulting in service
interruption.
Port
● 22 - SSH - Don\&#39;t want someone to try and brute force the router /
exploit an vulnerability
● 123 - NTP - No need to have open
● 161 - SNMP - No need to share our traffic stats with the world
See getting started for script.
Script to disable services:
/ip service
set telnet disabled=yes
set ftp disabled=yes
set www disabled=yes
set ssh address=192.168.111.0/24
set api disabled=yes
set api-ssl disabled=yes

Firewall

There are three chains in Mikrotik Firewalls
1. \&#39;\&#39;\&#39;Input\&#39;\&#39;\&#39; - used to process packets entering the router through
one of the interfaces with the destination IP address which is one of
the router\&#39;s addresses. Packets passing through the router are not
processed against the rules of the input chain
2. \&#39;\&#39;\&#39;Forward\&#39;\&#39;\&#39; - used to process packets passing through the router
3. \&#39;\&#39;\&#39;Output\&#39;\&#39;\&#39; - used to process packets originated from the router
and leaving it through one of the interfaces. Packets passing
through the router are not processed against the rules of the output
chain
ToDo:
Security http://wiki.mikrotik.com/wiki/Manual:IP/Firewall/Filter#Router_protec
tion
DHCP
To setup DHCP you must first give the interface an IP in form of X.X.X.X/Y.
The subnet mask is important as otherwise when running the wizard the
range would be x.x.x.x to x.x.x.0.
Then go to IP -&gt; DHCP -&gt; DHCP Setup -&gt; Complete the wizard.
Interface IPs
You can add via IP -&gt; Address -&gt; Plus button
or
/ip address add address=103.219.214.x
network=103.219.214.129

VLAN
VLANs on a mikrotik are easy once you get your head around it.
1. Make a bridge - VLAN Bridge (Bridge -&gt; New)
2. Add interfaces to the bridge (Bridge -&gt; Ports)
3. Add VLAN to bridge (Interfaces -&gt; VLAN). Naming scheme is VLAN
ID_CLIENT CODE-SITE e.g. 100_EST-NDC
4. Add IP address to VLAN interface (172.16.VLAN ID.1)
5. Run the DHCP setup (IP -&gt; DHCP Server -&gt; Setup)
The below script would add a VLAN Bridge, Add eth2 to it, create the VLAN
interface, add an IP &amp; DHCP scope for it.
/interface bridge add name=\&quot;VLAN Bridge\&quot;
/interface bridge port add bridge=\&quot;VLAN Bridge\&quot;
interface=ether2 path-cost=0
/interface vlan add interface=\&quot;VLAN Bridge\&quot; l2mtu=1576
name=100_EST-NDC vlan-id=100
/ip address add address=172.16.100.1/24 interface=100_EST-
NDC network=172.16.100.0

/ip pool add name=dhcp_pool1 ranges=172.16.100.2-
172.16.100.254
/ip dhcp-server network add address=172.16.100.0/24 dns-
server=172.16.100.1 gateway=172.16.100.1

SNMP
SNMP can be enabled to allow logging of interfaces in services such as
MRTG or PRTG. To do so go IP -&gt; SNMP
1. Enable the service
2. Set the location
3. Set community to Public
4. Set version to V1
/snmp set enabled=yes location=NextDC
/snmp community set [ find default=yes ]
addresses=0.0.0.0/0 authentication-password=\&quot;\&quot;\\
authentication-protocol=MD5 encryption-password=\&quot;\&quot;
encryption-protocol=DES \\
name=public read-access=yes security=none write-access=no
/snmp set trap-version=1

1. Allow the NMS ONLY to access SNMP trap via a firewall rule that
allows 161 only for that IP. Note this assumes you have
the recommended security rules enabled.
/ip firewall filter add chain=input comment=\&quot;Allow NMS
SNMP\&quot; dst-port=161 in-interface=\\
AAPT_200_200 protocol=udp src-address=54.79.3.118

Quick Start
# CHANGE THESE!!!!
/user set admin
password=fa6f99dae2b7284c13709aa2f0e74d223681d17da288c44a7e
b762b07aaf2b87
/system identity set name=CHANGEME
# Adjust these to the right subnet or interface
/ip address add address=172.16.1.1 network=172.16.1.0
interface=ether1
/ip pool
add name=dhcp_pool1 ranges=172.16.1.100-172.16.1.254
/ip dhcp-server
add add-arp=yes address-pool=dhcp_pool1 disabled=no
interface=ether1 name=LAN
/ip dhcp-server network
add address=172.16.1.0/24 dns-server=172.16.1.1
gateway=172.16.0.1
/ip firewall filter

add action=drop chain=input comment=\&quot;Drop Malicious
requests from internet\&quot; \\
dst-port=53,123,161 in-interface=ether1 protocol=udp
add action=drop chain=input comment=\&quot;Drop Malicious
requests from internet\&quot; \\
dst-port=22,53,123,161 in-interface=ether1 protocol=tcp
/ip cloud set ddns-enabled=yes
/ip dns set allow-remote-requests=yes
servers=8.8.8.8,8.8.4.4
/ip service
set telnet disabled=yes
set ftp disabled=yes
set www disabled=yes
set ssh disabled=yes
set api disabled=yes
set api-ssl disabled=yes
/ip firewall nat
add action=masquerade chain=srcnat out-interface=ether1
/system clock set time-zone-name=Australia/Brisbane
/tool graphing interface add
/tool graphing queue add
/tool graphing resource add
/ip cloud print

Basics
Routerboard devices come with a default IP address of 192.168.88.1/24 and
DHCP enabled on ETH1. Connect to a ETH1 port if it\&#39;s not a WAN port
otherwise connect on EHT2. You will receive an IP and will be able to
connect over Layer 3 (IP address) or Layer 2 (MAC address)
using WinBox you can configure the Mikrotik. There is also a web interface
called [2]
Default Username is admin password &lt;blank&gt;
RouterBoard Naming Convention

Frequently used ports
Port Service Protocol
21 FTP TCP
22 SSH TCP
25 SMTP TCP
53 DNS TCP/UDP
80 HTTP TCP
123 NTP UDP
143 IMAP TCP
161 SNMP UDP
443 HTTPS TCP
548 AFP TCP
3283 ARD TCP/UDP
587 SMTP Submission TCP
993 IMAP SSL TCP

1701 L2TP UDP
1723 PPTP TCP
2195 APNS TCP
2196 APNS TCP
3306 MySQL TCP
3389 RDP TCP/UDP
5900 VNC TCP/UDP
8291 WinBox TCP
CHR
```NDC``` Deploy using the ROS template, add ether1 to VLAN 30, add
ether2 to whichever VLAN you want.
1. Change the ROS password
/user set admin password=SECURE

1. Add an IP address to ether1 via
/ip address add address=103.219.214.x
network=103.219.214.129 interface=ether1

1. Add a default route
/ip route add distance=100 gateway=103.219.214.129
Once that\&#39;s done you can connect via winbox.
Configuring
First things to do when setting up a new MikroTik
1. Clear default config and reconnect over MAC address
2. Change admin password via System -&gt; Password
3. Set clock
4. Change the name via System -&gt; Identity
5. Configure WAN interface
● PPPoE
● Fibre
6. Add the following firewall rules - This could be either \&#39;\&#39;\&#39;PPPoE\&#39;\&#39;\&#39;
interface or the eth interface depending on the service type.
● Block all TCP 22,53,123,161 on WAN interface
● Block all UDP 53,123,161 on WAN interface
7. Allow remote DNS requests
8. Add Masq rule to allow
The below script will set time &amp; name, enable DNS server and add the
firewall rules to an interface named WAN

/system clock set time-zone-name=Australia/Brisbane
/system identity set name=EST-NDC-RT2
/ip dns set allow-remote-requests=yes
servers=8.8.8.8,8.8.4.4
/ip firewall filter
add action=drop chain=input comment=\&quot;Drop Malicious
requests from internet\&quot; \\
dst-port=53,123,161 in-interface=WAN protocol=udp
add action=drop chain=input comment=\&quot;Drop Malicious
requests from internet\&quot; \\
dst-port=22,53,123,161 in-interface=WAN protocol=tcp
/ip firewall nat
add action=masquerade chain=srcnat out-interface=WAN

DynDNS
Mikrotiks have a built in dyndns service you can enable it and view the
hostname via:
/ip cloud set ddns-enabled=yes
/ip cloud print

Safe Mode
When Safe Mode is enabled any changes made will be rolled back if
WinBox or Console are disconnected for 9min.
Use with caution as sometimes it does not work as it should / at all.
Tips
● You can use telnet to verify a port\&#39;s connectivity
● Under IP -&gt; Cloud there is a built in DynDNS updater that uses the
router serial number + .sn.mynetname.net
● If you hover over the letters in fields such as status in routing &amp; DHCP
then a key will be shown with what the letters mean

● D - Dynamic
● A - Active
● S - Static

● You can set a static DNS entry via IP -&gt; DNS -&gt; Static

This will create both A and PTR records

● You can torch an interface to track down what\&#39;s using
bandwidth

● To limit one IPs bandwidth add a queue for it and set a DHCP
reservation. This can also be done for IP ranges
● Tools -&gt; Graphing

Enable graphing all on interfaces and queues to track bandwidth usage over time

Port Forwards
● IP -&gt; Firewall -&gt; NAT
Chain: dstnat
Add the IP or the interface for the port (e.g. 203.1.1.1 or ether1)
TCP or UPD
Destination Port (e.g. port 3389)
Action: dst-nat
To Address (e.g. 172.16.0.1)
add action=dst-nat chain=dstnat comment=LI-NDC-
SQL dst-address=210.10.238.155 dst-port=3393
protocol=tcp to-addresses=172.16.106.10 to-
ports=338

Routing
What is routing? When a packet\&#39;s destination is beyond the
current network the packet is sent to the network gateway. The
gateway checks its routing tables and forwards the packet on to
the next router. This is called a hop.
Example 192.168.1.1/24 is another network connected via
VPN.
To route to this network a route must be added as so
route: 192.168.1.1/24 192.168.1.254 (Gateway for that network)
Because the MikroTik has IP on that network it can
communicate directly with that router and forward the packets.
● Distance

The router checks it\&#39;s routing table in ascending order of distance. A VLAN (e.g.
172.16.0.1/24) would have a distance of 0 and a WAN a distance of 100. That way all of
the other routes will be checked first and then as a catch all all other traffic (0.0.0.0/0) will
be passed through the WAN

● WAN Failover

Quite simple really, just set the PPPoE interface to add a default gateway with a higher
distance than the primary gateway.

PPPoE
Interface -&gt; Add PPPoE Client

or
/interface pppoe-client
add add-default-route=yes default-route-
distance=100 disabled=no interface=\\
ether1 max-mru=1480 max-mtu=1480
mrru=1600 name=WAN password= \\
use-peer-dns=yes user=

EoIP
PPTP VPN
1. Create Server interface
/interface pptp-server
add name=\&quot;PPTP\&quot;

1. Create profile - (Make sure local address is
router IP/Remote address is VPN Pool)
/ppp profile
add change-tcp-mss=yes local-
address=172.16.10.1 name=PPTP

1. Enable PPTP Server + Set profile
/interface pptp-server server
set default-profile=PPTP enabled=yes

1. Add user
/ppp secret
add name=vpn password=&lt;password&gt;
profile=PPTP remote-address=172.16.10.201
\\
service=pptp

L2TP VPN
Interfaces -&gt; L2TP Server (button) -&gt;
1. Tick enabled
2. Select only mschap2
Go tot he profiles tab and edit default-encryption
1. Set the local address to the routers LAN IP
2. Select the DHCP Pool for remote address

3. Set DNS accordingly
4. Set Change TCP MSS to yes
1. Go to the PPP -&gt; Secrets tab
1. IP -&gt; IPSec
2. New Peer
3. Set exchange mode to main-l2tp
4. Tick NAT Traversal
https://firstdigest.com/2015/01/mikrotik-l2tp-with-ipsec-
for-mobile-clients/
Graphing
By enabling graphing on all interfaces you can review
traffic over time.
/tool graphing interface add
/tool graphing queue add
/tool graphing resource add
File:Mikrotik Graph.png
Backup Script
Run this command via Terminal
/system scheduler
add interval=1d name=\&quot;Email Backup\&quot; on-
event=backup policy=\\
read,write,policy,test,password,sniff,sensi
tive start-date=jul/20/2016 \\
start-time=21:00:00
/system script
add name=backup owner=admin policy=\\
ftp,reboot,read,write,policy,test,password,
sniff,sensitive source=\&quot;:log in\\
fo (\\\&quot;Starting Backup of
\\\&quot;.[/system identity get name])\\r\\
\
:global backupfile ([/system identity get
name] . \\\&quot;-\\\&quot; . [/system clock\\
\\_get time])\\r\\
\
/system backup save name=\\$backupfile\\r\\
\
:delay 10s\\r\\
\
:log info \\\&quot;Backup sending\\\&quot;\\r\\
\
/tool e-mail send
to=\\\&quot;mikrotikbackup@estorm.com.au\\\&quot;
subject=([/system \\

identity get name] . \\\&quot; Backup \\\&quot;.
\\\&quot; \\\&quot; .[/system clock get date])
from=m\\
ikrotikbackup@estorm.com.au
file=\\$backupfile
server=132.245.42.242\\r\\
\
:delay 30s\\r\\
\
/file remove \\$backupfile\\r\\
\
:log info \\\&quot;Backup completed\\\&quot;\&quot;
/tool e-mail
set address=132.245.42.242
from=mikrotikbackup@estorm.com.au
password=\\
czZMNVVya3hs port=587 start-tls=yes
user=mikrotikbackup@estorm.com.au
This will
1. Create a schedule to run the job
2. Created the backup script
3. Set the email details
Wireless
Change the SSID and the wpa2-pre-shared-key
/interface wireless security-profiles
set [ find default=yes ] authentication-
types=wpa2-psk eap-methods=\&quot;\&quot; mode=\\
dynamic-keys supplicant-
identity=MikroTik wpa2-pre-shared-
key=wikirule4lyfe
add authentication-types=wpa2-psk eap-
methods=\&quot;\&quot; group-ciphers=tkip,aes-ccm \\
management-protection=allowed
mode=dynamic-keys name=WPA2 \\
supplicant-identity=\&quot;\&quot; unicast-
ciphers=tkip,aes-ccm wpa2-pre-shared-
key=wikirule4lyfe
/interface wireless
set [ find default-name=wlan1 ] band=2ghz-
b/g/n country=australia disabled=no \\
distance=indoors frequency-
mode=regulatory-domain mode=ap-bridge \\
security-profile=WPA2 ssid=SSID
wireless-protocol=802.11 wmm-support=\\
enabled wps-mode=disabled
Note that you will need to create a bridge between the
wireless interface and the ether switch otherwise the
two will be isolated. You can do this by
/interface bridge port

add bridge=\&quot;LAN Bridge\&quot; interface=ether2
add bridge=\&quot;LAN Bridge\&quot; interface=wlan1
Assuming wlan1 is your wireless interace and ether2 is
your LAN interface. If ether2 is running as a master port
all other ports that are slaves will automatically be
added to the bridge. You will need to change your
DHCP interface to LAN Bridge if you had a static bind
to ether2.
Updating
Both the firmware and the system need updating.
\&#39;\&#39;\&#39;Firmware\&#39;\&#39;\&#39;
You can update the firmware via System -&gt;
Routerboard.
\&#39;\&#39;\&#39;System\&#39;\&#39;\&#39;
You can update the system via QuickSet -&gt; Check for
Updates.
Security
WinBox Layer 2 access should be disabled on WAN
interfaces via Tools -&gt; MAC
A strong admin password should be set
The following Firewall rules are recommended
Add the following firewall rules
1. Block all TCP 22,53,123,161 on WAN interface
2. Block all UDP 53,123,161 on WAN interface
Ports 53 UDP &amp; TCP are used for DNS and blocking
them will prevent DNS amplification attacks in which
DNS queries are spoofed to appear as though they are
coming from the target. The router will respond to the
query and send the results to the target. When enough
packets are aimed at the target the incoming traffic will
exceed their connection ability resulting in service
interruption.
Port
● 22 - SSH - Don\&#39;t want someone to try and brute
force the router / exploit an vulnerability
● 123 - NTP - No need to have open
● 161 - SNMP - No need to share our traffic stats
with the world
See getting started for script.
Script to disable services:

/ip service
set telnet disabled=yes
set ftp disabled=yes
set www disabled=yes
set ssh address=192.168.111.0/24
set api disabled=yes
set api-ssl disabled=yes

Firewall
There are three chains in Mikrotik Firewalls
1. \&#39;\&#39;\&#39;Input\&#39;\&#39;\&#39; - used to process packets entering
the router through one of the interfaces with
the destination IP address which is one of the
router\&#39;s addresses. Packets passing through
the router are not processed against the rules
of the input chain
2. \&#39;\&#39;\&#39;Forward\&#39;\&#39;\&#39; - used to process packets
passing through the router
3. \&#39;\&#39;\&#39;Output\&#39;\&#39;\&#39; - used to process packets
originated from the router and leaving it
through one of the interfaces. Packets passing
through the router are not processed against
the rules of the output chain
ToDo:
Security http://wiki.mikrotik.com/wiki/Manual:IP/Firewall
/Filter#Router_protection
DHCP
To setup DHCP you must first give the interface an IP
in form of X.X.X.X/Y. The subnet mask is important as
otherwise when running the wizard the range would be
x.x.x.x to x.x.x.0.
Then go to IP -&gt; DHCP -&gt; DHCP Setup -&gt; Complete
the wizard.
Interface IPs
You can add via IP -&gt; Address -&gt; Plus button
or
/ip address add address=103.219.214.x
network=103.219.214.129

VLAN
VLANs on a mikrotik are easy once you get your head
around it.

1. Make a bridge - VLAN Bridge (Bridge -&gt; New)
2. Add interfaces to the bridge (Bridge -&gt; Ports)
3. Add VLAN to bridge (Interfaces -&gt; VLAN).
Naming scheme is VLAN ID_CLIENT CODE-
SITE e.g. 100_EST-NDC
4. Add IP address to VLAN interface
(172.16.VLAN ID.1)
5. Run the DHCP setup (IP -&gt; DHCP Server -&gt;
Setup)
The below script would add a VLAN Bridge, Add eth2
to it, create the VLAN interface, add an IP &amp; DHCP
scope for it.
/interface bridge add name=\&quot;VLAN Bridge\&quot;
/interface bridge port add bridge=\&quot;VLAN
Bridge\&quot; interface=ether2 path-cost=0
/interface vlan add interface=\&quot;VLAN
Bridge\&quot; l2mtu=1576 name=100_EST-NDC vlan-
id=100
/ip address add address=172.16.100.1/24
interface=100_EST-NDC network=172.16.100.0
/ip pool add name=dhcp_pool1
ranges=172.16.100.2-172.16.100.254
/ip dhcp-server network add
address=172.16.100.0/24 dns-
server=172.16.100.1 gateway=172.16.100.1

SNMP
SNMP can be enabled to allow logging of interfaces in
services such as MRTG or PRTG. To do so go IP -&gt;
SNMP
1. Enable the service
2. Set the location
3. Set community to Public
4. Set version to V1
/snmp set enabled=yes location=NextDC
/snmp community set [ find default=yes ]
addresses=0.0.0.0/0 authentication-
password=\&quot;\&quot;\\
authentication-protocol=MD5 encryption-
password=\&quot;\&quot; encryption-protocol=DES \\
name=public read-access=yes security=none
write-access=no
/snmp set trap-version=1

1. Allow the NMS ONLY to access SNMP trap via
a firewall rule that allows 161 only for that IP.
Note this assumes you have the recommended
security rules enabled.

/ip firewall filter add chain=input
comment=\&quot;Allow NMS SNMP\&quot; dst-port=161 in-
interface=\\
AAPT_200_200 protocol=udp src-
address=54.79.3.118