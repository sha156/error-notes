# 报错记录：langchain-openai 与 langchain-core 版本不兼容

## 日期：2026-04-10

**错误描述**：
```
ERROR: pip's dependency resolver does not currently take into account all the packages that are installed.
langchain-openai 0.1.6 requires langchain-core<0.2.0,>=0.1.46,
but you have langchain-core 0.3.84 which is incompatible.
```

**复现步骤**：
1. 项目原始环境：`langchain-openai==0.1.6`，`langchain-core==0.1.53`
2. 安装 `langchain-experimental`，pip 自动将 `langchain-core` 升级到 `0.3.84`
3. `langchain-openai 0.1.6` 要求 `langchain-core<0.2.0`，产生冲突

**环境**：
Windows 11 / Python 3.9 / `.venv` 虚拟环境 / langchain 生态链版本混用

**已尝试的修复**：
- 直接安装 `langchain-experimental`（触发 core 升级，导致冲突）

**最终解决方式**：
同步升级 `langchain-openai` 到兼容新 core 的版本：
```bash
python -m pip install "langchain-openai>=0.2.0" "langchain-core>=0.3.0" \
  -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
```
升级后：`langchain-openai==0.3.35`，`langchain-core==0.3.84`，冲突消除。

**学到的教训**（给下次 Claude 用）：
- langchain 生态版本迭代很快，0.1.x / 0.2.x / 0.3.x 不互相兼容
- 安装任何 `langchain-*` 子包前先确认当前 core 版本，整体一起升级比单独升级更安全
- 发现 incompatible 警告后立即检查 `langchain-openai` 版本，统一升到最新
