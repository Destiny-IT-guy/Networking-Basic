# HSRP and VRRP

Hot standby Router protocol (HSRP) and virtual router redundancy protocol (VRRP) allow your network to continue to work even if one router falls. <b> both protocol do also work for level 3 switches.</b> 

## HSRP

hsrp is a cisco proprietary protocol. It allows multiple routers to have the same virtual IP. You will select a different priority for each router and the highest one will be active and all other router will be in standby mode. if router a fails router b will takeover and its virtual IP will become active. Let say router A come back online it will not automatically become active. You have to enter th command ...`preempt`.<br> Also the virtual ip in hsrp has to be different from the router. it will be easier to understand when i show how to configure vrrp.

### configuration

Let say we have 3 router with ip 10.10.10.2, .3, .4 and the virtual ip will be .1 and port g0/2 is the link that connects each router to the WAN

router 1

```
conf t
int g0/1
ip add 10.10.10.2 255.255.255.0
standby 1 10.10.10.1 
standby 1 priority 120
standby 1 preempt
standby 1 track GigabitEthernet0/2
no shut
```

router 2

```
conf t
int g0/1
ip add 10.10.10.3 255.255.255.0
standby 1 10.10.10.1 
standby 1 priority 115
standby 1 preempt
standby 1 track GigabitEthernet0/2
no shut
```

router 3

```
conf t
int g0/1
ip add 10.10.10.4 255.255.255.0
standby 1 10.10.10.1 
standby 1 priority 111
standby 1 preempt
standby 1 track GigabitEthernet0/2
no shut
```

Each router has to be in the same standby group. In my example they are in group 1. There is a total of 256 group you can hav in hsrpv1 ans 4096 in hsrpv2, to change version use this command. `standby version 2` or 1.<br><br>
The track command will decrease are standby priority if the interface is down. the default decrement value is 10. if i want a custom decrement value i would just have to enter a number after the port like `standby 1 track GigabithEthernet0/2 15`. <b>Also if i have multiple track interface the decrement value will be added together</b>. So if 2 interface with default value goes down the priority value will go down by 20.<br><br>

### show commands
```
show standby
show standby brief
```

### hello and hold timers
hello timers determine how often the active router will send hello message to other routers in the same standby group. the default is 3 seconds.<br>
The hold timers determine how much time the stanby router will wait without receiving a hello packet before becoming the active router. <b> If i have more then 2 routers in the same group the highest priority will become the active the second will be in standby mode and all the lower priority will be in a listen state.</b><br><br>

to change timers use the following command
`standby 1 timers 2 5`
the first number is hello timers the second is hold timers.

## VRRP

Virtual router redundancy protocol is an open alternative to HSRP. It uses master and backup state instead of active/standby. VRRP allows multiple routers to act as a single one with a virtual IP and mac address. The routers create a VRRP group then they elect one to be the master and all other are backups. The router with the highest priority becomes master. the default priority value is 100. If multiple router has the same priority the one with the higher IP address will be elected master.

### timers

Vrrp does have timers but are not name hello or hold timers. The equivalent for vrrp are <b>advertisement</b> and <b>down interval</b>. The default values for them are 1 and 3 seconds. You cannot modify down interval in vrrp. IT will always be 3x as long as advertisement interval. Advertisement messages are sent to the multicast address 224.0.0.18. To configure them you can use the following commands.

```
vrrp 1 timers advertise 2
vrrp 1 timers advertise msec 250
```

The `msec` option is to configure time in milliseconds

### differences

Vrrp is very similar to HSRP but there are some differences. Preempt is enable by default for vrrp. The virtual mac address is different between both protocols. The vrrp mac format is the following `0000.5e00.01XX`. The xx are the vrrp group number you selected. Regular vrrp doesn't support ipv6 but VRRPv3 support its.

VRRP uses master/backup states instead of active/standby. preemption is enable by default. the default timers are 1 and 3 seconds instead of 3 and 10. There is no listening state all other router are in backup state.

### Configuration

This is a basic configuration for 3 routers routers running VRRP

router 1

```
conf t
interface GigabitEthernet0/1
ip address 192.168.1.1 255.255.255.0
vrrp 1 ip 192.168.1.1
vrrp 1 preempt
```

router 2

```
conf t
interface GigabitEthernet0/1
ip address 192.168.1.2 255.255.255.0
vrrp 1 ip 192.168.1.1
vrrp 1 priority 110
vrrp 1 preempt
```

router 3

```
conf t
interface GigabitEthernet0/1
ip address 192.168.1.3 255.255.255.0
vrrp 1 ip 192.168.1.1
vrrp 1 priority 110
vrrp 1 preempt
```

As you can see i configured the VRRP IP to a interface IP. This router will automatically have the highest priority possible which is 255, meaning configuring a priority for that interface is useless.

I would consider to use a dynamic routing protocol when using HSRP or VRRP or it will likely cause problems when your master router goes down. I tried to configure static routes with priority to work with vrrp but the problem still persist. I saw we could use `SLA` to bypass this problem but dynamic routing is simpler.

### Track

The track feature also appears in vrrp. It allows router to lower its priority it a tacked interface or <b> object </b> falls.