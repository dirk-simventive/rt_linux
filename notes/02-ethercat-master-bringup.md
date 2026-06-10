# EtherCAT Master Bring-up on Ubuntu 24.04

Date: 2025-06-08

## Objective

Evaluate Linux + EtherCAT + ROS2 as a future robot control platform and determine whether a modern Ubuntu laptop can be used as the starting point for development.

## Starting Point

System:

- Ubuntu 24.04
- ROS2 Jazzy
- Kernel 6.17.0-35-generic
- Realtek RTL8125 Ethernet controller
- Existing Simulink and Beckhoff experience

Goal architecture:

Simulink generated controller
-> ros2_control
-> ethercat_driver_ros2
-> EtherLab EtherCAT Master
-> Beckhoff I/O and drives

## Realtime Evaluation

Installed:

    sudo apt install rt-tests

Measured:

    sudo cyclictest -p 80 -m -i 1000 -D 300s

Result:

- Average latency approximately 1 us
- Maximum latency approximately 914 us

Conclusion:

The generic Ubuntu kernel is already suitable for development and experimentation. PREEMPT_RT is not yet required.

## Repository Structure

Created and configured:

    ~/github/rt_linux

Additional repositories:

    ~/github/ethercat_driver_ros2
    ~/github/ethercat
    ~/github/rt-linux

Meaning:

- rt_linux = personal evaluation and notes repository
- ethercat_driver_ros2 = ICube ROS2 EtherCAT driver
- ethercat = EtherLab master source
- rt-linux = existing project reference implementation

## ROS2 Investigation

Verified:

    ROS_DISTRO=jazzy

Observation:

The ICube repository itself contains a Jazzy branch and CI configuration for Jazzy, even though some documentation still refers to Humble.

Conclusion:

Remain on ROS2 Jazzy.

## EtherLab Investigation

### Attempt 1

Checked out:

    stable-1.5

Result:

Build failed on Linux 6.17.

Conclusion:

EtherLab stable-1.5 is not compatible with the current kernel.

### Investigation of Existing Project

Reviewed the existing project installation scripts.

Observation:

The project also used EtherLab stable-1.5 with generic driver mode.

Conclusion:

The project likely depended on an older Ubuntu / kernel combination rather than custom EtherLab modifications.

### Attempt 2

Checked out:

    stable-1.6

Configured:

    ./configure \
      --prefix=/usr/local/etherlab \
      --disable-8139too \
      --disable-eoe \
      --enable-generic

Result:

Successful build on Ubuntu 24.04 and kernel 6.17.

Version:

    EtherLab 1.6.9

Key finding:

EtherLab stable-1.6 is currently the correct branch for this laptop.

## EtherCAT Master Installation

Installed:

    sudo make modules_install install
    sudo depmod

Configured:

    /usr/local/etherlab/etc/sysconfig/ethercat

Values:

    MASTER0_DEVICE="b0:25:aa:87:4b:f7"
    DEVICE_MODULES="generic"

Important lesson:

The startup script reads:

    /usr/local/etherlab/etc/sysconfig/ethercat

and not:

    /etc/sysconfig/ethercat

## Master Startup

Started with:

    sudo /usr/local/etherlab/etc/init.d/ethercat start

Result:

    Starting EtherCAT master 1.6.9 done

Verification:

    sudo ethercat master

Result:

- Master started successfully
- Kernel modules loaded
- NIC accepted
- No slaves present
- Link down because no EtherCAT hardware connected

## Current Status

Working:

- GitHub workflow
- ROS2 Jazzy
- EtherLab 1.6.9
- EtherCAT master startup
- Realtek NIC configured

Not yet tested:

- Beckhoff hardware discovery
- Slave communication
- ros2_control integration
- ethercat_driver_ros2 build
- Simulink integration

## Main Conclusion

A modern Ubuntu 24.04 + ROS2 Jazzy laptop is a viable starting platform for Linux-based robot control development.

The first major technical risk has been retired:

EtherLab 1.6.9 runs successfully on this machine.

## Next Session

1. Connect Beckhoff hardware.
2. Run:

       sudo ethercat slaves

3. Verify EK1100/CX and EL terminals are detected.
4. Build ethercat_driver_ros2.
5. Create first ros2_control example.

## Hardware Scan Update

The Beckhoff hardware was connected and successfully detected.

Hardware stack:

Laptop enp131s0
→ EK1100
→ EL2008
→ EL1809

Physical link verification:

sudo ethtool enp131s0

Result:

Speed: 100Mb/s
Duplex: Full
Link detected: yes

Initially EtherLab still reported:

Phase: Waiting for device(s)
Link: DOWN
Slaves: 0

Investigation showed that ec_master was loaded but ec_generic was not.

Root cause:

DEVICE_MODULES="generic"
DEVICE_MODULES=""

The second line overrode the first.

Fixed configuration:

MASTER0_DEVICE="b0:25:aa:87:4b:f7"
DEVICE_MODULES="generic"

File:

/usr/local/etherlab/etc/sysconfig/ethercat

After restarting the EtherCAT master:

sudo /usr/local/etherlab/etc/init.d/ethercat stop
sudo /usr/local/etherlab/etc/init.d/ethercat start

the following modules were loaded:

ec_generic
ec_master

and slave detection succeeded:

0  0:0  PREOP  +  EK1100 EtherCAT-Koppler (2A E-Bus)
1  0:1  PREOP  +  EL2008 8K. Dig. Ausgang 24V, 0.5A
2  0:2  PREOP  +  EL1809 16K. Dig. Eingang 24V, 3ms

Artifacts are stored in:

artifacts/ethercat_scan_ek1100_el2008_el1809/

This completes the first successful EtherCAT hardware bring-up on the Linux laptop.
