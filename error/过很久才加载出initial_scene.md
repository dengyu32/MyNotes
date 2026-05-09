#### 问题

启动demo.launch.py过很久才加载出初始位置

![image-20251219160652573](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/error_initial_scene.png)

stl的模型面数太多,精度太高,导致RViz启动慢,甚至需要实时更新位置的交互式标记(Interactive Marker)无法显示

#### 解决

推测是STL精度过高导致的卡顿问题

尝试将STL转为低精度的stl,成功解决