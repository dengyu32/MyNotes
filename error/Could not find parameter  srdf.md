### 报错

~~~bash
[rviz2-3] [ERROR] [1765891191.396953615] [rviz2]: Could not find parameter robot_description_semantic and did not receive robot_description_semantic via std_msgs::msg::String subscription within 10.000000 seconds.
[rviz2-3] Error:   Could not parse the SRDF XML File. Error=XML_ERROR_EMPTY_DOCUMENT ErrorID=13 (0xd) Line number=0
[rviz2-3] [ERROR] [1765891191.402375153] [moveit_ros.planning_scene_monitor.planning_scene_monitor]: Robot model not loaded
~~~

#### 描述

**RViz 缺参数**：RViz 启动时，其中的 MotionPlanning 插件也需要知道机器人的语义描述（SRDF，即包含 Group 定义、姿态定义的那个文件）。但是它没有这个参数

#### 问题解决

 ROS 2 的 Node 定义中：

-   **parameters**: 用于传递 ROS 参数（比如你的 robot_description、srdf、kinematics 字典）
-   **arguments**: 用于传递命令行参数（比如 "-d", rviz_config_path）

而我错误的传入了参数

改为

~~~py
node_rviz = Node(
        package="rviz2",
        executable="rviz2",
        name="rviz2",
        output="screen",
        # 修正点 1: 将参数字典传给 parameters
        parameters=common_params,
        # 修正点 2: 仅将命令行标志传给 arguments
        arguments=["-d", rviz_config_path]
    )
~~~

