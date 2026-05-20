# 报错记录：Flappy Bird 计算机图形学项目 —— Pygame 渲染管线与 AI 系统全部报错汇总

**日期：** 2026-05-20
**仓库：** sha156/error-notes

---

## 错误 1：AI 评分全部为 0（20 个随机种子均失败）

### 错误描述

```
AI 性能测试 —— 运行 20 局游戏
  第 1局: 得分= 0, 存活帧数= 45
  第 2局: 得分= 0, 存活帧数= 52
  ...
  完整存活率: 0/20
```

AI 策略使用多段冲突规则（危险响应 + 振荡对齐），在管道间隙安全区仅 132px 的情况下，鸟振荡幅度 80px 已占 60%，错误相位拍翅导致冲顶或撞底。

### 环境信息

| 项目 | 值 |
|------|-----|
| 操作系统 | Windows 11 Home China 10.0.26200 |
| 语言/运行时 | Python 3.9.13 |
| 工具版本 | Pygame 2.6.1 |
| 特殊环境 | 无 |

### 复现步骤

1. 运行 `python test_ai_performance.py`
2. 观察 AI 在遇到第一个管道时立即碰撞死亡
3. 所有随机种子均得 0 分

### 已尝试的修复

- PD 控制器方案（比例-微分控制）— 失败：对二元离散控制（拍/不拍）不适用，无法微调
- 纯轨迹预测（模拟不拍翅到达管道时的位置）— 失败：无法处理管道已越过小鸟的情况，且未考虑拍翅后振荡

### 最终解决方式

```python
# 间隙中心追踪 (Gap-Center Tracking)
# trigger = gap_center + 50
# 鸟振荡区间 [gap_center-30, gap_center+50] ∈ 安全区 [gap_center-60, gap_center+60]
trigger = next_pipe.gap_y + 50
if bird.y > trigger and bird.vy >= 0:
    self._do_flap()
    return True
```

策略分为四段：硬边界保护（接地/撞顶紧急处理）、管道内部回避、近距离轨迹微调（<70px 帧级模拟）、中距离中心追踪（70~200px）。关键是推导出鸟的自然振荡区间与间隙安全区的数学关系，确保拍翅触发点落在正确相位。

---

## 错误 2：BAT 文件乱码，中文输出全部显示为问号

### 错误描述

```
'Bird' 不是内部或外部命令，也不是可运行的程序
或批处理文件。
```

BAT 文件中使用了 Unicode box-drawing 字符（═、║ 等），Windows 控制台默认 raster 字体不支持这些字符，导致整行乱码。乱码字符被解析器误认为命令，引发连锁报错。

### 复现步骤

1. 双击 `run.bat` 或在 cmd 中运行
2. 看到乱码输出和 "不是内部或外部命令" 错误

### 最终解决方式

将所有 Unicode 特殊字符替换为 ASCII 等效字符：
- `═` → `=`
- `║` → `|`

```batch
echo =============================================
echo   Flappy Bird - AI Auto Play
echo =============================================
```

**教训：Windows 控制台默认使用 raster 字体（代码页 437/936），不支持 Unicode box-drawing 字符。BAT 文件只用 ASCII 字符。**

---

## 错误 3：游戏中文字体显示为方框（tofu）

### 错误描述

游戏 HUD 中所有中文文字显示为空心方框（□□□□），无法阅读。

### 环境信息

| 项目 | 值 |
|------|-----|
| 操作系统 | Windows 11 |
| 语言/运行时 | Python 3.9.13 |
| 工具版本 | Pygame 2.6.1 |

### 复现步骤

1. 运行 `python main.py`
2. 观察左上角 HUD "模式"、"最高分" 等文字显示为方框

### 已尝试的修复

- `pygame.font.SysFont('Arial', 16)` — 失败：Arial 不含中文字形
- `pygame.font.SysFont('SimHei', 16)` — 失败：pygame 的 SysFont 在部分 Windows 上有注册表解析 bug

### 最终解决方式

