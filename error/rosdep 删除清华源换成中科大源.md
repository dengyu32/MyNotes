### rosdep 删除清华源并切换到中科大源

>   当使用 `sudo rosdep init` 出现无法从清华镜像下载默认源列表的情况时，可直接换用中科大镜像，以解决 403 Forbidden 或链接失效问题。

### 问题

执行：

```
sudo rosdep init
```

如果报错：

```
ERROR: cannot download default sources list from:
https://mirrors.tuna.tsinghua.edu.cn/github-raw/ros/rosdistro/master/rosdep/sources.list.d/20-default.list

Website may be down. <urlopen error HTTP Error 403: Forbidden>
```

说明清华源地址不可用或被限制。

### 处理步骤

```
# 删除旧配置
sudo rm -rf /etc/ros/rosdep/sources.list.d/*
sudo rm -rf ~/.ros/rosdep/sources.cache

# 手动创建新文件夹（防止不存在）
sudo mkdir -p /etc/ros/rosdep/sources.list.d

# 写入中科大镜像源配置（替代清华）
sudo bash -c 'cat > /etc/ros/rosdep/sources.list.d/20-default.list <<EOF
# os-specific listings first
yaml https://mirrors.ustc.edu.cn/rosdistro/rosdep/osx-homebrew.yaml osx
yaml https://mirrors.ustc.edu.cn/rosdistro/rosdep/base.yaml
yaml https://mirrors.ustc.edu.cn/rosdistro/rosdep/python.yaml
yaml https://mirrors.ustc.edu.cn/rosdistro/rosdep/ruby.yaml
gbpdistro https://mirrors.ustc.edu.cn/rosdistro/releases/fuerte.yaml fuerte

# newer releases
yaml https://mirrors.ustc.edu.cn/rosdistro/rosdep/base.yaml
yaml https://mirrors.ustc.edu.cn/rosdistro/rosdep/python.yaml
yaml https://mirrors.ustc.edu.cn/rosdistro/rosdep/ruby.yaml
yaml https://mirrors.ustc.edu.cn/rosdistro/rosdep/osx-homebrew.yaml osx
EOF'
```

更新：

```
rosdep update
```