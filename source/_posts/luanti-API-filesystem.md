---
title: luanti API 文件系统
date: 2026-04-11 12:17:04
categories:
- luanti
tags:
- luanti
- games
- mods
- API
---

# 文件系统
（官方文档风格 · 简洁准确 · 可直接用于模组开发）

# 文件系统（Filesystem）
Luanti 提供大量工具函数，用于管理**路径、文件、文件夹**。


# 重要说明
模组安全系统会**限制文件访问路径**：
- 加载时：只能访问**当前模组目录（mod path）**
- 运行时：只能访问**世界存档目录（world path）**
详见：Lua 环境（Lua Environment）

<!--more-->


# 路径（Paths）
## ⚠ 警告
**永远使用绝对路径，不要使用相对路径！**
不要假设当前工作目录是什么。

---

# 路径规范化（Normalization）
Luanti 会自动规范化传入的路径：
- 将 `/` 替换为**系统专属路径分隔符**
  - Windows：`\`
  - UNIX/Linux：`/`

所以你**永远可以用 `/` 写路径**，不会出错。
但注意：Luanti 返回的路径**通常不会被自动规范化**。

---

# DIR_DELIM
全局变量，保存**系统专属路径分隔符**。
- Windows：`\`
- UNIX：`/`

通常不需要手动使用（因为路径会自动规范化），
仅在需要**拼接、解析路径**时使用。

---

# core.get_modpath(modname)
获取一个已加载模组的路径。
如果模组未加载，返回 `nil`。

## 参数
- `modname`：模组名字（字符串）

## 推荐写法
```lua
local modpath = core.get_modpath(core.get_current_modname())
```
不要硬编码模组名！

## 返回
- `modpath`：模组目录的**绝对路径（未规范化）**

---

# core.get_worldpath()
无参数。

## 返回
- `worldpath`：世界存档目录的**绝对路径（未规范化）**

---

# 目录操作（Directories）
## 通用返回值
- `success`：布尔值，表示操作是否成功。

## 提示
可用 `assert(...)` 让失败时直接抛出错误。

---

# core.mkdir(path)
创建一个目录，**自动创建不存在的父目录**。

## 参数
- `path`：要创建的目录路径

---

# core.cpdir(source_path, destination_path)
将**源目录**复制到**目标目录**。

## 参数
- `source_path`：源目录路径
- `destination_path`：目标目录路径

## ⚠ 警告
如果目标目录已存在，**会被覆盖**。

---

# core.mvdir(source_path, destination_path)
将**源目录**移动到**目标目录**。

## 参数
- `source_path`：源目录路径
- `destination_path`：目标目录路径

## ⚠ 警告
如果目标是**非空目录**，移动会失败。

---

# core.rmdir(path, recursive)
删除一个目录。

## 参数
- `path`：要删除的目录路径
- `recursive`：布尔值，是否**递归删除内容**

## 规则
- 如果不设为 `true`，目录**非空时删除失败**。

---

# core.get_dir_list(path, is_dir)
列出目录**直接内容**（不递归）。

## 参数
- `path`：要遍历的目录路径
- `is_dir`：nil / 布尔值
  - `nil`：同时列出**文件夹 + 文件**
  - `true`：只列出**文件夹**
  - `false`：只列出**文件**

## 返回
- `entry_list`：字符串列表
  只返回**条目名称**，**不是完整路径**！
  顺序不保证。

---

# 文件操作（Files）

# core.safe_file_write(path, content)
**原子性二进制安全写入**：要么全部写入，要么完全不写。

原理：先写临时文件 → 再替换原文件。
用于**防止存档损坏**。

等价于（但原子安全）：
```lua
local f = io.open(path, "wb")
f:write(content)
f:close()
```

## 参数
- `path`：文件路径
- `content`：要写入的内容（字符串）

## 返回
- `success`：是否成功

---

# Lua 内置函数（已被安全重写）
所有函数都已被 Luanti **重写并应用模组安全限制**。

# io.open(filename, mode)
打开文件用于读/写。

## 重要提示
处理**二进制文件**必须用：
- `rb`（读）
- `wb`（写）
不要用 `r` / `w`，否则在 Windows 上会出错！

---

# io.lines([filename])
获取文件行的**迭代器**。

---

# io.tmpfile()
获取临时文件句柄。

---

# os.tmpname()
获取临时文件名。

---

# os.remove(filename)
删除一个条目（文件 或 空目录）。

---

# os.rename(oldname, newname)
移动/重命名一个条目（文件 或 目录）。

---

# 翻译说明
- 保持官方术语：`path`、`directory`、`atomic`、`normalization`
- 简洁、可直接用于中文模组文档
- 保留所有警告、提示、重要规则
- 符合模组开发者阅读习惯
