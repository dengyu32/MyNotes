### 修改特定 USB 设备永久权限

>   本笔记整理了在 Linux 下为特定 USB 设备设置永久权限的方法，以 DAPLink 虚拟串口为例。

------

#### 1. 查看 USB 设备信息

```
lsusb
```

示例输出：

```
Bus 001 Device 017: ID 0483:5740 STMicroelectronics Virtual COM Port
```

-   `0483` → **idVendor**（厂商 ID）
-   `5740` → **idProduct**（产品 ID）

------

#### 2. 搜索设备节点

```
sudo dmesg | grep ttys*
ls -l /dev/ttyACM0
```

>   如果未烧录代码，可能 `/dev/ttyACM0` 不存在。

**注意**：有时 USB 设备无法显示，可能被 **brltty**（盲文终端驱动）占用：

```
sudo dmesg | tail -n 20
sudo apt remove brltty -y
```

然后重新插拔 USB 设备即可。

------

#### 3. 临时修改权限

```
sudo chmod 666 /dev/ttyACM0
```

>   修改权限只在当前会话有效，设备重插或重启后会恢复默认。

------

#### 4. 永久修改权限

1.  编辑 udev 规则文件：

```
sudo nano /etc/udev/rules.d/99-ttyacm.rules
```

2.   写入以下内容：

```
# 针对 STMicroelectronics Virtual COM Port (VID=0483, PID=5740)
# 每次插入时自动设置 /dev/ttyACM* 为 0666 权限
SUBSYSTEM=="tty", ATTRS{idVendor}=="0483", ATTRS{idProduct}=="5740", MODE="0666"
```

3.   激活规则：

```
sudo udevadm control --reload-rules
sudo udevadm trigger
```

4.   重新插拔设备并检查权限：

```
ls -l /dev/ttyACM*
```

------

#### 5. 删除规则文件

```
sudo rm /etc/udev/rules.d/99-ttyacm.rules
sudo udevadm control --reload-rules
sudo udevadm trigger
```

>   删除后重新插拔设备即可恢复默认权限。