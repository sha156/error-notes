# 报错记录：GitHub SSH 认证失败（未配置公钥）

## 日期：2026-04-10

**错误描述**：
```
git@ssh.github.com: Permission denied (publickey).
fatal: Could not read from remote repository.
```

**复现步骤**：
1. 运行 `git clone git@github.com:sha156/error-notes.git`
2. ~/.ssh/ 目录下没有 ed25519 或 rsa 密钥对

**环境**：
Windows 11 / Git Bash / 首次配置 GitHub SSH

**已尝试的修复**：
- 直接 clone（无密钥，必然失败）

**最终解决方式**：
1. 生成密钥：`ssh-keygen -t ed25519 -C "error-notes-deploy" -f ~/.ssh/id_ed25519 -N ""`
2. 复制公钥内容到 GitHub → Settings → SSH Keys → New SSH Key
3. 验证：`ssh -T git@github.com` → 显示 "Hi sha156! You've successfully authenticated"

**学到的教训**（给下次 Claude 用）：
- 新机器首次使用 GitHub SSH 必须先生成密钥并添加公钥
- `ssh -T git@github.com` 是验证连通性最快的方式，exit code 1 但输出 "Hi xxx" 表示成功
- Windows 下密钥存在 `C:/Users/<用户>/.ssh/`，不是 `/root/.ssh/`
