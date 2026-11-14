# F1TENTH-Zandvoort-Autonomous-Follow-the-Gap-Controller
## 1. Project Description:

This project implements an autonomous driving controller for the F1TENTH simulator. The controller uses LiDAR measurements to detect obstacles, identify free driving space and calculate steering and speed commands. The objective is to navigate the Zandvoort circuit autonomously while avoiding collisions.The project was developed as part of the Autonomous Vehicles course.

**Main Features**
-Autonomous driving using Lidar.
-Follow the gap reactive navigation.
-Obstacle detection.
-Disparity extension.
-Safety bubble implementation.
-Automotive steering Calculation.

## 2. Requirements

The following software and tools are required to run the autonomous driving controller.

| Software | Version |
|---|---|
| Ubuntu | 22.04 |
| ROS 2 | Humble |
| Python | 3.10 |
| NumPy | Compatible with ROS 2 Humble |
| F1TENTH Simulator | Official ROS 2 Simulator |
| RViz2 | ROS 2 Humble |


## 3. Repository Structure

The project uses the existing F1TENTH simulator workspace and a separate ROS 2 package for the autonomous Follow-the-Gap controller.

The intended project structure is shown below:

```text
F1Tenth-Repository/
│
├── src/
│   └── my_gap_controller/
│       ├── my_gap_controller/
│       │   ├── __init__.py
│       │   ├── gap_controller.py
│       │   └── follow_gap_zandvoort.py
│       │
│       ├── resource/
│       ├── package.xml
│       ├── setup.py
│       └── setup.cfg
│
├── f1tenth_gym_ros/
│   ├── config/
│   │   └── sim.yaml
│   └── maps/
│
├── build/
├── install/
└── log/
```

### Main Controller File

The main autonomous driving controller is:

`follow_gap_zandvoort.py`

It is responsible for processing LiDAR measurements, identifying available driving space, calculating steering angles, and controlling vehicle speed.

**Note:** This is the intended repository structure. The actual files and folder locations should be checked against the installed workspace.


### Install ROS 2 Ackermann Messages

```bash
sudo apt install ros-humble-ackermann-msgs
```

## 4. ROS 2 Communication

The autonomous controller communicates with the F1TENTH simulator through ROS 2 topics.

The following topics are used for communication between the simulator and the controller.

### ROS 2 Topics

| Topic | Message Type | Purpose |
|---|---|---|
| `/scan` | `sensor_msgs/msg/LaserScan` | Receives LiDAR measurements |
| `/drive` | `ackermann_msgs/msg/AckermannDriveStamped` | Sends steering and speed commands |
| `/ego_racecar/odom` | `nav_msgs/msg/Odometry` | Monitors vehicle position and movement |

### Communication Process

The communication between the simulator and the autonomous controller works as follows:

1. The simulator publishes LiDAR measurements through `/scan`.
2. The controller subscribes to the LiDAR topic.
3. The Follow-the-Gap algorithm processes the incoming measurements.
4. The controller calculates the appropriate steering angle and vehicle speed.
5. The resulting driving commands are published through `/drive`.
6. Vehicle odometry can be monitored through `/ego_racecar/odom` to observe the vehicle's movement and position.


## 5. Follow-the-Gap Algorithm

Follow-the-Gap (FTG) is a reactive obstacle-avoidance algorithm used for autonomous driving.

The algorithm uses LiDAR measurements to detect nearby obstacles, identify available free space, and select a safe driving direction.

The controller continuously processes sensor measurements and updates the steering and speed commands.

### Step 1 – LiDAR Preprocessing

The controller receives LiDAR measurements from the `/scan` topic.

The measurements are processed to:

- Handle invalid and infinite values.
- Restrict the detection area to the forward-facing region.
- Limit excessively large readings.
- Apply smoothing to reduce measurement noise.

This provides cleaner sensor data for obstacle detection.

### Step 2 – Closest Obstacle Detection

The controller identifies the closest obstacle within the relevant LiDAR field of view.

If an obstacle is detected within the emergency stopping distance, the controller reduces the vehicle speed or activates an emergency stop.

### Step 3 – Disparity Extension

Disparity extension identifies sudden changes between neighboring LiDAR measurements.

When an obstacle boundary is detected, nearby LiDAR points are adjusted to account for the vehicle's width and safety clearance.

