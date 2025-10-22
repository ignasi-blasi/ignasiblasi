# CCNA

## Basic
This set of commands establishes the fundamental security and administrative settings for a Cisco device. This includes:
- Security hardening by setting encrypted passwords (enable secret), encrypting all simple passwords in the config (service password-encryption), and restricting access via a login banner (banner motd).
- Access control for both local physical connections (console line) and remote connections (VTY lines) by applying passwords.
Preventing common operational slowdowns by disabling automatic DNS lookups for mistyped commands (no ip domain-lookup).

```
R1(config)# no ip domain-lookup
R1(config)# enable secret class
R1(config)# service password-encryption
R1(config)# banner motd $Access restricted$
R1(config)# clock set 12:58:20 JAN 2022
---
R1(config)# line vty 0 9999
R1(config-line)# password cisco
R1(config-line)# login
---
R1(config)# line console 0
R1(config-line)# password cisco
R1(config-line)# login
```

## Basic device configuration
##### SSH
Secure Shell (SSH) is the secure, encrypted replacement for Telnet for remote administrative access. These commands are necessary to enable it:

- SSH requires a Fully Qualified Domain Name (FQDN), so a local domain name must be defined (ip domain-name).
- As an encrypted protocol, SSH relies on asymmetric cryptography (RSA keys), which must be generated (crypto key generate rsa).
- Access requires a local username and password for authentication (user admin secret ccna), enforced by setting VTY lines to use local login (login local) and only permit SSH traffic (transport input ssh).

```
R1(config)# ip domain-name cisco.com
R1(config)# crypto key generate rsa
R1(config)# user admin secret ccna
R1(config)# line vty 0 15
R1(config-line)# transport input ssh
R1(config-line)# login local
```

## InterVLAN routing
InterVLAN routing is the process of forwarding network traffic between different VLANs. The Router-on-a-Stick method uses a single physical router interface connected to a switch trunk port.

- The router interface is divided into logical subinterfaces (e.g., g0/0.10), with each subinterface acting as the default gateway for a unique VLAN.

- 802.1Q encapsulation (encapsulation dot1Q) is configured on each subinterface to enable the router to process frames tagged for specific VLAN IDs, allowing it to correctly route traffic between the separate broadcast domains.

##### Subinterfaces
```
R1(config)# int g0/0.10
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# encapsulation dot1Q 11 native
R1(config-subif)# ip address 172.17.10.1 255.255.255.0
```

## VLANs
Virtual Local Area Networks (VLANs) are a Layer 2 method of segmenting a physical network into multiple logical broadcast domains, improving security and reducing broadcast traffic.

- VLAN creation and naming logically separates network devices (vlan 100).

- Trunk ports (switchport mode trunk) carry traffic for multiple VLANs (using 802.1Q tags) and are used to connect switches or routers. Configuration includes defining which VLANs are allowed (switchport trunk allowed vlan) and which VLAN's traffic is sent untagged (the native VLAN).

- Access ports (switchport mode access) carry traffic for only one single VLAN and are used for end-device connectivity.

- The VLAN database is stored in the vlan.dat file on the switch, which can be deleted to remove all configured VLANs.

##### VLANs and switchport basics
```
S1(config)# vlan 100
S1(config-vlan)# name Operations
---
S1(config-if)# switchport mode trunk
S1(config-if)# switchport trunk native vlan 1000
S1(config-if)# switchport trunk allowed vlan 100,200,1000
S1(config-if)# switchport mode access
S1(config-if)# switchport access vlan 10
---
S1# delete vlan.dat
```
##### DTP

DTP is a Cisco proprietary protocol that dynamically manages trunk formation between two switches.

- It operates in various modes (auto, desirable, trunk, access) to either actively seek a trunk or passively respond to a trunk request.

- In production networks, DTP is often explicitly disabled (switchport nonegotiate) and trunking is statically configured (switchport mode trunk) for better security and predictability, preventing unintended trunk formation.

|                   | Dynamic auto  | Dynamic desirable | Trunk   | Access
|-------------------|---------------|-------------------|---------|--------
| Dynamic Auto      | Access        | Trunk             | Trunk   | Access
| Dynamic desirable | Trunk         | Trunk             | Trunk   | Access
| Trunk             | Trunk         | Trunk             | Trunk   | Limited
| Access            | Access        | Access            | Limited | Access

