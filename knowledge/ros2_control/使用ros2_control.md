### 如何使用 ros2_control 框架

>   ros2_control 提供了统一的控制接口，用于管理机器人硬件、控制器和关节状态，支持模拟和真实硬件。

------



>   参考： https://blog.csdn.net/qq_27865227/article/details/132503010?spm=1001.2014.3001.5501
>
>   https://control.ros.org/rolling/doc/resources/resources.html
>
>   梳理这三种ros2_control控制链
>
>   ros与rviz交互
>
>   控制fake_system
>
>   控制gazebo
>
>   并给出可以使用的launch文件

![image-20251211154500916](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/20251211154504946.png)

#### **使用流程**

1.  **定义机器人模型**

    -   使用 **URDF** 或 **SDF** 文件描述机器人的结构。

    -   指定每个关节、传感器、执行器以及对应的控制接口。

    -   可以添加传感器插件、关节限制和传动系统信息。

    -   示例：

        ```
        <joint name="joint_1" type="revolute">
          <origin xyz="0 0 0" rpy="0 0 0"/>
          <limit lower="-1.57" upper="1.57" effort="10" velocity="1.0"/>
        </joint>
        ```

2.  **配置 ros2_control**

    -   创建 **YAML 配置文件**，指定硬件接口类型（如 `fake_system` 或自定义硬件接口）。

    -   配置每个控制器的类型（position, velocity, effort）以及绑定的关节。

    -   示例：

        ```
        controller_manager:
          ros__parameters:
            update_rate: 100
            joint_state_broadcaster:
              type: joint_state_broadcaster/JointStateBroadcaster
            position_controller:
              type: position_controllers/JointTrajectoryController
              joints: [joint_1, joint_2]
        ```

3.  **启动 ros2_control_node**

    -   使用 **controller_manager** 包中的 `ros2_control_node` 可执行文件。
    -   节点会加载机器人模型和 YAML 配置文件。
    -   可选择加载 **fake_system** 或实际硬件接口。

4.  **管理控制器**

    -   使用 **controller_manager** 提供的接口加载、启动、停止控制器。

    -   命令示例：

        ```
        ros2 control load_controller joint_state_broadcaster
        ros2 control load_controller position_controller
        ros2 control switch_controller --start position_controller
        ```

5.  **与控制器交互**

    -   通过发布命令到控制器主题，可以控制关节位置、速度或力矩。

    -   例如：

        ```
        ros2 topic pub /position_controller/joint_trajectory_controller/commands trajectory_msgs/JointTrajectory "{...}"
        ```

6.  **监控状态**

    -   订阅关节状态和传感器数据，获取机器人当前状态。
    -   可用于调试、验证控制算法或反馈闭环控制。
    -   典型话题：
        -   `/joint_states`
        -   `/controller_manager/list_controllers`





### 如何使用 fake_system

>   `fake_system` 可以模拟机器人硬件，使开发者在没有真实机器人时调试控制器和控制算法。以下是完整使用流程：

------

#### **使用步骤**

1.  **定义机器人描述**

    -   在机器人描述文件（URDF 或 SDF）中，定义关节、传感器等部件。
    -   指定它们使用 **fake_system** 作为硬件接口。
    -   这样控制器就可以读取和操作这些虚拟关节。

2.  **配置 ros2_control**

    -   在 ros2_control 的 YAML 配置文件中，将 **fake_system** 设置为硬件接口。

    -   配置相关参数，例如关节名称、初始位置、最大速度等。

    -   例子：

        ```
        hardware_interface: fake_system/GenericSystem
        joints:
          - joint_1
          - joint_2
          - joint_3
        ```

3.  **启动 ros2_control_node**

    -   启动节点后，fake_system 会被加载，并开始模拟关节状态。
    -   节点会持续发布 `sensor_msgs/JointState` 消息，就像真实硬件一样。

4.  **加载控制器**

    -   使用 **controller_manager** 来加载并启动所需控制器（如 `joint_state_broadcaster`、`position_controllers` 等）。
    -   控制器将与 fake_system 交互，获取模拟关节状态，并执行控制命令。

5.  **测试和验证**

    -   向控制器发送命令，观察 fake_system 的响应。
    -   可以验证控制算法、轨迹规划、PID 调节等功能。
    -   整个过程无需真实机器人硬件，减少风险与调试成本。

------

#### **注意事项**

-   fake_system 只模拟关节状态和简单动力学，不具备真实硬件的精确力学特性。
-   对于复杂机器人或高精度控制，仍需在真实硬件上进行最终验证。
-   可以结合 **joint_state_broadcaster** 或其他控制器，实现数据流统一和更复杂的仿真测试。





### 如何切换到真实硬件

>   在完成 fake_system 的模拟测试后，可以将系统切换到实际机器人硬件，以进行真实控制和验证。

------

#### **切换步骤**

1.  **修改机器人描述文件**

    -   在 URDF 或 SDF 文件中，将各个关节和传感器的硬件接口从 **fake_system** 替换为实际的硬件接口。

    -   确保每个关节名称和硬件接口对应正确。

    -   例子：

        ```
        <transmission name="joint_1_trans">
          <type>transmission_interface/SimpleTransmission</type>
          <hardwareInterface>hardware_interface/PositionJointInterface</hardwareInterface>
          <joint name="joint_1"/>
        </transmission>
        ```

2.  **更新 ros2_control 配置**

    -   在 ros2_control 的 YAML 文件中，将 **hardware_interface** 设置为实际机器人所使用的接口（如 `ros2_control_hardware/GenericSystem` 或自定义硬件接口）。
    -   配置参数如关节名称、控制模式、最大力矩、最大速度等。

3.  **启动 ros2_control_node**

    -   节点将加载实际硬件接口，并与机器人硬件进行通信。
    -   机器人关节状态会实时发布，控制命令会直接作用于物理关节。

4.  **加载控制器**

    -   使用 **controller_manager** 来加载和启动控制器。
    -   控制器会与真实硬件交互，实现位置、速度或力矩控制。

5.  **验证和测试**

    -   初次运行时建议在低速或安全模式下测试，观察关节响应。
    -   可逐步增加运动幅度或速度，确保控制器和硬件通信正常。
    -   在真实硬件上运行控制算法前，确认所有安全机制（限位、急停）已启用。

------

#### **注意事项**

-   确保硬件接口驱动已经安装并正常工作。
-   控制器和硬件接口的参数必须匹配，避免发送错误命令导致硬件损坏。
-   切换到真实硬件前，可以先在模拟环境中充分测试，减少风险。
-   对于多关节机器人，建议分批次加载控制器，逐步验证硬件响应。

