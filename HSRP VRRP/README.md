# HSRP and VRRP
Hot stanby Router protocol(hsrp) and virtual router redundancy protocol(VRRP) allow your network to continue to work even if one router falls. <b> both protocol do also work for level 3 switches.</b> 

## HSRP
hsrp is a cisco proprietary protocol. It allows multiple routers to have the same virtual IP. You will select a different priority for each router and the higest one will be active and all other router will be in standby mode. if router a fails router b will takeover and its virtual IP will become active. Let say router A come back online it wil not automaticly become active. You have to enter th command ...`preempt`.<br> Also the virtual ip in hsrp has to be different from the router. it will be easier to understand when i show how to configure vrrp.

### configuration
Let say we have 3 router with ip 10.10.10.2, .3, .4 and the virtual ip will be .1 and port g0/2 is the link that connects each routr to the WAN

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
The track command will decrease are standby priorty if the interfac is down. the default decrement value is 10. if i want a custom decrement value i would just have to enter a number after the port like `standby 1 track GigabithEthernet0/2 15`. <b>Also if i have multiple track interface the decrement value will be added together</b>. So if 2 interface with default value goes down the priority value will go down by 20.<br><br>

### show commands
```
show standby
show standby brief
```

### hello and hold timers
hello timers determine how often the active router will send hello message to other routers in the same stanby group. the default is 3 seconds.<br>
The hold timers determine how much time the stanby router will wait without receiving a hello packet before becoming the active router. <b> If i have more then 2 routers in the same group the highest priority will become the active the second will be in stanby mode and all the lower priority will be in a listen state.</b><br><br>

to change timers use the following command
`standby 1 timers 2 5`
the first number is hello timers the seccond is hold timers. 

## VRRP
VRRP uses master/backup states instead of active/standby. preemption is enable by default. the default timers are 1 and 3 seconds instead of 3 and 10. There is no listening state all other router are in backup state.

### configuration
I will finish thhis part when i get my hands on a physical router since packet tracer does not support vrrp configuraiton.