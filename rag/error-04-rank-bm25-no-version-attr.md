# 报错记录：rank_bm25 模块没有 __version__ 属性

## 日期：2026-04-10

**错误描述**：
```
AttributeError: module 'rank_bm25' has no attribute '__version__'
```
注：其余所有 import 均成功，末尾打印了「✅ 所有依赖导入成功」。

**复现步骤**：
1. 安装 `rank-bm25==0.2.2`
2. 运行：
   ```python
   import rank_bm25
   print('rank_bm25 version:', rank_bm25.__version__)
   ```

**环境**：
Windows 11 / Python 3.9 / rank-bm25==0.2.2

**已尝试的修复**：
无需修复，属于无害报错。

**最终解决方式**：
`rank_bm25` 包故意不暴露 `__version__` 属性（作者未添加），功能完全正常。
检查版本改用：
```bash
python -m pip show rank-bm25
```

**学到的教训**（给下次 Claude 用）：
- 并非所有 Python 包都实现了 `__version__`，AttributeError 不代表包安装失败
- 验证包是否可用应测试实际功能（如 `BM25Okapi(['test'])` ），而非检查 `__version__`
- 此报错可忽略，不影响项目运行
