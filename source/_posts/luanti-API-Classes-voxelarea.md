---
title: luanti API-Classes voxelarea
date: 2026-04-11 12:15:31
categories:
- luanti
tags:
- luanti
- games
- mods
- API
---

# VoxelArea
VoxelArea 是用于**VoxelManip（地图区块操作）**的面向对象工具类，专门用来**计算坐标索引**，避免手写复杂的三维下标公式。

---

## VoxelArea.new(self, o)
创建一个 VoxelArea 实例。
会自动设置 `self.__index = self`，并将 `self` 设为 `o` 的元表。

### 必须字段
- **MinEdge**：区域最小坐标（包含）
  为整数向量，默认 `vector.new(1, 1, 1)`
- **MaxEdge**：区域最大坐标（包含）
  为整数向量，默认 `vector.new(0, 0, 0)`

<!--more-->

### 要求
- 两个坐标都必须是**整数向量**
- MinEdge 的每个分量必须 **小于** MaxEdge 的对应分量
- 可以用 `minp, maxp = vector.sort(minp, maxp)` 自动排序

### 自动计算
创建时会自动计算并存储：
- `ystride`
- `zstride`

---

## 典型用法
```lua
local voxelmanip = core.get_voxel_manip(pos_min, pos_max)
local emin, emax = voxelmanip:read_from_map(pos_min, pos_max)

-- 必须传入实际 emerged 的最小/最大坐标
local voxelarea = VoxelArea:new{
    MinEdge = emin,
    MaxEdge = emax
}
```

---

# 重要警告
1. **必须传入实际 emerge 后的最小/最大坐标**
   不要传入你“想要”的最小/最大坐标。
2. **绝对不能传入小数坐标**
   必须是整数，可用：
   - `vector.floor()`
   - `vector.round()`
   - `vector.apply(vec, math.ceil)`

---

# 方法（OOP 调用方式）
下面示例中 `area` 是一个合法的 VoxelArea 实例。

## area:getExtent()
返回区域的**尺寸向量**（长、宽、高）。
返回值：整数向量 `{x=dx, y=dy, z=dz}`

## area:getVolume()
返回区域内**总方块数量**（体积）。

## area:index(x, y, z)
输入**绝对世界坐标**，返回在 VoxelManip 数据表里的**索引下标**。

### 警告
- 会自动向下取整，**不会报错**
- 你必须确保坐标是整数
- 你必须确保坐标在区域范围内

## area:indexp(p)
`area:index(p.x, p.y, p.z)` 的简写。
传入坐标向量，返回索引。

## area:position(index)
`indexp` 的**逆运算**。
输入索引，返回对应的**绝对世界坐标**。

### 提示
返回的坐标**没有 vector 元表**。
如果需要标准向量：
```lua
local p = vector.new(area:position(index))
```

## area:contains(x, y, z)
判断坐标 (x,y,z) **是否在区域范围内**（包含边界）。
返回：`true` / `false`

## area:containsp(p)
`contains(p.x, p.y, p.z)` 的简写。
传入向量，返回是否在区域内。

## area:containsi(i)
判断索引 `i` 是否在 **1 到 区域体积** 之间（包含）。

### 警告
`area:containsi(area:indexp(p))` **不等于** `area:containsp(p)`
因为 `indexp` 对区域外坐标也可能生成合法索引。

---

## area:iter(minx, miny, minz, maxx, maxy, maxz)
返回一个**迭代器**，按 **XYZ 顺序**遍历区域内所有方块：
1. 先遍历 X
2. 再遍历 Y
3. 最后遍历 Z

## area:iterp(minp, maxp)
`iter()` 的简写，直接传入两个向量。

### 示例（最常用）
```lua
for index in area:iterp(pos_min, pos_max) do
    content_id_data[index] = 方块ID
end
```

等价于（但更简短、更快）：
```lua
for z = pos_min.z, pos_max.z do
    for y = pos_min.y, pos_max.y do
        for x = pos_min.x, pos_max.x do
            local index = area:index(x, y, z)
            content_id_data[index] = 方块ID
        end
    end
end
```