# 报错记录：Python 3.9 不支持 X | Y 联合类型注解语法

## 日期：2026-04-11

**错误描述**：
```
File "D:\Project\py\RAG\app.py", line 153, in <module>
    filter_books: list[str] | None = None,
TypeError: unsupported operand type(s) for |: 'types.GenericAlias' and 'NoneType'
```
以及随后的：
```
File "D:\Project\py\RAG\app.py", line 23
    from __future__ import annotations
    ^
SyntaxError: from __future__ imports must occur at the beginning of the file
```

**复现步骤**：
1. `app.py` 使用了 `list[str] | None`、`dict[str, float]`、`tuple[str, str, float, list[str] | None]` 等 Python 3.10+ 语法
2. miniconda 使用 Python 3.9，不支持运行时求值该语法
3. 第一次修复时将 `from __future__ import annotations` 插入到普通 import 语句之后，触发第二个语法错误

**环境**：
Windows 11 / Python 3.9（miniconda）

**已尝试的修复**：
- 在文件中部插入 `from __future__ import annotations`（位置错误，触发 SyntaxError）

**最终解决方式**：
将 `from __future__ import annotations` 置于文件最顶部，紧跟模块 docstring 之后、所有其他 import 之前：
```python
"""
app.py — ...
"""

from __future__ import annotations

import os
...
```
此方式让 Python 将所有类型注解作为字符串延迟求值，无需逐行修改注解语法。

**学到的教训**：
- `from __future__ import annotations` 必须是文件中第一条非 docstring 语句
- 该方式可一次性兼容所有 3.10+ 类型注解语法，无需逐行改为 `Optional[X]` / `Union[X, Y]`
- Python 3.9 支持 `list[str]`（lowercase generic）作为类型注解，但 `|` 联合语法在运行时不支持，加 `__future__` 后均可用
