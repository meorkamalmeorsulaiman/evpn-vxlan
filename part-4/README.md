# BGP EVPN

## Feature Require
spine
```
feature bgp
feature nv overlay
nv overlay evpn
```

leaf
```
feature bgp
nv overlay evpn
```

## Turning on BGP EVPN
spine
```
router bgp 65100
  router-id 10.10.10.R
  neighbor 10.10.10.0/24
    remote-as 65100
    update-source loopback10
    address-family l2vpn evpn
      send-community
      send-community extended
      route-reflector-client
```

leaf
```
router bgp 65100
  router-id 10.10.10.R
  neighbor 10.10.10.1
    remote-as 65100
    update-source loopback10
    address-family l2vpn evpn
      send-community both
  neighbor 10.10.10.2
    remote-as 65100
    update-source loopback10
    address-family l2vpn evpn
      send-community both
```

Validate the session are up on the leaf
```
leaf03# show bgp l2vpn evpn summary 
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.10.10.3, local AS number 65100
BGP table version is 4, L2VPN EVPN config peers 2, capable peers 2
0 network entries and 0 paths using 0 bytes of memory
BGP attribute entries [0/0], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/
PfxRcd
10.10.10.1      4 65100          7          7        4    0    0 00:01:04 0     
    
10.10.10.2      4 65100          7          7        4    0    0 00:01:02 0     
    

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5    
10.10.10.1      I 65100 0          0          0          0          0         
10.10.10.2      I 65100 0          0          0          0          0
```

We still haven't announce anything yet. At the moment, you can already see the route types for EVPN.

## Switching from Multicast to EVPN

First, let's activate the L2VNI for EVPN
```
evpn
  vni 1000 l2
    rd auto
    route-target import auto
    route-target export auto
```

We then switch the VTEP to use BGP instead of multicast group and removing `VNI 2000` for cleaner output later

```
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback10
  member vni 1000
    ingress-replication protocol bgp
```

These configurations should apply on all the leaf switches. Once you complete, you can start received EVPN routes from the spines
```
leaf03# show bgp l2vpn evpn summary 
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.10.10.3, local AS number 65100
BGP table version is 20, L2VPN EVPN config peers 2, capable peers 2
5 network entries and 7 paths using 1380 bytes of memory
BGP attribute entries [5/1760], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [4/16]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/
PfxRcd
10.10.10.1      4 65100         35         27       20    0    0 00:19:01 2     
    
10.10.10.2      4 65100         35         27       20    0    0 00:19:00 2     
    

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5    
10.10.10.1      I 65100 2          0          2          0          0         
10.10.10.2      I 65100 2          0          2          0          0
```

You can see that there are 2 type 3 routes and we already discover the VxLAN Tunnel
```
leaf03# show nve peers 
Interface Peer-IP                                 State LearnType Uptime   Route
r-Mac       
--------- --------------------------------------  ----- --------- -------- -----
------------
nve1      10.10.10.4                              Up    CP        00:03:09 n/a  
            
nve1      10.10.10.5                              Up    CP        00:04:42 n/a
```
## The Peers Discovered?

What we are seeing is the tunnel get dynamically build with the help of BGP EVPN. When we activate L2VNI for EVPN, the switch essentially create an route for a particular L2VNI
```
leaf03# show bgp l2vpn evpn | i Route
BGP table version is 20, Local Router ID is 10.10.10.3
Route Distinguisher: 10.10.10.3:32867    (L2VNI 1000)
Route Distinguisher: 10.10.10.4:32867
Route Distinguisher: 10.10.10.5:32867
```

The RD is derive from `Router ID` + `VLAN ID` + `32767` This is the format for L2VNI. Since we enable on all of the leaf switches, thus we also learning route from other leaf.
Since it route table are seperated, it were exported and imported automatically with the command `route-target import auto` and `route-target export auto`. The RT derive from `ASN`:`VNI`
```
leaf03# show bgp l2vpn evpn rd 10.10.10.5:32867 | i RT
      Extcommunity: RT:65100:1000 ENCAP:8
      Extcommunity: RT:65100:1000 ENCAP:8
leaf03# show bgp l2vpn evpn rd 10.10.10.4:32867 | i RT
      Extcommunity: RT:65100:1000 ENCAP:8
      Extcommunity: RT:65100:1000 ENCAP:8
leaf03# show bgp evi 1000
-----------------------------------------------
  L2VNI ID                     : 1000 (L2-1000)
  RD                           : 10.10.10.3:32867
  Prefixes (local/total)       : 1/3
  Created                      : Oct 27 02:22:30.726102
  Last Oper Up/Down            : Oct 27 02:22:30.734699 / never
  Enabled                      : Yes
  Active Export RT list        : 
        65100:1000 
  Active Import RT list        : 
        65100:1000 
```

## The Type-3 Route

