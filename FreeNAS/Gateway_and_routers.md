[Origin URL](https://www.freebsd.org/doc/handbook/network-routing.html)

## Gateway and Routes
Routing is the mechanism that allows a system to find the network path to another system. A route is a defined pair of addresses which represent the “destination” and a “gateway”. The route indicates that when trying to get to the specified destination, send the packets through the specified gateway. There are three types of destinations: individual hosts, subnets, and “default”. The “default route” is used if no other routes apply. There are also three types of gateways: individual hosts, interfaces, also called links, and Ethernet hardware (MAC) addresses. Known routes are stored in a routing table.

This section provides an overview of routing basics. It then demonstrates how to configure a FreeBSD system as a router and offers some troubleshooting tips.

### Routing Basics
To view the routing table of a FreeBSD system, use [netstat(1)](https://www.freebsd.org/cgi/man.cgi?query=netstat&sektion=1&manpath=freebsd-release-ports):
```
% netstat -r
Routing tables

Internet:
Destination      Gateway            Flags     Refs     Use     Netif Expire
default          outside-gw         UGS        37      418       em0
localhost        localhost          UH          0      181       lo0
test0            0:e0:b5:36:cf:4f   UHLW        5    63288       re0     77
10.20.30.255     link#1             UHLW        1     2421
example.com      link#1             UC          0        0
host1            0:e0:a8:37:8:1e    UHLW        3     4601       lo0
host2            0:e0:a8:37:8:1e    UHLW        0        5       lo0 =>
host2.example.com link#1            UC          0        0
224              link#1             UC          0        0
```

The entries in this example are as follows:


| Entries|	Description |
| ---------| ----------------|
|default|	<p>The first route in this table specifies the default route. When the local system needs to make a connection to a remote host, it checks the routing table to determine if a known path exists. If the remote host matches an entry in the table, the system checks to see if it can connect using the interface specified in that entry.</p><p>If the destination does not match an entry, or if all known paths fail, the system uses the entry for the default route. For hosts on a local area network, the Gateway field in the default route is set to the system which has a direct connection to the Internet. When reading this entry, verify that the Flags column indicates that the gateway is usable (UG). </p><p>The default route for a machine which itself is functioning as the gateway to the outside world will be the gateway machine at the Internet Service Provider (ISP).</p> |
| localhost | The second route is the localhost route. The interface specified in the Netif column for localhost is lo0, also known as the loopback device. This indicates that all traffic for this destination should be internal, rather than sending it out over the network. |
| MAC address | The addresses beginning with 0:e0: are MAC addresses. FreeBSD will automatically identify any hosts, test0 in the example, on the local Ethernet and add a route for that host over the Ethernet interface, re0. This type of route has a timeout, seen in the Expire column, which is used if the host does not respond in a specific amount of time. When this happens, the route to this host will be automatically deleted. These hosts are identified using the Routing Information Protocol (RIP), which calculates routes to local hosts based upon a shortest path determination.|
|subnet	| <p>FreeBSD will automatically add subnet routes forthelocal subnet. In this example, 10.20.30.255 is the broadcast address for the subnet 10.20.30 and example.com is the domain name associated with that subnet. The designation link#1 refers to the first Ethernet card in the machine.</p><p>Local network hosts and local subnets have their routes automatically configured by a daemon called [routed(8)](https://www.freebsd.org/cgi/man.cgi?query=routed&sektion=8&manpath=freebsd-release-ports). If it is not running, only routes which are statically defined by the administrator will exist.</p>
| host | <p>The host1 line refers to the host by its Ethernet address. Since it is the sending host, FreeBSD knows to use the loopback interface (lo0) rather than the Ethernet interface.</p><p>The two host2 lines represent aliases which were created using [ifconfig(8)](https://www.freebsd.org/cgi/man.cgi?query=ifconfig&sektion=8&manpath=freebsd-release-ports). The => symbol after the lo0 interface says that an alias has been set in addition to the loopback address. Such routes only show up on the host that supports the alias and all other hosts on the local network will have a link#1 line for such routes.</p>
|224|The final line (destination subnet 224) deals with multicasting.|

Various attributes of each route can be seen inthe Flags column. Table 30.1, “Commonly Seen Routing Table Flags” summarizes some of these flags and their meanings:


**Table 30.1. Commonly Seen Routing Table Flags**

| Command | Purpose |
| ------------- | ------------ |
|U|	The route is active (up).|
|H|	The route destination is a single host.|
|G|	Send anything for this destination on to this gateway, which will figure out from there where to send it.|
|S|	This route was statically configured.|
|C|	Clones a new route based upon this route for machines to connect to. This type of route is normally used for local networks.|
|W|	The route was auto-configured based upon a local area network (clone) route.|
|L|	Route involves references to Ethernet (link) hardware.|


On a FreeBSD system, the default route can definedin **/etc/rc.conf** by specifying the IP address of the default gateway:
```
defaultrouter="10.20.30.1"
```
It is also possible to manually add the route using route:

```
# route add default 10.20.30.1
```
Note that manually added routes will not survive a reboot. For more information on manual manipulation of network routing tables, refer to [route(8)](https://www.freebsd.org/cgi/man.cgi?query=route&sektion=8&manpath=freebsd-release-ports).

### Configuring a Router with Static Routes
A FreeBSD system can be configured as the default gateway, or router, for a network if it is a dual-homed system. A dual-homed system is a host which resides on at least two different networks. Typically, each network is connected to a separate network interface, though IP aliasing can be used to bind multiple addresses, each on a different subnet, to one physical interface.

In order for the system to forward packets between interfaces, FreeBSD must be configured as a router. Internet standards and good engineering practice prevent the FreeBSD Project from enabling this feature by default, but it can be configured to start at boot by adding this line to **/etc/rc.conf**:
```
gateway_enable="YES"          # Set to YES if this host will be a gateway
```
To enable routing now, set the [sysctl(8)](https://www.freebsd.org/cgi/man.cgi?query=sysctl&sektion=8&manpath=freebsd-release-ports) variable net.inet.ip.forwarding to 1. To stop routing, reset this variable to 0.

The routing table of a router needs additional routes so it knows how to reach other networks. Routes can be eitheraddedmanually using static routes or routes can be automatically learned using a routing protocol. Static routes are appropriate for small networks and this section describes how to add a static routing entry for a small network.

>For large networks, static routes quickly become unscalable. FreeBSD comes with the standard BSD routing daemon routed(8), which provides the routing protocols RIP, versions 1 and 2, and IRDP. Support for the BGP and OSPF routing protocols can be installed using the net/zebra package or port.

Consider the following network:

![](images/static-routes.png)

In this scenario, RouterA is a FreeBSD machine that is acting as a router to the rest of the Internet. It has a default route set to 10.0.0.1 which allows it to connect with the outside world. RouterB is already configured to use 192.168.1.1 as its default gateway.

Before adding any static routes, the routing table on RouterA looks like this:
```
% netstat -nr
Routing tables

Internet:
Destination        Gateway            Flags    Refs      Use  Netif  Expire
default            10.0.0.1           UGS         0    49378    xl0
127.0.0.1          127.0.0.1          UH          0        6    lo0
10.0.0.0/24        link#1             UC          0        0    xl0
192.168.1.0/24     link#2             UC          0        0    xl1
```
With the current routing table, RouterA does not have a route to the 192.168.2.0/24 network. The following command adds the Internal Net 2 network to RouterA's routing table using 192.168.1.2 as the next hop:

```
# route add -net 192.168.2.0/24 192.168.1.2
```
Now, RouterA can reach any host on the 192.168.2.0/24 network. However, the routing information will not persist if the FreeBSD system reboots. If a static route needs to be persistent, add it to */etc/rc.conf*:

```
# Add Internal Net 2 as a persistent static route
static_routes="internalnet2"
route_internalnet2="-net 192.168.2.0/24 192.168.1.2"
```
The *static_routes* configuration variable is a list of strings separated by a space, where each string references a route name. The variable *route_internalnet2* contains the static route for that route name.

Using more than one string in *static_routes* creates multiple static routes. The following shows an example of adding static routes for the *192.168.0.0/24* and *192.168.1.0/24* networks:
```
static_routes="net1 net2"
route_net1="-net 192.168.0.0/24 192.168.0.1"
route_net2="-net 192.168.1.0/24 192.168.1.1"
```

### Troubleshooting
When an address space is assigned to a network, the service provider configures their routing tables so that all traffic for the network will be sent to the link for the site. But how do external sites know to send their packets to the network's ISP?

There is a system that keeps track of all assigned address spaces and defines their point of connection to the Internet backbone, or the main trunk lines that carry Internet traffic across the country and around the world. Each backbone machine has a copy of a master set of tables, which direct traffic for a particular network to a specific backbone carrier, and from there down the chain of service providers until it reaches a particular network.

It is the task of the service provider to advertise to the backbone sites that they are the point of connection, and thus the path inward, for a site. This is known as route propagation.

Sometimes, there is a problem with route propagation and some sites are unable to connect. Perhaps the most useful command for trying to figure out where routing is breaking down is traceroute. It is useful when ping fails.

When using traceroute, include the address of the remote host to connect to. The output will show the gateway hosts along the path of the attempt, eventually either reaching the target host, or terminating because of a lack of connection. For more information, refer to [traceroute(8)](https://www.freebsd.org/cgi/man.cgi?query=traceroute&sektion=8&manpath=freebsd-release-ports).

### Multicast Considerations
FreeBSD natively supports both multicast applications and multicast routing. Multicast applications do not require any special configuration in order to run on FreeBSD. Support for multicast routing requires that the following option be compiled into a custom kernel:
```
options MROUTING
```

The multicast routing daemon, mrouted can be installed using the [net/mrouted](https://www.freebsd.org/cgi/url.cgi?ports/net/mrouted/pkg-descr) package or port. This daemon implements the DVMRP multicast routing protocol and is configured by editing **/usr/local/etc/mrouted.conf** in order to set up the tunnels and DVMRP. The installation of mrouted also installs map-mbone and mrinfo, as well as their associated man pages. Refer to these for configuration examples.

> DVMRP has largely been replaced by the PIM protocol in many multicast installations. Refer to [pim(4)](https://www.freebsd.org/cgi/man.cgi?query=pim&sektion=4&manpath=freebsd-release-ports) for more information.