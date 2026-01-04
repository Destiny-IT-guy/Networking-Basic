# Ether channel

What is it? Etherchannel allow us to increase bandwidth between 2 networking device by connecting multiple ports to each other to act as one singular connection.
They are 2 different ethechannel protocol. First there is the cisco proprietary protocol Port aggregation protocol **PAgp** and the link aggregation protocol **LACP**. I will show you how to use LACP. 

## benefits

| benefits                | explanation |
|-------------------------|-------------|
|Increase bandwidth | Allow you to combined many links to a more powerful one|
| Redundancy| If a port in the groups falls the other continue working |
|Faster configuration | Only need to configure the group and t applies to all ports in the group| 

## active vs passive

Each ether channel group you create can have up to 16 physical but 8 can by active at a time. 

| type                   | explanation                   |
|------------------------|-------------------------------|
|active | initiate and sends lacp packet to negotiate with other side |
|passive | no initiation only respond |

| ports on swtich1 | ports on switch2 | result   |
|------------------|------------------|----------|
|active | active | channel created |
|active | passive | channel created |
|passive | active | channel created |
|passive | passive | no channel created |

**example :** if i have configure a 16 port channel on switch 2 to passive and 8 ports channel on switch 1 to active 8 ports on channel 2 will forward data. the other 8 ports will be in standby mode if any of the forwarding goes down.

## Requirements  

To be able to create a channel both sides and all interfaces/ports must have the following<br><br> **1.** all must be on the same duplex. all half-duplex or all full-duplex <br> **2** all must have the same port speed. ex: all 100 mbps or all 1 gps <br> **3** Same switchport mode and vlan

how to change speed and duplex on ports
```
interface g0/1
duplex auto
duplex half
duplex full
speed 1000
speed 100
speed 10
```

speed command is mbps so if you want 1gbps you will write `speed 1000`.
To verify a interface speed and duplex use the command `show interface gigabitethernet...`

## Configuration

let say i want to upgrade my bandwitch between a trunk port on 2 switch from 1gbps to 8gbps and make them full duplex with no passive ports

switch 1
```
conf t
int range g1/0/1-8
duplex full
speed 1000
channel-group 1 mode active
int port-channel 1
switchport mode trunk
switchport trunk native vlan 100
switchport trunk allowed vlan  10,20,30,40,50
no shut
```

switch2
```
conf t
int range g1/0/1-8
duplex full
speed 1000
channel-group 1 mode active
int port-channel 1
switchport mode trunk
switchport trunk native vlan 100
switchport trunk allowed vlan  10,20,30,40,50
no shut
```
Do not configure speed and duplex on port channel. The port channel number doesn't have to be the same on each switch and they can also be configured on routers.