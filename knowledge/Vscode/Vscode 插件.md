### VSCode 插件

>   本笔记整理了 ROS2/C++ 开发常用 VSCode 插件及配置方法，包括调试、代码补全和可视化工具。

------

### 总览

#### 必选插件

-   **Chinese**
-   **C/C++**
    -   C/C++ Extension Pack
    -   C/C++ Themes
-   **CMake**
    -   CMake Tools
    -   cmake-format
-   **CodeLLDB**
-   **Jinja**
-   **Python**
-   **ROS**
-   **URDF Visualizer**

#### 其他插件

-   clangd
-   git history
-   GitHub Copilot
-   Jupyter
-   Remote SSH
-   vscode-pdf

------

### 配置

#### CodeLLDB

1.  **单个项目调试**

```
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "lldb",
      "request": "launch",
      "name": "Debug C++ Program",
      "program": "${workspaceFolder}/build/main",
      "args": [],
      "cwd": "${workspaceFolder}",
      "stopOnEntry": false
    }
  ]
}
```

2.   **ROS2 节点调试**

```
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "lldb",
      "request": "launch",
      "name": "Debug ROS2 Node",
      "program": "${workspaceFolder}/install/clb_bringup/lib/clb_bringup/clb_demo_node",
      "args": [],
      "cwd": "${workspaceFolder}",
      "env": {
        "AMENT_PREFIX_PATH": "/home/wrj/ws_moveit/install:/opt/ros/humble",
        "LD_LIBRARY_PATH": "/home/wrj/ws_moveit/install/lib:/opt/ros/humble/lib"
      },
      "stopOnEntry": false
    }
  ]
}
```

>   编译方式：

```
colcon build --symlink-install
```

F5 运行调试配置。

------

#### Clangd

1.  **安装依赖**

```
sudo apt install g++-12 libstdc++-12-dev 
sudo apt install clang clangd libclang-dev libstdc++-9-dev
```

2.   **设置环境变量**

```
export CC=/usr/bin/gcc
export CXX=/usr/bin/g++
```

3.   **用户配置（Preference: Open Settings(JSON)）**

```
{
  "files.associations": {
    "iostream": "cpp",
    "intrinsics.h": "c",
    "ostream": "cpp",
    "vector": "cpp"
  },
  "editor.formatOnPaste": true,
  "editor.formatOnSave": true,
  "editor.formatOnType": true,
  "C_Cpp.errorSquiggles": "disabled",
  "C_Cpp.intelliSenseEngine": "disabled",
  "C_Cpp.autocomplete": "disabled",
  "clangd.path": "/usr/bin/clangd",
  "clangd.arguments": [
    "--log=verbose",
    "--pretty",
    "--all-scopes-completion",
    "--completion-style=bundled",
    "--cross-file-rename",
    "--header-insertion=iwyu",
    "--header-insertion-decorators",
    "--background-index",
    "--clang-tidy",
    "--clang-tidy-checks=cppcoreguidelines-*,performance-*,bugprone-*,portability-*,modernize-*,google-*",
    "-j=2",
    "--pch-storage=disk",
    "--function-arg-placeholders=false",
    "--compile-commands-dir=build"
  ],
  "cmake.ignoreCMakeListsMissing": true,
  "security.workspace.trust.untrustedFiles": "open",
  "workbench.settings.applyToAllProfiles": []
}
```

4.   **工作区配置**

```
{
  "workbench.iconTheme": "vs-seti",
  "editor.minimap.enabled": false,
  "editor.mouseWheelZoom": true,
  "explorer.confirmDelete": false,
  "files.autoSave": "onFocusChange",
  "C_Cpp.intelliSenseEngine": "disabled",
  "git.autofetch": true,
  "clangd.path": "/usr/bin/clangd",
  "clangd.arguments": [
    "--compile-commands-dir=${workspaceFolder}/build",
    "--background-index=false",
    "-j=12",
    "--all-scopes-completion",
    "--completion-style=detailed",
    "--header-insertion=iwyu",
    "--query-driver=/usr/bin/clang++,/usr/bin/g++",
    "--clang-tidy",
    "--enable-config",
    "--fallback-style=WebKit",
    "--pretty"
  ],
  "clangd.fallbackFlags": [
    "-std=c++17",
    "-I/opt/ros/humble/include/**",
    "-I/usr/include/x86_64-linux-gnu/qt5",
    "-I/usr/include/x86_64-linux-gnu/qt5/QtCore",
    "-I/home/wrj/code/ws_myEngineer/install/**/include",
    "-I/home/wrj/code/ws_myEngineer/include",
    "-I${workspaceFolder}"
  ]
}
```

5.   **ROS2 项目使用 clangd**

-   在每个功能包 `CMakeLists.txt` 添加：

```
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# clangd in ROS2
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
```

>   用于生成 `compile_commands.json` 文件，配置完成无显示可重启 VSCode。
>    clangd 报 `rclcpp.hpp` 错误，可检查是否设置 `CMAKE_CXX_STANDARD`。

------

#### URDF Visualizer

-   用于 URDF 可视化，方便检查机器人模型。
-   配置示例：

```
"urdf-visualizer.packages": {
    "my_package":"src/my_package"
}
```
