<img src="https://raw.githubusercontent.com/alstondu/Tello-SLAM/fbfb2d10c64432fe60b3cd47ad782f9ca4442845/fig/drone-thin.svg" width="100" />

# Tello-SLAM
Visual SLAM with DJI Tello

<div align="left">
<img src="https://img.shields.io/github/license/alstondu/Tello-SLAM ?style=flat-square&color=5D6D7E" alt="GitHub license" />
<img src="https://img.shields.io/github/last-commit/alstondu/Tello-SLAM?style=flat-square&color=5D6D7E" alt="git-last-commit" />
<img src="https://img.shields.io/github/commit-activity/m/alstondu/Tello-SLAM?style=flat-square&color=5D6D7E" alt="GitHub commit activity" />
<img src="https://img.shields.io/github/languages/top/alstondu/Tello-SLAM?style=flat-square&color=5D6D7E" alt="GitHub top language" />
</div>

---
## 🤝 Author
Yuang Du (ucab190@ucl.ac.uk)

---
## 📍 Overview
The aim to this project is to use DJI Tello drone to perform simultaneous localization and mapping (SLAM) while navigating in 3D space.

---
<!--## Initial Project Plan

1.	VSLAM with keyboard/joystick control (ORB-SLAM 3)  
	*Scaling the map*
2.	Navigation with map (simulation)  
	*RRT**
3.	Navigation with map (drone)
4. ---
4.	VSLAM and Navigation without map (simulation)
5.	VSLAM and Navigation without map (drone)-->

## Expected Pipeline

1. Tello feeds the RGB video stream to the ubuntu ROS PC via Wi-FI. ✅
2. The RGB image stream is passed through depth estimation network to get depth image stream.
3. Both RGB and depth image stream is fed to Visual SLAM (RGB-Deep D SLAM) for localization (6DOF pose: translation + orientation)
4. With the global goal point selected, the Fast-Planner performes path planning given the 6DOF pose and the depth image stream.
5. Design a PID controller to track the generated trajectory and send commands to Tello.

Visual SLAM: RGB-Deep D SLAM ([orb-slam3-ros](https://github.com/thien94/orb_slam3_ros) + monocular depth estimation)  
Monocular Depth Estimation: [Packnet-sfm](https://github.com/TRI-ML/packnet-sfm?tab=readme-ov-file)/[monodepth2](https://github.com/nianticlabs/monodepth2)  
Path planner: [Fast-Planner](https://github.com/HKUST-Aerial-Robotics/Fast-Planner)

## Project Plan
1. Monocular SLAM with keyboard/joystick control ([orb-slam3-ros](https://github.com/thien94/orb_slam3_ros) and [tello driver](https://github.com/jordy-van-appeven/tello_driver))  

	*[Image distortion](https://github.com/surfii3z/image_undistort)  rectification*

2. Monocular SLAM with keyboard/joystick control in Gazebo ✅
3. Training monocular depth estimation ([Packnet-sfm](https://github.com/TRI-ML/packnet-sfm?tab=readme-ov-file)/[monodepth2](https://github.com/nianticlabs/monodepth2))  
*Camera calibration*
4. Pseudo RGB-D SLAM with keyboard/joystick control
5. Deploying Path Planner ([Fast-Planner](https://github.com/HKUST-Aerial-Robotics/Fast-Planner)) with keyboard/joystick control
6. Designing [PID controller](https://github.com/surfii3z/drone_controller/tree/thesis)（testing with pre-built scaled map and manually defined way points）
7. Final integration 

## Tello Manual Control

```terminal 1
roscore
```

```terminal 2
rosrun rviz rviz
```

```terminal 3
roslaunch tello_driver tello_teleop.launch
```

### Keyboard Control

```terminal 4
rosrun tello_driver keyboard_teleop_node.py
```

### Joystick Control (Xbox)
```terminal 4
roslaunch tello_driver joy_teleop.launch
```

## Tello Simulation in Gazebo
### 1. Camera Calibration
Launch the world with calibration board and tello:

```
roslaunch tello_driver cam_cal.launch
```

Run keyboard teleop node in terminal 2:

```
rosrun keyboard_teleop keyboard_teleop_node.py _repeat_rate:=10.0
```
Run cam_calibration node:

```
rosrun cam_calibration cameracalibrator.py --size 7x7 --square 0.25 image:=/front_cam/camera/image camera:=/front_cam
```
Drive the drone around the board until ```X, Y, Size, Skew``` all turn green. Click on the 'CALIBRATE' button, 'Save' the parameters and exit with 'COMMIT'.

### 2. Monocular SLAM
1. Launch the system in terminal 1:

	```
	roslaunch tello_driver tello.launch
	```

2.  Keyboard Control(To Do: Joystick Control)

	Run teleop node in terminal 2:

	```
	rosrun keyboard_teleop keyboard_teleop_node.py _repeat_rate:=10.0
	```

3.  Save trajectory

	Before driving the drone, in terminal 3, record the predicted trajectory(```/orb_slam3_ros/camera_pose```) and the ground truth(```/ground_truth/state```) as rosbag:
	
	```
	rosbag record /orb_slam3_ros/camera_pose /ground_truth/state
	```

4.  EVO Evaluation

	Convert the saved rosbag to tum format:
	
	```
	evo_traj bag [bag_name] /orb_slam3_ros/camera_pose /ground_truth/state --save_as_tum
	```
	
	Change the suffix of the files from ```.tum``` to ```.txt```, plot the trajectories with(```-a``` for alignment, ```-as``` for alignment and scale):
	
	```
	evo_traj tum orb_slam3_ros_camera_pose.txt --ref ground_truth_state.txt -a -p --plot_mode=xyz
	```
	For APE(Absolute Pose Error), run:
	
	```
	evo_ape tum ground_truth_state.txt orb_slam3_ros_camera_pose.txt -vas -r full -p --plot_mode=xyz
	```

