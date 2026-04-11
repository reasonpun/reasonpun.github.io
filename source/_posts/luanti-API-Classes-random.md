---
title: luanti API-Classes 随机数（Random）
date: 2026-04-11 11:48:32
categories:
- luanti
tags:
- luanti
- games
- mods
- API
---

# 随机数（Random）
Luanti 提供**四种随机数生成器**，各有优缺点，模组开发者需要根据场景选择。

## Lua 内置随机（全局）
不受模组安全限制，服务端和客户端模组都能用。

### `math.randomseed(seed)`
设置全局随机种子。
引擎**已经用系统时间自动帮你设置过**。

<!--more-->

### 重要提醒
- **不要自己手动设置种子**，会破坏其他模组的随机性
- 全局只有一个种子，一旦修改会影响所有代码
- 如果你必须临时用固定种子，用完要立刻还原

```lua
-- 安全用法示例
local reseed = math.random(2147483647) -- 先存一个随机种子
math.randomseed(我的固定种子) -- 临时 deterministic
-- 做你的事...
math.randomseed(reseed) -- 恢复全局随机性
```

### `math.random()`
获取随机数：
- 无参：0~1 浮点数
- 1 个参：1~n
- 2 个参：min~max

### 缺点
- 不同平台、不同 Lua 版本结果不一样
- **不能用于地图生成**（会出现不同步）
- 精度约 32 位，不是完整 52 位

### 适用场景
快速、非确定性、不需要跨平台一致的随机。

---

## 四种随机生成器详解

### 1. PcgRandom（推荐！）
**可设置种子、32 位、高质量、跨平台一致**
适合地图生成、结构、树、矿石等。

```lua
local r = PcgRandom(seed)
r:next([min, max])
```

- 默认范围：-2³¹ ~ 2³¹-1
- 质量高、速度不错
- **全平台结果完全一致**

### `:rand_normal_dist(min, max, [num_trials])`
近似正态分布（高斯）
均值在中间， trials 越大越接近正态。
**官方不推荐使用**。

---

### 2. PseudoRandom（老古董，不推荐）
16 位 LCG 随机算法，质量最差。

```lua
local r = PseudoRandom(seed)
r:next()
```

- 范围：0~65535
- 分布差、功能弱
- **官方结论：永远不要用**

---

### 3. SecureRandom（密码级安全随机）
系统级加密安全随机数，**不可预测、不可设置种子**。

用于：
- 生成令牌
- 密钥
- 抽奖、防作弊

```lua
local r = SecureRandom()
local bytes = r:next_bytes([count])
```

- 最多一次取 2048 字节
- Windows 使用 CryptoAPI
- Linux/Android 使用 /dev/urandom

---

## 性能对比（越快越好）
- math.random：最快（1x）
- PcgRandom：约 14x
- PseudoRandom：约 15x
- SecureRandom：约 30x

---

## 总结对比表（精简版）
| 类型 | 可设种子 | 跨平台一致 | 质量 | 用途 |
|------|----------|-------------|------|------|
| math.random | 全局 | ❌ 不保证 | 一般 | 快速普通随机 |
| **PcgRandom** | ✅ 实例独立 | ✅ 完全一致 | 高 | **地图生成、模组逻辑** |
| PseudoRandom | ✅ | ✅ | 差 | 不用 |
| SecureRandom | ❌ | — | 最高 | 密码、防作弊 |

---

## 官方最终结论
- **永远不要用 PseudoRandom**，被 PcgRandom 全面吊打
- 要**快、简单** → `math.random`
- 要**确定性、地图生成、全平台一致** → **PcgRandom**
- 要**加密安全** → SecureRandom
