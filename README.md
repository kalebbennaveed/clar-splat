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
```

Similarly to build the Safe Splat container:

```
cd ~/SafeSplat
docker compose build
```

### On Jetson

To build visual SLAM container follow the instructions on [ISAAC ROS Visual SLAM](https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_visual_slam/isaac_ros_visual_slam/index.html#quickstart) website.

Note: Make sure to build the repository from source as a few changes are needed to be made in the launch file explained further. And after restarting the docker make sure to follow the build the workspace and source it inside the container.

## Record Training Data

First let's make some changes to the visual SLAM launch file

```
cd ~/workspaces/issac_ros_dev/src/issac_ros_visual_slam/launch
gedit isaac_ros_visual_slam_realsense.launch.py 
```

Update the realsense_camera_node section to:

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

Note: If required change the topic mapping as visible upon listing all the topics.

In Terminal 1 start the camera and visual slam node:

```
ros2 launch isaac_ros_visual_slam isaac_ros_visual_slam_realsense.launch.py
```

And simultaneouly on Terminal 2 record a bag file inside the visual SLAM container

```
ros2 bag record \
/camera/depth/image_rect_raw/compressed \
/camera/aligned_depth_to_color/image_raw \
/visual_slam/tracking/odometry \
-o <bag_name>
```

## GSplat Training

Copy the bag file to your laptop inside the colcon_ws/ of ActiveSplat folder. Then on your laptop initialize the Active Splat container on two terminals using:

```
cd ~/clar-splat/ActiveSplat
docker compose up -d
docker exec -it <container_name> bash
```

On Termnial 1 play the bag file in paused mode (in pause mode a bag can be played by pressing spacebar):

```
ros2 bag play <bag_name> --start-paused
```

On Terminal 2 enter the following command to start spatfacto training model:

```
ns-train ros-splatfacto --data configs/desk.json --pipeline.datamanager.use-compressed-rgb True --pipeline.datamanager.dataparser.scene-scale-factor 0.5 --pipeline.datamanager.data-update-freq 8.0
```

After some initialization a message will appear stating that (NerfBridge) Images recieved: 0, at this point you should open the Nerfstudio viewer in a browser tab to visualize the Nerfacto model as it trains. Training will start after completing the next step.

Now press the spacebar key to start playing the rosbag [Terminal 1]. Once the pre-training image buffer is filled (defaults to 10 images) then training should commence, and the usual Nerfstudio print messages will appear in Terminal 2.

After the rosbag finishes playing NerfBridge will continue training the model on all of the data that it has recieved, but no new data will be added. Feel free to take images and video renders of your newly trained model.

## Running safety filter

The below example is demonstrated for single-integrator dynamics which is ideal for mobile robots. If you want to explore double-integrator check out the instructions on [safer-splat](https://github.com/chengine/safer-splat/tree/master). We will similarly run the Safe Splat container and perforns steps to run various trajectories, perform filteration adn finally visualize them.

```
cd ~/clar-splat/SafeSplat/
docker compose up -d
docker exec -it <container_name> bash
```

In order to perform experiments with ease, follow the given steps:

```
cd ~/clar-splat/SafeSplat/colcon_ws/src/safer-splat/
mkdir output
cd output
mkdir configs
cd configs
```

Copy the trained model from the Active Splat folder inside the newly created configs folder.

Make sure the folder has the following structure:

```
├──outputs                                                                                                                                           
│   └── configs                                                                                                  
│       └── ros-splatfacto                                                                                                                             
│           └── <folder_name> (saved with the date you run it on)                                                                              
│               └── nerfstudio_models
|               └── config.yml
|               └── dataparser_transforms.json # This file contains the transform that transforms from "Data" frame to "Nerfstudio" frame (which is typically a unit box)
```