```
S1(config-if)# switchport mode dynamic auto
S1(config-if)# switchport mode dynamic desirable
S1(config-if)# switchport mode trunk
S1(config-if)# switchport mode access
! DTP can be disable with nonegotiate
S1(config-if)# switchport nonegotiate
```
##### Shows
```
S1# show interfaces trunk
S1# show vlan
S1# show dtp interface fa0/1
```

## EtherChannel
EtherChannel (or link aggregation) is a technology that bundles multiple parallel physical links (up to 8) between switches or routers into a single logical channel.

- This provides increased bandwidth and link redundancy. The bundle is treated as one link by Spanning Tree Protocol (STP).

- Channel formation is managed using negotiation protocols: PAgP (Cisco proprietary) or LACP (IEEE 802.3ad standard).

  - PAgP modes (auto for passive, desirable for active) initiate or respond to negotiation.

  - LACP modes (passive for passive, active for active) initiate or respond to negotiation.

- The physical interfaces are shut down, grouped into a logical channel, and then brought back up to ensure a clean configuration process.

##### Modes
- **active** -> Enable LACP unconditionally
- **auto** -> Enable PAgP only if a PAgP device is detected
- **desirable** -> Enable PAgP unconditionally
- **on** -> Enable Etherchannel without any negotiation protocol
- **passive** -> Enable LACP only if a LACP device is detected

|           | active  | passive | desirable | auto
|-----------|---------|---------|-----------|------
| active    | LACP    | LACP    | -         | -
| passive   | LACP    | -       | -         | -
| desirable | -       | -       | PAgP      | PAgP
| auto      | -       | -       | PAgP      | -

##### PAgP
```
S1(config)# interface range f0/21 – 22
S1(config-if-range)# shutdown
S1(config-if-range)# channel-group 1 mode desirable
S1(config-if-range)# no shutdown
```

##### LACP
```
S1(config)# interface range g0/1 - 2
S1(config-if-range)# shutdown
S1(config-if-range)# channel-group 2 mode active
S1(config-if-range)# no shutdown
```


## DHCPv4
DHCP automates the process of assigning IP addresses and network configuration parameters (subnet mask, default gateway, DNS server) to client devices, simplifying network administration.

- A DHCP Server is configured with an address pool (ip dhcp pool), defining the range of available addresses, the network, the default router, and DNS information. Excluded addresses are reserved for static assignment.

- A DHCP Relay Agent (ip helper-address) is used when the DHCP server is on a different subnet from the clients. It forwards the client's DHCP broadcast requests as a unicast packet to the server's remote IP address.

- A device configured as a DHCP Client simply requests an IP address from a server (ip address dhcp).

##### DHCP server
```
R1(config)# ip dhcp excluded-address 192.168.10.1 192.168.10.10
R1(config)# ip dhcp pool R1-LAN
R1(dhcp-config)# network 192.168.10.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.10.1
R1(dhcp-config)# dns-server 192.168.20.254
```
##### DHCP relay
```
R1(config)# interface g0/0
R1(config-if)# ip helper-address 10.1.1.2
```
##### DHCP Client
```
R1(config)# interface g0/1
R1(config-if)# ip address dhcp
```
##### Shows
```
R1# show ip dhcp binding
R1# show ip dhcp pool
R1# show ip dhcp server statistics
```
## FHRP (First Hop Reduncy Protocol)
First Hop Redundancy Protocols (FHRPs), like the Cisco proprietary Hot Standby Router Protocol (HSRP), provide default gateway redundancy for end-user devices.

- Two or more physical routers share a single Virtual IP Address (VIP) and Virtual MAC Address. Client devices use the VIP as their default gateway.

- Routers negotiate Active (the forwarding router) and Standby (the listening/backup router) roles based on the configured priority (highest is Active).

- Preemption (standby 1 preempt) ensures that the highest-priority router automatically re-takes the Active role if it recovers from a failure, maintaining consistent network performance.

##### HSRP (Hot Standby Router Protocol)

```
R1(config-if)# standby version 2
R1(config-if)# standby 1 ip 192.168.1.254
! Default priority is 100, higher -> active Router
R1(config-if)# standby 1 priority 150
! Preempt makes this router active after recovery
R1(config-if)# standby 1 preempt
---
R1# show standby
```

## Switch security
Port security is a Layer 2 feature that limits which devices (based on MAC address) are allowed to connect to a specific switch port, enhancing network access security.

