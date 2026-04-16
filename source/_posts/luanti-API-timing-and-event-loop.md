---
title: luanti API 计时与事件循环
date: 2026-04-16 16:39:52
categories:
- luanti
tags:
- luanti
- games
- mods
- API
---

# 计时与事件循环
Luanti 的 Lua API 可用于获取时间、日期，或调度**一次性/每帧执行**的事件。

## 设置项
- **服务器**：`dedicated_server_step` 控制**服务端帧间隔**，这是无忙等待时游戏循环的最大粒度。
- **单人模式**：帧率决定服务端帧执行频率。

<!--more-->

## Lua 内置函数
不受模组安全限制，服务端模组（SSM）与客户端模组（CSM）均可使用：
- `os.clock`：已使用的 CPU 时间（近似秒）
- `os.time`：当前时间，通常是**秒级 UNIX 时间戳**
- `os.difftime`：计算两个时间戳差值（基本等价 `b - a`）
- `os.date`：获取与格式化日期

### 示例：性能基准测试
```lua
local function slow_sum(n)
    local sum = 0
    for i = 1, n do
        sum = sum + i
    end
    return sum
end

local function benchmark(calls, func, ...)
    local time = os.clock()
    for _ = 1, calls do
        func(...)
    end
    return (os.clock() - time) / calls
end

print("slow_sum(1e6) 每次调用耗时：", benchmark(1e3, slow_sum, 1e6), "秒")
```

---

# 核心函数

## `core.get_us_time`
**用法**
```lua
time = core.get_us_time()
```
**返回**
- `time`：数字，系统相关时间戳，**微秒（µs）**，起点任意

💡 小贴士
除以 `1e6` 即可转为秒。

ℹ️ 说明
返回值**不可跨设备/重启**，只适合在运行时内存中使用。

💡 小贴士
适合做**现实时间差判断**（如限流）。
游戏内计时建议累加 `dtime`，或精度要求不高时用游戏时间。

### 示例：忙等短时间
```lua
local start = core.get_us_time()
repeat until core.get_us_time() - start > 1e3 -- 等待1000微秒
```
⚠️ 警告
会**阻塞服务端线程**造成卡顿，仅在绝对必要、且极短时间时使用。

---

## `core.get_gametime`
**用法**
```lua
time = core.get_gametime()
```
**返回**
- 世界创建后经过的**游戏内秒数**，暂停/关闭时不增加，随世界保存。

### 示例：高精度自定义游戏时间
```lua
myapi = {}
core.after(0, function()
    local gametime = core.get_gametime()
    core.register_globalstep(function(dtime)
        gametime = gametime + dtime
    end)
    function myapi.get_precise_gametime()
        return gametime
    end
end)
```
ℹ️ 说明
可能比官方 `get_gametime` 快/慢一帧，初始值有四舍五入，**不要持久化**。

---

## `core.register_globalstep`
**每服务端帧执行一次**的回调。

**用法**
```lua
core.register_globalstep(function(dtime)
    -- 每一帧都执行
end)
```
**参数**
- `dtime`：距离上一帧的时间（秒）

💡 小贴士
适合轮询没有专属回调的事件，如玩家控制 `player:get_player_control()`。

ℹ️ 说明
全局帧函数**极度影响性能**，必须极致优化，否则会造成服务器卡顿。

### 示例：定时执行（降低性能消耗）
```lua
local function run_periodically(period, func)
    local timer = 0
    core.register_globalstep(function(dtime)
        timer = timer + dtime
        if timer > period then
            func()
            timer = 0
        end
    end)
end

run_periodically(5, function()
    core.chat_send_all("5秒过去了！")
end)
```
如需**补帧执行**（catchup），把 `timer = 0` 改为：
```lua
timer = timer - period
```

---

## `core.after(time, func, ...)`
**延迟至少 `time` 秒后执行一次**。

💡 小贴士
`core.after(0, func)` 用于：
- 加载期执行**只能在运行时做**的操作
- 把逻辑调度到**下一帧**

**用法**
```lua
job = core.after(time, func, ...)
job:cancel() -- 取消
```
**参数**
- `time`：延迟秒数
- `func`：执行函数
- `...`：传递给函数的参数

**返回**
- `job`：可取消对象，带 `cancel()` 方法

⚠️ 重要
参数中**不要带 nil**，否则会被截断。
必须传 nil 时用闭包：
```lua
core.after(1, function()
    my_func(nil, a, b, nil)
end)
```

ℹ️ 说明
大量使用 `core.after` 可能导致线性遍历变慢，谨慎滥用。

---

# 实体相关
## `entity:on_step(dtime, ...)`
每个实体**每帧都会执行**的回调。

**用法**
```lua
function entity:on_step(dtime, ...)
    -- 实体专属每帧逻辑
end
```
**参数**
- `dtime`：帧间隔时间（与全局帧一致）
- `...`：其他不影响计时的参数（如移动结果）
