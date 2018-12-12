---
layout: post
title: 从Eclipse切换到IDEA（一）：切换操作
category: 教程
tags: 
- IDEA
---
我们项目组之前一直使用Eclipse进行开发。对于我们项目而言，由于Eclipse存在现实的问题，我便开始研究IDEA的用法，并且试着将工作切换到IDEA上面，之后逐步地在项目组内推广使用IntelliJ IDEA。本文面向Eclipse使用者，讲述IDEA的基本操作，以便平稳地进行切换。
<!--more-->

备注：我们项目组采用SVN版本控制，未使用Maven、Gradle等构建工具，程序代码的字符编码是GB18030。

# 切换到IDEA的理由
下面是我向同事推荐使用IDEA（商业版）的一些理由：

1. 公司版本管理软件从ClearCase切换到了SVN，这意味着开发工作不再受ClearCase拖累，开发工具也可以随意换成更好用的了。
2. 公司电脑配置不高，而且项目文件非常多，Eclipse用起来很卡。虽然IDEA资源消耗比较高，但是用起来比Eclipse顺畅得多（备注：对于内存只有4GB的电脑，要么加条，要么换旧版IDEA，否则会很糟糕）。
3. IDEA的搜索比Eclipse快得多，按全项目搜索的话，Eclipse要等把所有文件都过一遍，而IDEA可以一下子搜出来。
4. IDEA的SVN支持非常好，可以很方便地检查自己修改的代码、查询同事修改、对比文件版本，虽然TortoiseSVN和Eclipse也有类似功能，但是IDEA用起来更舒服。

以下也是IDEA的长处，不过有些是个人体验，或者未作为推销的内容向同事介绍：

1. IDEA的工作空间对版本控制友好，只要我把项目配置好，加到版本库中，其他人就可以直接用IDEA打开项目，等IDEA把文件索引完，然后再配一下JDK和Tomcat路径就能投入开发了。Eclipse的话每次全新checkout都要从头开始配置，而且我们项目组有时也会出现因工作安排变动或电脑难用而重新搭环境甚至重装系统的情况。
2. IDEA集成了数据库插件，代码提示比其他数据库软件好一些。另外修改数据时PL/SQL Developer需要附上FOR UPDATE，Toad需要用EDIT语句，SQL Developer需要自己写UPDATE语句，而IDEA可以直接在查询结果上面改。
3. IDEA对代码风格方面的提示和警告比Eclipse丰富，例如IDEA可以检测出哪些代码是一模一样的，或者哪些类与函数根本没有人使用。
4. IDEA自带反编译工具，可以方便地反编译代码，以分析一些jar包的原理。

当然，Eclipse的用户和插件也不少，如果能配置好，实际上也不会太难用。

# 差异
Eclipse和IDEA的界面与操作方法有些区别。概念上的重大差异主要是“Project Structure”，即项目文件结构的管理，不过如果有人把项目配置好并提交版本控制之后，其他成员反倒不太需要关注这些东西，因此我会另外写一篇文章进行叙述。

# 初始化配置
下面只讲Windows的操作方法（尽管截图是macOS）。对于Linux或苹果用户而言，我觉得既然能使用这两种系统，就说明不太需要这种详细教程吧。

