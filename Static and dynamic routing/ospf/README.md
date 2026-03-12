# OSPF

Open shortest past first (OSPF) is a commonly used dynamic routing protocol. It is a link state routing protocol, which means every router shares information about their links to other router. OSPF use the <b>Dijkstra algorithm </b> to calculate the shortest path to the destination. Every router create the same map of the network. Every router run the SPF algorithm to determine the best path through a network.

<b>How ospf works.</b>

1- Establish neighbor relationships. Routers discover other OSPF routers on the same network and form adjacencies. </br>
2- Exchange LSAs. Routers share information about their interfaces, links, and networks with their neighbors </br>
3- Build the LSDB. Each router collects all LSAs from the area to create a complete map of the network topology. </br>
4- Execute the SPF. Using the LSDB, the router calculates the shortest paths to all networks in the area. </br>
5- Choose the best routes. The router adds the lowest-cost paths to its routing table, which it will use to forward traffic.

## databases

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
`Cost = reference bandwidth/interface bandwidth.` . Any cost under 1 like 0.1 will be rounded up to 1. If i need to go thru 2 interfaces one with a cost of 1 and other with cost of 10 my total cost will be 11. By default the reference bandwidth will be 100 Mbps. to change it use the following command

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
| Database description (DBD) | Contains abbreviated list od LSDB. during adjacency router sends dbd. They are used to check if their LSA are out of date |
| Link-state request packet (LSR) | This packet is used to request more information or up to date information from other routers. Usually after finding out his database is out of date with DBD packets. |
| Link state update (LSU) | To respond to a LSR other devices will use LSU packet to carry up to date LSAs to the other devices. This happens usually if a link goes up or down or added some new network to the ospf area |
| Link state acknowledgement (LSack) | After receiving LSA from LSU the receiver send a LSack to ensure the sender that he receive the data. |

## LSA

LSA or link state advertisement is a packet of data that details part of a ospf topology. Different types of LSA types are used by routers to create the topology. They are stored in the LSDB. A Router must get and learn every LSAs of that area.

LSA flooding is an important part of the ospf process. IT is a way to distribute LSAs to all router within the same area. When a toplogy change happens the originating router floods its updated LSA to its ospf neighbors. Other routers can also request new LSA if the notice that their own are missing or out of date.

In ospfv2 (IPV4) There are a total of 11 different lsa type. 8-11 are used for special purposes. The following table will explain lsa 1-7

| Type | Name | Purpose |
|------|------|---------|
| 1 | Router LSA | ... |
| 2 | Network LSA | ... |
| 3 | ABR Summary | ... |
| 4 | ASBR Summary | ... |
| 5 | ASBR External LSA | ... |
| 6 | Multicast OSPF LSA | ... |
| 7 | NSSA External LSA | ... |
| 8 | |

## DR BDR election

To reduce traffic and LSA flooding ospf use a <b>DR</b>(Designated router) and a <b>BDR</b>(backup designated router). All none DR or BDR ospf device are considered as <b>Drother</b>. All Drother exchange information like LSA with the DR and then the DR floods them to all ospf device in the same area.<br><br>
To send information to DR you would use the IP 224.0.0.5 <br> When the DR floods LSA to all devices it use the IP 224.0.0.6

How does ospf select the DR and BDR? There are 2 possible ways to accomplish the goal

<b>1-</b> The ospf devices with the highest priority will be named DR and BDR<br>
<b>2-</b> If ospf device has the same priority the one with the highest router-id will be named DR and BDR

By default the ospf priority will be configured to 1. The lowest possible value will be 0 (never be elected DR) and highest possible value is 255. You configure priority on interface. Her is an example:

``` 
interface g0/1
ip ospf priority 100
```

If no router-id has been configured the IP address loopback interface with highest ip address will be selected as router-id. If no loopback has been created the highest ip address on any interface will be selected as router-id

### adjacency and state

Ospf device need to form adjacencies with all other ospf device within a network to exchange information and to finish the DR election process.

adjacency types

## configuration

There are generally 2 ways to configure ospf. First you can configure it from global configuration mode or you can configure them directly on each interface. I will show you both

lvl 3 switch 1
```
router ospf 1
router-id 1.1.1.1
network 10.10.10.0 0.0.0.255 area 0
network 10.10.11.0 0.0.0.255 area 0
network 10.10.20.0 0.0.1.255 area 0
```

Router

```
router ospf 1
router-id 1.1.1.2 
network 10.10.10.0 0.0.0.255 area 0
network 10.10.11.0 0.0.0.255 area 0
network 10.10.20.0 0.0.1.255 area 0
```

In this i connected a level 3 switch in trunk mode to a port on the router which has 3 sub interface.

This is how you configure ospf on interfaces directly

```
router ospf 1
router-id 1.1.1.1
int g0/1.10
ip add ...
ip ospf 1 area 0 
int g0/1.11
ip add ...
ip ospf 1 area 0
```

As you can see I had to configure ospf on each interface which might take a little bit more time.

### Passive interface

Ospf passive interface allow us to add a network to our ospf database but disabling hello packets from being sent from that interface. Like i previously said when you add a network to ospf all interface that has a ip in the network will start advertising. This might be a waste of resources and a security risk if the other part of that interface isn't connected to a ospf device. Passive interface are useful if a layer 2 switch is connected to a ospf device or you can also try with loopback interfaces.

```
router ospf 1
router-id 1.1.1.1
network 10.10.10.0 0.0.0.255 area 0
network 10.10.11.0 0.0.0.255 area 0
passive-interface g0/1.11
```

as you see i have to specify which interface to not participate in sending hello packets.

# The end

I will add a area section, add more on DR process and finish the adjacency section on a later date. I might add how to configure ospf on pfsense (GUI and shell) in the proxmox repo.
