---
title: 给Luanti文档添砖加瓦
date: 2026-04-09 08:30:34
categories:
- luanti
tags:
- luanti
- games
- mods
---

咱这 Luanti 文档全是用 **Markdown** 写的，靠 **Hugo**（免费开源静态网站生成器）转成网页。源码搁 **GitHub** 上，用 **GitHub Actions** 自动发布。

<!--more-->

---

## 本地开搞（本地开发）
克隆仓库必须**递归**，把主题子模块一起拉下来：
```bash
git clone --recursive https://github.com/luanti-org/docs.luanti.org
```

已经克隆过？补一下子模块：
```bash
git submodule init
git submodule update --remote
```

这项目用 **Hugo** 建站，一堆 **Node 包**做校验。
你可以直接把 Hugo 当 Node 包装，省事；Node 还用来跑拼写检查、无障碍校验等脚本，都写在 `package.json` 和 `readme.md` 里。

用 Node 安装+运行：
```bash
npm install  # 装依赖
npm start    # 编译+本地预览
```

想手动装也行：去下**扩展版 Hugo**，版本必须跟 `hugo.yaml` 里一致。
直接跑 Hugo：
```bash
hugo server  # 就是 npm start 里藏的命令
```

---

## 拼写检查器
跑拼写检查：
```bash
npm run test:spelling
```

推荐 VS Code 装 **Code Spell Checker** 插件，实时标错词。
但插件不一定全覆盖，最终以 `npm run test:spelling` 为准。

遇到词拼对了但被误报？
右键 → **Spelling → Add Words to CSpell Configuration**，
或者手动往 `cspell.json` 里加词。

想临时关掉拼写检查？用 Hugo 注释：
```markdown
这段会检查拼写。

{{% comment %}} cspell:disable {{% /comment %}}

这段老子就不拼对～

{{% comment %}} cspell:enable {{% /comment %}}

这段又恢复检查了。
```

---

## 提交修改（提 PR 走你）
每页底部都有 **「Edit this page」** 按钮，点它直接进 GitHub 网页编辑器改。
或者在仓库页按一下 **`.`** 键，秒开网页版 VS Code。

修改流程标准**Fork + Pull Request（PR）**，不懂就去看快速指南。
有问题去 IRC/Discord/Matrix 的 **#luanti-docs** 频道喊人。

### 网页版操作
1. 先 Fork 仓库（点「Edit this page」会引导你）
2. 进仓库页按 `.` 开网页版 VS Code
3. 左下角切个新分支
4. 开改
5. 左边点「源代码管理」
6. 写提交信息 → **Commit and Push**

### 本地版操作
```bash
git clone https://github.com/你的用户名/docs.luanti.org
git checkout -b 你的分支名
# 随便用编辑器改
git add -A && git commit -m "写你改了啥" && git push
```
然后去 GitHub 提 PR 回主仓库。
等着文档组审核就行，我们**基本每天都会 Review**。

---

## 规矩要懂（指南）
### 必须加 Front Matter
每篇文件顶部都要有**三段式横线包裹**的头部元数据，YAML 格式。
```yaml
---
title: 你这页的标题
aliases:
  - /旧/页面/路径
---
```
- **title 必填**
- 目录下的 `_index.md` 必须加 `bookCollapseSection: true`
- 移动/重命名文件，一定要加 `aliases` 做跳转，保住老链接

### 格式要求
- 用**美式英语**拼写
- 内容用 **Markdown** 写
- 图片统一用 **WebP** 格式，别用 jpg/png 那些
- 代码块必须标**语言**

---

## 合并规矩（Merge Policies）
- 改东西先去 **#luanti-docs** 频道说一声（可简单提一句）
- `/for-creators/api` 下的内容：必须**另一个文档组成员**批准
- 引擎核心流程相关：至少**一个引擎组成员**批准（不能是你自己）
- 其他内容：文档组成员可**酌情自行合并**
-  trivial 小改动（错别字、格式）：豁免上面两条

