UR10e-ROS-setup-with-MOVEIT
Universal Robot (UR) is one of the most popular and used robot worldwide. This short tutorial explains and details the procedures of installing UR10e ROS robot driver in the Ubuntu system, and how to control the robot remotely using TCP/IP communication. 

! ![image](https://user-images.githubusercontent.com/7438736/113154392-e2231400-922f-11eb-8810-e0e610833b1a.png)

Picture 1: The UR10e robot in our lab

Recommended requirements:
1.	Ubuntu 20.04 .2.0 LTS (Focal Fossa)
2.	ROS Noetic (targeted for Ubuntu Focal release)
3.	Polyscope v.5.9.3 (URSoftware)

The official documentation of installing ROS driver for the UR is described at https://github.com/UniversalRobots/Universal_Robots_ROS_Driver, however it could be bit tedious and difficult to understand time to time. In this document the most specific version for the current robot and system has been described in following sections:
1.	Setting up URCap on UR10e.
2.	Setting up TCP/IP connection. 
3.	Installing drivers and testing. 



1.	Setting up URCap on the UR10e:
URCaps is a platform where distributors and integrators can present accessories that run successfully in UR robot applications at end users. Caps are to robots, what apps are to smartphones: useful accessories, hardware and software extending the capabilities of UR robots handling many different tasks. 
!![image](https://user-images.githubusercontent.com/7438736/113154565-07178700-9230-11eb-9b1c-941835ebdff8.png)
	 
Picture 2: Polyscope URCap section

URCap is important for the remote control connection with the robot, and to setup the URCap first a registration of the robot is necessary. Registration is fairly simple at the universal-robot.com/register website. After the registration a file will be generated which needs to be download and put into a USB dive. This file needs to ne sent to the Polyscope using ‘Open’ file and load options, and once done the ‘Robot Registration’ page will show the updated registration information and the robot is ready for installing URCap. 
To move further, first you need to download the UR robot driver directory from the https://github.com/ros-industrial/universal_robot repository. 
For using the ur_robot_driver you need to install the ‘externalcontrol-1.0.4.urcap’ on the Polyscope itself, and it can be found inside the resources folder of the downloaded driver. To install it you first need to copy it to the robot's programs folder which can be done with using a USB stick. 
After putting the file in USB just plug it into the Polyscope’s USB slot and switch it on. As seen in the Picture 3, go to the ‘Settings’ tab and simply click the little plus sign at the bottom to open the file selector. There you should see all URCcap files stored inside the plugged USB drive. Select and open the externalcontrol-1.0.4.urcap file and click open. Your URCaps view should now show the External Control in the list of active URCaps and a notification to restart the robot. 
 !![image](https://user-images.githubusercontent.com/7438736/113155611-1e0aa900-9231-11eb-998a-71df7256ce82.png)

Picture 3: Setting up the URCap for the first time

After the reboot you should find the External Control URCaps inside the Installation section. For this select Program Robot on the welcome screen, select the Installation tab and select External Control from the list. Here you need to setup the IP address of the external PC (see Picture 4) which will be running the ROS driver.
! ![image](https://user-images.githubusercontent.com/7438736/113155717-3b3f7780-9231-11eb-933a-45b0439f54d8.png)

Picture 4: Setting up host IP address 

We will do the IP setup later, now for the information all the installed URCaps can be found in Settings -> System -> URCaps section (see Picture 5). 
!![image](https://user-images.githubusercontent.com/7438736/113155773-4e524780-9231-11eb-8acd-10fd07a0904d.png)

 
Picture 5: Sucessfull URCap will be stored in the URCap section at the Settings -> System -> URCaps 

To use this External Control URCaps we need to create a new program and insert the External Control program node into the program tree (see Picture 6), check if the information is correct and save it for later. 
 !![image](https://user-images.githubusercontent.com/7438736/113155892-6b871600-9231-11eb-8e10-bd1754a79f3b.png)

Picture 6: URCap can be loaded under Program -> URCaps

The next task is to install the UR10e robot driver in Ubuntu assuming you already have installed ROS Noetic in the system, if yes then please follow the below instructions:
In a terminal:
1.	Source global ROS: source /opt/ros/noetic/setup.bash
2.	Clone the driver into the src folder of your catkin workspace: $ git clone https://github.com/UniversalRobots/Universal_Robots_ROS_Driver.git src/Universal_Robots_ROS_Driver
3.	Clone fork of the description. This is currently necessary, until the changes are merged upstream: $ git clone -b calibration_devel https://github.com/fmauch/universal_robot.git src/fmauch_universal_robot
4.	Install dependencies: $ sudo apt update -qq, $ rosdep update, $ rosdep install --from-paths src --ignore-src -y
5.	Build the workspace. We need an isolated build because of the non-catkin library package: catkin_make
6.	Lastly activate the workspace (ie: source it): $ source devel/setup.bash

Following the above-mentioned steps should work without issue, alternatively this can be built using all-source build method which can be found on the GitHub page given before.   

2.	Setting up TCP/IP connection. 
Now the pending part of the External Control should be done which is setting up IP in both on Polyscope and in your system. 
First you need to connect the robot and PC using an Ethernet cable. The ethernet port on the robot can be found inside the control box shown in picture 7: 
 
Picture 7
After plugin the cables on both devices, open a terminal on Ubuntu and perform an ifconfig. Look for eth0 IP address and take a note on it. This address should be different than Wi-Fi IP. 
Then on the Polyscope go to the Settings -> Network -> Static Address. I tried to use DHCP but for some reason this cannot make automatic IP assignment, also by using Static address we don’t need to change IP on Polyscope every time. 
In the Static Address, the IP address should be similar to your PC’s (i.e. host) eth0 address but slightly modified, e.g., if the host IP is 169.254.23. 160, then the client (i.e. robot) should be 169.254.23.100 – the last bit should be anything less than 160. 
Then the Subnet mask should be 255.255.0.0, and the Default gateway as 0.0.0.0. The rest of Network section i.e. Preferred DNS and Alternate DNS should automatically take as 0.0.0.0. 
 
Picture 8
If everything goes okay, then Polyscope will show that “Network is connected”, now if you click on Setting -> About, a new screen with the assigned IP will show: 
 
 Picture 9
3.	Installing drivers and testing
Now our systems are ready to connect each other. On Ubuntu open a new terminal and type: 
roslaunch ur_robot_driver ur10e_bringup.launch robot_ip:=169.254.23.100
The ROS should start with a master node and should wait for command from robot side.  Then on the Polyscope, go to “Run” and load the External control URCaps (see picture 10 & 11). 
 
Picture 10
 
Picture 11
Once this External Control URCaps is running on the Polyscope, a message should show on the terminal that “Robot is ready to receive control commands”. 
 
Picture 12
Now in a new terminal type: rostopic echo and you will see all the topics which are currently available.
 
Picture 13
From such topics some can be controlled using rviz GUI, such as joint_trajectory_controller. To do that simply type: rosrun rqt_joint_trajectory_controller rqt_joint_trajectory_controller
A new GUI will come up where by selecting /control_manager and then scaled_pos_joint_traj_controller under controller will give the options to control all 6 joints of the robot (see picture 14). 
   
Picture 14
Further you can use rqt_graph command to see all the nodes and topics associated: 
 
Picture 15
To start with Moveit run: roslaunch ur10_moveit_config demo.launch rviz_tutorial:=true
A new graphical window should pop up see picture 16.
 
Picture 16
Here robot pose is followed by the real robot in real time and graphical planning for joints and end-effector is possible. 


   


