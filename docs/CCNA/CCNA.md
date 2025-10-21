# CCNA

## Basic

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
```
R1(config)# ip domain-name cisco.com
R1(config)# crypto key generate rsa
R1(config)# user admin secret ccna
R1(config)# line vty 0 15
R1(config-line)# transport input ssh
R1(config-line)# login local
```

## InterVLAN routing

##### Subinterfaces
```
R1(config)# int g0/0.10
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# encapsulation dot1Q 11 native
R1(config-subif)# ip address 172.17.10.1 255.255.255.0
```

## VLANs
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