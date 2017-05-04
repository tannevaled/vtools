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
```
### Firewall
```
[root@gateway-idrac ~]# firewall-cmd --permanent --zone=external --add-interface=eth0 
[root@gateway-idrac ~]# firewall-cmd --permanent --zone=internal --add-interface=eth1
[root@gateway-idrac ~]# firewall-cmd --complete-reload
[root@gateway-idrac ~]# firewall-cmd --list-all-zones
[root@gateway-idrac ~]# firewall-cmd --permanent --zone=external --add-masquerade
[root@gateway-idrac ~]# firewall-cmd --permanent --direct --passthrough ipv4 -t nat -I POSTROUTING -o eth0 -j MASQUERADE -s $NET_IDRAC
```
## rstation gateway config

### idrac
#### racadm
cf http://linux.dell.com/repo/hardware/DSU_15.01.00/
```
wget -q -O - http://linux.dell.com/repo/hardware/dsu/bootstrap.cgi | bash
yum install dell-system-update
yum install srvadmin-all
```

```
racadm -r %s -u %s -p %s set system.thermalsettings.thirdpartypcifanresponse 0
```
## rstation config
### sysctl
```
[root@rstation-001 ~]# cat /etc/sysctl.d/00-ip_forward.conf
net.ipv4.ip_forward = 1
```
### Network
#### em1.3996 vlan interface
```
[root@rstation-001 ~]# nmcli con add type vlan con-name em1.3996 dev em1 id 3996
[root@rstation-001 ~]# nmcli con mod em1.3996 ipv4.method manual ipv4.addresses $RSTATION_001_IP ipv4.gateway $RSTATION_GATEWAY
[root@rstation-001 ~]# nmcli con up em1.3996

```
#### vstation bridge over em1.3997 vlan interface
```
[root@rstation-001 ~]# nmcli con add type bridge ifname vstation con-name vstation
[root@rstation-001 ~]# nmcli con mod vstation bridge.stp no ipv4.method disabled ipv6.method ignore
[root@rstation-001 ~]# nmcli con add type vlan con-name em1.3997 dev em1 id 3997
[root@rstation-001 ~]# nmcli con mod em1.3997 ipv4.method disabled \
                                              ipv6.method ignore \
                                              connection.master vstation \
                                              connection.slave-type bridge
[root@rstation-001 ~]# nmcli con up em1.3997
[root@rstation-001 ~]# nmcli con up vstation
```
### Firewall

```
[root@gateway-rstation ~]# firewall-cmd --permanent --zone=external --add-interface=eth0 
[root@gateway-rstation ~]# firewall-cmd --permanent --zone=internal --add-interface=eth1
[root@gateway-rstation ~]# firewall-cmd --get-active-zones
internal
  interfaces: eth1
external
  interfaces: eth0
```
```
[root@gateway-rstation ~]# firewall-cmd --permanent --direct --passthrough ipv4 -t nat -I POSTROUTING -o eth0 -j MASQUERADE -s $NET_RSTATION
[root@gateway-rstation ~]# firewall-cmd --permanent --zone=internal --add-service=dns
success
[root@gateway-rstation ~]# firewall-cmd --permanent --zone=internal --add-service=dhcp
success
[root@gateway-rstation ~]# firewall-cmd --reload
success
```
### Services
#### dnsmasq
```
[root@gateway-rstation ~]# cat /etc/dnsmasq.d/rstation.conf
bind-interfaces
except-interface=eth0

domain-needed

dhcp-host=00:11:22:33:44:55,rstation-001.rstation.,10.xx.yy.1
```
