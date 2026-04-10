# 报错记录：清华镜像被代理拦截 ProxyError

## 日期：2026-04-10

**错误描述**：
```
ProxyError('Cannot connect to proxy.', FileNotFoundError(2, 'No such file or directory'))
ERROR: Could not find a version that satisfies the requirement rank-bm25
ERROR: No matching distribution found for rank-bm25
```

**复现步骤**：
1. 运行：
   ```bash
   python -m pip install rank-bm25 langchain-experimental zhipuai \
     -i https://pypi.tuna.tsinghua.edu.cn/simple \
     --trusted-host pypi.tuna.tsinghua.edu.cn
   ```
2. pip 检测到系统代理配置（http://127.0.0.1:7897），尝试通过代理连接清华镜像
3. 代理进程此时未运行，导致 FileNotFoundError

**环境**：
Windows 11 / Python 3.9 / 系统代理 127.0.0.1:7897 已关闭或不可用

**已尝试的修复**：
- 加 `--trusted-host`（仅绕过证书，不解决代理连接问题）

**最终解决方式**：
改用 http 协议的阿里云镜像，pip 不会对 http 地址走系统代理：
```bash
python -m pip install rank-bm25 \
  -i http://mirrors.aliyun.com/pypi/simple/ \
  --trusted-host mirrors.aliyun.com
```

**学到的教训**（给下次 Claude 用）：
- pip 在 Windows 下会自动读取系统代理设置（INTERNET OPTIONS），https 镜像会触发代理
- 代理不可用时用 http 镜像可以完全绕开代理逻辑
- 清华镜像（https）在该机器上不可靠，优先用阿里云 http 镜像
