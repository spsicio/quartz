---
title: SSH
created: 2025-10-11
modified: 2025-10-11
---

## 密钥

```bash
# 搜集网站的公钥
ssh-keyscan github.com >> ~/.ssh/known_hosts

# 查看公钥的指纹
ssh-keygen -lf ~/.ssh/known_hosts | grep github.com

# 清除网站的公钥
ssh-keygen -R github.com
```
