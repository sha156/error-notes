# error-notes

个人报错记录库。每个文件对应一个真实报错，包含复现步骤、已尝试的修复和最终解决方式。

## 文件命名规则

```
YYYYMMDD-error-NN-简短描述.md
```

## 记录索引

### 2026-04-10

| 编号 | 文件 | 描述 |
|------|------|------|
| 01 | [20260410-error-01-pypi-ssl-failed.md](20260410-error-01-pypi-ssl-failed.md) | pip 直连 PyPI 时 SSL 握手失败 |
| 02 | [20260410-error-02-agent-reach-not-on-pypi.md](20260410-error-02-agent-reach-not-on-pypi.md) | agent-reach 未发布到 PyPI |
| 03 | [20260410-error-03-python-version-mismatch.md](20260410-error-03-python-version-mismatch.md) | agent-reach 要求 Python >=3.10，系统默认是 3.9 |
| 04 | [20260410-error-04-github-ssh-permission-denied.md](20260410-error-04-github-ssh-permission-denied.md) | GitHub SSH 认证失败（未配置公钥） |
| 05 | [20260410-error-05-pip-pypi-ssl-eof.md](20260410-error-05-pip-pypi-ssl-eof.md) | pip 直连 PyPI 安装 rank-bm25 时 SSL EOF 失败 |
| 06 | [20260410-error-06-pip-tsinghua-proxy-error.md](20260410-error-06-pip-tsinghua-proxy-error.md) | 清华镜像被代理拦截 ProxyError |
| 07 | [20260410-error-07-langchain-version-conflict.md](20260410-error-07-langchain-version-conflict.md) | langchain-openai 与 langchain-core 版本不兼容 |
| 08 | [20260410-error-08-uuid-utils-dll-python39.md](20260410-error-08-uuid-utils-dll-python39.md) | uuid-utils DLL 加载失败（Python 3.9 与新版 langchain-core 不兼容） |
| 09 | [20260410-error-09-rank-bm25-no-version-attr.md](20260410-error-09-rank-bm25-no-version-attr.md) | rank_bm25 模块没有 `__version__` 属性 |
| 10 | [20260410-error-10-langchain-community-not-found.md](20260410-error-10-langchain-community-not-found.md) | `ModuleNotFoundError: No module named 'langchain_community'`（未激活虚拟环境） |
| 11 | [20260410-error-11-hf-xethub-ssl-cert-failed.md](20260410-error-11-hf-xethub-ssl-cert-failed.md) | HuggingFace 下载 BAAI/bge-m3 时 SSL 证书验证失败（XetHub 后端） |

## 每个文件包含

- **错误描述** — 完整报错信息
- **复现步骤** — 如何触发
- **环境** — OS / Python 版本 / 相关工具版本
- **已尝试的修复** — 无效方案，避免重复踩坑
- **最终解决方式** — 有效命令或代码
- **学到的教训** — 给下次参考的结论
