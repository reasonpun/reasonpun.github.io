---
title: luanti API-Classes ModStorage
date: 2026-04-11 11:43:13
categories:
- luanti
tags:
- luanti
- games
- mods
- API
---

# ModStorage
ModStorage 是**每个世界、每个模组独立**的持久化字符串键值存储，**完全继承 MetaData 的所有方法**。


## 保存机制
数据的保存频率由服务器配置 `map_save_interval` 决定。

<!--more-->

---

## 存储后端（两种）
- **JSON**：纯文本存储，简单但不支持二进制数据
- **SQLite3**：数据库存储，支持任意字节串，效率更高

### ⚠️ 重要警告
- JSON **不能存原始二进制数据**
- 即使 SQLite3 支持二进制，也**不要依赖**它，因为用户可能用 JSON 后端
- SQLite3 只保存**变化的部分**，更快、更安全
- JSON 每次都要**全量重写**，崩溃时更容易丢数据

---

## `core.get_mod_storage()`
- 必须在**模组加载时调用**
- 返回当前模组的**私有存储对象**

---

## 示例代码（欢迎语持久化）
```lua
-- 获取模组存储
local storage = core.get_mod_storage()

-- 玩家进入时发送欢迎语
core.register_on_joinplayer(function(player)
    local greeting = storage:get("greeting")
    if greeting then
        core.chat_send_player(player:get_player_name(), greeting)
    end
end)

-- 管理员设置欢迎语
core.register_chatcommand("/set_greeting", {
    params = "<欢迎语>",
    description = "设置玩家进服欢迎语",
    privs = {server = true},

    func = function(name, param)
        param = param:trim()
        storage:set_string("greeting", param)
        return true, param == "" and "欢迎语已清空" or "欢迎语已设置"
    end
})
```
