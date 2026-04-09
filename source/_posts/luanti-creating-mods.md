---
title: luanti creating-mods
date: 2026-04-09 09:41:36
categories:
- luanti
tags:
- luanti
- games
- mods
---


# 制作模组
Luanti 内置一套**脚本 API**，专门用来写游戏和[模组](https://docs.luanti.org/for-players/mods)，你可以用它做全新玩法，也可以给现有游戏疯狂扩内容。
这套 API 用 [**Lua**](https://api.luanti.org/) 语言编写——简单、好学、轻量。引擎目前使用 **Lua 5.1**，但很多人会跑 **LuaJIT** 来提速。
你只需要有**基础编程常识**就能上手。

---

## 官方文档与教程
- 官方 Lua API 文档：**api.luanti.org**
- [本站 API 板块也有全套指南](https://docs.luanti.org/for-creators/api)

想做你的第一个模组/游戏，最强入门材料是：
[**《Luanti Modding Book》**](https://rubenwardy.com/minetest_modding_book/en/index.html)
作者：rubenwardy，校对：Shara

⚠️ 小知识：
Luanti 里的**游戏**和**模组**是同一套系统，游戏本质就是**一堆模组的集合**。
详细看[《模组开发书》](https://rubenwardy.com/minetest_modding_book/en/games/games.html)里的「Games」章节。

---

## 推荐工具（大佬都在用）
- **VS Code / VSCodium**：最强代码编辑器
  我们还出了 [**Luanti Tools**](https://marketplace.visualstudio.com/items?itemName=GreenXenith.minetest-tools) 插件，自带代码补全，插件商店直接搜
- [**luacheck**](https://github.com/lunarmodules/luacheck)：Lua 代码静态检查工具，帮你找bug
- [**Lua Language Server**](https://luals.github.io/)：带类型提示的Lua智能提示工具（作者：[Archie](https://git.minetest.land/andro/luanti-api)）
