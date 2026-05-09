## clangd 配置笔记

### 目录

[TOC]

### 原理

配置 clangd 最重要的是两部分，分别是：

**编译数据库和 .clangd**

具体作用机理如下：

  - compile_commands.json（仓库根）

    - 由 colcon build --cmake-args -DCMAKE_EXPORT_COMPILE_COMMANDS=ON 或脚本生成

        ROS 工程也可以在 CMakeLists.txt 中加入 set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

    - clangd 启动时自动加载，逐条提供：编译目录（directory）、源文件路径、完整编译参数（含 -I、宏、标准）。

    - 机理：当你打开某个源/头文件，clangd 选取最近的编译命令（或推断），用其中的目录 + 参数还原一次“假编译”，从而知道头文件搜索路径和宏定义。
    - 注意：header-only 包通常没有 TU，默认不会出现在 compile_commands.json，需要额外补齐（脚本生成合成条目或 .clangd Add）。

  - .clangd（仓库根）

    - 配置的 CompileFlags.Add：-std=c++17 和四个 -I../../...。
    - 机理：clangd 会把 Add 里的选项追加到它选中的那条编译命令后面（或推断命令），这样即便目标是 header-only / INTERFACE（编译数据库里没有单独条目），也能补全到缺失的 include 路径。
    - 相对路径为何生效：编译命令的工作目录是 build/<pkg>（来自 compile_commands.json 的 directory 字段），从该目录 ../../ 恰好回到仓库根，再指向 src/... 和 install/...。
    - compile_commands.json 里的 directory 字段就是 CMake 生成该编译命令时所在的工作目录。
      在 ROS2/colcon 默认配置下，每个包都做“out-of-source”构建，构建产物放在 build/<pkg>/，CMake 在这个目录下调用编译器，所以记录到数据库里的 directory 也就是 build/<pkg>。clangd 重放这条命令时，会把 directory 当作工作目录，去解析相对路径和 -I，因此我们在 .clangd 里用 ../../ 回到仓库根是基于这一点。
    - 如果有未被 cpp 跟踪的 hpp 文件时，需要配置 .clangd，否则会报错。
    - TU 是 Translation Unit（翻译单元），简单说就是编译器一次实际编译的源文件及其展开后的所有头文件。
          在 C/C++ 里，一个 .cpp 文件（加上它 #include 的所有内容）就是一个 TU。
          header-only 包没有 .cpp，所以不会生成 TU，也就不会出现在 compile_commands.json

  - --symlink-install（构建时选项）

    - 不是直接配置 clangd，但影响它解析头文件：安装目录变成指向源头文件的符号链接，
      install/... 时拿到的就是最新源，改头文件即刻生效，无需重建。

  - 源码中的 include 用法

    - 例如 #include "atcf_core/log_format.hpp"：依赖于 -I../../src/utils/atcf_core/include 或 -I../../install/atcf_core/include 才能解析；这些 -I 由 Add 补全或编译数据库提供。

  整体流程（一次补全或诊断时）：

    1. clangd 读取 compile_commands.json，确定当前文件用哪条编译命令；拿到工作目录和已有 -I。
        2. 将 .clangd 的 CompileFlags.Add 追加到该命令。
        3. 以步骤1的工作目录为基准解析追加的相对路径，构建完整的搜索路径集合。
        4. 用这个参数集做语义分析、补全、--check 等静态诊断。

  因此，对 clangd 生效的配置点就在：compile_commands.json（核心真实参数来源） + .clangd（追加缺口参数），其余构建选项（如 --symlink-install）是为了让这些路径指向最新、存在的头文件。



### 具体配置

>   自查

~~~ .clangd
当前 .clangd 配置
  CompileFlags:
    # 追加到每条编译命令；适合 header-only / INTERFACE 目标补
    # 全 include 路径
    Add:
      -std=c++17
      # compile_commands 的工作目录是 build/<pkg>，两级返回仓库根
      -I../../src/utils/atcf_core/include
      -I../../src/utils/atcf_ros/include
      -I../../install/atcf_core/include
      -I../../install/atcf_ros/include
  要点：使用 Add（新版支持），不要再用已废弃的 Fallback；路径
  全部相对，换电脑/路径也可用。
