---
title: luanti Lua 编码规范
date: 2026-04-16 16:41:47
categories:
- luanti
tags:
- luanti
- games
- mods
---


# Lua 编码规范
Lua 是一门简洁、极简、优雅、高效、可嵌入、表现力强的脚本语言。想要高效开发 Luanti 模组与游戏，必须熟练掌握 Lua。

<!--more-->

Lua 非常灵活——甚至对很多人来说**过于灵活**。
> Lua 给你能力，你来构建机制。
> ——[《Programming in Lua》第一版](https://www.lua.org/pil)

本文介绍 Luanti 中使用的**机制与编码规范**。

---

## 学习资源
本文不教编程基础或 Lua 基础，推荐以下资源：
- 强烈推荐：**Programming in Lua (PIL)**
  在线版针对 Lua 5.0，但大部分内容适用于 5.1
- lua-users.org 教程
- 常备手册：**Lua 5.1 参考手册**
- Lua 5.1 速查（小抄）

---

# 整数（Integers）
Lua 5.1 **只有一种数字类型**：64 位 IEEE754 浮点数。

除非特别说明，Luanti 只接受**有限数字**：
- 不能是 ±无穷大（`math.huge` / `-math.huge`）
- 不能是 NaN（`0/0`）

满足 `math.floor(number) == number` 的值被视为**整数**。
从 **−2⁵³ 到 2⁵³** 的所有整数都能精确表示，称为**安全整数**，请尽量使用这个范围。
超出后数字不再连续（例如 `2^53 + 1` 无法精确表示）。

⚠️ 重要
Luanti 底层 C++ 经常会把 Lua 数字转为 **32 位有符号整数**，PUC Lua 5.1 本身也会这么做（比如 `math.random`）。
因此建议把数值控制在 **−2³¹ ～ 2³¹−1** 以内更安全。

不要向需要整数的函数传入小数（包括 `math.random`）。
四舍五入行为未定义，请手动用 `math.floor` / `math.ceil` / `math.round`。

---

# 字符串（Strings）
Lua 字符串是**字节串**，理论上可使用任意编码，但 Luanti 的标准是 **UTF‑8**。
处理 Unicode 时默认按 UTF‑8 处理。

注意：
`core.compress` 这类函数**不处理文本**，而是把字符串当作**二进制数据**。
不要混淆这两种用法。

---

# 表（Tables）
表是 Lua 最核心的“万能”数据结构。
表将 **键 → 值** 映射，一组键值叫一个**条目（entry）**。

示例：
```lua
{a = 1, ["b"] = 2, [3] = true}
```
键：`"a"`、`"b"`、`3`
值：`1`、`2`、`true`

---

## 列表（List / Sequence）
从 **1 开始**的连续整数键，**不能有洞（nil 后面跟非 nil）**。

合法：
```lua
{1, 2, 3}
{[1]=1, [2]=2, [3]=3}
```

不合法：
```lua
{nil, 2}
{1, [3] = 3}
```

---

## 集合（Set）
以**键作为元素**、值固定为 `true` 的表。

示例：
```lua
{spam = true, ham = true}
```

下面这不是集合：
```lua
{spam = true, ham = false}
```

---

## 命名空间（Namespace）
以名称为键、存放 API 函数/表的全局表叫**命名空间**。
例如 `core` 就是一个命名空间。

强烈建议：
- 给你的模组 API 加命名空间
- 其他所有变量都用 `local`

示例：
```lua
mymath = {}

function mymath.clamp(x, min, max)
    return math.max(math.min(x, max), min)
end
```

---

## 表结构常见错误
一定要分清**列表**和**单表**。

错误（会被当成空列表）：
```lua
{name = "tex.png", backface_culling = false}
```

正确（是包含一个元素的列表）：
```lua
{
    {name = "tex.png", backface_culling = false}
}
```

---

# 面向对象编程（OOP）
Lua 本身**没有类**，通过**元表（metatable）** 实现基于原型的 OOP。
Luanti 就是这么做的。

通常只使用 `__index` 元方法实现原型继承。
向量是例外：重载了算术运算符。

---

## 构造器
返回类实例的函数，例如：
- `VoxelManip(...)`
- `VoxelArea:new(...)`
- `vector.new(...)`

由于历史原因，构造器写法不统一：
- `Class:new(...)`
- `Class.new(...)`
- `Class(...)`

---

## 实例与方法
实例可以是表或 userdata。
- 实例变量：实例自己的字段
- 方法：以 `instance:method(...)` 调用的函数（语法糖：`instance.method(instance, ...)`）

---

## 类变量（坑点预警）
类变量是**所有实例共享**的，通过 `__index` 查找。
如果类变量是**引用类型（表）**，修改它会**影响所有实例**！

典型坑：Lua 实体
```lua
-- 错误：会修改所有同类型实体的默认纹理！
entity.initial_properties.textures = {"foo.png"}
```

这是最常见的**全局污染式 Bug**。