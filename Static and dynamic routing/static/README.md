# Static routes

Routing happens at layer 3 of ths OSI model. Routing is the process of selecting a path to arrive at an IP address across a network.There exist two types of routing. Dynamic routing is when devices learn routes dynamically from other devices. This is very useful for large networks with multiple different subnets and devices. There exist many dynamic routing protocol but some popular are `OSPF`,`EIGRP` and `BGP`. For smaller networks I would recommend to learn and use static routes.

static routing is the process of manually configuring routes to network on each devices. This can be done on network equipment or end device like computers and servers but today I will stick to network equipment. There are 3 essentials requirement to make a static route.

| name | description |
|------|-------------|
| destination address | This can be a whole network or a specific ip host |
| destination prefix | This is what that separate specific host from networks |
| Next hop | This tell the system where to go. It can be a system interface or a another device IP address |

## Administrative distance

Administrative distance (AD) is what determines what is the best routes for each destination. AD uses 8 bits from 0 to 255 to determine the best routes. The lowest values is the one chosen. each routing protocol default. Here is a table of some of them.

| source | Value |
|--------|-------|
| Connected | 0 |
| static | 1 |
| external BGP | 20 |
| EIGRP | 90 |
| OSPF | 110 |
| External EIGRP | 170 |
| Internal BGP | 200 |
| DHCP route | 254 |
| Unknown | 255 |

<b>Maybe explain dhcp and unknown.</b>

This is the default value of routes. There are ways to change AD values of dynamic routing protocol and static routes that I will show later for static and ospf.

## Metric and cost

In dynamic protocol there might be multiple path to arrive to the same destination. All path will be advertise but how does the system choose which one is the best if they all have the same AD? Metrics is what they used to decide which route to use. Here is a basic list of what metric dynamic routing protocols uses.

| name | metric  | metric explanation |
|------|-------------|--------------------|
| OSPF | cost | calculate `Reference bandwidth` / `interface bandwidth` of each interfaces on a path and adds them up. Then it chooses the path with the lowest cost |
| EIGRP | 2 metrics by default can go up to 5 metrics | by default it takes in account 2 metric, bandwidth and delays. But it can also take into account load and reliability.|

## Configuration

Let say you want to connect a Nating router to a level 3 switch with multiple vlan and lvl 2 switches connected to it.

router

```cisco
conf t
int g0/0
ip add 10.10.100.1 255.255.255.0
no shut
enable
ip route 10.10.100.0 255.255.255.0 g0/0
ip route 10.10.10.0 255.255.255.0 g0/0
ip route 10.10.20.0 255.255.255.0 g0/0
```

LVL 3 switch

```ios-xr
enable
ip route 10.10.100.0 255.255.255.0 g0/0
ip route 0.0.0.0 0.0.0.0 10.10.100.1
```

### changing AD

To change AD value of static route you just have to add the AD value you want to the end of you configuration. Here is an example.

```cisco
ip route 10.10.100.0 255.255.255.0 10.10.100.1 5
ip route 10.10.100.0 255.255.255.0 10.10.100.2 10
```

Interface only static routes fails only if the local interface is down.<br>
next hop IP goes down if the destination IP becomes unreachable.


