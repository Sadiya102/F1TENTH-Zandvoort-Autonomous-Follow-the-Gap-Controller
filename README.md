# F1TENTH-Zandvoort-Autonomous-Follow-the-Gap-Controller


An autonomous racing project developed using the F1TENTH Gym Simulator, ROS 2 Humble, Ubuntu 22.04, and Python.

The project focuses on autonomous vehicle navigation, waypoint-based path tracking, lap detection, speed management, safety mechanisms, and racing performance evaluation on the Zandvoort circuit.

## 1. Main Objectives

- Develop an autonomous controller for the Zandvoort circuit.
- Generate and follow a manually created racing path.
- Implement Pure Pursuit-based path tracking.
- Detect and count completed laps automatically.
- Measure individual lap times and total race duration.
- Implement progressive speed scheduling.
- Monitor vehicle performance through race telemetry.
- Implement tracking-error monitoring and safety mechanisms.
- Improve vehicle stability and autonomous racing performance

## 2. Requirements

The following software and tools are required to run the autonomous driving controller.

| Software | Version |
|---|---|
| Ubuntu | 22.04 |
| ROS 2 | Humble |
| Python | 3.10 |
| NumPy | Compatible with ROS 2 Humble |
| F1TENTH Simulator | ROS 2 Simulator |
| RViz2 | ROS 2 Humble |
| Ackermann Messages | ROS 2 Humble |

Install Required Dependencies
```
sudo apt update

sudo apt install python3-numpy

sudo apt install ros-humble-ackermann-msgs
```
## 3. Simulation Environment

The F1TENTH Gym simulator is configured to use the Zandvoort racing circuit.

The simulator configuration is maintained in:

```f1tenth_gym_ros/config/sim.yaml```

Map Configuration

| Configuration | Value |
|---|---|
| Map Resolution | 0.05109 |
| Map Image | zandvoort.png |
| Starting X Position | -0.788 |
| Starting Y Position | -0.028 |
| Starting Heading | -0.023 |

RViz2 is used to visualize the map, vehicle position, and reference path.

The starting position and orientation provide the initial conditions for autonomous navigation.


## 4. Repository Structure

The project uses the F1TENTH simulator workspace and a ROS 2 package for autonomous vehicle control.

The main project files include:

```text
F1Tenth-Repository/
│
├── src/
│   └── my_gap_controller/
│       ├── my_gap_controller/
│       │   ├── hybrid_zandvoort.py
│       │   └── hybrid_zandvoort_10lap.py
│       │
│       ├── package.xml
│       ├── setup.py
│       └── setup.cfg
│
├── f1tenth_gym_ros/
│   ├── config/
│   │   └── sim.yaml
│   │
│   └── maps/
│
├── track_sections/
│   └── zandvoort_manual_waypoints.csv
│
├── build/
├── install/
└── log/
```

> **Note:** This is a representative structure of the project. The actual workspace may contain additional files and directories.

## 5. Path Generation

A manually generated waypoint route was created for the Zandvoort circuit.

**Waypoint file:** `track_sections/zandvoort_manual_waypoints.csv`

### Waypoint Configuration

| Parameter | Value |
|---|---|
| Original Waypoints |  193  |
| Interpolated Waypoints |  Approximately 2,106  |
| Interpolation Spacing |  0.20 m  |

The original 193 manually recorded waypoints were interpolated into approximately 2,106 points to create a more continuous reference path. 

The resulting route is the reference for the autonomous vehicle controller and allows the vehicle to follow a predefined racing path.

---

## 6. Autonomous Controller

The main working path-following controller is `hybrid_zandvoort.py`.

It uses the **Pure Pursuit** algorithm, a geometric path-tracking method used in autonomous vehicle navigation.

It calculates steering commands from the vehicle's current position and a selected lookahead point on the reference path, and continuously updates the steering command to guide the vehicle along the manually generated Zandvoort route.

### Controller Parameters

| Parameter | Value |
|---|---|
| Wheelbase | 0.33 m |
| Lookahead Distance | 1.0 m |
| Maximum Steering Angle | 0.418 rad |
| Initial Constant Speed | 1.0 m/s |
| Control Frequency | 20 Hz |
| Path Spacing | 0.20 m |

### Mathematical Formulation

