---
title: 你的第一个生物
date: 2026-04-09 15:53:18
categories:
- luanti
tags:
- luanti
- games
- mods
---

# 你的第一个生物
这篇教程带你从零做一只**简单的攻击型怪物**，名字叫 **Bloblet（小绿团）**。
教程会教你实体（entity）基础用法——凡是比方块更“能动”的东西，全都靠实体实现。

前提：你已经有一个基础模组 `mymod`，要编辑 `mymod/init.lua`。
版本要求：**Luanti 5.5.0 及以上**。

{% asset_img bloblets.png 示意图 width="400" %}

<!--more-->

---

## 实体定义
实体用 `core.register_entity` 注册，最简写法：
```lua
core.register_entity("mymod:bloblet", {})
```
拿到生成权限后，用指令召唤：
`/spawnentity mymod:bloblet`

现在它只是个**无贴图白方块**，接下来我们一步步完善。

---

## 视觉外观
外观写在 `initial_properties` 里：
- `visual = "cube"`：用立方体显示
- `visual_size`：大小（这里设为 0.5 倍）
- `textures`：6 个面的贴图（上、下、X+、X-、Z+、Z-）

```lua
initial_properties = {
	visual = "cube",
	visual_size = vector.new(0.5, 0.5, 0.5),
	textures = {
		"[fill:1x1:green", -- 上
		"[fill:1x1:green", -- 下
		"[fill:1x1:green", -- 右
		"[fill:1x1:green", -- 左
		"[fill:1x1:red",   -- 前（红）
		"[fill:1x1:green", -- 后
	},
},
```

---

## 基础物理（重力+碰撞）
让怪物**会掉下来、有碰撞**：
- `physical = true`：开启物理
- `collisionbox`：碰撞箱（中心向外扩展）
- `on_activate`：激活时加重力

```lua
initial_properties = {
	physical = true,
	collisionbox = {-0.25, -0.25, -0.25, 0.25, 0.25, 0.25},
},

on_activate = function(self)
	self.object:set_acceleration(vector.new(0, -10, 0)) -- 重力
end,
```

---

## 可被攻击（血量+选择箱）
给怪物加血量，让玩家能打它：
```lua
initial_properties = {
	hp_max = 5, -- 5点血量 = 2.5颗心
	selectionbox = {-0.25, -0.25, -0.25, 0.25, 0.25, 0.25},
},
```

---

## 基础行为逻辑
### 追逐玩家
`on_step` 每帧执行：
- 找最近的玩家
- 朝目标移动
- 保留 Y 轴速度（防止浮空）

```lua
on_step = function(self, dtime)
	local obj_pos = self.object:get_pos()
	local min_distance = 20
	local target

	for _, player in ipairs(core.get_connected_players()) do
		local dist = player:get_pos():distance(obj_pos)
		if dist < min_distance then
			min_distance = dist
			target = player
		end
	end

	if target then
		local diff = target:get_pos() - obj_pos
		diff.y = 0
		local dir = diff:normalize()
		local vel = dir * 2
		vel.y = self.object:get_velocity().y
		self.object:set_velocity(vel)
	end
end
```

### 跳上方块
加 `stepheight` 让它能**跨上1格高**：
```lua
stepheight = 1.01,
```

### 面朝玩家
让红色正面永远对着你：
```lua
self.object:set_rotation(vector.new(diff.x, 0, diff.z):dir_to_rotation())
```

### 攻击玩家
加攻击计时器，**2 秒冷却**，靠近 1 格内造成伤害：
```lua
on_activate = function(self)
	self._attack_timer = 0
end,

on_step = function(self, dtime)
	self._attack_timer = self._attack_timer + dtime

	if target and self._attack_timer >= 2 and min_distance <= 1 then
		target:set_hp(target:get_hp() - 1)
		self._attack_timer = 0
	end
end
```

---

## 自然生成
每 10 秒在玩家附近**随机尝试生成**：
- 随机位置
- 射线检测找地面
- 距离玩家大于 10 格才生成

```lua
local function try_spawn()
	local players = core.get_connected_players()
	local player = players[math.random(#players)]

	for _ = 1,100 do
		local start = player:get_pos() + vector.new(
			math.random(-25,25), 10, math.random(-25,25)
		)
		if start:distance(player:get_pos()) >=10 then
			local ray = core.raycast(start, start+vector.new(0,-10,0), false):next()
			if ray then
				core.add_entity(ray.above, "mymod:bloblet")
				break
			end
		end
	end
end

local spawn_timer = 0
core.register_globalstep(function(dtime)
	spawn_timer = spawn_timer + dtime
	if spawn_timer >=10 then
		spawn_timer = 0
		try_spawn()
	end
end)
```

---

## 完整代码（直接复制可用）
```lua
core.register_entity("mymod:bloblet", {
	initial_properties = {
		visual = "cube",
		visual_size = vector.new(0.5, 0.5, 0.5),
		textures = {
			"[fill:1x1:green",
			"[fill:1x1:green",
			"[fill:1x1:green",
			"[fill:1x1:green",
			"[fill:1x1:red",
			"[fill:1x1:green",
		},
		physical = true,
		collisionbox = {-0.25, -0.25, -0.25, 0.25, 0.25, 0.25},
		hp_max = 5,
		selectionbox = {-0.25, -0.25, -0.25, 0.25, 0.25, 0.25},
		stepheight = 1.01,
	},

	on_activate = function(self)
		self.object:set_acceleration(vector.new(0, -10, 0))
		self._attack_timer = 0
	end,

	on_step = function(self, dtime)
		local obj_pos = self.object:get_pos()
		local min_distance = 20
		local target

		for _, player in ipairs(core.get_connected_players()) do
			local dist = player:get_pos():distance(obj_pos)
			if dist < min_distance then
				min_distance = dist
				target = player
			end
		end

		self._attack_timer = self._attack_timer + dtime

		if target then
			local diff = target:get_pos() - obj_pos
			diff.y = 0
			local dir = diff:normalize()
			local vel = dir * 2
			vel.y = self.object:get_velocity().y
			self.object:set_velocity(vel)

			self.object:set_rotation(vector.new(diff.x, 0, diff.z):dir_to_rotation())

			if self._attack_timer >= 2 and min_distance <= 1 then
				target:set_hp(target:get_hp() - 1)
				self._attack_timer = 0
			end
		end
	end,
})

-- 生成逻辑
local function try_spawn()
	local players = core.get_connected_players()
	if #players == 0 then return end
	local player = players[math.random(#players)]

	for _ = 1,100 do
		local start = player:get_pos() + vector.new(math.random(-25,25),10,math.random(-25,25))
		if start:distance(player:get_pos()) >=10 then
			local ray = core.raycast(start, start+vector.new(0,-10,0), false):next()
			if ray then
				core.add_entity(ray.above, "mymod:bloblet")
				break
			end
		end
	end
end

local spawn_timer = 0
core.register_globalstep(function(dtime)
	spawn_timer = spawn_timer + dtime
	if spawn_timer >=10 then
		spawn_timer = 0
		try_spawn()
	end
end)
```

---

## 下一步进阶方向
- 给怪物画**真正的贴图**
- 加入**动画模型**
- 增加**随机漫步**、逃跑、回血等行为
- 支持攻击**其他实体**
- 做友好/敌对不同阵营
- 加**粒子特效、声音**
- 优化生成逻辑（不在玩家眼前刷出）
- 直接用现成生物库模组，不用重复造轮子
