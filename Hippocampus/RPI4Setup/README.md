# Installation Raspberry Pi 4 

Setup of Raspberry pi 4 for the Control of Hippocampus

What is installed in this guide:
* Ubuntu 18.04
* ROS Melodic
* Mavros and ROS-USB-Camera


What is necessary:
* RPI4
* Power Setup for RPI4
* Second Computer
* Basic understanding of Linux
* SD-Card at least 16gb
* RPI-Camera

# Ubuntu 18.04 Installation RPI4
The image of Ubuntu 18.04 64-bit can be downloaded [here](https://ubuntu.com/download/raspberry-pi).
[Etcher](https://www.balena.io/etcher/) is a great tool for copying the image on the SD-Card.

It is important that this version is only a server-version, which means there is no Graphical User Interface. Everything can be done using SSH.

After downloading and flashing the SD-Card the pi can be booted. Make sure to use a LAN cable the first time it is used.
First find out which IP Address the RPI has. This can be done with the help of the Router where the LAN cable is connected to.

Now you can log in with the terminal:

`ssh ubuntu@IP-address`

Which is in my example:

`ssh ubuntu@192.168.0.27`

now connect with typing `yes`
The password is `ubuntu` but has to change immediately. This standard Password we use is hippocampus.
Now re log in to the pi with the newly set password.
[Here](https://serverfault.com/questions/241588/how-to-automate-ssh-login-with-password) is a guide to connect to the PI4 without a password in the future. 

For overview reasons the command line can be optionally colored:

```
echo "PS1='\[\033[1;36m\]\u\[\033[1;31m\]@\[\033[1;32m\]\h:\[\033[1;35m\]\w\[\033[1;31m\]\$\[\033[0m\] '">> ~/.bashrc

source .bashrc
```

It should be run the following commands to update the software.

```
sudo apt update -y 
sudo apt upgrade -y
```


For the WIFI connection 
```
mising
```

# ROS-Melodic isntallation
The best way to install ROS is to follow this [guide](http://wiki.ros.org/melodic/Installation/Ubuntu) from the ROS website directly.
Use the Bare Bones installation since we are on a RPI4.

After setting up the Environment Setup and `source ~/.bashrc` ROS should be usable. Testing with `roscore`.


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
# Mavros and ROS-USB-Cam
To install the current version of mavros use:
```
sudo apt install ros-melodic-mavros* -y
```
To install ROS-USB-CAM use the following:
```
sudo apt install ros-melodic-usb-cam -y  
```

Additionally the camera has to be turned on on the PI:
use 
```
sudo nano /boot/firmware/config.txt
```
to open the config file and add in the last line `start_x=1`
Now `sudo reboot`

As a test if everything works correctly start the USB Camera package and see if you can see a picture.
First start `roscore` on the Master computer and then start the camera on the pi with:
```
roslaunch usb_cam usb_cam-test.launch
```
Dont worry about the error message in the first place.
Second open on your main computer `rqt_image_view ` and see if the video is streaming correctly.


# Installing additional packages
The best way to install different packages of ROS is always using apt. 
Sometimes it is not possible to do that. Additionally if we want to use our own packages an own workspace has to be compiled. 

Installing dependencies for compiling:

```
sudo apt-get install python-catkin-tools -y
sudo apt-get install -y python-rosdep python-rosinstall-generator python-wstool python-rosinstall build-essential -y
```
now create workspace:

```
cd ~
mkdir catkin_ws
cd catkin_ws
sudo rosdep init -y
rosdep update -y
cd src/
```
Now we install to State estimation tool with Apriltags:
```
git clone https://github.com/HippoCampusRobotics/mu_auv_localization.git
cd ..

rosdep install --from-paths src --ignore-src --rosdistro melodic -y
source ~/.bashrc
catkin build
```
rosdep always looks which dependencies are necessary and installs them directly.
Now additional packages can be installed. 
In the end the setup of the custom workspace has to be saved in the .bashrc script directly under the setup of the basic ROS.
```
sudo nano ~/.bashrc
source ~/catkin_ws/devel/setup.bash
```

