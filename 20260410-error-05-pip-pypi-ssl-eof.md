# 报错记录：pip 直连 PyPI 安装 rank-bm25 时 SSL EOF 失败

## 日期：2026-04-10

**错误描述**：
```
SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1123)'))
Could not fetch URL https://pypi.org/simple/rank-bm25/
ERROR: Could not find a version that satisfies the requirement rank-bm25
ERROR: No matching distribution found for rank-bm25
```

**复现步骤**：
1. 在 `D:/Project/py/RAG/.venv` 虚拟环境中运行：
   ```bash
   python -m pip install rank-bm25
   ```
2. pip 尝试直连 pypi.org，SSL 握手在 EOF 处中断

**环境**：
Windows 11 / Python 3.9（`.venv` 虚拟环境）/ 系统代理 http://127.0.0.1:7897 未被 pip 继承

**已尝试的修复**：
- `pip install rank-bm25`（直连失败）
- `pip install rank-bm25 -q --index-url https://pypi.tuna.tsinghua.edu.cn/simple`（清华镜像，被代理拦截，见 error-06）

**最终解决方式**：
改用阿里云镜像并加 `--trusted-host`：
```bash
python -m pip install rank-bm25 -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
```

**学到的教训**（给下次 Claude 用）：
- 该环境下 pypi.org 和清华镜像（https）均不可达，阿里云镜像（http）可用
- 优先尝试 `http://mirrors.aliyun.com/pypi/simple/` + `--trusted-host mirrors.aliyun.com`
- 如果 pip 无输出（exit code 1），通常是网络层被拦截，不要误以为命令本身有问题
