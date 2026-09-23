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
```