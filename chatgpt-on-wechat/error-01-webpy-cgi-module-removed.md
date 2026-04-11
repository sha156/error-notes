# error-01-webpy-cgi-module-removed.md

## 描述

安装 `chatgpt-on-wechat` 时，`pip install -r requirements.txt` 失败。`web.py 0.62` 依赖 Python 的 `cgi` 模块，该模块在 Python 3.13+ 中已被彻底移除。

## 环境

- OS：Windows 11
- Python：3.14.3
- web.py：0.62

## 完整报错

```
ERROR: Failed to build 'web.py' when getting requirements to build wheel

ModuleNotFoundError: No module named 'cgi'

  File "web/webapi.py", line 6, in <module>
    import cgi
  File "web/webapi.py", line 19, in <module>
    from . import webapi as web
  File "web/__init__.py", line 4, in <module>
    from . import (...)
```

## 根本原因

`web/webapi.py` 第 6 行 `import cgi`，第 391 行继承 `cgi.FieldStorage`。`cgi` 模块在 Python 3.11 被标记为废弃，Python 3.13 正式移除。

## 已尝试（无效）

- 直接 `pip install web.py`：同样报错，源码构建阶段就失败
- `pip install web.py --no-build-isolation`：同样失败

## 解决方案

从 pip 缓存中找到已编译的 wheel 文件直接安装，绕过构建阶段：

```bash
# 1. 找到缓存的 wheel
python -m pip cache list   # 确认有 web_py-0.62-py3-none-any.whl

# 2. 找到路径
find ~/AppData/Local/pip/cache -name "web_py*.whl"

# 3. 直接安装 wheel
python -m pip install <path>/web_py-0.62-py3-none-any.whl
```

然后手动修改已安装的 `web/webapi.py`：

**第 6 行**：把 `import cgi` 替换为：

```python
import email.parser
import urllib.parse
```

**第 391–405 行**：把 `cgiFieldStorage(cgi.FieldStorage)` 替换为不依赖 `cgi` 的纯标准库实现（用 `urllib.parse.parse_qsl` 解析 GET/POST，用 `email.parser` 解析 multipart headers）。

**`rawinput` 中的 `dictify`**：移除 `if fs.list is None` 判断（新实现的 list 永远不为 None）。

## 相关 Issue

已提交至上游：[zhayujie/chatgpt-on-wechat#2757](https://github.com/zhayujie/chatgpt-on-wechat/issues/2757)

## 教训

- Python 3.13+ 移除了多个历史模块（`cgi`、`cgitb`、`imghdr` 等），安装旧项目前先确认 Python 版本兼容性
- pip 缓存中的 wheel 可以绕过构建失败，是临时修复的有效手段
- 上游安装脚本写死了 Python 3.9–3.13，遇到版本拒绝时要看清楚原因而非只升级 Python
