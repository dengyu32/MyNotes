# 初步学习LaTex

>   LaTex 是论文排班工具，具有一定的学习门槛
>
>   今日作为初学者（2025.11.23）花30分钟粗浅的学习一下
>
>   学习视频为 https://www.bilibili.com/video/BV1Mc411S75c/?spm_id_from=333.337.search-card.all.click&vd_source=43bff3dbe361c224a6ecfc545aa2b0b5
>
>   推荐网址 https://www.overleaf.com/learn/latex/Learn_LaTeX_in_30_minutes
>
>   配置环境链接https://blog.csdn.net/qq_36265860/article/details/82972402

### 目录

[TOC]

### 环境

>线上 Latex 编辑环境 Overleaf
>
>本地 Vscode + TeXLive + ChkTeX
>
>TeXLive -->  编译.tex成为PDF文件，进行宏包的管理
>
>ChkTeX --> 检查拼写和LaTeX风格问题

本地开发时，需要安装 vscode 插件 LaTex Workshop 、Latex 和 系统 TeXLive 、 ChkTeX

~~~bash
sudo apt update
sudo apt install texlive-full
sudo apt install chktex
~~~

等待下载的时间可能会很长，会在这里卡住

~~~bash
Running mtxrun --generate. This may take some time... done.
Pregenerating ConTeXt MarkIV format. This may take some time... 
~~~

等待时间超过二十分钟证明已经卡死，输入

~~~bash
ps aux | grep mtxrun
~~~

~~~bash
root      125827  0.0  0.3  97356 78872 pts/7    S+   01:55   0:00 texlua /usr/bin/mtxrun --script base --make cont-en
wrj       146369  0.0  0.0  12328  2240 pts/8    S+   02:58   0:00 grep --color=auto mtxrun
~~~

~~~bash
sudo pkill mtxrun
sudo pkill luatex
sudo pkill context
~~~

即可

LaTeX Workshop插件设置搜索latex-workshop.latex.recipes，加入settings.json

~~~json
 
    "latex-workshop.latex.recipes": [
        {
            "name": "xelatex",
        "tools": [
          "xelatex"
        ]
        },
        {
        "name": "xelatex->bibtex->exlatex*2",
        "tools": [
          "xelatex",
          "bibtex",
          "xelatex",
          "xelatex"
        ]
      }],
 
    "latex-workshop.latex.tools":[
        {
            "name":"xelatex",
            "command": "xelatex",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOC%"
            ]
        }, {
            "name":"bibtex",
            "command": "bibtex",
            "args": [
                "%DOCFILE%"
            ]
        }
    ],
~~~

搜索latex-workshop.formatting.latex设置为 latexindent

安装微软核心指令

~~~bash
sudo apt update
sudo apt install ttf-mscorefonts-installer
~~~

### Latex 模版

>规定一片文章的所有公式
>
>修改标题、作者、撰写正文、插入公式、图标、添加引用等

具体分为

-   **注释 %** 

    在一行中 % 号后面的内容均会被注释掉，生成PDF文件是不会显示

-   **命令或特殊符号**

    \ 代表这是一个命令或者特殊符号

-   **普通文本**

![image-20251123003333066](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/20251211155831829.png)

### 正文

-   **设定区域和正文区域**

    ![image-20251123003529500](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/20251211155833091.png)

-   各级标题

    ![image-20251123003646741](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/20251211155835367.png)

-   换行 换段 换页 首行缩进 命令

    ![image-20251123003749569](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/20251211155843564.png)

### 特殊指令

使用中文

~~~tex
\usepackage{xeCJK}
\setCJKmainfont{WenQuanYi Micro Hei}
~~~

两个换行代表分段

居中

~~~tex
\begin{center}
(x,y,z) = (253.500, 146.354, 169.000)\ \text{mm}.
\end{center}

\centerline{(x,y,z) = (253.500, 146.354, 169.000)\ mm.}

\begin{center}
\textbf{(x,y,z) = (253.500, 146.354, 169.000)\ mm.}	加粗 + 居中
\end{center}
~~~

向左看齐

~~~tex
\begin{flushleft}
By setting $\alpha$ = $60^\circ$, the rotation matrix for a 60° rotation about the y-axis becomes:
\end{flushleft}
~~~



