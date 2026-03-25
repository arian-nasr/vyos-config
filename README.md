***

# VyOS Hypervisor Configuration Guide w/ SR-IOV on Mellanox ConnectX-3
**Arian Nasr**\
**March 19, 2026**
## 1. Configure Hypervisor

### Edit mlx4 configuration

Add the following to `/etc/modprobe.d/mlx4.conf`:

```bash
options mlx4_core port_type_array=2 num_vfs=8 probe_vf=0 log_num_mgm_entry_size=-1
```


### Rebuild initramfs

```bash
update-initramfs -u -k all
```


### Reboot the system

```bash
reboot
```


### Tag virtual function

```bash
ip link set dev enp4s0 vf 0 vlan 40
```


***

## 2. Add PCI Devices to VyOS VM

Configure the network interfaces:

```bash
set interfaces ethernet eth0 description 'WAN'
set interfaces ethernet eth1 description 'LAN'
set interfaces ethernet eth1 address '192.168.0.1/24'
```


***

## 3. Enable SSH

```bash
set service ssh port '22'
```


***

## 4. Configure PPPoE Interface

```bash
set interfaces pppoe pppoe0 description 'Distributel'
set interfaces pppoe pppoe0 source-interface eth0
set interfaces pppoe pppoe0 authentication username <PPPoE username>
set interfaces pppoe pppoe0 authentication password <PPPoE password>
```


***

## 5. Firewall Configuration

### Define groups

```bash
set firewall group interface-group WAN interface pppoe0
set firewall group interface-group LAN interface eth1
set firewall group network-group NET-INSIDE-v4 network '192.168.0.0/24'
```


### Global options

```bash
set firewall global-options state-policy established action accept
set firewall global-options state-policy related action accept
set firewall global-options state-policy invalid action drop
```


***

## 6. IPv4 Firewall Rules

### Outside-In rules

```bash
set firewall ipv4 name OUTSIDE-IN default-action 'drop'
set firewall ipv4 forward filter rule 100 action jump
set firewall ipv4 forward filter rule 100 jump-target OUTSIDE-IN
set firewall ipv4 forward filter rule 100 inbound-interface group WAN
set firewall ipv4 forward filter rule 100 destination group network-group NET-INSIDE-v4
```


### Default input policy

```bash
set firewall ipv4 input filter default-action 'drop'
```


***

## 7. Allow LAN SSH

```bash
set firewall ipv4 name VyOS_MANAGEMENT default-action 'return'
set firewall ipv4 input filter rule 20 action jump
set firewall ipv4 input filter rule 20 jump-target VyOS_MANAGEMENT
set firewall ipv4 input filter rule 20 destination port 22
set firewall ipv4 input filter rule 20 protocol tcp

set firewall ipv4 name VyOS_MANAGEMENT rule 15 action 'accept'
set firewall ipv4 name VyOS_MANAGEMENT rule 15 inbound-interface group 'LAN'
```


***

## 8. Allow LAN Ping

```bash
set firewall ipv4 input filter rule 30 action 'accept'
set firewall ipv4 input filter rule 30 icmp type-name 'echo-request'
set firewall ipv4 input filter rule 30 protocol 'icmp'
set firewall ipv4 input filter rule 30 state new
set firewall ipv4 input filter rule 30 source group network-group NET-INSIDE-v4
```


***

## 9. Allow Localhost Traffic

```bash
set firewall ipv4 input filter rule 50 action 'accept'
set firewall ipv4 input filter rule 50 source address 127.0.0.0/8
```


***

## 10. Apply Configuration

```bash
commit
save
exit
```


***
