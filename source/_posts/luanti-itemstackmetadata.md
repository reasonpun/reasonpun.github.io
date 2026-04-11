---
title: luanti API-Classes ItemStackMetaData
date: 2026-04-11 11:39:16
categories:
- luanti
tags:
- luanti
- games
- mods
- API
---

# ItemStackMetaData
ItemStackMetaData 是 **MetaData 的子类**，通过 `stack:get_meta()` 获取，用于**给物品堆绑定持久化的键值对数据**。

<!--more-->

---

## 重要警告
物品堆元数据会和物品一起序列化，会**增加物品名称字符串长度**。
攻击者可能故意把元数据撑到最大，导致背包无法正常传输。
**请始终限制物品元数据的大小**，保证背包能正常发送。

---

## 特殊字段
### 描述文字
- `description`：鼠标悬停时显示的物品描述（对应 `ItemStack:get_description`）
- `short_description`：物品简短描述（对应 `ItemStack:get_short_description`）

### 硬件着色
- `color`：用于物品硬件着色的颜色字符串
- `palette_index`：物品硬件着色使用的调色板索引（如果物品有调色板）

### 数量显示
需要 Luanti 5.6+ 客户端支持（低版本客户端会显示默认数量）；服务端全 5.x 版本兼容。

- `count_meta`：在背包中显示的**自定义数量文字**，覆盖真实数量
- `count_alignment`：数量文字对齐方式（用 `:get_int` / `:set_int`）

对齐值计算规则：
`对齐值 = x_align + 4 * y_align`

| 数值 | X 轴对齐（水平） | Y 轴对齐（垂直） |
|------|------------------|------------------|
| 0 | 默认（同 3） | 默认（同 3） |
| 1 | 左 | 上 |
| 2 | 居中 | 居中 |
| 3 | 右 | 下 |

### 小提示
直接写数字会降低可读性，建议加注释或封装成函数：
```lua
-- 右上角对齐
local align_top_right = 3 + 4 * 1
meta:set_int("count_alignment", align_top_right)
```

---

## 方法
### `:set_tool_capabilities(tool_capabilities)`
允许**覆盖物品定义里的工具属性**。

参数：
- `nil`：清除工具属性覆盖，恢复默认
- 工具属性表：覆盖原有工具属性（格式见 ItemDefinition）

注意：
对应的获取方法 `:get_tool_capabilities` **不属于 ItemStackMetaData**，而是属于父级 ItemStack。
