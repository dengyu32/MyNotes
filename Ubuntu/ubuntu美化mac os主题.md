### Ubuntu 美化为 macOS 风格

>   本笔记整理了在 Ubuntu 上将界面美化为 macOS 风格的流程，包括主题、图标和扩展。

------

#### 1. 安装主题配置工具

```
sudo apt install gnome-tweaks
```

>   GNOME Tweaks 可用于修改主题、图标、字体等

------

#### 2. 安装 macOS 风格 GTK 主题

```
git clone https://github.com/vinceliuice/WhiteSur-gtk-theme
cd WhiteSur-gtk-theme
./install.sh -t all -N glassy -m -HD
```

-   `-t all` → 安装全部主题
-   `-N glassy` → 设置透明风格
-   `-m` → 安装图标
-   `-HD` → 高分辨率支持

------

#### 3. 安装 GNOME Shell 外观增强脚本

```
sudo ./tweaks.sh -g -f monterey
```

>   可进一步美化顶部栏、窗口按钮等为 macOS Monterey 风格。

------

#### 4. 安装 macOS 风格图标

1.  下载图标主题：[Mkos-Big-Sur](https://www.gnome-look.org/p/1400021)
2.  复制到系统目录：

```
sudo cp -R Mkos-Big-Sur /usr/share/icons/
```

------

#### 5. 安装 GNOME Shell 扩展管理器

```
sudo apt install gnome-shell-extension-manager
sudo apt install gnome-shell-extensions
```

-   使用扩展增强顶部栏、Dock、快捷手势等功能

------

#### 6. 刷新 GNOME Shell

```
alt + f2
r
回车
```

>   或重启系统，确保主题和扩展生效。

------

#### 7. 选择主题和图标

-   打开 **GNOME Tweaks**

~~~bash
gnome-tweaks
~~~

-   在 **Appearance** 中选择已安装的 **GTK 主题** 和 **图标主题**

------

>   注意事项：
>
>   -   美化主题可能与 Ubuntu 默认系统更新存在冲突
>   -   可在 Tweaks 中随时切换或恢复默认主题
>   -   高分辨率屏幕建议使用 HD 主题版本