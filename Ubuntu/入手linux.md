### 入手 Linux

>   本笔记整理了从零开始安装 Ubuntu 系统、配置开发环境及 ROS2/MoveIt2 的完整流程。

------

#### 流程

1.  **卸载旧系统**

    -   卸载 Ubuntu 并删除启动项
    -   参考：[CSDN 教程](https://blog.csdn.net/mtllyb/article/details/78635757?sharetype=blog&shareId=78635757&sharerefer=APP&sharesource=m0_71713242&sharefrom=qq)

2.  **插入系统盘**

3.  **进入 BIOS 设置启动顺序**

    -   按 `fn + F2` 进入 BIOS
    -   用 `F5/F6` 调整启动盘顺序，U 盘优先
    -   关闭 Secure Boot

4.  **安装 Ubuntu**

    -   配置语言：先英文再中文
    -   分区（参考分配）：
        -   swap 16GB
        -   /boot 2GB
        -   /home 200GB
        -   / 234GB
    -   参考：[CSDN 分区教程](https://blog.csdn.net/u012052268/article/details/77145427/)

5.  **下载常用应用**

    -   **输入法**：搜狗输入法（先连热点）
         [下载指南](https://shurufa.sogou.com/linux/guide)

    -   **QQ / WeChat**

    -   **Clash**：需配置网络代理，导入订阅 URL，Global 可选，Rule 必须选择“境内使用”，开启开机自启

    -   **Google Chrome**：修复切换输入法问题

        ```
        sudo apt install fcitx5-frontend-gtk4
        ```

    -   **Typora / VSCode / Qt**

        -   VSCode 配置：安装插件 C/C++、CMake、Python、ChineseF，重启切换中文
        -   配置 `.vscode`：`c_cpp_properties.json`、`launch.json`、`tasks.json`
        -   开启自动保存

6.  **配置开发环境**

    -   **ROS2 + MoveIt2**

        -   更换系统源和 ROS 源（除清华源）

        -   安装二进制环境快速上手：

            ```
            sudo apt install ros-humble-moveit*
            ```

        -   若需要完整环境，建议源码构建（mtc 仅在源码构建中可用）

        -   安装 Qt5SerialPort（必要）

7.  **注册 GitHub 并配置 SSH**

    ```
    ssh-keygen -t rsa -C "邮箱"
    # 连续三次回车
    ```

    -   在 GitHub 添加公钥
    -   配置用户信息：

    ```
    git config --global user.name "gitname"
    git config --global user.email "git邮箱"
    ```

8.  **配置 NVIDIA 驱动**

    ```
    nvidia-smi
    glxinfo | grep "OpenGL"
    sudo apt install -y nvidia-driver-580
    sudo reboot
    ```

------

#### 安装系统盘

1.  **准备 Ventoy**

    -   下载：[Ventoy Releases](https://github.com/ventoy/Ventoy/releases)

    -   挂载 U 盘：

        ```
        lsblk
        sudo umount /dev/sda1
        cd ~/Documents/ventoy-1.107-linux/ventoy-1.1.07/
        sudo ./Ventoy2Disk.sh -i /dev/sda
        ```

2.  **下载 Ubuntu 镜像**

    -   官方下载：[Ubuntu Desktop](https://ubuntu.com/download/desktop)

3.  **MoveIt2 中间件配置（CycloneDDS）**

    ```
    sudo apt install ros-$ROS_DISTRO-rmw-cyclonedds-cpp
    export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
    ```

------

>   提示：
>
>   -   安装应用和配置环境建议按顺序操作，避免网络或权限问题
>   -   Moveit2 源码构建可获取完整功能，但耗时较长
>   -   NVIDIA 驱动安装完成后，可通过 `nvidia-smi` 确认驱动生效
