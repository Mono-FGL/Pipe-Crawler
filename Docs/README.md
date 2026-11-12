Markdown

# Orbbec Astra Mini Pro & ROS 2 Jazzy Setup Guide

This guide details how to build, compile, patch, and run the Orbbec Astra Mini Pro depth camera package (`ros2_astra_camera`) inside a Docker container running **ROS 2 Jazzy** on a Raspberry Pi 5, along with bag recording and local WSL playback workflows.

---

## Part 1: Workspace & Docker Environment Setup

### Step 1: Clone Repositories on Host
Create your workspace directory on the Raspberry Pi host and clone the necessary camera and driver source repositories:

mkdir -p /home/project-lumen/ros2_ws/src
cd /home/project-lumen/ros2_ws/src

git clone https://github.com/orbbec/ros2_astra_camera.git
cd /home/project-lumen/ros2_ws
git clone https://github.com/libuvc/libuvc.git


### Step 2: Launch the Docker Container
Start and attach to your interactive ROS 2 Jazzy Docker container, mounting your host workspace into `/root/ros2_ws`:

sudo docker run -it --name jazzy --net host --privileged

-v /dev:/dev

-v /home/project-lumen/ros2_ws:/root/ros2_ws

ros:jazzy

*(For subsequent sessions, resume the container using: `sudo docker start -ai jazzy`)*

---

## Part 2: Installing Dependencies & Building libuvc

Run all following steps **inside** the running `jazzy` Docker container.

### Step 1: Install System Dependencies

apt update && apt install -y

build-essential cmake pkg-config git python3-colcon-common-extensions

python3-rosdep libusb-1.0-0-dev libjpeg-dev libudev-dev libgflags-dev

libgoogle-glog-dev libeigen3-dev libopenni2-dev libopencv-dev nlohmann-json3-dev

ros-jazzy-image-geometry ros-jazzy-cv-bridge ros-jazzy-camera-info-manager

ros-jazzy-image-transport ros-jazzy-image-publisher ros-jazzy-diagnostic-updater


### Step 2: Build & Install libuvc

cd /root/ros2_ws/libuvc
mkdir -p build && cd build
cmake ..
make -j1
make install
ldconfig


---

## Part 3: Applying Code Fixes for ROS 2 Jazzy

Due to header and API changes in ROS 2 Jazzy, apply these automated patches before compiling the camera package:

cd /root/ros2_ws
Fix 1: Update pinhole camera model header extensions from .h to .hpp

sed -i 's#pinhole_camera_model.h>#pinhole_camera_model.hpp>#'

src/ros2_astra_camera/astra_camera/src/point_cloud_proc/point_cloud_xyz.cpp

src/ros2_astra_camera/astra_camera/src/point_cloud_proc/point_cloud_xyzrgb.cpp
Fix 2: Update Parameter Callback syntax across the package

sed -i 's#OnParametersSetCallbackType#OnSetParametersCallbackType#g'

$(grep -rl "OnParametersSetCallbackType" src/ros2_astra_camera)
Fix 3: Include constants header explicitly

cd /root/ros2_ws/src/ros2_astra_camera/astra_camera/include/astra_camera
sed -i '1i #include "constants.h"' constants.h 2>/dev/null || true


---

## Part 4: Building astra_camera (Low-Memory Optimization)

To prevent out-of-memory (OOM) crashes on the Raspberry Pi during compilation, limit worker threads to `1` and set the package flag:

cd /root/ros2_ws
source /opt/ros/jazzy/setup.bash
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig
export MAKEFLAGS=-j1

colcon build --packages-select astra_camera

--parallel-workers 1

--cmake-args -DCMAKE_BUILD_TYPE=Release

*(To build the full workspace later with the same memory safety guard, use: `colcon build --parallel-workers 1 --cmake-args -DCMAKE_BUILD_TYPE=Release`)*

---

## Part 5: Launching & Recording Data

### Step 1: Launch the Camera Node (Terminal 1)

cd /root/ros2_ws
source /opt/ros/jazzy/setup.bash
source /root/ros2_ws/install/setup.bash
Example launch with custom resolution and streams enabled:

ros2 launch astra_camera astra.launch.xml

color_width:=640 color_height:=480 color_fps:=15

depth_width:=640 depth_height:=480 depth_fps:=15

enable_ir:=false enable_point_cloud:=false enable_colored_point_cloud:=false


### Step 2: Record a Rosbag (Terminal 2)
Open a second terminal on the Raspberry Pi host and attach to the container:

sudo docker exec -it jazzy bash
source /opt/ros/jazzy/setup.bash
source /root/ros2_ws/install/setup.bash
cd /root/ros2_ws
Record depth, color streams, and camera info

ros2 bag record -o pipe_run1

/camera/color/image_raw /camera/depth/image_raw

/camera/color/camera_info /camera/depth/camera_info

*(Press `Ctrl + C` to stop recording when finished).*

### Step 3: Fix Host File Permissions
Because Docker creates files as root, fix ownership on the Pi host machine so you can transfer files over SSH:

sudo chown -R project-lumen:project-lumen /home/project-lumen/ros2_ws/pipe_run1


---

## Part 6: WSL Ubuntu Playback & Verification (Laptop)

### Step 1: Transfer Bag File to WSL
Pull the recorded bag directory directly from your laptop's WSL terminal:

scp -r project-lumen@10.0.0.67:/home/project-lumen/ros2_ws/pipe_run1 ~/


### Step 2: Play Back and Visualize Bag Data

* **Terminal 1 (Play loop):**

ros2 bag play ~/pipe_run1 --loop


* **Terminal 2 (Visualize stream):**

ros2 run rqt_image_view rqt_image_view

*(Select `/camera/depth/image_raw` or `/camera/color/image_raw` from the dropdown).*
