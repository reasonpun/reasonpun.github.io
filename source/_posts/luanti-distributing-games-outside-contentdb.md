---
title: luanti 脱离 ContentDB 分发游戏
date: 2026-04-09 15:28:51
categories:
- luanti
tags:
- luanti
- games
- mods
---

# 脱离 ContentDB 分发游戏
这篇教你**不依赖官方商店**，把你的 Luanti 游戏打包成独立客户端发布。
前提：你得会从源码编译 Luanti。

Luanti 引擎本身**没有官方一键导出**功能，游戏默认都走 [ContentDB](https://content.luanti.org/packages/?type=game)、在游戏内安装。但因为 Luanti 是自由软件，你可以**fork 引擎 → 改头换面 → 绑定你的游戏 → 独立发布**。

<!--more-->

---

## 通用定制技巧
### 自定义主菜单
主菜单代码在：`builtin/mainmenu/`
用 Lua + Formspec 随便改，想改成啥样都行。

### 锁定单机模式为你的唯一游戏
≥ 5.8.0-dev 版本：引擎会自动识别已装游戏，并默认选第一个。
你只带自己的游戏，它就会默认选中。

**彻底隐藏游戏选择栏：**
修改 `builtin/mainmenu/tab_local.lua`，**删掉游戏栏循环代码**，并把按钮尺寸设为 0：
```lua
-- 把这一大段全删掉
for _, game in ipairs(pkgmgr.games) do
    ...
end

-- 尺寸改成 x=0,y=0
{x=-0.3,y=5.9}, "horizontal", {x=0,y=0})
```

### 主菜单换皮
把你游戏的 `menu/` 目录图片复制到：`textures/base/pack/`
- `header.png` → 重命名为 `menu_header.png`
- `background.png` → 重命名为 `menu_bg.png`
- `icon.png` → 重命名为 `logo.png`

### 给“关于”页面加信息
必须保留 **Luanti 版权声明**（遵守协议），但你可以把你的开发团队放在上面。
建议在 Luanti 声明上方加一句：
```lua
local minetest_info = {
    "<游戏名> 基于 Luanti 引擎开发，",
    "该引擎是 LGPLv2.1 许可下的自由软件。",
    ""
}
```

### 过滤服务器列表（只显示你的游戏服务器）
修改 `builtin/mainmenu/serverlistmgr.lua`，加入**游戏 ID 过滤**：
```lua
-- 只保留你的游戏服务器
local kiosk_game = "你的游戏ID"
for _,entry in pairs(retval.list) do
    if entry.gameid == kiosk_game then
        table.insert(list, entry)
    end
end
```

### 过滤 ContentDB 内容（只显示适配你游戏的模组）
修改 `builtin/mainmenu/dlg_contentstore.lua`，在 API 地址末尾加上：
```
&game=作者名/游戏包名
```
比如：`&game=Warr1024/nodecore`

### 修改默认设置
编辑 `src/defaultsettings.cpp` 改默认配置。
安卓专用配置在 `#ifdef __ANDROID__` 代码块里。

---

## Windows 定制
- 改引擎名：修改 `CMakeLists.txt` 里的项目名，会改变 exe 名称、标题栏等
- 改图标：替换 `misc/minetest-icon.ico`
- 绑定你的游戏：在 CMakeLists 里把安装 `minetest_game` 的行，改成安装**你的游戏**
- 可删掉：lua_api.txt、devtest 等不需要的文件

---

## Android 定制
前提：你会编译安卓版、会签名 APK。

### 改包名（避免和官方 Luanti 冲突）
- 修改 `android/app/build.gradle` 里的 `applicationId`
- 同步修改 `AndroidManifest.xml` 和 `GameActivity.java` 里的 **file provider** 名称

### 换名、换图标
- 应用名：修改 `res/values/strings.xml` 里的 `label`
- 图标：替换 `mipmap/ic_launcher.png`
- 绑定游戏：修改 `build.gradle` 里的 `gameToCopy` 变量（默认是 minetest_game）
- 加载背景：替换 `drawable/background.png`

---

## Linux 定制
编辑 `misc/net.minetest.minetest.desktop` 改桌面图标、名称、启动项，让系统更好集成。

---

## macOS 定制
TODO（官方还没写）

---

## 授权许可（非常重要）
- 引擎是 **LGPLv2.1**：你对引擎源码的任何修改**必须公开**
- 游戏本身可以**闭源、商用**（只要不用其他 copyleft 模组）
- 这不是法律建议，重要项目最好咨询律师

### 发布源码最简命令
```bash
stashName=`git stash create`; git archive $stashName | gzip > ../游戏名引擎-日期.tar.gz
```

---

## 示例参考
NodeCore 安卓版就是按这篇教程改的，可当范例参考（版本略旧）。
