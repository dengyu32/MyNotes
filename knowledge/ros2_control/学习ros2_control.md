### learn ros2 control

>   学习 ROS2 Control 基础概念、Gazebo 中的 hardware_interface、controller_manager 调用流程

------

[TOC]

### 学习

>   参考学习资源

https://www.bilibili.com/video/BV186pjzzEKY/?spm_id_from=333.337.search-card.all.click&vd_source=43bff3dbe361c224a6ecfc545aa2b0b5

https://www.bilibili.com/video/BV1ku411G7UR/?spm_id_from=333.337.search-card.all.click&vd_source=43bff3dbe361c224a6ecfc545aa2b0b5

------

### 概念

>   ROS2 Control 作为机器人中间件，用于衔接 ROS2 与实体机器人硬件

ROS2 和实体机器人之间依靠 **中间件（middleware）** 完成控制流程：

-   controller（控制器，执行控制算法）
-   hardware_interface（硬件接口层，用于与真实或仿真硬件通讯）
    -   command_interface（控制指令接口）
    -   state_interface（状态反馈接口）

![image-20250921161103414](https://raw.githubusercontent.com/dengyu32/note_images/main/images/20251211150234269.png)

------

### 调用 controllers 和 hardware_interface

#### Gazebo 演示

>   使用 Gazebo 搭建一个完整 ros2_control 机器人控制流程示例

##### 准备 description 功能包

>   构建机器人模型及其组件

-   bbot_description.xacro：主要模型描述文件
-   intertial_macros.xacro：惯性等宏函数
-   bbot.urdf.xacro：总的 URDF 入口文件

##### 编写 bbot.ros2_control.xacro

>   定义 ros2_control 插件、硬件资源、controller_manager、Gazebo 映射等

###### 插入 hardware interface plugin

![image-20250921171459163](https://raw.githubusercontent.com/dengyu32/note_images/main/images/20251211150237357.png)

上述是 Gazebo 的 hardware plugin。
 若使用真实硬件，则需要 **自定义 hardware interface plugin**，并实现读写接口。

###### 编写 hardware resources

![image-20250921171650567](https://raw.githubusercontent.com/dengyu32/note_images/main/images/20251211150238610.png)

包含：

-   command_interface：控制量（速度、力矩、位置等）
-   state_interface：当前状态（速度、位置、电流等）

###### 加入 gazebo 标签

![image-20250921172013011](https://raw.githubusercontent.com/dengyu32/note_images/main/images/20251211150240552.png)

gazebo 标签的作用：

-   和 hardware_interface 交互，将 left_wheel_joint、right_wheel_joint 映射到 Gazebo
-   接收 robot_description 发布的 URDF，在 Gazebo 中生成实体模型
-   启动 controller_manager，管理各 controllers 的生命周期
     （初始化时需加载一个 YAML 文件，定义要加载的控制器及其参数）

###### 在 bbot.urdf.xacro 中包含 bbot.ros2_control.xacro