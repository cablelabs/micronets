---
title: "Example /etc/network/interfaces"
weight: 20
---

```
# Note: The contained entries should augment the system-provided /etc/network/interfaces file.
#       IOW, this file should not over-write the existing /etc/network/interfaces on 
#       your system - the entries should be added.

#
# Define an OpenVSwitch bridge
#
auto brmn001
allow-ovs brmn001
iface brmn001 inet dhcp
  ovs_type OVSBridge
  # This is the port that's connected to the Internet
  ovs_bridge_uplink_port enp3s0
  # the ovs_ports should list all wired and wireless interfaces under control of Mironets
  ovs_ports enp3s0 wlp2s0 enxac7f3ee61832 enx00e04c534458
  ovs_protocols OpenFlow10,OpenFlow11,OpenFlow12,OpenFlow13

# Assign IP addresses to the bridge that may be configured as Micronets
# Note: This will not be necessary once L3 routing is performed via OVS

iface brmn001 inet static
  address 192.168.250.1
  netmask 255.255.255.0

iface brmn001 inet static
  address 192.168.251.1
  netmask 255.255.255.0

iface brmn001 inet static
  address 192.168.252.1
  netmask 255.255.255.0

# Attach physical ports to the bridge (requesting particular port numbers)

# This entry would represent the trunk/uplink interface (the interface with 
# a configured gateway). This can be configured with a fixed IP or DHCP, as
# documented in the interfaces man page
allow-brmn001 enp3s0
iface enp3s0 inet manual
  ovs_type OVSPort
  ovs_bridge brmn001
  ovs_port_req 2
  ovs_port_initial_state enabled

# A wireless interface (managed by hostapd)
allow-brmn001 wlp2s0
iface wlp2s0 inet manual
  ovs_type OVSPort
  # The ovs_bridge must match the bridge definition (above)
  ovs_bridge brmn001
  # The port number needs to be unique for the bridge
  ovs_port_req 3
  # Indicates that the port is blocked at startup (until enabled via command)
  ovs_port_initial_state blocked

# A wired interface
allow-brmn001 enxac7f3ee61832
iface enxac7f3ee61832 inet manual
  ovs_type OVSPort
  ovs_bridge brmn001
  ovs_port_req 4
  ovs_port_initial_state blocked

# A second wired interface
allow-brmn001 enx00e04c534458
iface enx00e04c534458 inet manual
  ovs_type OVSPort
  ovs_bridge brmn001
  ovs_port_req 5
  ovs_port_initial_state blocked
```