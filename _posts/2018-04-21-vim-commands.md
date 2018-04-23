---
layout: post
title:  "Vi/Vim 基础命令"
date:   2018-04-21 20:11:20 +0800
categories: jekyll update
---
You can contact me via e-mail: [mistydew@qq.com](https://en.mail.qq.com).

Vim(Vi IMproved) 是 Vi 文本编辑器的升级版，在 Linux 系统中作为 “编辑器之神” 与 “神之编辑器” Emacs 并驾齐驱。

下图展示了各自的学习曲线，其中不乏调侃之意。
<br>
![images](/images/20180421/vi_emacs_learning_curves.jpg)
<br>
（横坐标对应熟练度，纵坐标对应技能）

## 模式 modes
* 基本分为两大模式，**普通（命令）模式**和**插入模式**。
* 普通模式为默认模式用于修改文本，插入模式用于输入文本。
* 普通模式下按`i`进入插入模式，插入模式下按`Esc`返回普通模式。
* 普通模式下可以输入命令进行文本的修改。

## 命令（普通模式）

> ### 插入 insert
`i` 从当前光标位置开始插入文本。
<br>
`I` 从当前光标所在行的行首开始插入文本。
<br>
`a` 从当前光标的后一个位置开始插入文本。
<br>
`A` 从当前光标所在行的行尾开始插入文本。
<br>
`o` 从当前光标所在行的下一行的行首开始插入文本。

> ### 保存和退出 write and quit
`:w` 保存当前内容。
<br>
`:q` 退出当前文档，前提：内容未改变 或 改变的内容已保存。
<br>
`:wq` 保存并退出。
<br>
`:x` 同 `:wq`。

> ### 移动光标
`w` 移动光标到下一个单词。
<br>
`b` 移动光标到上一个单词。
<br>
`^` 移动光标到所在行的行首。
<br>
`$` 移动光标到所在行的行尾。
<br>
`k` 上移，移动光标到所在行的上一行。
<br>
`j` 下移，移动光标到所在行的下一行。
<br>
`h` 左移，移动光标到所在位置的前一个位置。
<br>
`l` 右移（小写字母 `L`），移动光标到所在位置的后一个位置。
<br>
`H` 移动光标到所在页首行的行首。
<br>
`L` 移动光标到所在页尾行的行首。
<br>
`gg` 移动光标到文档首行的行首。
<br>
`G` 移动光标到文档尾行的行首。

> ### 选择 visual
`v` 进入选择模式，横向选择字符串，使用上下左右移动光标来进行选择，类似于 windows 下鼠标左键点选文字，你又可以抛弃鼠标了。
<br>
`V` 进入选择模式，纵向选择字符串，使用 `I` （大写字母 `i`）进行插入，一行输入完成后按 `Esc` 完成整个选中区域的修改，常用于批量注释代码端 或 代码对齐。

> ### 复制 yank
`y^` 复制当前光标的前一个位置到行首的内容。
<br>
`y$` 复制当前光标所在位置到行尾的内容。
<br>
`yy` 复制当前光标所在行的内容。
<br>
`nyy` n 表示数字，复制包含当前光标所在行之后的 n 行的内容。

> ### 粘贴 paste
`p` 从当前光标的后一个位置开始粘贴缓冲区中（复制/删除）的内容，若使用`yy`复制行，则从当前光标所在行的下一行开始粘贴。

> ### 剪贴/删除 delete
`x` 删除当前光标所在位置的字符，并送入缓冲区。
<br>
`dd` 删除当前光标所在行的内容，并送入缓冲区。
<br>
`ndd` n 表示数字，删除包含当前光标所在行之后的 n 行的内容，并送入缓冲区。
<br>
`D` 删除当前光标所在位置到行尾的内容，并送入缓冲区，常用于删除注释。
<br>
**注：复制 和 删除共用同一个缓冲区，所以复制后删除 或 剪贴后复制均会覆盖第一次操作的内容。**

> ### 撤销操作 undo
`u` 撤销上次（最近一次）操作，使用次数不限一次。
<br>
`R` 撤销上次（最近一次） `u` 操作，使用次数不限一次。

> ### 搜索和定位
`/xxx` 查找 xxx 字符串并高亮所有结果，同时定位光标到第一个结果，大小写不敏感。
<br>
`:n` 跳转到第 n 行。
<br>
`%` 括号匹配，适用于 `{...}`, `[...]`, `(...)`。

> ### 字符串替换 substitute
`:s/src/dst` 替换当前光标所在行内第一个 src 字符串为 dst 字符串，同时高亮全文所有 src 字符串。
<br>
`:s/src/dst/ig` 替换当前光标所在行内所有 src 字符串为 dst 字符串，i 表示忽略大小写，g 表示全部，同时高亮全文所有 src 字符串。
<br>
`:n,m s/src/dst/g` 替换第 n 行到第 m 行内所有 src 字符串为 dst 字符串，同时高亮全文所有 src 字符串。
<br>
`:%s/src/dst/g` 替换文档内所有 src 字符串为 dst 字符串，同时高亮全文所有 src 字符串，然而此时已经不存在 src 字符串了（笑）。

> ### 进制转换
`:%!xxd` 转换全文为 16 进制。
<br>
`:%!xxd -r` 恢复 16 进制为原文。

> ### 锁定
`Ctrl + s` 进入锁定状态，即不响应除 xxx （见下一条命令）的键盘输入，一定是用惯 windows 下快捷键了（笑）。
<br>
`Ctrl + q` 解除锁定状态。

> ### 显示行号
`:set nu` or `:set number` 显示行号。
<br>
`:set nonu` or `:set nonumber` 隐藏行号。

## 参考
* [Vim (text editor) - Wikipedia](https://en.wikipedia.org/wiki/Vim_(text_editor))
* [Vim - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/Vim)
* [Learning the vi Editor/Vim/Modes - Wikibooks](https://en.wikibooks.org/wiki/Learning_the_vi_Editor/Vim/Modes)
* [...][md]

Check out the [mistydew][md] for more info on who am I.

[md]: http://github.com/mistydew