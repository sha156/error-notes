# error-01-gh-cli-not-found-and-406.md

## 描述

GitHub CLI (`gh`) 未安装导致命令找不到；安装后登录时返回 HTTP 406，原因是本地代理未被 `gh` 使用。

## 环境

- OS：Windows 11
- 代理：Clash，监听 `127.0.0.1:7897`

## 完整报错

**阶段一：gh 未安装**
```
/usr/bin/bash: line 1: gh: command not found
```

**阶段二：安装后 PowerShell 找不到**
```
gh : 无法将"gh"项识别为 cmdlet、函数、脚本文件或可运行程序的名称
```

**阶段三：安装后登录 406**
```
error validating token: HTTP 406: 406 Not Acceptable (https://github.com/)

! First copy your one-time code: A103-8AD3
Open this URL to continue in your web browser: https://github.com/login/device
failed to authenticate via web browser: non-200 OK status code: 406 Not Acceptable body: ""
```

## 复现步骤

```bash
# bash 环境直接调用
gh auth status

# PowerShell 调用（安装后但未刷新 PATH）
powershell -Command "gh auth status"

# 安装后登录，未设置代理
"/c/Program Files/GitHub CLI/gh.exe" auth login
```

## 根本原因

1. `gh` 未预装，bash 环境的 PATH 不含 Windows 安装路径
2. `winget` 安装后需新开终端才能识别，当前 session PATH 未更新
3. `gh` 默认不读取 Windows 系统代理，`api.github.com` 被本地代理解析到 `127.0.0.1`，但 `gh` 未走代理端口，导致 406

## 解决方案

**安装 gh：**
```powershell
winget install --id GitHub.cli -e --source winget
```

**用完整路径调用（不依赖 PATH）：**
```bash
"/c/Program Files/GitHub CLI/gh.exe" auth status
```

**设置代理环境变量后再登录：**
```bash
export HTTPS_PROXY="http://127.0.0.1:7897"
export HTTP_PROXY="http://127.0.0.1:7897"
echo "your_token" | "/c/Program Files/GitHub CLI/gh.exe" auth login --with-token
```

**查询系统代理端口：**
```powershell
Get-ItemProperty -Path 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings' | Select-Object ProxyEnable, ProxyServer
```

## 教训

- Windows 上 `winget` 安装的工具当前 bash session 不会自动识别，需用完整路径
- 国内有代理时，`gh` 必须手动设置 `HTTPS_PROXY`，否则一律 406
- 每次调用 `gh` 都要带上代理变量，或写入 shell profile
