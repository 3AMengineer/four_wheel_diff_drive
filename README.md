# four_wheel_diff_drive

A **4-wheel differential-drive rover built from scratch** as a ROS 2 package: URDF model, Gazebo simulation, a 360° 2D LiDAR and an Intel RealSense D455-style depth camera. It is the simulation platform I used to develop LiDAR- and depth-camera-based autonomous navigation ([apf-navigation](https://github.com/3AMengineer/apf-navigation)) before moving to a real rover.

<!-- Add a screenshot or GIF of the rover in Gazebo and RViz here, e.g. docs/gazebo_rover.gif -->

---

## The model

| Part | Details |
|---|---|
| Chassis | 1.0 × 0.6 × 0.3 m box body |
| Drive | 4 continuous wheel joints, 0.3 m diameter, 0.8 m track. Two `gazebo_ros_diff_drive` plugins (front and rear pairs) give 4-wheel skid-steer behaviour from a single `/cmd_vel` |
| 2D LiDAR | 360 samples over 360°, 10 Hz, range up to 12 m → `/scan` (`laser_frame`) |
| Depth camera | Modelled on the D455: 87° horizontal FOV, 848 × 480, 0.3–6 m, 30 Hz → `/camera/depth/image_raw` with a proper optical frame (`camera_link_optical`) |
| Odometry | Wheel odometry and TF from the drive plugins, plus ground-truth pose from `gazebo_ros_p3d` on `/odom` |

Two variants:

- `urdf/four_wheel_diffdrive.urdf` is the LiDAR-only rover.
- `urdf/depth_four_wheel_diffdrive.urdf` is the LiDAR plus depth camera rover.

---

## Run it

**Tested on:** ROS 2 Humble, Gazebo Classic 11 (`gazebo_ros_pkgs`).

```bash
mkdir -p ~/ws_diffdrive/src && cd ~/ws_diffdrive/src
git clone https://github.com/3AMengineer/four_wheel_diff_drive.git
cd .. && colcon build --packages-select four_wheel_diff_drive
source install/setup.bash

# LiDAR rover
ros2 launch four_wheel_diff_drive gazebo_model.launch.py
# or: LiDAR + depth camera rover
ros2 launch four_wheel_diff_drive depth_gazebo_model.launch.py

# Drive it
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

In RViz, set the fixed frame to `odom` and add `/scan`, `/camera/depth/image_raw` and the RobotModel display.

---

## What I learned building it

- **Frames matter more than geometry.** A wrong joint origin or a camera without an optical frame looks fine in Gazebo and silently breaks everything downstream.
- **Skid-steer isn't textbook differential drive.** Four driven wheels slip when they turn, so the odometry from the plugins drifts. That's why the navigation stack compares it against ground truth.
- **Simulating the sensor you'll actually use pays off.** Matching the D455's FOV and range early made the move from LiDAR to depth perception much less surprising.

## Built on top of this

- **[apf-navigation](https://github.com/3AMengineer/apf-navigation):** artificial potential field obstacle avoidance on 2D LiDAR and on D455 depth images
- The UAS-DTU autonomous rover for the **DARPA Triage Challenge**: ROS 2, Nav2, a Blickfeld 3D LiDAR, LiDAR-inertial odometry and ArduRover on real hardware

## Author

**Shourya Sharma**, Mathematics & Computing @ DTU, UAS-DTU
[LinkedIn](https://www.linkedin.com/in/shourya-sharma-6a365a341)
