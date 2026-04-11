# error-02-installer-python-version-blocked.md

## 描述

运行官方一键安装脚本时，因系统 Python 为 3.14.3，脚本硬编码只允许 3.9–3.13，直接退出，无法安装。

## 环境

- OS：Windows 11
- Python：3.14.3

## 完整报错

```
=========================================
   CowAgent Installation (Windows)
=========================================

Python 3.9-3.13 not found. Please install from https://www.python.org/downloads/
Exit code 1
```

## 复现步骤

```powershell
powershell -Command "irm https://cdn.link-ai.tech/code/cow/run.ps1 | iex"
```

## 根本原因

安装脚本 `run.ps1` 对 Python 版本做了硬编码检测，3.14 不在允许范围内，直接 `exit 1`，不给任何绕过方式。

## 已尝试（无效）

- 直接运行脚本：被版本检测拦截

## 解决方案

跳过安装脚本，手动克隆仓库并安装依赖：

```bash
git clone https://github.com/zhayujie/chatgpt-on-wechat.git
cd chatgpt-on-wechat
python -m pip install -r requirements.txt
python -m pip install -e .
```

## 教训

- 官方安装脚本有版本白名单，Python 版本过新时要绕过脚本手动安装
- 遇到版本拒绝先看脚本源码，确认是版本检测还是真正不兼容
