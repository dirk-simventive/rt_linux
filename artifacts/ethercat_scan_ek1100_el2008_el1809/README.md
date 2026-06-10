# EtherCAT Scan: EK1100 + EL2008 + EL1809

## Purpose

This folder contains the first successful EtherCAT hardware scan from the Linux laptop.

It captures the raw EtherLab output needed later to configure ethercat_driver_ros2 and ros2_control.

## Hardware stack

Laptop enp131s0
→ EK1100 EtherCAT coupler
→ EL2008 8-channel digital output
→ EL1809 16-channel digital input

## Files

master.txt
- Output of sudo ethercat master
- Shows master state, link state, frame counters and slave count

slaves.txt
- Output of sudo ethercat slaves
- Confirms detected slave order and names

pdos.txt
- Output of sudo ethercat pdos
- Most useful file for future ROS2/EtherCAT configuration

beckhoff_stack.xml
- Output of sudo ethercat xml

beckhoff_stack.c
- Output of sudo ethercat cstruct

## Important discovery

Initial situation:

Phase: Waiting for device(s)
Link: DOWN
Slaves: 0

while:

ethtool enp131s0

reported:

Link detected: yes
Speed: 100Mb/s

Root cause:

DEVICE_MODULES="generic"
DEVICE_MODULES=""

The second line overrode the first.

Fix:

MASTER0_DEVICE="b0:25:aa:87:4b:f7"
DEVICE_MODULES="generic"

in:

/usr/local/etherlab/etc/sysconfig/ethercat

After restart:

ec_generic loaded
ec_master loaded
all slaves detected

## Detected slaves

0  0:0  PREOP  +  EK1100 EtherCAT-Koppler (2A E-Bus)
1  0:1  PREOP  +  EL2008 8K. Dig. Ausgang 24V, 0.5A
2  0:2  PREOP  +  EL1809 16K. Dig. Eingang 24V, 3ms

## Conclusion

Linux laptop + EtherLab 1.6.9 + Realtek RTL8125 + Beckhoff EK1100/EL2008/EL1809 works at EtherCAT discovery level.

Next step:

Build ethercat_driver_ros2 and create the first ros2_control configuration.
