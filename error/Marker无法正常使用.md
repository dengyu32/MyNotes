### 报错

~~~bash
[rviz2-5] Warning: class_loader.impl: SEVERE WARNING!!! A namespace collision has occurred with plugin factory for class rviz_default_plugins::displays::InteractiveMarkerDisplay. New factory will OVERWRITE existing one. This situation occurs when libraries containing plugins are directly linked against an executable (the one running right now generating this message). Please separate plugins out into their own library or just don't link against the library and use either class_loader::ClassLoader/MultiLibraryClassLoader to open.
[rviz2-5]          at line 253 in /opt/ros/humble/include/class_loader/class_loader/class_loader_core.hpp

~~~

![image-20251221083520067](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/error_marker.png) 

移动Marker时脱离机械臂末端

plugin factory（插件工厂）
RViz 是插件化架构，每一种显示（Grid、TF、InteractiveMarker）都是插件。

namespace collision（命名空间冲突）
同一个插件类，被加载了两次

OVERWRITE existing one（覆盖）
 后加载的插件，把先加载的插件实例覆盖了



luanch中去掉move_group_node的use_sim_time即可