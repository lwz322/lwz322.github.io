---
layout: article
title: 配置Windows下的终端与Shell
mathjax: true
mermaid: true
chart: true
toc: true
mode: immersive
tags : CLI Shell Terminal
key: terminal
header:
  theme: dark
article_header:
  type: overlay
  theme: dark
  background_color: '#ffffff'
  background_image: 
    gradient: 'linear-gradient(0deg, rgba(0, 0, 0 , .7), rgba(0, 0, 0, .7))'
    src: assets/background/Terminal.png
---
在Microsoft Build 2019上微软发布了Windows Terminal，我也提前体验了一把，趁此机会配置一下Windows下的终端和shell，另外就是附带了对现阶段的Windows Terminal的配置文件，使其接近宣传图中的效果
<!--more-->

## Windows下的终端与Shell

> 这里需要了解一下终端terminal和 shell 的区别：
> 在命令行中，shell 提供了访问操作系统内核功能的途径，比如说我们所熟悉的 bash、zsh，都是不同的 shell；而终端则为 shell 提供视觉界面（窗口），比如我们所熟悉的 iTerm2、Linux 桌面上的终端工具等。甚至于我们在 VSCode 中所使用的命令行，也是某种意义上的终端。
> 我们在 Windows 下所使用的 CMD、Powershell 既然是一个终端，也是一个 Shell，还是同名的脚本系统。

所以Windows Termianl只是一个终端而已，而不是一个更加好用的Shell，改善体验的话两边都要下手

对于终端，最熟悉的莫过于CMD的黑色背景的终端以及Powershell的蓝色背景的终端，用过WSL的可能还会注意到默认的Bash终端也有些不同，相比于其他“现代终端软件”...

