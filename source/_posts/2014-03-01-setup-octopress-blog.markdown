---
title: 使用 Octopress 搭建博客
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

后来试过像 CSDN 之类的博客，总觉得不好用，尤其是文本编辑，感觉非常麻烦， Vim 用多了，喜欢纯文本编辑的乐趣与效率，搜索过自带格式的语言，HTML 更适合展示，大量的格式信息把文本“淹没”了，而 Latex 又过于复杂，直到通过 [Github](http://github.com) 学习了 [Markdown](http://wowubuntu.com/markdown/)，终于找到了一个适合写作的语言了，在用 Markdown 写作时有种开心的感觉，就像拿 LAMY 的钢笔在 MUJI 的本子上写作一样，各种爱不释手，哈哈。为什么这么喜欢 Markdown呢？我觉得有以下原因：

- 纯文本，所以兼容性强，可以用文本编辑器打开
- Markdown 的标记语法简单清晰，具有很好的可读性，并且不会影响文本
- 专注于文字而不是排版
- 格式转换非常方便，可以轻松转换成 HTML、PDF 等

用惯了 Markdown 之后开始 YY，既然 Markdown 转换 HTML 这么容易，如果能有支持 Markdown 的博客就好了，于是我 Google 了一下，发现了 [Octopress](http://octopress.org)，一个非常酷的博客框架，而且可以 deploy 到 Github Pages 上，我擦，Github + Markdown，这一下子击中了我，所以立马用 Octopress 搭建了一个博客，以后要渐渐将思考、笔记记录到这个博客上了。

### 专注+坚持，一定能够有所收获！

下面就介绍下如何通过 Octopress 搭建 Blog：

## Octopress搭建原理

Octopress 的[文档](http://octopress.org/docs)上关于搭建步骤已经写的非常详细了，所以这里就不再重复。我是一个喜欢刨根问底的人（处女座），所以对搭建跟使用过程中的这些疑问很感兴趣：

- 在搭建过程中 gem/bundle/rake 都是干什么的，为什么 `rake cmd` 就执行相关操作？
- 为什么 deploy 到 Github 上的目录跟本地目录不一样（为什么有 source 跟 master 两个分支）？

### 搭建步骤原理

对于第一个问题，Octopress 是基于 ruby 的，因此 gem/bundle/rake 都是 ruby 相关的一些工具，可以看一下[整理 Ruby 相关的各种概念](http://henter.me/post/ruby-rvm-gem-rake-bundle-rails.html)这篇文章了解下：

- Gem：ruby 中的包管理器（Package Manager），就如同 Ubuntu 中的 apt-get，方便管理与使用各类 ruby 包。但是一个项目有可能用很多包，总不能一个一个手动用 Gem 获取吧？于是 Bundle 出现了：
- Bundle：相当于多个 Gem 批处理运行。在配置文件 Gemfile 里说明应用依赖哪些第三方包，Bundle 自动帮你下载安装多个包，并且会下载这些包依赖的包。
- Rake：Ruby Make，顾名思义，一个用 ruby 开发的代码构建工具。由于 ruby 不需要编译，所以 Rake 其实相当于一个任务管理工具：
  1. 以任务的方式创建和运行脚本（Rakefile）
  2. 追踪和管理任务间的依赖

所以我们就理解了安装步骤：

1. 首先用 `gem install bundler` 安装 Bundle
2. 通过 `bundle install`，Bundle 读取 Gemfile 中 Octopress 需要的包，进行下载安装
3. 通过 `rake cmd`，Rake 读取 Rakefile 脚本中的内容，执行对应的任务

### Octopress部署框架

对于第二个问题，Octopress 是一个生成器，它根据你的配置与主题，将 Markdown 格式的文档转化成 HTML 页面，生成出一个完成 Blog（在 `_deploy` 目录中），所以应该将生成器与 Markdown 编写的内容同生成的 Blog 网页分开，于是在 Github 上存在两个分支：

- master：生成的 Blog 是期望访问 username.github.io 就访问到的，因此利用 Github 默认显示的 master 分支的特性，将生成的 Blog 设置为 master
- source：Octopress 生成器与用 Markdown 编写的内容放置在 source 分支

这样，对多台电脑同步 Octopress 博客的操作也明白为什么要这样做了：

### Octopress 多台电脑同步方法

建立本地 Octopress 仓库，操作如下：

```
$ git clone -b source https://github.com/username/username.github.io octopress //先 clone 生成器与文章
$ cd octopress // 生成的 Blog 是被放置在生成器的一个子目录中的，因此要进入生成器目录
$ git clone https://github.com/username/username.github.io _deploy // 然后再 clone 所生成的内容
```

更新

```
$ cd octopress
$ git pull origin source # 更新 local 的 source 分支
$ cd _deploy
$ git pull origin master # 更新 local 的 master 分支
```

推送

```
$ rake generate
$ git add .
$ git commit -m "Commit Message"
$ git push origin source # 更新 remote 的 source 分支
$ rake deploy # 更新 remote 的 master 分支
```

### Octopress 搭建过程中问题总结

#### 1. You have already activated rake 0.9.0, but your Gemfile requires rake 0.8.7

说明 Rake 版本不匹配，使用 `bundle update rake` 更新下 Rake 即可，具体见[这里](http://stackoverflow.com/questions/6080040/you-have-already-activated-rake-0-9-0-but-your-gemfile-requires-rake-0-8-7)。

#### 2. 如何在侧边栏显示 Github 仓库

仅仅在 `_config.yml` 中将 `github_user` 填上 Github 的用户名是不行的，还需要修改 `default_asides`，将 `asides/github.html` 加入。



