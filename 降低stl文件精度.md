### 降低stl文件精度

#### 目录

[TOC]

#### 前言

从sw中导出的stl精度太高，move_group在规划时的碰撞检测对显存是一个很大的挑战

同时Interactive Marker需要实时更新位置，过高的精度会导致rviz渲染队列爆炸

无法加载出交互式标记（小球），同时引起Rviz无响应

#### 减面

>   由于依赖问题 A module that was compiled using NumPy 1.x cannot be run in NumPy 2.2.6
>
>   open3d需要依赖旧版的NumPy 1.x，而当前环境使用的新版NumPy 2.x，直接降级会有很大的风险
>
>   所以要使用Docker容器

1.   首先克隆项目https://github.com/at-engineer-lab/meshes_tool

2.   创建Docker

     1.   编写 simplify_stl.py

     2.   编写Dockerfile,并使用下面指令构建镜像

          ~~~bash
          docker build -t meshes-tool:v1.0 . # -t 后面是镜像的名字:版本号
          # 一些常见指令
          docker images #  查看本地所有镜像
          docker tag # 打标签
          docker inspect meshes-tool:v1.0 # 查看镜像的详细信息
          docker run --rm meshes-tool:v1.0 python3 --version#docker自检
          docker system df # 查看 docker 使用的磁盘空间
          docker rmi xxx # 删除镜像
          ~~~
     
     3.   构建成功有效信息
     
          ~~~txt
          ## Docker Build Information
          
          - Base Image: python:3.9-slim-bookworm
          - OS: Debian 12 (bookworm)
          - Python Version: 3.9
          
          ### System Dependencies
          - libgl1 (OpenGL runtime, required by Open3D)
          - libgomp1 (GNU OpenMP runtime)
          
          ### Python Dependencies
          - open3d
          - numpy
          
          ### Docker Image
          - Name: meshes-tool
          - Tag: v1.0
          
          ### Build Command
          ```bash
          docker build -t meshes-tool:v1.0 .
          ~~~
     
     4.   尝试运行
     
          ~~~bash
          docker run --rm -v $(pwd)/descend_stl_precision/stl:/data dengyu32/meshes-tool:v1.0 # 将电脑的绝对路径映射到容器里的/data  -v 表示插入
          ~~~
     
     5.   出现日志
     
          ~~~bash
          wrj@wrj-Legion-Y7000-IRX9:<main>meshes_tool$ docker run --rm -v $(pwd)/descend_stl_precision/stl:/data meshes-tool:v1.0 
          找到 9 个文件，开始处理...
          正在处理: link4.STL ...
            -> 完成! 面数从 4848 降至 2000. 保存为: link4_collision.stl
          正在处理: left_finger_link.STL ...
            -> 完成! 面数从 8720 降至 2000. 保存为: left_finger_link_collision.stl
          正在处理: right_finger_link.STL ...
            -> 完成! 面数从 8720 降至 2000. 保存为: right_finger_link_collision.stl
          正在处理: link6.STL ...
            -> 完成! 面数从 115984 降至 2000. 保存为: link6_collision.stl
          正在处理: link1.STL ...
            -> 完成! 面数从 35800 降至 2000. 保存为: link1_collision.stl
          正在处理: link3.STL ...
            -> 完成! 面数从 145782 降至 2000. 保存为: link3_collision.stl
          正在处理: link5.STL ...
            -> 完成! 面数从 175914 降至 2000. 保存为: link5_collision.stl
          正在处理: base_link.STL ...
            -> 完成! 面数从 596658 降至 1999. 保存为: base_link_collision.stl
          正在处理: link2.STL ...
            -> 完成! 面数从 64308 降至 1999. 保存为: link2_collision.stl
          
          全部处理完毕
          ~~~
     
     6.   已经push到docker hub，方便他人使用，只需运行最后一条指令即可
     
          登录Docker Hub账号
          docker tag mesh-processor:v1.0 dengyu32/meshes-tool:v1.0
          docker push wrjbot/mesh-processor:v1.0
          docker run --rm -v $(pwd):/data dengyu32/meshes-tool:v1.0

#### 凸包

#### 附录

##### **报错日志**

~~~bash
[rviz2-4] [INFO] [1766192446.913693537] [move_group_interface]: Ready to take commands for planning group engineer_arm.
[rviz2-4] [WARN] [1766192446.923618129] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 101
[rviz2-4] [WARN] [1766192446.923824205] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 102
[rviz2-4] [WARN] [1766192446.923967493] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 103
[rviz2-4] [WARN] [1766192446.924082890] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 104
[rviz2-4] [WARN] [1766192446.924194884] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 105
[rviz2-4] [WARN] [1766192446.924305508] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 106
[rviz2-4] [WARN] [1766192446.924416162] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 107
[rviz2-4] [WARN] [1766192446.924526054] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 108
[rviz2-4] [WARN] [1766192446.924633943] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 109
[rviz2-4] [WARN] [1766192446.924742415] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 110
[rviz2-4] [WARN] [1766192446.924865943] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 111
[rviz2-4] [WARN] [1766192446.924979437] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 112
[rviz2-4] [WARN] [1766192446.925089131] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 113
[rviz2-4] [WARN] [1766192446.925277813] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 114
[rviz2-4] [WARN] [1766192446.925392579] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 115
[rviz2-4] [WARN] [1766192446.925502983] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 116
[rviz2-4] [WARN] [1766192446.925610772] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 117
[rviz2-4] [WARN] [1766192446.925718771] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 118
[rviz2-4] [WARN] [1766192446.925841777] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 119
[rviz2-4] [WARN] [1766192446.925955830] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 120
[rviz2-4] [WARN] [1766192446.926065006] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 121
[rviz2-4] [WARN] [1766192446.926174794] [interactive_marker_display_103814305599408]: Update queue too large. Erasing message with sequence number 122
[ERROR] [rviz2-4]: process has died [pid 194801, exit code -11, cmd '/opt/ros/humble/lib/rviz2/rviz2 -d /home/wrj/github/dengyu-lab/ATCF-Engineer-2025/install/engineer_moveit_config/share/engineer_moveit_config/config/moveit.rviz --ros-args --params-file /tmp/launch_params_g4aangl_ --params-file /tmp/launch_params_waam3spo --params-file /tmp/launch_params_e66w2gcq'].

~~~

##### **simplify_stl.py**

```
import open3d as o3d
import os
import glob