```python
# 绕过 SysFont，直接用字体文件路径加载
fonts_dir = r'C:\Windows\Fonts'
candidates = ['msyh.ttc', 'simhei.ttf', 'simsun.ttc', 'msjh.ttc']
for name in candidates:
    path = os.path.join(fonts_dir, name)
    if os.path.exists(path):
        return pygame.font.Font(path, size)  # 直接文件加载
```

**教训：Pygame 的 `pygame.font.SysFont()` 在部分 Windows 系统上因注册表枚举 bug（`initsysfonts_win32` 遇到非字符串类型注册表值崩溃）而失败。中文渲染应直接使用 `pygame.font.Font()` 加载字体文件，而不是依赖系统字体枚举。同时需要 `chcp 65001` 切换控制台到 UTF-8。**

---

## 错误 4：Pygame SysFont 报错 "expected str, bytes or os.PathLike object, not int"

### 错误描述

```
File "...\pygame\sysfont.py", line ..., in initsysfonts_win32
    for name in fonts:
TypeError: expected str, bytes or os.PathLike object, not int
```

Pygame 内部在枚举 Windows 字体注册表项时，遇到非字符串类型的注册表值，直接崩溃。

### 最终解决方式

完全不使用 `pygame.font.SysFont()`，全部替换为 `pygame.font.Font(path, size)` 直接加载字体文件。这是错误 3 的根因。

---

## 错误 5：test_ai_performance.py 导入错误

### 错误描述

```
ImportError: cannot import name 'Cloud' from 'game_objects'
AttributeError: 'Pipe' object has no attribute 'width'
NameError: name 'BIRD_SIZE' is not defined
```

### 原因

代码从几何绘制迁移到精灵贴图后，`Cloud` 类被移除、`pipe.width` 改为 `PIPE_WIDTH` 常量、`BIRD_SIZE` 被 `BIRD_SPRITE_W/BIRD_SPRITE_H` 替代。但测试脚本未同步更新。

### 最终解决方式

```python
# 移除: from game_objects import Cloud
# 移除: cloud = Cloud() 相关代码
# 替换: pipe.width → PIPE_WIDTH
# 替换: BIRD_SIZE → bird_rect = bird.get_rect()
```

**教训：修改模块 API 后必须同步更新所有引用文件（test、debug、screenshot），否则遗留脚本会因过期引用而崩溃。建议维护一个 `scripts_to_check.txt` 列表。**

---

## 错误 6：Ground._base_width AttributeError（测试中）

### 错误描述

```
AttributeError: 'NoneType' object has no attribute 'get_width'
```

### 原因

`test_ai_performance.py` 创建 `Ground()` 但未调用 `init_sprites()`，导致模块级 `_BASE_SPRITE` 仍为 `None`。`Ground._base_width` 属性尝试调用 `_BASE_SPRITE.get_width()` 时报错。

### 最终解决方式

AI 性能测试不需要地面渲染，直接将 Ground 相关代码从测试中移除。

**教训：延迟初始化模式（`init_sprites()` 依赖 `display.set_mode()` 先调用）在测试中容易遗漏。测试文件需注意模块的初始化依赖链。**

---

## 错误 7：`from module import CONSTANT` 布尔常量导入不更新

### 错误描述

```python
from game_objects import USE_CG_ENGINE  # 捕获导入时的值 True

# 后续按 F 键切换:
game_objects.USE_CG_ENGINE = False  # 修改了模块变量

# 但 main.py 中:
if USE_CG_ENGINE:  # 仍然是 True！本地绑定未更新
    ...
```

### 原因

Python 中 `from module import name` 将 `name` 绑定到 `module.name` 当前引用的对象。对于不可变的 `bool` 值，后续修改 `module.name` 不会更新本地绑定。

### 最终解决方式

```python
# 不使用 from import，始终通过模块点号访问
import game_objects
if game_objects.USE_CG_ENGINE:  # 每次都读取模块变量
    ...
```

**教训：需要在运行时切换的模块级标志（feature flag、debug 开关等），永远不要用 `from module import FLAG` 导入，必须通过 `module.FLAG` 点号访问以保持动态绑定。**

