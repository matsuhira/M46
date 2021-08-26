# About M46

## What is M46
This is a technology that performs IPv4 over IPv6 encapsulation and IPv4-IPv6 conversion using the M46 address (M46A) that maps an IPv4 address to an IPv6 address.<br>
To accommodate IPv4 private addresses, the M46A includes a plane-ID to identify the IPv4 private address. If IPv6 Prefix is ​​64bit, 32bit can be assigned to Plane-ID, so 2 ^ 32 = about 4.2 billion private networks can be accommodated.<br>
The following is the address format of M46A.<br>

```
 |      96 - m bits      |          m bits          |     32 bits  |
 +-----------------------+--------------------------+--------------+
 |      M46A prefix      |  IPv4 network plane ID   | IPv4 address |
 +-----------------------+--------------------------+--------------+
 ```
 
An IP address has two meanings: a host address indicated by the entire address and a route that aggregates them. The host address is represented by / 32 in IPv4, / 128 in M46A, and / 24 in IPv4. The route to be used is represented by / 120 in M46A.<br>

M46 is an abbreviation for Multiple IPv4 --IPv6 mapping, and M46A is an abbreviation for Multiple IPv4 --IPv6 mapped IPv6 address.<br>

## What is M46E
This technology is applied to IPv4 over IPv6 encapsulation using M46A. Encapsulation is performed to add an IPv6 header to an IPv4 packet, and M46A is used as the IPv6 address at that time.<br>
By using an IPv6 address mapped to IPv6 for IPv4 communication, communication in the IPv6 space is possible. This makes it possible to make the backbone network IPv6 only, and also enables IPv4 communication over the IPv6 only network.<br>

M46E generates M46A differently depending on whether or not it advertises the route of M46E. It is called M46E-FP when it is advertised by route, and M46E-PR when it is not advertised by route.<br>

M46E is an abbreviation for Multiple IPv4-IPv6 address mapping encapsulation.<br>

### M46E-FP
M46E-FP assigns a prefix dedicated to M46A to the IPv6 prefix of M46A.<br>
By fixing the prefix of M46A, the plane-ID to which the IPv4 network belongs and the IPv4 subnet are determined, and then the M46A address is determined. That is, the only settings required for the M46E-FP are the M46A prefix, its own plane-ID, and IPv4 subnet information. The communication partner M46A can be automatically generated from the destination IPv4 address. Then, the route information to the communication partner is transmitted by the routing protocol. The routing protocol is optional and can be used with OSPFv3, IS-IS, or BGP. Whether realistic or not, it should work with static routing as well.<br>

There is an advantage that the number of settings is small, but the total number of routes does not change because it is necessary to map IPv4 routes to IPv6 addresses and advertise the routes.<br>

M46E-FP is an abbreviation for Multiple IPv4 --IPv6 address mapping encapsulation --fixed prefix.<br>

### M46E-PR
M46E-PR is premised on not advertising IPv6 routes, and by holding the IPv6 prefix of the IPv6 network to which the IPv4 subnet is connected in the table, the IPv6 prefix to which the destination IPv4 address is connected is resolved and M46A Is what you want. Assuming that IPv6 is available, the image is that IPv4 communication will be added to the route advertisement that realizes IPv6 communication.<br>

Since the route advertisement of M46A is not performed, it can be realized by the end user initiative, but it is necessary to manage the correspondence between the IPv4 subnet and the prefix of the IPv6 address to which the subnet is connected in the table. However, since this correspondence is the same information for the users of that plane, I think it is possible to create simplification of settings.<br>

The M46E-PR has two modes, a host mode that operates as an IPv4 host and a router mode that operates as an IPv4 router.<br>

Host mode assumes that the stub network is one subnet, and the host receives packets by Neighbor Discovery's Neighbor Advertisement, or IPv4 arp resolution. For IPv4 communication by M46E-PR, when the address of M46A corresponding to the IPv4 host is resolved, M46E-PR makes a proxy response and receives the IPv6 packet addressed to the corresponding M46A. After receiving, remove the IPv6 header, decapsulate, take out the IPv4 packet and send it to the IPv4 host.<br>

