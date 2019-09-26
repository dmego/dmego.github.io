---
title: 使用Travis CI自动部署Hexo到GitHub
date: 2017-10-13 17:46
author: Dmego
categories:
 - 技术
tags: 
 - Hexo
---
<!--more-->

## 前言

使用 `hexo + gitPages` 搭建个人博客的人都知道，每当要发表一篇博文，第一步得手动使用 `hexo g` 命令生成静态网页，然后还得通过 `hexo d` 命令将静态文件推送到GitHub远程仓库,不说麻烦不麻烦，更重要的是有时候环境换了，没有搭建 hexo 环境，想发篇博客的时候就没有可能了。而现在通过 Travis CI 就能自动构建自己的博客。我们只需将写好的 `Markdown` 格式的博文`push` 到 hexo源文件 分支即可。

## Travis CI 介绍

[Travis CI](https://travis-ci.org/) 是目前新兴的开源持续集成构建项目，它与 jenkins，GO的很明显的特别在于采用 yaml 格式，简洁清新独树一帜。目前大多数的 github 项目都已经移入到 Travis CI 的构建队列中，据说 Travis CI 每天运行超过 4000 次完整构建。

## hexo 介绍

[Hexo](https://hexo.io/) 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

## 使用 Travis 自动构建

我的博客自动部署思路是，将  `hexo` 源码 `push` 到博客 项目的另外一个分支，
既一个分支放源码，一个分支放静态文件，使用 `Travis CI` 自动部署 hexo 源码的分支，构建完成后自动推送到 静态文件的分支上，而这一切都在一个仓库上进行操作。<br>
注意：如果使用的是`GitPage`的个人站点来搭建博客的 ，则博客静态文件在 `master`分支上；如果使用的是 `gitPages` 的项目站点来搭建博客，则博客的静态文件在 `gh-pages` 分支上。

### 在GitHub 上生成 Access Token

如果想要 让`travis CI` 构建完成之后自动 push 到 master 分支，则travis需要有对这个仓库进行操作的权限，此时我们就需要为Travis CI 配置Access Token（访问令牌）。<br>

在GitHub上生成Access Token 的步骤是，点击头像进入设置（Settings）,r然后点击左边菜单栏最下面的`Developer settings` 选项，进入后点击左边的 `Personal access tokens` 选项，进入后点击右上角的`Generate new token` 按钮

![生成Access Token](deylpoy-hexo-with-TravisCI/G0hFA1LkK7.png)
点击后就会来到下面的界面，先给 Token 起一个名字，然后为它设置一些权限，其中红框内的权限是必须的，其他可以随意添加。

![设置一些权限](deylpoy-hexo-with-TravisCI/5G22L5hCcK.png)
点击下面的 `create token` 按钮，就会生成一个已经赋予好权限的 token 值，接下来我们Travis CI 网站的配置中。
![create token](deylpoy-hexo-with-TravisCI/fldkB30k3m.png)

### 配置 Travis CI

如果之前从未使用 [Travis CI](https://travis-ci.org/) 来构建项目，则我们先需要使用GitHub账号来登录网站,登录进来后，会进到如下图界面，如果底下 没有把 GitHub 仓库中的项目加载进来，可以手动点击右上角的  `Sync account` 按钮，待到同步完成后将要自动构建的项目开启。
![将要自动构建的项目开启](deylpoy-hexo-with-TravisCI/0IbbdiJh18.png)
开启后点击设置图标就可以进行一系列的设置，如下图所示，先开启 `General` 里的两项选项：

- `Build only if .travis.yml is present`:只有在`.travis.yml`文件中配置的分支改变了才构建
- `Build branch updates`:当分支更新后开始构建

然后在  `Environment Variables` 一栏里将在 GitHub 下获取的 `Access Token` 值添加进来
![将Access Token值添加进来](deylpoy-hexo-with-TravisCI/3b875iHdi4.png)

### 添加配置文件到Hexo源码分支下

上面提到的 `.travis.yml` 配置文件需要添加到hexo 源码的根目录下，因为Travis CI 在自动构建时需要获取这些配置将信息，以此来完成构建任务；这些配置信息主要包括源码分支，静态文件推送分支，仓库地址等信息。
![添加.travis.yml 配置文件](deylpoy-hexo-with-TravisCI/CaBF4laGji.png)
其中主要内容如下：

```yml
language: node_js
node_js: stable

# S: Build Lifecycle
install:
  - npm install

#before_script:
 # - npm install -g gulp

script:
  - hexo g

after_script:
  - cd ./public
  - git init
  - git config user.name "dmego" --{GitHub账户名称}
  - git config user.email "zengkai12138@outlook.com" --{Github账户邮箱}
  - git add .
  - git commit -m "Update docs"
  - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master
# E: Build LifeCycle

branches:
  only:
    - hexo --{Hexo源码分支名称}
env:
 global:
   - GH_REF: github.com/dmego/dmego.github.io.git --{仓库地址}
```

配置到这一步就已经把所有配置全部完成，下面就是验证的过程

## 构建并自动部署结果

将某篇文章中的一个表格增加一行后将修改推送到hexo源码所在的`hexo`分支
,然后等Travis CI 构建并自动部署成功后。
![自动部署成功](deylpoy-hexo-with-TravisCI/F83mk0a09k.png)
点击博文发现表格多了一行。
![博文表格多了一行](deylpoy-hexo-with-TravisCI/hk2hCAma3D.png)

## 总结

这样做虽然能很好的实现自动部署的功能，但是有个问题也要注意，就是博客源码公开问题，如果对博客源码不介意的可以直接使用公开仓库，如果介意那就没有办法了，除非使用付费的私有仓库，或者把项目放在`Coding`上去，因为上有提供免费的私有仓库。就我个人认为，既然是自己的博客，本来就是要给人看的，博客源码也谈不上存在什么隐私。
