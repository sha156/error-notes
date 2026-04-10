# 报错记录：agent-reach 未发布到 PyPI

## 日期：2026-04-10

**错误描述**：
```
ERROR: Could not find a version that satisfies the requirement agent-reach (from versions: none)
ERROR: No matching distribution found for agent-reach
```

**复现步骤**：
1. 运行 `pip install agent-reach`

**环境**：
Windows 11 / pip 25.3

**已尝试的修复**：
- 直接 pip install（404）
- 换镜像源（仍然 404）

**最终解决方式**：
从 GitHub 源码安装：
```bash
pip install --proxy http://127.0.0.1:7897 "git+https://github.com/Panniantong/Agent-Reach.git"
```

**学到的教训**（给下次 Claude 用）：
- 新项目 / 个人项目经常只有 GitHub 源码，没有发布到 PyPI
- 遇到 "No matching distribution" 先去 GitHub 确认是否有 pyproject.toml，再用 git+ 方式安装