~~~

>   静态诊断思路

  - 基准：compile_commands.json 放在仓库根，clangd 自动读取。
  - 生成：colcon build --symlink-install --cmake-args
    -DCMAKE_EXPORT_COMPILE_COMMANDS=ON（或项目里的 ./
    gen_compile_commands.sh）。
  - 如果是 header-only 包，优先用脚本合并 + 合成编译命令，避免 clangd 对孤立头文件“猜测”失败。
  - header-only / INTERFACE 包缺少编译命令时，通过 .clangd 的
    Add -I 补齐。
  - 相对路径计算：编译目录是 build/<pkg>，所以头文件在仓库根
    需 ../../ 回溯两级。
  - 安装目录：推荐 --symlink-install，修改头文件后无需重建即
    可生效。
  - 检查命令：clangd --check=src/utils/atcf_ros/include/
    atcf_ros/log_macros.hpp 可验证 include 是否已展开为
    -I../../...。

>   常见错误与解决

    1. pp_file_not_found: atcf_core/log_format.hpp
       - 原因：clangd 命令的工作目录是 build/<pkg>，找不到头文
         件。
       - 处理：在 .clangd 用相对 -I../../src/utils/atcf_core/
         include 等补齐后，重启 clangd。
        2. rclcpp/rclcpp.hpp 或 rcl/guard_condition.h not found
       - 原因：header-only 包没有 TU，clangd 用“猜测命令”解析，缺失 ROS include。
       - 处理：用脚本合并 compile_commands + 合成头文件条目；或在 .clangd Add 里补齐。
        3. IncludeCleaner: resolved path ''
       - 实际是上一条导致的缺失路径，修好 include 后会消失。
        4. DefineOutline ==> FAIL: Couldn't find a suitable
           implementation file.
       - 纯头文件/宏定义找不到对应实现，提示级别，可忽略。
        5. Unknown CompileFlags key 'Fallback'
       - 表示使用了旧配置字段，改为 CompileFlags.Add。
        6. __GLIBC_PREREQ 相关报错
       - 原因：把 /opt/ros/humble/include/* 全部塞进 -I，误引入 dds/features.h 等冲突头。
       - 处理：优先沿用真实编译命令里的 -I/-isystem，不要硬编码扫整个目录。
        7. 头文件仍旧老版本
       - 若未用 --symlink-install，记得重建 colcon build
         --symlink-install；或手动清理旧的 install/ 头文件。

>   自查清单（排错顺序）

  - compile_commands.json 是否在仓库根，是否包含目标包条目。
  - header-only 头文件是否有合成条目（或 .clangd Add 是否补齐）。
  - .clangd 路径层级是否与 build/<pkg> 匹配（两级 ../../）。
  - install/atcf_core/include 等目录是否存在且为最新（优先用
    符号链接安装）。
  - AMENT_PREFIX_PATH / COLCON_INSTALL 是否指向正确的 ROS/安装目录。
  - VSCode 是否重启 clangd 扩展；若同时装了 C/C++ 扩展，禁用
    其 IntelliSense 避免干扰。
  - clangd --check <file> 输出中的 -I 是否含 ../../src/
    utils/...

>   常用命令

**生成/更新编译数据库并符号链接安装**

  colcon build --symlink-install --cmake-args
  -DCMAKE_EXPORT_COMPILE_COMMANDS=ON

**单文件快速诊断（不启动 LSP）**

  clangd --check=src/utils/atcf_ros/include/atcf_ros/
  log_macros.hpp

作用是不启动 LSP，直接输出诊断

**header-only 快速诊断（示例）**

  clangd --check=src/utils/param_utils/include/param_utils/
  param_snapshot.hpp --compile-commands-dir=/home/wrj/Desktop/rm_engineer



### 新人快速上手

    1. 先跑一次 colcon build --symlink-install
        2. 确认仓库根有 compile_commands.json
        3. 按上面的 .clangd 放入仓库根。
        4. 重启编辑器/clangd，打开任意头文件验证无缺失。

  这样即使换机器或移动仓库路径，也能保持 clangd 正常补全与诊
  断。



#### Vscode 配置

>   目前 user settings.json

~~~json
{
    // =============================
    // C/C++ / Clangd
    // =============================
    "C_Cpp.default.compilerPath": "/usr/bin/g++",
    "C_Cpp.default.compileCommands": "${workspaceFolder}/compile_commands.json",
    "C_Cpp.default.cppStandard": "c++20",
    "C_Cpp.default.cStandard": "c23",
    "C_Cpp.default.includePath": [
      "${workspaceFolder}/src",
      "${workspaceFolder}/install/atcf_core/include",
      "${workspaceFolder}/install/atcf_ros/include",
      "${workspaceFolder}/src/utils/atcf_core/include",
      "${workspaceFolder}/src/utils/atcf_ros/include",
      "/usr/include/c++/13",
      "/usr/include/x86_64-linux-gnu/c++/13",
      "/usr/include/c++/13/backward",
      "/usr/lib/gcc/x86_64-linux-gnu/13/include",
      "/usr/local/include",
      "/usr/include/x86_64-linux-gnu",
      "/usr/include"
    ],
    "C_Cpp.errorSquiggles": "disabled",        // 使用 clangd 提示，停用 MS 引擎
    "C_Cpp.intelliSenseEngine": "disabled",    // clangd 负责补全
    "C_Cpp.autocomplete": "disabled",

    "clangd.path": "/home/wrj/.config/Code/User/globalStorage/llvm-vs-code-extensions.vscode-clangd/install/21.1.0/clangd_21.1.0/bin/clangd",
    "clangd.arguments": [
      "--compile-commands-dir=${workspaceFolder}",
      "--background-index",
      "--clang-tidy",
      "--completion-style=bundled",
      "--header-insertion=iwyu",
      "--pch-storage=disk",
      "--function-arg-placeholders=false",
      "-j=2"
    ],
    "clangd.inactiveRegions.useBackgroundHighlight": true,
    "clangd.onConfigChanged.forceEnable": false,
    "clangd.restartAfterCrash": false,

    // =============================
    // Git
    // =============================
    "git.autofetch": true,
    "git.enableSmartCommit": false,
    "git.autofech": false,       // 注意：原键名疑似拼写错误，保留以防扩展读取
    "git.comfirmSync": false,    // 同上

    // =============================
    // CMake
    // =============================
    "cmake.configureOnOpen": false,
    "cmake.saveBeforeBuild": true,
    "cmake.copyCompileCommands": "${workspaceFolder}/.vscode/compile_commands.json",
    "cmake.ignoreCMakeListsMissing": true,

    // =============================
    // ROS / Python
    // =============================
    "ROS2.distro": "humble",
    "python.analysis.extraPaths": ["/usr/bin/python3"],
    "python.analysis.typeCheckingMode": "basic",

    // =============================
    // 文件与编辑器
    // =============================
    "files.autoSave": "afterDelay",
    "files.exclude": {
      "**/.git": true,
      "**/.svn": true,
      "**/.hg": true,
      "**/.DS_Store": true,
      "**/Thumbs.db": true
    },
    "files.associations": {
      "iostream": "cpp",
      "intrinsics.h": "c",
      "ostream": "cpp",
      "vector": "cpp"
    },
    "editor.insertSpaces": true,
    "editor.tabSize": 2,
    "editor.formatOnPaste": false,
    "editor.formatOnSave": false,
    "editor.formatOnType": false,
    "editor.autoIndent": "none",
    "editor.minimap.enabled": false,
    "editor.rulers": [],

    // =============================
    // 终端 / 同步
    // =============================
    "terminal.integrated.defaultProfile.linux": "bash",
    "terminal.integrated.profiles.linux": {
      "bash": { "path": "bash", "icon": "terminal-bash" },
      "zsh": { "path": "zsh" },
      "pwsh": { "path": "pwsh", "icon": "terminal-powershell" }
    },
    "security.workspace.trust.untrustedFiles": "open",
    "settingsSync.ignoredSettings": ["-clangd.path"],
    "workbench.settings.applyToAllProfiles": [



],

    // =============================
    // LaTeX
    // =============================
    "latex-workshop.latex.recipes": [
      { "name": "xelatex", "tools": ["xelatex"] },
      { "name": "xelatex->bibtex->exlatex*2", "tools": ["xelatex", "bibtex", "xelatex", "xelatex"] }
    ],
    "latex-workshop.latex.tools": [
      { "name": "xelatex", "command": "xelatex", "args": ["-synctex=1", "-interaction=nonstopmode", "-file-line-error", "%DOC%"] },
      { "name": "bibtex",  "command": "bibtex",  "args": ["%DOCFILE%"] }
    ],
    "[latex]": { "editor.defaultFormatter": "James-Yu.latex-workshop" },
    "latex-workshop.formatting.latex": "latexindent",

    // =============================
    // Markdown / 其他
    // =============================
    "[markdown]": {
      "editor.quickSuggestions": { "other": "on", "comments": "on", "strings": "on" },
      "editor.acceptSuggestionOnEnter": "on"
    },
    "xlsxViewer.md.stickyToolbar": false,
    "xlsxViewer.md.wordWrap": true,
    "xlsxViewer.md.syncScroll": true,
    "xlsxViewer.md.previewPosition": "right",
    "workbench.editorAssociations": {
      "*.copilotmd": "vscode.markdown.preview.editor"
    },
    "lldb.suppressUpdateNotifications": true,
    "github.copilot.advanced": {},
    "editor.stickyScroll.enabled": false
  }
