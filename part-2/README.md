# The VxLAN

This part we are going to look at the VxLAN, the goal here is trying to connect hosts of `VLAN 200` over a Layer-3 (L3) network.

## Features required

There are few features that need to turn on. This feature only required on the leaf 

```
feature vn-segment-vlan-based
feature nv overlay
```

## VLAN to VNI Mapping

Once feature enabled, let's proceed to map the VLAN to VNI. In this case for `VLAN 200` we will be using `VNI 2000`. This has to be configured on the leaf where the `VLAN 200` endpoint connected.

```
vlan 200
  name VLAN200
  vn-segment 2000
```

## VxLAN Tunnel Termination with Static Ingress Replication

After we created the mapping, we then proceed to create the VxLAN tunnel termination. These again require on the leaf where the `VLAN 200` endpoint connected.

leaf03
```
interface nve1
  no shutdown
  source-interface loopback10
  member vni 2000
    ingress-replication protocol static
      peer-ip 10.10.10.4
```

leaf04
```
interface nve1
  no shutdown
  source-interface loopback10
  member vni 2000
    ingress-replication protocol static
      peer-ip 10.10.10.3
```

As you can see from the configuration, the VxLAN tunnel are configured in bidrectional manner. The configuration look like a GRE? Let's configure the access port on both leaf switches:

```
interface Ethernet1/6
  description host09
  switchport access vlan 200
```

## Let's ping 

Let's give a ping from `host10 ` to `host09`

```
host10#ping 192.168.200.9
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.200.9, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 14/17/21 ms
```
As you can see the ping is successful, let see the mac address on both leaf switches:

leaf03
```
leaf03# show mac address-table vlan 200
Legend: 
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan,
        (NA)- Not Applicable
   VLAN     MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
*  200     aabb.cc00.9000   dynamic  NA         F      F    Eth1/6
*  200     aabb.cc00.a000   dynamic  NA         F      F    nve1(10.10.10.4)
```
leaf04
```
leaf04# show mac address-table vlan 200
Legend: 
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan,
        (NA)- Not Applicable
   VLAN     MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
*  200     aabb.cc00.9000   dynamic  NA         F      F    nve1(10.10.10.3)
*  200     aabb.cc00.a000   dynamic  NA         F      F    Eth1/6
```
Host09: `aabb.cc00.9000` and Host10: `aabb.cc00.a000` mac address.

## Digest

What we are seeing from the mac entry, both switches are learning the local mac address from `Eth1/6` which is expected. However, the remote mac address was learnt from the `nve1` that we have created earlier. 
The `NVE` interface is an interface that use to terminate a VxLAN tunnel. VxLAN is essentially a tunneling protocol that tunnel a VLAN over L3 network. The protocol runs is an IP based that run over UDP. 
VxLAN allow us to bridge or stretch a layer-2(L2) domain over an IP network and there are benefit running VxLAN. You can see the mac learnt from a particular VxLAN Tunnel:

```
leaf03# show mac address-table interface nve 1
Legend: 
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan,
        (NA)- Not Applicable
   VNI      MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
*  200     aabb.cc00.a000   dynamic  NA         F      F    nve1(10.10.10.4)
```

More details of VxLAN header format [VxLAN Header Format](https://datatracker.ietf.org/doc/html/rfc7348#autoid-12). Within a VxLAN header the is a portion called `VxLAN Segment ID`, this ID essentialy isolate a particular L2 domain. In this case, 
the pair of leaf switches will know what is the L2 domain attach to particular VNI, in this case `VLAN 200` as we configured:

```
vlan 200
  name VLAN200
  vn-segment 2000
```

You can see what `VNI` carried over the VxLAN tunnel:

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
nve1      2000     UnicastStatic     Up    DP   L2 [200]
```
Looking at the previous `NVE` interface configuration which specify `ingress-replication protocol static` This is a mode for the VxLAN tunnel to replicate any data plane traffic toward the remote tunnel. 
Example, an arp request received by `leaf03` from `host09` will replicated to `leaf10`. You can add more peers on the `NVE` interace as the remote VxLAN tunnel and the data plane traffic will then replicate to all the `NVE` peers. 
At some point we have many peers essentially become full mesh? Let's continue in part-3.


