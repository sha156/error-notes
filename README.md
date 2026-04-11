# error-notes

个人报错记录库。按项目分类，每个文件对应一个真实报错，包含复现步骤、已尝试的修复和最终解决方式。

## 目录结构

```
error-notes/
├── agent-reach/          # Agent Reach 项目相关报错
├── rag/                  # 战锤40K RAG 项目相关报错
├── chatgpt-on-wechat/    # chatgpt-on-wechat 项目相关报错
└── common/               # 通用环境 / 工具报错（pip、git、SSH 等）
```

## 文件命名规则

```
error-NN-简短描述.md   （每个项目内从 01 开始）
```

---

## agent-reach/

| 编号 | 文件 | 描述 |
|------|------|------|
| 01 | [error-01-pypi-ssl-failed.md](agent-reach/error-01-pypi-ssl-failed.md) | pip 直连 PyPI 时 SSL 握手失败 |
| 02 | [error-02-not-on-pypi.md](agent-reach/error-02-not-on-pypi.md) | agent-reach 未发布到 PyPI，需从 GitHub 安装 |
| 03 | [error-03-python-version-mismatch.md](agent-reach/error-03-python-version-mismatch.md) | agent-reach 要求 Python >=3.10，系统默认是 3.9 |

## rag/

| 编号 | 文件 | 描述 |
|------|------|------|
| 01 | [error-01-pip-ssl-eof.md](rag/error-01-pip-ssl-eof.md) | pip 安装 rank-bm25 时 SSL EOF 失败 |
| 02 | [error-02-langchain-version-conflict.md](rag/error-02-langchain-version-conflict.md) | langchain-openai 与 langchain-core 版本不兼容 |
| 03 | [error-03-uuid-utils-dll-python39.md](rag/error-03-uuid-utils-dll-python39.md) | uuid-utils DLL 加载失败（Python 3.9 与新版 langchain-core 不兼容） |
| 04 | [error-04-rank-bm25-no-version-attr.md](rag/error-04-rank-bm25-no-version-attr.md) | rank_bm25 模块没有 `__version__` 属性（无害，可忽略） |
| 05 | [error-05-langchain-community-not-found.md](rag/error-05-langchain-community-not-found.md) | `ModuleNotFoundError: No module named 'langchain_community'`（未激活虚拟环境） |
| 06 | [error-06-hf-xethub-ssl-cert-failed.md](rag/error-06-hf-xethub-ssl-cert-failed.md) | HuggingFace 下载 BAAI/bge-m3 时 SSL 证书验证失败（XetHub 后端） |

## chatgpt-on-wechat/

| 编号 | 文件 | 描述 |
|------|------|------|
| 01 | [error-01-webpy-cgi-module-removed.md](chatgpt-on-wechat/error-01-webpy-cgi-module-removed.md) | web.py 0.62 依赖已移除的 `cgi` 模块，Python 3.13+ 安装失败 |

## common/

| 编号 | 文件 | 描述 |
|------|------|------|
| 01 | [error-01-github-ssh-permission-denied.md](common/error-01-github-ssh-permission-denied.md) | GitHub SSH 认证失败（未配置公钥） |
| 02 | [error-02-pip-tsinghua-proxy-error.md](common/error-02-pip-tsinghua-proxy-error.md) | 清华镜像被代理拦截 ProxyError |

---

## 每个文件包含

- **错误描述** — 完整报错信息
- **复现步骤** — 如何触发
- **环境** — OS / Python 版本 / 相关工具版本
- **已尝试的修复** — 无效方案，避免重复踩坑
- **最终解决方式** — 有效命令或代码
- **学到的教训** — 给下次参考的结论
