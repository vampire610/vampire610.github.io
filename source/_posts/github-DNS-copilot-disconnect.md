---
title: Copilot 失效与 GitHub DNS 污染排查
date: 2025-08-16 12:03:05
tags: 
  - github
  - DNS
  - vscode copilot
categories: 
  - github
---



# Copilot 失效与 GitHub DNS 污染排查

> 记录一次 VS Code Copilot 及 GitHub 无法访问的排查与解决过程。

近期在 VS Code 使用 Copilot 时，发现无论在 WSL 还是 Windows 环境下均无法正常工作，且 GitHub 也无法访问。初步怀疑为 DNS 问题，尝试手动设置 IPv4 DNS（本机和路由器端）均无效。

通过 `ping github.com` 发现域名被解析为 `127.0.0.1`。进一步检索后，有网友提及 IPv6 可能影响 DNS 解析。关闭 IPv6 后恢复正常，重新开启则问题复现。

由此判断 DNS 解析优先走了 IPv6。更换为可靠的 IPv6 DNS 后，问题解决。

**推荐 DNS 设置：**

IPv4 DNS：

```
114.114.114.114   # 国内通用
8.8.8.8           # Google
```

IPv6 DNS：

```
2400:3200::1           # 阿里云
2001:4860:4860::8888   # Google
```

如遇 GitHub 无法访问、Copilot 失效等问题，建议优先检查 DNS，尤其是 IPv6 配置。
