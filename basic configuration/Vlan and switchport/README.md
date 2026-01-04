# Vlan

Vlan(Virtual local area network) are very useful to have multiple different subnet group on a single switch. let say
<br>
Vlan are a 12 bit system so there is a total of 4096 different possibility for vlan but number 0 and 4095 are not usable and number 1002 to 1105 are reserved for a legacy system. Vlan have a lot of upside but they have some downside which why `vxlan` was created. that is a lot more complex so i will explain it another day.

### Example
Let say im a company with 4 different departments, engineering, hr, finance, customer service. If i didn't use vlan I would need 5 different switches, one for each department connecting to the router. that is expensive.
Instead with vlan i can have all different department on the same vlan but each subnet still separated.

## Configuration

When creating vlan it is important to name them. also it is strongly create a natif vlan and a management vlan.<br>
**Vlan do not go in routers**

```
vlan 10
name engineering
vlan 20
name hr
vlan 30
name finance
vlan 40
name Customer_service
vlan 99
name natif vlan
vlan 100
name management_vlan
```

to check your vlan you can use th command `show vlan`.

## Switchport modes

There are plenty of switchport mode but i will explain 4 of the main modes.

| Mode                 | What It Does                                      | Typical Use Case                        |
|----------------------|--------------------------------------------------|----------------------------------------|
| access               | Port carries traffic for a single VLAN only.    | Connecting PCs, printers, servers.     |
| trunk                | Port carries multiple VLANs using 802.1Q tags. | Switch-to-switch, router links, AP uplinks. |
| dynamic desirable    | Actively negotiates trunking with neighbor.     | Lab setups or optional trunk links.    |
| dynamic auto         | Passively waits for neighbor to request trunk. | Default on some switches; rarely used in production. |

Dynamic desirable will send DTP(Dynamic trunking packets) and try to form a trunk link with the other device.<br>
Dynamic auto will also send dtp but will not actively try to form a trunk link. Here is a table representation of this 

| Port 1 \ Port 2      | dynamic desirable | dynamic auto | trunk      | access     |
|----------------------|-----------------|--------------|------------|------------|
| dynamic desirable     | trunk           | trunk        | trunk      | access     |
| dynamic auto          | trunk           | access       | trunk      | access     |

In real network i recommend to not use any dynamic mode and only use access or trunk.

### nonegotiate

the command `switchport nonegotiate` will stop the interface from sending DTP. This is very useful to protect your network from sending trunking information to unauthorized equipments.

### encapsulation dot1q

<b>802.1Q is the standard protocol that tags Ethernet frames with VLAN IDs</b><br>
By default, most new switches only support 802.1Q, so the trunk encapsulation is automatically set.
Older switches support older method as well, so you must use the following command to tell the switch which tagging method to use

```
switchport trunk encapsulation dot1q
```

Vlan tagging only happens if packet are sent to a trunk port. So when traffic enters a access port it remains untagged until it needs to go thru a trunk port. <b>Also all untagged traffic arriving at a trunk port will e tag with the native vlanID.</b>

## Configuration

I will show a set up with 2 switches. 1 with vlan 10,20,30,99,100 the other 30,40,50,99,100
Switch 1

```
vlan 10
nam engineering
vlan 20
name hr
vlan 30 
name it
vlan 99 
name natif
vlan 100
nam management
interface f0/1
description engineering 
switchport mode acces
switchport access vlan 10
no shut
interface f0/2
description hr
switchport mode acces
switchport access vlan 20
no shut
interface range f0/3 - 7
description IT_department
switchport mode acces
switchport access vlan 30
no shut
interface g0/1
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,100
no shut
```

switch 2

```
vlan 30
nam it
vlan 40
name finance
vlan 50 
name R&D
vlan 99 
name natif
vlan 100
nam management
interface f0/1
description finance 
switchport mode acces
switchport access vlan 40
no shut
interface f0/2
description R&D
switchport mode acces
switchport access vlan 50
no shut
interface range f0/3 - 7
description IT_department
switchport mode acces
switchport access vlan 30
no shut
interface g0/1
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 99
switchport trunk allowed vlan 30,40,50,100
no shut
```

These switched are connected to each other by a trunk port on both device interface g0/1.
in this set up a computer connected to a IT interface on switch 1 will be able to communicate with a IT interface on switch 2. no other vlan will be to communicate with another vlan. <br>

## No switchport

Sometimes you will need to add a ip address on a layer 3 switch port instead of their vlan interface. To do this use the following command

```
interface g0/1
no switchport
ip add 192.168.0.x 255.255.255.0
no shut
```
