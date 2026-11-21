# F1TENTH-Zandvoort-Autonomous-Follow-the-Gap-Controller


An autonomous racing project developed using the F1TENTH Gym Simulator, ROS 2 Humble, Ubuntu 22.04, and Python.

The project focuses on autonomous vehicle navigation, waypoint-based path tracking, lap detection, speed management, safety mechanisms, and racing performance evaluation on the Zandvoort circuit.

## 1. Main Objectives

- Develop an autonomous controller for the Zandvoort circuit.
- Generate and follow a manually created racing path.
- Implement waypoint-based autonomous navigation.
- Detect and count completed laps automatically.
- Measure individual lap times and total race duration.
- Experiment with speed optimization.
- Monitor path-tracking errors and implement safety mechanisms.
- Improve vehicle stability and racing performance

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

### ROS 2 Communication

The controller communicates with the simulator through ROS 2 topics.

| Topic | Message Type | Purpose |
|---|---|---|
| `/ego_racecar/odom` | `nav_msgs/msg/Odometry` | Receives vehicle position and orientation |
| `/drive` | `ackermann_msgs/msg/AckermannDriveStamped` | Publishes steering and speed commands |
| `/manual_waypoint_path` | `nav_msgs/msg/Path` | Visualizes the reference path in RViz2 |

The controller successfully completed one full lap at a constant speed of 1.0 m/s while following the manually generated waypoint route.

---

## 7. Ten-Lap Controller

A separate controller was developed for multi-lap testing: `hybrid_zandvoort_10lap.py`.

It builds on the working Pure Pursuit controller and adds race-management functionality.

### Implemented Features

- Automatic lap detection using waypoint progression
- Lap counting up to 10 laps
- Individual lap-time measurement
- Average lap-time calculation
- Elapsed race-time display
- Estimated remaining-time calculation
- Automatic stopping after 10 completed laps
- Speed adjustment after each completed lap
- Path-tracking error monitoring
- Emergency stopping when the tracking error becomes excessive

The original one-lap controller remains separate from the experimental ten-lap controller, so the basic path-following implementation stays available while additional race-management features are tested.

---

## 8. Five-Lap Test Results

During the first ten-lap controller experiment, the vehicle successfully completed five consecutive laps using an increasing speed schedule.

### Test Results

| Lap | Target Speed | Lap Time |
|---|---|---|
| 1 | 1.50 m/s | 258.09 s |
| 2 | 1.80 m/s | 215.10 s |
| 3 | 2.16 m/s | 179.60 s |
| 4 | 2.59 m/s | 150.20 s |
| 5 | 3.11 m/s | 125.60 s |

**Average lap time:** approximately 185.72 s

### Observations

- All five laps were completed successfully.
- Lap times decreased as the target speed increased.
- Automatic lap detection and timing functions operated during the test.
- Tracking error increased at higher speeds.
- The waypoint index eventually stopped progressing around waypoints 759–765.
- The vehicle could no longer reliably maintain the reference path.

### Test Evaluation

The five-lap experiment demonstrated that the controller can perform consecutive autonomous laps and measure individual lap times. 

It also revealed limitations in high-speed path tracking.

Further controller tuning is required to improve vehicle stability and maintain reliable waypoint progression at higher speeds.

---

## 9. Speed Optimization and Safety

Further development focuses on experimenting with different speed schedules while maintaining stable vehicle movement.

One experimental approach uses a descending speed schedule, beginning at a higher speed and reducing the target speed by 20% after each completed lap.

### Experimental Speed Schedule

| Lap | Target Speed |
|---|---|
| 1 | 8.00 m/s |
| 2 | 6.40 m/s |
| 3 | 5.12 m/s |
| 4 | 4.10 m/s |
| 5 | 3.28 m/s |
| 6 | 2.62 m/s |
| 7 | 2.10 m/s |
| 8 | 1.68 m/s |
| 9 | 1.34 m/s |
| 10 | 1.07 m/s |

