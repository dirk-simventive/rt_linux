# Beckhoff EtherCAT ROS2 Bringup Notes

## Hardware

EtherCAT chain:

```text
EK1100 → EL2008 (8 digital outputs) → EL1809 (16 digital inputs)
```

Test wiring:

```text
EL2008 Output 1 → EL1809 Input 1
```

---

# EtherCAT Verification

Start EtherCAT master:

```bash
sudo /usr/local/etherlab/etc/init.d/ethercat start
```

If slaves show as unknown:

```bash
sudo ethercat rescan
sudo ethercat slaves
```

Expected:

```text
0  0:0  PREOP  +  EK1100
1  0:1  PREOP  +  EL2008
2  0:2  PREOP  +  EL1809
```

---

# ROS2 Control Configuration

## EL2008

Use command interfaces:

```text
DO.1
DO.2
DO.3
DO.4
DO.5
DO.6
DO.7
DO.8
```

## EL1809

Use state interfaces:

```text
DI.1
DI.2
DI.3
DI.4
DI.5
DI.6
DI.7
DI.8
DI.9
DI.10
DI.11
DI.12
DI.13
DI.14
DI.15
DI.16
```

The names in the Xacro must match the names defined in the EtherCAT YAML files.

Incorrect:

```text
d_output.1
d_input.1
```

Correct:

```text
DO.1
DI.1
```

---

# Build

```bash
cd ~/ros2_ws

source /opt/ros/jazzy/setup.bash

colcon build --packages-select beckhoff_io_demo --symlink-install

source install/setup.bash
```

---

# Launch

```bash
ros2 launch beckhoff_io_demo bringup.launch.py
```

Verify interfaces:

```bash
ros2 control list_hardware_interfaces
```

Expected:

```text
command interfaces
  el2008_outputs/DO.1
  ...
  el2008_outputs/DO.8

state interfaces
  el1809_inputs/DI.1
  ...
  el1809_inputs/DI.16
```

---

# Custom GPIO Controller

Plugin:

```text
beckhoff_io_demo/GpioCommandController
```

Load:

```bash
ros2 control load_controller gpio_controller
ros2 control set_controller_state gpio_controller active
```

Verify:

```bash
ros2 control list_controllers
```

Expected:

```text
gpio_controller active
```

---

# Functional Test

Controller behavior:

* Toggle DO.1 every second
* Read DI.1 every cycle

Console output:

```text
Input1=0 Output1=1
Input1=1 Output1=0
Input1=0 Output1=1
```

Observed:

* EL2008 output LED blinks
* EL1809 input follows wiring state
* ROS2 receives input state correctly

Result:

```text
ROS2 Controller
      ↓
ros2_control
      ↓
EtherCAT Driver
      ↓
EL2008 Output
      ↓
Physical Wiring
      ↓
EL1809 Input
      ↓
ros2_control State Interface
      ↓
ROS2 Controller
```

---

# Important Discovery

When EtherCAT slaves appear as:

```text
???
0x00000000:0x00000000
```

run:

```bash
sudo ethercat rescan
```

before launching ROS2.

The EtherCAT bus may be physically present but not yet scanned after boot.

---

# Current Status

* [x] EtherCAT master operational
* [x] EK1100 operational
* [x] EL2008 operational
* [x] EL1809 operational
* [x] Custom ros2_control controller operational
* [x] Digital output writing verified
* [x] Digital input reading verified

## Next Step

Replace the demo toggle logic with Simulink-generated code.

Target architecture:

```text
GpioCommandController
 ├─ Read EtherCAT Inputs
 ├─ Copy into Simulink Inputs
 ├─ model_step()
 ├─ Copy Simulink Outputs
 └─ Write EtherCAT Outputs
```

