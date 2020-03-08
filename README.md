# ubiquity_robot

Robot control with raspberry-pi 3

OS : ubuntu16.04

Memo in Qiita: https://qiita.com/taka_horibe/private/abc85f486aaef32c44c8

## How to launch

```
$ roslaunch robot_launch robot.launch
```

The launch above includes
 - ydLiDAR driver
 - map server
 - AMCL for localization
 - move_base for planning
 
 
 Change this line to load your own map data.
 https://github.com/TakaHoribe/ubiquity_robot/blob/master/src/robot_launch/launch/robot.launch#L19
 
 ## Create your map
 
 Launch LiDAR sensor driver and make the robot move around. 
 
 ```
 $ roslaunch ydlidar lidar.launch
 ```
 
 Save the rosbag with the command such as `$ rosbag record --all`.
 
 Then, launch `gmapping` to create map. Set args to fit in your environment. (NOTE: planning fails with the large map due to computational cost.)
 
```
 $ rosrun gmapping slam_gmapping scan:=base_scan _delta:=0.1 _xmin:=-20 _xmax:=20 _ymin:=-20 _ymax:=20
```

Check with your map in Rviz. After creating the map, save it.

```
$ rosrun map_server map_saver -f <saved_map_name>
```


## Use Realsense (Optional)

Run realsense T265 to compensate for the odometry accuracy.

When the robot starts up, the `/odom` tf is published by the `/motor_driver` node. You need kill the publishing to use the realsense camera odometry.

Run the dynamic reconfigure. 

```
$ rosrun rqt_reconfigure rqt_reconfigure
```

Check the `publish_odom_tf` off in velocity controller.

Enable the following to run realsense with some tf_publishers to connect the realsense and robot in tf space.
https://github.com/TakaHoribe/ubiquity_robot/blob/master/src/robot_launch/launch/robot.launch#L9-L14



## Check robot status from laptop

Set `ROS_IP` and `ROS_MASTER_URI`

In robot, 
```
$ export ROS_IP=<your robot ip>  // e.g. ROS_IP=192.168.1.1
```

In your laptop,
```
$ export ROS_IP=<your laptop ip>
$ export ROS_IP=http://<your ROBOT ip>:11311  // e.g. ROS_MASTER_URI=http://192.168.1.1:11311
```

Run any ros commands in your laptop to see if it is connected correctly. e.g.
```
$ rostopic list
```

You can check the robot status with rviz in your laptop.

## Important notes for the first setup

 - The move_base with the latest release version (1.16.4) cannot be launched in raspberry-pi. The 1.16.3 works.
 - The realsense driver binary for the raspberry-pi is not released. Build it by yourself. see https://qiita.com/taka_horibe/private/abc85f486aaef32c44c8#raspiarm%E3%81%AE%E5%A0%B4%E5%90%88
 - Set swap area before build src. It failed even with -j1 command.
