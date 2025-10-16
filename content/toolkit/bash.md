---
title: Bash
created: 2025-09-30
modified: 2025-10-11
---

在读写 Bash 脚本以外的时候，我不讨厌 Bash。

## 使用须知

使用 [ShellCheck](https://github.com/koalaman/shellcheck) 对脚本进行静态分析。  
可以使用 [Starship](https://starship.rs/) 在 shell 中进行美化。

## 环境变量

### 提示符

Bash 中存在若干影响提示符的环境变量：

- `PS0`：在命令执行前进行提示。
- `PS1`：主要的提示符，也就是我们通常看到的 `user@host:dir$ `。
- `PS2`：在输入多行命令时的提示符。
- `PS3`：在执行选择时的提示符。
- `PS4`：调试时的提示符。（使用 `set -x` 与 `set +x` 进入与退出。）
- `PROMPT_COMMAND`：在主提示符显示前执行的命令。

[bash-git-prompt](https://github.com/magicmonty/bash-git-prompt) 和 [Python Venv](https://docs.python.org/zh-cn/3.13/library/venv.html) 修改了 `PS1` 实现在命令行提示符中显示仓库的状态或者提醒目前的 Python 环境是虚拟环境。

### 杂项

- `TMOUT`：超时结束会话的秒数。**在服务器上时需要留意这个环境变量。**
