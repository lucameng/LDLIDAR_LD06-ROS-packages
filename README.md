[LDROBOT]:https://github.com/ldrobotSensorTeam/ldlidar_stl_ros
[SLAM]:https://github.com/ros-perception/slam_gmapping

[TOC]
# Fix some problems of [ldlidar_stl_ros][LDROBOT] and make it adapted to [slam_mapping][SLAM]
>- Change the **topic** published by **node**`LD06`from`LiDAR/LD06`to `scan`, so as to better fit the gmapping launch file and let `slam_gmapping`subscribes the`scan`published by`LD06`.
>- Change the frame-id of LIDAR in `src/gmapping/launch/test_gmapping.launch`, from `/laser` to `/lidar_frame`
>- Change the **Class** of `src/ldlidar_stl_ros/rviz/test_map.rviz`, from`rviz/RPLidarScan`to`rviz/LaserScan`
  
Now you can customize the param yourself and try to gmap with your own lidar!



# Instructions for [ldlidar_stl_ros][ldlidar_stl_ros] and [ros-perception/slam_mapping][ros-perception/slam_gmapping]

> This SDK is only applicable to the LiDAR products sold by Shenzhen LDROBOT Co., LTD. The product models are :
> - LDROBOT LiDAR LD06
> - LDROBOT LiDAR LD19
## step 0: get LiDAR ROS Package
```bash
$ cd ~

$ mkdir -p ldlidar_ros_ws/src

$ cd ldlidar_ros_ws/src

$ git clone  https://github.com/lucameng/LDLIDAR_LD06-ROS-packages.git

```
## step 1: system setup
- Connect the LiDAR to your system motherboard via an onboard serial port or usB-to-serial module (for example, CP2102 module).

- Set the -x permission for the serial port device mounted by the radar in the system (for example, /dev/ttyUSB0)

  - In actual use, the LiDAR can be set according to the actual mounted status of your system, you can use 'ls -l /dev' command to view.

``` bash
$ cd ~/ldlidar_ros_ws

$ sudo chmod 777 /dev/ttyAMA0 
```
- Modify the `port_name` value in the Lanuch file corresponding to the radar product model under `launch/`, if your run it on RPI, it should be `ttyAMA0` or `ttyS0` , or it'd be `ttyUSB0` on a Computer. Using `ld06.launch` as an example, as shown below.

``` xml
<launch>
<!-- ldldiar message publisher node -->
 <node name="LD06" pkg="ldlidar_stl_ros" type="ldlidar_stl_ros_node" output="screen" >
  <param name="product_name" value="LDLiDAR_LD06"/>
  <param name="topic_name" value="scan"/>
  <param name="port_name" value ="/dev/ttyAMA0"/>
  <param name="frame_id" value="lidar_frame"/>
  <!-- Set laser scan directon: -->
  <!--    1. Set counterclockwise, example: <param name="laser_scan_dir" type="bool" value="true"/> -->
  <!--    2. Set clockwise,        example: <param name="laser_scan_dir" type="bool" value="false"/> -->
  <param name="laser_scan_dir" type="bool" value="true"/>
  <!-- Angle crop setting, Mask data within the set angle range -->
  <!--    1. Enable angle crop fuction: -->
  <!--       1.1. enable angle crop,  example: <param name="enable_angle_crop_func" type="bool" value="true"/> -->
  <!--       1.2. disable angle crop, example: <param name="enable_angle_crop_func" type="bool" value="false"/> -->
  <param name="enable_angle_crop_func" type="bool" value="false"/>
  <!--    2. Angle cropping interval setting, The distance and intensity data within the set angle range will be set to 0 --> 
  <!--       angle >= "angle_crop_min" and angle <= "angle_crop_max", unit is degress -->
  <param name="angle_crop_min" type="double" value="135.0"/>
  <param name="angle_crop_max" type="double" value="225.0"/>
 </node>
<!-- ldlidar message subscriber node -->
 <node name="ListenLD06" pkg="ldlidar_stl_ros" type="ldlidar_stl_ros_listen_node" output="screen">
  <param name="topic_name" value="scan"/>
 </node>
</launch>
```
## step 2: build

- run the following command.

```bash
$ cd ~/ldlidar_ros_ws

$ catkin_make
```
- you should see 3 packages prompted on your terminal, respectively are `ldlidar_stl_ros`, `gmapping`and `slam_gmapping`.
## step 3: run

### step3.1: package environment variable settings

- After the compilation is completed, you need to add the relevant files generated by the compilation to the environment variables, so that the ROS environment can recognize them. The execution command is as follows. This command is to temporarily add environment variables to the terminal, which means that if you reopen a new terminal, you also need to re-execute it. The following command.
  
```bash
$ cd ~/ldlidar_ros_ws
$ source devel/setup.bash
```
- noted that you are in `bash` instead of `fish` or `zsh`, etc.

- In order to never need to execute the above command to add environment variables after reopening the terminal, you can do the following.

```bash
$ echo "source ~/ldlidar_ros_ws/devel/setup.bash" >> ~/.bashrc
$ source ~/.bashrc
```
### step3.2: start LiDAR node

- The product is LDROBOT LiDAR LD06
``` bash
roslaunch ldlidar_stl_ros ld06.launch
```
## step 4: test

> The code was tested under ubuntu16.04 ROS kinetic、ubuntu18.04 ROS melodic、ubuntu20.04 ROS noetic, using rviz visualization.

- new a terminal (Ctrl + Alt + T) and use Rviz tool,open the `ldlidar.rviz` file below the rviz folder of the readme file directory
```bash
rosrun rviz rviz
```

| Product:          | Fixed Frame: | Topic:        |
| ------------------ | ------------ | ------------- |
| LDROBOT LiDAR LD06 | lidar_frame  | /scan   |
| LDROBOT LiDAR LD19 | lidar_frame  | /scan   |

- you can check the topic by doing the following.
```bash
rostopic
```
- for more info between`publisher`and`subscriber`, do
```bash
rosrun rqt_tf_tree rqt_tf_tree
```

## step 5: gmapping

>`test_map.launch`is the launch file of`gmapping.launch`. you can turn on`rviz`and other tools by customizing`test_map.launch`, while`gmapping.launch`are some basic parameters of lidar scan, which can be customized as well.

- new a terminal (Ctrl + Alt + T) and launch the gmapping file:
```bash
roslaunch gmapping test_map.launch
```
- the rviz visual interface should be display on the screen, click `add` and then `tf` to check the status of  `tf`, `by topic` and then `map` to build your own map!
- you can check those coordinates and tf by doing
```bash
rosrun rqt_graph rqt_graph
```
