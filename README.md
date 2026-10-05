# MechaUGV Bot 🤖

A custom **ROS 2 UGV (Unmanned Ground Vehicle)** simulation project built from scratch using **URDF/Xacro, RViz2, TF, ros2_control, and diff_drive_controller**.

The project is structured as a ROS 2 workspace with separate packages for the robot description and robot bringup/control stack.

## 🚀 Features

- Custom UGV model built with **URDF/Xacro**
- Modular robot description using reusable Xacro files
- Four main robot frames/links:
  - `base_footprint`
  - `base_link`
  - `left_wheel_link`
  - `right_wheel_link`
  - `caster_wheel_link`
- Continuous joints for the left and right drive wheels
- Rear/front caster wheel representation
- **robot_state_publisher** for publishing the robot TF tree
- **RViz2** configuration for visualizing the robot and TF frames
- **ros2_control** integration using a mock hardware interface
- **joint_state_broadcaster**
- **diff_drive_controller**
- Odometry and `odom -> base_footprint` TF publishing
- Velocity limits configured for linear and angular motion
- Clean ROS 2 package separation

## 🧠 Project Architecture

```text
MechaUGV_bot/
│
├── .gitignore
│
└── src/
    │
    ├── my_robot_description/
    │   ├── urdf/
    │   │   ├── my_robot.urdf.xacro
    │   │   ├── mobile_base.xacro
    │   │   ├── mobile_base.ros2_control.xacro
    │   │   └── common_properties.xacro
    │   │
    │   ├── launch/
    │   │   ├── display.launch.py
    │   │   └── display.launch.xml
    │   │
    │   └── rviz/
    │       └── urdf_config.rviz
    │
    └── my_robot_bringup/
        ├── config/
        │   └── my_robot_controllers.yaml
        │
        └── launch/
            └── my_robot.launch.xml
```

## 🔧 Robot Description

The main robot description is generated from:

```text
my_robot.urdf.xacro
        │
        ├── common_properties.xacro
        ├── mobile_base.xacro
        └── mobile_base.ros2_control.xacro
```

### Chassis

The UGV chassis is represented by a box with:

| Parameter | Value |
|---|---:|
| Base length | 0.60 m |
| Base width | 0.40 m |
| Base height | 0.20 m |
| Wheel radius | 0.10 m |
| Wheel thickness | 0.05 m |

### Drive system

The robot uses a differential-drive configuration:

- Left wheel → `base_left_wheel_joint`
- Right wheel → `base_right_wheel_joint`
- Wheel radius → **0.10 m**
- Wheel separation → **0.45 m**

The drive wheel joints are continuous joints with their rotation axis along the Y-axis.

## 🦾 ros2_control

The robot description includes a `ros2_control` system:

```text
MobileBaseHardwareInterface
        │
        ├── base_left_wheel_joint
        │     ├── velocity command interface
        │     ├── velocity state interface
        │     └── position state interface
        │
        └── base_right_wheel_joint
              ├── velocity command interface
              ├── velocity state interface
              └── position state interface
```

The current implementation uses:

```text
mock_components/GenericSystem
```

with dynamics calculation enabled, making it suitable for development and simulation before connecting real motor hardware.

## 🎮 Differential Drive Controller

The controller configuration is located at:

```text
src/my_robot_bringup/config/my_robot_controllers.yaml
```

The project uses:

```text
diff_drive_controller/DiffDriveController
```

along with:

```text
joint_state_broadcaster/JointStateBroadcaster
```

The controller publishes odometry using:

```text
odom → base_footprint
```

and is configured to publish the odometry TF.

### Motion limits

Current configured limits:

| Motion | Limit |
|---|---:|
| Maximum linear velocity | 1.0 m/s |
| Minimum linear velocity | -1.0 m/s |
| Maximum angular velocity | 1.0 rad/s |
| Minimum angular velocity | -1.0 rad/s |
| Controller update rate | 50 Hz |
| Odometry publish rate | 50 Hz |

## 👁️ RViz2 Visualization

The project includes a predefined RViz2 configuration.

RViz displays:

- Robot model
- TF frames
- Grid
- `base_footprint`
- `base_link`
- Left wheel
- Right wheel
- Caster wheel

