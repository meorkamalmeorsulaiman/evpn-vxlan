# Where is the MAC/IP Binding?

As we have seen from the previous section, we only see the mac learning thru BGP. However, there is still no clarity on what is it use for? The MAC/IP advertisement is for inter-subnet communication. The method of inter-subnet called Integrated Routing and Bridging (IRB).
More details on [RFC9135](https://www.rfc-editor.org/rfc/rfc9135.html)

## IRB Component

The are few component require which is:
- IP-VRF table (VRF)
- MAC-VRF table (Bridge table)

The MAC-VRF table is where we activated in the earlier section - part-4 The `VNI` or sometime it is called `L2VNI` that defined the L2 domain. The `L2VNI` will then attached to IP-VRF using L3 interface called **IRB Interface**

The IP address field is optional in the MAC/IP advertisement. Up to this point, we only setup L2VNI.  It does not provide any Layer-3(L3) service - default gateway. In order to provide L3 serivce, the leaf switch has to be a default gateway. 
The switch will then advertise the default gateway address using MAC/IP advertisement. A consideration for default gateway is that all the leaf switches will need to use the same mac address.

## Symetric and Asymetric IRB

There are 2 mode of IRB:
- Symmetric
- Asymmetric

In symmetric, both ingress and egress switches perform MAC and IP lookup. But inter-subnet forwarding between IP-VRF table and for asymmetric, the forwarding for inter-subnet was done thru MAC-VRF table. 
They are different, more details [RFC 9135](https://www.rfc-editor.org/rfc/rfc9135.html#name-symmetric-and-asymmetric-ir) In the case of EVPN-VxLAN, symetric IRB is enough. 
Example, when the data plane hit the ingress leaf. It will perform lookup on the IP-VRF table and then encapsulate into an IP packet (VxLAN) including ingress and egress MAC address. Then sent it to the remote leaf. Later the remote leaf will perform lookup on the IP-VRF table. 
As mentioned earlier, each MAC-VRF table associated with IP-VRF and therefore, the switch will know the correct MAC/IP. In symmetric IRB, the local switch doesn't need to store the remote MAC address entry - Why? This is because it is learnt thru EVPN.

## Activate 2nd MAC-VRF - VLAN200

Let's activate `VLAN 200` MAC-VRF table on `leaf03`
```
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback10
  member vni 1000
    ingress-replication protocol bgp
  member vni 2000
    ingress-replication protocol bgp
```

Let's look at the type-3 route
```
leaf03# show bgp l2vpn evpn rd 10.10.10.3:32967 | i Tunnel
      PMSI Tunnel Attribute:
        flags: 0x00, Tunnel type: Ingress Replication
        Label: 2000, Tunnel Id: 10.10.10.3
```
We only turning it on `leaf03`, let's activate on the rest of the leaf switches
```
leaf03# show bgp l2vpn evpn rd 10.10.10.3:32967 | i Tunnel
      PMSI Tunnel Attribute:
        flags: 0x00, Tunnel type: Ingress Replication
        Label: 2000, Tunnel Id: 10.10.10.3
      PMSI Tunnel Attribute:
        flags: 0x00, Tunnel type: Ingress Replication
        Label: 2000, Tunnel Id: 10.10.10.4
      PMSI Tunnel Attribute:
        flags: 0x00, Tunnel type: Ingress Replication
        Label: 2000, Tunnel Id: 10.10.10.5
```
Now it work correctly. Let's continue to activate the IP-VRF table

## IP-VRF Table - L3VNI

`L3VNI` is the term for a VNI that attached to IP-VRF table. First turn on the feature, as we are using anycast gateway
```
feature fabric forwarding
feature interface-vlan
```

Set the anycast gateway mac on all the leaf switches
```
fabric forwarding anycast-gateway-mac 0001.0001.0001
```

Enable VLAN and VNI Mapping
```
vlan 1000
  name TN-A
  vn-segment 10000
```

Proceed with enabling VRF and active EVPN on `leaf03`
```
vrf context TN-A
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
  vni 10000
router bgp 65100
  vrf TN-A
    address-family ipv4 unicast
```

Configure default-gateway
```
interface Vlan100
  no shutdown
  vrf member TN-A
  ip address 192.168.100.254/24
  fabric forwarding mode anycast-gateway
interface Vlan1000
  no shutdown
  vrf member TN-A
  ip forward
```

Let's associate the L3VNI to the NVE
```
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback10
  member vni 1000
    ingress-replication protocol bgp
  member vni 2000
    ingress-replication protocol bgp
  member vni 10000 associate-vrf
```

Let's ping the gateway and see the routing table
```
leaf03# show bgp l2vpn evpn | be Network
   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.10.10.3:32867    (L2VNI 1000)
*>l[2]:[0]:[0]:[48]:[aabb.cc00.6000]:[0]:[0.0.0.0]/216
                      10.10.10.3                        100      32768 i
*>i[2]:[0]:[0]:[48]:[aabb.cc00.8000]:[0]:[0.0.0.0]/216
                      10.10.10.5                        100          0 i
*>l[2]:[0]:[0]:[48]:[aabb.cc00.6000]:[32]:[192.168.100.6]/272
                      10.10.10.3                        100      32768 i
*>l[3]:[0]:[32]:[10.10.10.3]/88
                      10.10.10.3                        100      32768 i
*>i[3]:[0]:[32]:[10.10.10.4]/88
                      10.10.10.4                        100          0 i
*>i[3]:[0]:[32]:[10.10.10.5]/88
                      10.10.10.5                        100          0 i
```

Now you can see the MAC/IP advertisement, and if we look further
```
leaf03# show bgp l2vpn evpn rd 10.10.10.3:32867 192.168.100.6
BGP routing table information for VRF default, address family L2VPN EVPN
Route Distinguisher: 10.10.10.3:32867    (L2VNI 1000)
BGP routing table entry for [2]:[0]:[0]:[48]:[aabb.cc00.6000]:[32]:[192.168.100.
6]/272, version 30
Paths: (1 available, best #1)
Flags: (0x000102) (high32 00000000) on xmit-list, is not in l2rib/evpn

  Advertised path-id 1
  Path type: local, path is valid, is best path, no labeled nexthop
  AS-Path: NONE, path locally originated
    10.10.10.3 (metric 0) from 0.0.0.0 (10.10.10.3)
      Origin IGP, MED not set, localpref 100, weight 32768
      Received label 1000 10000
      Extcommunity: RT:65100:1000 RT:65100:10000 ENCAP:8 Router MAC:5003.0000.1b08
```

We can see the MAC, IP, MPLS Label2 and the RT's. We still yet to see the IP-VRF for this TN. Let's proceed to activate on all other leaf switches.

## IP-VRF

We still yet to see the L3VNI in action, let's enable anycast gateway for `VLAN200`
```
interface Vlan200
  no shutdown
  mtu 9216
  vrf member TN-A
  ip address 192.168.200.254/24
  fabric forwarding mode anycast-gateway
```

At this point we should be able to ping between subnet. Before let's ping the `VLAN200` gateway and look at the routing table
```
host09#ping 192.168.200.254
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.200.254, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 2/3/4 ms
```

Now we can see the IP-VRF table 
```
leaf03# show bgp l2vpn evpn | be L3VNI
Route Distinguisher: 10.10.10.3:3    (L3VNI 10000)
*>i[2]:[0]:[0]:[48]:[aabb.cc00.7000]:[32]:[192.168.100.7]/272
                      10.10.10.4                        100          0 i
*>i[2]:[0]:[0]:[48]:[aabb.cc00.8000]:[32]:[192.168.100.8]/272
                      10.10.10.5                        100          0 i
*>i[2]:[0]:[0]:[48]:[aabb.cc00.a000]:[32]:[192.168.200.10]/272
                      10.10.10.4                        100          0 i
```
The L3VNI correct, that's are the remote endpoint from other leaf switch. Based on the steps, we can see that when we enable L3 service. The MAC/IP get's advertised and when we enabled 2nd gateway. We started to see the IP-VRF table. 
The IP-VRF table rd derive from `Router ID` + `VRF ID`