M46E = PR is an abbreviation for Multiple IPv4 --IPv6 address mapping encapsulation --prefix resolution.<br>

### M46E-PT
M46E-PT is a technology that interconnects the network composed of M46E-FP and the network composed of M46E-PR, and converts M46A. Only the part corresponding to the prefix is ​​translated, and the plane-id and IPv4 address are not translated.<br>

M46E-PT is an abbreviation for Multiple IPv4 --IPv6 address mapping encapsulation --prefix translator.<br>

## Difference from tunnel
A well-known technique for encapsulation is the tunnel. A tunnel is a virtual point-to-point link and can be said to be a technology for creating L2 links.<br>
On the other hand, M46E is a technology that maps IPv4 addresses to the IPv6 address space, and can map not only host addresses but also subnets. In other words, IPv4 communication is performed in the IPv6 address space, and IPv4 communication is performed using the IPv6 address.<br>

The tunnel and M46E are similar in terms of encapsulation, but with some differences.<br>

For example, in the case of routing, in the case of a tunnel, the tunnel is effectively a data link, so it is necessary to operate a routing protocol on the data link. On the other hand, in M46E, route control for the mapped address is required, but this can be operated in the underlay, not in the overlay.<br>

Also, in the case of a tunnel, the tunnel endpoint has to be fixed in one place, and it is not easy to have a redundant configuration. On the other hand, with the M46E, it is possible to configure multiple encapsulation / decapsulation points, for example, to select points by route control.<br>

In the case of tunnels, it is possible to configure recursive tunnels, but in the case of M46E, recursive encapsulation is not performed. Originally, recursive tunnels are not desirable and can cause troubles, so I think that the possibility of troubles can be reduced.<br>

## What is M46T
M46T is an IPv4-IPv6 header conversion technology using M46A.<br>
Speaking of IPv4-IPv6 header conversion technology, NAT64 + DNS64 is famous, but the major difference is that M46T is based on the address mapping rules of M46A. When we came up with the M46T, we limited the IPv4 server to a form that can be accessed from an IPv6 client. As a result, it is possible to access the IPv4 server with private address allocation by IPv6, and it is possible to directly type the IPv6 address, that is, to access without DNS linkage.<br>

M46T can also be used in combination with M46E.<br>

M46T is an abbreviation for Multiple IPv4-IPv6 address mapping translator.<br>

## About M4P6E
M4P6E is an IPv4 over IPv6 encapsulation technology that supports sharing of IPv4 addresses. This is a technology that maps port numbers to IPv6 addresses in addition to IPv4 addresses. The idea is the same as M46E, but by including the port number, one IPv4 address can be shared by multiple hosts.<br>

## About SA46T
The M46 was formerly known as the SA46T, and most proposals and activities were flourishing around the time the name SA46T was used.<br>

SA46T is an abbreviation for Stateless automatic IPv4 over IPv6 tunneling, and I gave it a name that expresses its technical features in a straightforward manner, but to my embarrassment, I gave it the name tunneling. Since I was doing various activities using the name SA46T, I thought that some people knew the name SA46T, but since the essence of the technology is address mapping, not a tunnel, I took the plunge and changed the name. I did. The following shows the correspondence between the old and new names.<br>

 | new | old |
 ----|----
 | M46A | SA46T |
 | M46E-FP | SA46T |
 | M46E-PR | SA46T-PR |
 | M46E-PT | SA46T-PT |
 | M46T | SA46T-AT |
 | M4P6E | SA46T-AS |
 
The first proposal was the SA46T only, but then the idea spread. There was also a motive that it would be easier to understand if all the ideas were organized and renamed. As a result of the name change, it may be confusing, but there was such a background and background.<br>