This helps prevent the vehicle from selecting a path that is too narrow.

### Step 4 – Safety Bubble

A safety bubble is created around the closest obstacle.

The LiDAR points inside this safety region are excluded from gap selection.

This prevents the controller from choosing a driving direction that passes too close to an obstacle.

### Step 5 – Largest Free-Gap Detection

After obstacle regions have been removed, the controller searches for continuous regions of available free space.

These regions are called gaps.

The largest suitable gap is selected as the preferred driving region, provided it has sufficient width and clearance.

### Step 6 – Target Point Selection

A target point is selected within the chosen gap.

The target point represents the preferred direction of travel.

A smoothed or averaged selection method can be used to reduce sudden changes in the target direction and improve steering stability.

### Step 7 – Steering Angle Calculation

The selected target angle is converted into a steering command.

The steering angle is restricted to the maximum permitted steering limit of the vehicle.

This helps prevent excessively sharp steering commands.

### Step 8 – Adaptive Speed Control

The vehicle speed is adjusted according to the selected driving direction and surrounding obstacles.

- Higher speeds may be used on clear and relatively straight sections.
- Lower speeds are used when the steering angle increases.
- Speed is reduced when obstacles are detected nearby.
- The vehicle stops if no sufficiently safe driving direction is available.

Finally, the calculated steering and speed commands are published to the `/drive` topic.

The entire process repeats continuously while the simulator is running.

## 6. Initial Controller Configuration

The Follow-the-Gap controller uses several parameters to control vehicle movement, steering behavior, and obstacle avoidance.

The following values are the initial configuration settings for the simulation.

### Controller Parameters

| Parameter | Initial Value |
|---|---:|
| Maximum Speed | 0.8 m/s |
| Minimum Speed | 0.3 m/s |
| Maximum Steering Angle | 0.45 rad |
| Emergency Distance | 0.25 m |
| LiDAR Maximum Distance | 8.0 m |
| Forward Detection Angle | 1.50 rad |
| Minimum Gap Distance | 1.0 m |
| Vehicle Width | 0.30 m |
| Safety Margin | 0.20 m |


## 7. Installation and Environment Setup

Before running the autonomous controller, the F1TENTH workspace must be prepared and the required dependencies must be installed.

### Step 1 – Open the F1TENTH Workspace

Open a terminal and navigate to the project directory.

```bash
cd ~/F1Tenth-Repository
```

### Step 2 – Source ROS 2 Humble

Load the ROS 2 environment.

```bash
source /opt/ros/humble/setup.bash
```

### Step 3 – Install Required Dependencies

Update the package list.

```bash
sudo apt update
```

Install NumPy and Ackermann messages.

```bash
sudo apt install python3-numpy
sudo apt install ros-humble-ackermann-msgs
```

### Step 4 – Install ROS 2 Dependencies

If ROS dependencies have not already been installed, use:

```bash
sudo apt install python3-rosdep2
```

Initialize rosdep:

```bash
sudo rosdep init
rosdep update
```

Install the required dependencies:

```bash
rosdep install -i --from-path src --rosdistro humble -y
```

**Note:** If `rosdep init` has already been completed on the computer, do not repeat it.

### Step 5 – Build the Workspace

Navigate to the project directory and build the ROS 2 workspace.

```bash
cd ~/F1Tenth-Repository

source /opt/ros/humble/setup.bash

colcon build
```

### Step 6 – Source the Workspace

After a successful build, load the workspace environment.

```bash
source install/setup.bash
```

### Installation Verification

The workspace should build successfully without errors before running the autonomous controller.

This completes the initial environment setup required for the F1TENTH autonomous driving project.


## 8. Running the F1TENTH Simulator

The F1TENTH simulator must be running before launching the autonomous driving controller.

### Step 1 – Open Terminal 1

Open a new terminal and source ROS 2 Humble.

```bash
source /opt/ros/humble/setup.bash
```

### Step 2 – Source the F1TENTH Workspace

Load the workspace environment.

```bash
source ~/F1Tenth-Repository/install/setup.bash
```

### Step 3 – Launch the Simulator

Run the following command to start the F1TENTH simulator.

```bash
ros2 launch f1tenth_gym_ros gym_bridge_launch.py
```

### Step 4 – Wait for the Simulator to Start

Wait until the simulator launches successfully and the vehicle appears in the simulation environment.

