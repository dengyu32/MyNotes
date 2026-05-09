#### 使用 Docker

>   常用 Docker 操作整理，按场景分类，便于实际操作

------

**安装与配置（Ubuntu 22.04 + USTC 源）**

更新系统并安装依赖

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release
```

添加 Docker 官方 GPG 密钥（USTC 镜像）

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

添加 Docker 软件源（USTC 镜像）

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

更新并安装 Docker

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

启动 Docker 服务

```bash
sudo systemctl start docker
```

设置开机自启（可选）

```bash
sudo systemctl enable docker
```

验证安装

```bash
docker --version
sudo docker run hello-world
systemctl status docker
```

可选：免 `sudo` 运行 Docker

```bash
sudo usermod -aG docker $USER
```

------

**常用 Docker 命令**

```bash
docker pull ubuntu                   # 下载指定镜像
docker images                        # 查看本地已有镜像
docker rmi ubuntu                    # 删除本地镜像

docker run -it ubuntu bash           # 交互式运行容器并进入 bash
docker ps                            # 查看当前正在运行的容器
docker ps -a                         # 查看所有容器（包含已退出）
docker stop <容器ID>                 # 停止指定容器
docker rm <容器ID>                   # 删除指定容器

docker logs <容器ID>                 # 查看容器日志
docker logs -f <容器ID>              # 实时跟踪容器日志
docker exec -it <容器ID> bash        # 进入正在运行的容器

sudo systemctl start docker          # 启动 Docker 服务
sudo systemctl stop docker           # 停止 Docker 服务
sudo systemctl status docker         # 查看 Docker 服务状态
```

------

**使用 Docker Hub**

1.  注册 Docker Hub 账号
2.  创建 Hub 仓库
3.  在终端登录 Docker Hub

```bash
docker login
```

4.  推送镜像

```bash
docker push dengyu32/meshes-tool:v1.0
```

5.  直接运行远程镜像（本地无需预先存在镜像）

```bash
docker run --rm -v $(pwd):/data dengyu32/meshes-tool:v1.0
```

------

**报错 / 注意事项**

1.  **拉取镜像超时，报错 `context deadline exceeded`**

>   原因：Docker 默认不继承系统代理，可能无法直接访问 Docker Hub

解决方法（配置 Docker 代理）：

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo nano /etc/systemd/system/docker.service.d/http-proxy.conf
```

写入：

```ini
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:7890"
Environment="HTTPS_PROXY=http://127.0.0.1:7890"
Environment="NO_PROXY=localhost,127.0.0.1,docker-registry.somecorporation.com"
```

重载并重启 Docker：

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl restart docker
```

2.  **Docker Hub 仓库限制**

>   私有仓库数量受账号类型限制，公开仓库通常可不限量创建

------