无序列表

~~~tex
\begin{itemize}
    \item 第一项
    \item 第二项
    \item 第三项
\end{itemize}
~~~

添加空白行

~~~tex
  \vspace{0.5em}
~~~

给文字加粗

~~~tex
This is \textbf{bold text} in a sentence.
{\bfseries This whole paragraph is bold.}	段落
\mathbf{x} \quad \text{or} \quad \boldsymbol{\theta} 	% \mathbf{}：粗体拉丁字母 	\boldsymbol{}：粗体希腊字母或符号
~~~

数学符号

$\mathbf{P}_0 = {(0,\,0,\,338)}^\mathrm{T}$​ mm.

~~~tex
$\mathbf{P}_0 = {(0,\,0,\,338)}^\mathrm{T}$ mm.
~~~

插入图片

~~~tex
\usepackage{graphicx}  % 用于插入图片

\begin{figure}[htbp]       % h: here, t: top, b: bottom, p: page
  \centering               % 图片居中
  \includegraphics[width=0.5\textwidth]{example.png} % 图片路径和宽度  图片放在相同目录
  \caption{This is the caption of the image.}       % 图题
  \label{fig:example}      % 标签，用于交叉引用
\end{figure}

\begin{figure}[htbp]
  \centering
  \includegraphics[width=\textwidth]{figure2-1.jpg}  % 图片路径
  \renewcommand{\thefigure}{2--1}
  \caption{Overall Problem-Solving Framework Diagram}         % 图题
\end{figure}
~~~

插入表格、数学模式 

汇聚在一个表格里

~~~tex
\section{Notations}

\begin{table}[htbp]
  \centering
  \begin{tabular}{|c|c|c|} % c: 居中 l: 左对齐 r: 右对齐
    \hline
    Symbol                     & Description                                             & Unit                               \\
    \hline
    $O_{SL}$                   & Origin of the left shoulder joint coordinate system     & $--$                               \\
    $L$                        & Total arm length                                        & $\mathrm{mm}$                      \\
    $\theta_{\text{trunk}}(t)$ & Torso rotation angle around the z-axis (time-dependent) & $^\circ$                           \\
    $\mathbf{R}_y(\alpha)$     & Rotation matrix about the y-axis by angle α             & $--$                               \\
    $\mathbf{R}_z(\beta)$      & Rotation matrix about the z-axis by angle β             & $--$                               \\
    $T_{\text{total}}$         & Total duration of leg walking motion                    & $\mathrm{s}$                       \\
    $T_0$                      & Gait cycle period                                       & $\mathrm{s}$                       \\
    $\theta_K(t)$              & Knee joint angle as a function of time                  & $^\circ$                           \\
    $\omega_K(t)$              & Knee joint angular velocity as a function of time       & $^\circ/\mathrm{s}$                \\
    $I$                        & Joint moment of inertia                                 & $\mathrm{kg \cdot m^2}$            \\
    $B$                        & Joint damping coefficient                               & $\mathrm{N \cdot m \cdot s / rad}$ \\
    $M(t)$                     & Total joint torque (time-dependent)                     & $\mathrm{N \cdot m}$               \\
    $P(t)$                     & Joint power (time-dependent)                            & $\mathrm{W}$                       \\
    $E$                        & Total energy consumption                                & $\mathrm{Wh}        $              \\
    $m$                        & Limb mass                                               & $\mathrm{kg}$                      \\
    $g$                        & Gravitational acceleration                              & $\mathrm{m/s^2}   $                \\
    $L_g$                      & Distance from limb center of gravity to the joint       & $\mathrm{m}$                        \\
    \hline
  \end{tabular}
  \caption{示例表格}
  \label{tab:example}
\end{table}
~~~

向量和矩阵

~~~tex
\begin{center}
  $\mathbf{P}_0 = {(0,\,0,\,338)}^\mathrm{T}$ mm.
\end{center}


\[
  R_y(\alpha) = \begin{bmatrix}
​    \cos \alpha  & 0 & \sin \alpha \\
​    0            & 1 & 0           \\
​    -\sin \alpha & 0 & \cos \alpha
  \end{bmatrix}
\]
~~~

