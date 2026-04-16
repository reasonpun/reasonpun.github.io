---
title: luanti HTTP API
date: 2026-04-16 16:25:05
categories:
- luanti
tags:
- luanti
- games
- mods
- API
---

# HTTP API
HTTP API 允许模组发起网络请求（GET/POST 等）。

## 设置（必须配置）
要使用此 API，模组必须在以下**任一配置项**中被列出（使用 CSV 格式，即逗号分隔）：
- `secure.trusted_mods`
- `secure.http_mods`

<!--more-->

示例：
```
secure.http_mods = modname1, modname2, modname3
```

---

## 数据类型

### HttpRequest
一个表（table），包含以下参数：

- **`url`**（必填）
  类型：字符串
  示例：`https://subdomain.domain.tld`

- **`timeout`**（可选）
  单位：秒
  默认值为配置项 `curl_timeout` 的值（注意：`curl_timeout` 单位是毫秒）

- **`method`**（可选）
  类型：字符串
  支持：`GET`、`POST`、`PUT`、`DELETE`
  默认：`GET`

- **`data`**（可选）
  支持：字符串 或 表
  如果传入表，会被自动编码为 `x-www-form-urlencoded` 格式的键值对

- **`post_data`**（可选）
  已废弃，请使用上面的 `data`

- **`user_agent`**（可选）
  类型：字符串
  如果提供，将覆盖默认的 User-Agent

- **`extra_headers`**（可选）
  字符串组成的表
  每个字符串必须符合 HTTP 标准格式：`"键: 值"`

- **`multipart`**（可选）
  布尔值，默认为 false
  必须在 `method` 为 `POST` 时使用

---

### HttpRequestResult
请求结果表，包含以下字段：

- **`completed`**
  布尔值
  请求是否结束（无论成功、失败、超时）

- **`succeeded`**
  布尔值
  请求是否成功

- **`timeout`**
  布尔值
  是否超时

- **`code`**
  整数
  HTTP 状态码（如 200、404、500）

- **`data`**
  表 或 字符串
  服务器返回的响应内容

---

## 函数

### `core.request_http_api()`
获取 HTTP API 对象。

只有满足以下条件时才会返回有效对象：
1. 模组已被加入 `secure.http_mods` 或 `secure.trusted_mods`
2. 引擎编译时包含 curl 支持
3. 此函数**必须在模组的 init.lua 顶层调用**，不能在其他函数内部调用

⚠️ **重要警告**
请将返回值存为**局部变量（local）**，**不要存为全局变量**，防止其他模组非法访问。
每个模组应独立申请自己的 HTTP API 权限。

使用示例（判断 API 是否存在、是否授权）：
```lua
-- 在模组 init.lua 中
local http = core.request_http_api and core.request_http_api()

if http then
    -- 在这里调用 HTTP 方法
end
```

---

### `HttpApi.fetch(request, callback)`
发起网络请求，完成后调用回调函数。
**推荐大多数场景使用**。

- `request`：类型为 `HttpRequest`
- `callback`：回调函数，接收一个 `HttpRequestResult` 类型参数

---

### `HttpApi.fetch_async(request)`
发起异步请求，返回一个句柄（handle）。
需配合下面的 `fetch_async_get` 使用。

---

### `HttpApi.fetch_async_get(handle)`
传入 `fetch_async` 返回的句柄，获取请求结果。

---

# 教学用途说明（给你备注）
- HTTP API 可用于：**联网题库、AI 对话 NPC、在线成绩上报、云端任务**等教学功能。
- 必须在配置文件中开启 `secure.http_mods`，否则模组**无权访问网络**。
- 适合做：**AI 助教 NPC、联网答题、云端作业提交**等高级教学案例。
