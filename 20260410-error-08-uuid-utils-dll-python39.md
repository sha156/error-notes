# 报错记录：uuid-utils DLL 加载失败（Python 3.9 与新版 langchain-core 不兼容）

## 日期：2026-04-10

**错误描述**：
```
ImportError: DLL load failed while importing _uuid_utils: 找不到指定的程序。

Traceback:
  langchain_experimental/text_splitter.py → langchain_core.documents →
  langchain_core.runnables.config → langchain_core.callbacks.manager →
  langchain_core.utils.uuid → uuid_utils.compat → uuid_utils._uuid_utils
  ImportError: DLL load failed while importing _uuid_utils
```

**复现步骤**：
1. 环境：Python 3.9，`langchain-core==0.3.84`，`uuid-utils==0.14.1`
2. 运行任何使用 `langchain_experimental` 或新版 `langchain_core` 的 import
3. `uuid_utils 0.14.1` 包含针对 Python 3.10+ 编译的 `.pyd`，在 Python 3.9 上 DLL 找不到入口

**环境**：
Windows 11 / Python 3.9.x / `uuid-utils==0.14.1`（langchain-core 0.3.84 自动安装）

**已尝试的修复**：
- 重新安装 `uuid-utils`（同版本，仍然失败）

**最终解决方式**：
降级到 Python 3.9 兼容的旧版本：
```bash
python -m pip install "uuid-utils==0.6.1" \
  -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
```
注意：pip 会提示 `langchain-core 0.3.84 requires uuid-utils>=0.12.0` 的警告，但实际运行正常。

**学到的教训**（给下次 Claude 用）：
- `uuid-utils` 从某版本起放弃了对 Python 3.9 的 wheel 支持，但 langchain-core 新版依赖它
- Python 3.9 + langchain 0.3.x 生态会有此问题，降级 `uuid-utils` 到 `0.6.1` 可临时解决
- 长远方案：将项目 venv 升级到 Python 3.10+，避免此类 DLL 兼容问题
