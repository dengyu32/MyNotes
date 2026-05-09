## Typora 配置笔记

### 目录

[TOC]

### 概要

- 覆盖安装/激活（版本 <= 1.8.10）
- 启动 GPU 报错的处理
- 图床配置（PicGo + GitHub）



### GPU 报错与补丁

>   自查

-   报错示例

~~~bash
Hook Init
Require events
Require electron-fetch
Hooking electron-fetch
Require package.json console.log override blocked
MESA-LOADER: failed to retrieve device information
MESA-LOADER: failed to open nvidia-drm: /usr/lib/dri/nvidia-drm_dri.so: 无法打开共享目标文件: 权限不够
(search paths /usr/lib/x86_64-linux-gnu/dri:$${ORIGIN}/dri:/usr/lib/dri, suffix _dri)
MESA-LOADER: failed to open zink: /usr/lib/dri/zink_dri.so: 无法打开共享目标文件: 权限不够
(search paths /usr/lib/x86_64-linux-gnu/dri:$${ORIGIN}/dri:/usr/lib/dri, suffix _dri)
MESA-LOADER: failed to open kms_swrast: /usr/lib/dri/kms_swrast_dri.so: 无法打开共享目标文件: 权限不够
(search paths /usr/lib/x86_64-linux-gnu/dri:$${ORIGIN}/dri:/usr/lib/dri, suffix _dri)
MESA-LOADER: failed to open swrast: /usr/lib/dri/swrast_dri.so: 无法打开共享目标文件: 权限不够
(search paths /usr/lib/x86_64-linux-gnu/dri:$${ORIGIN}/dri:/usr/lib/dri, suffix _dri)
~~~

>   临时规避

-   运行 `typora --disable-gpu xxx.md`

>   永久补丁（包装启动）

~~~bash
sudo mv /usr/bin/typora /usr/bin/typora-real
printf '#!/bin/bash\n/usr/bin/typora-real --disable-gpu "$@"\n' | sudo tee /usr/bin/typora > /dev/null
sudo chmod +x /usr/bin/typora
~~~



### 激活流程（版本 <= 1.8.10）

>   注意

-   仅适用于 1.8.10 及以下版本
-   如果注册失败，删除 `/usr/share/typora/node` 后重试

>   流程

-   安装 Typora（可从官网获取 1.8.10 版本）
-   克隆 Yporaject

~~~bash
git clone https://github.com/hazukieq/Yporaject.git
mv Yporaject/ Documents/
cd Documents/
~~~

-   配置 Rust 环境

~~~bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
sudo apt install cargo
~~~

-   使用 Yporaject

~~~bash
cd Yporaject/
cargo build
ls target/debug
cargo run
sudo cp target/debug/node_inject /usr/share/typora
~~~

-   新开终端

~~~bash
cd /usr/share/typora
sudo chmod 777 node_inject
sudo ./node_inject
~~~

-   返回之前终端

~~~bash
cd license-gen/
cargo build
cargo run
~~~

-   得到激活码后激活（若失败可重启后再试）



### 图床配置（PicGo + GitHub）

>   使用 GitHub 图床

1.  建立仓库 `note_images`，设为 public
2.  进入 GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token
    -   Repository access: Only select repositories（选 `note_images`）
    -   Expiration: 可设为不过期
    -   Permissions: Contents（read/write）
3.  立刻复制个人访问令牌

~~~tokens
# 请不要把真实 token 写进笔记
github_pat_xxx
~~~

4.  下载 PicGo App（选择 `PicGo-2.4.0.AppImage`）

~~~http
https://mirrors.sdu.edu.cn/github-release/1765364126/github-release/Molunerfinn_PicGo/v2.4.0/
https://github.com/Molunerfinn/PicGo/releases
~~~

5.  赋予执行权限并启动

~~~bash
chmod +x PicGo-2.4.0.AppImage
./PicGo-2.4.0.AppImage
~~~

6.  在 PicGo 配置图床（GitHub 图床），并在 Typora 偏好设置里填入 PicGo AppImage 路径

![](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/20251211153142114.png)

![](https://raw.githubusercontent.com/dengyu32/note_images/main/images/20251210212524943.png)



### 实际测试

1.  使用 `Ctrl + Shift + A` 截图
2.  发到 QQ 中
3.  拖动图片到 PicGo
4.  复制上传地址，在 Typora 插入图片即可自动替换链接

或

1.  本地图片从 QQ 拖到本地
2.  拖进 Typora
3.  右键上传图片，自动上传并替换路径



### GitHub 2FA

1.  手机安装 Authenticator，登录 Microsoft 账号
2.  扫描二维码，输入手机验证码
3.  保存 GitHub 恢复代码
4.  配置 GitHub 通行密钥



### 卸载

~~~bash
sudo apt remove typora
sudo apt purge typora
sudo apt autoremove
~~~



### 参考

-   https://blog.csdn.net/weixin_65657501/article/details/142788747
-   https://blog.csdn.net/qq_44231797/article/details/131658184
-   https://juejin.cn/post/6844903993529860109
-   https://linux.do/t/topic/642020
