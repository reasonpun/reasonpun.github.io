---
title: luanti API LuaJIT
date: 2026-04-16 16:33:11
categories:
- luanti
tags:
- luanti
- games
- mods
- API
---

# LuaJIT
LuaJIT 是一款面向 Lua 的**即时编译器（JIT）**，它完整实现了 **Lua 5.1 标准**（为区分起见，官方标准 Lua 称为 PUC Lua），同时附带一些扩展与差异。
LuaJIT 运行速度通常远快于 PUC Lua，因此**强烈推荐使用**。

除非你编译 Luanti 时**手动禁用了 LuaJIT**，否则 Luanti 默认都会用 LuaJIT 作为 Lua 运行时。
官方 Windows、Android 以及大多数 Linux 发行版安装包，均已启用 LuaJIT。
你可以在终端执行这条命令确认：
```
./luanti --version
```

<!--more-->

## 为什么有人会用 PUC Lua 而不是 LuaJIT
- 追求极致稳定性
- 运行在**非常冷门的架构**上，暂时没有 LuaJIT 支持

因此你写模组时**必须同时兼容两者**，否则会出现：在 LuaJIT 上跑得好好的，放到 PUC Lua 直接崩溃。

---

## 查看运行时版本示例
使用 LuaJIT 时输出类似：
```
$ ./luanti --version
[...]
Using LuaJIT 2.1.1702233742
```

使用 PUC Lua 时输出类似：
```
$ ./luanti --version
[...]
Using Lua 5.1.5
```

---

## 编译说明
从源码编译时：
- 系统检测到 LuaJIT → **自动启用**（通常需要安装开发包）
- 没找到 LuaJIT 或显式设置 `-DENABLE_LUAJIT=FALSE` → 回退到内置的 **PUC Lua 5.1**

---

# LuaJIT 关键差异
## 1. 表的遍历顺序
Lua 官方手册**不保证** `pairs` / `next` 的遍历顺序，任何实现都可能按任意顺序访问。
**永远不要依赖固定遍历顺序**。

LuaJIT 与 PUC Lua 内部实现不同，遍历顺序通常不一样。
特别地：**LuaJIT 出于安全，哈希使用随机种子**，每次启动遍历顺序都可能不同。

---

## 2. goto 语句
LuaJIT 支持 `goto` 与标签语法（语言扩展）。
**PUC Lua 不支持**。
→ **你的模组绝对不要用 goto**。

---

## 3. 十六进制转义字符
LuaJIT 支持字符串里的十六进制转义：
```lua
"\x41"  → 等于 "A"
```
但 **PUC Lua 会直接忽略反斜杠**，不报错：
```lua
"\x41"  → 变成字面 "x41"
```
**正确跨平台写法**：用 PUC Lua 支持的**三位十进制转义**：
```lua
"\065" → 等于 "A"
```

---

## 4. xpcall()
LuaJIT 允许给被调用函数 `f` 传额外参数：
```lua
xpcall(print, err_func, "hello world")
```
→ `print` 能收到 `"hello world"`。

但 **PUC Lua 会直接忽略这些参数**。
→ 写跨平台模组时不要依赖此特性。

---

## 5. math.log()
LuaJIT 支持**指定底数**：
```lua
math.log(x, base)
```
PUC Lua **只支持自然对数**，多余参数会被忽略。

**跨平台兼容写法**：
```lua
math.log(x) / math.log(base)
```

---

## 6. math.random()
### （1）随机数生成器不同
LuaJIT 自带伪随机数发生器（PRNG）。
PUC Lua 直接用系统提供的 PRNG。
→ **同种子、不同运行时，结果完全不一样**。

⚠️ **重要**
为保证地图生成（mapgen）跨平台可复现，
**地图生成代码绝对禁止使用 math.random()**。

### （2）参数顺序宽容度不同
LuaJIT 很宽容：`math.random(3,1)` 会自动交换 min/max。
PUC Lua 直接报错：
```
bad argument #2 to 'random' (interval is empty)
```

→ 你必须自己保证：**最小值 ≤ 最大值**。

---

## 更多扩展
LuaJIT 提供的完整语言扩展列表可在官方查看：
[LuaJIT.org Extensions](https://luajit.org/extensions.html)

---

# 旧版 Luanti 兼容说明
从 **Luanti 5.5.0** 开始，即使使用 PUC Lua 编译，也会自带 `bit` 位运算库。
→ 面向 5.5+ 的模组可以放心用 `bit`。

LuaJIT 一直自带 `bit`，
但 **Luanti 5.4 及更早版本不保证可用**。
