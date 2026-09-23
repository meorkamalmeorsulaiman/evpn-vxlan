# Part 8 - External Connectivity

Create fabric

![Ext fab crt](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-8/images/ndfc/8-ext-fab-crt.png)

![Ext fab asn](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-8/images/ndfc/8-ext-fab-asn.png)

![Ext fab mode](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-8/images/ndfc/8-ext-fab-mode.png)

Add Switch

![Ext fab sw](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-8/images/ndfc/8-ext-fab-sw.png)

![Ext fab sw add](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-8/images/ndfc/8-ext-fab-sw-add.png)

Edit link - auto created 

![Ext fab link 1](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-8/images/ndfc/8-ext-fab-link-1.png)

![Ext fab link 2](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-8/images/ndfc/8-ext-fab-link-2.png)

![Ext fab link 3](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-8/images/ndfc/8-ext-fab-link-3.png)

![Ext fab link 4](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-8/images/ndfc/8-ext-fab-link-4.png)

![Ext fab link 5](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-8/images/ndfc/8-ext-fab-link-5.png)

It will apply the policy, not the VRF-Lite peering
```
ip prefix-list default-route seq 5 permit 0.0.0.0/0 le 1
ip prefix-list host-route seq 5 permit 0.0.0.0/0 eq 32
route-map extcon-rmap-filter deny 10
  match ip address prefix-list default-route
route-map extcon-rmap-filter deny 20
  match ip address prefix-list host-route
route-map extcon-rmap-filter permit 1000
route-map extcon-rmap-filter-allow-host deny 10
  match ip address prefix-list default-route
route-map extcon-rmap-filter-allow-host permit 1000
ipv6 prefix-list default-route-v6 seq 5 permit 0::/0
ipv6 prefix-list host-route-v6 seq 5 permit 0::/0 eq 128
route-map extcon-rmap-filter-v6 deny 10
  match ipv6 address prefix-list default-route-v6
route-map extcon-rmap-filter-v6 deny 20
  match ipv6 address prefix-list host-route-v6
route-map extcon-rmap-filter-v6 permit 1000
route-map extcon-rmap-filter-v6-allow-host deny 10
  match ipv6 address prefix-list default-route-v6
route-map extcon-rmap-filter-v6-allow-host permit 1000
configure terminal
```

VRF-Lite peering

![Ext fab vrf 1](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-8/images/ndfc/8-ext-fab-vrf-1.png)

![Ext fab vrf 2](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-8/images/ndfc/8-ext-fab-vrf-2.png)

![Ext fab vrf 3](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-8/images/ndfc/8-ext-fab-vrf-3.png)

![Ext fab vrf 4](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-8/images/ndfc/8-ext-fab-vrf-4.png)

![Ext fab vrf 5](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-8/images/ndfc/8-ext-fab-vrf-5.png)

![Ext fab vrf 6](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-8/images/ndfc/8-ext-fab-vrf-6.png)

![Ext fab vrf 7](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-8/images/ndfc/8-ext-fab-vrf-7.png)

![Ext fab vrf 8](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-8/images/ndfc/8-ext-fab-vrf-8.png)

![Ext fab vrf 9](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-8/images/ndfc/8-ext-fab-vrf-9.png)

It will apply the vrf-lite peering config on the border leaf
```
vrf context non-prod
  ip route 0.0.0.0/0 10.33.0.6
exit
router bgp 64521
  vrf non-prod
    address-family ipv4 unicast
      network 0.0.0.0/0
      exit
    neighbor 10.33.0.6
      remote-as 64541
      address-family ipv4 unicast
        send-community both
        route-map extcon-rmap-filter out
configure terminal
interface ethernet1/3.2
  encapsulation dot1q 2
  mtu 9216
  vrf member non-prod
  ip address 10.33.0.5/30
  no shutdown
configure terminal
```

You can proceed to apply the peering config on the external device and validate the BGP session established

```
bgw07# show bgp ipv4 unicast summary vrf non-prod 
BGP summary information for VRF non-prod, address family IPv4 Unicast
BGP router identifier 10.5.0.1, local AS number 64521
BGP table version is 13, IPv4 Unicast config peers 1, capable peers 1
9 network entries and 9 paths using 2340 bytes of memory
BGP attribute entries [8/2944], BGP AS path entries [3/30]
BGP community entries [0/0], BGP clusterlist entries [3/12]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/
PfxRcd
10.33.0.6       4 64541          7          5       13    0    0 00:01:12 4 

R12#show bgp ipv4 uni summary 
BGP router identifier 10.1.1.12, local AS number 64541
BGP table version is 5, main routing table version 5
4 network entries using 576 bytes of memory
4 path entries using 336 bytes of memory
3/3 BGP path/bestpath attribute entries using 480 bytes of memory
2 BGP AS-PATH entries using 48 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1440 total bytes of memory
BGP activity 4/0 prefixes, 4/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.12.17.17     4        64512      73      71        5    0    0 01:01:30        3
10.33.0.5       4        64521      14      17        5    0    0 00:10:17        0
```

Few type-5 routes added in the ip-vrf table
```
bgw07# show bgp l2vpn evpn vni-id 50000
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 40, Local Router ID is 10.2.0.5
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.2.0.5:4    (L3VNI 50000)
*>i[2]:[0]:[0]:[48]:[5003.0000.1b08]:[0]:[0.0.0.0]/216
                      10.3.0.2                          100          0 i
*>i[2]:[0]:[0]:[48]:[5004.0000.1b08]:[0]:[0.0.0.0]/216
                      10.3.0.2                          100          0 i
*>l[2]:[0]:[0]:[48]:[5007.0000.1b08]:[0]:[0.0.0.0]/216
                      10.3.0.5                          100      32768 i
*>l[5]:[0]:[0]:[0]:[0.0.0.0]/224
                      10.3.0.5                          100      32768 i
*>l[5]:[0]:[0]:[24]:[192.168.23.0]/224
                      10.3.0.5                                       0 64541 64512 i
*>l[5]:[0]:[0]:[24]:[192.168.24.0]/224
                      10.3.0.5                                       0 64541 64512 i
*>l[5]:[0]:[0]:[32]:[10.5.0.1]/224
                      10.3.0.5                 0        100      32768 ?
*>l[5]:[0]:[0]:[32]:[10.5.0.2]/224
                      10.3.0.5                 0        100          0 ?
*>l[5]:[0]:[0]:[32]:[10.5.0.3]/224
                      10.3.0.5                 0        100          0 ?
*>l[5]:[0]:[0]:[32]:[10.5.0.4]/224
                      10.3.0.5                 0        100          0 ?
*>l[5]:[0]:[0]:[32]:[12.12.12.12]/224
                      10.3.0.5                 0                     0 64541 i
*>l[5]:[0]:[0]:[32]:[14.14.14.14]/224
                      10.3.0.5                                       0 64541 64512 64542 i
```