---
title: luanti API-Classes 射线检测（Raycast）
date: 2026-04-11 11:50:17
categories:
- luanti
tags:
- luanti
- games
- mods
- API
---

# 射线检测（Raycast）
射线用于模拟一条“线”，检测它碰到了哪些**选择箱**（selection box）。
效果和玩家准星指向几乎完全一样（有少量限制）。

## 指向结果（pointed_thing）格式
射线碰到的东西只有 3 种：
1. **nothing**：没碰到任何东西
2. **object**：实体、生物、玩家
3. **node**：方块（固体/液体）

<!--more-->

### 对象（实体/玩家）
```lua
{
    type = "object",
    ref = ObjectRef  -- 引用
}
```

### 方块
```lua
{
    type = "node",
    under = 被指向的方块位置,
    above = 方块上方放置位置
}
```

### 射线特有的额外字段
- `box_id`：碰到的选择箱编号（从 1 开始）
- `intersection_normal`：接触面法线向量
- `intersection_point`：精确碰撞点坐标

**射线不会返回 nothing，没碰到就直接返回 nil。**

---

## `core.line_of_sight(pos1, pos2)`
**检查两点之间是否有非空气方块遮挡视线**
适合做生物视野判断、能否看到目标。

返回：
- `free_sight`：是否无遮挡
- `pos`：第一个遮挡方块的位置

---

## `Raycast(起点, 终点, 检测对象?, 检测液体?)`
创建射线（等价于 `core.raycast`）

参数：
- `from_pos`：起点（通常是眼睛位置）
- `to_pos`：终点
- `include_objects`：是否检测实体/玩家（默认 true）
- `include_liquids`：是否检测液体（默认 true）

返回：
**可迭代的射线对象**（一次性、不可重启）

---

## 重要说明
- 射线检测 **选择箱**，不是碰撞箱
- 服务端射线**不能正确处理挂载/模型动画**
- 射线迭代**只能用一次**，用完就“耗尽”了
- 可以中途 break，下次循环会**继续上次位置**

---

## 使用方法（最简单）
```lua
local ray = Raycast(pos1, pos2, true, true)

for pointed_thing in ray do
    -- 处理碰到的东西
end
```

---

## 实用示例：重现玩家手里工具的射线
```lua
local eye_pos = player:get_pos()
eye_pos.y = eye_pos.y + player:get_properties().eye_height

-- 眼睛偏移
local first = player:get_eye_offset()
eye_pos = vector.add(eye_pos, vector.divide(first, 10))

-- 方向与距离
local look_dir = player:get_look_dir()
local range = player:get_wielded_item():get_definition().range or 4
local end_pos = vector.add(eye_pos, vector.multiply(look_dir, range))

-- 射线
for pt in core.raycast(eye_pos, end_pos, true, true) do
    if pt.ref ~= player then
        -- 你看到了 pt
    end
end
```