# ACL Quickly explained

First of all there are 2 type of ACL as you know we will start with the basic

## Where can i apply access list
There are generaly 3 areas where we can apply a acces list. You can put them on **Interfaces**, on **VTY lines or console line** or **Routing protocols like nat**. Interfaces are usually use to control access to the network and block host, lines are usually to Restrict ssh connection to routers and switch and routing protocols are a little bit more complex.

## Standard ACL
standard ACL filter traffic only by the ip address no ports or anything. **Important** It is best to put standard access list near the **<u>destination</u>** because they dont check the destination address or ports which might block necessary traffic.

there are 2 list for standard access list number.
| Access list type | Number Range |
|----------|----------|
| standard list    | 1-99     | 
| extended standard list    | 1300-1999     | 

### Configuring standard acces list

```
access-list <number> <permit/deny> <source> <wildcard>
interface gigabitethernet0/1
ip access-group 1 in
```

**Important** wildcard mask is inverse of normal wildcard so 
if im using /24 255.255.255.0 it will become 0.0.0.255

ok let say i want to allow IP coming from 192.168.0.1-62 but deny anything else i would do the following 
```
access-list 1 permit 192.168.0.0 0.0.0.63
access-list 1 deny any
interface gigabitethernet0/1
ip access-group 1 in
```

let say i want to allow anything but the ip 192.168.0.11 i would do 
```
access-list 1 deny 192.168.0.11 0.0.0.0
access-list 1 permit any
interface gigabitethernet0/1
ip access-group 1 in
```

why permi any or deny any goes at the end? Well ACL goes from top to down so if i write deny any at the start it will ignore all other command i put in the access list same goes fo the permit any.

also you can also replace ACL number by name so i could do name ip access-list standard ALLOW_vlan3 to make it easier to remember.

## configuring Extended ACL 
```
access-list <number> permit|deny <protocol> <source> <wildcard> <destination> <wildcard> [operator]
int g0/1
```
The extended ACL adds 3 important category. first of all there is the **Destination ip address**. This is configured the exact same way the source ip address. than there is the **Port** service. This allow you to select which port to allow or deny. Than there is the operator section which i will show you a table to simplify it 

## Extended ACL Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| eq       | equal to | `eq 22` → only port 22 |
| neq      | not equal | `neq 23` → any port except 23 |
| lt       | less than | `lt 1024` → ports 0–1023 |
| gt       | greater than | `gt 1023` → ports 1024+ |
| range    | range of ports | `range 1000 2000` → ports 1000–2000 |

With all of this i can create a better ACL. 
Now let say i want to deny a specific group of ip from remotely connecting to my router. I would do the following
```
ip access-list extended BLOCK_SSH
 deny tcp host 10.34.57.11 any eq 22
 deny tcp host 10.34.58.22 any eq 22
 deny tcp host 10.37.61.22 any eq 22
 deny tcp host 10.37.61.52 any eq 22
 permit ip any any
 exit

line vty 0 15
 access-class BLOCK_SSH in
 exit
```
This is a small part of the ACL i use to deny all other network equipement from remotely connecting to a specific router.

Let say i want to allow 192.168.0.11 to send http traffic to 5 specific ip but deny him from sending http traffic to other IP i would do the following
```
ip access-list ALLOW_HTTP_TO_ADMIN
 permit tcp host 192.168.0.11 host 10.0.1.141 eq 80
 permit tcp host 192.168.0.11 host 10.0.1.10 eq 80
 permit tcp host 192.168.0.11 host 10.0.3.55 eq 80
 permit tcp host 192.168.0.11 host 10.0.1.66 eq 80
 permit tcp host 192.168.0.11 host 10.0.2.71 eq 80
 deny tcp host 192.168.0.11 any eq 80
 permit ip any any

interface g0/1
 ip access-group ALLOW_HTTP_TO_ADMIN in
```
This would be good if traffic from .11 come into g0/1 before going to other vlan. 
ALC can be put on **Routers** or on **LVL3 Switch**. They cannot be put on **LVL2 Switch** because they forward data based off mac address and not IP address.
You can also filter source port and destination port at the same time if required

## Conclusion
Access list are a good way to scure your ntwork. They allow you to filter traffic based off source ip adress destination ip adress and source ports ports and destination port. **Standard** access list should be put closer to the end device and **Extended** access list should be near the source device and always have a permit ip any any at the end.

In the future i will put my nat guide which i will also use extended Acess list.
