### 使用 Foxglove

>   适用于 ROS2

先速通一下文档：

https://docs.foxglove.dev/docs

https://docs.foxglove.dev/docs/getting-started/frameworks/ros2



#### 目录

[TOC]

### 环境

>下载 foxglove -- 注意是amd架构
>
>​	 Download the correct package for your system architecture, then run &
>
> 	Install future updates with

sudo apt install ./foxglove-studio-*.deb

sudo apt update && sudo apt install foxglove-studio

>   下载 ros-humble-foxglove-bridge  节点并启动
>
>   ​	本地开发 ws://localhost:8765`：当服务器与 Foxglove 运行在同一台机器上时使用
>
>   ​	机器人连接：用于`ws://ROBOT_IP:8765`连接到在机器人上运行的服务器，其中`ROBOT_IP`是机器人在网	络上的 IP 地址

sudo apt install ros-$ROS_DISTRO-foxglove-bridge
ros2 launch foxglove_bridge foxglove_bridge_launch.xml port:=8765



