# Linux RT baseline

## Goal

Establish whether the current Linux laptop is suitable as a first development platform for real-time or soft real-time robot control experiments.

The intended learning path is:

- configure Linux for real-time control experiments
- install and test an EtherCAT master
- connect Beckhoff I/O modules
- integrate with ros2_control
- later wrap a Simulink-generated C++ controller

## System

Machine:

- Linux laptop
- Ubuntu 24.04
- Kernel: `6.17.0-35-generic`
- Kernel type: `SMP PREEMPT_DYNAMIC`
- Logical CPUs: 24

Network interface:

- Wired Ethernet: `enp131s0`
- NIC: Realtek RTL8125 2.5GbE Controller
- Wi-Fi: `wlp128s20f3`

## Secure Boot and kernel modules

Commands used:

```bash
mokutil --sb-state
sudo modprobe dummy
lsmod | grep dummy
cat /sys/kernel/security/lockdown

sudo apt install rt-tests
ls
cat > README.md <<'EOF'
# RT Linux Evaluation

Goal: evaluate Linux + EtherCAT + ros2_control as a practical platform for robot control development.

This repository documents setup steps, experiments, configuration files, and lessons learned while building a minimal real-time Linux control demonstrator.