~~~

>   目前 workspace settings.json

~~~json
{
      // =========================================================
      // C/C++ / Clangd
      // =========================================================
      "cmake.ignoreCMakeListsMissing": true,
      "C_Cpp.default.compileCommands": "/home/wrj/github/dengyu-lab/ATCF-Engineer-2025/compile_commands.json",
      "C_Cpp.default.includePath": [
          "${workspaceFolder}/src"
      ],


      // =========================================================
      // Clangd 
      // ---------------------------------------------------------
      //   - fallbackFlags 只有找不到compile_command.json才起作用
      // =========================================================
      "clangd.arguments": [
          "--compile-commands-dir=${workspaceFolder}",
          "--background-index",
          "--clang-tidy"
      ],
      "clangd.fallbackFlags": [
          "-std=c++17",
          "-I${workspaceFolder}/src/utils/atcf_core/include",
         "-I${workspaceFolder}/src/utils/atcf_ros/include"
      ],

      // =========================================================
      // ROS / Python
      // =========================================================
      "ROS2.distro": "humble",
      "urdf-visualizer.packages": {
          "panda_description": "src/robots/panda/panda_description",
          "elite_description": "src/robots/elite/elite_description",
          "engineer_description": "src/robots/engineer/engineer_description"
      },
      "python.autoComplete.extraPaths": [
          "/opt/ros/humble/lib/python3.10/site-packages",
          "/opt/ros/humble/local/lib/python3.10/dist-packages"
      ],

      // =========================================================
      // LaTeX
      // =========================================================
      "latex-workshop.latex.autoBuild.run": "never",
      "latex-workshop.latex.outDir": "%DIR%/build",
      "latex-workshop.latex.recipes": [
          { "name": "latexmk (build)", "tools": ["latexmk"] },
          { "name": "latexmk (build, shell-escape)", "tools": ["latexmk-shell-escape"] }
      ],
      "latex-workshop.latex.tools": [
          {
              "name": "latexmk",
              "command": "latexmk",
              "args": ["-xelatex", "-interaction=nonstopmode", "-file-line-error", "-outdir=%OUTDIR%", "%DOC%"]
          },
          {
              "name": "latexmk-shell-escape",
              "command": "latexmk",
              "args": ["-xelatex", "-shell-escape", "-interaction=nonstopmode", "-file-line-error", "-outdir=%OUTDIR%", "%DOC%"]
          } 
    ],
    "editor.tabSize": 2
  }
~~~
