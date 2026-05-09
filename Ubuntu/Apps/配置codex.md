# 配置 codex (CLI)

步骤：

1.   安装node.js

>   https://nodejs.org/en/download 
>
>   ~~~bash
>   # Download and install nvm:
>   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
>   
>   # in lieu of restarting the shell
>   \. "$HOME/.nvm/nvm.sh"
>   
>   # Download and install Node.js:
>   nvm install 24
>   
>   # Verify the Node.js version:
>   node -v # Should print "v24.12.0".
>   
>   # Verify npm version:
>   npm -v # Should print "11.6.2".
>   ~~~

2.   确认版本

     ~~~bash
     node -v    # >= 18
     # 或
     python3 --version  # >= 3.9
     ~~~

3.    安装Codex (通过npm安装)

     ~~~bash
     npm i -g @openai/codex
     ~~~

4.    进入项目目录，运行指令

     codex

这里就可以使用codex了，注意plus才能使用codex功能

视频讲解：

https://www.bilibili.com/video/BV1wm4UzfEbr/?spm_id_from=333.337.search-card.all.click&vd_source=43bff3dbe361c224a6ecfc545aa2b0b5