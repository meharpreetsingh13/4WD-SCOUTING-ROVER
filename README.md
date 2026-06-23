# 🤖 4WD Skid-Steer Scouting Rover
### Post-Earthquake Urban Search & Rescue Simulation

![ROS2](https://img.shields.io/badge/ROS2-Humble-blue)
![Gazebo](https://img.shields.io/badge/Gazebo-Fortress-orange)
![Python](https://img.shields.io/badge/Python-3.10-green)
![Platform](https://img.shields.io/badge/Platform-WSL2%20%2B%20WSLg-lightgrey)
---
## 📋 Overview

A fully simulated 4-wheeled skid-steer ground rover designed for scouting
post-earthquake urban environments. The rover navigates through rubble,
narrow passages and uneven terrain while building a real-time map using SLAM.

**Based on:** Panther v0.2 CAD model (GrabCAD)
**Drive type:** Skid-steer (tank-style, no steering axle)
**Simulation:** Gazebo Fortress (Ignition 6.17.1)
**Framework:** ROS2 Humble

---

## ✨ Features

- ✅ Realistic 4WD skid-steer kinematics
- ✅ Full URDF with real CAD mesh geometry (STL/DAE)
- ✅ 360° LiDAR sensor (12 m range)
- ✅ Forward-facing RGB camera
- ✅ IMU sensor
- ✅ Post-earthquake Gazebo world with rubble + narrow passages
- ✅ Arrow-key hold-to-move teleoperation
- ✅ Real-time SLAM mapping (slam_toolbox)
- ✅ RViz visualization
- 🔲 Human detection (planned)

---

## 🗂️ Project Structure

```
rover_description/
├── rover_description/
│   └── teleop_hold_node.py      ← custom teleop node
├── urdf/
│   ├── rover.urdf.xacro         ← main robot description
│   └── rover_gazebo_fortress.xacro  ← Gazebo plugins
├── meshes/
│   ├── top_chasis_4WD_r.dae
│   ├── small_chasis_1_r.dae
│   ├── small_chasis_2_r.dae
│   ├── Tire_r.dae
│   └── tire_rim_r.dae
├── launch/
│   ├── gazebo.launch.py         ← simulation launch
│   └── slam.launch.py           ← SLAM + Gazebo launch
├── worlds/
│   └── earthquake_world.sdf     ← custom rubble environment
├── config/
│   └── slam_toolbox_params.yaml ← SLAM tuning parameters
├── models/
│   └── human_target.sdf         ← survivor model (planned)
├── package.xml
├── setup.py
└── setup.cfg
```

---

## 🔧 Prerequisites

### System
- Ubuntu 22.04 (or WSL2 + WSLg on Windows 11)
- ROS2 Humble
- Python 3.10

### Install Dependencies

```bash
# Gazebo Fortress
sudo apt install ignition-fortress

# ROS2 ↔ Gazebo bridge
sudo apt install ros-humble-ros-gz \
                 ros-humble-ros-gz-sim \
                 ros-humble-ros-gz-bridge \
                 ros-humble-ros-gz-image

# SLAM
sudo apt install ros-humble-slam-toolbox

# Map server (for saving maps)
sudo apt install ros-humble-nav2-map-server
```

---

## 🚀 Quick Start

### 1. Clone & Build

```bash
cd ~/ros2_ws/src
# Copy rover_description package here

cd ~/ros2_ws
colcon build --packages-select rover_description
source install/setup.bash
```

### 2. Set Display (WSL2 only)

```bash
export DISPLAY=:0
export WAYLAND_DISPLAY=wayland-0
export XDG_RUNTIME_DIR=/tmp/runtime-root
mkdir -p /tmp/runtime-root && chmod 700 /tmp/runtime-root
```

### 3. Launch Simulation

```bash
# Gazebo only
ros2 launch rover_description gazebo.launch.py

# Gazebo + SLAM + RViz
ros2 launch rover_description slam.launch.py rviz:=true
```

### 4. Teleoperate

```bash
ros2 run rover_description teleop_hold_node
```

---

## 🎮 Controls

| Key | Action |
|-----|--------|
| `↑` (hold) | Forward |
| `↓` (hold) | Backward |
| `←` (hold) | Rotate Left |
| `→` (hold) | Rotate Right |
| `SPACE` | Emergency Stop |
| `q` | Speed Up |
| `z` | Speed Down |
| `CTRL+C` | Quit |

> Key must be **held** to move — rover stops immediately on release.

---

## 🤖 Robot Specifications

| Parameter | Value |
|-----------|-------|
| Chassis | 220 × 120 × 34 mm |
| Wheel Radius | 63 mm |
| Wheel Track | 180 mm |
| Total Mass | ~8.0 kg |
| Ground Clearance | 63 mm |
| Max Linear Speed | 2.0 m/s |
| Max Angular Speed | 2.0 rad/s |

### Sensors

| Sensor | Range | Topic | Rate |
|--------|-------|-------|------|
| LiDAR (360°) | 0.15 – 12.0 m | `/scan` | 10 Hz |
| RGB Camera | 80° FOV, 640×480 | `/camera/image_raw` | 30 Hz |
| IMU | — | `/imu/data` | 100 Hz |

---

## 🌍 Earthquake World

A 10×10 m bounded environment simulating urban earthquake damage:

```
╔══════════════════════════════════╗
║         [Survivor Zone 🟧]      ║
║    rubble    [passage 2]  wall   ║
║  wall remnant    debris         ║
║                [passage 1]      ║
║    fallen    pillars   chunks   ║
║         [SPAWN POINT]           ║
╚══════════════════════════════════╝
```

**Features:**
- 2 narrow passages (~0.35 m gap)
- Fallen pillars + wall chunks
- Tilted ground sections (uneven terrain)
- Orange survivor zone marker at north end
- ODE physics at 250 Hz

---

## 🗺️ SLAM

Real-time mapping using `slam_toolbox` async online mapper.

```bash
# Launch with SLAM and RViz
ros2 launch rover_description slam.launch.py rviz:=true

# Save map when done
ros2 run nav2_map_server map_saver_cli -f ~/rover_map

# Convert to PNG
convert ~/rover_map.pgm ~/rover_map.png
```

**RViz Setup:**
- Fixed Frame → `map`
- Add `Map` → `/map`
- Add `LaserScan` → `/scan`
- Add `Image` → `/camera/image_raw`

---

## 📡 Topics

| Topic | Type | Direction |
|-------|------|-----------|
| `/cmd_vel` | `geometry_msgs/Twist` | Teleop → Gazebo |
| `/odom` | `nav_msgs/Odometry` | Gazebo → ROS |
| `/scan` | `sensor_msgs/LaserScan` | Gazebo → ROS |
| `/map` | `nav_msgs/OccupancyGrid` | SLAM → RViz |
| `/camera/image_raw` | `sensor_msgs/Image` | Gazebo → ROS |
| `/imu/data` | `sensor_msgs/Imu` | Gazebo → ROS |
| `/tf` | `tf2_msgs/TFMessage` | Gazebo → ROS |

---

## ⚠️ Known Issues

| Issue | Fix |
|-------|-----|
| `Ogre::UnimplementedException` on launch | Change `ogre2` → `ogre` in world SDF |
| `qt.qpa.xcb` display error | Run `export DISPLAY=:0` first |
| Rover slides after stopping | Joint damping set to 0.5 |
| Zone.Identifier files in meshes | Run `rm -f meshes/*Zone.Identifier*` |
| `odom.twist` always zero | Known Fortress bridge issue — PID runs passthrough |

---

## 📦 Package Dependencies

```xml
<exec_depend>ros_gz_sim</exec_depend>
<exec_depend>ros_gz_bridge</exec_depend>
<exec_depend>ros_gz_image</exec_depend>
<exec_depend>robot_state_publisher</exec_depend>
<exec_depend>joint_state_publisher</exec_depend>
<exec_depend>slam_toolbox</exec_depend>
<exec_depend>xacro</exec_depend>
```

---

## 🔮 Future Work

- [ ] Human detection node (architecture ready, implementation pending)
- [ ] Nav2 autonomous navigation
- [ ] SLAM map → Nav2 costmap integration
- [ ] Multi-robot coordination
- [ ] Real hardware deployment
---

*4WD Scouting Rover — ROS2 Humble + Gazebo Fortress*