## 安装IDEA
IDEA可以从[官网](https://www.jetbrains.com/idea/download/)下载，下载完成之后只要下一步下一步地操作就行。如果没钱买正版，请从[这里](https://www.jetbrains.com/idea/download/previous.html)找个稍微旧一点的版本，因为太新的版本可能没有破解工具或激活码。

安装结束后，如果没给JetBrains公司掏过钱，那么还需要从网上搜一下激活工具进行激活。

## 安装SVN客户端
SVN客户端推荐使用[TortoiseSVN](https://tortoisesvn.net/downloads.html)，因为和系统整合得比较好。安装时注意把“Command Line Tool”也安装上，否则后面IDEA和SVN配置会比较麻烦。

## 项目配置
下面假设项目代码库内已经有统一配置，后续会专门解释如何从头开始配置。

第一次打开项目时，系统需要花一段时间去建立索引，而第一次启动时系统也要花时间去编译和复制文件，但是这两项工作完成以后响应就比较快了。

在开始工作之前，需要检查和调整系统与项目的设置，例如SVN、JDK和Tomcat的路径：

### SVN路径
点击File下面的Settings菜单，进入系统设置界面，选择“Version Control”下面的“Subversion”，在右侧设置中进行SVN路径的设置。找到SVN的安装目录，然后去里面找bin和svn.exe。如果没有发现svn.exe，说明你在安装TortoiseSVN时未安装命令行工具，需要重新安装。

![SVN设置](/img/2018-12-01-switch-to-idea-1/svnconfig.png)

### JDK路径
点击File下面的Project Structure菜单，在弹出的对话框中选择左面的“Project”，可以看到类似下面的图：

![Project Structure](/img/2018-12-01-switch-to-idea-1/structure.png)

图中Project SDK为空，所以需要添加项目所使用的JDK，而且Project language level选择项目所用的JDK版本。下面Project compiler output则是项目编译之后文件的存放位置，只要别提交到SVN上面，放在哪里其实都无所谓，不过一定要设置，否则无法启动。

### Tomcat路径
点击Run下面的Edit Configuration菜单，进入启动设置界面，如下图所示：

![缺少Tomcat](/img/2018-12-01-switch-to-idea-1/tomcat-1.png)

点击Tomcat右边的Configure按钮，选择Tomcat安装路径：

![Tomcat设置](/img/2018-12-01-switch-to-idea-1/tomcat-2.png)

如果路径正确，IDEA会给出Tomcat的版本。

若有自定义的context.xml并在其中指定了数据源，那么还要在Tomcat的lib目录中加入相应的数据库驱动，例如[ojdbc6.jar](https://www.oracle.com/technetwork/apps-tech/jdbc-112010-090769.html)。

# 日常工作
## 保存
默认情况下IDEA会自动保存文件修改，所以在IDEA里面不需要按Ctrl+S，也找不到保存按钮。如果不希望自动保存的话，可以在系统设置里面把它关掉。

## 快捷键
快捷键可以从菜单上面查看，下面列举几个默认而且最常用的按键设置：

功能                           | Eclipse按键      | IDEA按键
-------------------------------|-----------------|---------------
全局搜索字符串                   | Ctrl+H          | Ctrl+Shift+F
全局替换字符串                   |                 | Ctrl+Shift+R
快速修复（设置Getter和Setter、设置import等） | Ctrl+1          | Alt+Enter
清理多余import                  | Ctrl+Shift+O                | Ctrl+Alt+O
根据文件名找文件                 | Ctrl+Shift+R    | 按两下Shift
查看变量定义/函数使用情况          | Ctrl+点击                | Ctrl+点击
查看实现                        | Ctrl+点击，或者在函数上面按住Ctrl，在弹出菜单中选择“Open Implementation” | Ctrl+Alt+点击
格式化                          | Ctrl+Shift+F               | Ctrl+Alt+L
单步执行，碰到函数就进到函数内部    | F5              | F7
单步执行，碰到函数直接返回结果     | F6               | F8
运行到当前函数的return部分        | F7              | Shift+F8
暂停之后继续运行程序              | F8              | F9

如果习惯了Eclipse的按键，也可以将快捷键调成Eclipse模式。具体方法是找到系统设置中的“Keymap”，然后在右面的设置中选择“Eclipse”方案，这样各功能快捷键就和Eclipse一致了。

## 调试
在第一次调试时，建议检查一下启动配置是否符合自己需要。

![启动配置](/img/2018-12-01-switch-to-idea-1/debug-1.png)

和Eclipse一样，如果想使用断点调试等功能，需要用虫子按钮而非普通的启动按钮来启动应用。点击启动之后，IDEA会进行编译并生成Web相关文件（见上图下侧Before launch部分，实际上IDEA之所以做这两件事是因为你在启动设置里面设置了Build和Build artifacts这两步操作），准备完成之后便会自动启动Tomcat。

另外也可以直接启动某一个类。启动之前那个类需要有一个静态的main函数，而且相关的配置文件要放对地方（例如src下面）。

## 版本控制
版本控制通常在IDEA的Version Control窗口中进行。如果未发现这个窗口，可点击“View”菜单下面“Tool Buttons”，然后检查窗口下面的工具栏按钮。

除了在Version Control窗口以外，也可以对着文件或代码点鼠标右键，在弹出的版本管理相关菜单上面进行操作。

### Changelist
IDEA的Version Control窗口可以很直观地列出你所进行的修改。如果发现修改未同步到列表中，可以点击左面的刷新按钮刷新一下（如果是新增文件，需要检查一下有没有将其加入到版本控制中）。

代码修改会以“Changelist”的形式组织。举个例子，假如修改了10个文件，有4个文件是为了处理问题A，有3个文件是为了处理问题B，而剩下3个文件是临时修改，不打算提交，这样可以建立三个Changelist（右击然后点击New changelist菜单），将三组文件分别拖拽到三个Changelist中，这样检查和提交的时候可以按Changelist分组进行。

![Changelist可以直观地列出你的代码改动，并根据实际需要分组管理。如果未及时刷新状态，可点击左面的刷新按钮。](/img/2018-12-01-switch-to-idea-1/changelist-1.png)

### 提交代码与回退
当准备进行提交时，只要在Version Control窗口对着Changelist或待提交文件（按Ctrl键多选）点右键，选择Commit，就会看到提交代码的窗口。

相反地，如果需要撤销更改，选择Revert菜单即可。

![检查待提交文件列表和内容，输入摘要，点击Commit提交。](/img/2018-12-01-switch-to-idea-1/changelist-2.png)

{% callout 针对新开发者的提醒 %}
以下内容转载自[编程随想的博客](https://program-think.blogspot.com/2009/02/6.html)：

据俺多年的观察，发觉 RCS（备注：版本管理软件）主要有如下几种误用的情况：

◇不正确的提交频度
　　有很多新手不习惯使用 RCS，很少进行代码的提交。有些人甚至到项目快结束时还没有提交过一行代码。结果导致整个 RCS 形同虚设。
　　还有一些人则走向另一个极端，频繁提交代码。有些人每改完一个文件就提交一次，还有些人甚至把修改一半，尚【不能】编译通过的代码也提交到 RCS。
　　俺个人认为，正确的提交频度应该分两种情况：在编写功能代码时，每完成一个功能点，并且自己经过了自测之后，提交一次；在调试的时候，每修复一个 bug，提交一次。这样能够保证提交到 RCS 的代码都是【能编译通过】的（详见[每日构建系列](https://program-think.blogspot.com/2009/02/daily-build-0-overview.html)），并且业务逻辑上也是相对完整的（对于每日构建后的测试很重要）。

◇提交时不写注释
　　很多人提交代码时不写注释，将来再想到版本历史里面找代码就犹如大海捞针般困难。
　　比如在俺经手的代码中，有些源代码文件历时3年，提交次数上百次，如果提交时不写注释，日后根本没法找。

◇不做代码基线（Baseline）、不做代码分支（Branch）
　　在正规的开发团队，每当有一个版本发布（Release）并交付给用户使用时，都需要在 RCS 中制作一个基线，以便于进行相应的 bug 跟踪和补丁制作。因此，诸如【做基线】之类的事情，属于整个团队的集体行为，需要由 Team Leader 牵头来搞。
　　假如一个开发团队从来不做代码基线或者代码分支，仅仅是把 RCS 当成一个源文件的备份工具来用，那至少说明这个团队的 Team Leader 在软件工程管理方面非常失败。

{% endcallout %}

### 查看文件提交历史
对着文件点击右键，选择“Subversion”里面的“Show History”，即可在下面的Version Control窗口看到文件的提交历史。对着版本点击右键，选择“Show Diff”，即可看到某个版本是哪个人进行了哪些修改。

除SVN提交历史外，也可以查看文件的本地编辑历史。假如不慎错误地修改了某些内容，又不方便直接回退文件，可对着文件点右键，找到“Local History”，查看本地的历史记录以便恢复。

### 查看代码库版本变更记录
Version Control第一个页签是你的待提交修改，第二个页签就是其他人已经提交的修改。如果没有看到提交记录，点击刷新按钮即可。

在变更记录窗口中，你可以选择提交记录，查看某一次提交涉及的修改内容。如果需要多版本审查，可以按Ctrl多选，右侧变更记录会将多条提交合并在一起。

<!--## 数据库操作
IDEA Ultimate版内置了DataGrip插件。
-->
