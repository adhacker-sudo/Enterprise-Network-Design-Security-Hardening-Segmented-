 PROJECT Enterprise-Network-Design-Security-Hardening-Segmented-
 CATEGORY Networking
 project link
 https://drive.google.com/file/d/1elt9ggJUyH-8Pk-hcHs_jTZ6JjgWzjm1/view?usp=sharing
 DESCRIPTION
VLAN network built and secured in Cisco Packet Tracer: DHCP, NAT, EtherChannel, Port Security, DHCP Snooping, DAI.   
A segmented enterprise LAN built in Cisco Packet Tracer, then hardened against real Layer 2/3 attacks. The project covers the full cycle: design, configuration, verification, security hardening, and live troubleshooting.
Table of Contents
Overview
Topology
Phase 1: Core Infrastructure
Phase 2: Security Hardening
Attack Simulation and Results
Troubleshooting Case Study
Key Results
Repository Structure
Skills Demonstrated
Author
OVERVIEW
The network simulates a small company with three departments (HR, IT, Sales) spread across two access switches and connected through a single router.

Department	VLAN	Network	Gateway
HR	10	  192.168.10.0/24	192.168.10.1
IT	20	  192.168.20.0/24	192.168.20.1
Sales	30	192.168.30.0/24	192.168.30.1
Phase 1: Core Infrastructure
Feature	What it does
VLAN segmentation	Isolates HR, IT, and Sales into separate broadcast domains
Router-on-a-stick	Routes between VLANs over a single trunk link
DHCP	Per-VLAN pools assign address, gateway, and DNS automatically
EtherChannel (LACP)	Bundles two S1-S2 links for redundancy and bandwidth
NAT overload	Gives all VLANs internet access through one outside address
The work was done in two connected phases: first build a working, redundant network, then defend it against an attacker on the LAN   
TOPOLOGY
R1: router-on-a-stick with 802.1Q subinterfaces (.10, .20, .30), DHCP server, and NAT
S1 and S2: access switches linked by a two-cable EtherChannel (LACP)
Hosts: six PCs (two per VLAN), plus an attacker host for the security phase
COMMANDS
 VLANs (both switches)
vlan 10
 name HR
vlan 20
 name IT
vlan 30
 name SALES
 EtherChannel between switches
 interface range fa0/23 - 24
 switchport mode trunk
 channel-group 1 mode active
 interface port-channel 1
 switchport mode trunk
 Router-on-a-stick (R1)
 interface gig0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 DHCP pool (R1)
 ip dhcp excluded-address 192.168.10.1
 ip dhcp pool HR
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
 NAT overload (R1)
 access-list 1 permit 192.168.0.0 0.0.255.255
 ip nat inside source list 1 interface gig0/1 overload
 Phase 2: Security Hardening
Control	Threat it stops
Port Security (sticky, max 1, shutdown)	Unauthorized devices plugged into the LAN
DHCP Snooping	Rogue DHCP servers handing out fake gateways
Dynamic ARP Inspection	ARP spoofing and man-in-the-middle attacks
VLAN ACL	Unwanted traffic between departments (HR to IT blocked)
Storm Control	Broadcast floods and loop-based DoS
SSH-only management	Plain-text Telnet access to the router

COMMADS
! Port Security
interface fa0/4
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown

! VLAN ACL on R1 (HR cannot reach IT)
access-list 100 deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
access-list 100 permit ip any any
interface gig0/0.10
 ip access-group 100 in

! DHCP Snooping
ip dhcp snooping
ip dhcp snooping vlan 10,20,30
interface fa0/5
 ip dhcp snooping trust

! Dynamic ARP Inspection
ip arp inspection vlan 10,20,30
interface fa0/5
 ip arp inspection trust

! Storm Control
interface fa0/1
 storm-control broadcast level 10

! SSH-only management
hostname Router
ip domain-name lab.local
crypto key generate rsa
username admin secret <your-password>
line vty 0 4
 transport input ssh
 login local
 Attack Simulation and Results
Attack	Defense	Verification	Outcome
Unauthorized device on an access port	Port Security	show port-security	Port shut down
Rogue DHCP server	DHCP Snooping	show ip dhcp snooping	Offer dropped
ARP spoofing	Dynamic ARP Inspection	show ip arp inspection	Packet denied
HR host reaching IT VLAN	VLAN ACL	show access-lists	Traffic denied
Broadcast flood	Storm Control	show storm-control broadcast	Traffic capped
Telnet access attempt	SSH-only VTY lines	show ip ssh	Telnet refused
Troubleshooting Case Study

Symptom: PCs received APIPA addresses (169.254.x.x) instead of DHCP leases.

Investigation:

show ip dhcp binding on R1 returned no entries, so requests never reached the router.
show interfaces trunk on S1 returned nothing, so no port was trunking.
show cdp neighbors identified the exact port connected to R1 (Fa0/5).
Root cause: the router-facing switch port was still in access mode, so VLAN-tagged DHCP traffic was dropped. A Layer 2 fault that looked like a DHCP problem.

Fix: set the port to trunk mode with all VLANs allowed, and enabled PortFast on access ports. Hosts then received correct leases and inter-VLAN routing worked end to end.
KEY RESULTS
10 verification tests passed
6 security controls deployed
1 real fault diagnosed and resolved with CLI evidence
Skills Demonstrated

Networking: VLAN design, 802.1Q trunking, inter-VLAN routing, DHCP, NAT Redundancy: EtherChannel (LACP), Spanning Tree behavior, PortFast Security: Port Security, DHCP Snooping, Dynamic ARP Inspection, ACLs, SSH hardening Troubleshooting: CLI-driven fault isolation, CDP topology checks, evidence-based root cause analysis Tools: Cisco Packet Tracer, Cisco IOS CLI
