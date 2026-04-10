# 报错记录：pip install 直连 PyPI 时 SSL 握手失败

## 日期：2026-04-10

**错误描述**：
```
SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1123)'))
Could not fetch URL https://pypi.org/simple/agent-reach/
```

**复现步骤**：
1. 运行 `pip install agent-reach`
2. pip 尝试直连 pypi.org，SSL 握手被中断

**环境**：
Windows 11 / Python 3.9（Miniconda）/ 系统设置了代理 http://127.0.0.1:7897 但 pip 未使用

**已尝试的修复**：
- `pip install agent-reach --trusted-host pypi.org`（仍然失败，代理未生效）
- 换用清华镜像 `-i https://pypi.tuna.tsinghua.edu.cn/simple`（同样被代理拦截）

**最终解决方式**：
显式指定代理参数：
```bash
pip install --proxy http://127.0.0.1:7897 "git+https://github.com/..."
```

**学到的教训**（给下次 Claude 用）：
- Windows 下系统代理不会自动传递给 pip，必须用 `--proxy` 参数或设置 `HTTP_PROXY` 环境变量
- `--trusted-host` 只绕过证书验证，不解决代理问题