"""

# 保存为文件
with open("/mnt/luanti_docs_contributing_translation.md", "w", encoding="utf-8") as f:
    f.write(translation)

print("翻译完成！文件已保存至：/mnt/luanti_docs_contributing_translation.md")
print("\n=== 翻译预览 ===")
print(translation[:1000] + "...")

# 给 Luanti 文档添砖加瓦（诙谐接地气版）
咱 Luanti 文档全是用 **Markdown** 写的，靠 **Hugo**（免费开源静态网站生成器）转成网页。源码搁 **GitHub** 上，用 **GitHub Actions** 自动发布。

---

## 本地开搞（本地开发）
克隆仓库必须**递归**，把主题子模块一起拉下来：
```bash
git clone --recursive https://github.com/luanti-org/docs.luanti.org
```

已经克隆过？补一下子模块：
```bash
git submodule init
git submodule update --remote
```

这项目用 **Hugo** 建站，一堆 **Node 包**做校验。
你可以直接把 Hugo 当 Node 包装，省事；Node 还用来跑拼写检查、无障碍校验等脚本，都写在 `package.json` 和 `readme.md` 里。

用 Node 安装+运行：
```bash
npm install  # 装依赖
npm start    # 编译+本地预览
```

想手动装也行：去下**扩展版 Hugo**，版本必须跟 `hugo.yaml` 里一致。
直接跑 Hugo：
```bash
hugo server  # 就是 npm start 里藏的命令
```

---

## 拼写检查器
跑拼写检查：
```bash
npm run test:spelling
```

推荐 VS Code 装 **Code Spell Checker** 插件，实时标错词。
但插件不一定全覆盖，最终以 `npm run test:spelling` 为准。

遇到词拼对了但被误报？
右键 → **Spelling → Add Words to CSpell Configuration**，
或者手动往 `cspell.json` 里加词。

想临时关掉拼写检查？用 Hugo 注释：
```markdown
这段会检查拼写。

{{% comment %}} cspell:disable {{% /comment %}}

这段老子就不拼对～

{{% comment %}} cspell:enable {{% /comment %}}

这段又恢复检查了。
```

---

## 提交修改（提 PR 走你）
每页底部都有 **「Edit this page」** 按钮，点它直接进 GitHub 网页编辑器改。
或者在仓库页按一下 **`.`** 键，秒开网页版 VS Code。

修改流程标准**Fork + Pull Request（PR）**，不懂就去看快速指南。
有问题去 IRC/Discord/Matrix 的 **#luanti-docs** 频道喊人。

### 网页版操作
1. 先 Fork 仓库（点「Edit this page」会引导你）
2. 进仓库页按 `.` 开网页版 VS Code
3. 左下角切个新分支
4. 开改
5. 左边点「源代码管理」
6. 写提交信息 → **Commit and Push**

### 本地版操作
```bash
git clone https://github.com/你的用户名/docs.luanti.org
git checkout -b 你的分支名
# 随便用编辑器改
git add -A && git commit -m "写你改了啥" && git push
```
然后去 GitHub 提 PR 回主仓库。
等着文档组审核就行，我们**基本每天都会 Review**。

---

## 规矩要懂（指南）
### 必须加 Front Matter
每篇文件顶部都要有**三段式横线包裹**的头部元数据，YAML 格式。
```yaml
---
title: 你这页的标题
aliases:
  - /旧/页面/路径
---
```
- **title 必填**
- 目录下的 `_index.md` 必须加 `bookCollapseSection: true`
- 移动/重命名文件，一定要加 `aliases` 做跳转，保住老链接

### 格式要求
- 用**美式英语**拼写
- 内容用 **Markdown** 写
- 图片统一用 **WebP** 格式，别用 jpg/png 那些
- 代码块必须标**语言**

---

## 合并规矩（Merge Policies）
- 改东西先去 **#luanti-docs** 频道说一声（可简单提一句）
- `/for-creators/api` 下的内容：必须**另一个文档组成员**批准
- 引擎核心流程相关：至少**一个引擎组成员**批准（不能是你自己）
- 其他内容：文档组成员可**酌情自行合并**
- 小改动（错别字、格式）：豁免上面两条