---

## 错误 8：replace_all 导致双重命名空间 bug

### 错误描述

```python
# 原始代码:
elif event.key == pygame.K_f:
    game_objects.USE_CG_ENGINE = not game_objects.USE_CG_ENGINE

# 执行 replace_all: USE_CG_ENGINE → game_objects.USE_CG_ENGINE 后:
elif event.key == pygame.K_f:
    game_objects.game_objects.USE_CG_ENGINE = ...  # BUG!
```

### 原因

批量替换工具（`Edit` 的 `replace_all`）将文件中所有 `USE_CG_ENGINE` 替换为 `game_objects.USE_CG_ENGINE`，但原代码已包含 `game_objects.` 前缀的部分会被替换为 `game_objects.game_objects.USE_CG_ENGINE`。

### 最终解决方式

手动修正为 `game_objects.USE_CG_ENGINE`。**教训：使用 replace_all 前务必检查是否有已包含目标前缀的字符串。先用 Grep 检查所有出现位置，评估替换安全性。**

---

## 错误 9：Pygame Surface 像素数组 (surfarray) 维度不匹配

### 错误描述

在 `cg_engine.py` 的 `_surface_to_arrays()` 函数中：

```python
rgb = pygame.surfarray.pixels3d(surf)  # shape: (w, h, 3)
# 直接赋值给 rgba[:,:,0] = rgb 会因维度不匹配报错
```

### 原因

- `pygame.surfarray.pixels3d()` 返回 **(width, height, channels)** 的数组
- numpy 图像处理通常期望 **(height, width, channels)** 的布局
- 直接索引会导致行列颠倒

### 最终解决方式

```python
rgba = np.zeros((h, w, 4), dtype=np.float32)
rgba[:, :, 0] = rgb[:, :, 0].T.astype(np.float32)  # .T 转置 (w,h)→(h,w)
rgba[:, :, 1] = rgb[:, :, 1].T.astype(np.float32)
rgba[:, :, 2] = rgb[:, :, 2].T.astype(np.float32)
rgba[:, :, 3] = alpha.T.astype(np.float32)
```

**教训：Pygame 的 `surfarray` 使用 (宽, 高) 坐标顺序（和 Surface 的 (x, y) 一致），而 numpy 数组通常以 (行, 列) = (高, 宽) 组织。两者之间转换必须使用 `.T` 或 `np.transpose` 交换前两个轴。写回 Surface 时同样需要转置回去。**

---

## 学到的教训（给下次 Claude 用）

- **Pygame 中文字体：** 永远不要用 `pygame.font.SysFont()`，直接用 `pygame.font.Font(字体文件路径, size)` 加载 C:\Windows\Fonts\ 下的 .ttc/.ttf 文件
- **BAT 文件编码：** Windows 控制台用 `chcp 65001` 切换 UTF-8，但 box-drawing 等特殊 Unicode 仍不被 raster 字体支持，只用 ASCII
- **Python 模块级 Feature Flag：** 运行时需要切换的开关必须通过 `module.FLAG` 点号访问，不能 `from module import FLAG`
- **replace_all 危险：** 批量替换前用 Grep 查看所有出现位置，检查是否有已包含目标前缀的字符串
- **Pygame surfarray 维度：** `pixels3d()` 返回 (w, h, c)，numpy 操作需要 (h, w, c)，用 `.T` 转置
- **AI 二元控制：** 对离散动作空间的物理模拟，简单的间隙中心追踪比复杂的多段规则或 PD 控制器更鲁棒
- **延迟初始化模式：** 测试脚本需注意模块的初始化依赖链（如 `init_sprites()` 必须在 `display.set_mode()` 之后）
- **API 迁移：** 修改模块 API（如 Cloud 类删除、pipe.width → PIPE_WIDTH）后必须同步更新所有引用文件

---

<!-- Tags: pygame / python / surfarray / 中文字体 / SysFont / BAT 编码 / import 陷阱 / 布尔常量 / AI 策略 / numpy 维度 -->
