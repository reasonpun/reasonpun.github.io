---
title: 如何设计一个有吸引力的GitHub简介
date: 2022-08-06 07:47:45
categories:
- GitHub
tags:
- GitHub
- Profile
---

{% asset_img 1.jpeg 示意图 width="400" %}

如何创建一个具有视觉吸引力的GitHub配置文件自述。你们都知道会有这样的效果吧？
你们中的一些人可能会想："唉，太好了，又是一篇关于如何制作一个令人印象深刻的GitHub简介的博客😑"。
嗯......是的。

<!--more-->

但仅仅因为某些东西已经存在，并不意味着你不能创造自己独特的版本。
大多数人都会把他们的简介的自述写得很有趣，
但看起来不错的简介和有美感的简介之间是有差别的！

所以在这里，我将向你们展示如何设计一个明星级的GitHub简介自述，我将以我的简介为例。
我会提到我的简介所使用的所有资源，以及你如何根据你的风格来定制它。

但在我们进入简介的设计和风格之前，你们中的许多人可能会问 "这样做的原因是什么？"和 "有必要吗？"。
对此，我想说，没有必要为你的简介设计风格并使其独特。
不创建readme简介没有什么大的坏处，但肯定有很多好处。

每个人都把GitHub当作他们项目的储藏室，只是简单地使用它的设计，也就是源代码控制，以及与他人合作开展项目或为开源做贡献。
但在2020年，当GitHub发布了创建我们自己独特的readme profile的新功能时，它为开发者和艺术家提供了一种方式，在GitHub上以 "个人简历 "的形式专业地展示他们的工作。
这就是它的全部：一个特殊的 repo，作为一个视觉上令人印象深刻的组合，供其他开发者和雇主查看。
因此，请继续阅读一些关于GitHub简介风格的难以置信的提示吧

1. 最开始的步骤 🐤
要开始设计你的 GitHub 配置文件，我们首先要创建一个新的公共仓库。
点击右上方的 "+"图标，选择 "新仓库"。
之后就是关键的一步，确保仓库的名称与你的用户名相同。
请参考下面的例子：

{% asset_img 2.jpg 示意图 width="400" %}

GitHub 会让你知道你找到了一个特殊的 repo，其 README.md 文件可以被定制。
我们希望保持这个 repo 的 "公开性"，以便它能显示在你的 GitHub 配置文件上。你可以提供该仓库的简要描述（例如 "我的 GitHub 配置文件"），不过这一步是可选的。
之后，勾选 "添加 README 文件"，并点击 "创建仓库"。接下来，我们将修改这个 README 文件并使其个性化。

{% asset_img 3.jpg 示意图 width="400" %}

2. 一个独特的标题 ❄️
你的资料标题是人们观察的第一件事，所以它必须从其他资料中脱颖而出。
我们希望最初的 "钩子 "能够吸引浏览者。
要做到这一点，我的建议是避免遵循常见的设计规范。
例如，许多开发者在他们的 "关于 "部分使用这种布局。

```
### Hi there 👋
* 👂 My name is ...
* 👩 Pronouns: ...
* 🔭 I’m currently working on ...
* 🌱 I’m currently learning ...
* 🤝 I’m looking to collaborate on ...
* 🤔 I’m looking for help with ...
* 💬 Ask me about ...
* 📫 How to reach me: ...
* ❤️ I love ...
* ⚡ Fun fact: ...
```

使用这个模板是完全可以的，只要你改变你的个人资料的其他方面。
我从另一个方向来创建标题，然后在这之后添加 "关于我 "部分。
我将用我所用的资源来指导你。

