---
title: luanti 开发问题排查
date: 2026-04-09 15:24:05
categories:
- luanti
tags:
- luanti
- games
- mods
---

# 开发问题排查

---

## Lua API 常见坑
先把 **Lua 5.1 官方手册**吃透，少犯低级错误！

### Q：我写了新类，但调用方法报“userdata value”错误
**A：你忘了注册类！**
C++ 暴露给 Lua 的对象默认是裸 userdata，必须配 `__index` 和 `__metatable` 才能当对象用。
照着 `LuaPerlinNoise` 的 `Register()` 写法抄，并且在 `scriptapi_export()` 里调用。

<!--more-->

---

## Lua 里那些气死人的小细节
- **循环次数坑**：`for i=0,20 do` 会跑 **21 次**，不是 20 次！
- **数组下标坑**：Lua 数组**默认从 1 开始**，不是 0！所有标准库和 Luanti API 都遵守这个规则。

### Q：数组传进/传出 API 就变 nil，坏了！
**A：你用了浮点数当下标！**
因为浮点精度误差，`z-z1` 可能是 `0.0000000002` 而不是 0，Lua 一查表直接返回 nil。

**解决：**
```lua
array[math.floor(z-z1)]
```
强行转整数下标！

---

## 节点定义（Nodedef）问题
### Q：初始化时创建 MapNode，结果全是 CONTENT_IGNORE
**A：执行时机太早！**
必须等 **Lua 脚本加载完成** 再创建节点，否则定义还没注册。

### Q：用别名注册节点，default:* 能用，自定义别名却返回 CONTENT_IGNORE
**A：别名映射表没更新！**
需要手动调用 `CNodeDefManager::updateAliases()` 刷新。
临时方案：先存名字，后面再取 ID。

---

## 柏林噪声（Perlin Noise）问题
### Q：噪声完全错乱，全是斜线/乱码
**A：99% 是下标越界或坐标顺序错了！**
- 检查 `noise->result` 取值是否正常
- 必须用**相对坐标**访问数组
- 检查 `stride`（行偏移）是否正确
- 加数据断点，看是不是被意外改写了

### Q：噪声正常，但区块之间接缝对不上
**A：两种典型错误：**
1. **矩阵转置了**：x/y 循环写反
2. **坐标传错**：必须传**绝对坐标**给 `getPerlinMap2D/3D`，不要自己算偏移

---

## 地图生成光照问题
### Q：用 VoxelManipulator 生成区块后，世界全黑！只有开飞行才看到蓝天
**A：你在顶部生成了 CONTENT_IGNORE！**
`CONTENT_IGNORE` **不透光**，会挡住太阳光。
`VoxelManipulator` 缓冲区别乱填非忽略方块！

---

## Luanti 崩溃排查
### Q：一开启公共服务器列表就崩溃，怎么查原因？
**A：Linux 用 gdb 抓崩溃栈！**
1. 编译时加：`-DCMAKE_BUILD_TYPE=Debug`
2. 安装 gdb
3. 运行：
```bash
bin/luanti --debugger --verbose
```
4. 复现崩溃，自动打印调用栈
5. 也可以用 `dmesg` 看系统级错误
