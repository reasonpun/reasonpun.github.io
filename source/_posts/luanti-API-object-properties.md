---
title: luanti API 对象属性
date: 2026-04-16 16:35:46
categories:
- luanti
tags:
- luanti
- games
- mods
- API
---

# 对象属性
本页面列出对象可拥有的全部属性。

<!--more-->

## 仅玩家适用的属性
### `hp_max`
- 内部类型：`u16`
- 默认值：`20`
- 设置玩家的最大生命值。

⚠️ 注意
把 `hp_max` 降到**当前生命值以下**，会触发受伤闪烁效果。

### `breath_max`
- 内部类型：`u16`
- 默认值：`10`
- 设置玩家的最大氧气值。

### `eye_height`
- 内部类型：`f32`
- 默认值：`1.625`
- 设置相机高度，单位为**方块**，从玩家脚底算起。

### `zoom_fov`
- 内部类型：`f32`
- 单位：角度(degrees)
- 默认值：`0`
- 设置玩家**缩放时**的视野角度。设为 `0` 表示禁用缩放。

---

## 所有对象通用属性
### `physical`
- 类型：布尔值(bool)
- 默认值：`false`
- 控制对象是否与**可行走(walkable)**的方块发生碰撞。

### `collide_with_objects`
- 类型：布尔值(bool)
- 默认值：`true`
- 控制对象是否与**其他对象**碰撞。

ℹ️ 说明
必须先把 `physical` 设为 `true`，此项才生效！

⚠️ 注意
对玩家也有效，但可能出现奇怪bug。

### `collisionbox`
- 类型：`cuboid_def`
- 默认值：`{-0.5, 0.0, -0.5, 0.5, 1.0, 0.5}`
- 对象的碰撞盒。格式：`{xmin, ymin, zmin, xmax, ymax, zmax}`
- 单位：方块，以对象中心点/位置为基准。

### `selectionbox`
- 类型：`cuboid_def`
- 默认值：同 `collisionbox`
- 对象的选中盒。不设置则自动使用碰撞盒。
- 格式：`{xmin, ymin, zmin, xmax, ymax, zmax}`

⚠️ 注意
格式与方块盒(nodebox cuboid)一致。

### `pointable`
- 类型：布尔值(bool)
- 默认值：`true`
- 控制对象是否可以被准星指向。

### `visual`
- 类型：字符串(string)
- 默认值：`"sprite"`
- 可选值：
  - `"cube"`：和普通方块一样大的立方体
  - `"item"`：类似手持物品，但忽略 `wield_item` 属性
  - `"mesh"`：使用模型，需搭配 `mesh` 属性
  - `"sprite"`：永远面向玩家的平面贴图
  - `"upright_sprite"`：竖直平面贴图
  - `"wielditem"`：掉落物品专用，搭配 `wield_item`
- 设置对象的显示外观。

💡 小贴士
`"wielditem"` 支持物品硬件着色。

### `mesh`
- 类型：字符串(string)
- 默认值：`""`
- 模型文件名，**仅当 visual 为 "mesh" 时有效**。

### `visual_size`
- 类型：向量(vector)
- 默认值：`{x = 1, y = 1, z = 1}`
- 视觉尺寸缩放倍数。格式：`{x = x, y = y, z = z}`
- 不提供 z 时，z 自动等于 x。

### `textures`
- 类型：字符串列表(string list)
- 默认值：`{"no_texture.png"}`
- 格式：`{"纹理.png", ...}`
- 贴图数量由 `visual` 决定：
  - `"cube"`：6 张贴图
  - `"item"`：第一张贴图为物品ID（已弃用）
  - `"mesh"`：每个模型材质 1 张
  - `"sprite"`：1 张
  - `"upright_sprite"`：2 张
  - `"wielditem"`：忽略贴图

### `colors`
- 内部类型：`ColorSpec list`
- 默认值：`{{r=255,g=255,b=255,a=255}}`

ℹ️ 说明
此功能**暂不可用**。

### `spritediv`
- 类型：2D向量(2d_vector)
- 默认值：`{x = 1, y = 1}`
- 设置精灵图集的**列数、行数**，用于动画。
- 格式：`{x=列数, y=行数}`

### `initial_sprite_basepos`
- 类型：2D向量(2d_vector)
- 默认值：`{x = 0, y = 0}`
- 设置精灵图集**起始帧位置**。
- 格式：`{x=列, y=行}`

### `is_visible`
- 类型：布尔值(bool)
- 默认值：`true`
- 对象是否可见、可被指向。设为 `false` 时会**覆盖** `pointable`。