- It restricts the maximum number of MAC addresses that can be learned on a port (maximum 1).

- The allowed MAC addresses can be statically configured or dynamically learned and saved to the configuration (sticky).

- A violation mode (restrict, shutdown, or protect) defines the action taken when an unauthorized MAC address attempts to access the port, ranging from dropping traffic to administratively disabling the port.

##### Port-security
```
S1(config-if)# switchport port-security
S1(config-if)# switchport port-security maximum 1
S1(config-if)# switchport port-security mac-address sticky
S1(config-if)# switchport port-security violation restrict
```
##### Shows
```
S1# show port-security
S1# show port-security address
S1# show port-security interface f0/2
```

## Single-area OSPF
OSPF is an open standard link-state routing protocol that uses the Shortest Path First (SPF) algorithm to calculate the best paths through a network.

- It operates by establishing adjacencies (neighbor relationships) and exchanging link-state advertisements (LSAs) to build a synchronized topology database.

- The OSPF process is identified by a Process ID and a unique Router ID (RID).

- Interfaces are included in the OSPF domain (Area 0 in a single-area setup) using the network command or the interface-specific ip ospf command.

- Interface parameters like priority (for DR/BDR election) and hello/dead intervals (for neighbor maintenance) are crucial for proper operation. The bandwidth must be set correctly as it is used to calculate the path cost (metric).

##### Global
```
! Where 10 is the ospf process-id
R1(config)# router ospf 10
R1(config-router)# router-id 1.1.1.1
R1(config-router)# network 192.168.10.0 0.0.0.255 area 0
R1(config-router)# default-information originate
R1# clear ip ospf process
```
##### Interface
```
R1(config)# interface s0/0/0
R1(config-if)# ip ospf 10 area 0
R1(config-if)# ip ospf priority 200
R1(config-if)# ip ospf hello-interval 15
R1(config-if)# ip ospf dead-interval 60
R1(config-if)# bandwidth 64 !in kbps
```
##### Shows
```
R1# show ip ospf
R1# show ip ospf neighbor
R1# show ip ospf interface
R1# show ip route ospf
R1# show ip protocols
```

## ACLs for IPv4
ACLs are **sequential lists of permit/deny rules** used to filter or classify traffic based on defined criteria. They are fundamental for security and traffic management.

- Standard ACLs (1-99 or named) filter traffic based only on the source IPv4 address. They should be placed closest to the destination.

- Extended ACLs (100-199 or named) filter traffic based on source/destination IP address, protocol, and port number. They should be placed closest to the source.

- All ACLs have an implicit deny any at the end.

- ACLs are applied to an interface, specifying the direction of traffic flow: in (inbound, before routing) or out (outbound, after routing).

#### Standard
##### Create ACLs rules
```
R1(config)# access-list 1 remark Allow R1 LANs Access
R1(config)# access-list 1 permit 192.168.10.0 0.0.0.255
R1(config)# access-list 1 permit 192.168.20.0 0.0.0.255
R1(config)# access-list 1 deny any
```
##### Modify ACLs rules
```
R1#(config)# ip access-list standard BRANCH-OFFICE-POLICY
R1(config-std-nacl)# 30 permit 209.165.200.224 0.0.0.31
R1(config-std-nacl)# 40 deny any
```

#### Extended
##### Create ACLs rules
```
! Option 1
R1(config)# access-list 89 permit tcp 172.2.34.4 0.0.0.31 host 10.22.4.62 eq ftp
! Option 2
R1(config)# ip access-list extended HTTP_ONLY
R1(config-ext-nacl)# permit tcp 172.22.34.96 0.0.0.15 host 172.22.34.62 eq www
R1(config-ext-nacl)# permit icmp 172.22.34.96 0.0.0.15 host 172.22.34.62
```
##### Modify ACLs rules
```
R1(config)# ip access-list extended HTTP_ONLY
R1(config-ext-nacl)# 25 permit tcp 172.22.34.96 0.0.0.15 host 172.22.34.62 eq 53
R1(config-ext-nacl)# 4 permit icmp 172.22.34.96 0.0.0.15 host 172.22.34.62
```
#### Apply ACL to an interface
```
R1(config)# interface g0/0/0
R1(config-if)# ip access-group 1 out
R1(config-if)# ip access-group BRANCH-OFFICE-POLICY in
```
#### Shows
```
R1# show access-lists
R1# show access-lists 34
R1# show access-lists LAN-ACL
R1# show running-config | begin access-list
R1# show ip access-lists
!To see which ACL is applied to the interface
R1# show ip interface g0/0/0

```