### Step 5 – Verify the Zandvoort Map

The Zandvoort map should be selected in the simulator configuration file:

`f1tenth_gym_ros/config/sim.yaml`

The map image and its corresponding YAML configuration must be correctly installed and referenced.

### Expected Result

After successful initialization:

- The F1TENTH simulator should open.
- The vehicle should appear in the simulation environment.
- The selected track map should be loaded.
- The simulator should be ready to receive ROS 2 communication from the autonomous controller.

**Note:** Keep Terminal 1 running while using the simulator. Do not close it before starting the autonomous controller.


## 9. Running the Autonomous Controller

After successfully launching the F1TENTH simulator, the autonomous Follow-the-Gap controller can be started in a separate terminal.

### Step 1 – Open Terminal 2

Open a new terminal and source ROS 2 Humble.

```bash
source /opt/ros/humble/setup.bash
```

### Step 2 – Source the F1TENTH Workspace

Load the workspace environment.

```bash
source ~/F1Tenth-Repository/install/setup.bash
```

### Step 3 – Run the Autonomous Controller

Execute the Follow-the-Gap Python controller.

```bash
python3 ~/F1Tenth-Repository/src/my_gap_controller/my_gap_controller/follow_gap_zandvoort.py
```

### Step 4 – Controller Initialization

After running the command, the controller should initialize its ROS 2 node and begin waiting for LiDAR measurements.

Expected terminal messages:

```text
ZANDVOORT FOLLOW-THE-GAP CONTROLLER

Waiting for LiDAR measurements...
```

### Step 5 – Autonomous Driving

Once LiDAR measurements are received, the controller begins processing sensor data and calculating steering and speed commands automatically.

The vehicle should then navigate using the Follow-the-Gap algorithm.

### Important Notes

- The F1TENTH simulator must remain running in Terminal 1.
- The autonomous controller runs in Terminal 2.
- No manual driving commands should be active simultaneously.
- The controller requires incoming LiDAR measurements to operate.

The controller continuously processes sensor information and publishes driving commands through the `/drive` topic.


## 10. Monitoring and Verification

After starting the simulator and autonomous controller, ROS 2 commands can be used to monitor communication and verify the vehicle's behavior.

### Step 1 – Check LiDAR Measurements

Open a new terminal and run:

```bash
source /opt/ros/humble/setup.bash

ros2 topic hz /scan
```

This command checks whether the LiDAR sensor is publishing measurements and displays its publishing frequency.

### Step 2 – Monitor Vehicle Driving Commands

To observe the steering and speed commands published by the controller, run:

```bash
ros2 topic echo /drive
```

The output should contain steering and speed values.

Example output:

```yaml
drive:
  steering_angle: 0.12
  speed: 0.65
```

The values change according to the LiDAR measurements and the selected driving direction.

### Step 3 – Check Vehicle Odometry

To monitor the vehicle's position and movement, run:

```bash
ros2 topic echo /ego_racecar/odom
```

This command displays odometry information, including vehicle position and velocity.

### Step 4 – Check Driving Topic Information

To inspect the `/drive` topic and its communication details, run:

```bash
ros2 topic info /drive --verbose
```

This command helps verify whether the autonomous controller is publishing driving commands.

### Verification

The following conditions should be checked:

- LiDAR measurements are being received.
- The controller publishes steering and speed commands.
- The vehicle responds to the published commands.
- Odometry information is available.
- ROS 2 communication is functioning correctly.

Successful monitoring helps confirm that the simulator and autonomous controller are communicating properly.


## 11. Stage 1 – Basic Controller Development and Safety Testing

The first development stage focuses on checking the basic operation of the Follow-the-Gap controller in the F1TENTH simulator.

The following tests are planned to verify the controller's basic functionality and safety behavior.

### Testing Checklist

- [ ] Confirm that the F1TENTH simulator launches successfully.
- [ ] Confirm that LiDAR measurements are being received from the `/scan` topic.
- [ ] Start the Follow-the-Gap controller and check for Python or ROS 2 errors.
- [ ] Confirm that the controller publishes steering and speed commands to the `/drive` topic.
- [ ] Check whether the steering direction responds to the detected free space.
- [ ] Check whether the vehicle reduces speed during sharp turns.
- [ ] Test the vehicle's response when an obstacle is detected nearby.
- [ ] Check whether the controller stops the vehicle when no safe driving gap is detected.

