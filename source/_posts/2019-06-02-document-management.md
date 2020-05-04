---
layout: post
title: 文档/知识库软件推荐
category: 闲谈
tags:
  - 文档管理
---
我们曾经使用源代码版本控制软件来组织文档，然而版本控制软件的长项是管理代码，而非管理文档。用作档案库还算凑合，但是管理和检索知识并不方便，因此考虑了一些文档软件。大家可根据实际需求选用不同的系统。
<!-- more -->

本文有很多系统需要自行搭建。从信息安全的角度讲，建议内部文档不要使用现成的服务，而是自行准备服务器，并做好安全防范措施。从操作的角度上将，建议使用Docker部署，操作便捷。

## 博客
博客的特点是“流水账”，文章主题不固定，彼此之间关系也不紧密，访问也没有限制（当然你仍然可以在服务器上面进行限制），适合公开分享个人/项目经历以及经验教训之类的内容。

### WordPress
* 格式：富文本
* 官网：[https://wordpress.org](https://wordpress.org)和[https://wordpress.com](https://wordpress.com)（前者是软件，后者是把软件搭好的现成服务）
* 部署：需自行搭建
* 最流行的博客系统，不需要再详细叙述。文档可以直接在系统里面维护和发布，操作非常便捷。

### Hexo
* 格式：Markdown
* 官网：[https://hexo.io](https://hexo.io)
* 部署：需自行搭建Web Server，或者放到GitHub Pages/GitLab Pages等里面。每次更改文档需要重新生成代码和部署。
* 本博客就是用Hexo搭建的。但是，为了使博客运行，需要至少准备一个Web Server。想要玩得转的话，需要把源代码放在Git上面，还要需要配置CI，使用时还要先Git Pull再Git Push，对于大家来说实在太麻烦，不推荐。

## 维基和书籍类
这两类适合存放成体系的内容，检索起来也比较方便，可以用来建设知识库、资料库等。

### MediaWiki
* 格式：富文本（需使用MediaWiki语法）
* 官网：[https://mediawiki.org](https://mediawiki.org)
* 部署：需自行搭建
* 该软件最初就用于维基百科，适合构建百科类网站，组织知识零碎、有一定范围但各内容彼此之间关系不密切的内容。
* 软件有三个值得注意的缺点：
  - 学习成本比较高，需要熟悉维基语法和系统；
  - 从零开始构建比较麻烦，需要设计一些模板，即使从维基百科导入工作也不轻松；
  - 使用时需要明确内容编写与分类规范，否则很容易就写乱套了。

### BookStack
* 格式：富文本或Markdown，取决于网站设置
* 官网：[https://bookstackapp.com](https://bookstackapp.com)
* 部署：需自行搭建
* 该软件适合存放成体系、内容彼此之间关联密切的内容，例如书籍、手册等。我们目前正在使用此软件管理资料库。

## 笔记
Evernote、OneNote、Notable等软件的重点是个人笔记，本文不再介绍。

### Etherpad
* 类型：纯文本或富文本（取决于网站设置）
* 官网：[https://etherpad.org](https://etherpad.org)
* 部署：需自行搭建。个人建议直接放自己电脑上跑，需要的时候才启动。
* 该软件适合多人同时编辑同一个文件，而且彼此之间可以看到谁在修改哪些内容，建议协作时使用。

## 粘贴箱类
粘贴箱类适合临时分享一些小东西。

### Ubuntu Pastebin
* 类型：仅纯文本（代码）
* 网站：[https://paste.ubuntu.com](https://paste.ubuntu.com)
* 该网站适合临时分享一些文字或代码。想象一下用聊天软件给对方发东西时，是发一大坨代码（有时候系统可能还给自动转成表情符号）好看呢，还是发个小链接（网站顺便会给格式化一下）好看呢？

### GitHub Gist
* 类型：一个或多个代码文件
* 网站：[https://gist.github.com](https://gist.github.com)
* 该网站适合分享一些零碎的代码片段，或者一些小到不值得开repo的程序。不过该网站已经被墙，所以分享时需要保证他人也能访问该网站。

### img.vim-cn.com
* 类型：图片
* 网站：[https://img.vim-cn.com/](https://img.vim-cn.com/)
* 该网站可用于临时分享图片。如果处于不能或者不便发图的场合，可以先把图片传到这里，再去发图片的链接。

### Firefox Send
* 类型：文件
* 网站：[https://send.firefox.com/](https://send.firefox.com/)
* 该网站可用于临时分享文件。同样是发送成功之后取一个下载链接。

## 其他软件附带的功能
### GitHub/GitLab
* GitHub Pages可以通过Git仓库运行静态网站，这样的话，你可以利用Hexo等软件维护一个博客，然后部署到GitHub上，从而运行文档库。另外请注意GitHub Pages的内容是公开的。
* 每个Git项目都有Wiki功能，使用Markdown语法，但只适合存放规模不大、层次简单的文档。

GitLab也有类似的功能。

### Phabricator
Phabricator是一个项目管理平台，包含了很多模块，而且各模块之间可以无缝衔接。然而，我们认为这些都是Phabricator顺带的东西，不值得专门为了管理文档而新搭Phabricator。

以下是Phabricator中与文档相关的几个模块：
* Phame：博客
* Phriction：Wiki，适合分享规模不大、内容有组织的文档
* Paste：代码粘贴箱
* File：文件分享（备注：一般附件也会自动保存在该应用中）

这些功能全部使用Phabricator自己的语法，另外可以直接和P站其他资源链接，例如在Blog中直接输入T145就能自动链接到该任务。
