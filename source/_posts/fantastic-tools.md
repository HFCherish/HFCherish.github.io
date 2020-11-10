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
3. `cat`：不打开文件的情况下查看内容
4. `imgcat`：同上，不过查看的是图片
5. `less [filename]`：同 `cat`，不过如果长文件，可以用这个，仅显示一部分
5. `pbcopy < [filename]`：复制文件内容到 clipboard
6. `touch`：创建各种各样的文件，可以同时创建多个。eg. `touch index.html readme.md index.php`
7. `lsof -i :[port]`：查看端口占用
8. `&&`：实现 command chaining，即同时写多个命令，依次执行。eg. `npm i && npm start`
9. `open .`：打开当前目录

### iterm2 + oh my zsh + theme 配置

1. [download iterm2](https://www.iterm2.com/)，解压安装
2. 设置 iterm2 为默认 terminal：iterm2 -> make iterm2 default Term
2. 安装 [oh my zsh](https://github.com/robbyrussell/oh-my-zsh): 
	* `sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
`
3. 配置主题和颜色
	4. 主题，我选用默认的 robbyrussell。其他下载主题然后在 `~/.zshrc` 中配置 `ZSH_THEME`
	5. 颜色，下载 [iterm2 color schemes](https://github.com/mbadolato/iTerm2-Color-Schemes)，然后在 iterm2 perferences -> profiles -> colors -> color presets -> import -> 从前边下载的库中选择自己喜欢的 scheme
6. 配置高亮
	* `brew install zsh-syntax-highlighting`
	* 在 `~/.zshrc` 中添加：`source /usr/local/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh`
	* 刷新配置使其生效：`source ~/.zshrc`
7. 显示快捷键配置
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

