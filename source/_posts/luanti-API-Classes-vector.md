---
title: luanti API-Classes 向量 API
date: 2026-04-11 11:51:26
categories:
- luanti
tags:
- luanti
- games
- mods
- API
---

# 向量 API
Vector（向量）过去只是带 `x`/`y`/`z` 的普通表，现在已升级为带元表的增强类型。
除非特别说明，本文提到的 `vector` 均指**带元表的向量**。


## vector 命名空间函数

### `vector.new(a, b, c)`
创建新向量。
- `a`、`b`、`c` 为数字 → 返回 `{x=a, y=b, z=c}`
- `a` 是向量 → 等价 `vector.copy(a)`
- 全空 → 等价 `vector.zero()`

<!--more-->

### `vector.zero()`
返回零向量：`{x=0, y=0, z=0}`

### `vector.copy(v)`
深拷贝一个向量，并正确设置元表。

### `vector.from_string(s, init)`
从字符串解析向量，格式：`(x, y, z)`
返回：**向量对象**、**下一个字符位置**

### `vector.to_string(v)`
把向量转成字符串：`(x, y, z)`

### `vector.equals(a, b)`
判断两个向量是否完全相等。

### `vector.length(v)`
返回向量长度（模）：√(x²+y²+z²)

### `vector.normalize(v)`
单位化向量（长度变为 1），长度为 0 时返回零向量。

### `vector.floor(v)`
对 x/y/z 分别向下取整。

### `vector.round(v)`
对 x/y/z 分别四舍五入。

### `vector.apply(v, func)`
对 x/y/z 分别应用函数 `func`。

### `vector.combine(v, w, func)`
对两个向量的 x/y/z 分别应用函数。

### `vector.distance(a, b)`
返回 a 和 b 两点之间的距离。

### `vector.direction(pos1, pos2)`
返回从 pos1 指向 pos2 的**单位方向向量**。

### `vector.angle(a, b)`
返回两个向量之间的夹角（弧度）。

### `vector.dot(a, b)`
向量点积。

### `vector.cross(a, b)`
向量叉积，返回新向量。

---

## 数学运算

### `vector.add(a, b)`
- b 是向量 → 对应分量相加
- b 是数字 → 所有分量加 b

### `vector.subtract(a, b)`
- b 是向量 → 对应分量相减
- b 是数字 → 所有分量减 b

### `vector.multiply(a, b)`
- b 是向量 → 分量相乘
- b 是数字 → 所有分量乘 b

### `vector.divide(a, b)`
- b 是向量 → 分量相除
- b 是数字 → 所有分量除以 b

### `vector.offset(v, x, y, z)`
给向量 v 分别加上 x/y/z 偏移量。

### `vector.sort(a, b)`
返回两个新向量：
1. 每个分量取较小值
2. 每个分量取较大值

常用于获取区域最小/最大角点。

### `vector.check(v)`
检查是否为合法的增强向量。

### `vector.rotate_around_axis(v, axis, angle)`
让向量 v 绕 axis 轴逆时针旋转 angle（弧度）。

### `vector.rotate(v, rot)`
按 rot 旋转向量：
- rot.x = pitch（俯仰）
- rot.y = yaw（偏航）
- rot.z = roll（滚转）

### `vector.dir_to_rotation(forward, up)`
把方向向量转为旋转量（ you 看哪里 → 旋转值）。
up 默认是 `{x=0,y=1,z=0}`。

---

## 元表操作（最常用！）
增强向量支持**直接数学运算**，代码更干净：

```lua
local a = vector.new(1,2,3)
local b = vector.new(4,5,6)

local c = a + b      -- 相加
local d = a - b      -- 相减
local e = a * 2      -- 乘数字
local f = a / 2      -- 除数字
local g = -a         -- 取反
```

支持：
- `==` → `vector.equals`
- `+` `-` `*` `/`
- 负号 `-v`

⚠️ 注意：
元表运算**不支持向量+数字**，只能用 `vector.add`。
