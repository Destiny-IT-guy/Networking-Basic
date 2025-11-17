# Password
As you know we can configure password in cisco device to protect your system. by default there is no password parameters so you can create password like **1234**. This coud be a security risk so it is always recommended to put harsher requirement. Before i start it is also possible to use a external server like tacacs to take care of password enforcement. I might show this on a later date when i get more equipement because packet tracer is very limited.

## Min lenght
By default the min lenght of cisco devices is very low so you can create password with a couple of characters. to set your own minimun lenght of characters use the following command.
```
configure terminal
security passwords min-length <number>
```
This will force any new password to follow the minimum lenght of characters.

## Password strenght check
to prevent you from creating weak password you can use the following command.
```
conf t
password strength-check
exit
show password strength-check
```
this will enable harsher requiremnt for password and also show the requirement.


# Other devices 
Some buisness switched with difference ios allows a deeper level of password configuraiton parameters. I will not go in detail but here is a command that you might see.

```
enable password [level privilege-level] [unencrypted-password | encrypted encrypted-password]
```