---
title: luanti 保持世界兼容
date: 2026-04-09 16:19:21
categories:
- luanti
tags:
- luanti
- games
- mods
---

# 保持世界兼容
如果你的游戏或模组已经稳定、有很多玩家在玩，**尽量保持对旧世界的兼容**非常重要！
不然玩家升级后会出现**未知方块、贴图错误、数据崩坏**，直接炸档。

---

## 别名（Aliases）
方块和物品都支持**名称别名**，可以把旧的内部名称映射到新名称。

最简单的用法：
```lua
core.register_alias('mymod:旧物品名', 'mymod:新物品名')
```

批量重命名/重构模组命名空间时超好用：
```lua
for _, item in ipairs({
	"stone", "cobble", "dirt", "grass"
}) do
	core.register_alias('旧模组名:'..item, '新模组名:'..item)
end
```

---

## 加载块修正器（LBM）
LBM = **Loading Block Modifier**
用于在**地图块加载时**自动对方块执行修复逻辑，适合处理**复杂升级**，比如：
- 更新方块的界面（formspec）
- 修复元数据（meta）
- 升级方块状态

示例：升级方块表单到 v2 版本
```lua
core.register_lbm({
	label = "把酷炫方块的UI升级到v2",
	name = "mymod:update_formspec_coolblock_v2",
	nodenames = {"mymod:coolblock"},
	run_at_every_load = false, -- 只跑一次
	action = function(pos, node)
		local meta = core.get_meta(pos)
		if meta:get_int('form_ver') < 2 then
			meta:set_int('form_ver', 2)
			meta:set_string('formspec', get_coolblock_formspec())
		end
	end
})
```
