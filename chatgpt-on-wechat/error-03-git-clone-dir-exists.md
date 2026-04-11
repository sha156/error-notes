# error-03-git-clone-dir-exists.md

## 描述

执行 `git clone` 时，目标目录 `chatgpt-on-wechat` 已存在且非空，克隆失败。

## 环境

- OS：Windows 11
- Git：2.47.1.windows.1

## 完整报错

```
fatal: destination path 'chatgpt-on-wechat' already exists and is not an empty directory.
```

## 复现步骤

```bash
git clone https://github.com/zhayujie/chatgpt-on-wechat.git
# 再次执行同一命令
git clone https://github.com/zhayujie/chatgpt-on-wechat.git
```

## 解决方案

目录已存在说明之前已克隆过，直接进入使用即可：

```bash
ls chatgpt-on-wechat/   # 确认内容完整
cd chatgpt-on-wechat
```

若需要重新克隆：

```bash
rm -rf chatgpt-on-wechat
git clone https://github.com/zhayujie/chatgpt-on-wechat.git
```

## 教训

- 克隆前先检查目录是否已存在，避免重复克隆
- 报这个错不是真正的失败，目录存在即代表已克隆成功过
