---
title: luanti L‑System 自动种树教程
date: 2026-04-09 16:23:11
categories:
- luanti
tags:
- luanti
- games
- mods
---

# L‑System 自动种树教程
Luanti 内置两种树：
1. 简单小树（原版 MTG 那种）
2. **L‑System 智能大树**（用公式自动生成复杂树形）

L‑System 树用 `core.spawn_tree` 生成，靠一个 `treedef` 表格完全定义。

---

## 核心原理：海龟绘图（Turtle Graphics）
你可以把它当成**一只会画图的笔（海龟）**：
- 移动
- 旋转
- 放方块（树干/树叶/果实）

### 移动指令
| 符号 | 作用 |
|------|------|
| G | 抬笔前进（不放方块） |
| F | 落笔前进（放树枝） |
| f | 落笔前进（放树叶） |
| T | 落笔前进（只放树干） |
| R | 落笔前进（放果实） |

### 旋转指令
| 符号 | 作用 |
|------|------|
| + | 向右转（偏航） |
| - | 向左转（偏航） |
| & | 向下低头 |
| ^ | 向上抬头 |
| / | 向右滚转 |
| * | 向左滚转 |

海龟**初始朝上**。

---

## 公理（Axiom）
就是**起始指令串**，海龟从这行命令开始执行。

例子：
- 公理 `TTTT` → 垂直向上长 4 格树干
- 公理 `TTTTT&TTTTT&TTTTT&TTTTT` + 角度 90° → 画出一个正方形

---

## 栈（保存/恢复状态）
| 符号 | 作用 |
|------|------|
| [ | 保存当前位置+方向 |
| ] | 读档回到保存点 |

用来长**分叉**！

例子：
`TTTTT[^TTTTT][&TTTTT]` + 60° → 直接长成 **Y 字形树**。

---

## L‑System 基础：替换规则
A/B/C/D 会被**递归替换**成对应规则，直到达到最大迭代次数。

| 符号 | 作用 |
|------|------|
| A | 替换成规则 A |
| B | 替换成规则 B |
| C | 替换成规则 C |
| D | 替换成规则 D |

### 举个栗子：Y 形分形树
公理：`TTTTTA`
规则 A：`[^TTTTTA][&TTTTTA]`

迭代 3 次后，就会变成一棵**超级分叉树**，顶端有 8 个分叉。

---

## 概率替换（随机树形）
小写 a/b/c/d 有**概率触发**，让树长得自然不重复：

| 符号 | 概率 |
|------|------|
| a | 90% 执行 A |
| b | 80% 执行 B |
| c | 70% 执行 C |
| d | 60% 执行 D |

---

# L‑System 树实例（直接复制可用）

## 巨型枯灌木
```lua
treedef={
   axiom = "A/A/A/A/A/A/A/A/A/A/A/A",
   rules_a = "[B+B+B+B]",
   rules_b = "[FFFFFFFFFF]",
   trunk = "mapgen_tree",
   angle = 30,
   iterations = 1,
   random_level = 0,
   trunk_type = "single",
   thin_branches = true
}
```

## 苹果树
```lua
treedef = {
	axiom="FFFFFAFFBF",
	rules_a="[&&&FFFFF&&FFFF][&&&++++FFFFF&&FFFF][&&&----FFFFF&&FFFF]",
	rules_b="[&&&++FFFFF&&FFFF][&&&--FFFFF&&FFFF][&&&------FFFFF&&FFFF]",
	trunk="mapgen_tree",
	leaves="mapgen_leaves",
	angle=30,
	iterations=2,
	random_level=0,
	trunk_type="single",
	thin_branches=true,
	fruit_chance=10,
	fruit="mapgen_apple",
}
```

## 金合欢树（Acacia）
```lua
treedef={
   axiom="FFFFFFccccA",
   rules_a = "[B]//[B]//[B]//[B]",
   rules_b = "&TTTT&TT^^G&&----GGGGGG++GGG++"
         .."fffffffGG++G++"
         .."Gffffffff--G--"
         .."ffffffffG++G++"
         .."fffffffff--G--"
         .."fffffffff++G++"
         .."fffffffff--G--"
         .."ffffffffG++G++"
         .."Gffffffff--G--"
         .."fffffffGG"
         .."^^G&&----GGGGGGG++GGGGGG++"
         .."ffffGGG++G++"
         .."GGfffff--G--"
         .."ffffffG++G++"
         .."fffffff--G--"
         .."ffffffG++G++"
         .."GGfffff--G--"
         .."ffffGGG",
   rules_c = "/",
   trunk="default:acacia_tree",
   leaves="default:acacia_leaves",
   angle=45,
   iterations=3,
   random_level=0,
   trunk_type="single",
   thin_branches=true,
}
```
