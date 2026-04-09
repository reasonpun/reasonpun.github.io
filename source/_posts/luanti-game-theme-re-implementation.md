---
title: 游戏主题重实现
date: 2026-04-09 15:40:54
categories:
- luanti
tags:
- luanti
- games
- mods
---

# 游戏主题重实现
主菜单里负责**背景、标题栏**等主题的逻辑，默认是用 `mm_game_theme`（在 `game_theme.lua` 里）实现的。
如果你只是想给**所有页面统一用一张背景 + 一个标题图**，完全没必要去啃这套复杂逻辑。

---

## `mm_game_theme` 到底干了啥？
它本质就是**封装调用**了这个核心函数：
`core.set_background`
用来设置：背景、顶部标题、底部栏、覆盖层。

函数用法：
```lua
core.set_background(类型, 图片路径, 是否平铺, 最小尺寸)
```
- **type**：background（背景）、overlay（覆盖层）、header（顶部标题）、footer（底部）
- **tile**：是否平铺（只对背景有效）
- **minsize**：平铺前最小缩放尺寸

---

## 极简重写示例（直接抄就能用）
下面是来自 **Voxelmanip Classic** 的超简方案，
直接替换掉 `mm_game_theme`，实现**静态背景 + 固定标题**：

```lua
-- 设置顶部标题
core.set_background('header', defaulttexturedir .. "menu_header.png")

-- 设置主背景
core.set_background('background', defaulttexturedir .. "menu_bg.png")

-- 关闭云层动画
core.set_clouds(false)
```

把这段代码**放进 `init_globals()` 函数里**，
直接删掉原来调用 `mm_game_theme` 的代码就行。
