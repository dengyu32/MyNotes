### 向 SD 卡烧录 Ubuntu 22.04

>   本笔记整理了在 Linux 系统下将 Ubuntu 22.04 镜像烧录到 SD 卡的流程。

------

#### 1. 格式化 SD 卡

1.  打开 **磁盘** 工具
2.  选择要烧录的 SD 卡
3.  点击 **格式化**
    -   通常 SD 卡显示为 **Mass Storage Device**（大容量存储设备）

------

#### 2. 使用 AppImage 烧录工具

##### 安装 balenaEtcher

```
wget https://github.com/balena-io/etcher/releases/download/v1.18.11/balenaEtcher-1.18.11-x64.AppImage
chmod +x balenaEtcher-1.18.11-x64.AppImage
./balenaEtcher-1.18.11-x64.AppImage
```

>   直接运行即可启动图形界面烧录 Ubuntu 镜像到 SD 卡

------

##### 解压 AppImage

```
./balenaEtcher-1.18.11-x64.AppImage --appimage-extract
```

-   会生成一个 `squashfs-root` 文件夹
-   进入目录并运行：

```
cd squashfs-root
./AppRun
```

>   使用 AppRun 运行程序，即可进行烧录

------

>   注意事项：
>
>   -   烧录前请确保 SD 卡数据已备份
>   -   烧录过程中不要拔出 SD 卡
>   -   烧录完成后可直接用于启动或安装 Ubuntu 22.04