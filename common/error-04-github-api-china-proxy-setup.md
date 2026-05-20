# 报错记录：GitHub API 国内环境访问失败 — hosts 劫持 + 502 Bad Gateway

**日期：** 2026-05-20
**仓库：** sha156/error-notes（common 通用环境类）

---

## 错误描述

在国内 Windows 11 环境下，`gh auth login` 和所有 GitHub API 请求均返回 `HTTP 502 Bad Gateway`，但 `git@github.com` SSH 连接正常。

---

## 环境信息

| 项目 | 值 |
|------|-----|
| 操作系统 | Windows 11 中国版 |
| 工具 | gh CLI 2.x、git 2.x |
| 代理工具 | SteamCommunity 302 V14（Caddy 反代，监听 127.0.0.1:443） |
| 本地代理 | 127.0.0.1:7897（Clash / 类似工具） |
| 关键问题 | hosts 文件将 *.github.com 全部指向 127.0.0.1 |

---

## 复现步骤

1. 运行 `gh auth login --hostname github.com --web`
2. 输出：`failed to authenticate via web browser: HTTP 502`
3. 运行 `gh auth login --with-token`
4. 输出：`error validating token: HTTP 502`
5. 运行 `curl -I https://api.github.com/`
6. 响应头显示 `Server: Caddy`，状态 `HTTP 502`

---

## 排查过程

### 尝试 1：直接访问 GitHub API
```bash
curl -I https://api.github.com/
```
→ 返回 502，Server 头显示 `Caddy`（非 GitHub 服务器），说明流量被本地劫持

### 尝试 2：检查 hosts 文件
发现 `C:\Windows\System32\drivers\etc\hosts` 中所有 GitHub 域名被重定向到 `127.0.0.1`：
```
127.0.0.1 github.com #S302
127.0.0.1 api.github.com #S302
127.0.0.1 *.github.com #S302
```
→ SteamCommunity 302 反代工具所为，但其 Caddy 上游代理对 GitHub API 不可达

### 尝试 3：SSH 连接测试
```bash
ssh -T git@github.com
```
→ 成功认证为 sha156！原因：`~/.ssh/config` 配置了 `Hostname ssh.github.com`，绕过 hosts 劫持

### 尝试 4：通过代理绕过 hosts 劫持
```bash
export HTTPS_PROXY=http://127.0.0.1:7897 HTTP_PROXY=http://127.0.0.1:7897
curl -I https://api.github.com/
```
→ 返回 `HTTP 200 OK`，Server 头显示 `github.com`！代理直连成功

### 尝试 5：Token 权限问题
使用 Fine-grained PAT 认证后，读取仓库 OK，写入返回 403：
```
Resource not accessible by personal access token
```
→ Fine-grained PAT 需单独设置 "Contents: Read and write" 权限。改用 Classic PAT 勾选 `repo` scope 即可

---

## 最终解决方式

```bash
# 1. 设置代理环境变量（绕过 hosts 劫持）
export HTTPS_PROXY=http://127.0.0.1:7897
export HTTP_PROXY=http://127.0.0.1:7897

# 2. 用 Classic Token 代替 Fine-grained Token
# 在 https://github.com/settings/tokens 创建 Classic token，勾选 repo 权限

# 3. 通过代理认证 gh CLI
echo "ghp_xxx" | gh auth login --with-token

# 4. gh CLI 使用 SSH 协议操作 git
gh config set git_protocol ssh --host github.com
```

**为什么有效：**
- 代理（Clash）在 7897 端口直接代理流量，DNS 解析在代理端进行，不受本机 hosts 劫持
- Classic PAT 的 `repo` scope 覆盖所有仓库读写操作，不像 Fine-grained PAT 需逐项配置权限
- SSH 流量走 `ssh.github.com:443`，不受 hosts 劫持（由 `~/.ssh/config` 控制）

---

## 学到的教训（给下次 Claude 用）

- 国内环境所有 GitHub API 请求必须走 `HTTPS_PROXY=http://127.0.0.1:7897`，hosts 劫持会拦截直连
- 排查顺序：curl 测试 → 检查 hosts 文件 → 检查代理进程 → 检查 token 权限
- Fine-grained PAT ≠ Classic PAT：前者需按 Permission 逐项授权，`Contents` 默认只读
- SteamCommunity 302 的 Caddy 反代对 GitHub 不稳定，代理直连更可靠
- SSH 不受 hosts 影响（`~/.ssh/config` 指定了 `ssh.github.com` 而非 `github.com`）

---

<!-- Tags: GitHub API, 502, hosts劫持, Caddy, SteamCommunity 302, 代理, gh CLI, PAT, Fine-grained token -->
