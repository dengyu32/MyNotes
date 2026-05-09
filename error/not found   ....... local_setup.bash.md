### not found : " ... local_setup.bash"

>   记录在更改或删除 ROS2 工作区路径后，
>
>   终端出现 local_setup.bash 找不到的常见错误及解决方案。

#### 报错分析

>   删除或移动工作区路径后，终端重新打开即报错

终端出现如下提示：

```
not found : " .../local_setup.bash"
```

该错误通常发生在删除或移动工作区（如 ws_moveit2）之后。

#### 原因

>   .bashrc 中的 source 路径失效导致加载失败

在 `.bashrc` 中通常会手动加入如下语句，用于自动加载 ROS2 工作区：

```
source ~/ws_moveit/install/setup.bash
```

该脚本会继续去加载：

```
local_setup.bash
```

当你删除、移动、重命名工作区后，原路径失效，终端每次启动都会尝试加载这个不存在的文件，从而报 *not found*。

#### 解决方案

>   清理旧构建文件并重新编译工作区，使路径重新生成

处理方式如下：

1.  删除旧工作区中的以下目录：
    -   build
    -   install
    -   log
2.  对原项目和 ws_moveit2 **全部重新编译**：

```
colcon build --symlink-install
```

这样会重新生成正确的 `local_setup.bash`，路径恢复正常，终端不再报错。



