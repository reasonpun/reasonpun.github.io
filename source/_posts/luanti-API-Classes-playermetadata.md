---
title: luanti API-Classes PlayerMetaData
date: 2026-04-11 11:47:27
categories:
- luanti
tags:
- luanti
- games
- mods
- API
---

# PlayerMetaData
PlayerMetaData 是**每个世界、每个玩家独立**的持久化字符串键值存储，**完全继承 MetaData 所有方法**。

## 保存频率
数据保存间隔由服务器配置 `map_save_interval` 控制。

## 重要规范（必看）
PlayerMetaData 是**所有模组共用**的，为了避免键名冲突，**必须给你的 key 加模组名前缀**。

例子：
- 任务模组的分数 → `fancy_quests:score`
- 生物模组的分数 → `fancy_mobs:score`

<!--more-->

## `player:get_meta()`
获取一个玩家的元数据对象。

### 注意
- 必须传入**在线玩家对象**才能使用
- **玩家离线时无法访问** PlayerMetaData
- 如果你需要存**离线玩家数据**，请用 **ModStorage**，并把玩家名加进键名里

### 离线存储替代方案（ModStorage）
```lua
-- 把玩家数据存在 ModStorage 中，支持离线读取
local storage = core.get_mod_storage()
local player_name = player:get_player_name()

-- 存
storage:set_string(player_name .. ":score", "100")

-- 取
local score = storage:get_string(player_name .. ":score")
```

---

## 总结一句话
- **ModStorage**：模组全局，世界维度，离线可读写
- **NodeMetaData**：单个方块，带界面、带背包
- **PlayerMetaData**：单个玩家，在线才能读写
