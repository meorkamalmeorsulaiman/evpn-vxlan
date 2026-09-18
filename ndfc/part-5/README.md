# Part 5 - Multi-site

![Fab add](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-5/images/ndfc/5-fab-add.png)

![Fab ms](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-5/images/ndfc/5-fab-ms.png)

![Fab dci](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-5/images/ndfc/5-fab-dci.png)

![Fab flag](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-5/images/ndfc/5-fab-flag.png)

![Fab ifc](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-5/images/ndfc/5-fab-ifc-subnet.png)

![Fab child](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-5/images/ndfc/5-fab-child.png)

Validate that both border gateway form control plane session

```
bgw07# show bgp l2vpn evpn summary 
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.2.0.5, local AS number 64521
BGP table version is 5, L2VPN EVPN config peers 2, capable peers 2
0 network entries and 0 paths using 0 bytes of memory
BGP attribute entries [0/0], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/
PfxRcd
10.2.0.3        4 64521        101        102        5    0    0 01:32:58 0     
    
20.2.0.2        4 64522          6          6        5    0    0 00:00:48 0     
    

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5     T
ype-12   
10.2.0.3        I 64521 0          0          0          0          0          0
         
20.2.0.2        E 64522 0          0          0          0          0          0
```

```
bgw08# show bgp l2vpn evpn summary 
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 20.2.0.2, local AS number 64522
BGP table version is 5, L2VPN EVPN config peers 2, capable peers 2
0 network entries and 0 paths using 0 bytes of memory
BGP attribute entries [0/0], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/
PfxRcd
10.2.0.5        4 64521          6          6        5    0    0 00:00:19 0     
    
20.2.0.1        4 64522         80         80        5    0    0 01:10:51 0     
    

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5     T
ype-12   
10.2.0.5        E 64521 0          0          0          0          0          0
         
20.2.0.1        I 64522 0          0          0          0          0          0
```