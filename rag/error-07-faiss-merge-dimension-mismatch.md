# 报错记录：FAISS merge_from 索引维度不匹配

## 日期：2026-04-11

**错误描述**：
```
File ".venv\lib\site-packages\faiss\swigfaiss_avx2.py", line 2661, in merge_from
    return _swigfaiss_avx2.IndexFlatCodes_merge_from(self, otherIndex, add_id)
RuntimeError: Error in void __cdecl faiss::IndexFlatCodes::check_compatible_for_merge(
    const struct faiss::Index &) const at faiss\IndexFlatCodes.cpp:89:
    Error: 'other->d == d' failed
```

**复现步骤**：
1. 曾用某个嵌入模型构建过 `local_vector_store/index.faiss`
2. 更换或重装嵌入模型（BAAI/bge-m3，1024维）后，再次运行 `python ingest.py`（增量合并模式）
3. 新索引与旧索引维度不一致，`merge_from` 抛出异常

**环境**：
Windows 11 / Python 3.9（`.venv` 虚拟环境）/ faiss-cpu / BAAI/bge-m3 嵌入模型

**已尝试的修复**：
- 直接运行 `python ingest.py`（增量合并，失败）

**最终解决方式**：
删除旧索引，使用 `--rebuild` 参数重建：
```bash
python ingest.py --rebuild
# 或手动删除后正常运行
rm -rf local_vector_store/
python ingest.py
```

**学到的教训**：
- 换了嵌入模型或嵌入维度有变化后，旧的 FAISS 索引必须删除重建，不能增量合并
- 增量合并前应检查新旧模型维度是否一致（`index.d` 属性）
- 如果报 `'other->d == d' failed`，直接 `--rebuild`，不要浪费时间排查其他原因
