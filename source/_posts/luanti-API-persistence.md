---
title: luanti API 数据持久化
date: 2026-04-16 16:37:04
categories:
- luanti
tags:
- luanti
- games
- mods
- API
---

# 数据持久化
本页概览 Luanti 提供的**序列化与数据持久化方案**。

选择持久化方式主要看这些：
- 数据结构与类型：数据长啥样？有哪些类型？
- 数据大小与修改频率：能不能塞进内存？更新开销多大？
- 所需粒度：数据需要多久存一次？
- 查询需求：是否需要索引快速查找？
- 数据绑定对象：物品？玩家？方块？

<!--more-->

## 序列化与反序列化
### JSON
Luanti 基于 **jsoncpp** 提供 JSON 序列化。

JSON 支持的类型：
- 布尔值
- 数字
- 字符串
- 表（对象/数组）

📌 注意
正负无穷大会用远超双精度范围的数字表示（±`1e9999`）。

⚠️ 警告
空表 `{}`（歧义：可能是 `{}` 或 `[]`）和 `nan` **不被 JSON 支持**，会被转成 `null`。

表必须满足以下其一：
- 只包含正整数键 → 解析为数组
- 只包含字符串键 → 解析为对象/字典

很多 Lua 表无法直接表示（布尔键、混合键、小数键、无穷大键等）。

---

### `core.write_json(data, [styled])`
- 数据可序列化 → 返回对应 JSON 字符串
- `styled` 为真 → 格式化输出，方便阅读
- 失败 → 返回 `nil` + 错误信息

常见错误：
- `Can't use indexes with fractional part in JSON`
- `Lua key to convert to JSON is not a string or number`
- `Can't mix array and object values in JSON`

⚠️ 警告
纯整数键的哈希表（比如 `core.hash_node_position` 的哈希表）会被转成**带空洞的数组**，1 到最大索引之间的 `nil` 会全被填成 `null`，**性能爆炸、文件暴大**。
不要随便暴露 `core.write_json`，很容易被用来**DoS 服务器**。

💡 小贴士
稀疏哈希表序列化 JSON 时，**务必转成字符串键的对象**。

⚠️ 警告
JSON 序列化**最大递归深度 16**，超深结构无法序列化。
循环引用会直接爆栈。

💡 小贴士
推荐写法：`json = assert(core.write_json(data))`，避免静默失败。

---

### `core.parse_json(json, [nullvalue])`
- 合法 JSON → 返回对应 Lua 值，`null` 转为 `nullvalue`（默认 `nil`）
- 不合法 → 返回 `nil` 并打印错误

💡 小贴士
必须用 JSON 时（比如对接网页），可以用纯 Lua 库如 `lunajson`。

---

### Lua 原生序列化
Luanti 提供**序列化到 Lua 代码**的方案：
`core.serialize` 和 `core.deserialize`。

支持：
- `nil`
- 布尔
- 数字
- 字符串
- 表（**支持循环引用**）
- 函数（不推荐）

不支持：
- 用户数据（如 `ItemStack`）
- 协程（线程）

#### 函数序列化坑：
- 内部用 `string.dump`
- 模组安全开启时反序列化函数会报错
- LuaJIT / PUC Lua 字节码不兼容
- 无法保存上值、函数环境
- C 函数（如 `math.sqrt`）完全不能序列化

---

### `core.serialize(data)`
- 序列化数据为**可执行的 Lua 代码字符串**
- 支持循环引用
- 遇到不支持类型直接报错：`Can't serialize data of type <type>`

### `core.deserialize(str, [safe])`
- 不是字符串 → `nil, "Cannot deserialize type..."`
- 以 `\27` 开头（字节码）→ `nil, "Bytecode prohibited"`
- 运行报错 → `nil, 错误信息`
- 正常 → 返回执行结果

`safe` 为真时：
- 序列化的函数反序列化为 `nil`
- 函数作键时会报错

否则：
- 函数会被设置空环境，只能操作字面量与参数

💡 小贴士
推荐：`data = assert(core.deserialize(lua, safe))`

⚠️ 警告
LuaJIT 下反序列化大对象可能报错。

---

## 引擎自带默认持久化
- **方块**（名称、param1、param2、元数据）随地图块自动保存
- 少量玩家属性（HP、氧气、位置、朝向、背包、元数据）自动保存
- 粒度由配置 `server_map_save_interval` 控制

---

## 存储方案
### 数据库 / 云端
用 Luanti HTTP 库对接网页服务存储。
也可用 IPC 如 sockets（`luasockets`，需要不安全环境）。
同机数据库可用文件桥接。

### 轻量数据库库
需要**不安全环境** + 库安装。
常用：SQLite3（`lsqlite3`），如 sban 模组。

### 字符串存储
#### 实体静态数据
绑定实体。由 `get_staticdata` 返回，传给 `on_activate`。

#### 文件存储
通常绑定世界/模组目录。
最简单：启动读、关闭写。
但崩溃/断电会**丢全部运行时数据**。

每次更新都存：大数据/高频更新会卡。
定期存：提升性能但降低安全性。
事务日志：只存变更，省性能但费空间。

💡 小贴士
混合方案最佳：只记变更 + 定期重写日志。

### 键值存储
#### 文件系统键值存储
除 Windows 外都好用：文件名=键，文件=值。
可嵌套目录做分级存储。

可用：
- `core.mkdir`
- `core.rmdir`
- `core.cpdir`
- `core.mvdir`
- `core.get_dir_list`

防丢失用：`core.safe_file_write`。

#### 配置文件（Settings）
`Settings` 对象提供简单键值读写。
`core.settings` 是全局配置。
⚠️ 不要乱用它存游戏数据，要做好命名空间隔离。

优点：用户易编辑。
缺点：可能导致玩家作弊。

---

## 元数据（MetaData）
Luanti 提供四类**字符串键值元数据**：
1. **物品栈**：`ItemStackMetaData` → 全量发给客户端
2. **方块位置**：`NodeMetaData` → 可设私有字段
3. **玩家**：`PlayerMetaData` → SQLite 存储，仅在线时可用
4. **模组**：`ModStorage` → SQLite 存储（旧版用 JSON）

提供存取数字、浮点数的工具，但**不保存类型**。
所有元数据的保存粒度由 `server_map_save_interval` 控制。