# ================= 配置区域 =================
# 输入你的 meshes 文件夹路径 
INPUT_DIR = "."

# 目标面数 (Target number of triangles)
# 对于碰撞模型，通常 1000 - 3000 面就足够足够了
# 相比原本几万个面，这能提升几十倍性能
TARGET_TRIANGLES = 2000 
# ===========================================

def simplify_mesh(file_path):
    print(f"正在处理: {os.path.basename(file_path)} ...")
    
    # 1. 读取网格
    mesh = o3d.io.read_triangle_mesh(file_path)
    if not mesh.has_triangles():
        print(f"  -> 跳过: 不是有效的网格文件")
        return

    original_count = len(mesh.triangles)
    
    # 如果面数本来就很少，就不处理了
    if original_count <= TARGET_TRIANGLES:
        print(f"  -> 跳过: 面数已很少 ({original_count})")
        return

    # 2. 减面 (使用二次误差度量 Decimation)
    # 这是一个非常经典的保留形状的算法
    simplified_mesh = mesh.simplify_quadric_decimation(target_number_of_triangles=TARGET_TRIANGLES)
    
    # 3. 重新计算法线 (保证光照/碰撞计算正确)
    simplified_mesh.compute_vertex_normals()
    
    new_count = len(simplified_mesh.triangles)
    
    # 4. 保存为新文件
    # 命名规则：原来的名字 + _collision.stl
    # 例如: link1.STL -> link1_collision.stl
    file_dir = os.path.dirname(file_path)
    file_name = os.path.basename(file_path)
    name_without_ext = os.path.splitext(file_name)[0]
    
    new_filename = f"{name_without_ext}_collision.stl"
    new_path = os.path.join(file_dir, new_filename)
    
    o3d.io.write_triangle_mesh(new_path, simplified_mesh)
    print(f"  -> 完成! 面数从 {original_count} 降至 {new_count}. 保存为: {new_filename}")

def main():
    # 递归查找目录下所有的 .stl 或 .STL 文件
    # recursive=True 表示会搜索子文件夹
    stl_files = glob.glob(os.path.join(INPUT_DIR, "**/*.stl"), recursive=True)
    stl_files += glob.glob(os.path.join(INPUT_DIR, "**/*.STL"), recursive=True)
    
    if not stl_files:
        print("错误: 没找到任何 STL 文件，请检查路径是否正确！")
        return

    print(f"找到 {len(stl_files)} 个文件，开始处理...")
    
    for f in stl_files:
        # 跳过已经是 collision 的文件，防止重复处理
        if "_collision" in f:
            continue
        simplify_mesh(f)
        
    print("\n全部处理完毕！")

if __name__ == "__main__":
    main()
```

##### **Dockerfile**

~~~dockerfile
# 1. 基础镜像
FROM python:3.9-slim-bookworm

# 2. 安装 Open3D 依赖的系统库 (必不可少)
RUN apt-get update && apt-get install -y \
    libgl1 \
    libgomp1 \
    && rm -rf /var/lib/apt/lists/*

# 3. 安装 Python 库
RUN pip install --no-cache-dir open3d numpy

# 4. 创建内部工作目录
WORKDIR /app

# 5. [关键] 复制脚本
# 因为 Dockerfile 在根目录，脚本在子目录，所以路径要写全
COPY descend_stl_precision/simplify_stl.py /app/simplify_stl.py

# 6. 设置启动命令
CMD ["python3", "/app/simplify_stl.py"]
~~~

##### bookworm / trixie

>   https://zh.wikipedia.org/wiki/Debian

Debian Linux的发行版代号，bookworm是 Debian 12 , trixie是 Debian 13

Debian是一个类Unix的操作系统，和Ubuntu类似

