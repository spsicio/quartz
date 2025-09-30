---
title: Bash
data: 2025-09-30
---

在读写 Bash 脚本以外的时候，我不讨厌 Bash。

## 使用须知

使用 [ShellCheck](https://github.com/koalaman/shellcheck) 对脚本进行静态分析。

## 环境变量

### 提示符

Bash 中存在若干影响提示符的环境变量：

- `PS0`：在命令执行前进行提示。
- `PS1`：主要的提示符，也就是我们通常看到的 `user@host:dir$ `。
- `PS2`：在输入多行命令时的提示符。
- `PS3`：在执行选择时的提示符。
- `PS4`：调试时的提示符。（使用 `set -x` 与 `set +x` 进入与退出。）
- `PROMPT_COMMAND`：在主提示符显示前执行的命令。

通过修改这些变量，[bash-git-prompt](https://github.com/magicmonty/bash-git-prompt) 插件在提示符中显示仓库的 Git 状态，Python 使用虚拟环境时在提示符前面添加 `(venv) ` 等字符串作为提醒。

### 杂项

- `TMOUT`：用户在设定的秒数内没有输入时，将超时结束会话。**在服务器上执行命令时需要格外留意这个环境变量！**
