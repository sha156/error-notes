# 报错记录：HuggingFace 下载 BAAI/bge-m3 时 SSL 证书验证失败（XetHub 后端）

## 日期：2026-04-10

**错误描述**：
```
SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] 
certificate verify failed: unable to get local issuer certificate (_ssl.c:1123)'))
thrown while requesting GET https://cas-bridge.xethub.hf.co/xet-bridge-us/...
```

**复现步骤**：
1. 设置 `HF_ENDPOINT=https://hf-mirror.com`，通过代理 `http://127.0.0.1:7897` 下载 BAAI/bge-m3
2. 模型元数据从 hf-mirror.com 下载成功
3. 实际模型权重文件（pytorch_model.bin）被重定向到 `cas-bridge.xethub.hf.co`（HuggingFace 的 XetHub 存储后端）
4. 代理对 HTTPS 做 SSL 中间人检查，其自签证书不在信任列表，导致验证失败

**环境**：
Windows 11 / Python 3.9 / 代理 `http://127.0.0.1:7897` / huggingface_hub + langchain_community

**已尝试的修复（均无效）**：
- `CURL_CA_BUNDLE=""` + `REQUESTS_CA_BUNDLE=""` — 对 XetHub 下载路径无效
- `ssl._create_default_https_context = ssl._create_unverified_context` — 对 requests 库无效
- `HF_HUB_DISABLE_XET=1` — 该版本 huggingface_hub 不支持此变量，XetHub 仍被启用

**最终解决方式**：
在所有 HuggingFace 相关导入之前，直接猴子补丁 `requests.Session.request`：

```python
import requests
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

_orig_request = requests.Session.request
def _no_verify_request(self, method, url, **kwargs):
    kwargs.setdefault("verify", False)
    return _orig_request(self, method, url, **kwargs)
requests.Session.request = _no_verify_request
```

**学到的教训**（给下次 Claude 用）：
- HuggingFace 新版模型文件走 XetHub（`cas-bridge.xethub.hf.co`）而非 CDN，`HF_ENDPOINT` 只影响 API 元数据请求，无法重定向 XetHub 的实际文件下载
- `REQUESTS_CA_BUNDLE=""` 和 `ssl._create_default_https_context` 对 `requests` 库的 SSL 验证无效，因为 `requests` 通过 `certifi` 独立管理证书
- 必须在 `requests.Session.request` 层打补丁才能全局禁用验证，且必须在 `huggingface_hub` 导入之前执行
- 代理做 SSL 中间人检查时，所有第三方 HTTPS 域名都会受影响，不只是主域名
