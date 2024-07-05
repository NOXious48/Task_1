
# Task 1
This is a guide on how to do mapping and autonomous naviagtaion using turtlebot in gazebo and rviz
## Procedure

### 1. Creating workspace and downloading required packages
The TurtleBot3 Simulation Package requires `turtlebot3` and `turtlebot3_msgs` packages as prerequisite. 

```
$ cd ~/catkin_ws/src/
$ git clone -b noetic-devel https://github.com/ROBOTIS-GIT/turtlebot3_simulations.git
$ git clone https://github.com/marinaKollmitz/gazebo_ros_2Dmap_plugin
$ cd ~/catkin_ws && catkin_make
```
Download the desired world and it's model from [here](https://github.com/mlherd/Dataset-of-Gazebo-Worlds-Models-and-Maps).

In my case i am using hospital world. copy all the models from hospital.zip and paste it in .gazebo/models directory . Copy `hospital.world` file in ~/catkin_ws/src/turtlebot3_simulations/turtlebot3_gazebo/worlds

Now create a launch file by name `hospital.launch`
```
<launch>
  <arg name="model" default="$(env TURTLEBOT3_MODEL)" doc="model type [burger, waffle, waffle_pi]"/>
  <arg name="x_pos" default="0.0"/>
  <arg name="y_pos" default="0.0"/>
  <arg name="z_pos" default="0.0"/>

  <include file="$(find gazebo_ros)/launch/empty_world.launch">
    <arg name="world_name" value="$(find turtlebot3_gazebo)/worlds/hospital.world"/>
    <arg name="paused" value="false"/>
    <arg name="use_sim_time" value="true"/>
    <arg name="gui" value="true"/>
    <arg name="headless" value="false"/>
    <arg name="debug" value="false"/>
  </include>

  <param name="robot_description" command="$(find xacro)/xacro --inorder $(find turtlebot3_description)/urdf/turtlebot3_$(arg model).urdf.xacro" />

  <node pkg="gazebo_ros" type="spawn_model" name="spawn_urdf" args="-urdf -model turtlebot3_$(arg model) -x $(arg x_pos) -y $(arg y_pos) -z $(arg z_pos) -param robot_description" />
</launch>

```
Save `hospital.launch` in directory ~/catkin_ws/src/turtlebot3_simulations/turtlebot3_gazebo/launch


### 2. Mapping

For the mapping purpose i have used this [plugin](https://github.com/marinaKollmitz/gazebo_ros_2Dmap_plugin) . Download this plugin in your src folder as explained in step 1

The reason for not using `explore_lite` package or teleop method is because the map generated from them was not good for navigation.
This is was output of `explore_lite` package:

![improper mapping](https://github.com/NOXious48/Task_1/blob/main/improper_map.png.png)

While the map generated from this custom plugin :

![hospital_map](https://github.com/NOXious48/Task_1/blob/main/hospital.png.png)

#### how to use this plugin

Check out the plugin in your `catkin_ws` and build it with `catkin_make`.

To include the plugin, add the following line in between the `<world> </world>` tags of your Gazebo world file:

```
<plugin name='gazebo_occupancy_map' filename='libgazebo_2Dmap_plugin.so'>
    <map_resolution>0.1</map_resolution> <!-- in meters, optional, default 0.1 -->
    <map_height>0.3</map_height>         <!-- in meters, optional, default 0.3 -->
    <map_size_x>100</map_size_x>          <!-- in meters, optional, default 10 -->
    <map_size_y>100</map_size_y>          <!-- in meters, optional, default 10 -->
    <init_robot_x>0</init_robot_x>          <!-- x coordinate in meters, optional, default 0 -->
    <init_robot_y>0</init_robot_y>          <!-- y coordinate in meters, optional, default 0 -->
</plugin>
```
open your terminal and run `roscore` 

source the `setup.bash` file in new terminal
```
$ source ~/catkin_ws/devel/setup.bash
```

export turtlebot model
```
$ export TURTLEBOT3_MODEL=waffle_pi
```
run the gazebo world
```
$ roslaunch turtlebot3_gazebo <mapname>.launch
```
To generate the map, call the `/gazebo_2Dmap_plugin/generate_map` ros service:

```
$ rosservice call /gazebo_2Dmap_plugin/generate_map
```

The generated map is published on the `/map2d` ros topic. 

You can use the `map_saver` node from the `map_server` package inside ros navigation to save your generated map to a .pgm and .yaml file:

```
$ rosrun map_server map_saver -f <mapname> /map:=/map2d
```
The last map generated with the ```/gazebo_2Dmap_plugin/generate_map``` call is saved.

in place of `<mapname>`, I will be using `hospital`.


### 3. Navigation
open your terminal and run `roscore` 

source the `setup.bash` file in new terminal
```
$ source ~/catkin_ws/devel/setup.bash
```

export turtlebot model
```
$ export TURTLEBOT3_MODEL=waffle_pi
```
run the gazebo world 
```
$ roslaunch turtlebot3_gazebo hospital.launch
```
in the new terminal source `setup.bash` file and export same turtlebot model and then launch navigaiton stack of roscore
 ```
$ roslaunch turtlebot3_navigation turtlebot3_navigation.launch map_file:=$HOME/hospital.yaml
```
For estimation of initial position of robot in rviz follow section 0.3.4 this [tutorial](https://emanual.robotis.com/docs/en/platform/turtlebot3/nav_simulation/)

Now you can use the 2D NAV tool for navigaiton in RVIZ

## Resources used

1. [Robotis e manual](https://emanual.robotis.com/docs/en/platform/turtlebot3/overview/#overview)
