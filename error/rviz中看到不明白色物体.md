### 报错

![image-20251216221922188](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/error_unorigined_thing.png)=

[move_group-2] [WARN] ... Failed to fetch current robot state.
[move_group-2] [INFO] ... Didn't receive robot state (joint angles) with recent timestamp ... latest received state has time 0.000000.

rviz中显示没有各关节到word的tf

![image-20251217183410256](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/error_no_transform.png)

#### 问题描述

move_group缺少/joint_states

#### 解决方案

添加joint_state_publisher_gui

~~~py
    node_joint_state_publisher = Node(
        package='joint_state_publisher_gui',
        executable='joint_state_publisher_gui',
        name='joint_state_publisher_gui',
        output='screen'
    )
~~~

#### joint_states链路

~~~mermaid
graph TD
    A[joint_state_publisher_gui] --发布--> B((/joint_states))
    
    B --订阅--> C[robot_state_publisher]
    B --订阅--> D[move_group]
    
    C --计算并发布--> E((/tf 和 /tf_static))
    
    E --订阅--> D[move_group]
    E --订阅--> F[RViz2]
    
    style A fill:#f9f,stroke:#333,stroke-width:2px
    style B fill:#ff9,stroke:#333,stroke-width:2px
    style C fill:#9cf,stroke:#333,stroke-width:2px
    style D fill:#9f9,stroke:#333,stroke-width:2px
~~~

##### Source

>   产生或读取机器人的关节传感器数据

**节点**：joint_state_publisher_gui（仿真） / 机器人驱动、ros2_control（真机）

**作用**：

-   维护关节名列表，如 ['joint1', 'joint2', ...]
-   根据滑块或编码器读数生成关节角度
-   以固定频率发布关节状态（常见 10Hz–50Hz）

**发布消息**：sensor_msgs/JointState

-   header：时间戳
-   name：关节名数组
-   position：关节角度（单位：弧度）
-   velocity：关节角速度（可为空）
-   effort：力/力矩（可为空）

------

##### Topic

>   在节点之间传递关节状态数据

**话题名**：/joint_states

**作用**：

-   ROS 的标准关节状态通道
-   所有关心机器人当前姿态的节点都会订阅
-   如果没有数据发布，下游节点会“听不到声音”

------

##### Transformer

>   把关节角度翻译成空间位姿

**节点**：robot_state_publisher

**输入**：

-   URDF 机器人模型（连杆长度、关节关系）
-   /joint_states 关节角度数据

**作用**：

-   正运动学（Forward Kinematics，FK）计算
-   根据关节角度，算出各个连杆坐标系在三维空间中的位置和姿态
-   将结果发布到 /tf 话题

**为什么重要**：

-   RViz、MoveIt 不直接理解“关节角度”
-   它们依赖 TF（坐标变换关系）来工作

------

##### Planner

>   负责运动规划与碰撞判断

**节点**：move_group（MoveIt 核心节点）

**用途一：确定起点（Start State）**

-   订阅 /joint_states 获取当前机器人状态
-   规划时需要知道“现在在哪”
-   听不到 /joint_states 就无法开始规划

**用途二：碰撞检测（Collision Checking）**

-   根据实时关节状态更新内部模型
-   判断是否与环境或自身发生碰撞

------

##### Visualizer

>   把机器人显示出来

**节点**：rviz2

**作用**：

-   渲染机器人模型（STL/DAE）
-   主要订阅 /tf，而不是 /joint_states
-   根据 TF，把每个零件放到正确的空间位置

**常见现象说明**：

-   有 /joint_states 但 robot_state_publisher 不工作
-   RViz 中机器人会“散架”，零件堆在原点
-   原因是缺少 TF 坐标变换信息