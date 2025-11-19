# DHCP SNOOPING
As you know we can create a dhcp service on cisco device but there is a better way to do it. Use a external dedicated DHCP server on windows or linux, i might do a guide on how to make a dhcp/dns serveron linux. When clien use dhcp is start a 4 step process to communicate with any available dhcp server to receive a valid IP address. If bad actors set up their own dhcp server on your network they might

There are 2 essential job that dhcp snooping does.<br>
**1-** It block dhcp server message from untrusted ports.<br>
**2-** it limits how many dhcp request a untrusted port can do 

## Trusted vs Untrusted 
trusted ports or interfaces are ports that end deviced need to go thru to arrive at the dhcp server. Router dhcp server or dedicated dhcp server. DHCP is very useful while using vlan environment. <br><br> Also let say you are using routing protocols to make your network redundant you should make multiple path trusted so if one interface fall you can still reach the server but it isnt necessary to make all path trusted. let say 4 of your path fall you might have a bigger problem them reciving IP address.

## dhcp snooping limit rate 
as the name says this feature allow a switch to limit how many dhcp packet request a untrusted port can send out a second. this is very useful when someone is trying to steal all available ip address from dhcp pool. if a port exceed your limit the port shutdown and goes to err-disabled like some port-security violation.


## Configuration
also before we start to make dhcp snooping work in a network with multiple vlan you need to select which vlan will you apply. If im using a trunk port with 4 vlan but only 2 vlan activated vlan snooping only those 2 will be affected by dhcp snooping. So if i put a rate limit only those 2 vlan will be affected by that limit. Tho if the port goes in err disable it will affect everyone.

first of all let us enable dhcp snooping
```
configure terminal
ip dhcp snooping
```
this will enable dhcp snooping but wont affect affect any port that has vlan on them(trunk or access).<br><br>
The following command will enable dhcp snooping on any interface that hasany vlan in the list
```
configure terminal
ip dhcp snooping vlan x,y,z
```
By default any interface that has these vlan will have those interfaces turn untrusted. to make a interfce trusted you the following command
```
interface gigabitethernet0/1
ip dhcp snooping trust
```

Now let say you want to put a limit on how many dhcp discover or offer he can do in a second. You should use the following command.
```
interface fastethernet0/1
ip dhcp snooping limit rate x
```
the x is packet/s.<br>
**Note-** to stop any of the command write **no** before the command. 

## practice
in this same folder I will leave a packet tracer file with a small set up. I set up dhcp snooping on the level 2 and 3 switch while the router has the dhcp server role. You can try and make a interface untrusted and see if the 2 pc still get an IP address.