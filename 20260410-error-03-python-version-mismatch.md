# 报错记录：agent-reach 要求 Python >=3.10，系统默认是 3.9

## 日期：2026-04-10

**错误描述**：
```
ERROR: Package 'agent-reach' requires a different Python: 3.9.1 not in '>=3.10'
```

**复现步骤**：
1. 运行 `pip install --proxy ... "git+https://github.com/Panniantong/Agent-Reach.git"`
2. pip 对应的是 Miniconda Python 3.9

**环境**：
Windows 11 / Miniconda Python 3.9 / 同时装有 Python 3.11、3.13、3.14

**已尝试的修复**：
- 直接 pip install（版本不符）

**最终解决方式**：
指定 Python 3.11 的 pip：
```bash
"C:/Users/Administrator/AppData/Local/Programs/Python/Python311/python.exe" -m pip install --proxy http://127.0.0.1:7897 "git+https://github.com/Panniantong/Agent-Reach.git"
```

**学到的教训**（给下次 Claude 用）：
- Windows 多版本 Python 共存时，`pip` 命令对应的版本不一定是 `python` 命令的版本
- 遇到版本不符，先用 `py -0` 或 `where python` 列出所有可用版本，再选合适的解释器
- 用 `<python_path> -m pip install` 替代直接 `pip install` 可以精确控制版本
