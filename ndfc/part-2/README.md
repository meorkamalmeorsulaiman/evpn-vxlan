# Part 1 - Onboard Switches

![Sw](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-2/images/ndfc/2-sw-add.png)

![Sw cred](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-2/images/ndfc/2-sw-cred.png)

![Sw IP](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-2/images/ndfc/2-sw-ip.png)

![Sw Onboard](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-2/images/ndfc/2-sw-onboard.png)

Wait untill all swithces status added and close the window

![Sw added](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-2/images/ndfc/2-sw-added.png)

Make sure switch in normal mode

![Sw mode](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-2/images/ndfc/2-sw-mode.png)

# Set Role

![Sw set role 1](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-2/images/ndfc/2-sw-role-1.png)

![Sw set role 2](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-2/images/ndfc/2-sw-role-2.png)

![Sw set role 3](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-2/images/ndfc/2-sw-role-3.png)

# Set VPC

![Sw set vpc 1](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-2/images/ndfc/2-sw-vpc-1.png)

![Sw set vpc 2](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-2/images/ndfc/2-sw-vpc-2.png)

# Recalculate and Deploy

![Sw deploy 1](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-2/images/ndfc/2-sw-deploy-1.png)

![Sw deploy 2](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-2/images/ndfc/2-sw-deploy-2.png)

![Sw deploy 3](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-2/images/ndfc/2-sw-deploy-3.png)


Check sessions on spine

```
spine01# show bgp l2vpn evpn summary 
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.2.0.3, local AS number 64521
BGP table version is 6, L2VPN EVPN config peers 4, capable peers 4
0 network entries and 0 paths using 0 bytes of memory
BGP attribute entries [0/0], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/
PfxRcd
10.2.0.1        4 64521         19         19        6    0    0 00:13:54 0     
    
10.2.0.2        4 64521          6          6        6    0    0 00:00:20 0     
    
10.2.0.4        4 64521         19         19        6    0    0 00:13:49 0     
    
10.2.0.5        4 64521          6          6        6    0    0 00:00:07 0     
    

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5     T
ype-12   
10.2.0.1        I 64521 0          0          0          0          0          0
         
10.2.0.2        I 64521 0          0          0          0          0          0
         
10.2.0.4        I 64521 0          0          0          0          0          0
         
10.2.0.5        I 64521 0          0          0          0          0          0
```