# Customized Openvslam for IR and RGB images

** Refered from openvslam:   
https://github.com/xdspacelab/openvslam   
https://openvslam.readthedocs.io/en/master/installation.html**

## Requirements for OpenVSLAM

Eigen : version 3.3.0 or later.  
g2o : Please use the latest release. Tested on commit ID 9b41a4e.  
SuiteSparse : Required by g2o.  
DBoW2 : Please use the custom version of DBoW2 released in https://github.com/shinsumicco/DBoW2.  
yaml-cpp : version 0.6.0 or later.  
OpenCV : version 3.3.1 or later.

## Installation

Install dependencies
```bash
sudo apt-get update
sudo apt-get --no-install-recommends
sudo apt-get install build-essential pkg-config cmake git wget curl unzip
sudo apt-get install libatlas-base-dev libsuitesparse-dev
sudo apt-get install libgtk-3-dev
sudo apt-get install ffmpeg
sudo apt-get install libavcodec-dev libavformat-dev libavutil-dev libswscale-dev libavresample-dev
sudo apt-get install libgoogle-glog-dev libgflags-dev
sudo apt-get install libopenblas-dev
sudo apt-get install --no-install-recommends libboost1.58-all-dev
sudo apt-get install libx11-dev
sudo apt-get install libgl1-mesa-dev sudo apt-get install libglu1-mesa-dev
sudo apt-get install freeglut3-dev
sudo apt-get install doxygen
wget https://nchc.dl.sourceforge.net/project/glew/glew/2.1.0/glew-2.1.0.tgz --no-check-certificate
tar -xzvf glew-2.1.0.tgz cd glew-2.1.0/
make -j2
sudo make install
sudo ln -s /usr/lib64/libGLEW.so.2.1 /usr/lib/libGLEW.so.2.1
sudo ldconfig -v

## Download Repo

Download my repo from  
https://gitlab.com/go_zm/eece5554_roboticssensing/-/tree/master/Final_project

Go to your workspace
```bash
cd /path/to/Final_project/
```

## Eigen Installation

```bash
cd eigen
mkdir build && cd build
cmake \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr/local \
    ..
make -j2
sudo make install
sudo ldconfig -v
```

## g2o Installation

```bash
cd g2o
mkdir build && cd build
cmake \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr/local \
    -DCMAKE_CXX_FLAGS=-std=c++11 \
    -DBUILD_SHARED_LIBS=ON \
    -DBUILD_UNITTESTS=OFF \
    -DBUILD_WITH_MARCH_NATIVE=ON \
    -DG2O_USE_CHOLMOD=ON \
    -DG2O_USE_CSPARSE=ON \
    -DG2O_USE_OPENGL=OFF \
    -DG2O_USE_OPENMP=ON \
    ..
make -j2
sudo make install
sudo ldconfig -v
```

## OpenCV3 Installation

```bash
cd opencv-3.4.5
mkdir -p build && cd build
cmake \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr/local \
    -DENABLE_CXX11=ON \
    -DBUILD_DOCS=OFF \
    -DBUILD_EXAMPLES=OFF \
    -DBUILD_JASPER=OFF \
    -DBUILD_OPENEXR=OFF \
    -DBUILD_PERF_TESTS=OFF \
    -DBUILD_TESTS=OFF \
    -DWITH_EIGEN=ON \
    -DWITH_FFMPEG=ON \
    -DWITH_OPENMP=ON \
    ..
make -j2
sudo make install
sudo ldconfig -v
```

## Pangolin Installation

```bash
cd Pangolin
mkdir build && cd build
cmake \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr/local \
    ..
make -j
sudo make install
sudo ldconfig -v
```

## DBoW2 Installation

```bash
cd DBoW2
mkdir build && cd build
cmake \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr/local \
    ..
make -j2
sudo make install
sudo ldconfig -v
```

## yaml-cpp Installation

```bash
cd yaml-cpp
mkdir build && cd build
cmake \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr/local \
    ..
make -j2
sudo make install
sudo ldconfig -v
```

## openvslam Installation

```bash
cd openvslam
git submodule init
git submodule update
mkdir build && cd build
cmake \
    -DBUILD_WITH_MARCH_NATIVE=OFF \
    -DUSE_PANGOLIN_VIEWER=ON \
    -DUSE_STACK_TRACE_LOGGER=ON \
    -DBOW_FRAMEWORK=DBoW2 \
    -DBUILD_TESTS=OFF \
    -DUSE_SOCKET_PUBLISHER=OFF \	
    ..
make -j2
```

## ROS Version Installation

Install the dependencies via apt.
```bash
apt update -y
apt install ros-${ROS_DISTRO}-image-transport
```

Download the source of cv_bridge.
```bash
cd /path/to/openvslam/ros
git clone --branch ${ROS_DISTRO} --depth 1 https://github.com/ros-perception/vision_opencv.git
cp -r vision_opencv/cv_bridge src/
rm -rf vision_opencv
```

Build with support for PangolinViewer. 
```bash
cd /path/to/openvslam/ros
catkin_make \
    -DBUILD_WITH_MARCH_NATIVE=ON \
    -DUSE_PANGOLIN_VIEWER=ON \
    -DUSE_SOCKET_PUBLISHER=OFF \
    -DUSE_STACK_TRACE_LOGGER=ON \
    -DBOW_FRAMEWORK=DBoW2
```

## Replicate my demo with IR Rosbag file

Leave the roscore run
```bash
roscore
```

Publish a IR rosbag file
```bash
rosbag play your_ir_rgb_lidar_rosbag.bag
```

Convert ir to img topic
```bash
rosrun image_transport republish   raw in:=/flir_boson/image_raw raw out:=/camera/image_raw
```

Run ROS Openvslam for Tracking and Mapping
```bash
cd /path/to/openvslam/ros
source devel/setup.bash
rosrun openvslam run_slam     -v /path/to/openvslam/demo/orb_vocab.dbow2     -c /path/to/openvslam/demo/config.yaml --frame-skip 3 --no-sleep --auto-term --map-db map.msg
```

Run ROS Openvslam for Localization
```bash
rosrun openvslam run_localization   -v /path/to/openvslam/demo/orb_vocab.dbow2   -c /path/to/openvslam/demo/config.yaml   --map-db map.msg --frame-skip 3 --no-sleep --auto-term --map-db map.msg
```