The type-3 route is a route advertisement is use to handle flooding traffic. The route essentially carry P-Tunnel ID - PSMI Tunnel Attribute, which can be use by the local leaf switch to forward the BUM traffic to remote P-Tunnel.
You can see the peer tunnel IP from the route-type 3 attribute
```
leaf03# show bgp l2vpn evpn rd 10.10.10.3:32867 | i Tunnel
      PMSI Tunnel Attribute:
        flags: 0x00, Tunnel type: Ingress Replication
        Label: 1000, Tunnel Id: 10.10.10.3
      PMSI Tunnel Attribute:
        flags: 0x00, Tunnel type: Ingress Replication
        Label: 1000, Tunnel Id: 10.10.10.4
      PMSI Tunnel Attribute:
        flags: 0x00, Tunnel type: Ingress Replication
        Label: 1000, Tunnel Id: 10.10.10.5
```

Once the switch received, it know how to forward the BUM traffic. This method is similar to VPLS with Auto-Discovery where the remote PE's are discovery dynamically. Below show the VNI Egress remote peer
```
leaf03# show l2route evpn imet all

Flags- (F): Originated From Fabric, (W): Originated from WAN

Topology ID VNI         Prod  IP Addr                                 Flags  
----------- ----------- ----- --------------------------------------- -------
100         1000        BGP   10.10.10.4                              -      
100         1000        BGP   10.10.10.5                              -      
100         1000        VXLAN 10.10.10.3                              -  
```
More details:
- Multi-Destination Traffc [RFC 7432](https://datatracker.ietf.org/doc/html/rfc7432#autoid-45)
- Route Type-3 [RFC 7432](https://datatracker.ietf.org/doc/html/rfc7432#autoid-15)
- P-Tunnel Identification [RFC 7432](https://datatracker.ietf.org/doc/html/rfc7432#autoid-47)

## The Type-2 Route
Let's boot `host06` and `host08`, Host MAC:
- `host06` - `aabb.cc00.6000`
- `host07` - `aabb.cc00.8000`
  
Check the EVPN summary
```
leaf03# show bgp l2vpn evpn summary 
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.10.10.3, local AS number 65100
BGP table version is 23, L2VPN EVPN config peers 2, capable peers 2
8 network entries and 11 paths using 2208 bytes of memory
BGP attribute entries [9/3168], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [4/16]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.1      4 65100         73         64       23    0    0 00:56:12 3         
10.10.10.2      4 65100         73         64       23    0    0 00:56:10 3         

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5    
10.10.10.1      I 65100 3          1          2          0          0         
10.10.10.2      I 65100 3          1          2          0          0
```

Now we can see few entry for route Type-2 
```
leaf03# show bgp l2vpn evpn route-type 2 | i entry
BGP routing table entry for [2]:[0]:[0]:[48]:[aabb.cc00.6000]:[0]:[0.0.0.0]/216, version 21
BGP routing table entry for [2]:[0]:[0]:[48]:[aabb.cc00.8000]:[0]:[0.0.0.0]/216, version 23
BGP routing table entry for [2]:[0]:[0]:[48]:[aabb.cc00.8000]:[0]:[0.0.0.0]/216, version 22
```
Route Type-2, is a MAC/IP Advertisement. The use of this route is to learning local and remote mac address as part of determining the reachability of unicast mac address. When we boot up both hosts, the host may have signal the switch to register their mac address locally and then advertise it thru BGP using Type-2 route.
With this information, the local switch should have a mac entry for the remote mac address. The Type-2 route also carries MPLS-Label1 that indicate the VNI

MPLS-Label1
```
leaf03# show bgp l2vpn evpn aabb.cc00.8000 | i Received
      Received label 1000
      Received label 1000
      Received label 1000
```

RT's
```
leaf03# show bgp l2vpn evpn aabb.cc00.8000 | i RT
      Extcommunity: RT:65100:1000 ENCAP:8
      Extcommunity: RT:65100:1000 ENCAP:8
      Extcommunity: RT:65100:1000 ENCAP:8
```

This will then be use to import and stitch the VLAN to VNI mapping. More details on Route Type-2 Attributes [RFC7434](https://datatracker.ietf.org/doc/html/rfc7432#autoid-14) As a result, you will see the mac learning:
```
leaf03# show mac address-table interface nve 1
Legend: 
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan,
        (NA)- Not Applicable
   VNI      MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
C  100     aabb.cc00.8000   dynamic  NA         F      F    nve1(10.10.10.5)
leaf03# show mac address-table vlan 100
Legend: 
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan,
        (NA)- Not Applicable
   VLAN     MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
*  100     aabb.cc00.6000   dynamic  NA         F      F    Eth1/7
C  100     aabb.cc00.8000   dynamic  NA         F      F    nve1(10.10.10.5)
```

At this point, both host should have L2 reachability. Let's ping
```
host06#ping 192.168.100.8
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.100.8, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 7/16/30 ms
```

That's work. At this point, we still using ingress replication where the switch will unicast all the packet to all the leaf
```
leaf03# show nve vni 1000
Codes: CP - Control Plane        DP - Data Plane          
       UC - Unconfigured         SA - Suppress ARP        
       SU - Suppress Unknown Unicast 
       Xconn - Crossconnect      
       MS-IR - Multisite Ingress Replication 
       HYB - Hybrid IRB mode
    
Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags
--------- -------- ----------------- ----- ---- ------------------ -----
nve1      1000     UnicastBGP        Up    CP   L2 [100]
```

Let's continue with next part and see what EVPN can offer.