> **Important:** This schedule is experimental and has not been successfully completed.
> An initial test at 8.0 m/s caused the tracking error to reach approximately 0.80 m within a few seconds. The safety mechanism stopped the vehicle.

### Current Safety Mechanisms

- Speed reduction when tracking error exceeds 0.25 m
- Emergency stop when tracking error reaches 0.80 m

These mechanisms prevent the vehicle from continuing to operate when it cannot reliably maintain the reference path. Further tuning and controlled speed testing are required before attempting the complete ten-lap schedule.

---

## 10. Building the ROS 2 Package

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

## 11. Running the F1TENTH Simulator

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

## 12. Running the Autonomous Controllers

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

## 13. Monitoring and Verification

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




## 14. Troubleshooting

During the development and testing of the F1TENTH controller, some common issues may occur.

The following table describes possible problems and basic solutions.

| Problem | Possible Cause | Suggested Solution |
|---|---|---|
| Simulator does not launch | Missing packages or incorrect environment setup | Check /ego_racecar/odom. |
| Vehicle does not move | Controller is not publishing commands | Check the `/drive` topic. |
| No LiDAR data | Simulator or sensor connection issue | Check whether the `/scan` topic is available. |
| No odometry commands | Simulator communication issue | Check the terminal for errors and restart the controller. |
| Vehicle stops unexpectedly | Safety mechanism is activated | Check tracking error and safety thresholds. |
| Vehicle moves in an unexpected direction | Incorrect steering calculation or parameter settings | Review the steering angle and controller configuration. |
| Vehicle losses the reference path | Excessive speed or unstable steering response | Reduce speed and tune the lookahead distance. |
| Python import errors | Missing dependencies or incorrect Python environment | Install the required dependencies and source the ROS 2 environment. |

## 15. Current Development Status

The following table summarizes the current progress of the project.

| Component | Status |
|---|---|
| F1TENTH Simulator Setup | ✅ Completed |
| Zandvoort Map Configuration | ✅ Completed |
| RViz2 Visualization | ✅ Completed |
| Manual Waypoint Generation | ✅ Completed |
| Path Interpolation | ✅ Completed |
| Pure Pursuit Path Tracking | ✅ Working |
| Full Lap at 1.0 m/s | ✅ Completed |
| Automatic Lap Detection | ✅ Implemented |
| Lap-Time Measurement | ✅ Implemented |
| Five Consecutive Laps | ✅ Completed |
| Speed Scheduling | 🧪 Implemented Experimentally |
| Tracking-Error Safety Stop | ✅ Implemented |
| Reliable High-Speed Tracking | ⚠️ Requires Improvement |
| Complete 10-Lap Run | ⏳ Pending |

---

## 16. Future Improvements

The next development stage focuses on improving the controller's reliability, stability, and overall racing performance.

Planned improvements:

- [ ] Improve high-speed path tracking
- [ ] Tune lookahead distance and steering response
- [ ] Test safe operating speeds systematically
- [ ] Improve speed adaptation around sharp corners
- [ ] Investigate waypoint progression and tracking-error behavior
- [ ] Complete a reliable ten-lap run
- [ ] Compare lap times under different speed schedules
- [ ] Optimize overall autonomous racing performance

---

## 17. Conclusion

This project explores autonomous vehicle navigation and racing using the **F1TENTH Gym Simulator**, **ROS 2 Humble**, and **Python**.

Development has progressed from simulator configuration and manual waypoint generation to autonomous path tracking, lap detection, performance measurement, and experimental speed management.

The Pure Pursuit controller has successfully completed a full lap at 1.0 m/s and five consecutive laps during an increasing-speed experiment.

The experiments also identified limitations in high-speed path tracking, highlighting the importance of steering stability, speed management, and safety mechanisms.

The overall goal is to develop a reliable autonomous racing controller capable of completing ten consecutive laps on the Zandvoort circuit while evaluating performance through vehicle velocity, lap times, and tracking stability.

> **Project Status:** Active Development
>
> **Current Focus:** Improving path-tracking stability, conducting controlled speed experiments, and working toward a reliable ten-lap autonomous racing run.
