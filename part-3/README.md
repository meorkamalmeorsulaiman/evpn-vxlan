# The BUM

We have extend our L2 domain in part-2. We already thinking that it's going to be a full mesh. That's how we going to forwards the BUM traffic. It stands for Broadcast, Unknown Unicast and Multicast. The are multiple mode that we can implement on VxLAN fabric.
Another method is to turn on multicast on the undelay instead of relying on ingress replication.

## Feature Required

Activate PIM on all the switches within the fabric

`feature pim`

## Enable Multicast Underlay

There are 2 type multicast which we can use:
- ASM
- Bidir

We will use ASM with Anycast RP

leaf's

```
ip pim rp-address 10.10.10.110 group-list 224.0.0.0/4
interface Ethernet1/1-2
  ip pim sparse-mode
interface loopback10
  ip pim sparse-mode
```

spines
```
ip pim rp-address 10.10.10.110 group-list 224.0.0.0/4
ip pim anycast-rp 10.10.10.110 10.10.10.1
ip pim anycast-rp 10.10.10.110 10.10.10.2
!
interface loopback110
  ip pim sparse-mode
interface Ethernet1/1-3
  ip pim sparse-mode
interface loopback10
  ip pim sparse-mode
interface loopback110
  ip address 10.10.10.110/32
  ip ospf network point-to-point
  ip router ospf 65100 area 0.0.0.0
  ip pim sparse-mode
```
We introduce new interface on the spine for the RP address. Let's validate the multicast

Anycast RP member:
```
spine01# show ip pim rp
PIM RP Status Information for VRF "default"
BSR disabled
Auto-RP disabled
BSR RP Candidate policy: None
BSR RP policy: None
Auto-RP Announce policy: None
Auto-RP Discovery policy: None

Anycast-RP 10.10.10.110 members:
  10.10.10.1*  10.10.10.2  

RP: 10.10.10.110*, (0), 
 uptime: 00:09:44   priority: 255, 
 RP-source: (local),  
 group ranges:
 224.0.0.0/4   
spine01# 

```
Leaf static Anycast RP

```
leaf03# show ip pim rp
PIM RP Status Information for VRF "default"
BSR disabled
Auto-RP disabled
BSR RP Candidate policy: None
BSR RP policy: None
Auto-RP Announce policy: None
Auto-RP Discovery policy: None

RP: 10.10.10.110, (0), 
 uptime: 00:06:59   priority: 255, 
 RP-source: (local),  
 group ranges:
 224.0.0.0/4
```
We are not having any multicast received yet, thus the mroute cache will be empty. 

## Turning on VLAN100 Endpoint

We have there endpoint for `VLAN 100`, we are going to use multicast underlay to forward the BUM traffic instead of ingress replication. Let's configure the VLAN to VNI mapping:

```
vlan 100
  name VLAN100
  vn-segment 1000
```
Let's proceed to configure the VxLAN tunnel

## Attach a Multicast Group for VNI 1000

Let's attach `VNI 1000` to a particular multicast group on all leaf switches:

```
interface nve1
  member vni 1000
    mcast-group 239.0.0.1
```
At this point, the leaf switches acting as multicast sender and receiver.

## Let's ping

The ping work's

```
host06#ping 192.168.100.7
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.100.7, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 10/16/22 ms
host06#ping 192.168.100.8
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.100.8, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 10/14/21 ms
```
At the time of writing, the VxLAN tunnels configuration has to match on all of the leaf switch. Otherwise the ping won't work:

```
interface nve1
  no shutdown
  source-interface loopback10
  member vni 1000
    mcast-group 239.0.0.1
  member vni 2000
    ingress-replication protocol static
      peer-ip 10.10.10.4
      peer-ip 10.10.10.5
```
Let's see the mac entry and the 

leaf03
```
leaf03# show mac address-table interface nve 1
Legend: 
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan,
        (NA)- Not Applicable
   VNI      MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
*  100     aabb.cc00.7000   dynamic  NA         F      F    nve1(10.10.10.4)
*  100     aabb.cc00.8000   dynamic  NA         F      F    nve1(10.10.10.5)
```
leaf04
```
leaf04# show mac address-table interface nve 1
Legend: 
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan,
        (NA)- Not Applicable
   VNI      MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
*  100     aabb.cc00.6000   dynamic  NA         F      F    nve1(10.10.10.3)
*  100     aabb.cc00.8000   dynamic  NA         F      F    nve1(10.10.10.5)
```
leaf05
```
leaf05# show mac address-table interface nve 1
Legend: 
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan,
        (NA)- Not Applicable
   VNI      MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
*  100     aabb.cc00.6000   dynamic  NA         F      F    nve1(10.10.10.3)
*  100     aabb.cc00.7000   dynamic  NA         F      F    nve1(10.10.10.4)
```

## Digest

What happen was, the traffic was multicast over the VxLAN fabric. You can see the VNI status:

```
leaf03# show nve vni 
Codes: CP - Control Plane        DP - Data Plane          
       UC - Unconfigured         SA - Suppress ARP        
       SU - Suppress Unknown Unicast 
       Xconn - Crossconnect      
       MS-IR - Multisite Ingress Replication 
       HYB - Hybrid IRB mode
    
Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags
--------- -------- ----------------- ----- ---- ------------------ -----
nve1      1000     239.0.0.1         Up    DP   L2 [100]                
nve1      2000     UnicastStatic     Up    DP   L2 [200]
```
The VNI 1000 is part of the multicast group. When the data plane hit the source leaf, it will then multicast toward other leaf that join the multicast group. At this point, we can scale the fabric easily without the need to form full-mesh ingress replication. 
The RP probably need an MSDP but up till this point. We have not implement BGP-EVPN and what is the purpose? Let's look at part-4?



