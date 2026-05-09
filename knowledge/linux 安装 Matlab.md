

### Linux 安装 MATLAB（Ubuntu 22.04）

>   整理在 Ubuntu 上安装 MATLAB 的完整流程，包括挂载镜像、安装、创建软链接、常见依赖及磁盘检查。



[TOC]

#### 参考资料

-   [Ubuntu22.04 安装 MATLAB R2024a](https://blog.csdn.net/Explorer_XZH/article/details/140584837?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-140584837-blog-145740693.235^v43^pc_blog_bottom_relevance_base2&spm=1001.2101.3001.4242.1&utm_relevant_index=3)
-   [Ubuntu MATLAB R2024a 安装 + 破解](https://blog.csdn.net/m0_60931383/article/details/145740693?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2~default~BlogCommendFromBaidu~PaidSort-1-145740693-blog-134266206.235^v43^pc_blog_bottom_relevance_base2&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2~default~BlogCommendFromBaidu~PaidSort-1-145740693-blog-134266206.235^v43^pc_blog_bottom_relevance_base2&utm_relevant_index=1)

------

#### 挂载 MATLAB 镜像

```
sudo mount -o loop R2024a_Linux.iso ~/Documents/Matlab/matlab
```

进入目录：

```
cd ~/Documents/Matlab/
```

------

#### 启动安装程序

```
./install
```

安装界面选择：

```
I have license...
```

按步骤继续即可。(从csdn上复制会有后缀，注意删除)

------

#### 创建全局 matlab 启动链接

```
sudo ln -s /home/wrj/Documents/Matlab/R2025a/bin/matlab /usr/local/bin/matlab
```

运行：

```
matlab
```

------

#### 查看磁盘空间

```
df -h
```

------

出现网络许可证不能单机使用 --> 运行下面两条指令即可

#### 安装 MATLAB 支持软件包（可选）

```
sudo apt install matlab-support
```

------

#### 修复 lmgrimpl 库缺失问题（常见启动错误）

```
sudo cp ~/Downloads/Matlab/matlab_install_linux/libmwlmgrimpl.so \
~/Documents/Matlab/R2025a/bin/glnxa64/matlab_startup_plugins/lmgrimpl/
```

------

#### 修复 GTK 模块依赖（界面渲染异常时）

```
sudo apt-get install libcanberra-gtk-module
```
