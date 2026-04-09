---
title: luanti creating-texture-packs
date: 2026-04-09 09:45:58
categories:
- luanti
tags:
- luanti
- games
- mods
---

# 制作材质包
材质包就是一个**装图片的文件夹**，文件名和原版保持一致就能替换贴图。Luanti 对材质包要求极低，基本是个游戏都能用。

<!--more-->

---

## 图片结构（5.1 版本）
### 引擎自带纹理
Luanti 本身自带的基础纹理很少，放在这里：
`你的路径/luanti/textures/base/pack`
包括：服务器图标、小地图、Logo、加载条等。

### 游戏专属纹理
每个游戏用的模组不一样，纹理也不一样。
比如 Minetest Game 5.1 自带 31 个模组，有的有纹理，有的没有。
举个例子：
`games/minetest_game/mods/default/textures` 有大量纹理
`games/minetest_game/mods/spawn` 则一个纹理都没有

### 引擎 + 游戏完整材质包
想做一套完整的自定义材质包，你需要：
1. 把 `base/pack` 里的引擎纹理全复制到新文件夹
2. 把游戏每个模组里的 `textures` 全复制到同一个文件夹

---

## 搭建自定义材质包文件夹
### 新建文件夹
在 `你的路径/luanti/textures` 下新建文件夹，名字就是你的材质包名。
目录里自带一个 `texture_packs_here.txt` 告诉你位置对不对。

---

## 复制纹理（简单版）
### Windows
1. 打开 Luanti 目录
2. 进入 `games/minetest_game`
3. 搜索 `*.png`
4. Ctrl+A 全选 → 复制 → 粘贴到你的材质包文件夹

想替换引擎基础纹理：
去 `textures/base/pack` 复制这些：
- `logo.png` —— 主界面图标
- `minimap_*.png` —— 小地图
- `unknown_*.png` —— 未知物体
- `server_*.png` —— 服务器图标
- `progress_*.png` —— 加载条

### Linux
终端一键复制：
```bash
TP_NAME=你的材质包名
cd 你的路径/luanti
mkdir textures/$TP_NAME
find games/minetest_game/ -name '*.png' -exec cp '{}' "textures/$TP_NAME" ';'
```

---

## 复制纹理（复杂版，保留模组结构）
### Windows（用 robocopy）
```cmd
robocopy "C:\你的路径\luanti\games\minetest_game\mods" "C:\你的路径\luanti\textures\你的包名" *.png /s
```

复制引擎纹理：
```cmd
robocopy "C:\你的路径\luanti\textures\base" "C:\你的路径\luanti\textures\你的包名" *.png /s
```

结构会变成：
```
你的包名/
├─ default/
│  └─ textures/ *.png
```

你可以用这个批处理把文件上提一级：
```bat
@echo off
set thisdir=%cd%
for /f "delims=" %%B in ('dir /b /ad') do (
    cd "%%~dpnB"
    for /f "delims=" %%C in ('dir /b /ad') do (
        cd "%%~dpnC"
        move *.png ..\
        cd..
        rd "%%~dpnC"
    )
    cd..
)
cd /d "%thisdir%"
```
保存成 `.bat` 放在材质包文件夹里双击运行。

### Linux
自己看着办，不会就给文档组提 issue。

---

## 特殊纹理
Luanti 有一些**特殊用途纹理**，完整列表看源码里的 `texture_packs.md`。

---

## 编辑 / 制作纹理
### 版权问题（非常重要）
- 偷别的体素游戏的纹理**绝对不行**，上传 ContentDB 直接被拒
- 必须用**免费授权**的图片（CC 系列为主）
- 即使 CC0 也要标注来源，基本礼貌

### 画图工具推荐
- **GIMP**：全能免费画图工具，有点难但很强
- **Libresprite**：专门画像素画
- **Paint.NET**：Windows 好用小工具
- Lospec 像素工具网站

### 怎么画
- 尺寸最好是**2的次方**：16×16、32×32、64×64、128×128、256×256
- 超过 512×512 性能会崩，没必要
- 格式优先 **PNG**，也支持 JPG、BMP、TGA

---

## 从哪开始做？
- 先给小模组做材质，试试水
- 优先做 `default` 模组，约 250 张贴图，影响最大
- 难点：动画纹理（火把、水）
- 简单：木板、树叶（改个色就行）

---

## 发布前准备
1. 加 `license.txt` 写清楚授权
2. 截几张游戏内效果图
3. 新建 `texture_pack.conf` 写信息：
```
name = very_cool_texture_pack
title = 超酷材质包
description = 这是一个超酷的材质包
```

---

## 发布渠道
### Git 托管
- GitHub
- Codeberg
- Gitlab

### 官方平台
**ContentDB** —— 官方发布渠道，游戏内可直接安装。
