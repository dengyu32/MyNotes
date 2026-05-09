#### 问题

启动的是msa生成的jdemo.launch.py文件

![Generated Image December 19, 2025 - 3_49PM](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/error_ompl_fail.png)

#### 解决方法

错误的ompl_planning.yaml导致无法加载ompl规划库

在相应的moveit_config里去掉自己添加的ompl_planning.yaml即可

改成

~~~yaml
planning_plugin: ompl_interface/OMPLPlanner

request_adapters: >-
  default_planner_request_adapters/FixWorkspaceBounds
  default_planner_request_adapters/FixStartStateBounds
  default_planner_request_adapters/FixStartStateCollision
  default_planner_request_adapters/FixStartStatePathConstraints
  default_planner_request_adapters/AddTimeOptimalParameterization

start_state_max_bounds_error: 0.1

planner_configs:
  RRTConnectkConfigDefault:
    type: geometric::RRTConnect
    range: 0.2

group_planner_configs:
  engineer_arm:
    - RRTConnectkConfigDefault
~~~

也可以