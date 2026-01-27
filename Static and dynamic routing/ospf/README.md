# OSPF

Open shortest past first (OSPF) is a commonly used dynamic routing protocol. It is a link state routing protocol, which means every router shares information about their links to other router. OSPF use the <b>Dijkstra algorithm </b> to calculate the shortest path to the destination. Every router create the same map of the network. Every router run the SPF algorithm to determine the best path through a network.

How ospf works.

1- Establish neighbor relationships. Routers discover other OSPF routers on the same network and form adjacencies. </br>
2- Exchange LSAs. Routers share information about their interfaces, links, and networks with their neighbors </br>
3- Build the LSDB. Each router collects all LSAs from the area to create a complete map of the network topology. </br>
4- Execute the SPF. Using the LSDB, the router calculates the shortest paths to all networks in the area. </br>
5- Choose the best routes. The router adds the lowest-cost paths to its routing table, which it will use to forward traffic.

## database

Before exchanging information, routers has to establish neighbors with other routers in network. They exchange information such as router IDs, area id, packets and more.
To function efficiently, ospf use various databases (3):

### adjacency database (Neighbor table)

This table holds information about all neighbors which ospf establish a connection with and with whom it will exchange information with.

These 2 following command will show information about each neighbor.

```
show ip ospf neighbor
show ip ospf neighbor detail
show ip ospf neighbor <neighbor.id>
example
sh0ow ip ospf neighbor 1.1.1.1
```

### Link state database (Topology table)

This table (LSDB) contains the complete ospf network topology for a specific area.This table is built by using Link-state advertisement<b> LSA</b> from other neighbor in the same area. If a router has multiple area it will have multiple LSDB. This table data is used to run the spf algorithm. All router within the identical area have identical LSDB after synchronization.


```
show ip ospf database
show ip ospf database network
show ip ospf database router <router.id>
```

When you run the basic commands you will see 2 different link state.

<u>The router link state</u>: every router in the same area will create a type 1 LSA. This LSA describes the router interfaces, link cost and neighbor relationship in the area. 

|row|name|explanation|
|---|----|-----------|
| 1 | Link ID | Router ID of the router being described |
| 2 | ADV Router | Router ID that originated the LSA. often same as link ID. |
| 3 | age | Time since LSA was generated in seconds |
| 4 | seq# | sequence number of the LSA version. Every time a router changes its LSA(a ospf enabled interface (even passive int) goes up or down the seq# will go up by 1.) This can be useful to determine if a router is talking way more then others |
| 5 | checksum | A fingerprint to verify integrity of the LSA |
| 6 | Link count | Number of ospf enable interface on the router. Active and passive interface. |

<u> The net link state </u> is a type 2 LSA. The DR device is the only one allowed to modify this link state. IT describe the multi-access network and all routers connected to it, which allows ospf to know which router share broadcast segment.

The link ID section defines IP address of interfaces of the DR router. ADV  router show the router ID of the DR.

The <b>network</b> option shows only the net link state with additional information. The <b>router</b> option  will show you the router link state with additional detail. 

### Forwarding database (Routing table)

After a router finishes it area LSDB and run the spf algorithm it create the ospf Routing table. In this table the router chooses the best path to reach every network included in OSPF. the best path is the one with the lowest cost. The ospf cost for an interface is calculated by using the formula:
`Cost = reference bandwidth/interface bandwidth.` . Any cost under 1 like 01 will be rounded up to 1. If i need to go thru 2 interfaces one with a cost of 1 and other with cost of 11 my total cost will be 11. By default the reference bandwidth will be 100 Mbps. to change it use the following command

```
router ospf <process id>
 auto-cost reference-bandwidth <value in Mbps.>
 auto-cost reference-bandwidth 1000
```

This is local to each device so you will have to enter it into every ospf enable device.

The following command will show all ospf routes.

```
show ip route ospf
```

## Types of packets

OSPF devices communicate with each other using packets. There are a total of 5 types of packets serving a specific purpose. I will make a table to explain them.

| Packets type | Purpose |
|--------------|---------|
| hello packet | Routers send these every 10 seconds to find neighbor and keep connection up. These are also used for The DR and BDR election process |
| Database description (DBD) | Contains abreviated list od LSDB. during adjcency router sends dbd. They are used to check if their LSA are out of date |
| 










# The end

types of packets

reason for DR and br and other categories

ospf area and types

how to configure ospf

passive interfaces 