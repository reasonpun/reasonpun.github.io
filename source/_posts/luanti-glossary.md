---
title: luanti glossary
date: 2026-04-09 08:39:02
categories:
- luanti
tags:
- luanti
- games
- mods
---

# Luanti 术语 Glossary
这一页基本把 Luanti 相关黑话全收了，看完你就是圈内人！

<!--more-->


---

## Luanti 及其组件
**Luanti**
你现在看的这个游戏引擎，一般直接指**引擎本体**。

**Minetest**
Luanti 的**曾用名**，老名字了。

**IrrlichtMt / Irrlicht¹**
Luanti 用的 3D 渲染库。是开发者从原版 Irrlicht  fork 过来的魔改版——原版又老又没人维护。
现在已经合并进 Luanti 源码，目录名叫 `irr/`。

**Game（游戏）**
一堆模组 + 元数据的集合，**主菜单能直接选**。
别和 Modpack 搞混：Modpack 只是模组包，Game 是完整游戏。
以前叫 Subgame，现在过时了，别用。

**Minetest Game（MTG）**
以前 Luanti 自带的官方默认游戏，现在要去 ContentDB 下。

**minetest_game**
Minetest Game 的仓库名与技术名称。

**Mods（模组）**
所有游戏的组成部分，也能给现有游戏额外加装。
用 Lua + Luanti API 编写。
有时叫 SSM（服务端模组），用来和客户端 CSM 区分。

**Client-Side Mod（CSM）客户端模组**
实验性 API，允许客户端自己跑 Lua 代码。

**Server-Sent Client-Side Mod（SSCSM）**
还没实现的功能：服务端把 CSM 发给客户端跑，降低延迟用。

**Modpack（模组包）**
一组打包的模组，像普通模组一样勾选。
**不是 Game**，Game 是主菜单独立游戏。

**Texture pack（材质包）**
替换游戏/模组贴图的资源包，通常只适配特定游戏。

**ContentDB（CDB）**
官方内容平台，上传/下载游戏、模组、材质包。
主菜单里的内容浏览器就是靠它。

**Server（服务端）**
负责管理世界、分发数据、跑所有模组逻辑的程序。

**Client（客户端）**
玩家用来玩单机/连机的程序，负责渲染世界。
单机时客户端会后台偷偷开一个服务端，既是客户端也是服务端。

**minetest.conf**
Luanti 用户目录根配置文件，存所有设置。
完整配置列表看主菜单“所有设置”或 `minetest.conf.example`。

---

## 游戏内内容
**World（世界）**
Luanti 里的存档，包含：
- Map：地图方块、箱子内容等数据库
- 玩家密码、血量、背包
- 黑名单、种子、保护区域等文件

**Nodes（节点/方块）**
游戏里 1×1×1 米的立方体，按 MapBlock 加载。
玩家日常就叫 **blocks（方块）**。

**Node Metadata（节点元数据）**
绑在方块位置上的额外数据。
比如箱子存背包、牌子存文字。

**Liquids（液体）**
有液体属性的节点。
开发里别乱用这个词，要说清楚：液体渲染类型、流动、粘度、物理等。

**Blocks / MapBlocks**
16×16×16 个节点组成的**加载单元**。

**Sectors（扇区）**
从世界底到顶的一整列 MapBlock。

**MapChunks**
80×80×80 节点（5×5×5 MapBlock），只为地图生成性能做的抽象层。

**Object（对象）**
能移动、不固定在网格上的东西：
- **Entity（实体）**：Lua 定义的生物、掉落物、点燃的 TNT 等
- **Player（玩家）**：玩家对象

**Items（物品）**
能放进背包的东西，可挖掘、攻击、合成、放置。
三种技术类型：

**Tool / Tool item（工具）**
不可堆叠，有**耐久（wear）**，会用坏。

**Node item（节点物品）**
方块的“物品形态”，放置后变回节点。
每个方块都对应一个节点物品。

**Craftitem（合成物品）**
既不是工具也不是节点物品的纯物品。

**Item stacks / ItemStack**
一组相同物品的堆叠，带数量。

**Environment（环境）**
Lua/C++ 全局对象，能访问所有节点、实体、玩家。

**Active Block Modifier（ABM）**
在特定节点上定时运行的函数，比如草、苔藓生长。

**Loading Block Modifiers（LBM）**
类似 ABM，但**只在地图块加载时跑一次**。

---

## Lua 与模组开发相关
**Lua**
简单、小巧、飞快的编程语言，Luanti 脚本全靠它。

**Modding API / Lua API / Luanti API**
给模组用的一套函数与变量，用来扩展/修改游戏功能。
API = 应用程序编程接口。

**lua_api.md**
`doc/` 目录下的 Lua API 完整参考文档，也有 HTML 版。

---

## 注释
¹ 官方渲染库叫 **IrrlichtMt**，但大家常混用 **Irrlicht**。
我们说 Irrlicht 时，**基本都指 Luanti 的魔改版**，不是上古原版。
