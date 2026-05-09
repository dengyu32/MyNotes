 ## GNOME Quarter Tiling 笔记

  ### 目录

  [TOC]

  ### 原理

  - GNOME Shell 默认只做左右/上下 1/2 平铺，不含四角 1/4。
  - 四分屏需额外扩展：Tiling Assistant（官方扩展站提供）。
  - 扩展依赖浏览器连接器：chrome-gnome-shell + 浏览器插件。

  ### 具体配置

  > 自查：确认桌面环境

  echo $XDG_CURRENT_DESKTOP

  输出含 GNOME 才继续。

  > 安装扩展支持（Ubuntu 22.04）

  sudo apt update
  sudo apt install gnome-shell-extensions chrome-gnome-shell

  22.04 无 gnome-browser-connector 包，忽略它即可。

  > 安装浏览器 GNOME Shell 插件

  - 打开 https://extensions.gnome.org/
  - 顶部提示 “Install GNOME Shell integration” → 点击安装（Chrome/Chromium/
    Firefox 均可）。

  > 打开并安装 Tiling Assistant

  - 访问 https://extensions.gnome.org/extension/3733/tiling-assistant/
  - 右上角开关 OFF → ON，浏览器弹窗选 Install。

  > 确认扩展安装成功

  gnome-extensions list

  应看到：tiling-assistant@leleat-on-github

  > 打开扩展设置

  gnome-extensions prefs tiling-assistant@leleat-on-github

  > 开启关键选项

  - Enable Quarter Tiling（四分屏）
  - Snap to Screen Corners（拖到角落吸附）
  - Show Tile Previews（显示预览）

  > 测试四分屏

  gnome-terminal &

  把窗口拖到左上 / 右上 / 左下 / 右下，出现预览后松手，应各占 1/4 屏。

  ### 常见问题与处理

  1. 找不到 gnome-browser-connector
      - 22.04 不提供该包，使用 chrome-gnome-shell 即可。
  2. 扩展开关灰色不可点
      - 浏览器插件未装或未授权；重新按网站提示安装插件并刷新。
  3. 无预览/不吸附
      - 在 Tiling Assistant 里勾选 Quarter Tiling 与 Snap to Screen Corners。若
        仍无效，Alt+F2 输入 r 重启 GNOME Shell（Wayland 需重登）。

### 新人快速上手

  1. sudo apt install gnome-shell-extensions chrome-gnome-shell
  2. 浏览器装 GNOME Shell integration，打开 Tiling Assistant 页面 → 开关置 ON 安
     装
  3. gnome-extensions list 看到 tiling-assistant@leleat-on-github
  4. gnome-extensions prefs ... 勾选 Quarter Tiling / Snap / Previews
  5. 拖窗口到四角验证 1/4 平铺

  ### 记忆要点

  - GNOME 默认无四角平铺，Tiling Assistant 才能实现。
  - 22.04 用 chrome-gnome-shell，无需 gnome-browser-connector。
  - 开关安装后在 prefs 勾选 Quarter Tiling 即可拖到四角。