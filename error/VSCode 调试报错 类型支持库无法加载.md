### VSCode 调试报错 类型支持库无法加载

>   说明在 VSCode 调试 ROS2 程序时，由于缺少类型支持库导致 subscription 创建失败、终端报错、调试器异常退出的问题。

#### 一、调试问题描述

>   调试过程中，VSCode 显示 subscription.hpp 报错，终端提示无法加载类型支持库。

-   调试运行时出现 subscription.hpp 报错
-   终端提示无法加载 `libbase_interfaces__rosidl_typesupport_fastrtps_cpp.so`
-   subscription.c 中出现 **invalid allocator** 错误
-   订阅者创建失败

终端报错信息：

```
[rcutils|error_handling.c:108] rcutils_set_error_state()
Type support not from this implementation.
Could not load library libbase_interfaces__rosidl_typesupport_fastrtps_cpp.so
invalid allocator, at ./src/rcl/subscription.c:219
terminate called after throwing 'RCLError'
```

#### 二、问题分析

>   关键问题是 **未能正确加载类型支持库**，导致 rcl 层创建订阅者失败。

核心点：

-   最初错误：**缺少类型支持库（.so 文件）**
-   导致 subscription 创建失败，报 **message_memory_strategy** 异常
-   最终触发 **invalid allocator** 报错

相关说明：

-   `libbase_interfaces__rosidl_typesupport_fastrtps_cpp.so` 属于动态链接库
-   `base_interfaces` 为自定义消息包
-   `_rosidl_typesupport_fastrtps_cpp` 表示 **Fast DDS 的 C++ 类型支持**

#### 三、ROS2 消息传输的三层结构

>   缺少类型支持库，会导致 DDS 无法进行序列化解析，订阅者/发布者无法创建。

消息定义示例：

```
src/base_interfaces/msg/IsThereOre.msg
uint8 UNCHANGED = 0
uint8 NO = 1
uint8 YES = 2
uint8 has_ore_on_left
uint8 has_ore_on_right
```

ROS2 编译流程：

1.  `.msg` → `.idl`
2.  生成 C++ 序列化/反序列化代码
3.  生成类型支持库（.so）
     包括：
    -   IsThereOre 的序列化逻辑
    -   反序列化逻辑
    -   FastDDS 所需元数据
    -   Topic 类型注册信息

如果 `.so` 文件缺失：

-   订阅者无法告诉 DDS“这个消息怎么传输”
-   rclcpp 创建 subscription 时直接失败

#### 四、类型支持库的生成方式

>   CMakeLists 会通过 `rosidl_generate_interfaces` 自动生成类型支持库。

示例：

```
rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/ArmMode.msg"
  "msg/IsThereOre.msg"
  "msg/MoveGroupStatus.msg"
  "msg/PumpStatus.msg"
  "msg/RobotMode.msg"
)
```

编译成功后，会生成：

```
install/base_interfaces/lib/libbase_interfaces__rosidl_typesupport_fastrtps_cpp.so
```

目前该文件已在目录存在，因此进一步推测：

-   VSCode 调试时 **没有执行 source install/setup.bash**
-   导致环境变量 `LD_LIBRARY_PATH` 未包含安装目录
-   最终 GDB 调试器加载失败

终端表现：

```
echo $LD_LIBRARY_PATH
-var-create: unable to create variable object
```
