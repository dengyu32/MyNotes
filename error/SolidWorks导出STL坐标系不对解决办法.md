### 问题

SolidWorks 导出的 STL 在 RViz / MoveIt / Gazebo 里方向不对，常见现象：

- 模型看起来被镜像或翻转
- 机械臂末端坐标系和模型朝向对不上
- 同一个零件在 SolidWorks 里正常，导出后坐标轴方向异常

### 原因

SolidWorks 导出 STL 时，若启用了将数据转换到正坐标空间的处理，会改变原始模型坐标基准，导致外部软件加载后坐标系不一致。

### 解决办法

导出 STL 时进入 `选项`，勾选：

- `不要转换 STL 输出数据到正的坐标空间`

![37aefac03daa986188b03e7ada890f7d](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/SolidWorks%E5%AF%BC%E5%87%BASTL%E9%80%89%E9%A1%B9.jpg)

### 验证

1. 重新导出 STL 并替换原 mesh 文件。  
2. 在 RViz/Gazebo 中重新加载 URDF。  
3. 检查模型与 `base_link`、工具坐标系方向是否一致。  

如果仍有偏差，再检查：

- URDF 中 `<origin xyz="" rpy="">` 是否有额外旋转
- 是否误用了旧 STL 缓存文件
