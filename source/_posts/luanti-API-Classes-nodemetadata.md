---
title: luanti API-Classes NodeMetaData
date: 2026-04-11 11:46:11
categories:
- luanti
tags:
- luanti
- games
- mods
- API
---

# NodeMetaData
NodeMetaData 让你可以给**世界里的每一个方块**绑定独立的持久化键值数据，完全继承 MetaData 所有方法。
它最常用的功能是给箱子、熔炉等方块**存储背包与界面**。

## 传统方块只能存这些（非常少）
- 方块ID（名称）
- param1：通常用于光照（0–255）
- param2：旋转、颜色、形状等（0–255）

有了 NodeMetaData，你想存多少就存多少。

<!--more-->

---

## 获取方块元数据
### `core.get_meta(pos)`
获取指定位置方块的元数据对象。
- 参数：`pos` 坐标（会自动取整）
- 返回：NodeMetaData 对象

### `core.find_nodes_with_meta(pos1, pos2)`
在一个长方体区域内**搜索所有带有元数据的方块**。
- 参数：区域两个对角坐标
- 返回：带元数据的位置列表

---

## 特殊字段（引擎自带功能）
### `formspec`
右键点开方块时显示的界面。
- 如果方块定义了 `on_rightclick`，这个字段**不生效**
- 优点：**客户端秒开**，不用等服务器往返（RTT）
- 缺点：界面基本是静态的，不能动态生成
- 只有这里能用 `context`  inventory 位置

### `infotext`
准星指向方块时显示的文字（比如箱子里有什么）。
- 自动换行、超长自动截断

---

## 方法
### `:mark_as_private(keys)`
把字段标记为**私有**，**不发给客户端**，防作弊、减流量。
- 可以传单个 key 或 key 列表
- 删除键后标记会消失
- 重新创建键必须重新标记
- `to_table` / `from_table` **不保留私有标记**

### `:get_inventory()`
获取这个方块绑定的**背包**（箱子、熔炉等）。

### `:to_table()`
在 MetaData 基础上**额外包含 inventory**，导出所有背包物品。

### `:from_table(table)`
从表格批量恢复**元数据 + 背包**。
