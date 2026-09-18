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