### 运行 VSCode + Ozone + CubeMX

>   本笔记整理了在 Ubuntu 和 Windows 平台下，使用 CubeMX 生成 STM32 工程模板，通过 Makefile 编译并用 Ozone 调试的完整流程。

#### Ubuntu

1.  **下载 VSCode 并安装插件**

    -   打开 VSCode，安装以下必要插件（根据需要）：

        -   **C/C++ 开发相关**

            -   Better C++ Syntax（增强 C++ 语法高亮）
            -   C/C++（IntelliSense、调试、代码浏览）
            -   C/C++ Snippets（C/C++ 代码片段）

            **调试相关**

            -   Cortex-Debug（ARM Cortex-M GDB 调试支持）
            -   Cortex-Debug: Device Support Pack - STM32F4（SVD 寄存器支持）

            **辅助工具**

            -   Hex Editor（查看/编辑 HEX 文件）
            -   IntelliCode（AI 辅助开发）
            -   Makefile Tools（Makefile 构建支持）

2.  **安装 CubeMX 并配置支持包**

    -   下载 CubeMX 压缩包并解压：

        ```
        chmod -R +x ~/Documents/STM32CubeMX
        cd installation
        ./SetupSTM32CubeMX-6.15.0
        ```

    -   解压 F4 支持包（对应 MCU 系列）

    -   从 GitHub 安装 SVD 文件（用于调试工具识别寄存器）

3.  **下载并安装 J-Link 驱动**

    -   支持调试器与 Ozone 通信

4.  **安装 Ozone**

    -   Ozone 用于加载 ELF 文件并进行硬件调试
    -   需要 SVD 文件以显示寄存器结构

5.  **安装 ARM 交叉编译工具链**

    -   安装 `arm-none-eabi-gcc`

6.  **流程概览**

    -   CubeMX → 生成工程模板

    -   VSCode → Makefile 编译：

        ```
        make -j24
        ```

    -   在 `build` 文件夹获取 ELF 文件

    -   Ozone 导入 ELF 文件调试

------

#### Windows

1.  **安装工具**

    -   VSCode、CubeMX、Ozone

2.  **安装 MinGW 并配置环境变量**

3.  **VSCode 安装插件并配置 Makefile**

    -   Makefile 插件需要指向 MinGW
    -   CubeMX 生成makefile项目

4.  **安装 MSYS2 类 Linux 终端**

    -   安装 ARM 工具链：

        ```
        pacman -S mingw-w64-x86_64-arm-none-eabi-toolchain
        ```

    -   编译工程：

        ```
        cd /e/Learning/SuperBoardProject/RM2024-SuperCapacitorController-master
        make -j$(nproc) HARDWARE_ID=101
        ```

5.  **运行 `mingw-make -j24` 进行编译**

6.  **用 Ozone 导入 ELF 文件调试**

------

#### 参考

1.  [NeoZng CSDN - VSCode + CubeMX + Ozone 教程 1](https://blog.csdn.net/NeoZng/article/details/127980878?spm=1001.2014.3001.5502)
2.  [NeoZng CSDN - CubeMX 模板生成与配置 2](https://blog.csdn.net/NeoZng/article/details/127980949?spm=1001.2014.3001.5502)