# ROS Robotics Simulator <img src="https://media4.giphy.com/media/6GgbijcRpz4MQbshdW/giphy.gif?cid=ecf05e47p45gw24cpa3igtszy60a6lmsqupd3wssnphnpd7f&rid=giphy.gif&ct=s" width="50"></h2>
## Second Assignment of the course [Research_Track_1](https://unige.it/en/off.f/2021/ins/51201.html?codcla=10635) , [Robotics Engineering](https://courses.unige.it/10635).
###  Professor. [Carmine Recchiuto](https://github.com/CarmineD8).

-----------------------

This project makes use of a set of software libraries [__ROS__](http://wiki.ros.org) (__Robot-Operating-Systems__) to build a robot application that consists in making a robot run autonomously in the Monza F1 circuit.

This assignment aims to get in touch with ROS, the use of packages, nodes and standard ways to make those nodes communicate such as messages and services.

In Particular, I created two nodes: 
* The first one to control the movement of the robot in the environment.
* The second one to create a basic UI (Uman-Interface) to increase or decrease, run-time, the velocity of the robot and also to reset it to its starting position.

Installing and running <img src="https://media3.giphy.com/media/LwBuVHh34nnCPWRSzB/giphy.gif?cid=ecf05e47t4j9mb7l8j1vzdc76i2453rexlnv7iye9d4wfdep&rid=giphy.gif&ct=s" width="50"></h2>
-----------------------

In order to run more than one node at the same time i created a `.launch` file named `run.launch`.

__`run.launch`__ : 
```xml
<launch>
	
	<node name="carcontroller_node" pkg="second_assignment" type="carcontroller_node" required="true"/>
	
	<node name="UI_node" pkg="second_assignment" type="UI_node" output="screen" launch-prefix="xterm -fg white -bg black -e "  required="true"/>
	
	<node name="stageros" pkg="stage_ros" type="stageros" required ="true" args = "$(find second_assignment)/world/my_world.world"/>

</launch>
```

In order to open the UI_node in a new terminal console I installed and used xterm which is a standard terminal emulator build for Unix-like environments.

__Command to Install xterm__:

```bash
	sudo apt-get install xterm
```

The user can easily run all the nodes I created and also the stageros node that opens the enviroment and also provides a list of usable messages and services. 

__Command to launch the project__:

```bash
	roslaunch second_assignment run.launch
```
where `second_assignment` is the name of the ros package I created for this assignment.

StageRos_Node ########image########
-------------
The stageros node wraps the Stage 2-D multi-robot simulator, via libstage. Stage simulates a world as defined in the
`my_world.world` file in the folder world. This file tells Stage everything about the environment, in this project the world is passed to Stage via a `.png` image which represents the Monza Formula 1 circuit.

__Enviroment__

#########image#########

The robot which will be moved in the circuit is also defined in the `.world` file and elaborated by the stageros node. It's a simple red cube.

__robot__

#########image#########

A very handy feature of the Stageros Node is its subscription to the topic `cmd_vel` from `geometry_msgs` package which provides a [`Twist`](https://docs.ros.org/en/api/geometry_msgs/html/msg/Twist.html) type message to express the velocity of a robot in free space, broken into its linear and angular parts.

The Stageros Node also publishes on the `base_scan` topic from `sensor_msgs` package which provides a [`LaserScan`](https://docs.ros.org/en/api/sensor_msgs/html/msg/LaserScan.html) type message to express a single scan from a planar laser range-finder. 

Besides that, I also used a standard service called `reset_position` from the `std_srvs` package. 
`std_srvs` contains a type of service called [`Empty`](https://docs.ros.org/en/api/std_srvs/html/srv/Empty.html) where no actual data is exchanged between the service and the client, but has proved very useful for restoring the robot's position to its starting point.

UI_Node <img src="https://media0.giphy.com/media/jQzFUZrBsZ6wse4RH1/giphy.gif?cid=ecf05e47cmge9t3j75v23at26fs7uii5ru9lpmsvpm506o0q&rid=giphy.gif&ct=s" width="50"></h2>
-------

This node manages the UI ( __USER INTERFACE__ ) of the project. The user can control the speed of the robot by increasing and decreasing its acceleration. He can also reset the robot's position to its initial state.
The UI node receives input from a terminal window.
The keyboard buttons that can be pressed are:

* [a] to accelerate
* [d] to decelerate 
* [r] to reset the position  

##########image###########

As soon as an input arrives from outside, it is transmitted to the Controller_Node, which in turn responds by sending back the robot's degree of acceleration.
This client-server communication system has been implemented by creating a custom service called Accelerate.srv . 
The structure of the service is:
``` xml
     char input
     ---
     float32 value
```
Where:
* char input is the character typed by the user on the keyboard: [a],[d] or [r].

* float32 value is the degree of acceleration of the robot that is transmitted as a response from the server to the client. 

Controller_Node <img src="https://media2.giphy.com/media/LMQ5h5QFw7olMCaYpm/giphy.gif?cid=ecf05e47sontjyw8wcsovww1576ckg25lp4ouoxy8musu8la&rid=giphy.gif&ct=s" width="50"></h2>
---------------

The Controller_Node is the core of the logic for the robot's movement in the circuit, but it's also the handler of the inputs coming from the UI_Node. 

In this Node, implemented in controller.cpp in the folder src, there are 3 main functions:

 * __Wall_detection(range_min, range_max , Laser_Array)__

	I designed this function to calculate the minimum distance between the robot and the walls. This function enables the sensor to detect obstacles in a visual cone between the range_min and the range_max argument, not expressed in degrees but in the number of elements of an array of 721 elements. The array comes from the `base_scan` topic and simply returns the distances between the robot and the circuit contours at a 180° angle in front of it.

   	`Arguments` :

   	* range_min (`int`) : the lower end of the array Laser_Scan 

   	* range_max (`int`) : the top end of the array Laser_Scan 

   	* Laser_Scan (`float`) : the array of 721 elements representing the 180° field of view in front of the robot.

   	`Returns` :

   	* low_value (`float`) : the minimum value of the array, between the two bounds range_min and range_max, that represent the minimum distance between the robot and the walls in the interval considered.

* __Rotation_Callback__

I have implemented a Callback function that will be called whenever a message is posted on the `base_scan` topic.

Thanks to the function Wall_detection(), the robot can detect the shortest distance to the walls on its right, left and front :

``` C
	dist_right = Wall_Detection(0,100,Laser_Array); // detecting the shortest distance to wall on its right 
	
	dist_left = Wall_Detection(620,720,Laser_Array); // detecting the shortest distance to wall on its left 
	
	dist_front = Wall_Detection(310,410,Laser_Array); // detecting the shortest distance to wall on its front

```

The Logic implemented to move the robot in the track is similar to the first assignment of this course [__Assignment_1__](https://github.com/MatteoCarlone/RT1_Assignment_1) :

First, the robot checks for the presence of a wall in front of it at a distance of less than 1.5, if there is not it will move straight ahead by publishing a linear velocity on the "cmd_vel" topic which, by default, is equal to 2, otherwise it will detect walls to its left and right.
If the wall is closer to the robot's right, it will turn left and vice versa.

##########GIF###########

* __Server__

This function 





 


