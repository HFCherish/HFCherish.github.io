---
title: fantastic tools
tags:
  - tool

---

# 设计

收集一些在设计时可能会用到的比较好的网站

* [icons from the fontawesome](https://fontawesome.com/v4.7.0/icons/): 各种各样的图标，很漂亮
* [ant design](https://ant.design/docs/spec/introduce-cn)：里边有 UI 设计价值观及图标资源等，还有前端组件库

## 前端组件库

[组件库大合集](https://zhuanlan.zhihu.com/p/24650288)

[组件库大合集2](https://github.com/JingwenTian/awesome-frontend)

# mac

[强迫症的Mac设置指南](https://insights.thoughtworks.cn/ocds-guide-to-setting-up-mac/)

## terminal

[10 Must know terminal commands and tips for productivity](10 Must know terminal commands and tips for productivity) 介绍了一些，简单罗列下：

1. `iterm2`：是一种 terminal，将其作为默认 terminal，可以方便的分屏等
2. `oh my zsh`：管理 zsh 配置的，使用 `~/.zshrc`，利用 `source ~/.zshrc` 可以是配置立马生效
   1. [oh my zsh 插件](https://hustyichi.github.io/2018/09/19/oh-my-zsh/)
3. `cat`：不打开文件的情况下查看内容
4. `imgcat`：同上，不过查看的是图片
5. `less [filename]`：同 `cat`，不过如果长文件，可以用这个，仅显示一部分
6. `pbcopy < [filename]`：复制文件内容到 clipboard
7. `touch`：创建各种各样的文件，可以同时创建多个。eg. `touch index.html readme.md index.php`
8. `lsof -i :[port]`：查看端口占用
9. `&&`：实现 command chaining，即同时写多个命令，依次执行。eg. `npm i && npm start`
10. `open .`：打开当前目录

### iterm2 + oh my zsh + theme 配置

1. [download iterm2](https://www.iterm2.com/)，解压安装
2. 设置 iterm2 为默认 terminal：iterm2 -> make iterm2 default Term
3. 安装 [oh my zsh](https://github.com/robbyrussell/oh-my-zsh): 
   * `sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
     `
4. 配置主题和颜色
   4. 主题，我选用默认的 robbyrussell。其他下载主题然后在 `~/.zshrc` 中配置 `ZSH_THEME`
   5. 颜色，下载 [iterm2 color schemes](https://github.com/mbadolato/iTerm2-Color-Schemes)，然后在 iterm2 perferences -> profiles -> colors -> color presets -> import -> 从前边下载的库中选择自己喜欢的 scheme
5. 配置高亮
   * `brew install zsh-syntax-highlighting`
   * 在 `~/.zshrc` 中添加：`source /usr/local/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh`
   * 刷新配置使其生效：`source ~/.zshrc`
6. 显示快捷键配置
   * preferences -> keys -> hotKey -> Show/Hide all... -> 设置为 `⌘.`
7. 常用的配置：
   1. 配置新开重开使用之前的目录：preferences -> profiles -> general -> working directory -> reuse previous session's directory
   2. 配置快速切换 iterm 的快捷键：preferences -> keys -> hot key -> show/hide all windows with a system-wide hotkey
   3. 配置窗口透明度和默认启动大小：preferences -> profiles -> window
   4. 配置 command line move-by-word, delete-by-word: preferences -> profiles -> keys ([stackoverflow](https://apple.stackexchange.com/questions/154292/iterm-going-one-word-backwards-and-forwards))
      1. 向左移动 by word (⌥b)，向右移动 by word (⌥f)，删除右边 by word (⌥d)
         1. Under Profile Shortcut Keys, click the + sign.
         2. Type your key shortcut (option-b, option-f, option-d, option-left, etc.)
         3. For Action, choose Send Escape Sequence.
         4. Write b, d or f in the input field.
      2. 删除左边 by word (⌥⌫)
         1. preferences -> profiles -> keys -> left option (⌥) key : Esc+

### iterm2 常用快捷键

* `⌘d`：横分屏
* `⌘⇧d`：竖分屏
* `⌘⌥ + direction`：navigate between panes
* `⌘.`：show/hide iterm2
* `⌘⇧↩︎`: 最大化当前 pane / 回复当前 pane

# Git

* [git-fitler-repo](https://github.com/newren/git-filter-repo)
* 常用的 git config 配置

```sh
# 在 ~/.zshrc 中添加 alias
$ vi ~/.zshrc
alias g="/usr/bin/git"

# 在 .gitconfig 中添加 alias 配置
$ vi ~/.gitconfig
[user]
        name = xxx
        email = xxx@some.com

[alias]
        a = add
        aa = add .
        b = branch
        c = commit
        cm = commit -m
        ca = commit --amend
        d = diff
        l = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)%Creset | %C(bold)%an' --abbrev-commit --date=relative
        o = checkout
        pl = pull
        pb = pull --rebase
        ps = push
        r = reset
        st = status
```



# reading

* [我的小书屋](http://www.shuwu.mobi/)
* [readfree](http://readfree.me/)：可以直接推送到 kindle，很方便
* [不老歌](http://bulaoge.cn/)：写日记
* [宝书网](https://www.baoshuu.com/)

# music

* [超高无损音乐下载](https://www.sq688.com)

# json

* [json formatter](https://jsonformatter.org/xml-viewer): json、xml 等都支持，可以格式化，有 json tree，统计了节点数
* [json editor online](https://jsoneditoronline.org/): 界面简单，有 json tree，统计了 tree 节点数
* [json viewer](http://jsonviewer.stack.hu/): 功能精简，格式化 json，没有 tree
* [yaml to json](https://codebeautify.org/yaml-to-json-xml-csv): yaml 和 json 之间的转化

# Editing

* typora: markdown tool
* [clipy](https://github.com/Clipy/Clipy): free tool for pasting

# Others

* [karabiner](https://karabiner-elements.pqrs.org/): 定义快捷键。可以将 caps 键重新映射，在自定义快捷键时避免冲突
* [shiftit](https://github.com/fikovnik/ShiftIt): 窗口 size & location 定义。用 brew 安装 `brew cask install shiftit`
* Eudic: 好用的词典，可以有一个简单悬浮窗口
* Alfred: easy used spotlight substitute to search in Mac
* [cheatsheet](https://mediaatelier.com/CheatSheet/?lang=en): 显示当前应用的快捷键。Just hold the ⌘-Key a bit longer to get a list of all active short cuts of the current application.

# 给图片加水印

* 美图秀秀：批量添加水印，编辑水印位置、大小、透明度
* [Photoshop 批量添加水印](https://jingyan.baidu.com/article/0eb457e555170643f1a90592.html)
* 免费软件：
  * [XnConvert](https://www.xnview.com/en/xnconvert/#downloads)：[使用 XnConvert 批量添加水印](https://uiiiuiii.com/software/202192.html)
  * [imagemagick](https://imagemagick.org/index.php): 基于命令行的图形处理库（现有的图像处理软件大多都用到此库）
* [6 个小工具，打造图片批处理工作流](https://sspai.com/post/53079)

## ImageMagick

```sh
# 安装
$ brew install imagemagick

$ cd /images
# 获取图片基本信息
$ identify a.jpg
# 转换图片格式
$ magick a.jpg a.png


# 批量为图片添加水印
  1 #!/bin/bash
  2
  3 dir=$1
  4 mark=$2
  5
  6 echo "the images dir to process: $dir"
  7 echo "the mark location: $mark"
  8
  9 shopt -s nullglob # 如果不添加这个，当目录中没有 .png 类型的文件时，他会产生 "$dir"/*.png，那么后边就会报错
 10 for each in "$dir"/{*.jpg,*.jpeg,*.png}
 11 do
 12         echo "each is: $each"
 13         convert $each $mark -gravity southeast -geometry +5+20 -composite $each
 14         convert $each $mark -gravity center -composite $each
 15         convert $each $mark -gravity northwest -geometry +5+20 -composite $each
 16         echo "$each: done"
 17 done
 18 shopt -u nullglob
 19 exit 0
```

# Windows

## terminal

[打造 Windows 10 下最强终端方案：WSL + Terminus + Oh My Zsh + The Fuck](https://p3terx.com/archives/the-strongest-terminal-solution-under-windows-10.html)

> 注意这里装的是 wsl1，可以改成 wsl2，参见[install wsl](https://docs.microsoft.com/en-us/windows/wsl/install)

## copy

mac 下有免费的 clipy 来实现 copy history，windows 下有 [copyq](https://copyq.readthedocs.io/en/latest/basic-usage.html)（也可以用于 mac）

copyq 在它的窗口里可以配置各种命令的快捷键（全局或程序内），包括显示和隐藏 copyq 窗口的

## search

mac 下有 alfred 来搜索文件，windows 有一些替代的：

1. listary：有免费版
2. wox：开源