The Pure Pursuit algorithm calculates the steering angle using the vehicle's position, heading, wheelbase, and target waypoint.

The target-point distance is calculated as:

```
Ld = √((xt − x)² + (yt − y)²)
```

The heading difference is:

```
α = atan2(yt − y, xt − x) − θ
```

The steering angle is calculated using:

```
δ = arctan( (2 · L · sin(α)) / Ld )
```

Where:

- **Ld** = Lookahead distance
- **L** = Vehicle wheelbase
- **α** = Heading difference
- **θ** = Vehicle orientation
- **δ** = Steering angle
- **(xt, yt)** = Target waypoint coordinates

The calculated steering angle is limited by the maximum steering constraint.

---

## 7. Ten-Lap Controller

A separate controller was developed for multi-lap testing: "hybrid_zandvoort_10lap.py".

It builds on the working Pure Pursuit controller and adds race-management functionality.

Implemented Features

- Automatic lap detection using waypoint progression
- Lap counting up to 10 laps
- Individual lap-time measurement
- Average lap-time calculation
- Elapsed race-time display
- Estimated remaining-time calculation
- Automatic stopping after 10 completed laps
- Progressive speed adjustment after each completed lap
- Path-tracking error monitoring
- Emergency stopping when the tracking error becomes excessive

### Adaptive Path Tracking

The ten-lap controller uses an expanded waypoint search and adaptive lookahead distance to improve path tracking during autonomous driving.

### Curvature-Based Speed Management

Vehicle speed is adjusted according to path curvature to help maintain stability when navigating corners.

### Tracking-Error Monitoring and Safety

The controller continuously monitors the distance between the vehicle and the reference path. Speed is reduced when tracking error increases, with an emergency stop mechanism for excessive deviation.

### Race Telemetry System

The controller displays race information during execution, including:

- Current lap and target speed
- Individual lap time
- Average lap time
- Total elapsed race time
- Estimated remaining time
- Race completion status

This telemetry helps monitor vehicle performance and evaluate the progress of each lap.

The original one-lap controller remains separate from the ten-lap controller, keeping the basic path-following implementation available for independent testing.

---

## 8. Ten-Lap Test Results

During the experiment, the vehicle successfully completed ten consecutive laps using an increasing speed schedule.

### Test Results

| Lap | Target Speed | Lap Time |
|---|---|---|
| 1 | 1.50 m/s | 291.7 s |
| 2 | 1.66 m/s | 274.2 s |
| 3 | 1.80 m/s | 261.5 s |
| 4 | 2.00 m/s | 250.1 s |
| 5 | 2.20 m/s | 239.8 s |
| 6 | 2.42 m/s | 230.5 s |
| 7 | 2.77 m/s | 222.1 s |
| 8 | 2.90 m/s | 214.1 s |
| 9 | 3.33 m/s | 207.5 s |
| 10 | 3.33 m/s | 🏆200.9 s |

### Observations

- The controller completed ten consecutive laps in the simulation.
- Lap times decreased as the target speed increased.
- Automatic lap detection and timing functions operated during the test.
- The controller used path-tracking and safety mechanisms throughout the experiment.

---

## 9. Building the ROS 2 Package

The project uses a ROS 2 Python package named `my_gap_controller`.

To build it, open a terminal and navigate to the repository directory:

```bash
cd ~/F1Tenth-Repository
source /opt/ros/humble/setup.bash
colcon build --packages-select my_gap_controller
source install/setup.bash
```

After a successful build, the workspace is ready to run the installed controller executables.

---

## 10. Running the F1TENTH Simulator

The simulator must be running before starting either autonomous controller.

**Terminal 1:**

```bash
# Step 1 – Source the ROS 2 environment
source /opt/ros/humble/setup.bash

# Step 2 – Set the ROS domain ID
export ROS_DOMAIN_ID=42

# Step 3 – Source the workspace
cd ~/F1Tenth-Repository
source install/setup.bash

# Step 4 – Launch the simulator
ros2 launch f1tenth_gym_ros gym_bridge_launch.py
```

### Expected Result

- The F1TENTH simulator launches.
- The Zandvoort map is loaded.
- The vehicle appears in the simulation environment.
- ROS 2 communication is available.

Keep Terminal 1 running while using the simulator.