### Expected Result

The expected result of this stage is a controller that can receive LiDAR data, identify a safe driving direction, and publish appropriate steering and speed commands.

These tests will help identify basic controller issues before proceeding to autonomous lap testing.


## 12. Stage 2 – Zandvoort Autonomous Lap Testing

After completing the basic controller tests, the next stage is to test autonomous driving on the Zandvoort racing track.

The main goal of this stage is to check whether the vehicle can follow the racing track continuously and complete laps without collisions.

### Planned Testing Steps

- [ ] Confirm that the Zandvoort map is loaded correctly in the simulator.
- [ ] Check the vehicle's initial position and orientation.
- [ ] Start the simulator and the Follow-the-Gap controller.
- [ ] Begin testing at a low speed to observe the vehicle's movement.
- [ ] Check whether the vehicle follows the track boundaries safely.
- [ ] Adjust controller parameters if the vehicle moves unpredictably.
- [ ] Gradually increase the speed after basic stability is achieved.
- [ ] Test whether the vehicle can complete one full lap.
- [ ] Record the vehicle's velocity and lap time during testing.

### Expected Result

The expected result is an autonomous vehicle that can follow the Zandvoort racing track, complete laps safely, and provide basic performance measurements such as velocity and lap time.

This stage will be carried out after the basic controller and safety tests are completed.



## 13. Troubleshooting

During the development and testing of the F1TENTH controller, some common issues may occur.

The following table describes possible problems and basic solutions.

| Problem | Possible Cause | Suggested Solution |
|---|---|---|
| Simulator does not launch | Missing packages or incorrect environment setup | Check the ROS 2 installation and required packages. |
| Vehicle does not move | Controller is not publishing commands | Check the `/drive` topic. |
| No LiDAR data | Simulator or sensor connection issue | Check whether the `/scan` topic is available. |
| No `/drive` commands | Controller is not running correctly | Check the terminal for errors and restart the controller. |
| Vehicle stops unexpectedly | Emergency stop condition is activated | Check the LiDAR readings and safety distance settings. |
| Vehicle moves in an unexpected direction | Incorrect steering calculation or parameter settings | Review the steering angle and controller configuration. |
| Python import errors | Missing dependencies or incorrect Python environment | Install the required dependencies and source the ROS 2 environment. |

### Useful ROS 2 Commands

Check available topics:

```bash
ros2 topic list
```

Check LiDAR data:

```bash
ros2 topic echo /scan
```

Check vehicle commands:

```bash
ros2 topic echo /drive
```

Check odometry data:

```bash
ros2 topic echo /ego_racecar/odom
```

### Note

If a problem continues, check the terminal output and ROS 2 topic information to identify where the issue is occurring.


## 14. Current Development Scope and Future Work

### Current Development Scope

The current project focuses on understanding and setting up the F1TENTH simulation environment and exploring the Follow-the-Gap algorithm for autonomous vehicle navigation.

The main areas of work include:

- Understanding the ROS 2 communication system.
- Exploring LiDAR data through the `/scan` topic.
- Understanding vehicle control commands through the `/drive` topic.
- Studying the basic principles of the Follow-the-Gap algorithm.
- Preparing the controller for simulation-based testing.

### Future Work

The following tasks are planned for further development:

- Complete basic controller and safety testing.
- Improve steering and speed control.
- Test autonomous driving on the Zandvoort racing track.
- Work toward completing full laps without collisions.
- Measure and record vehicle velocity and lap time.
- Adjust controller parameters to improve driving stability and performance.

### Project Goal

The long-term goal is to develop a functional autonomous racing controller that can safely navigate the Zandvoort track and evaluate its performance through lap completion, velocity, and lap-time measurements.


## 15. Conclusion

This project explores the development of an autonomous racing controller using the F1TENTH simulation environment, ROS 2, and the Follow-the-Gap algorithm.

Through this project, the main focus is on understanding LiDAR-based obstacle detection, ROS 2 communication, and basic vehicle control.

The project is being developed step by step, starting with the simulation environment and basic controller setup, followed by safety testing and autonomous lap testing on the Zandvoort track.

The overall goal is to gain practical experience with autonomous vehicle navigation and improve the controller's driving stability and performance through testing and further development.

