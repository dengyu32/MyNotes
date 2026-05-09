### 使用 submodule

>   用于在项目中嵌入其他 Git 仓库，并支持递归结构，使“库里有库”的层级依赖可以统一管理，常用于第三方库或内部模块管理。

[TOC]

#### 基本概念

>   子模块是一种把其他 Git 仓库以目录形式嵌入当前仓库的机制，本质上记录的是目标仓库的特定 commit，而不是文件内容。

**核心理解**

-   子模块目录是一个独立 Git 仓库
-   主仓库只记录子模块指向的 commit
-   更新主仓库不会自动更新子模块
-   可形成多层嵌套

#### 示例结构

>   子模块在主仓库中的目录结构示例，说明项目可能存在多层嵌套依赖。

```
MainRepo/
└── ThirdPartyLib/          ← 子模块
```

子模块内继续包含子模块时：

```
ThirdPartyLib/
└── MoreLib/                ← 子模块的子模块
```

#### recurse-submodules 的作用

>   用于递归处理所有层级子模块，使 clone、pull、update 时自动同步多层子模块。

默认 clone：

```
git clone <repo>
```

表现为：

-   子模块目录为空
-   仅有 `.gitmodules` 文件

递归 clone：

```
git clone --recurse-submodules <repo>
```

Git 会自动拉取所有层级子模块。

------

#### 子模块的加载流程

##### 第一步：拉取主仓库

>   与普通 clone 完全相同，只下载主仓库内容。

##### 第二步：扫描 `.gitmodules`

>   Git 读取 `.gitmodules` 文件中的 path、url、branch 等信息，并自动 clone 子模块。

示例：

```
[submodule "rmcs_ws/src/serial"]
    path = rmcs_ws/src/serial
    url = https://github.com/Alliance-Algorithm/ros2-serial.git
    branch = ros2

[submodule "rmcs_ws/src/hikcamera"]
    path = rmcs_ws/src/hikcamera
    url = https://github.com/Alliance-Algorithm/ros2-hikcamera.git

[submodule "rmcs_ws/src/fast_tf"]
    path = rmcs_ws/src/fast_tf
    url = https://github.com/qzhhhi/FastTF.git

[submodule "rmcs_ws/src/rmcs_core/include/librmcs"]
    path = rmcs_ws/src/rmcs_core/include/librmcs
    url = git@github.com:Alliance-Algorithm/librmcs.git

[submodule "rmcs_ws/src/rmcs_core/include/rmcs_core/librmcs"]
    path = rmcs_ws/src/rmcs_core/include/rmcs_core/librmcs
    url = git@github.com:Alliance-Algorithm/librmcs.git

[submodule "rmcs_ws/src/rmcs_core/librmcs"]
    path = rmcs_ws/src/rmcs_core/librmcs
    url = git@github.com:Alliance-Algorithm/librmcs.git
```

**字段说明**

-   **path**：子模块目录
-   **url**：子模块仓库地址
-   **branch**：可选指定分支

##### 第三步：递归检查子模块

>   如果子模块目录也含 `.gitmodules`，Git 会继续 clone 更深层级。

总结：

**每层子模块都是独立仓库，都会被递归解析并 clone。**

------

#### 常用命令总结

##### clone 时递归拉取

>   用于完整下载所有层级子模块。

```
git clone --recurse-submodules <repo>
```

##### 已经 clone，补拉所有子模块

>   当首次 clone 没带递归时使用。

```
git submodule update --init --recursive
```

##### 拉取主仓库最新内容并同步子模块

>   用于保持子模块与主仓库记录的版本一致。

```
git pull --recurse-submodules
git submodule update --recursive --remote
```

------

#### 添加 submodule

>   在项目中新增子模块时使用，会自动生成目录和 `.gitmodules`。

在主仓库根目录执行：

```
git submodule add <子模块仓库地址> <放置路径>
```

示例：

```
git submodule add https://github.com/xxx/serial.git third_party/serial
```

执行后会生成：

**1. 目录：`third_party/serial/`**

子模块内容会 clone 到此目录。

**2. 文件：`.gitmodules`**

Git 自动生成：

```
[submodule "third_party/serial"]
    path = third_party/serial
    url = https://github.com/xxx/serial.git
```

------

#### 删除 submodule

>   删除子模块必须按顺序执行，否则会留下残留配置或导致 Git 报错。

```
git submodule deinit -f third_party/serial
rm -rf .git/modules/third_party/serial
git rm -f third_party/serial
```

然后提交即可。