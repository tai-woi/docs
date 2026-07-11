
+ 来源：![Deepseek](#bot-message-square "purple")
+ 编辑：[太亮](/贡献者/太亮.md)
+ 修订日期：2026-7-6

tmux 是一个终端复用器（terminal multiplexer），可以让你在一个终端窗口中管理多个会话、窗口和面板。下面是最基本的使用指南：

核心概念
====

+ 会话（Session）：一个独立的工作环境，可以包含多个窗口
+ 窗口（Window）：类似浏览器标签页，每个窗口有独立 shell
+ 面板（Pane）：窗口内分割的小区域，可同时显示多个终端



启动与退出
====

+ 启动一个新会话：`tmux`
+ 退出当前会话（关闭所有窗口后）：`exit`
+ 创建并命名会话：`tmux new -s 名称`
+ 列出所有会话：`tmux ls`
+ 重新连接会话：`tmux attach -t 名称`
+ 杀死指定会话：`tmux kill-session -t 名称`
+ 杀死所有 tmux 会话：`tmux kill-server`



前缀键
====

进入 tmux 会话后，tmux 的所有快捷键都以 `Ctrl+b` 为前缀（Prefix Key），先按 `Ctrl+b`，再按其他键。


常用快捷键
====

会话管理
----

+ 暂时脱离（detach）当前会话，回到终端：`Ctrl+b d`
+ 显示会话列表，选择切换：`Ctrl+b s`
+ 重命名当前会话：`Ctrl+b $`


窗口管理
----

+ 创建新窗口：`Ctrl+b c`
+ 切换到下一个窗口：`Ctrl+b n`
+ 切换到上一个窗口：`Ctrl+b p`
+ 切换到指定编号的窗口：`Ctrl+b 数字`（0-9）
+ 窗口列表，选择切换：`Ctrl+b w`
+ 重命名当前窗口：`Ctrl+b ,`
+ 关闭当前窗口（需确认）：`Ctrl+b &`


面板管理
----

+ 垂直分割面板（左右分屏）：`Ctrl+b %`
+ 水平分割面板（上下分屏）：`Ctrl+b "`
+ 切换光标到相邻面板：`Ctrl+b 方向键`
+ 调整面板大小：`Ctrl+b` + `Ctrl+方向键`
+ 当前面板最大化/恢复（切换）：`Ctrl+b z`
+ 关闭当前面板（需确认）：`Ctrl+b x`
+ 将当前面板拆分为独立窗口：`Ctrl+b !`
+ 切换到下一个面板：`Ctrl+b o`


复制搜索模式
----

+ 进入复制模式：`Ctrl+b [`
+ 翻页：`PgUp / PgDn`
+ 向下搜索：`/`
+ 向上搜索：`?`
+ 开始搜索：输入关键词后按 `Enter`
+ 跳到下一个匹配：`n`
+ 跳到上一个匹配：`N`
+ 开始选择文本：`Space`
+ 复制选中的文本：`Enter`
+ 粘贴复制的内容：`Ctrl+b ]`



配置建议
====

创建 `~/.tmux.conf` 配置文件，常用配置：

```
# 开启鼠标支持
set -g mouse on

# 重新加载配置文件（不用退出 tmux 就能生效）
bind r source-file ~/.tmux.conf \; display "Reloaded!"

# 设置复制模式使用 Vi 快捷键（按 前缀 + [ 进入）。检查当前模式： tmux show -g mode-keys
set -g mode-keys vi

# 当前窗口：使用 reverse（反转）属性
set -g window-status-current-format "#[reverse] #I:#W "
set -g window-status-current-style reverse

# 状态栏右侧显示日期时间
set -g status-right " %Y-%m-%d %H:%M "

# | 垂直分屏， - 水平分屏
bind | split-window -h
bind - split-window -v
```

配置生效：`tmux source-file ~/.tmux.conf`



快速上手流程
====

1. 启动：`tmux new -s work`
2. 分割面板：`Ctrl+b %` 左右分屏，`Ctrl+b "` 上下分屏
3. 切换面板：`Ctrl+b 方向键`
4. 创建新窗口：`Ctrl+b c`
5. 切换窗口：`Ctrl+b n/p` 或 `Ctrl+b 数字`
6. 临时离开：`Ctrl+b d`
7. 回来：`tmux attach`
