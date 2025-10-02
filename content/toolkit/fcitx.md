---
title: Fcitx
data: 2025-10-01
---

[Fcitx](https://fcitx-im.org/wiki/Fcitx_5/zh-cn) /ˈfaɪtɪks/ 是开源的输入法框架，搭载不同的输入法引擎（例如 [[rime|RIME]]）支持不同语言的输入。

## 配置

配置使用可供使用的输入法时，第一个输入法将在 fcitx 处于非激活状态时启用，建议设置为「键盘」。虽然大部分中文输入法都支持通过「Shift」键切换中西文，但这个状态是由输入法引擎管理的，而框架对外部暴露的接口不应当依赖引擎的状态。保留框架级别的中西文切换使我们可以通过命令行程序 `fcitx5-remote` 切换输入法状态。这在使用 [[vim|Vim]] 以及支持 Vim 键位的程序时非常有用。

```bash
fcitx5-remote -c # cancel - 取消输入法激活
fcitx5-remote -o # open - 激活输入法
fcitx5-remote -n # name - 获取当前输入法的名字
fcitx5-remote -s <imname> # switch - 切换到目标名字的输入法
```
