### 错误

#### 描述

~~~bash
[move_group-3] [ERROR] ... Action client not connected to action server: elite_arm_controller/follow_joint_trajectory
~~~

规划成功，但是move_group无法将算好的路径传给elite_arm_controller控制器

因为没有启动ros2_control_node中的control_manager并且spawn这个elite_arm_controller

#### 解决

>   1.   urdf配置虚拟硬件接口fakeSystem
>   2.   launch添加ros2_control架构
>   3.   确认ros2_controllers.yaml（一般msa直接生成）

一、修改urdf

~~~xml
<ros2_control name="FakeSystem" type="system">
    <!-- 这里的插件非常重要，它是用来造假的 -->
    <hardware>
        <plugin>mock_components/GenericSystem</plugin>
    </hardware>
    
    <joint name="joint1"> <!-- 替换成你真实的关节名字 -->
        <command_interface name="position"/>
        <state_interface name="position">
          <param name="initial_value">0.0</param>
        </state_interface>
        <state_interface name="velocity"/>
    </joint>
    <!-- ... 对每一个关节都要写一遍 ... -->
</ros2_control>
~~~

二、修改launch文件

~~~py
  # 启动 ros2_control 架构 
    node_ros2_control = Node(
        package="controller_manager",
        executable="ros2_control_node",
        parameters=[robot_description, ros2_controllers],
        output="screen"
    )
    
    spawn_joint_state_broadcaster = Node(
        package="controller_manager",
        executable="spawner",
        arguments=["joint_state_broadcaster", "--controller-manager", "/controller_manager"],
        output="screen",
    )
    
    spawn_elite_arm_controller = Node(
        package="controller_manager",
        executable="spawner",
        arguments=["elite_arm_controller", "--controller-manager", "/controller_manager"],
        output="screen",
    )
~~~





### ros2_control链路分析

####  核心链路图 (The Loop)

这是一个闭环系统，分为下行指令（Command）和上行反馈（State/Feedback）。

~~~mermaid
graph TD
    %% 上层应用
    User[点击 MoveIt Plan & Execute] --> MoveIt[MoveIt]

    %% ros2_control 控制层
    subgraph Control_Layer [ros2_control 控制层]
        MoveIt -- FollowJointTrajectory Action --> CM[Controller Manager]

        CM -->|调用 update 执行轨迹插补| JTC[Joint Trajectory Controller]

        CM -->|read 读取状态| HW[Hardware Interface]
        CM -->|write 写入命令| HW
    end

    %% 硬件 / 仿真层
    subgraph Hardware_Layer [硬件 / 仿真层]
        HW -->|写入控制命令| Fake[Fake System / Mock Components]
    end

    %% 反馈层
    subgraph Feedback_Layer [状态反馈与可视化]
        HW -->|关节状态| JSB[Joint State Broadcaster]
        JSB -->|/joint_states| RSP[Robot State Publisher]
        RSP -->|/tf| RViz[RViz 可视化]
    end

    style Fake fill:#f9f,stroke:#333,stroke-width:2px
    style JTC fill:#9cf,stroke:#333,stroke-width:2px
    style CM fill:#cfc,stroke:#333,stroke-width:2px

~~~

