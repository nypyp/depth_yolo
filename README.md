
[![ROS1 VERSION](https://img.shields.io/badge/ros-melodic-green)](http://wiki.ros.org/melotic)
&nbsp;
[![Ubuntu VERSION](https://img.shields.io/badge/Ubuntu-18.04-yellowgreen)](https://ubuntu.com/)
&nbsp;
[![Jetson Xavier NX](https://img.shields.io/badge/Nvidia-Jetson%20Xavier%20NX-brightgreen)](https://www.nvidia.cn/autonomous-machines/embedded-systems/jetson-xavier-nx/)
&nbsp;
[![LICENSE](https://img.shields.io/badge/license-GPL%203-informational)](https://github.com/0nhc/depth_yolo/blob/master/LICENSE)
&nbsp;

# Description
This is a package combining darknet_ros and kinect1 in order to get the 3D location of the objects detected.

It will automatically send tf transforms between the objects detected and kinect.

![b](imgs/b.jpeg)
  


# Installation
## step1 Install dependencies
1. ~~Remember to downgrade the gcc and g++ version of your system to 7 (my Ubuntu 20.04 is default 8)~~
2. install kinect drivers
```bash
sudo apt install ros-melodic-openni-*
```
3. python-pcl (if you are using Ubuntu 20.04 with ROS noetic, pip install python3-pcl)
```bash
pip install python-pcl 直接在NX或nano上pip会报错 所以需要下载源码编译
```
下载源码
```bash
git clone git@github.com:nypyp/python-pcl 这是为Ubuntu18更改setup.py的版本
```
确保numpy版本大于1.16.2，不然检查依赖过程中会自动下载高版本的numpy导致编译错误
```bash
python -m pip install upgrade numpy==1.16.6
```
编译
```bash
python setup.py build_ext -i
python setup.py install #install 过程中可能会出现一些依赖下载失败报错 自己pip安装即可
```

4. ros_numpy (if you are using Ubuntu 18.04 with ROS Melodic, sudo apt-get install ros-melodic-ros-numpy)
```bash
sudo apt-get install ros-melodic-ros-numpy
```
5. [darknet_ros](https://github.com/nypyp/darknet_ros.git) (This links to my forked repo. I modified it for supporting jeston nano)
## step2 Clone the dependencies packages
If you haven't cloned the dependencies packages yet, clone them in your workspace first.

```bash
cd <your_ws>/src
git clone --recursive https://github.com/nypyp/darknet_ros.git 
```
记得使用--recursive 不然编译时可能会报未找到active_layer错误

## step3 install depth_yolo
Then, you can continue installing depth_yolo
```bash
git clone https://github.com/nypyp/depth_yolo.git
cd ..
catkin_make -DCMAKE_BUILD_TYPE=Release
source devel/setup.bash
```
# Usage

It will automatically launch darknet_ros node(yolo v3) and kinect2_bridge.
```bash
roslaunch depth_yolo depth_yolo.launch
```
# Eorros might be occured

## **直接pip install pyhton-pcl报错**

如果直接pip install python-pcl报错：

应该参考官方仓库：https://github.com/strawlab/python-pcl
和这个安装帖子：https://blog.csdn.net/qq_54609718/article/details/124997287
直接下载源代码在本地编译

安装过程中，使用`python setup.py install`安装时，使用的是默认指令`python`的环境，即默认为python2.7就安装在2.7，而此时在pyhton2.7环境中的numpy如果低于1.16.1就会自动下载临时numpy进行编译，可能会导致高版本numpy导致编译失败，报错语法错误

所以安装之前可以先检查python中numpy版本：
```shell
python -c "import numpy; print(numpy.version.version)"
```
如果低于1.16.1，最好先更新相应pyhton版本的numpy，比如默认指令python环境为2.7的话，指定python版本更新numpy才有用：
```shell
python -m pip install upgrade numpy==1.16.6
```
如果要指定python3的话：
```shell 
python3 -m pip install upgrade numpy==1.16.6
```
值得注意的是默认pip是安装在python2环境中的，但有时pip安装也会直接安装到python3中去，所以直接使用pip不能指定卸载或更新相应python2或3环境中的包，最好通过上面指令指定

## **编译时cv_bridge找不到opencv**

```bash
CMake Error at /opt/ros/melodic/share/cv_bridge/cmake/cv_bridgeConfig.cmake:113
```

一般情况下安装ros会自动在`usr/include/opencv`中安装opencv，但是由于nano系统中自带了4.0版本的opencv在`usr/include/opence4/opence2`中，所以安装ros时就没有安装opencv，所以默认opencv路径应该为`usr/include/opence4/opence2`，此时就需要在cv_bridgeConfig.cmake中更改找寻opencv的路径，参考：https://blog.csdn.net/qq_38146340/article/details/113478745
将相应的行改为：
```cmake
set(_include_dirs "include;/usr/include;/usr/include/opence4/opence2")
```
即可成功找到opencv

## **编译时报算力不匹配错误**

作者源码为适配3090显卡的算力更改了darknet_ros/CMakeList.txt中第22行之后的内容，若遇到报错：
```shell
nvcc fatal : Unsupported gpu architecture ‘compute_86‘
```
需要在nano或者nx上编译需要更改相应的算力配置：
更改前：
```cmake
  set(
    CUDA_NVCC_FLAGS
    ${CUDA_NVCC_FLAGS};
    -O3
    -gencode arch=compute_35,code=sm_35
    -gencode arch=compute_50,code=[sm_50,compute_50]
    -gencode arch=compute_52,code=[sm_52,compute_52]
    -gencode arch=compute_61,code=sm_61
    -gencode arch=compute_62,code=sm_62
    -gencode arch=compute_75,code=sm_75
    -gencode arch=compute_86,code=sm_86 #3090的算力是8.6
  )
```
更改后：
```cmake
  set(
    CUDA_NVCC_FLAGS
    ${CUDA_NVCC_FLAGS};
    -O3
    -gencode arch=compute_35,code=sm_35
    -gencode arch=compute_50,code=[sm_50,compute_50]
    -gencode arch=compute_52,code=[sm_52,compute_52]
    -gencode arch=compute_53,code=sm_53 #jeston nano
    -gencode arch=compute_72,code=sm_72 #nx 其中nano和nx这两行2选1
  )
```
## **节点detph_yolo启动失败**

提示由于是script（py脚本）所以需要在首行添加`#!`声明python环境,添加后运行正常

# Contributing
Contributions are welcome! Please read the [contributing guidelines](CONTRIBUTING.md) before submitting a pull request.



# License
This project is licensed under the GPL 3 License. See [LICENSE](LICENSE) for more information.
```
    Copyright (C) 2023  0nhc

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program.If not, see <https://www.gnu.org/licenses/>.                               
```

# Acknowledgments 
I would like to thank the following people for their contributions to this project:

- **Herman Ye**  

  I would like to take a moment to express my sincere gratitude to Herman Ye for his invaluable contribution to this project's README. His attention to detail and clear communication have greatly improved the overall quality of the documentation, making it easier for others to understand and contribute to the project.  
  Herman Ye's dedication and hard work have not gone unnoticed, and I am truly grateful for his efforts. Thank you, Herman Ye, for your outstanding work and for being an integral part of this project's success.for implementing the search functionality.


