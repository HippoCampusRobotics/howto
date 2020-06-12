# How to Set up Raspberry OS Buster with ROS

Setup of Raspberry pi 4 for the Control of Hippocampus

What is installed in this guide:
* Raspberry OS Buster
* ROS Melodic
* Mavros and ROS-USB-Camera


What is necessary:
* RPI4
* Power Setup for RPI4
* Second Computer
* Basic understanding of Linux
* SD-Card at least 16gb
* RPI-Camera


# Raspberry OS Buster
The image of Raspberry OS Buster can be downloaded [here](https://www.raspberrypi.org/downloads/raspberry-pi-os/).
[Etcher](https://www.balena.io/etcher/) is a great tool for copying the image on the SD-Card.

Everything can be done using SSH.
When possible use a Mouse and keyboard together with an HDMI Cable

Else the SD-Card image has to be Modified to use SSH immediately, because it is turned off by default.
This can be done by adding “SSH” File to the SD Card Root.

The pi can be booted now. Make sure to use a LAN cable the first time it is used.
Find out which IP Address the RPI has. This can be done with the help of the Router where the LAN cable is connected to.

Now you can log in with the terminal:

`ssh pi@IP-address`

Which is in my example:

`ssh pi@192.168.0.27`

now connect with typing `yes`
The password is `raspberry` but can be changed.
[Here](https://serverfault.com/questions/241588/how-to-automate-ssh-login-with-password) is a guide to connect to the PI4 without a password in the future. 

For overview reasons the command line can be optionally colored:

```
echo "PS1='\[\033[1;36m\]\u\[\033[1;31m\]@\[\033[1;32m\]\h:\[\033[1;35m\]\w\[\033[1;31m\]\$\[\033[0m\] '">> ~/.bashrc

source .bashrc
```

It should be run the following commands to update the software. After the first installation.

```
sudo apt update -y 
sudo apt upgrade -y
```

# ROS-Melodic isntallation
ROS has to be installed from source for RPI PI

This can be done with the following commands:

Get the correct KEY and update accordingly:
```
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
sudo apt-get update -y
sudo apt-get upgrade -y
```
Install catkin tools and essential tools necessary:
```
sudo apt-get install python-catkin-tools -y
sudo apt-get install python-rosdep python-rosinstall-generator python-wstool python-rosinstall build-essential -y
```

Init rosdep to install Dependencies in the future:
```
sudo rosdep init -y
rosdep update -y
```
Create Workspace and download the source of Ros-Barebones, usb_cam mavros and image_proc:
```
mkdir -p ~/catkin_ws
cd ~/catkin_ws
rosinstall_generator ros_comm usb_cam image_proc mavlink mavros mavros_extras --rosdistro melodic --deps --wet-only --tar > melodic-ros_comm-wet.rosinstall
wstool init src melodic-ros_comm-wet.rosinstall
```
Install Dependencies of the source:
```
rosdep install -y --from-paths src --ignore-src --rosdistro melodic -r --os=debian:buster
```
Compile ROS and source everything:
```
catkin build
echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

# Network Setup
It is important that every client in the ROS network knows who the other clients are. Which means they have to know the name and ip-address
This can be done in /etc/hosts which can be opened with `sudo nano /etc/hosts`
For me the following two lines are added:
```
192.168.0.23 tim-linux
192.168.0.27 pi
```
First the main computer and second the pi itself(change names accordingly)
Next is the ROS_HOSTNAME and ROS_MASTER_URI again open with `sudo nano ~/.bashrc` and put in the last lines:
```
export ROS_HOSTNAME="pi"
export ROS_MASTER_URI="http://tim-linux:11311"
```