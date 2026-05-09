#### ROS2 组件

>   ROS2 Component（可组合节点）开发流程、核心概念、配置方法与常用指令

------

**开发流程**

>   推荐的开发顺序

先将节点独立运行、调试并验证功能稳定性，再将其转换为 **ComposableNode（可组合组件）**，以便在容器中进行进程内通信优化。

------

**基础知识**

>   理解 Composition 机制的核心概念

**Composable Node**

原本 `rclcpp::Node` 以可执行程序方式运行。  
可组合节点需要编译为 **共享库（SHARED Library）**。  
可在同一进程内加载多个节点。  
通过 **进程内通信（Intra-process communication）** 获得零拷贝性能优势：避免序列化、极大降低延迟、适合大数据与频繁消息通信场景（类似 ROS1 的 nodelet）。

**Container**

`rclcpp_components` 提供加载组件的容器进程。  
常见两类容器：  
`component_container`（单线程）  
`component_container_mt`（多线程）  
容器负责动态加载共享库并创建节点实例，节点运行在同一进程中。

**Pluginlib / class_loader**

组件作为共享库对外暴露工厂接口。  
容器通过 `class_loader` 动态加载并实例化。  
使用 `RCLCPP_COMPONENTS_REGISTER_NODE` 注册类。

**NodeOptions**

允许用于：启用进程内通信（`use_intra_process_comms`）、参数声明、重映射等。  
组件构造函数必须接受：

```
MyNode(const rclcpp::NodeOptions & options)
```

**ComposableNode / ComposableNodeContainer（launch API）**

在 `launch.py` 中使用：  
`ComposableNodeContainer` 创建容器进程。  
`ComposableNode` 指定要加载的组件。  
支持 `extra_arguments` 传入 NodeOptions（如启用进程内通信）。

------

**配置方法**

>   从源代码到 launch 加载的完整流程

**一、C++ 源码配置**

在组件类实现文件（cpp）中注册组件：

```
#include "rclcpp_components/register_node_macro.hpp"

RCLCPP_COMPONENTS_REGISTER_NODE(my_composition::MyNode)
```

------

**二、CMakeLists.txt 配置**

>   组件必须以 **共享库** 形式编译

```
cmake_minimum_required(VERSION 3.5)
project(my_composition_pkg)

set(CMAKE_CXX_STANDARD 17)

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclcpp_components REQUIRED)
find_package(std_msgs REQUIRED)

# 构建共享库组件
add_library(my_talker_component SHARED src/my_talker.cpp)

ament_target_dependencies(my_talker_component
  rclcpp rclcpp_components std_msgs
)

# 注册组件（可选生成可执行文件）
rclcpp_components_register_node(my_talker_component
  PLUGIN "my_composition::MyTalker"
  EXECUTABLE my_talker_exec
)

install(TARGETS
  my_talker_component
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

ament_package()
```

------

**三、package.xml 配置**

>   确保加入组件系统依赖

```
<?xml version="1.0"?>
<package format="2">
  <name>my_composition_pkg</name>
  <version>0.0.0</version>
  <description>example</description>
  <maintainer email="you@example.com">you</maintainer>
  <license>Apache-2.0</license>

  <buildtool_depend>ament_cmake</buildtool_depend>

  <depend>rclcpp</depend>
  <depend>rclcpp_components</depend>
  <depend>std_msgs</depend>
</package>
```

------

**四、launch.py 中动态加载组件**

>   使用 `ComposableNodeContainer` 将多个组件加载进同一进程

```
from launch import LaunchDescription
from launch_ros.actions import ComposableNodeContainer
from launch_ros.descriptions import ComposableNode

def generate_launch_description():
    container = ComposableNodeContainer(
        name='my_container',
        namespace='',
        package='rclcpp_components',
        executable='component_container_mt',
        composable_node_descriptions=[
            ComposableNode(
                package='my_composition_pkg',
                plugin='my_composition::MyTalker',
                name='my_talker',
                extra_arguments=[{'use_intra_process_comms': True}],
            ),
        ],
        output='screen',
    )
    return LaunchDescription([container])
```

------

**五、运行方式**

>   支持静态加载和动态加载两种方式

**静态加载（使用 launch）**

```
ros2 launch my_pkg my_system_launch.py
```

**动态加载（手动加载组件）**

```
ros2 run rclcpp_components component_container_mt --ros-args -r __node:=my_container
```

加载组件：

```
ros2 component load /my_container pkg_a pkg_a::NodeA
ros2 component load /my_container pkg_b pkg_b::NodeB
```

------

**指令与工具**

>   ROS2 component 系统的常用指令

查看帮助：

```
ros2 component -h
```

查看所有可加载组件类型：

```
ros2 component types
```

启动一个容器：

```
ros2 run rclcpp_components component_container
```

查看容器内运行的组件：

```
ros2 component list
```

加载 Talker/Listener：

```
ros2 component load /ComponentManager composition composition::Talker
ros2 component load /ComponentManager composition composition::Listener
```

加载服务端/客户端：

```
ros2 component load /ComponentManager composition composition::Server
ros2 component load /ComponentManager composition composition::Client
```

一次性加载多个组件：

```
ros2 run composition manual_composition
```

使用 dlopen 加载：

```
ros2 run composition dlopen_composition `ros2 pkg prefix composition`/lib/libtalker_component.so `ros2 pkg prefix composition`/lib/liblistener_component.so
```

用 launch 加载：

```
ros2 launch composition composition_demo.launch.py
```

卸载组件：

```
ros2 component unload /ComponentManager 1 2
```

重映射节点名及命名空间：

```
ros2 component load /ComponentManager composition composition::Talker --node-name talker3 --node-namespace /ns2
```

传递参数：

```
ros2 component load /ComponentManager image_tools image_tools::Cam2Image -p burger_mode:=true
ros2 component load /ComponentManager composition composition::Talker -e use_intra_process_comms:=true
```
