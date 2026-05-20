# 报错记录：Python 3.14 移除 cgi 模块 — legacy-cgi 安装与适配过程

**日期：** 2026-04-12
**仓库：** sha156/chatgpt-on-wechat（扩展关联 sha156/error-notes）

---

## 错误描述

Python 3.14 移除了内置 `cgi` 模块（PEP 594），chatgpt-on-wechat 依赖的 web.py 0.62 在 `import cgi` 时报 `ModuleNotFoundError`。官方推荐安装 `legacy-cgi` 作为替代，但安装过程中经历了多次失败。

---

## 环境信息

| 项目 | 值 |
|------|-----|
| 操作系统 | Windows 11 |
| 语言/运行时 | Python 3.14.3 |
| 工具 | pip（系统内置）、Claude Code |

---

## 复现步骤 / 尝试过程

### 尝试 1：直接 pip install
```bash
pip install legacy-cgi
```
→ 失败：SSL/TLS 握手超时（未走代理）

### 尝试 2：切换阿里云镜像
```bash
pip install legacy-cgi -i https://mirrors.aliyun.com/pypi/simple/
```
→ 失败：阿里云镜像未收录该包或连接超时

### 尝试 3：跳过 HTTPS 验证
```bash
pip install legacy-cgi --trusted-host pypi.org --trusted-host files.pythonhosted.org
```
→ 失败：`--trusted-host` 只绕过证书验证，不解决网络不可达问题

### 尝试 4：验证 cgi 模块确实被移除
```bash
python -c "import cgi; print('cgi module OK, from:', cgi.__file__)"
```
→ 失败：`ModuleNotFoundError: No module named 'cgi'` — 确认 Python 3.14 内置 cgi 已被移除

### 尝试 5：走代理安装
```bash
set HTTPS_PROXY=http://127.0.0.1:7897
pip install legacy-cgi
```
→ 成功：通过本地代理访问 PyPI 完成安装

---

## 最终解决方式

```bash
# 核心操作
set HTTPS_PROXY=http://127.0.0.1:7897 & pip install legacy-cgi
```

安装后 Python 3.14 中 `import cgi` 替换为：
```python
import legacy_cgi as cgi
```

根本原因：国内网络直连 PyPI 被阻断，镜像是绕过方案但并非所有包都有镜像。设置代理走 7897 端口（SteamCommunity 302 / Clash 等工具）是最稳定的解决方案。

---

## 学到的教训（给下次 Claude 用）

- Python 3.14+ 没有 `cgi` 模块，永远先检查 `legacy-cgi` 是否已安装
- 国内环境所有 pip 操作应预设 `HTTPS_PROXY=http://127.0.0.1:7897`
- `--trusted-host` ≠ 网络可达，它只是跳过 TLS 证书校验
- 镜像源不保证收录所有包（尤其是小众包），代理 > 镜像
- 故障排查顺序：先确认网络连通性 → 再确认包是否存在 → 最后检查版本兼容

---

<!-- Tags: Python 3.14, cgi, legacy-cgi, pip, proxy, chatgpt-on-wechat -->