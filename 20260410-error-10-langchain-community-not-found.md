# 报错记录：ModuleNotFoundError: No module named 'langchain_community'

## 日期：2026-04-10

**错误描述**：
```
Traceback (most recent call last):
  File "D:\Project\py\RAG\ingest.py", line 29, in <module>
    from langchain_community.document_loaders import PyMuPDFLoader
ModuleNotFoundError: No module named 'langchain_community'
```

**复现步骤**：
1. 项目目录下存在 `.venv` 虚拟环境，已安装 `langchain-community`
2. 在 PowerShell 中直接运行 `python ingest.py`（未激活虚拟环境）
3. 系统 Python 找不到 `langchain_community`，报错

**环境**：
Windows 11 / Python 3.9 / PowerShell / 项目路径：`D:\Project\py\RAG`

**已尝试的修复**：
- 尝试用 `! .venv/Scripts/python.exe ingest.py`（bash 语法，PowerShell 不支持 `!`，报解析错误）

**最终解决方式**：
PowerShell 中使用虚拟环境的 Python 直接运行：
```powershell
.venv\Scripts\python.exe ingest.py
```
或先激活虚拟环境：
```powershell
.venv\Scripts\Activate.ps1
python ingest.py
```

**学到的教训**（给下次 Claude 用）：
- Windows PowerShell 不支持 `!` 前缀运行命令（那是 bash/zsh 语法）
- 虚拟环境未激活时，`python` 指向系统 Python，不会找到 `.venv` 里的包
- PowerShell 激活虚拟环境用 `Activate.ps1`，不是 `activate`（bash 用法）
