###  VC++ 6.0 Can't Find or Open PDB file

>   二级 C 语言考试中遇到 VC++ 6.0 无法加载 PDB 文件的问题，
>
>   记录解决方法与参考资料。

#### 问题描述

>   VC++ 6.0 运行程序时报错：Can’t find or open PDB file

运行时提示找不到 **PDB（Program Database）** 文件，这是 VC 调试数据库，用于存储断点、符号表、中间调试信息。
 考试环境通常禁用/删除调试选项，因此程序运行完会直接闪退，看起来像 “找不到 PDB 文件”。

#### 问题原因

>   调试信息缺失导致程序运行后窗口一闪而过

-   VC++6 默认是 Debug 模式运行，需要 PDB 调试文件
-   考试环境一般会删掉文件或禁用调试
-   程序正常执行，但结束太快，看起来像错误

#### 解决方法

>   插入阻塞或强制暂停即可解决窗口闪退问题

**可用方案**

1.  在 `main()` 末尾加入

    ```
    getchar();
    ```

2.  在关键位置打断点

    -   让程序停在断点位置，窗口不会闪退

3.  使用

    ```
    Ctrl + F5
    ```

    -   即 “不调试运行”，可避免 PDB 需求

**以上任意一项即可解决闪退与 PDB 警告**

------

### 参考资料

>   查看更详细的原因分析与示例

1.  CSDN：VC++6.0 Can't Find or Open PDB File
     https://blog.csdn.net/qq_17820539/article/details/95963056
2.  Bilibili：相关讲解视频
     https://www.bilibili.com/video/BV1A5AUezEem/?spm_id_from=333.337.search-card.all.click&vd_source=43bff3dbe361c224a6ecfc545aa2b0b5