### `makes_footstep_sound`
- 类型：布尔值(bool)
- 默认值：`false`
- 对象踩在有脚步声的方块上时，是否播放脚步声。

### `stepheight`
- 内部类型：`f32`
- 默认值：`0`
- 设置对象可**自动迈上**的最大高度（单位：方块）。

### `automatic_rotate`
- 内部类型：`f32`
- 默认值：`0`
- 对象沿 Y 轴**自动旋转速度**，单位：弧度/秒。

⚠️ 注意
（暂时）对被挂载的实体无效。

### `automatic_face_movement_dir`
- 类型：整数(int) 或 `false`
- 单位：角度(degrees)
- 默认值：`0`

💡 小贴士
可设为 `false` 关闭此功能。

- 让对象自动朝向移动方向，数值为角度偏移。

### `backface_culling`
- 类型：布尔值(bool)
- 默认值：`true`
- 是否开启模型背面剔除（背对相机的面不渲染）。

### `glow`
- 类型：整数(int)
- 默认值：`0`
- 渲染贴图时额外增加的光照值。
- 最大值：`core.LIGHT_MAX`
- 小于 0 表示**禁用光照**。

### `nametag`
- 类型：字符串(string)
- 默认值：`""`
- 显示在对象上方的文字，一般用于名字。
  - 玩家：为空则自动显示玩家名；想隐藏需把 `nametag_color` 的 alpha 设为 0。
  - 非玩家：为空则隐藏标签。

### `nametag_color`
- 类型：`ColorSpec` 或 `false`
- 默认值：`{a=255, r=255, g=255, b=255}`
- 名字标签文字颜色。

💡 小贴士
可设为 `false` 关闭。

### `nametag_bgcolor`
- 类型：`ColorSpec` 或 `false`
- 名字标签背景颜色。

💡 小贴士
设为 `false` 不显示背景。

### `automatic_face_movement_max_rotation_per_sec`
- 类型：`f32`
- 默认值：`-1`
- 限制自动转向的**最大每秒旋转角度**。
- ≤0 表示无限制。

### `infotext`
- 类型：字符串(string)
- 玩家指向对象时显示的提示文字。

### `static_save`
- 类型：布尔值(bool)
- 默认值：`true`
- 对象是否被静态保存到地图。
- 设为 false：区块卸载时对象会被删除。

### `wield_item`
- 内部类型：`ItemString`
- **仅当 visual 为 "wielditem" 时有效**。

### `use_texture_alpha`
- 类型：布尔值(bool)
- 默认值：`false`
- 是否开启纹理半透明（有bug的那种）。

⚠️ 警告
半透明的脸、实体等可能渲染顺序错乱。

### `shaded`
- 类型：布尔值(bool)
- 默认值：`true`
- 是否对对象应用漫反射光照。

### `show_on_minimap`
- 类型：布尔值(bool)
- 默认值：`false`
- 对象是否在小地图上显示。

### `damage_texture_modifier`
- 类型：字符串(string)
- 默认值：`"^[brighten"`
- 对象受伤闪烁时，追加到当前纹理上的**修饰器**。

---

# 对象属性格式
所有属性以**表(table)** 形式定义。

⚠️ 注意
多余属性会被忽略，但未来引擎可能误读，建议用**下划线前缀**命名自定义属性。

⚠️ 注意
使用普通索引，元表行为正常。

⚠️ 警告
所有字符串属性的最大长度为 `u16` 最大值。

⚠️ 注意
数字属性会被**限制在合法范围**内。

---

## 示例
```lua
{
	physical = true,
	collide_with_objects = true,
	collisionbox = {-0.3, -0.3, -0.3, 0.3, 0.3, 0.3},
	selectionbox = {-0.3, -0.3, -0.3, 0.3, 0.3, 0.3},
	pointable = true,
	visual = "mesh",
	mesh = "mesh.obj",
	visual_size = {x = 1, y = 1, z = 1},
	textures = {"texture.png", "texture.png"},
	initial_sprite_basepos = {x = 0, y = 0},
	is_visible = true,
	makes_footstep_sound = true,
	stepheight = 0.4,
	automatic_rotate = 0.0,
	automatic_face_movement_dir = 0,
	automatic_face_movement_max_rotation_per_sec = 0,
	backface_culling = true,
	glow = 0,
	nametag = "My object",
	nametag_color = {r = 255, g = 255, b = 255},
	nametag_bgcolor = {r = 0, g = 0, b = 0},
	infotext = "Hello there!",
	static_save = true,
	use_texture_alpha = true,
	shaded = true,
	show_on_minimap = true,
	damage_texture_modifier = "^[colorize:#FF0000:150",
}
```