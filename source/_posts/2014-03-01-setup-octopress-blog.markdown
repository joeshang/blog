---
layout: post
title: 使用Octopress搭建博客
date: 2014-03-01 09:57:00 +0800
comments: true
categories: Tools
---

## 为什么选择Octopress

我一直是一个喜欢做笔记的人，觉得做笔记有以下优点：

- 在阅读时能够很好的帮助我思考，提炼精华，把握核心，在复习总结时非常有用
- 能把一些零散的知识通过分类给聚合起来，形成知识体系
- 每次看到满满一本全是笔记的时候，非常有成就感

之前一直是在用笔在本子上做笔记，比较喜欢自由涂鸦的那种感觉，但渐渐的发现了一些问题：

1. 纸制的保存时间有限，而且易丢失
2. 修改扩展起来比较麻烦
3. 检索、聚合起来不方便

后来试过像CSDN之类的博客，总觉得不好用，尤其是文本编辑，感觉非常麻烦，Vi用多了，喜欢纯文本编辑的乐趣与效率，搜索过自带格式的语言，Html/Xml更适合展示，大量的格式信息把文本“淹没”了，而Latex又过于复杂，直到通过[Github](http://github.com)学习了[Markdown](http://wowubuntu.com/markdown/)，终于找到了一个适合写作的语言了，在用Markdown写作时有种开心的感觉，就像拿LAMY的钢笔在MUJI的本子上写作一样，各种爱不释手，哈哈。为什么这么喜欢Markdown呢？我觉得有以下原因：

- 纯文本，所以兼容性强，可以用文本编辑器打开
- Markdown的标记语法简单清晰，具有很好的可读性，并且不会影响文本
- 专注于文字而不是排版
- 格式转换非常方便，可以轻松转换成Html、PDF等

用惯了Markdown之后开始YY，既然Markdown转换Html这么容易，如果能有支持Markdown的博客就好了，于是我Google了一下，发现了[Octopress](http://octopress.org)，一个非常酷的博客框架，而且可以deploy到Github Pages上，我擦，Github+Markdown，这一下子击中了我，所以立马用Octopress搭建了一个博客，以后要渐渐将思考、笔记记录到这个博客上了。

### 专注+坚持，一定能够有所收获！

下面就介绍下如何通过Octopress搭建Blog：

## Octopress搭建原理

Octopress的[文档](http://octopress.org/docs)上关于搭建步骤已经写的非常详细了，所以这里就不再重复。我是一个喜欢刨根问底的人（处女座），所以对搭建跟使用过程中的这些疑问很感兴趣：

- 在搭建过程中gem/bundle/rake都是干什么的，为什么`rake cmd`就执行相关操作？
- 为什么deploy到Github上的目录跟本地目录不一样（为什么有source跟master两个分支）？

### 搭建步骤原理

对于第一个问题，Octopress是基于ruby的，因此gem/bundle/rake都是ruby相关的一些工具，可以看一下[整理Ruby相关的各种概念](http://henter.me/post/ruby-rvm-gem-rake-bundle-rails.html)这篇文章了解下：

- Gem：ruby中的包管理器（Package Manager），就如同Ubuntu中的apt-get，方便管理与使用各类ruby包。但是一个项目有可能用很多包，总不能一个一个手动用Gem获取吧？于是Bundle出现了：
- Bundle：相当于多个Gem批处理运行。在配置文件Gemfile里说明应用依赖哪些第三方包，Bundle自动帮你下载安装多个包，并且会下载这些包依赖的包。
- Rake：Ruby Make，顾名思义，一个用ruby开发的代码构建工具。由于ruby不需要编译，所以Rake其实相当于一个任务管理工具：
  1. 以任务的方式创建和运行脚本（Rakefile）
  2. 追踪和管理任务间的依赖

所以我们就理解了安装步骤：

1. 首先用`gem install bundler`安装Bundle
2. 通过`bundle install`，Bundle读取Gemfile中Octopress需要的包，进行下载安装
3. 通过`rake cmd`，Rake读取Rakefile脚本中的内容，执行对应的任务

### Octopress部署框架

对于第二个问题，Octopress是一个生成器，它根据你的配置与主题，将Markdown格式的文档转化成HTML页面，生成出一个完成Blog（在`_deploy`目录中），所以应该将生成器与Markdown编写的内容同生成的Blog网页分开，于是在Github上存在两个分支：

- master：生成的Blog是期望访问username.github.io就访问到的，因此利用Github默认显示的master分支的特性，将生成的Blog设置为master
- source：Octopress生成器与用Markdown编写的内容放置在source分支

这样，对多台电脑同步Octopress博客的操作也明白为什么要这样做了：

### Octopress多台电脑同步方法

建立本地Octopress仓库，操作如下：

```
$ git clone -b source https://github.com/username/username.github.io octopress //先clone生成器与文章
$ cd octopress // 生成的Blog是被放置在生成器的一个子目录中的，因此要进入生成器目录
$ git clone https://github.com/username/username.github.io _deploy // 然后再clone所生成的内容
```

更新

```
$ cd octopress
$ git pull origin source # 更新local的source分支
$ cd _deploy
$ git pull origin master # 更新local的master分支
```

推送

```
$ rake generate
$ git add .
$ git commit -m "Commit Message"
$ git push origin source # 更新remote的source分支
$ rake deploy # 更新remote的master分支
```

### Octopress搭建过程中问题总结

#### 1. You have already activated rake 0.9.0, but your Gemfile requires rake 0.8.7

说明Rake版本不匹配，使用`bundle update rake`更新下Rake即可，具体见[这里](http://stackoverflow.com/questions/6080040/you-have-already-activated-rake-0-9-0-but-your-gemfile-requires-rake-0-8-7)。

#### 2. 如何在侧边栏显示Github仓库

仅仅在`_config.yml`中将`github_user`填上Github的用户名是不行的，还需要修改`default_asides`，将`asides/github.html`加入。
