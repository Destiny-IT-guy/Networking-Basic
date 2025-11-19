# basic device configuration
In the following minutes i will show you basic configuration that you should put in all your switches or routers. ill go down the line for commands you shoud put in both then i will seperate the distinc commands.

## 3 modes
In basic networking device there are 3 main modes. i will show a table to help you understand the difference.<br>
| Mode Name              | What you will see         | How to Enter mode              | What You Can Do                                | Brief Explanation |
|------------------------|-------------------------|----------------------------|------------------------------------------------|--------------------|
| User EXEC Mode         | Switch>                 | Power on / login           | Basic commands: `ping`, `show ip int br` (limited) | "View only" mode. Safe. Limited access. |
| Privileged EXEC Mode   | Switch#                 | `enable`                   | Advanced show/debug commands, copy files         | "Admin view" mode. Full read access. |
| Global Config Mode     | Switch(config)#         | `configure terminal` / `conf t` | Change device settings (VLANs, interfaces, passwords, etc.) | "Make changes" mode. You edit the running config. |

Global configuration mode has a lot under it which has some different name but you can call them submodes. they allow you to configure a specific feature like interfaces, vlan, protocol and more.

## device name
In a big network it is important to be able to differentiate device while you are connected to them so it is essential to give a device a unique name. to do this use the following command.
```
(config)# hostname router.1
```
Cisco Names you define cannot have **spaces** in it. So if i did `router 1` wouldn't work and i would receive a error message.

## Privileged exec mode password
To secure your network it is always good to require a password to enter privileged exec mode. There are 2 ways to do it you can use the command<br>
```
(config)# enable password Your_password
```
This isnt the best way cause the password will be written in clear text in your configuration. The better way to do it would be the following
```
(config)# enable secret Your_password
```
This will encrypt your password so nobody will be able to read it 

## Line password
Before even entering privilege exec mode user can enter user exec mode in two ways. using console line `console port on device` or using vty line `Use to connect remotely`. I think there is another way via aux port that were more used in the pass but might still be used on some routers. As you can assume it would be smart to require user to enter a password to enter user exec mode. The following command will require user to enter a password to enter that mode 
```
(config)# line cons 0
(config-line)#password strong-password
(config-line)#login
```
the **login** command force you to enter the password to gain access to user exec.

I will show the vty line password commands a little later because they require us to configure another featuure first.

## Banner messages
When configurating a devices it is always recomended to connfigure banner message for security purposes or network annoucement like maintenance. enter the following command to accomplish it 
```
(config)#banner motd #
Enter your banner message here 
you can use multiple lines if you want
#
```
You can use any delimiter you want. You could use !,$,% and more but i prefer to use #.

## ip domain name
configuring domain name is needed to generate ssh key and can also be very useful if you want to create a self signed certificate or import a real certificate from a CN for ***TLS*** purposes. 
if you have a domain name name `local.com` enter the following command.
```
(config)# ip domain-name local.com
```
`Ip domain-name lookup` is enable by default on most cisco device. This could be annoying if you are in a test environnement with no dns server. This might cause the device to try and resolve any unrecognize words type in the command line. This could be very annoying so i suggest you use `no ip domain-name lookup`.<br>

## crypto key 
To establish remote connection **ssh** we have to configure a crypto key. the following command will allow you to creata a ssh key. it is recomended to use a minimum of 2048 bits
```
(config)# crypto key generate rsa general-keys modulus 4096
```
The key name will be a mix of hostname + domain name. so my key name will be `router.1.local.com`. You can see the key by using the following command `show crypto key mypubkey rsa`.

### SSH version
After creating a key you have to select a ssh version. on cisco devices there are 2 ssh version, version 1 and 2. Version one is less secure, offer a weaker encryption and is obsolete so always chose version 2. to activate version 2 use the following
```
(config)# ip ssh version 2
```
## Vty lines password
When connection to vty line via ssh it is always recomended to require user to enter a username and password to ensure that only authorized users can access the device and to provide accountability for administrative actions. These command will set up a basic vty line.
```
(config)# username Yoursername privilege x secret Yourpassword
(config-line)# line vty 0 15
(config-line)# login local
(config-line)# transport input ssh
```
The x is a number between for 0 and 15. 0 is for lowest privileges and 15 is access to all commands.<br> `login local` make the device uses its database of local user for authentification.<br> `transport input ssh` force system to only use ssh connection. I could replace ssh by telnet to accept only telnet connection or all to allow both. I can also replace it by none to allow neither.

