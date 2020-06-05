# How to Drive an 8 with Hippocampus in Gazebo and Apriltag-EKF

Requirement:
* The [PX4 Firmware](https://github.com/PX4/Firmware) works with ROS-Melodic and Gazebo (the default IRIS version)
* Mavros is installed


Wir nehmen hier folgendes an: Die Firmware von PX4  wurde installiert und Funktioniert mit ROS (Guides gibt es dort dafuer)+Mavros. Die IRIS Standard Welt reicht.
Cloning of Firmware from HippoCampusRobotics:
```
cd ~
mkdir src
cd src
git clone https://github.com/HippoCampusRobotics/Firmware.git hippocampus_firmware
cd hippocampus_firmware
make px4_sitl_default gazebo_uuv_hippocampus
```
Now the Gazebo should run without a Hippocampus.
Again with ROS and IRIS Quadrocopter:
```
DONT_RUN=1 make px4_sitl_default gazebo_uuv_hippocampus

source Tools/setup_gazebo.bash $(pwd) $(pwd)/build/px4_sitl_default
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:$(pwd)
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:$(pwd)/Tools/sitl_gazebo
roslaunch px4 posix_sitl.launch
```
Now IRIS should be visible and with `rostopic list` a list of different Topics.

For the state estimation with the April-tags first the world has to be changed and the vehicle correct vehicle has to be spawned.
In `launch/posix_sitl.launch` the following lines have to be replaced:
```
<arg name="vehicle" default="iris"/>
<arg name="world" default="$(find mavlink_sitl_gazebo)/worlds/empty.world"/>
<arg name="sdf" default="$(find mavlink_sitl_gazebo)/models/$(arg vehicle)/$(arg vehicle).sdf"/>
```
With:
```
<arg name="vehicle" default="uuv_hippocampus"/>
<arg name="world" default="$(find mavlink_sitl_gazebo)/worlds/uuv_hippocampus.world"/>
<arg name="sdf" default="$(find mavlink_sitl_gazebo)/models/$(arg vehicle)/$(arg vehicle).sdf"/>
```

In `Tools/sitl_gazebo/worlds/uuv_hippocampus.world` the lines:
```
<include>
    <uri>model://uuv_apriltag_tank</uri>
    <pose>1.62 3.27431 0 0 0 3.1415</pose>
</include>
```

have to be added after the global light source.
with `roslaunch px4 posix_sitl.launch` the April tag World should be visible. 
The localization and State Estimation is possible with an EKF.
This can be installed in your catkin_ws with:

```
cd ~/catkin_ws/src
git clone https://github.com/HippoCampusRobotics/mu_auv_localization.git
git clone https://github.com/HippoCampusRobotics/hippocampus_common.git
cd .. 
rosdep install --from-paths src --ignore-src --rosdistro melodic -y
source ~/.bashrc
catkin build
source ~/.bashrc
```

With:
```
roslaunch mu_auv_localization gazebo_localization.launch
```
The State estimation can be started. 

The Estimation starts also:
* Mavros
* Image Rectification
* ENU to NED (Mavros to ROS Conversion)
* Barometer for Gazebo
* Apriltag Algorithm
* EKF Node (The actual State Estimation)

With the script `driving_infinity.py`, on this side the uuv can drive an infinity.

With Qgroundcontrol(QGC) the UUV_INPUT_MODE has to be changed on 3 and EKF2_AID_MASK to 24.
Offboard mode and arming is required.
Force Arming can be done with the following command:
```
rosservice call /mavros/cmd/command "{broadcast: false, command: 400,confirmation: 0, param1: 1, param2: 21196, param3: 0.0, param4: 0.0,param5: 0.0, param6: 0.0, param7: 0.0}"
```