## NAT
NAT is a service that translates private (non-routable) IP addresses into public (routable) IP addresses, enabling devices on a private network (like a LAN) to access the internet.

- Inside interfaces connect to the private network; outside interfaces connect to the public network.

- Static NAT (ip nat inside source static) is a one-to-one, permanent mapping of a private IP to a public IP, typically used for servers.

- Port Address Translation (PAT), also known as NAT Overload (ip nat inside source list... overload), is the most common type. It maps many private IPs to a single public IP address (or a small pool) by using unique source port numbers to differentiate sessions. This conserves public IP addresses.

##### Static
```
R1(config)# ip nat inside source static 172.16.16.1 64.100.50.1
R1(config)# interface g0/0
R1(config-if)# ip nat inside
R1(config)#interface s0/0/0
R1(config-if)#ip nat outside
```

##### Port Address Translation (PAT) / NAT overload
```
! Create ACL for insde IPs
R1(config)# access-list 1 permit 172.16.0.0 0.0.255.255

! Option 1 - Create IP pool for outside IPs
R1(config)# ip nat pool NAME 209.15.20.233 209.15.20.234 netmask 255.255.255.252
R1(config)# ip nat inside source list 1 pool NAME overload

! Option 2 - Set out interface instead of a pool
R1(config)# ip nat inside source list 2 interface s0/1/0 overload

```
##### NAT interfaces
```
R1(config)# interface s0/1/0
R1(config-if)# ip nat outside
R1(config-if)# interface g0/0/0
R1(config-if)# ip nat inside
```
##### Shows
```
R1# show run | include nat
R1# show ip nat translations
R1# show ip nat statistics
```
## Network Management
These protocols and commands are used for essential network monitoring, troubleshooting, and maintenance tasks.

- Cisco Discovery Protocol (CDP): A Cisco proprietary Layer 2 protocol that automatically discovers information about directly connected Cisco neighbor devices (model, capabilities, port ID).

- Link Layer Discovery Protocol (LLDP): An open standard (IEEE 802.1AB) Layer 2 protocol that provides the same neighbor discovery function as CDP but is vendor-neutral.

- Network Time Protocol (NTP): A protocol used to synchronize device clocks across a network with a reference time source (NTP server). This is critical for accurate log timestamps and security functions. The stratum level indicates the distance from the authoritative clock source.

- TFTP (Trivial File Transfer Protocol): A simple, connectionless protocol used for transferring files (like Cisco IOS images or configuration files) between network devices and a server. The boot system command is used to instruct the device on which image to load upon reload.

#### Cisco Discovery Protocol (CDP)
##### Enable/disable
```
R1(config)# cdp run
R1(config)# interface s0/0/1
R1(config-if)# no cdp enable
R1(config-if)# exit
```
##### Shows
```
R1# show cdp
R1# show cdp neighbors
R1# show cdp neighbors detail
```
#### Link Layer Discovery Protocol (LLDP)
##### Enable/disable
```
R1(config)# lldp run
R1(config)# int g0/0
R1(config-if)# no lldp transmit
R1(config-if)# exit
```
##### Shows
```
R1# show lldp
R1# show lldp neighbors
R1# show cdp neighbors detail
```
#### NTP - Network Time Protocol
NTP maximum hop count is 15 (stratum 16).

- **Stratum 0:** Authoritative  time sources which are high-precision timekeeping devices.
- **Stratum 1:** Devices that are directly connected (children) to authoritative time sources (stratum 0 devices)
- **Stratum 2 - 15:** Devices connected to other none authoritative devices which acts as a bridge from the time source
- **Stratum > 15:** Unsynchronized device

```
R1(config)# ntp server 209.165.200.225
---
R1# show clock detail
R1# show ntp status
R1# show ntp associations
```
#### TFTP and IOS images
```
! If more than one active interface source may be need to specified
R1(config)# ip tftp source interface g0/0
R1# copy tftp: flash:
R1# show flash:
! On router
R1(config)# boot system flash c1900-universalk9-mz.SPA.155-3.M4a.bin
! On switch
S1(config)# boot system c2960-lanbasek9-mz.150-2.SE4.bin
R1# reload
```
