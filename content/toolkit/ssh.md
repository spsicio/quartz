---
title: SSH
created: 2025-10-11
modified: 2025-11-21
---

## 密钥

通过储存公钥，`known_hosts` 用于客户端验证服务器的身份。相对应的，`authorized_keys` 用于服务器验证客户端的身份。

```bash
# 搜集网站的公钥
ssh-keyscan github.com >> ~/.ssh/known_hosts
ssh-keyscan localhost >> ~/.ssh/known_hosts

# 查看公钥的指纹
ssh-keygen -lf ~/.ssh/known_hosts | grep github.com
ssh-keygen -lf ~/.ssh/known_hosts | grep localhost

# 清除网站的公钥
ssh-keygen -R github.com
ssh-keygen -R localhost
```

使用 mDNS 之后，在局域网中可以直接使用 `ssh user@host.local` 连接主机。
