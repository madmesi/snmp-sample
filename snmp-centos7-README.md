Instructions on How to install snmp on Centos7

Install SNMP on server:

```
yum -y install net-snmp net-snmp-utils
# move the original file
mv /etc/snmp/snmpd.conf /etc/snmp/snmpd.conf.org
```

```
vim /etc/snmp/snmpd.conf
# Agent address
agentaddress    udp:161
agentaddress    udp6:161

## Access control
# @@ Firstly, Map the community into a security name
#           sec.name            source                  community
# com2sec    <sec.name>          <monitor_server>        <community_password>
# com2sec6   <sec.name>          <monitor_server_ipv6>   <community_password>

# replace the ip of the ubuntu server, that we created in previous README
# if you want the ubuntu server be the only snmpd server in the vlan do this.
# com2sec     AllowSpecific       192.168.109.136           MyCOMMUNITY!
# com2sec     AllowAll            192.168.109.136           MyCOMMUNITY!

# if you want the centos server be like the other servers in the vlan to send and receive traps:
com2sec     AllowSpecific       default           MyCOMMUNITY!
com2sec     AllowAll            default           MyCOMMUNITY!

# @@ Secondly, Map the security name into a group
# group.name sec.model               sec.name
#group      <group_name>        <security_mode>         <security_name>
group       SpecificGroup       v2c                     AllowSpecific
group       AllGroup            v2c                     AllowAll

# @@ Thirdly, Create a view to let group have rights to:
# @@ Open up the whole tree for ro, make the RFC 1213 required ones rw.
# Define 'SystemView', which includes everything under .1.3.6.1.2.1.1 (or .1.3.6.1.2.1.25.1)
# Define 'AllView', which includes everything under .1
#           view.name           incl/excl               subtree.mask(Optional)
view        SystemView          included                .1.3.6.1.2.1.1
view        SystemView          included                .1.3.6.1.2.1.25.1.1
view        AllView             included                .1

# @@ Finally, Grant right to group
# Give 'SpecificGroup' read access to objects in the view 'SystemView'
# Give 'AllGroup' read access to objects in the view 'AllView'
#           group.name      context model   level   prefix  read        write   notify
access      SpecificGroup   ""      any     noauth  exact   SystemView  none    none
access      AllGroup        ""      any     noauth  exact   AllView     none    none

## System contact information
# syslocation    <location set>
# syscontact     <contact_info>
syslocation     VASL INFRA, Tehran, IR
syscontact      Mohammad Meskarian, Email:mohammadmeskarian@gmail.com, Cellphone: 09125984305
```
```
systemctl enable --now snmpd
systemctl status snmpd
```

Also remember to allow snmpd on firewall.

walk:
```
# on localhost
snmpwalk -v 2c -c MyCOMMUNITY! -O e 127.0.0.1
# other servers in the network (like the ubuntu server):
snmpwalk -v 2c -c MyCOMMUNITY! -O e 192.168.109.136
```



