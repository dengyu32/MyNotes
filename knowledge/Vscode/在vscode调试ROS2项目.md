### 在 VSCode 调试 ROS2 项目

>   本笔记整理了在 VSCode 下调试 ROS2 功能包的流程，包含配置、编译和运行步骤。

------

### 要求

1.  功能包中需要有可执行文件。
2.  在功能包根目录 `CMakeLists.txt` 中需包含：

```
install(TARGETS executable_1 executable_2
    DESTINATION lib/${PROJECT_NAME}
)
```

------

### 流程

1.  **配置 launch.json**

```
{
  "configurations": [
    {
      "name": "(gdb) 启动",
      "type": "cppdbg",
      "request": "launch",
      "program": "${workspaceFolder}/install/${input:package}/lib/${input:package}/${input:executable}",
      "args": [],
      "stopAtEntry": false,
      "cwd": "${fileDirname}",
      "environment": [],
      "externalConsole": false,
      "MIMode": "gdb",
      "setupCommands": [
        {
          "description": "为 gdb 启用整齐打印",
          "text": "-enable-pretty-printing",
          "ignoreFailures": true
        },
        {
          "description": "将反汇编风格设置为 Intel",
          "text": "-gdb-set disassembly-flavor intel",
          "ignoreFailures": true
        }
      ]
    }
  ],
  "inputs": [
    {
      "id": "package",
      "type": "promptString",
      "description": "package_name",
      "default": "engineer_control_driver"
    },
    {
      "id": "executable",
      "type": "promptString",
      "description": "executable_name",
      "default": "engineer_robot_controller"
    }
  ]
}
```

1.  **安装 GDB**

```
sudo apt install gdbserver
sudo apt install gdb
```

2.   **编译功能包**

```
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=RelWithDebInfo
```

>   如果编译报错，可删除 `install`、`build`、`log` 文件夹后重新编译。

3.   **加载环境**

```
source install/setup.bash
```

4.   **获取可执行文件地址**

```
ros2 run --prefix 'gdbserver localhost:3000' package_name executable_name
```

------

### 参考

-   [YouTube 教程](https://www.youtube.com/watch?v=LDAM9aoDe4g&t=174s)
