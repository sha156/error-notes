# 报错记录：streamlit 环境缺少 flashrank 模块

## 日期：2026-04-11

**错误描述**：
```
File "D:\Project\py\RAG\app.py", line 32, in <module>
    from flashrank import Ranker, RerankRequest
ModuleNotFoundError: No module named 'flashrank'
```

**复现步骤**：
1. `app.py` 依赖 `flashrank` 做 rerank
2. miniconda 全局环境中未安装该包
3. 运行 `streamlit run app.py` 时崩溃

**环境**：
Windows 11 / Python 3.9（miniconda）/ flashrank 未安装

**已尝试的修复**：
- 直接运行（未提前安装依赖）

**最终解决方式**：
```bash
NO_PROXY="*" HTTP_PROXY="" HTTPS_PROXY="" \
  c:/programdata/miniconda3/Scripts/pip install flashrank \
  -i https://pypi.tuna.tsinghua.edu.cn/simple \
  --trusted-host pypi.tuna.tsinghua.edu.cn
```
安装了 flashrank==0.2.10 及依赖 onnxruntime==1.19.2

**学到的教训**：
- miniconda 全局环境和项目 `.venv` 是独立的，`.venv` 里装了的包不代表系统 Python 也有
- streamlit 默认用系统 Python 启动，若想使用项目 `.venv` 的包，需用 `.venv/Scripts/streamlit run app.py`
