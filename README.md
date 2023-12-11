# Creating a ROS Package for Pepper

This tutorial creates an example ROS package that reads Pepper’s joint states. The files included in this repo are the final product that should be created by following this tutorial.

**Pre-Requisite:** The catkin workspace and Naoqi Driver should be set up. This can be done by following the `Direct Installation` steps found in [ROS to NAO Bridge Setup Instructions](https://github.com/rosielab/ROStoNAO-Bridge-Docker-Setup).

## 1. Start ROS 
Open a terminal.

Make sure to source:
```
source /opt/ros/noetic/setup.bash
```

Launch ROS:
```
roscore
```

## 2. Create a Package Directory 

Steps taken from [ROS Creating Package Tutorial](https://wiki.ros.org/ROS/Tutorials/CreatingPackage).

In a new terminal window create the ros package directory
```
cd ~/catkin_ws/src
catkin_create_pkg beginner_tutorials std_msgs rospy roscpp
cd ~/catkin_ws
catkin_make
. ~/catkin_ws/devel/setup.bash
```

To view dependencies within the project you can run:
```
rospack depends1 beginner_tutorials # List of dependencies for 'beginner_tutorials'
rospack depends1 rospy # List of dependencies for 'ros-py'
rospack depends beginner_tutorials # Recursive list of dependencies for 'beginner_tutorials'
```

## 3. Creating ROS Nodes
Steps taken from [Simple Publisher and Subscriber (Python)](https://wiki.ros.org/ROS/Tutorials/WritingPublisherSubscriber(python)).

### i) Writing the Publisher Node

Run the following commands to create the node script and get the talker file from ROS tutorials:
```
roscd beginner_tutorials
mkdir scripts
cd scripts
wget https://raw.github.com/ros/ros_tutorials/kinetic-devel/rospy_tutorials/001_talker_listener/talker.py
chmod +x talker.py
```

Modify the `CMakeLists.txt` file to include: 
```
catkin_install_python(PROGRAMS scripts/talker.py
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
```

### ii) Writing the Subscriber Node

Run the following commands to create the node script and get the listener file from ROS tutorials:
```
roscd beginner_tutorials/scripts/
wget https://raw.github.com/ros/ros_tutorials/kinetic-devel/rospy_tutorials/001_talker_listener/listener.py
chmod +x listener.py
```

Modify the `catkin_install_python` in `CMakeLists.txt` file to be: 
```
catkin_install_python(PROGRAMS scripts/talker.py scripts/listener.py
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
```

### iii) Test Nodes are Working 

Run these commands to start the talker:
```
catkin_make
rosrun beginner_tutorials talker.py
```

You should begin getting messages similar to this being output to the terminal window:
```
[INFO] [WallTime: 1314931831.774057] hello world 1314931831.77
[INFO] [WallTime: 1314931832.775497] hello world 1314931832.77
[INFO] [WallTime: 1314931833.778937] hello world 1314931833.78
```


In a new terminal window run these commands to start the listener:
```
rosrun beginner_tutorials listener.py
```
You should begin getting messages similar to this being output to the terminal window:
```
[INFO] [WallTime: 1314931969.258941] /listener_17657_1314931968795I heard hello world 1314931969.26
[INFO] [WallTime: 1314931970.262246] /listener_17657_1314931968795I heard hello world 1314931970.26
[INFO] [WallTime: 1314931971.266348] /listener_17657_1314931968795I heard hello world 1314931971.26
```

## 4. Link to NaoQi Bridge
Full instructions can be found in [ROS to NAO Bridge Setup Instructions (Steps 5-6)](https://github.com/rosielab/ROStoNAO-Bridge-Docker-Setup)

### i) Connect to Pepper's Network

Ensure device and Pepper are connected to the same network (e.g., NETGEAR23-5G)

### ii) SSH into Pepper

In a new terminal window run:
```
ssh nao@<naoip>
```

### iii) Launch Naoqi Driver
In a new terminal window run:
```
roslaunch naoqi_driver naoqi_driver.launch nao_ip:=<nao_ip> nao_port:=<nao_port> roscore_ip:=<roscore_ip> network_interface:=<network_interface> username:=<username> password:=<password>
```

For example this could be:
```
roslaunch naoqi_driver naoqi_driver.launch nao_ip:=10.0.0.4 nao_port:=9503 roscore_ip:=127.0.0.1 network_interface:=eth0 username:=nao password:=nao
```

## 5. Modify the subscriber nodes to read joint states from pepper
To view a list of pepper’s topics run:
```
rostopic list
```

To view current joint states (to ensure they are being published from pepper):
```
rostopic echo /joint_states
```
To learn the data type of the joint states (this is needed to modify our Python file):
```
rostopic info /joint_angles
```

In our `listener.py` add the following import statement (we learned it should be sensor_msgs and JointState from the previous step):
```
from sensor_msgs.msg import JointState
```

In `listener.py` modify the subscriber line to be:
```
rospy.Subscriber('joint_states', JointState, callback)
```

In `listener.py` modify the callback to be:
```
rospy.loginfo(rospy.get_caller_id() + 'I heard %s', data)
```
## Run ROS Node Linked to Pepper

Run the listener:
```
rosrun beginner_tutorials listener.py
```

You should receive output of Pepper's joint states to that terminal window.
