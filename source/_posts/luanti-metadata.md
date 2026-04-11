---
title: luanti API-Classes metadata
date: 2026-04-11 11:41:31
categories:
- luanti
tags:
- luanti
- games
- mods
- API
---


# MetaData
MetaData 是一个**通用键值存储接口**，用于给对象持久化保存字符串数据。
下面所有方法，在它的**所有子类**里都可以用。

## 子类（它管哪些东西）
- **ModStorage**：模组全局存储（整个模组共用）
- **NodeMetaData**：地图上的单个方块存储
- **ItemStackMetaData**：物品堆存储
- **PlayerMetaData**：玩家存储

<!--more-->

---

## 核心注意
- 所有值**最终都会存成字符串**，没有类型标记
- 你自己要记住存的是字符串、整数还是浮点数
- **不要混用类型**，否则会出bug
- 读取时会自动解析，但不保证完全符合你的预期

### 推荐：存复杂数据用序列化
- 存 JSON 表格：`core.write_json` / `core.parse_json`
- 存任意 Lua 表格：`core.serialize` / `core.deserialize`

这样存数字、表格**绝对安全**，不怕类型丢失。

---

# 获取类方法（Getters）
所有 get 方法只传一个参数：**key（键名）**

### `:contains(key)`
检查某个键**是否存在**
返回：
- `nil`：对象无效
- `false`：不存在
- `true`：存在

### `:get(key)`
获取键的值（原始方法）
返回：
- `nil`：不存在
- 字符串：存在的值

### `:get_string(key)`
获取字符串，**不存在返回空字符串 ""**

### `:get_int(key)`
获取整数，**不存在返回 0**
自动把字符串转成数字

### `:get_float(key)`
获取浮点数，**不存在返回 0**

---

# 设置类方法（Setters）
所有 set 方法传两个参数：**key, value**

### `:set_string(key, value)`
存字符串
- 传 `""` 等于**删除这个键**

### `:set_int(key, value)`
存整数
⚠️ **警告**：
内部是 C++ 32位 int，范围：
**-2³¹ ~ 2³¹-1**
超出会溢出出错！

### `:set_float(key, value)`
存浮点数
⚠️ **警告**：
内部是 C++ 32位 float，**精度比 Lua 64位低**，大数可能丢失精度。

---

# 其他实用方法
### `:equals(other)`
比较两个元数据对象**是否完全一样**（键值全相同）

### `:to_table()`
把所有元数据**转成 Lua 表格**
结构：
```lua
{
    fields = { 键 = 值, ... }
}
```

### `:from_table(table)`
从表格**批量加载**元数据
- 传非表格 = **清空所有数据**
