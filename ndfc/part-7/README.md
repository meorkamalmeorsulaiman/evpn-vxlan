# Part 7 - Migration Link

Single-sided vPC

![Mig vpc 1](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-7/images/ndfc/7-mig-vpc-1.png)

![Mig vpc 2](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-7/images/ndfc/7-mig-vpc-2.png)

![Mig vpc 3](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-7/images/ndfc/7-mig-vpc-3.png)

![Mig vpc 4](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-7/images/ndfc/7-mig-vpc-4.png)

![Mig vpc 5](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-7/images/ndfc/7-mig-vpc-5.png)

![Mig vpc 6](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-7/images/ndfc/7-mig-vpc-6.png)

# Verification

Validate Po up


Validate ping working
```
R18#ping 192.168.23.11
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.23.11, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 13/14/16 ms
R11#ping 14.14.14.14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 14.14.14.14, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 13/17/20 ms
```

Validate mac learn correctly
```
leaf05# show mac address-table vlan 2300
Legend: 
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan,
        (NA)- Not Applicable A – ESI Active Path, S – ESI Standby Path
   VLAN     MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
* 2300     aabb.cc00.b000   dynamic  NA         F      F    Eth1/3
C 2300     aabb.cc01.1020   dynamic  NA         F      F    nve1(10.3.0.2)
C 2300     aabb.cc01.2000   dynamic  NA         F      F    nve1(10.3.0.2)
```

Control plane
```
leaf05# show bgp l2vpn evpn vni-id 30000
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 59, Local Router ID is 10.2.0.2
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.2.0.2:35067    (L2VNI 30000)
*>i[2]:[0]:[0]:[48]:[5007.0000.1b08]:[0]:[0.0.0.0]/216
                      10.3.0.5                          100          0 i
*>i[2]:[0]:[0]:[48]:[5008.0000.1b08]:[0]:[0.0.0.0]/216
                      30.10.0.1                         100          0 64522 i
*>l[2]:[0]:[0]:[48]:[aabb.cc00.b000]:[0]:[0.0.0.0]/216
                      10.3.0.4                          100      32768 i
*>i[2]:[0]:[0]:[48]:[aabb.cc01.1020]:[0]:[0.0.0.0]/216
                      10.3.0.2                          100          0 i
* i                   10.3.0.2                          100          0 i
* i[2]:[0]:[0]:[48]:[aabb.cc01.2000]:[0]:[0.0.0.0]/216
                      10.3.0.2                          100          0 i
*>i                   10.3.0.2                          100          0 i
```

