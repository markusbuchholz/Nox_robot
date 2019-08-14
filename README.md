<a href="url"><img src="/docs/images/preview1.png" align="center" height="300" width="405"></a>

# Nox Robot project
Nox is a nice (and time-consuming for me) robot which uses SLAM (ROS) with a Kinect to navigate in its environment. It is powered by ROS running on a Raspberry Pi 3B and an Arduino that controls two motors with encoders.

<a href="url"><img src="/docs/images/picture1.jpg" align="center" height="300" width="400"></a>
<a href="url"><img src="/docs/images/picture2.jpg" align="center" height="300" width="400"></a>

In its current state the robot can use SLAM (gmapping) to create a map of its surroundings (using the Kinect depth perception to detect wall and obstacles) and localize itself within the map. It can plan a path to a given goal and drive to it avoiding obstacles.

## Nox characteristics
Nox is a differential drive robot with the motors placed on the same axis. The base is made of wood with two caster wheels for support. The rest of the structure is made mainly of wood and metal brackets, easy to find in any DIY retail store. On the rear part of the robot plates can be stacked to put the electronic boards on.
More information (such as components used) can be found in the [Hackster page of the project](https://www.hackster.io/robinb/nox-a-house-wandering-robot-ros-652315).

### Schematics
<a href="url"><img src="/docs/images/Nox_Schematic.jpg" align="center" height="800" width="800"></a>

### ROS packages
* #### nox_description:
This package includes the URDF description of the robot and the associated CAD files for display in RViz. The launch file "nox_rviz.launch" launches the joint and robot state publisher as well as RViz to display the model.
* #### nox:
This package includes the base controller node "nox_controller.cpp". This node will convert velocity command data into proper command data for the robot (according to the robot kinematic model) and send it to the Arduino Mega (through ROSSerial). The package also includes several launch files used to start the navigation (see "How to use Nox" below).
* #### Arduino:
The arduino node receives the command order from the base controller node and commands the motor speeds. It also computes the odometry data (from the encoder data and according to the robot kinematic model) and send it back to other nodes through ROSSerial.


## How to use Nox
### Packages installation
The software for the Nox project was developped with ROS Kinetic and Ubuntu 16.04. More recent versions should work as well but might require some tweaking. This readme doesn't explain how to install ROS and packages. For more information you can check the [ROS tutorials](http://wiki.ros.org/ROS/Tutorials).

#### To use Nox you will need the following packages (most of them should already be installed by default or requested when building the nox packages):
* The [navigation stack](https://wiki.ros.org/navigation),
* The freenect package (for connecting to the Kinect)
* [RViz](http://wiki.ros.org/rviz)
* [TF](http://wiki.ros.org/tf), [Joint State](http://wiki.ros.org/joint_state_publisher) and [Robot State](http://wiki.ros.org/robot_state_publisher) Publishers
* [ROSSerial package](http://wiki.ros.org/rosserial) (for connecting to the Arduino Mega)
* [SLAM gmapping](https://wiki.ros.org/slam_gmapping).

#### Installation
You can install and build the package by copying the "nox" and "nox_description" folders in "your_catkin_workspace/src" and running:
  ```
  catkin_make
  ```

## Running Nox

In order to navigate Nox you will need to start two launch files using SSH and [ROS nodes over network](http://wiki.ros.org/ROS/NetworkSetup).
### 1. Connect to Nox Wi-Fi
I haven't included in my GitHub the RPi network configuration but basically the RPi creates an Ad-Hoc network I connect to with my computer in order to send SSH commands. It's also possible to connect both the Rpi and the computer to the same WiFi network. Once connected and before launching any ROS command you should sync the RPi to the computer using the following code in RPi:

`sudo ntpdate ip_address_of_the_computer`

You can also setup the ntpdate or use chrony to do that automatically.

### 2. Start the odometry and motor control
In the RPi, use the following command to start the main program:

`roslaunch nox nox_bringup.launch`

It will launch the serial controller node connected to the Arduino, the Kinect node, the odometry calculation node and the joint state publisher. If this step succeded you should see the side light of the robot going from a series of three quick blinks to a slow "breathing-like" type of blinking.

### 3. Start the navigation
In order to be able to do path planning calculation and mapping you have to use a launch file on your computer this time. Before doing so, assure yourself that you exported the ROS master from the RPi (modify the command to suit your own setup):

`export ROS_MASTER_URI=http://localhost:11311`

You can then launch:

`roslaunch nox nox_slam.launch`

It will launch the move_base node and RViz. By using the "2D Nav Goal" arrow you can give a goal for the robot to reach. Alternatively, you can use the ROS teleop keyboard to control the robot:

`rosrun teleop_twist_keyboard teleop_twist_keyboard.py` 
