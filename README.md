# Exploring Safety-Critical Control over Gaussian Splatting Maps

## About
This project extends SaferSplat with a simplified and efficient Control Barrier Function (CBF) framework for safe real-time robot navigation in Gaussian Splat (GSplat) mapped environments. We implement a single integrator formulation for CBF dynamics, significantly reducing complexity while maintaining strong safety guarantees. The approach enables fast and memory-efficient safety filtering over scenes with hundreds of thousands of Gaussian primitives, operating at 15 Hz during online GSplat training with minimal GPU usage. The safety layer is minimally invasive, correcting robot actions only when necessary to prevent collisions. We validate our system in both simulation and real-world hardware experiments.

## Dependencies
For this project we have simplified the installation dependencies with containarized installation. We have seperate containers for training the GSplat and for filtering safe trajectories.

## Setup

We use laptop (NVIDIA RTX 4080, 13th Gen i7) for training and filtering trajectories. The images are captured using Intel Realsense D455 along with pose estimate are recorded using ROS Humble wrapper and visual SLAM package on Jetson Orin NX. Furthermore experiments are performed on both simulations as well as on hardware (Aion Robotics Rover).

### On Laptop
Follow the intstructions below before building the containers on your laptop.

```
git clone https://github.com/kalebbennaveed/clar-splat
cd clar-spat
git submodule update --init --recursive
cd SafeSplat/colcon_ws/src
git submodule update --init --recursive
```

Now build the Active Splat container for training:

```
cd ~/ActiveSplat
docker compose build
docker compose up -d
```

Similarly to build the Safe Splat container:

```
cd ~/SafeSplat
docker compose build
docker compose up -d
```

### On Jetson

To build visual SLAM container follow the instructions on [ISAAC ROS Visual SLAM](https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_visual_slam/isaac_ros_visual_slam/index.html#quickstart) website.

Note: Make sure to build the repository from source as a few changes are needed to be made in the launch file explained further.

## Record Training Data

First let's make some changes to the visual SLAM launch file

'''
cd ~/workspaces/issac_ros_dev/src/issac_ros_visual_slam/launch
gedit isaac_ros_visual_slam_realsense.launch.py 
'''

Update the code realsense_camera_node section as:

```
realsense_camera_node = Node(
        name='camera',
        namespace='camera',
        package='realsense2_camera',
        executable='realsense2_camera_node',
        parameters=[{
            'enable_infra1': True,
            'enable_infra2': True,
            'depth_module.emitter_enabled': 0,
            'depth_module.profile': '640x360x90',
            'enable_gyro': True,
            'enable_accel': True,
            'gyro_fps': 200,
            'accel_fps': 200,
            'unite_imu_method': 2,
            'enable_rgbd': True,
            'enable_sync': True,
            'align_depth.enable': True,
            'enable_color': True,
            'enable_depth': True
        }],
    )
```