你可以看到的第一件事就是那个带有 "嘿，大家好！"文字的动画标题。
我使用了[capsule-render GitHub repo](https://github.com/kyechan99/capsule-render)来做这个。我在寻找装饰GitHub repo的方法时发现了这个伟大的资源。你可以在上面添加背景图片和文字，还有，谁不喜欢动画呢！它的使用非常简单，而且还能让你的网站变得更漂亮。它使用起来超级简单，而且在Repo上有很好的记录。下面是我对渲染器的配置。

```
<p align="center">
  <img src="https://capsule-render.vercel.app/api?text=Hey Everyone!🕹️&animation=fadeIn&type=waving&color=gradient&height=100"/>
</p>
```

在插入一个简单的标题后，提供我的各种账户的链接，如LinkedIn、Medium、Dev.to，我想用一种简约的、无文字的方式来做。
因此，我决定使用图标。
有许多在线工具可以提供成千上万的免费图标供使用:我使用了IconFinder，个人很喜欢它。
还有许多其他流行的选择，你可以使用，如Shields.io、markdown-badges、vector-logo-zone、simple-icons等等。

图标的png来源只需要导入到<img>标签中，如下图所示。

```
<a href="https://www.instagram.com/thepiyushmalhotra/">
  <img height="50" src="https://user-images.githubusercontent.com/46517096/166974368-9798f39f-1f46-499c-b14e-81f0a3f83a06.png"/>
</a>
```

现在到了有趣的部分，添加了那个NIU B 的 GIF！
GIFS 使我们的个人资料更加动态和引人注目。
老实说，你可以放任何你想要的 GIF。它可以是流行的表情包、编程 gif、电影或电视节目中的标志性场景，或者告诉人们一些关于你的爱好的东西。
就我而言，这是动漫，所以我就是这么做的。 Giphy 和 Tenor 等流行的 gif 共享网站可用于提取您喜欢的任何 gif，其工作方式与添加图标相同，只需复制图像地址并将其粘贴到 <img> 标签的“src”属性中。

3. 关于我 "部分 👨💻
这是我前面谈到的部分，大多数开发者使用上面的模板。
如果你想让你的简介脱颖而出，那么我建议也改变这一部分的设计。我在编辑readme时，继续使用YAML格式，这样当你预览简介时，信息读起来就像代码。

{% asset_img 5.jpg 示意图 width="400" %}

它增加了一丝专业性，而且作为奖励，看起来也很整洁! 要显示这种格式，只需将你的文本包起来，如下图所示，你就可以了。

```yaml
* YOUR TEXT GOES HERE *
```

4. 工具和技术的东西 🧰
在这一部分，你可以展示你的技能，并列出你所熟悉的工具和技术。
我总是喜欢简约明快的设计选择，而不是杂乱无章的数据，所以这次我也采用了图标。
我们人类更喜欢通过视觉媒介获取信息，而不是其他东西，对吗？

你可以使用我在第二步中提到的所有东西，如IconFinder、Shields.io、markdown-badges、vector-logo-zone、simple-icons等。但是对于这一部分，我个人推荐DevIcon。
与其他资源不同，DevIcon是专为提供与编程语言和开发工具有关的图标而建立的，这使它成为一个完美的选择。

{% asset_img 4.png 示意图 width="400" %}

只要从DevIcon的网站上复制SVG图像源，并将其粘贴在<p>标签内，就可以显示多个图标了!

```
<h2> 🚀 &nbsp;Some Tools I Have Used and Learned</h2>
<p align="left">
<img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/vscode/vscode-original.svg" alt="vscode" width="45" height="45"/>
<img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/bash/bash-original.svg" alt="bash" width="45" height="45"/>
<img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/php/php-original.svg" alt="php" width="45" height="45"/>
</p>
```

5. 你的 GitHub 历史 📈
最后，在你的简介README的结尾，你几乎可以包括任何东西。
有些开发者会在自己的Spotify简介中写上当前正在播放的音乐，有些会添加他们的GitHub统计数据，有些则会在你的GitHub贡献图上添加一个有趣的小蛇游戏.
就像我一样，我将向你们展示如何添加这些内容:

{% asset_img 6.jpg 示意图 width="400" %}

我先用两张GitHub ReadMe统计卡。一张显示我的星星总数、提交和拉动请求等。
另一张则显示我最常用的编程语言的百分比。你们可以从流行的GitHub ReadMe Stats Repo中获得这些卡片，这些卡片最好的部分是它们可以通过不同的设置和主题进行完全定制。

接下来可能是我所有资料中最喜欢的东西了。把你的GitHub贡献图谱做成一个蛇形游戏。它的设置相当简单，当蛇吞噬你的提交图时，看起来非常令人满意。

为了给你的档案设置，我们将使用一个叫做GitHub Actions的东西。GitHub Actions是GitHub中的CI/CD工具，你可以启动工作流程，自动运行、部署和构建你的东西。

```
![Snake animation](https://github.com/reasonpun/reasonpun/blob/output/github-contribution-grid-snake.svg)
```

 * 第一步是复制上面这一行，并将其添加到你的个人资料的README中。确保把用户名改成你的，而不是我的。
 * 现在我们需要创建一个GitHub工作流，这样蛇形动画中的贡献图就会根据我们将要设置的cronjob进行更新。
 * 进入README仓库的 "行动 "标签，创建一个新的工作流。这将在你的版本库中生成一个新的文件夹，名为".github/workflows"，之后，它将在其中生成一个新文件，名为 "main.yml"。

{% asset_img 8.jpg 示意图 width="400" %}

{% asset_img 9.jpg 示意图 width="400" %}

删除新创建的main.yml文件中的所有内容，并在下面添加这段代码。

```
name: Generate Datas
on:
  schedule: # execute every 12 hours
    - cron: "* */12 * * *"
  workflow_dispatch:
jobs:
  build:
    name: Jobs to update datas
    runs-on: ubuntu-latest
    steps:
      # Snake Animation
      - uses: Platane/snk@master
        id: snake-gif

        with:
          github_user_name: thepiyushmalhotra
          svg_out_path: dist/github-contribution-grid-snake.svg
      - uses: crazy-max/ghaction-github-pages@v2.1.3
        with:
          target_branch: output
          build_dir: dist
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

请确保将我的用户名替换成你的。
我们在这里使用一个cronjob，每12小时更新一次，所以只要你有一个新的提交，它就会被添加到你的蛇形动画中。
最后一步是回到你的README文件的 "行动 "页面，点击新创建的工作流 "生成数据 "或你给它起的任何名字，并点击 "运行工作流 "按钮。

{% asset_img 10.png 示意图 width="400" %}

哇！你的 "蛇 "GitHub贡献图谱现在已经激活了。
尽情欣赏那条蛇吃掉你的辛勤工作吧! 

<!-- https://bootcamp.uxdesign.cc/how-to-design-an-attractive-github-profile-readme-3618d6c53783 -->