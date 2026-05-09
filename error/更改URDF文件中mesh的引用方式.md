### 更改 URDF 文件中 mesh 的引用方式

>   主要说明在使用 Gazebo + ros2_control 进行仿真时，由于 URDF 中 mesh 路径引用方式不当引发的黑屏、闪退问题。

#### 问题描述

>   在 URDF 中引用 mesh（dae、stl 等）时，如果使用 `package://claw_description` 这种 ROS 路径查找方式，Gazebo 无法识别，会导致启动时卡死或闪退。

表现：

-   Gazebo 黑屏
-   Gazebo 闪退
-   Gazebo 卡在加载界面不动

#### 问题原因

>   Gazebo 不支持 `package://` 格式的路径解析，它需要绝对路径。如果继续使用相对格式，Gazebo 会直接卡死。

特别说明：

-   Gazebo 能用 `${find claw_description}` 这种方式找到绝对路径替换
-   但 **RViz 不认可这种绝对路径写法**

典型冲突：

-   Gazebo：不认 package://
-   RViz：不认绝对路径

| 工具   | 支持的路径格式 | 使用文件          | 推荐写法                          |
| ------ | -------------- | ----------------- | --------------------------------- |
| RViz   | package://     | robot.urdf        | 保留 package://                   |
| Gazebo | 绝对路径       | robot_gazebo.urdf | 使用 $(find pkg) 自动替换绝对路径 |

#### 解决方案

>   在 URDF 中修改 mesh 引用方式，将 `package://...` 改成 `${find PKG_NAME}` 的形式，使 Gazebo 能正确解析绝对路径。

准备两个urdf

示例：
 把以下写法（Gazebo 无法识别）：

```
mesh filename="package://claw_description/meshes/xxx.stl"
```

改为（Gazebo 可识别）：

```
mesh filename="$(find claw_description)/meshes/xxx.stl"
```

