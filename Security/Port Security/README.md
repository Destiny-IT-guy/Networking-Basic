# Port Security
A way to secure a switch port is by using port security. In the following min i will explain how to configure basic port security on a switch to secure your network.

## Configurating portfast/bpduguard

The following command goes on ports connected to end device like computer, server and can be put on router. they are not linked to port security but allows end device to connect immediately to the switch while skipping the stp protocol and protect the port if i connect a switch by mistake.

```
int g0/1
switchport mode access
spanning-tree portfast
spanning-tree bpduguard enable
```

## Configuring Port security
first of all to enable port security on a port use the following command 
```
switchport port-security
```
The next command will allow to select how many mac address can be stored on that port. Usefull if you do virtualization and need multiple mac on a port.
```
switchport port-security maximum 2
```
You can also configure a static mac address if you know exactly what device mac will be on that port
```
switchport port-security mac-address 0000.aaaa.bbbb
```
The following command will remove mac address after x min of inactivity. you can also change inactivity for absolute to do it regardless of inactivity. x is in minutes 
```
switchport port-security aging time x
switchport port-security aging type inactivity
```
Let say the number of max surpass the limit you selected it will go in violation. there are 3 ways to deal with a violation.
| Mode     | What happens                                        |
|----------|----------------------------------------------------|
| protect  | Drops unauthorized traffic, no log                 |
| restrict | Drops traffic, logs violation, increments counter |
| shutdown | Puts port in err-disabled (port shuts down completely) |
if you selected shutdown you will need to open back the port. Here is how you write this command.
```
switchport port-security violation resrtict
```
this is an example of me using all of these put together.
```
int f0/4
description PC-40
switchport mode access
switchport access vlan 40
mls qos trust cos
switchport voice vlan 160
spanning-tree portfast
spanning-tree bpduguard enable
switchport port-security
switchport port-security maximum 2
switchport port-security aging time 5
switchport port-security aging type inactivity
switchport port-security violation resrtict
switchport port-security mac-address sticky
no shut
```

# Verificaiton
The following command will allow you to verify you port security command.
```
show running-config | include sticky
show port-security 
show port-security interface g0/1
```