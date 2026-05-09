### Ubuntu 无 WiFi 选项(写了一半)

>   本笔记整理了在 Ubuntu 系统中 WiFi 选项消失或无法使用的排查与解决方法。

------

### 问题分析

1.  **关闭 BIOS 安全启动**
    -   按 `F2` 进入 BIOS
    -   关闭 **Secure Boot**
2.  **缺少无线网卡驱动**
    -   某些 WiFi 芯片（如 Realtek RTL88XX 系列）过新，Ubuntu 22.04 内核未自带对应驱动
3.  **Ubuntu 内核版本过低**
    -   内核版本需 >= 5.19 才能支持部分新型网卡

------

### Ubuntu 网卡驱动原理

Ubuntu 驱动支持依赖：

-   **内核（Kernel）**
-   **厂商驱动模块（Driver Module）**
-   **固件（Firmware）**

>   大多数情况下 WiFi 问题不是固件问题，需重点检查内核和驱动模块。

------

### 确定 WiFi 芯片型号

```
lspci | grep -i wireless
lspci | grep -i network
```

------

### 确定内核版本

```
uname -r
```

>   需要 >= 5.19

检查无线网卡是否被识别：

```
lspci | grep -i network
```

检查驱动模块是否加载：

```
lsmod | grep -E "rtw88|rtw89|iwlwifi"
```

------

### 确定有线网卡型号（备用网络）

```
lspci | grep -i Ethernet
```

示例输出：

```
08:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller (rev 15)
```

>   表示系统识别有线网卡 RTL8111/8168/8411，可临时通过有线网络安装驱动。

------

### 安装无线网卡驱动

-   可前往官方或厂商驱动下载：
     [Lenovo 驱动下载](https://newsupport.lenovo.com.cn/driveDownloads_index.html)
-   下载对应型号驱动并按照官方指南编译安装

------

>   提示：
>
>   -   若 WiFi 芯片过新，可考虑升级内核或使用官方 DKMS 驱动
>   -   安装驱动后需重启或重新插拔 WiFi 模块
>   -   遇到 RTL88XX 系列可搜索 DKMS 驱动安装教程