# joint_limits.yaml 必须使用浮点数（MoveIt 加载失败问题）

>   说明使用 MoveIt2 + RViz 时，由于 joint_limits 配置为整数导致 SRDF 无法解析，从而引发 Robot Model 加载失败的问题。

### 一、问题描述

>   MoveIt 可视化插件在导入参数时，由于 joint_limits.yaml 内容格式不符合预期，导致 SRDF 加载失败，整个模型无法在 RViz 中正常显示。

-   **moveit_ros_visualization** 导入参数时
     **RViz** 找不到参数 `robot_description_semantic`
-   SRDF 无法解析，报错：**XML_ERROR_EMPTY_DOCUMENT, ErrorID=13**
-   **moveit_rdf_loader.rdf_loader** 无法解析 SRDF 文件
-   **planning_scene_monitor** 未能加载 Robot Model
-   MotionPlanning 插件报错：**No Planning Scene Loaded**

典型终端提示：

```
[rviz2-4] [ERROR] Could not find parameter robot_description_semantic
[rviz2-4] Error: Could not parse the SRDF XML File. 
Error=XML_ERROR_EMPTY_DOCUMENT ErrorID=13
[rviz2-4] [ERROR] Unable to parse SRDF
[rviz2-4] [ERROR] Robot model not loaded
```

RViz 显示：

-   MotionPlanning Status: Error PlanningScene
-   No Planning Scene Loaded
-   Robot Description 为空

### 二、问题分析

>   SRDF 空文档错误通常意味着 MoveIt 在某个环节没有正确读取配置文件。

关键点在以下报错：

-   **无法解析 SRDF，ErrorID=13（空文档）**

查阅 Robotics Q&A 网站后发现，根因来自 **joint_limits.yaml（或 joint_limits.xml）里关节限制不是浮点数**。

英文原文说明内容：

```
Movelt2 Setup Assistant generates integer values for joint limits by default,
while MoveIt expects them to be doubles (floating-point numbers).
```

解释：

-   MoveIt2 Setup Assistant 默认生成 **整数 joint limits**
-   MoveIt 主程序实际期望 **浮点数格式**
-   导致读取 joint_limits 时失败 → SRDF 解析中断 → 机器人模型无法加载

### 三、背后原理

>   MoveIt 的 joint_limits 解析器严格要求关节限制为浮点类型（double），如果写成整数，内部 XML 解析器会认为节点格式非法，最终导致 SRDF 被判定为空文档，触发 ErrorID=13。