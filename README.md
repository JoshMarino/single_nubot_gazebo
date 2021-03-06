# Description   
This simulation system can simulate ONE robot soccer player for RoboCup Middle Size League. It can be adapted for other purposes. Note that this package is only designed for demonstration. If you want to test multi-robot cooperation strategies, please refer to another repository: [gazebo_visual](https://github.com/nubot-nudt/gazebo_visual). However, the tutorial regarding compliation and etc. is still useful.   

Please read the paper ["Weijia Yao et al., A Simulation System Based on ROS and Gazebo for RoboCup Middle Size League, 2015"](https://www.trustie.net/organizations/23/publications) for more information.   
   
- Maintainer status: maintained
- Maintainer: Weijia Yao <abcgarden@126.com>
- Author: Weijia Yao <abcgarden@126.com>
- License: Apache
- Bug / feature tracker: https://github.com/nubot-nudt/single_nubot_gazebo/issues
- Source: git https://github.com/nubot-nudt/single_nubot_gazebo (branch: master)   

# Recommended Operating Environment
1. Ubuntu 14.04; 
2. ROS Indigo or ROS Jade. (It is recommended to install ROS Jade)
3. Gazebo 5.0 or 5.1 or 7.1;
4. gazebo_ros_pkgs; (please read the **NOTE** below for more information)  
Other versions of Ubuntu, ROS or Gazebo may also work, but we have not tested yet.

**NOTE:** 
Concering how to install appropriate **gazebo_ros_pkgs**, please read the following according to your own situation:   
 - 1.  If you decide to use **ROS Indigo**, please read the following:   
If you choose "desktop-full" install of ROS Indigo, there is a Gazebo 2.0 included initially. In order to install Gazebo 5.0/5.1, you should first remove Gazebo 2.0 by running:   
(**The following command is dangerous; it might delete the whole ROS, so please do it carefully or you may find other ways to delete gazebo2**)   
` $ sudo apt-get remove gazebo2* `    
Then you should be able to install Gazebo 5.0 now. To install gazebo_ros_pkgs compatible with Gazebo
5.0/5.1, run this command:   
` $ sudo apt-get install ros-indigo-gazebo5-ros-pkgs ros-indigo-gazebo5-ros-control`   
HOWEVER,     
It seems ` $ sudo apt-get install ros-indigo-gazebo5-ros-pkgs ros-indigo-gazebo5-ros-control` no longer works now.    
These packages may be moved to other places. You can checkout [gazebo_ros](https://github.com/ros-simulation/gazebo_ros_pkgs.git),
and download and install the correct version.   
 - 2. If you decide to use **ROS Jade** with **gazebo 5.0 or 5.1**, read the following   
ROS Jade has gazebo_ros_pkgs with it; so you don't have to install gazebo_ros_pkgs again.  
However, you should do the following steps to fix some of the bugs in ROS Jade related to Gazebo:        
  -  (a) `$ sudo gedit /opt/ros/jade/lib/gazebo_ros/gazebo`    
In this file, go to line 24 and delete the last '/'. So    
`setup_path=$(pkg-config --variable=prefix gazebo)/share/gazebo/`    
is changed to     
`setup_path=$(pkg-config --variable=prefix gazebo)/share/gazebo`    
You can read this link for more [information](http://answers.ros.org/question/215796/problem-for-install-gazebo_ros_package/)   
  -  (b) Install Gazebo 5.     
   `$ sudo apt-get install gazebo5`     
If this fails, try to run the ['gazebo5_install.sh'](https://github.com/nubot-nudt/simatch/blob/master/gazebo5_install.sh)(obtained from Gazebo's official website).    
Read for more [information](http://answers.ros.org/question/217970/ros-jade-and-gazebo-50-migration-problem/)   
  -  (c) Optional: copy resource files to the new gazebo folder.    
   `$ sudo cp -r /usr/share/gazebo-5.0/* /usr/share/gazebo-5.1`      
 - 3. If you decide to use **ROS Jade** with **gazebo 7.1**, read the following,    
  -  (1) Install gazebo 7.0 by running [gazebo7_install.sh](https://github.com/nubot-nudt/simatch/blob/master/gazebo7_install.sh)(obtained from Gazebo's official website);      
  -  (2) Go to the github website and download the repository of gazebo_ros_pkgs on the branch 'kinetic-devel':      
`$ git clone https://github.com/ros-simulation/gazebo_ros_pkgs.git -b kinetic-devel`     
  -  (3) Go to gazebo_ros_pkgs and create src/, put all other files or folders inside src/.    
` $ cd src/`   
` $ catkin_init_workspace`   
  -  (4) To install dependencies, run    
` $ cd ../../`   
` $ rosdep install --from-paths gazebo_ros_pkgs --ignore-src --rosdistro=jade`      
  -  (5) Then install all these files to /opt/ros/jade   
` $ cd gazeo_ros_pkgs`   
` $ catkin_make -DCMAKE_INSTALL_PREFIX=/opt/ros/jade install`    
You may encounter problems such as 'cannot copu _setup_util.py to /opt/ros/...', if that is the case, you are supposed
to run this command as the root. For example,      
` $ sudo su`   
` $ source /opt/ros/jade/setup.bash`   
` $ rosdep install --from-paths gazebo_ros_pkgs --ignore-src --rosdistro=ROSDISTRO`   
` $ cd gazeo_ros_pkgs`   
` $ catkin_make -DCMAKE_INSTALL_PREFIX=/opt/ros/jade install`   

# Complie
1. Go to the package root directory (single_nubot_gazebo)
2. If you already have CMakeLists.txt in the "src" folder, then you can skip this step. 
   If not, run these commands:
       
    ```
    $ cd src
    $ catkin_init_workspace
    $ cd ..
    ```
3. $ ./configure   
You may encounter errors related to Git. In this case, if you did not use Git, you could just ignore these errors.   
4. $ catkin_make --pkg nubot_common
5. $ catkin_make   
You may have to run catkin_make for several times until 100% success. If this does not work, please contact us.   

# Tutorials

## Part I. Overview
The robot movement is realized by a Gazebo model plugin which is called "NubotGazebo" generated by source files "nubot_gazebo.cc" and "nubot_gazebo.hh". Basically the essential part of the plugin is realizing basic motions: omnidirectional locomotion, ball-dribbling and ball-kicking.

Basically, this plugin subscribes to topic "nubotcontrol/velcmd" for omnidirecitonal movement and subscribes to service "BallHandle" and "Shoot" for ball-dribbling and ball-kicking respectively. You can customize this code for your robot based on these messages and services as a convenient interface.

As for ball-dribbling, there are three ways for a robot to dribble a ball, i.e.
            
Method  | Description
:-----: | -------------
(a) Setting ball pose continually  | This is the most accurate one; nubot would hardly lose control of the ball, but the visual effect is not very good (the ball does not rotate).
(b) Setting ball secant velocity  | This is less acurate than method (a) but more accurate than method (c).
(c) Setting ball tangential velocity |  This is the least accurate. If the robot moves fast, such as 3 m/s, it would probably lose control of the ball. However, this method achieves the best visual effect under low-speed condition.
**By default, we use method (c) for ball-dribbling.**
    
 As for Gaussian noise, **by default, Gaussian noise is NOT added**, but you can add it by changing the flag in nubot_gazebo.cc in function update_model_info();
         
 
## Part II. Single robot automatic movement
 The robot will do motions according to states transfer graph. Steps are as follows:
 1. Go to the package root directory (single_nubot_gazebo)
 2. source the setup.bash file:   
   ` $ source devel/setup.bash`
 3.  `$ roslaunch nubot_gazebo sdf_nubot.launch`   
 
>  **Note:** Every time you open a new terminal, you have to do step 2. You can also write this command into the ~/.bashrc file so that you don't have to source it every time.

Finally, the robot rotates and translates with trajectory planning. That is, the robot accelerates at constant acceleration and stays at constant speed when it reaches the maximum velocity.
 
## Part III. Keyboad control robot movement
 1. In nubot_gazebo.cc, comment "nubot_auto_control();" and uncomment "nubot_be_control();" in function UpdateChild().
 2. Compile again and follow steps 1-3 listed in Part II.
 3. ` $ rosrun nubot_gazebo nubot_teleop_keyboard`
 
## Part IV. Appendix
  1. To launch an empty soccer field:   
  ` $ roslaunch nubot_gazebo empty_field.launch`
  2. To launch the simulation world with rqt_plot of nubot or ball's velocity:  
  ` $ roslaunch nubot_gazebo sdf_nubot.launch plot:=true`
  
## Q&A
