# vtools


## hypervisor config
### idrac bridge over vlan interface over bond
```
[root@hyper1 ~]# nmcli con add type bridge ifname idrac con-name idrac
[root@hyper1 ~]# nmcli con mod idrac bridge.stp no ipv4.method disabled ipv6.method ignore
[root@hyper1 ~]# nmcli con add type vlan con-name bond0.3995 dev bond0 id 3995
[root@hyper1 ~]# nmcli con mod bond0.3995 ipv4.method disabled ipv6.method ignore connection.master idrac connection.slave-type bridge
[root@hyper1 ~]# nmcli con up bond0.3995
[root@hyper1 ~]# nmcli con up idrac
```
### rstation bridge over vlan interface over bond
```
[root@hyper1 ~]# nmcli con add type bridge ifname rstation con-name rstation
[root@hyper1 ~]# nmcli con mod rstation bridge.stp no ipv4.method disabled ipv6.method ignore
[root@hyper1 ~]# nmcli con add type vlan con-name bond0.3996 dev bond0 id 3996
[root@hyper1 ~]# nmcli con mod bond0.3996 ipv4.method disabled ipv6.method ignore connection.master rstation connection.slave-type bridge
[root@hyper1 ~]# nmcli con up bond0.3996
[root@hyper1 ~]# nmcli con up rstation
```
### vstation bridge over vlan interface over bond
```
[root@hyper1 ~]# nmcli con add type bridge ifname vstation con-name vstation
[root@hyper1 ~]# nmcli con mod vstation bridge.stp no ipv4.method disabled ipv6.method ignore
[root@hyper1 ~]# nmcli con add type vlan con-name bond0.3997 dev bond0 id 3997
[root@hyper1 ~]# nmcli con mod bond0.3997 ipv4.method disabled ipv6.method ignore connection.master vstation connection.slave-type bridge
[root@hyper1 ~]# nmcli con up bond0.3997
[root@hyper1 ~]# nmcli con up vstation
```
## idrac gateway config
```
[root@gateway-idrac ~]# nmcli c m eth1 ipv4.method manual ipv4.addresses $NET_IDRAC_GATEWAY
[root@gateway-idrac ~]# nmcli c up eth1
[root@gateway-idrac ~]# cat /etc/sysctl.d/00-ip_forward.conf
net.ipv4.ip_forward = 1

[root@gateway-idrac ~]# firewall-cmd --zone=external --add-interface=eth0 --permanent
[root@gateway-idrac ~]# firewall-cmd --zone=internal --add-interface=eth1 --permanent
[root@gateway-idrac ~]# firewall-cmd --complete-reload
[root@gateway-idrac ~]# firewall-cmd --list-all-zones
[root@gateway-idrac ~]# firewall-cmd --zone=external --add-masquerade --permanent
[root@gateway-idrac ~]# firewall-cmd --permanent --direct --passthrough ipv4 -t nat -I POSTROUTING -o eth0 -j MASQUERADE -s $NET_IDRAC
```

## idrac
### racadm
cf http://linux.dell.com/repo/hardware/DSU_15.01.00/
```
wget -q -O - http://linux.dell.com/repo/hardware/dsu/bootstrap.cgi | bash
yum install dell-system-update
```
