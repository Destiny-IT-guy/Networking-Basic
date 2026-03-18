# Static routes

Routing is a process that happens on the layer 3 of ths OSI model. Routing is the process of selecting a path to arrive at an IP address across a network.There exist two types of routing. Dynamic routing is where devices learns routes to multiple networks dynamically from other equipments. This is very useful for large networks with multiple different subnets and devices. There exist many dynamic routing protocol but some popular are `OSPF`,`EIGRP` and `BGP`. For smaller networks I would recommend to learn and use static routes.

static routing is the process of manually configuring routes to network on each devices. This can be done on network equipment or end device like computers and servers but today I will stick to network equipment. There are 3 essentials requirement to make a static route.

| name | description |
|------|-------------|
| destination address | This can be a whole network or a specific ip host |
| destination prefix | This is what that separate specific host from networks |
| Next hop | This tell the system where to go. It can be a system interface or a another device IP address |

## Administrative distance

Administrative distance (AD) is what determines what is the best routes for each destination. AD uses 8 bits from 0 to 255 to determine the best routes. The lowest values is the one chosen. each routing protocol default. her is a table of some of them.

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

This is the default value of routes. There are ways to change AD values of dynamic routing protocol and static routes that i will show later for static and ospf.

## Metric and cost

In dynamic protocol there might be multiple path to arrive to the same destination. All path will be advertise but how does the system choose which one is the best if they all have the same AD? Metrics is what they used to decide which route to use. Here is a basic list of what metric dynamic routing protocols uses.

| name | metric name | metric explanation |
|------|-------------|--------------------|
| OSPF | cost | use band

## Configuration

Let say you want to connect a Nating router to a level 3 switch with multiple vlan and lvl 2 switches connected to it.

router

```cisco
enable
ip route ....
```