---

## 11. Running the Autonomous Controllers

Open a separate terminal after the simulator has launched.

```bash
# Step 1 – Source ROS 2 Humble
source /opt/ros/humble/setup.bash

# Step 2 – Source the workspace
cd ~/F1Tenth-Repository
source install/setup.bash
```

**Original Pure Pursuit controller (one lap):**

```bash
ros2 run my_gap_controller hybrid_zandvoort
```

**Experimental ten-lap controller:**

```bash
ros2 run my_gap_controller hybrid_zandvoort_10lap
```

> **Important notes**
> - Launch the simulator before starting either controller.
> - Run only one autonomous controller at a time.
> - Do not run manual driving commands simultaneously.
> - Ensure the reference waypoint file is available at the location expected by the controller.

---

## 12. Monitoring and Verification

ROS 2 commands can be used to monitor communication between the simulator and the autonomous controller.

```bash
# List available topics
ros2 topic list

# Monitor vehicle odometry (position, orientation, velocity)
ros2 topic echo /ego_racecar/odom

# Monitor driving commands (steering and speed)
ros2 topic echo /drive

# Inspect driving topic information
ros2 topic info /drive --verbose
```

### Verification Checklist

- [ ] Simulator launches successfully
- [ ] Odometry information is available
- [ ] Controller publishes steering and speed commands
- [ ] Vehicle responds to the commands
- [ ] Vehicle follows the reference waypoint path
- [ ] Lap progression and lap-time measurements are monitored
- [ ] Progessive speed adjustments are applied
- [ ] Ten-lap completion and automatic stopping are verified


---

## 13. Troubleshooting

During the development and testing of the F1TENTH controller, some common issues may occur.

The following table describes possible problems and basic solutions.

| Problem | Possible Cause | Suggested Solution |
|---|---|---|
| Simulator does not launch | Missing packages or incorrect environment setup | Check `/ego_racecar/odom`. |
| Vehicle does not move | Controller is not publishing commands | Check the `/drive` topic. |
| No LiDAR data | Simulator or sensor connection issue | Check whether the `/scan` topic is available. |
| No odometry commands | Simulator communication issue | Check the terminal for errors and restart the controller. |
| Vehicle stops unexpectedly | Safety mechanism is activated | Check tracking error and safety thresholds. |
| Vehicle moves in an unexpected direction | Incorrect steering calculation or parameter settings | Review the steering angle and controller configuration. |
| Vehicle losses the reference path | Excessive speed or unstable steering response | Reduce speed and tune the lookahead distance. |
| Python import errors | Missing dependencies or incorrect Python environment | Install the required dependencies and source the ROS 2 environment. |

---

## 14. Current Development Status

The following table summarizes the current progress of the project.

| Component | Status |
|---|---|
| F1TENTH Simulator Setup | ✅ Completed |
| Zandvoort Map Configuration | ✅ Completed |
| RViz2 Visualization | ✅ Completed |
| Manual Waypoint Generation | ✅ Completed |
| Path Interpolation | ✅ Completed |
| Pure Pursuit Path Tracking | ✅ Working |
| One-Lap Autonomous Run | ✅ Completed |
| Adaptive Lookahead | ✅ Implemented |
| Curvature-Based Speed Management | ✅ Implemented |
| Automatic Lap Detection | ✅ Implemented |
| Lap-Time Measurement | ✅ Implemented |
| Ten-Lap Autonomous  | ✅ Completed |
| Race Telemetry | ✅ Implemented  |
| Tracking-Error Safety Stop | ✅ Implemented |
| 10-Lap Autonomous Run | ✅ Completed |

---

## 15. Ten lap Autonomous Lap Demonstration

[![F1tenth Zandvoort Controller ](https://img.youtube.com/vi/VMc_mQlOQj0/0.jpg)](https://youtu.be/VMc_mQlOQj0)

---

## 16. Conclusion

- This project developed a Pure Pursuit-based autonomous conroller.
- The vehicle followed a manually generated Zandvoort waypoint path.
- The ten-lap controller implemented progressive and timing.
- Race telemetry tracked lap progression and timing.
-  The ten-lap simulation experiment was completed.
-  Further work focuses on repeatability, timing accuracy and high-speed performance.
