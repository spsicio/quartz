---
title: Vim
created: 2025-10-01
modified: 2025-10-05
---

[Vim](https://github.com/vim/vim)，一款大部分人不知道如何退出的神秘软件。

## 配置

### 输入法切换

在 Vim 的普通模式下使用中文输入法（且未按「Shift」键切换为西文）时，`hjkl` 等按键会被输入法读取，从而无法正常作用。

为了在进入与退出插入模式时自动实现输入法切换，我们可以在 vimrc 中加入以下两行代码：

```vim
" 取消激活输入法（也可以考虑存储当前输入法的名字）
autocmd InsertLeave * call system('fcitx5-remote -c')

" 激活输入法（也可以考虑切换到对应名字的输入法）
autocmd InsertEnter * call system('fcitx5-remote -o')
```

这里选择了 [[fcitx|Fcitx]] 作为示例。更通用地，可以使用 [im-select](https://github.com/daipeihust/im-select) 控制输入法状态。
