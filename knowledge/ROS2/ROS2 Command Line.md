## ROS2 常用命令行

>   面向 Ubuntu 20.04 / ROS2（Foxy/Humble）/ MoveIt2 上位机开发的常用 CLI（Command Line Interface，命令行接口）总结，方便快速查找包、节点、话题、服务、动作、控制器以及常用调试操作。

------

### 1. 包与节点管理

>   用于查询包信息、查看可执行文件、运行节点和启动 launch 文件。

| 功能             | 命令                            | 示例                                     |
| ---------------- | ------------------------------- | ---------------------------------------- |
| 查看已安装包     | `ros2 pkg list`                 |                                          |
| 查包安装路径     | `ros2 pkg prefix <pkg>`         | `ros2 pkg prefix rclcpp`                 |
| 查包内可执行文件 | `ros2 pkg executables <pkg>`    | `ros2 pkg executables demo_nodes_cpp`    |
| 运行节点         | `ros2 run <pkg> <exe>`          | `ros2 run demo_nodes_cpp talker`         |
| 启动 launch 文件 | `ros2 launch <pkg> <launch.py>` | `ros2 launch my_robot bringup.launch.py` |

------

### 2. Topic（话题）操作

>   ROS2 中最基本的通信方式，命令用于查看、监听、发布以及检查消息结构。

| 功能         | 命令                                  | 示例                                                   |
| ------------ | ------------------------------------- | ------------------------------------------------------ |
| 列出话题     | `ros2 topic list`                     |                                                        |
| 查看话题数据 | `ros2 topic echo <topic>`             | `ros2 topic echo /joint_states`                        |
| 发布测试消息 | `ros2 topic pub <topic> <msg> <data>` | `ros2 topic pub /chatter std_msgs/String "data: 'hi'"` |
| 查看话题信息 | `ros2 topic info <topic>`             |                                                        |
| 查看消息结构 | `ros2 interface show <msg>`           | `ros2 interface show std_msgs/msg/String`              |

------

### 3. Service（服务）

>   Request/Response 模式，适用于同步查询型任务。

| 功能         | 命令                                    | 示例                                          |
| ------------ | --------------------------------------- | --------------------------------------------- |
| 列出服务     | `ros2 service list`                     |                                               |
| 查看服务类型 | `ros2 service type <srv>`               |                                               |
| 调用服务     | `ros2 service call <srv> <type> <args>` | `ros2 service call /reset std_srvs/srv/Empty` |

------

### 4. Action（动作）

>   适用于带目标、可反馈、可取消的长耗时任务（如 MoveIt 轨迹执行）。

| 功能         | 命令                                           | 示例                                                         |
| ------------ | ---------------------------------------------- | ------------------------------------------------------------ |
| 列出动作     | `ros2 action list`                             |                                                              |
| 查看动作结构 | `ros2 action show <type>`                      | `ros2 action show control_msgs/action/FollowJointTrajectory` |
| 发送动作目标 | `ros2 action send_goal <action> <type> <goal>` | `ros2 action send_goal /arm_controller/follow_joint_trajectory control_msgs/action/FollowJointTrajectory '{...}'` |

------

### 5. 参数管理（param）

>   用于动态查看与修改节点参数，支持加载 yaml 文件。

| 功能         | 命令                                    | 示例 |
| ------------ | --------------------------------------- | ---- |
| 列出参数     | `ros2 param list`                       |      |
| 获取参数值   | `ros2 param get <node> <param>`         |      |
| 设置参数     | `ros2 param set <node> <param> <value>` |      |
| 加载参数文件 | `ros2 param load <node> <file>`         |      |
| 导出参数     | `ros2 param dump <node>`                |      |

------

### 6. 节点管理

>   查看当前系统中的节点与节点内部的发布/订阅/服务等信息。

| 功能         | 命令                    | 示例 |
| ------------ | ----------------------- | ---- |
| 查看所有节点 | `ros2 node list`        |      |
| 查看节点信息 | `ros2 node info <node>` |      |

------

### 7. 接口类型管理

>   查看所有消息（msg）、服务（srv）、动作（action）接口类型及其结构。

| 功能         | 命令                         | 示例 |
| ------------ | ---------------------------- | ---- |
| 查看所有接口 | `ros2 interface list`        |      |
| 查看消息列表 | `ros2 msg list`              |      |
| 查看服务列表 | `ros2 srv list`              |      |
| 查看动作列表 | `ros2 action list`           |      |
| 查看接口结构 | `ros2 interface show <type>` |      |

------

### 8. ros2_control 控制器管理

>   工业机器人与机械臂常用模块，用于管理控制器生命周期。

| 功能             | 命令                                                    |
| ---------------- | ------------------------------------------------------- |
| 查看已加载控制器 | `ros2 control list_controllers`                         |
| 查看控制器类型   | `ros2 control list_controller_types`                    |
| 加载控制器       | `ros2 control load_controller --controller-name <name>` |
| 激活控制器       | `ros2 control set_controller_state <name> active`       |
| 加载并激活       | `ros2 control load_start_controller <name>`             |
| 停用控制器       | `ros2 control set_controller_state <name> inactive`     |
| 卸载控制器       | `ros2 control unload_controller <name>`                 |
| 查看硬件接口     | `ros2 control list_hardware_interfaces`                 |

------

### 9. 调试工具命令

>   包含系统诊断、bag 录制/回放、话题带宽频率监测等。

| 功能           | 命令                    | 说明             |
| -------------- | ----------------------- | ---------------- |
| 检查 ROS 状态  | `ros2 doctor`           | 基础诊断工具     |
| 记录所有话题   | `ros2 bag record -a`    | -a：记录全部话题 |
| 播放 bag 文件  | `ros2 bag play <file>`  | 用于仿真回放     |
| 话题频率（Hz） | `ros2 topic hz <topic>` | 检查实时性       |
| 话题带宽（bw） | `ros2 topic bw <topic>` | 查看数据流量     |

------

### 10. colcon 工作空间构建

>   ROS2 默认构建工具，用于编译、选择性构建、清缓存等。

| 功能            | 命令                                   | 示例 |
| --------------- | -------------------------------------- | ---- |
| 构建全部包      | `colcon build`                         |      |
| 构建指定包      | `colcon build --packages-select <pkg>` |      |
| 软链接构建      | `colcon build --symlink-install`       |      |
| 清理 CMake 缓存 | `colcon build --cmake-clean-cache`     |      |
| 查看工作区列表  | `colcon list`                          |      |
| 加载环境变量    | `source install/setup.bash`            |      |

------

### 推荐 alias（写入 ~/.bashrc）

>   常用调试命令的快捷 alias，减少敲命令时间。

```
alias rctl='ros2 control list_controllers'
alias rcla='ros2 control load_start_controller'
alias rcli='ros2 control set_controller_state'
alias rcle='ros2 control unload_controller'
alias rtopic='ros2 topic list'
alias rnodes='ros2 node list'
alias rparam='ros2 param list'
alias rbuild='colcon build --symlink-install && source install/setup.bash'
```

------

### 查看 ROS 版本

>   查看当前 shell 中激活的 ROS 发行版名称。

```
echo $ROS_DISTRO
printenv | grep ROS_DISTRO
```