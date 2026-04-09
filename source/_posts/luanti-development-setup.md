---
title: luanti development-setup
date: 2026-04-09 10:05:23
categories:
- luanti
tags:
- luanti
- games
- mods
---

# 开发环境配置

## 前言
写模组随便用啥编辑器都行，但强烈建议用支持 [**LSP（语言服务器协议）**](https://microsoft.github.io/language-server-protocol/) 的编辑器，能自动补全、报错误，开发爽到飞起。

支持 LSP 的常见编辑器：
- Visual Studio Code
- Kate
- NeoVim
- Helix

<!--more-->

---

## 安装语言服务器
不同编辑器、系统安装方式不一样：
- **VS Code**：装 [Lua 插件](https://marketplace.visualstudio.com/items?itemName=sumneko.lua)，自带语言服务器
- **NeoVim**：用 mason 或其他 LSP 插件装
- 其他不自带的编辑器：需要在系统层面[手动安装](https://luals.github.io/#install)

---

## 配置自动补全
### 下载类型注解文档
类型注解能让编辑器**精准提示 Luanti API**，带参数类型、返回值提示。
我们用 [**luanti-api**](https://git.minetest.land/andro/luanti-api) 这个非官方实验性注解库。

不想用这个也可以直接用引擎自带的 `<Luanti安装目录>/builtin`，只是没有类型提示。

配置补全有两种方式：

---

### 方式1：全局配置（推荐）
1. 下载 `luanti-api` 仓库（git 克隆或下载 zip）
2. 放到电脑任意位置，比如：`/home/user/luanti-api`
3. 配置 Lua 语言服务器，把路径加到 `workspace.library`

VS Code 示例（在 `settings.json` 里加）：
```json
{
  "Lua.workspace.library": ["/home/user/luanti-api"]
}
```

---

### 方式2：按项目配置
1. 初始化 git 仓库
   ```bash
   git init
   git submodule init
   ```
2. 添加 luanti-api 为子模块
   ```bash
   git submodule add https://git.minetest.land/archie/luanti-api.git
   ```
3. 在项目根目录新建 `.luarc.json`：
   ```json
   {
     "$schema": "https://raw.githubusercontent.com/LuaLS/vscode-lua/master/setting/schema.json",
     "workspace.library": ["./luanti-api"]
   }
   ```

搞定！编辑器就会**全自动提示 Luanti 所有 API** 了。