The RViz configuration is stored at:

```text
src/my_robot_description/rviz/urdf_config.rviz
```

## 📦 Requirements

Recommended environment:

- Ubuntu 22.04
- ROS 2 Humble
- colcon
- Xacro
- RViz2
- robot_state_publisher
- ros2_control
- controller_manager
- diff_drive_controller

> The project is intended for ROS 2 Humble development.

## 🛠️ Installation

Clone the repository into your ROS 2 workspace:

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src

git clone https://github.com/devbytush/MechaUGV_bot.git
```

Build the workspace:

```bash
cd ~/ros2_ws

source /opt/ros/humble/setup.bash

colcon build
```

Source the workspace:

```bash
source ~/ros2_ws/install/setup.bash
```

## ▶️ Run the Robot Description in RViz2

To visualize the URDF/Xacro model:

```bash
ros2 launch my_robot_description display.launch.py
```

This starts:

1. `robot_state_publisher`
2. `joint_state_publisher_gui`
3. `rviz2`

The Xacro file is converted into a URDF robot description at launch time.

## ▶️ Run the Full Bringup

For the ros2_control-based setup:

```bash
ros2 launch my_robot_bringup my_robot.launch.xml
```

The bringup launches:

```text
my_robot.urdf.xacro
        ↓
robot_state_publisher
        ↓
ros2_control_node
        ↓
joint_state_broadcaster
        ↓
diff_drive_controller
        ↓
RViz2
```

## 🔍 Useful ROS 2 Commands

### Check active nodes

```bash
ros2 node list
```

### Check active topics

```bash
ros2 topic list
```

### Inspect robot description

```bash
ros2 topic echo /robot_description
```

### Check TF frames

```bash
ros2 topic echo /tf
```

### Check controller manager

```bash
ros2 control list_controllers
```

### Check hardware interfaces

```bash
ros2 control list_hardware_interfaces
```

### Check odometry

```bash
ros2 topic echo /diff_drive_controller/odom
```

### Send velocity commands

The differential-drive controller can receive `geometry_msgs/msg/Twist` commands on its command topic.

For example:

```bash
ros2 topic pub --rate 10 /diff_drive_controller/cmd_vel_unstamped geometry_msgs/msg/Twist \
"{linear: {x: 0.5}, angular: {z: 0.0}}"
```

Stop the robot with:

```bash
ros2 topic pub --rate 10 /diff_drive_controller/cmd_vel_unstamped geometry_msgs/msg/Twist \
"{linear: {x: 0.0}, angular: {z: 0.0}}"
```

> Topic names can vary with controller configuration/version. Use `ros2 topic list` to verify the active command topic.

## 🌳 TF Tree

The robot description creates the following main TF structure:

```text
base_footprint
      │
      └── base_link
            ├── left_wheel_link
            ├── right_wheel_link
            └── caster_wheel_link
```

When the differential-drive controller is running, odometry adds:

```text
odom
  │
  └── base_footprint
        │
        └── base_link
              ├── left_wheel_link
              ├── right_wheel_link
              └── caster_wheel_link
```

## 📚 Package Overview

### `my_robot_description`

Responsible for the robot model and visualization.

Contains:

- URDF/Xacro files
- Robot dimensions and geometry
- Wheel joints
- ros2_control tags
- RViz configuration
- Description launch files

### `my_robot_bringup`

Responsible for bringing the complete robot stack together.

Contains:

- Controller configuration
- ros2_control startup
- Joint state broadcaster
- Differential-drive controller
- RViz2 startup

## 🧩 Learning Goals

This project is also intended as a practical ROS 2 learning project covering:

- ROS 2 workspace and package structure
- URDF
- Xacro
- Links and joints
- TF / TF2
- robot_state_publisher
- RViz2
- ROS 2 launch files
- ros2_control
- controller_manager
- joint state broadcasting
- differential-drive control
- odometry
- ROS 2 topics and messages
- Robot bringup architecture

## 👤 Author

**Tushar**

GitHub: [@devbytush](https://github.com/devbytush)

---

⭐ If this project helps you learn ROS 2 or build your own UGV, consider starring the repository.