见过了[Cmder](https://cmder.net/)之后就没有考虑这方面问题，Cmder是一个Console Emulator,对这个概念个人的理解是这样的：

> 在图形化界面出现之前，与Unix系统交互的唯一方式就是借助Shell所提供的文本命令行界面（Command Line Interface,CLI）,而日常用的操作系统都配备了图形化界面，如一些Linux发行版进入CLI的一种方法是退出图形化界面，进入文本模式，也就是显示器上只有shell CLI，这种方式就叫做Linux控制台（Linux Console）,而Linux系统启动后又会创建出多个虚拟控制台（TTY）方便使用；而另外一种方式也就是在图形化界面打开图形化的终端窗口连接到TTY，也就是终端仿真器（Console Emulator）。

Cmder不仅仅解决了Windows下终端的美观性问题还引入了部分Linux命令使得shell的体验更好

然而这个并没有真正的集成到系统内去，比如说Git依然需要单独安装，尽管使用Scoop解决了安装软件和配置环境变量的问题，系统自带终端和Shell依然是不好用，因为个人一直觉得能用自带的就不用第三方的软件所以就往配置方面查了一些东西，发现主要是以下几个问题：

- 终端代码页不支持大部分的字体，如Consola

- 终端的配色、主题自定义相对不便

- Powershell用不习惯

下面就来一个个的解决

### 修改终端软件的字体

默认的Powershell终端窗口是无法使用Consola字体的，考虑到偶尔还是要用的，所以可以在运行的时候输入chcp 65001使代码页用UTF8进行编码，而在新版本的Win10中，在“控制面板-时钟和区域-区域-管理-更改系统区域设置”可以打开系统编码默认为UTF8的设置，看了下评论，现阶段还是可能存在一些问题，我暂时没有遇到，另外还有其他的字体可以不依赖于此使用:

[Microsoft YaHei Mono](https://github.com/Microsoft/BashOnWindows/files/1362006/Microsoft.YaHei.Mono.zip) on GitHub 微软为 WSL/Bash on Ubuntu on Windows 设计的字体，PowerShell 和 cmd也能用效果相当于微软雅黑和 Consolas 的混搭

更详细的可以参考：[自定义 Windows PowerShell 和 cmd 的字体](https://blog.walterlv.com/post/customize-fonts-of-command-window.html)

### 使用Pshazz改善Powershell体验

作者在其Github项目页面：[lukesampson/pshazz](https://github.com/lukesampson/pshazz)介绍如下：

给你的powershell一些pizazz，Pshazz扩展了您的Powershell配置文件以添加类似的内容：

- 一个更好的提示，包括Git和Mercurial信息
- Git和Mercurial标签完成
- 一个SSH帮助程序，可让您再也不输入私钥密码
- 明智的别名，以及添加自己的别名和删除不喜欢的别名的简单方法

主题方面，个人暂时使用的主题是基于某个自带主题修改的, 可以在```~\scoop\apps\pshazz\0.2019.04.02\themes```目录下修改或者创建主题配置文件，下面的Concfg也类似

```json
{
    "plugins": [ "git", "ssh", "z" ],
    "prompt": [
        [ "green",  "", " $path" ],
        [ "white",   "", "$(if ($git_branch) {' on git:'} else {':'})" ],
        [ "cyan",    "", "$git_branch" ],
        [ "red",   "", " $git_local_state" ],
        [ "white",   "", " $git_remote_state" ],
        [ "",        "", " [$([datetime]::now.tostring(\"HH:mm:ss\"))]" ],
        [ "white", "", "`n`$" ]
    ],
    "git": {
        "prompt_unstaged": "*",
        "prompt_staged": "+",
        "prompt_stash": "$",
        "prompt_untracked": "%",
        "prompt_remote_push": ">",
        "prompt_remote_pull": "<",
        "prompt_remote_same": "="
    }
}

```
这个作者同时也开发了Windows下的包管理软件scoop，在我[之前的文章](https://lwz322.github.io/2019/03/30/Efee.html#%E9%99%84windows%E4%B8%8B%E7%9A%84%E5%8C%85%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7scoop)中有介绍，网上搜一下也有其他人写的配置文章；另外一个就是自定义配色的问题了，依然使用这个作者开发的工具

### 使用Concfg修改shell的配色

先贴上Github项目链接：[lukesampson/concfg](https://github.com/lukesampson/concfg)

concfg 是一个实用程序，用于导入和导出Windows控制台设置，如字体和颜色，这里主要是更改一下颜色的映射关系，因为最后呈现的效果也取决于终端，所以还是时不时还是需要细调（建议给编辑器装些色彩方面的插件），我暂时的配置文件如下：

```json
{
    "cursor_size":  "small",
    "command_history_length":  50,
    "num_history_buffers":  4,
    "command_history_no_duplication":  false,
    "quick_edit":  true,
    "insert_mode":  true,
    "load_console_IME":  true,
    "font_face":  "Consolas",
    "font_true_type":  true,
    "font_size":  "0x18",
    "font_weight":  0,
    "screen_buffer_size":  "80x3000",
    "window_size":  "80x20",
    "fullscreen":  false,
    "popup_colors":  "cyan,white",
    "screen_colors":  "white,black",
    "black":  "#1E1E1E",
    "dark_blue":  "#2472C8",
    "dark_green":  "#0DBC79",
    "dark_cyan":  "#11A8CD",
    "dark_red":  "#CD3131",
    "dark_magenta":  "#BC3FBC",
    "dark_yellow":  "#E5E510",
    "gray":  "#E5E5E5",
    "dark_gray":  "#666666",
    "blue":  "#3B8EEA",
    "green":  "#23D18B",
    "cyan":  "#29B8DB",
    "red":  "#F14C4C",
    "magenta":  "#D670D6",
    "yellow":  "#F5F543",
    "white":  "#E5E5E5"
}
```

## Windows Terminal的配置

现阶段只有Json的方式做配置，添加了icon，修改了部分参数使得看起来更舒服，下面是主要修改的部分

```json
{
    "defaultProfile": "{5cc811bb-d066-4de2-aea7-0bd51430fa9c}",
    "initialRows": 30,
    "initialCols": 120,
    "alwaysShowTabs": true,
    "showTerminalTitleInTitlebar": false,
    "experimental_showTabsInTitlebar": true,
    "profiles": [
        {
            "guid": "{2cc811bb-d066-4de2-aea7-0bd51430fa9c}",
            "name": "CMD",
            "colorscheme": "Campbell",
            "historySize": 9001,
            "snapOnInput": true,
            "cursorColor": "#FFFFFF",
            "cursorShape": "bar",
            "commandline": "cmd.exe",
            "fontFace": "Consolas",
            "fontSize": 11,
            "acrylicOpacity": 0.75,
            "useAcrylic": true,
            "closeOnExit": false,
            "padding": "0, 0, 0, 0",
            "icon": "C:\\Users\\lwz32\\Pictures\\console.png"
        },
        {
            "guid": "{3cc811bb-d066-4de2-aea7-0bd51430fa9c}",
            "name": "WSL",
            "colorscheme": "Campbell",
            "historySize": 9001,
            "snapOnInput": true,
            "cursorColor": "#FFFFFF",
            "cursorShape": "bar",
            "commandline": "wsl.exe",
            "fontFace": "Consolas",
            "fontSize": 11,
            "acrylicOpacity": 0.75,
            "useAcrylic": true,
            "closeOnExit": false,
            "padding": "0, 0, 0, 0",
            "icon": "C:\\Users\\lwz32\\Pictures\\ubuntu_.ico"
        },
        {
            "guid": "{5cc811bb-d066-4de2-aea7-0bd51430fa9c}",
            "name": "PS",
            "colorscheme": "Campbell",
            "historySize": 9001,
            "snapOnInput": true,
            "cursorColor": "#FFFFFF",
            "cursorShape": "bar",
            "commandline": "powershell",
            "fontFace": "Consolas",
            "fontSize": 11,
            "acrylicOpacity": 0.75,
            "useAcrylic": true,
            "closeOnExit": false,
            "padding": "0, 0, 0, 0",
            "icon": "C:\\Users\\lwz32\\Pictures\\powershell.ico"
        }
    ]
        }
    ]
}
```