## time out
For security purpose users should be kicked out of the session after a certain amount of time of inactivity. By default devices don't implement this so you have to add it. Do the following
```
(config)# line vty 0 15
(config-line)# exec-timeout 8 30
(config-line)# line console 0
(config-line)# exec-timeout 8 30
```
This will kick users out of their session after 8 minutes and 30 seconds of inactivity.

## login block
Let say a attacker is trying to connect to your network. he doesnt know your users or password so he is just trying multiple different combination. after a while you will want to block him from tryin again. the next command will help you do just that.
```
(config)# login block-for 240 attempts 3 within 60
```
This command will block the user from login in for 240 sec after 3 failed attempts in a spand of 60 seconds.<br>I am not sure if this is true but people recomend to enter this command before any other login command or it might not work.

## Password encryption
Let say you used the password command that doesnt encrypt your password in the system it is not the end of the world. There is a command that encrypt every password on the system. Use 
```
(config)# service password-encryption 
```
## saving configuration
After entering all these configuration they are in what we call running-configuration. when we shut down device or reboot we lose all these configuration. To solve this we will put these configuration in our nvram(memory that stays after a device is powerd off). We are going to put this file in our startup configuration. Use the following command to realize this
```
# copy running-config startup-config
```
you can also use the command `write` that those the exact same thing but it is considered depricated

## show configuration
Now let say you want to examin you configuration. You can use the following
```
# show running config
```
to see your entire configuration. you can also use options afer configs to specify what you want to see. `show running config | ...`. here are 3 option you can use.
| Filter Option                  | What It Does                                                       | Example |
|--------------------------------|------------------------------------------------------------------|---------|
| `\| include <text>`             | Shows only lines that **contain** the specified text             | `show running-config \| include hostname` → only lines with "hostname" |
| `\| section <text>`             | Shows the **section** starting with the matching text     | `show running-config \| section line` → shows all line config sections |
| `\| begin <text>`               | Shows all lines starting **from the first match**               | `show running-config \| begin interface` → displays config from first interface onward |

Text after the filter is case sensitive so if the woord beign with a uppercase i have to use it. example `show running config | include FastEthernet`.<br>Another interesting command to use is `show startup configuration`. You can use this to compare to your running configuration.
## delete configuraiton quickly
Let say you already have a startup config you did some modification but it messed up everything. dont panic you can use following commands
```
# reload
```
this will allow you to reboot your device to your startup config.<br> Let say you want to wip your device clean of your running configuration and startup configuration. It it just as easy. Enter these folloing command.
```
# erase startup-config 
# reload
```
this will delete you configuration file and reboot the device.

## do
I assume you all know but if you are a beginner you might not know you can enter almost any priviliged exec mod command in global configuration mode but you have to add the prefix `do` before any command. example 
```
# show running config
# conf t
(config)# do show running config
```


## Specific to switch
I will brifeley explain what the following commands do. To be able to remotely connect a device it needs a ip. the 2-3 commands will add a ip to the switch and forward any traffic that has to go outside its local subnet.
```
(config)# ip default gateway 192.168.0.1
(config)#int vlan1
(config-if)#ip add 192.168.0.2 255.255.255.0
no shut
```

## Specific to router
I wont go in to too much detail but i will show you how to add a ip to a router interface 
```
(config-if)#int g0/0/1
(config-if)#ip add 192.168.0.1 255.255.255.0
(config-if)#no shut
```
The `no shut` command will activate that interface


## configuration example 
### Switch
```
enable
conf t
hostname Switch1
banner motd #You do not have access to my network#
login block-for 240 attempts 3 within 60
ip default-gateway 192.168.0.1
enable secret 123456
line cons 0
password password1234
login
exec-timeout 15 0
no ip domain lookup
ip domain-name bob.org
crypto key generate rsa general-keys modulus 2048
ip ssh version 2
username admin privilege 15 secret veryintensepassword
line vty 0 15
login local
transport input ssh
exec-timeout 15 0
exit
int vlan 1
ip add 192.168.0.1 255.255.255.0
no shut
exit
service password-encryption
do copy running-config startup-config
```

### Router 
```
enable
conf t
hostname Router1
banner motd #You don't have access to my netowrk#
login block-for 240 attempts 3 within 60
enable secret 123456
line cons 0
password password1234
login
exec-timeout 15 0
no ip domain lookup
ip domain-name bob.org
crypto key generate rsa general-keys modulus 2048
ip ssh version 2
username admin privilege 15 secret veryintensepassword
line vty 0 15
login local
transport input ssh
exec-timeout 15 0
exit
int g0/1
ip add 192.168.0.1 255.255.255.0
no shut
exit
service password-encryption
do copy running-config startup-config
```