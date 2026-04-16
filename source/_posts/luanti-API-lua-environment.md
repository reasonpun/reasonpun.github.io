---
title: luanti 运行环境
date: 2026-04-16 16:29:19
categories:
- luanti
tags:
- luanti
- games
- mods
- API
---

# Lua 运行环境

Luanti 采用 **Lua 5.1**。模组的实际运行环境取决于**四大要素**：
- 操作系统
- Luanti 构建版本：LuaJIT（默认）或 PUC Lua 5.1
- 模组类型：客户端侧（CSM）或服务端侧（SSM）
- 模组环境：安全模式 或 不安全模式

<!--more-->

---

## 平台无关性 / 可移植性
Platform Independence / Portability
与平台/操作系统相关的 Lua 特性请查阅 [Lua 5.1 参考手册](https://www.lua.org/manual/5.1/)，包括：
- 区域设置（Locale）：会影响模式匹配（字符类）及 `string.char`/`string.byte` 使用的字符编码
- `os` 库大部分功能，尤其是 `os.execute`（**仅不安全环境可用**）
- `io` 库部分功能，如 `io.popen`（**仅不安全环境可用**）或二进制文件处理
- `require` 与 `package.loadlib`（**无论如何都仅不安全环境可用**）

---

## 全局严格模式
Global Strictness
Lua 变量**默认全局**（赋值与访问都是），很容易踩坑，最常见两大问题：
1. 本地变量拼错，变成访问全局变量（通常是 `nil`）
2. 忘写 `local`，意外覆盖全局变量 → **全局污染**

Luanti 内置严格模式通过给全局表加元表实现，两种情况都会打警告：
- 读取**未声明**全局变量 → 警告：*未声明全局变量访问*
- 加载后赋值**未声明**全局变量 → 警告：*向未声明全局赋值*

声明规则：**主代码块里的全局赋值才算声明**，函数里赋值不算。
```lua
-- 符合 Luanti 定义的声明：
global_var = 42
for k, v in pairs{a = 1, b = 2} do _G[k] = v end

-- 属于“未声明全局赋值”，会报警告：
local function set_global()
	another_global_var = 42
end
set_global()
```

警告会按 `debug.getinfo` 返回的位置（`short_src` + `currentline`）去重，**不会重复打印**。

⚠️ **警告**
访问未声明全局变量会**慢一个数量级**，因为要跑严格模式检查代码。

💡 **小贴士**
这些警告只在运行时触发。建议开发时用 `luacheck` 这类**静态检查工具**提前抓错。

---

## 检查全局变量是否存在
Checking for global existence
做模组兼容时必须判断全局是否存在。直接写 `if name then ... end`，如果变量是 `nil` 且未声明，会触发“未声明全局访问”警告（常见于可选依赖未加载）。

Luanti 用元表实现全局严格模式，因此可以用：
- `rawget(_G, name)`：代替直接写 `name`，安全访问可能为 `nil` 的全局**不报警告**
- `rawset(_G, name, value)`：运行时安全设置全局

### `core.global_exists(name)`
给定名字的全局变量存在（非 `nil`）返回 `true`，否则 `false`；`name` 不是字符串会抛错。

📌 **注**
底层就是封装 `rawget(_G, name)`，但**意图更清晰、可读性更高**。

---

## 标准库扩展
Standard Library Extensions
📌 **注意**
**不建议自己扩展 Lua 标准库**！容易和其他模组冲突，也可能和未来引擎更新（包括 Lua 版本升级）打架。请把扩展放进独立 API 表，别改 Lua 内置库。

### `math` 库扩展
- `math.hypot(x, y)`
  按勾股定理求斜边长度：`z = √(x² + y²)`，等价简写 `math.sqrt(x*x + y*y)`。

- `math.sign(x, [tolerance])`
  `tolerance` 为空/假则默认 `0`。
  - 小于容差 → 返回 `-1`
  - 大于容差 → 返回 `1`
  - 在 `[-tolerance, tolerance]` 之间 → 返回 `0`
  - `x` 是 `nan` → 返回 `0`

- `math.factorial(x)`
  `x` 必须是非负整数，否则抛错：`"factorial expects a non-negative integer"`。
  `x ≥ 171` 时返回 `+inf`。

  因为浮点数精度限制，**1~22 精确**，23 开始不准。

- `math.round(x)`
  四舍五入到最近整数。
  - 中点情况（`x = k + 0.5`）：**远离0取整**
  - 极端接近中点的数（±0.49999999999999994）会被误当中点处理
  - 保留 `nan`/`inf` 及其符号

### `string` 库扩展
也可通过字符串元表用 `self:func(...)` 调用。
⚠️ Lua 语法不允许 `"str":func(...)`，必须加括号：`("str"):func(...)`。

- `string.trim(self)`
  去掉字符串**首尾连续空白**（`%s` 匹配类），包括：
  `\t`、`\n`、`\v`、`\f`、`\r`、空格。

  ⚠️ **不保证跨平台一致**：字母、空白等字符组定义取决于当前区域设置——[Lua 5.1 参考手册 5.4.1](https://www.lua.org/manual/5.1/manual.html#5.4.1)

- `string.split(str, [delim], [include_empty], [max_splits], [sep_is_pattern])`
  - `str`：待切分字符串
  - `delim`：分隔符，假值默认 `,`
  - `include_empty`：真值则保留空串 `""`
  - `max_splits`：最大切分次数，结果最多 `max_splits+1` 项；假/负/`nan`/`inf` 表示不限
  - `sep_is_pattern`：真值则把 `delim` 当模式使用
  返回不含分隔符的分段列表。

### `table` 库扩展
- `table.indexof(list, val)`
  线性查找 `val`，返回**第一个匹配下标**，找不到返回 `-1`。

- `table.copy(t, [seen])`
  **深度拷贝**表与子表（键值都拷）；非表类型（userdata/函数/协程）不拷贝。
  引用结构**完全保留**：同一张表无论被引用多少次，只拷贝一次，后续引用指向同一份副本。
  `seen` 用于记录已拷贝表，避免重复拷贝与循环引用。

- `table.insert_all(t, other)`
  把 `other` 的数组项**全部追加**到 `t`（数组合并）。

- `table.key_value_swap(t)`
  返回新表：键值互换。值重复时，对应键不确定。

- `table.shuffle(t, [from], [to], [random])`
  对数组指定区间做 **Fisher-Yates 洗牌**。
  - `from`：起始下标（默认开头）
  - `to`：结束下标（默认结尾）
  - `random`：随机函数 `function(from,to)`，默认 `math.random`

---

## LuaJIT 扩展
LuaJIT extensions
启用 LuaJIT 编译的 Luanti（`ENABLE_LUAJIT=1`）会提供 LuaJIT 扩展，包括 Lua 5.2 语法（如 `goto`），在 PUC Lua 5.1 上会报语法错。
十六进制转义（如 `"\xFF"`）在 LuaJIT 正常解析，在 PUC Lua 5.1 会被当成字面 `"xFF"`。

### 通用扩展
LuaJIT 的 `bit` 库在 PUC Lua 与 LuaJIT 构建中**均可用**，**不要写 `require("bit")`**：
- 安全环境下会崩溃
- 不安全环境下没必要

---

## 安全环境白名单
Secure environment whitelists
安全环境中，以下 Lua/JIT 内置库与函数**白名单可用**：
- `_VERSION`
- 垃圾回收：`collectgarbage`
- 协程：`coroutine`
- 错误处理：`assert`、`error`、`pcall`、`xpcall`
- 函数环境：`setfenv`、`getfenv`
- `math`、`string`、`table`
- 迭代：`next`、`pairs`、`ipairs`
- 元表：`setmetatable`、`getmetatable`（仅 SSM）
- 原始操作：`rawset`、`rawget`、`rawequals`
- 变长参数：`select`、`unpack`
- 类型转换：`tostring`、`tonumber`、`type`
- 输出：`print`

### 受限库（白名单内函数）
- `io`（仅 SSM）：`read`、`write`、`flush`、`close`、`type`
- `os`（ mostly 时间）：`clock`、`date`、`difftime`、`time`
  仅 SSM：`getenv`、`setlocale`（仅限查询区域）
- `debug`：`gethook`、`traceback`
  仅 SSM：`getinfo`、`getmetatable`、`setmetatable`、`upvalueid`、`sethook`
- `package`（仅 SSM）：`config`、`cpath`、`path`、`searchpath`
- `jit`（仅 LuaJIT）：`arch`、`flush`、`off`、`on`、`opt`、`os`、`status`、`version`、`version_num`

### 文件相关（安全重写）
所有文件操作被替换为安全版本：
- 加载 Lua 代码：若为字节码（以 `\27` 开头），报错：`"Bytecode prohibited when mod security is enabled."`
- CSM 的文件操作基于虚拟路径，仅能访问 CSM 文件
- 禁用函数：
  - `require` → 报错：`"require() is disabled when mod security is on."`
  - `dofile`、`load`、`loadfile`、`loadstring` 受安全管控

### 受限文件权限（仅部分 SSM 可用）
加载期间：
- 只读访问所有模组目录
- 只允许写**当前正在加载的模组目录**（句柄可保存后使用）
- 可读写世界目录（排除 `worldmods`、`game` 子目录）

可用函数：
- `io`：`open`、`input`、`output`、`lines`
- `os`：`remove`、`rename`

内置库在加载期间可全局读写。

Lua 标准库文档见 [Lua 5.1 Reference Manual](https://www.lua.org/manual/5.1/)。

---

## 不安全环境说明
- 关闭模组安全时，服务端模组运行在**不安全环境**：**无任何限制**
- 受信任服务端模组可通过 `core.request_insecure_environment` 获取不安全环境（表形式，库表浅拷贝，无全局限制）
