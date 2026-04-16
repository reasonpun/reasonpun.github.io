---
title: luanti API 模组通信频道
date: 2026-04-16 16:34:32
categories:
- luanti
tags:
- luanti
- games
- mods
- API
---

# 模组通信频道（Modchannels）

用于**服务端模组（SSM）** 和**客户端模组（CSM）** 互相发消息、双向通信。

<!--more-->

## 服务端 API（Server Side API）
### 函数（Functions）
**`core.mod_channel_join(channel_name)`**
- `channel_name`：字符串，模组频道名称
- 返回一个可调用方法的频道对象
- 频道不存在则自动创建，并立即加入该频道

**`core.register_on_modchannel_message(function(channel_name, sender, message))`**
- `channel_name`：字符串，已加入的模组频道名
- `sender`：字符串，服务端发来则为空，客户端发来则为玩家名
- `message`：字符串，消息内容
- 用于处理从客户端收到的消息

### 方法（Methods）
> 注意：下面的 `channel` 均指 `core.mod_channel_join` 返回的对象

**`channel:leave()`**
- 服务端退出该频道
- 退出后 `core.register_on_modchannel_message` 不再收到此频道消息

> 小贴士：退出后把频道变量设为 `nil` 释放资源

**`channel:is_writeable()`**
- 返回布尔值：可写返回 `true`，不可写返回 `false`

**`channel:send_all(message)`**
- `message`：字符串，最大 **65535 字节**
- 发给频道里所有服务端模组、客户端模组

> 说明：频道不可写或无效时，消息不会发送

---

## 客户端 API（Client Side API）
### 函数（Functions）
**`core.mod_channel_join(channel_name)`**
- `channel_name`：字符串，模组频道名称
- 返回频道对象，不存在则创建并加入
- 与服务端版本功能完全一致

**`core.register_on_modchannel_message(function(channel_name, sender, message))`**
- `channel_name`：字符串，已加入且已收到确认的频道
- `sender`：字符串，服务端发来则为空，客户端发来则为玩家名
- `message`：字符串，消息内容
- 用于处理收到的消息，与服务端功能一致

**`core.register_on_modchannel_signal(function(channel_name, signal))`**
- `channel_name`：字符串，信号来源频道名
- `signal`：整数，0–5，对应以下状态：
  - `join_ok`：加入成功
  - `join_failed`：加入失败
  - `leave_ok`：退出成功
  - `leave_failed`：退出失败
  - `event_on_not_joined_channel`：向未加入的频道发消息
  - `state_changed`：频道状态变更
- 用于处理模组频道系统发出的信号

### 方法（Methods）
> 注意：下面的 `channel` 均指 `core.mod_channel_join` 返回的对象

**`channel:leave()`**
- 客户端退出频道
- 退出后不再通过 `core.register_on_modchannel_message` 接收该频道消息

> 小贴士：退出后设为 `nil` 释放资源

**`channel:is_writeable()`**
- 返回布尔值：可写返回 `true`，不可写返回 `false`

**`channel:send_all(message)`**
- `message`：字符串，最大 **65535 字节**
- 发送给频道内所有服务端/客户端模组

> 说明：频道不可写或无效时，消息不会发送
